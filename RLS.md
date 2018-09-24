# Row-Level Security


  ![Row level security graphic](https://docs.microsoft.com/en-us/sql/relational-databases/security/media/row-level-security-graphic.png "Row level security graphic")  
  
 Row-Level Security enables customers to control access to rows in a database table based on the characteristics of the user executing a query (e.g., group membership or execution context).  
  
 Row-Level Security (RLS) simplifies the design and coding of security in your application. RLS enables you to implement restrictions on data row access. For example ensuring that workers can access only those data rows that are pertinent to their department, or restricting a customer's data access to only the data relevant to their company.  
  
 The access restriction logic is located in the database tier rather than away from the data in another application tier. The database system applies the access restrictions every time that data access is attempted from any tier. This makes your security system more reliable and robust by reducing the surface area of your security system.  


##  <a name="Description"></a> Description  
 RLS supports two types of security predicates.  
  
-   Filter predicates silently filter the rows available to read operations (SELECT, UPDATE, and DELETE).  
  
-   Block predicates explicitly block write operations (AFTER INSERT, AFTER UPDATE, BEFORE UPDATE, BEFORE DELETE) that violate the predicate.  
  
 Access to row-level data in a table is restricted by a security predicate defined as an inline table-valued function. The function is then invoked and enforced by a security policy. For filter predicates, there is no indication to the application that rows have been filtered from the result set; if all rows are filtered, then a null set will be returned. For block predicates, any operations that violate the predicate will fail with an error.  
  
 Filter predicates are applied while reading data from the base table, and it affects all get operations: **SELECT**, **DELETE** (i.e. user cannot delete rows that are filtered), and **UPDATE** (i.e. user cannot update rows that are filtered, although it is possible to update rows in such way that they will be subsequently filtered). Block predicates affect all write operations.  
  
-   AFTER INSERT and AFTER UPDATE predicates can prevent users from updating rows to values that violate the predicate.  
  
-   BEFORE UPDATE predicates can prevent users from updating rows that currently violate the predicate.  
  
-   BEFORE DELETE predicates can block delete operations.  
  
 Both filter and block predicates and security policies have the following behavior:  
  
-   You may define a predicate function that joins with another table and/or invokes a function. If the security policy is created with `SCHEMABINDING = ON`, then the join or function is accessible from the query and works as expected without any additional permission checks. If the security policy is created with `SCHEMABINDING = OFF`, then users will need **SELECT** or **EXECUTE** permissions on these additional tables and functions in order to query the target table.
  
-   You may issue a query against a table that has a security predicate defined but disabled. Any rows that would have been filtered or blocked are not affected.  
  
-   If the dbo user, a member of the **db_owner** role, or the table owner queries against a table that has a security policy defined and enabled, the rows are filtered or blocked as defined by the security policy.  
  
-   Attempts to alter the schema of a table bound by a schema bound security policy will result in an error. However, columns not referenced by the predicate can be altered.  
  
-   Attempts to add a predicate on a table that already has one defined for the specified operation (regardless of whether it is enabled or disabled) results in an error.  
  
-   For schema bound security policies, attempts to modify a function used as a predicate on a table within a security policy results in an error.  
  
-   Defining multiple active security policies that contain non-overlapping predicates, succeeds.  
  
 Filter predicates have the following behavior:  
  
-   Define a security policy that filters the rows of a table. The application is unaware that any rows have been filtered for **SELECT**, **UPDATE**, and **DELETE** operations, including situations where all the rows have been filtered out. The application can **INSERT** any rows, regardless of whether or not they will be filtered during any other operation.  
  
 Block predicates have the following behavior:  
  
-   Block predicates for UPDATE are split into separate operations for BEFORE and AFTER. Consequently, you cannot, for example, block users from updating a row to have a value higher than the current one. If this kind of logic is required, you must use triggers with the DELETED and INSERTED intermediate tables to reference the old and new values together.  
  
-   The optimizer will not check an AFTER UPDATE block predicate if none of the columns used by the predicate function were changed. For example: Alice should not be able to change a salary to be greater than 100,000, but she should be able to change the address of an employee whose salary is already greater than 100,000 (and thus already violates the predicate).  
  
-   No changes have been made to the bulk APIs, including BULK INSERT. This means that block predicates AFTER INSERT will apply to bulk insert operations just as they would regular insert operations.  
  
  
##  <a name="UseCases"></a> Use Cases  
 Here are design examples of how RLS can be used:  
  
-   A hospital can create a security policy that allows nurses to view data rows for their own patients only.  
  
-   A bank can create a policy to restrict access to rows of financial data based on the employee's business division, or based on the employee's role within the company.  
  
-   A multi-tenant application can create a policy to enforce a logical separation of each tenant's data rows from every other tenant's rows. Efficiencies are achieved by the storage of data for many tenants in a single table. Of course, each tenant can see only its data rows.  
  
 RLS filter predicates are functionally equivalent to appending a **WHERE** clause. The predicate can be as sophisticated as business practices dictate, or the clause can be as simple as `WHERE TenantId = 42`.  
  
 In more formal terms, RLS introduces predicate based access control. It features a flexible, centralized, predicate-based evaluation that can take into consideration metadata or any other criteria the administrator determines as appropriate. The predicate is used as a criterion to determine whether or not the user has the appropriate access to the data based on user attributes. Label-based access control can be implemented by using predicate-based access control.  
  
##  <a name="Typical"></a> A. Scenario for users who authenticate to the database  
 This short example creates three users, creates and populates a table with 6 rows, then creates an inline table valued function and a security policy for the table. The example shows how select statements are filtered for the various users.  
  
 Create three user accounts that will demonstrate different access capabilities.  
  
```sql  
CREATE USER Manager WITHOUT LOGIN;  
CREATE USER Sales1 WITHOUT LOGIN;  
CREATE USER Sales2 WITHOUT LOGIN;  
```  
  
 Create a simple table to hold data.  
  
```  
CREATE TABLE Sales  
    (  
    OrderID int,  
    SalesRep sysname,  
    Product varchar(10),  
    Qty int  
    );  
```  
  
 Populate the table with 6 rows of data, showing 3 orders for each sales representative.  
  
```  
INSERT Sales VALUES   
(1, 'Sales1', 'Valve', 5),   
(2, 'Sales1', 'Wheel', 2),   
(3, 'Sales1', 'Valve', 4),  
(4, 'Sales2', 'Bracket', 2),   
(5, 'Sales2', 'Wheel', 5),   
(6, 'Sales2', 'Seat', 5);  
-- View the 6 rows in the table  
SELECT * FROM Sales;  
```  
  
 Grant read access on the table to each of the users.  
  
```  
GRANT SELECT ON Sales TO Manager;  
GRANT SELECT ON Sales TO Sales1;  
GRANT SELECT ON Sales TO Sales2;  
```  
  
 Create a new schema, and an inline table valued function. The function returns 1 when a row in the SalesRep column is the same as the user executing the query (`@SalesRep = USER_NAME()`) or if the user executing the query is the Manager user (`USER_NAME() = 'Manager'`).  
  
```  
CREATE SCHEMA Security;  
GO  
  
CREATE FUNCTION Security.fn_securitypredicate(@SalesRep AS sysname)  
    RETURNS TABLE  
WITH SCHEMABINDING  
AS  
    RETURN SELECT 1 AS fn_securitypredicate_result   
WHERE @SalesRep = USER_NAME() OR USER_NAME() = 'Manager';  
```  
  
 Create a security policy adding the function as a filter predicate. The state must be set to ON to enable the policy.  
  
```  
CREATE SECURITY POLICY Security.SalesFilter  
ADD FILTER PREDICATE Security.fn_securitypredicate(SalesRep)   
ON dbo.Sales  
WITH (STATE = ON);  
```  
  
 Now test the filtering predicate, by selected from the Sales table as each user.  
  
```  
EXECUTE AS USER = 'Sales1';  
SELECT * FROM Sales;   
REVERT;  
  
EXECUTE AS USER = 'Sales2';  
SELECT * FROM Sales;   
REVERT;  
  
EXECUTE AS USER = 'Manager';  
SELECT * FROM Sales;   
REVERT;  
```  
  
 The Manager should see all 6 rows. The Sales1 and Sales2 users should only see their own sales.  
  
 Alter the security policy to disable the policy.  
  
```  
ALTER SECURITY POLICY SalesFilter  
WITH (STATE = OFF);  
```  
  
 Now the Sales1 and Sales2 users can see all 6 rows.  

 ### Clean-up

 Use the following code to clean up the objects created during the previous exercise.

 ```
 DROP SECURITY POLICY Security.SalesFilter;
GO

DROP FUNCTION Security.fn_securitypredicate;
GO 

DROP SCHEMA Security;
GO 

DROP TABLE Sales;
GO

DROP USER Manager;
DROP USER Sales1;
DROP USER Sales2;
GO
 ```
  
  
###  <a name="MidTier"></a> B. Scenario for users who connect to the database through a middle-tier application  
 This example shows how a middle-tier application can implement connection filtering, where application users (or tenants) share the same [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] user (the application). The application sets the current application user ID in [SESSION_CONTEXT &#40;Transact-SQL&#41;](../../t-sql/functions/session-context-transact-sql.md) after connecting to the database, and then security policies transparently filter rows that shouldn't be visible to this ID, and also block the user from inserting rows for the wrong user ID. No other app changes are necessary .  
  
 Create a simple table to hold data.  
  
```  
CREATE TABLE Sales (  
    OrderId int,  
    AppUserId int,  
    Product varchar(10),  
    Qty int  
);  
```  
  
 Populate the table with 6 rows of data, showing 3 orders for each application user.  
  
```  
INSERT Sales VALUES   
    (1, 1, 'Valve', 5),   
    (2, 1, 'Wheel', 2),   
    (3, 1, 'Valve', 4),  
    (4, 2, 'Bracket', 2),   
    (5, 2, 'Wheel', 5),   
    (6, 2, 'Seat', 5);  
```  
  
 Create a low-privileged user that the application will use to connect.  
  
```  
-- Without login only for demo  
CREATE USER AppUser WITHOUT LOGIN;   
GRANT SELECT, INSERT, UPDATE, DELETE ON Sales TO AppUser;  
  
-- Never allow updates on this column  
DENY UPDATE ON Sales(AppUserId) TO AppUser;  
```  
  
 Create a new schema and predicate function, which will use the application user ID stored in **SESSION_CONTEXT** to filter rows.  
  
```  
CREATE SCHEMA Security;  
GO  
  
CREATE FUNCTION Security.fn_securitypredicate(@AppUserId int)  
    RETURNS TABLE  
    WITH SCHEMABINDING  
AS  
    RETURN SELECT 1 AS fn_securitypredicate_result  
    WHERE  
        DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('AppUser')    
        AND CAST(SESSION_CONTEXT(N'UserId') AS int) = @AppUserId;   
GO  
```  
  
 Create a security policy that adds this function as a filter predicate and a block predicate on `Sales`. The block predicate only needs **AFTER INSERT**, because **BEFORE UPDATE** and **BEFORE DELETE** are already filtered, and **AFTER UPDATE** is unnecessary because the `AppUserId` column cannot be updated to other values, due to the column permission set earlier.  
  
```  
CREATE SECURITY POLICY Security.SalesFilter  
    ADD FILTER PREDICATE Security.fn_securitypredicate(AppUserId)   
        ON dbo.Sales,  
    ADD BLOCK PREDICATE Security.fn_securitypredicate(AppUserId)   
        ON dbo.Sales AFTER INSERT   
    WITH (STATE = ON);  
```  
  
 Now we can simulate the connection filtering by selecting from the `Sales` table after setting different user IDs in **SESSION_CONTEXT**. In practice, the application is responsible for setting the current user ID in **SESSION_CONTEXT** after opening a connection.  
  
```  
EXECUTE AS USER = 'AppUser';  
EXEC sp_set_session_context @key=N'UserId', @value=1;  
SELECT * FROM Sales;  
GO  
  
--  Note: @read_only prevents the value from changing again   
--  until the connection is closed (returned to the connection pool)  
EXEC sp_set_session_context @key=N'UserId', @value=2, @read_only=1;   
  
SELECT * FROM Sales;  
GO  
  
INSERT INTO Sales VALUES (7, 1, 'Seat', 12); -- error: blocked from inserting row for the wrong user ID  
GO  
  
REVERT;  
GO  
```  

### Clean-up

 Use the following code to clean up the objects created during the previous exercise.

 ```
 DROP SECURITY POLICY Security.SalesFilter;
GO

DROP FUNCTION Security.fn_securitypredicate;
GO 

DROP SCHEMA Security;
GO 

DROP TABLE Sales;
GO

DROP USER AppUser;
GO
 ```