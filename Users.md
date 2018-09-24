# Controlling and granting database access to SQL Database and SQL Data Warehouse

After firewall rules configuration, you can connect to Azure [SQL Database](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview) and [SQL Data Warehouse](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) as one of the administrator accounts, as the database owner, or as a database user in the database.  

>  [!NOTE]  
>  This topic applies to Azure SQL server, and to SQL Database and SQL Data Warehouse databases created on the Azure SQL server. For simplicity, SQL Database is used when referring to both SQL Database and SQL Data Warehouse. 

> [!TIP]
> For a tutorial, see [Secure your Azure SQL Database](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-security-tutorial).

## Unrestricted administrative accounts
There are two administrative accounts (**Server admin** and **Active Directory admin**) that act as administrators. To identify these administrator accounts for your SQL server, open the Azure portal, and navigate to the properties of your SQL server.

![SQL Server Admins](https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-manage-logins/sql-admins.png)

- **Server admin**   
When you create an Azure SQL server, you must designate a **Server admin login**. SQL server creates that account as a login in the master database. This account connects using SQL Server authentication (user name and password). Only one of these accounts can exist.   
- **Azure Active Directory admin**   
One Azure Active Directory account, either an individual or security group account, can also be configured as an administrator. It is optional to configure an Azure AD administrator, but an Azure AD administrator **must** be configured if you want to use Azure AD accounts to connect to SQL Database. For more information about configuring Azure Active Directory access, see [Connecting to SQL Database or SQL Data Warehouse By Using Azure Active Directory Authentication](sql-database-aad-authentication.md) and [SSMS support for Azure AD MFA with SQL Database and SQL Data Warehouse](sql-database-ssms-mfa-authentication.md).
 

The **Server admin** and **Azure AD admin** accounts have the following characteristics:
- Are the only accounts that can automatically connect to any SQL Database on the server. (To connect to a user database, other accounts must either be the owner of the database, or have a user account in the user database.)
- These accounts enter user databases as the `dbo` user and they have all the permissions in the user databases. (The owner of a user database also enters the database as the `dbo` user.) 
- Do not enter the `master` database as the `dbo` user, and have limited permissions in master. 
- Are **not** members of the standard SQL Server `sysadmin` fixed server role, which is not available in SQL database.  
- Can create, alter, and drop databases, logins, users in master, and server-level firewall rules.
- Can add and remove members to the `dbmanager` and `loginmanager` roles.
- Can view the `sys.sql_logins` system table.


### Administrator access path
When the server-level firewall is properly configured, the **SQL server admin** and the **Azure Active Directory admin** can connect using client tools such as SQL Server Management Studio or SQL Server Data Tools. Only the latest tools provide all the features and capabilities. The following diagram shows a typical configuration for the two administrator accounts.

![Administrator access path](https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-manage-logins/1sql-db-administrator-access.png)

When using an open port in the server-level firewall, administrators can connect to any SQL Database.


## Additional server-level administrative roles
In addition to the server-level administrative roles discussed previously, SQL Database provides two restricted administrative roles in the master database to which user accounts can be added that grant permissions to either create databases or manage logins.

### Database creators
One of these administrative roles is the **dbmanager** role. Members of this role can create new databases. To use this role, you create a user in the `master` database and then add the user to the **dbmanager** database role. To create a database, the user must be a user based on a SQL Server login in the master database or contained database user based on an Azure Active Directory user.

### Login managers
The other administrative role is the login manager role. Members of this role can create new logins in the master database. If you wish, you can complete the same steps (create a login and user, and add a user to the **loginmanager** role) to enable a user to create new logins in the master. Usually logins are not necessary as Microsoft recommends using contained database users, which authenticate at the database-level instead of using users based on logins. For more information, see [Contained Database Users - Making Your Database Portable](https://msdn.microsoft.com/library/ff929188.aspx).

## Non-administrator users
Generally, non-administrator accounts do not need access to the master database. Create contained database users at the database level using the [CREATE USER (Transact-SQL)](https://msdn.microsoft.com/library/ms173463.aspx) statement. The user can be an Azure Active Directory authentication contained database user (if you have configured your environment for Azure AD authentication), or a SQL Server authentication contained database user, or a SQL Server authentication user based on a SQL Server authentication login (created in the previous step.) For more information, see [Contained Database Users - Making Your Database Portable](https://msdn.microsoft.com/library/ff929188.aspx). 

### Create another user with limited rights

Some sections of this workshop require the existence of a user which is not database owner, but will have data reading rights. Let's create this user right now.

1. Connect to the **AdventureWorks** database, using either the built-in portal query tool as above, or by using **SQL Server Management Studio**. 

2. If you used **SQL Server Management Studio**, after you are authenticated as admin, switch to the **AdventureWorks** database.

3. Type the following query in the query editor pane.

```sql
CREATE USER Mary WITH PASSWORD = 'SomePassword234%^&';
ALTER ROLE db_datareader ADD MEMBER Mary; 
```

4. Make sure the query has successfully completed.



## Configure and manage Azure Active Directory authentication with SQL Database, Managed Instance, or SQL Data Warehouse

This article shows you how to create and populate Azure AD, and then use Azure AD with Azure [SQL Database](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview) and [SQL Data Warehouse](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is). For an overview, see [Azure Active Directory Authentication](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-aad-authentication).


## Create an Azure AD administrator for Azure SQL server
Each Azure SQL server (which hosts a SQL Database or SQL Data Warehouse) starts with a single server administrator account that is the administrator of the entire Azure SQL server. A second SQL Server administrator must be created, that is an Azure AD account. This principal is created as a contained database user in the master database. As administrators, the server administrator accounts are members of the **db_owner** role in every user database, and enter each user database as the **dbo** user. 

When using Azure Active Directory with geo-replication, the Azure Active Directory administrator must be configured for both the primary and the secondary servers. If a server does not have an Azure Active Directory administrator, then Azure Active Directory logins and users receive a "Cannot connect" to server error.

> [!NOTE]
> Users that are not based on an Azure AD account (including the Azure SQL server administrator account), cannot create Azure AD-based users, because they do not have permission to validate proposed database users with the Azure AD.
> 

## Provision an Azure Active Directory administrator for your Azure SQL Database server


### Azure portal
1. In the [Azure portal](https://portal.azure.com/), in the upper-right corner, select your connection to drop down a list of possible Active Directories. Choose the correct Active Directory as the default Azure AD. This step links the subscription-associated Active Directory with Azure SQL server making sure that the same subscription is used for both Azure AD and SQL Server. (The Azure SQL server can be hosting either Azure SQL Database or Azure SQL Data Warehouse.)   
    ![choose-ad][8]   
    
2. In the left banner select **All services**, and in the filter type in **SQL server**. Select **Sql Servers**. 

    ![sqlservers.png](media/sql-database-aad-authentication/sqlservers.png)    

    >[!NOTE]
    > On this page, before you select **SQL servers**, you can select the **star** next to the name to *favorite* the category and add **SQL servers** to the left navigation bar. 

1. On **SQL Server** page, select **Active Directory admin**.   
2. In the **Active Directory admin** page, select **Set admin**.   
    ![select active directory](https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/select-active-directory.png)  
    
4. In the **Add admin** page, search for a user, select the user or group to be an administrator, and then select **Select**. (The Active Directory admin page shows all members and groups of your Active Directory. Users or groups that are grayed out cannot be selected because they are not supported as Azure AD administrators. (See the list of supported admins in the **Azure AD Features and Limitations** section of [Use Azure Active Directory Authentication for authentication with SQL Database or SQL Data Warehouse](sql-database-aad-authentication.md).) Role-based access control (RBAC) applies only to the portal and is not propagated to SQL Server.   
    ![select admin](https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/select-admin.png)  
    
5. At the top of the **Active Directory admin** page, select **SAVE**.   
    ![save admin](https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/save-admin.png)   

The process of changing the administrator may take several minutes. Then the new administrator appears in the **Active Directory admin** box.

   > [!NOTE]
   > When setting up the Azure AD admin, the new admin name (user or group) cannot already be present in the virtual master database as a SQL Server authentication user. If present, the Azure AD admin setup will fail; rolling back its creation and indicating that such an admin (name) already exists. Since such a SQL Server authentication user is not part of the Azure AD, any effort to connect to the server using Azure AD authentication fails.
   > 


To later remove an Admin, at the top of the **Active Directory admin** page, select **Remove admin**, and then select **Save**.


## Configure your client computers
On all client machines, from which your applications or users connect to Azure SQL Database or Azure SQL Data Warehouse using Azure AD identities, you must install the following software:

* .NET Framework 4.6 or later from [https://msdn.microsoft.com/library/5a4x27ek.aspx](https://msdn.microsoft.com/library/5a4x27ek.aspx).
* Azure Active Directory Authentication Library for SQL Server (**ADALSQL.DLL**) is available in multiple languages (both x86 and amd64) from the download center at [Microsoft Active Directory Authentication Library for Microsoft SQL Server](http://www.microsoft.com/download/details.aspx?id=48742).

You can meet these requirements by:

* Installing either [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) or [SQL Server Data Tools for Visual Studio 2015](https://msdn.microsoft.com/library/mt204009.aspx) meets the .NET Framework 4.6 requirement.
* SSMS installs the x86 version of **ADALSQL.DLL**.
* SSDT installs the amd64 version of **ADALSQL.DLL**.
* The latest Visual Studio from [Visual Studio Downloads](https://www.visualstudio.com/downloads/download-visual-studio-vs) meets the .NET Framework 4.6 requirement, but does not install the required amd64 version of **ADALSQL.DLL**.

## Create contained database users in your database mapped to Azure AD identities

Azure Active Directory authentication requires database users to be created as contained database users. A contained database user based on an Azure AD identity, is a database user that does not have a login in the master database, and which maps to an identity in the Azure AD directory that is associated with the database. The Azure AD identity can be either an individual user account or a group. For more information about contained database users, see [Contained Database Users- Making Your Database Portable](https://msdn.microsoft.com/library/ff929188.aspx).

> [!NOTE]
> Database users (with the exception of administrators) cannot be created using the Azure portal. RBAC roles are not propagated to SQL Server, SQL Database, or SQL Data Warehouse. Azure RBAC roles are used for managing Azure Resources, and do not apply to database permissions. For example, the **SQL Server Contributor** role does not grant access to connect to the SQL Database or SQL Data Warehouse. The access permission must be granted directly in the database using Transact-SQL statements.
>

To create an Azure AD-based contained database user (other than the server administrator that owns the database), connect to the database with an Azure AD identity, as a user with at least the **ALTER ANY USER** permission. Then use the following Transact-SQL syntax:

```
CREATE USER <Azure_AD_principal_name> FROM EXTERNAL PROVIDER;
```

*Azure_AD_principal_name* can be the user principal name of an Azure AD user or the display name for an Azure AD group.

**Examples:**
To create a contained database user representing an Azure AD federated or managed domain user:
```
CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
CREATE USER [alice@fabrikam.onmicrosoft.com] FROM EXTERNAL PROVIDER;
```

To create a contained database user representing an Azure AD or federated domain group, provide the display name of a security group:
```
CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;
```

To create a contained database user representing an application that connects using an Azure AD token:

```
CREATE USER [appName] FROM EXTERNAL PROVIDER;
```

>  [!TIP]
>  You cannot directly create a user from an Azure Active Directory other than the Azure Active
Directory that is associated with your Azure subscription. However, members of other Active Directories that are imported users in the associated Active Directory (known as external users) can be added to an Active Directory group in the tenant Active Directory. By creating a contained database user for that AD group, the users from the external Active Directory can gain access to SQL Database.   

For more information about creating contained database users based on Azure Active Directory identities, see [CREATE USER (Transact-SQL)](http://msdn.microsoft.com/library/ms173463.aspx).


   
When you create a database user, that user receives the **CONNECT** permission and can connect to that database as a member of the **PUBLIC** role. Initially the only permissions available to the user are any permissions granted to the **PUBLIC** role, or any permissions granted to any Azure AD groups that they are a member of. Once you provision an Azure AD-based contained database user, you can grant the user additional permissions, the same way as you grant permission to any other type of user. Typically grant permissions to database roles, and add users to roles. For more information, see [Database Engine Permission Basics](http://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx). For more information about special SQL Database roles, see [Managing Databases and Logins in Azure SQL Database](sql-database-manage-logins.md).
A federated domain user account that is imported into a managed domain as an external user, must use the managed domain identity.

> [!NOTE]
> Azure AD users are marked in the database metadata with type E (EXTERNAL_USER) and for groups with type X (EXTERNAL_GROUPS). For more information, see [sys.database_principals](https://msdn.microsoft.com/library/ms187328.aspx). 
>

## Connect to the user database or data warehouse by using SSMS or SSDT  
To confirm the Azure AD administrator is properly set up, connect to the **master** database using the Azure AD administrator account.
To provision an Azure AD-based contained database user (other than the server administrator that owns the database), connect to the database with an Azure AD identity that has access to the database.

> [!IMPORTANT]
> Support for Azure Active Directory authentication is available with [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) and [SQL Server Data Tools](https://msdn.microsoft.com/library/mt204009.aspx) in Visual Studio 2015. The August 2016 release of SSMS also includes support for Active Directory Universal Authentication, which allows administrators to require Multi-Factor Authentication using a phone call, text message, smart cards with pin, or mobile app notification.
 
## Using an Azure AD identity to connect using SSMS or SSDT  

The following procedures show you how to connect to a SQL database with an Azure AD identity using SQL Server Management Studio or SQL Server Database Tools.

### Active Directory integrated authentication

Use this method if you are logged in to Windows using your Azure Active Directory credentials from a federated domain.

1. Start Management Studio or Data Tools and in the **Connect to Server** (or **Connect to Database Engine**) dialog box, in the **Authentication** box, select **Active Directory - Integrated**. No password is needed or can be entered because your existing credentials will be presented for the connection.   

    ![Select AD Integrated Authentication][11]
2. Select the **Options** button, and on the **Connection Properties** page, in the **Connect to database** box, type the name of the user database you want to connect to. (The **AD domain name or tenant ID**” option is only supported for **Universal with MFA connection** options, otherwise it is greyed out.)  

    ![Select the database name][13]

## Active Directory password authentication

Use this method when connecting with an Azure AD principal name using the Azure AD managed domain. You can also use it for federated accounts without access to the domain, for example when working remotely.

Use this method to authenticate to SQL DB/DW with Azure AD  for native of federated Azure AD users.
A native user is one explicitly created in Azure AD and being authenticated using user name and password, while a federated user is a Windows user whose domain is federated with Azure AD. The latter method (using user & password) can be used when a user wants to use his windows credential, but his local machine is not joined with the domain ( i.e. using a remote access). In this case a Windows user can indicate his domain account and password and can authenticate to SQL DB/DW using  federated credentials.

1. Start Management Studio or Data Tools and in the **Connect to Server** (or **Connect to Database Engine**) dialog box, in the **Authentication** box, select **Active Directory - Password**.
2. In the **User name** box, type your Azure Active Directory user name in the format **username@domain.com**. This must be an account from the Azure Active Directory or an account from a domain federate with the Azure Active Directory.
3. In the **Password** box, type your user password for the Azure Active Directory account or federated domain account.

    ![Select AD Password Authentication][12]
4. Select the **Options** button, and on the **Connection Properties** page, in the **Connect to database** box, type the name of the user database you want to connect to. (See the graphic in the previous option.)

## Using an Azure AD identity to connect from a client application

The following procedures show you how to connect to a SQL database with an Azure AD identity from a client application.

###  Active Directory integrated authentication

To use integrated Windows authentication, your domain’s Active Directory must be federated with Azure Active Directory. Your client application (or a service) connecting to the database must be running on a domain-joined machine under a user’s domain credentials.

To connect to a database using integrated authentication and an Azure AD identity, the Authentication keyword in the database connection string must be set to Active Directory Integrated. The following C# code sample uses ADO .NET.

```
string ConnectionString =
@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Integrated; Initial Catalog=testdb;";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

The connection string keyword ``Integrated Security=True`` is not supported for connecting to Azure SQL Database. When making an ODBC connection, you will need to remove spaces and set Authentication to 'ActiveDirectoryIntegrated'.

### Active Directory password authentication

To connect to a database using integrated authentication and an Azure AD identity, the Authentication keyword must be set to Active Directory Password. The connection string must contain User ID/UID and Password/PWD keywords and values. The following C# code sample uses ADO .NET.

```
string ConnectionString =
@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Password; Initial Catalog=testdb;  UID=bob@contoso.onmicrosoft.com; PWD=MyPassWord!";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

Learn more about Azure AD authentication methods using the demo code samples available at [Azure AD Authentication GitHub Demo](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth).

<!--Image references-->

[1]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/1aad-auth-diagram.png
[2]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/2subscription-relationship.png
[3]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/3admin-structure.png
[4]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/4select-subscription.png
[5]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/5ad-settings-portal.png
[6]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/6edit-directory-select.png
[7]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/7edit-directory-confirm.png
[8]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/8choose-ad.png
[10]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/10choose-admin.png
[11]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/active-directory-integrated.png
[12]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/12connect-using-pw-auth2.png
[13]: https://docs.microsoft.com/en-us/azure/sql-database/media/sql-database-aad-authentication/13connect-to-db2.png