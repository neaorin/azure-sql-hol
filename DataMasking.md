# Azure SQL Database dynamic data masking

SQL Database dynamic data masking limits sensitive data exposure by masking it to non-privileged users. 

Dynamic data masking helps prevent unauthorized access to sensitive data by enabling customers to designate how much of the sensitive data to reveal with minimal impact on the application layer. It’s a policy-based security feature that hides the sensitive data in the result set of a query over designated database fields, while the data in the database is not changed.

For example, a service representative at a call center may identify callers by several digits of their credit card number, but those data items should not be fully exposed to the service representative. A masking rule can be defined that masks all but the last four digits of any credit card number in the result set of any query. As another example, an appropriate data mask can be defined to protect personally identifiable information (PII) data, so that a developer can query production environments for troubleshooting purposes without violating compliance regulations.

## SQL Database dynamic data masking basics
You set up a dynamic data masking policy in the Azure portal by selecting the dynamic data masking operation in your SQL Database configuration blade or settings blade.

### Dynamic data masking permissions
Dynamic data masking can be configured by the Azure Database admin, server admin, or security officer roles.

### Dynamic data masking policy
* **SQL users excluded from masking** - A set of SQL users or AAD identities that get unmasked data in the SQL query results. Users with administrator privileges are always excluded from masking, and see the original data without any mask.
* **Masking rules** - A set of rules that define the designated fields to be masked and the masking function that is used. The designated fields can be defined using a database schema name, table name, and column name.
* **Masking functions** - A set of methods that control the exposure of data for different scenarios.

| Masking Function | Masking Logic |
| --- | --- |
| **Default** |**Full masking according to the data types of the designated fields**<br/><br/>• Use XXXX or fewer Xs if the size of the field is less than 4 characters for string data types (nchar, ntext, nvarchar).<br/>• Use a zero value for numeric data types (bigint, bit, decimal, int, money, numeric, smallint, smallmoney, tinyint, float, real).<br/>• Use 01-01-1900 for date/time data types (date, datetime2, datetime, datetimeoffset, smalldatetime, time).<br/>• For SQL variant, the default value of the current type is used.<br/>• For XML the document <masked/> is used.<br/>• Use an empty value for special data types (timestamp table, hierarchyid, GUID, binary, image, varbinary spatial types). |
| **Credit card** |**Masking method, which exposes the last four digits of the designated fields** and adds a constant string as a prefix in the form of a credit card.<br/><br/>XXXX-XXXX-XXXX-1234 |
| **Email** |**Masking method, which exposes the first letter and replaces the domain with XXX.com** using a constant string prefix in the form of an email address.<br/><br/>aXX@XXXX.com |
| **Random number** |**Masking method, which generates a random number** according to the selected boundaries and actual data types. If the designated boundaries are equal, then the masking function is a constant number.<br/><br/>![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/1_DDM_Random_number.png) |
| **Custom text** |**Masking method, which exposes the first and last characters** and adds a custom padding string in the middle. If the original string is shorter than the exposed prefix and suffix, only the padding string is used. <br/>prefix[padding]suffix<br/><br/>![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/2_DDM_Custom_text.png) |

<a name="Anchor1"></a>

### Recommended fields to mask
The DDM recommendations engine, flags certain fields from your database as potentially sensitive fields, which may be good candidates for masking. In the Dynamic Data Masking blade in the portal, you will see the recommended columns for your database. All you need to do is click **Add Mask** for one or more columns and then **Save** to apply a mask for these fields.


## Set up dynamic data masking for your database using the Azure portal
1. Launch the Azure portal at [https://portal.azure.com](https://portal.azure.com).
2. Navigate to the settings page of the database that includes the sensitive data you want to mask.
3. Click the **Dynamic Data Masking** tile that launches the **Dynamic Data Masking** configuration page.
   
   * Alternatively, you can scroll down to the **Operations** section and click **Dynamic Data Masking**.
     
     ![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/4_ddm_settings_tile.png)<br/><br/>
4. In the **Dynamic Data Masking** configuration page, you may see some database columns that the recommendations engine has flagged for masking. In order to accept the recommendations, just click **Add Mask** for one or more columns and a mask is created based on the default type for this column. You can change the masking function by clicking on the masking rule and editing the masking field format to a different format of your choice. Be sure to click **Save** to save your settings. 
  
    ![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/5_ddm_recommendations.png)<br/><br/>
5. To add a mask for any column in your database, at the top of the **Dynamic Data Masking** configuration page, click **Add Mask** to open the **Add Masking Rule** configuration page.
   
    ![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/6_ddm_add_mask.png)<br/><br/>
6. Select the **Schema**, **Table** and **Column** to define the designated field for masking.
7. Choose a **Masking Field Format** from the list of sensitive data masking categories.
   
    ![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/7_ddm_mask_field_format.png)<br/><br/>        
8. Click **Save** in the data masking rule page to update the set of masking rules in the dynamic data masking policy.
9. Type the SQL users or AAD identities that should be excluded from masking, and have access to the unmasked sensitive data. This should be a semicolon-separated list of users. Users with administrator privileges always have access to the original unmasked data.
   
    ![Navigation pane](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-dynamic-data-masking-get-started/8_ddm_excluded_users.png)
   
   > [!TIP]
   > To make it so the application layer can display sensitive data for application privileged users, add the SQL user or AAD identity the application uses to query the database. It is highly recommended that this list contain a minimal number of privileged users to minimize exposure of the sensitive data.
   > 
   > 
10. Click **Save** in the data masking configuration page to save the new or updated masking policy.

11. Now you can test the masking rules, by connecting with the unprivileged user you created during the [Create an Azure SQL database](CreateDatabase.md) part of this workshop. You can login to the server with the user **Mary**, and run a SELECT query on the masked columns.

   > [!NOTE]
   > If you're getting a *Login Failed for user 'Mary'* when trying to login via SSMS, you need to select the **Connection Properties** tab and enter the name of the database you're trying to connect to.
   > 
   ![ConnectTo](./media/connecttodb.jpg)

For instance, suppose you chose to mask the **EmailAddress** column from the **Customer** table. The query below will return masked values when executed as *Mary*, but will return the unmasked values when run as your admin user:

```sql
select * from SalesLT.Customer
```

![DataMasking](./media/datamasking.jpg)
