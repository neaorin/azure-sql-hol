# Create an Azure SQL database

## Log in to the Azure portal

Log in to the [Azure portal](https://portal.azure.com/).

## Create a SQL database

An Azure SQL database is created with a defined set of [compute and storage resources](sql-database-service-tiers.md). The database is created within an [Azure resource group](../azure-resource-manager/resource-group-overview.md) and in an [Azure SQL Database logical server](sql-database-features.md).

Follow these steps to create an empty SQL database.

1. Click **Create a resource** in the upper left-hand corner of the Azure portal.

2. Select **Databases** from the **New** page, and select **Create** under **SQL Database** on the **New** page.

   ![create database-1](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/create-database-1.png)

3. Fill out the SQL Database form with the following information, as shown on the preceding image:   

   | Setting       | Suggested value | Description |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Database name** | **AdventureWorks[XX]** | For valid database names, see [Database Identifiers](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers). |
   | **Subscription** | Your subscription  | For details about your subscriptions, see [Subscriptions](https://account.windowsazure.com/Subscriptions). |
   | **Resource group**  | **sqlhol[XX]** | For valid resource group names, see [Naming rules and restrictions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions). |
   | **Select source** | Blank database | Creates an empty database. |

   > [!IMPORTANT]
   > You must select the Blank database on this form. You will migrate a database from SQL Server in the next steps.
   >

4. Under **Server**, click **Configure required settings** and fill out the SQL server (logical server) form with the following information, as shown on the following image:   

   | Setting       | Suggested value | Description |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Server name** | **sqlhol[XX]** | For valid server names, see [Naming rules and restrictions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions). |
   | **Server admin login** | Any valid name | For valid login names, see [Database Identifiers](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers). |
   | **Password** | Any valid password | Your password must have at least 8 characters and must contain characters from three of the following categories: upper case characters, lower case characters, numbers, and non-alphanumeric characters. |
   | **Subscription** | Your subscription | For details about your subscriptions, see [Subscriptions](https://account.windowsazure.com/Subscriptions). |
   | **Resource group** | **sqlhol[XX]** | For valid resource group names, see [Naming rules and restrictions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions). |
   | **Location** | **East US** | For information about regions, see [Azure Regions](https://azure.microsoft.com/regions/). |

   > [!IMPORTANT]
   > The server admin login and password that you specify here are required to log in to the server and its databases later in this quick start. Remember or record this information for later use.
   >  

   ![create database-server](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/create-database-server.png)

5. When you have completed the form, click **Select**.

6. Click **Pricing tier** to specify the service tier, the number of DTUs, and the amount of storage. Explore the options for the amount of DTUs and storage that is available to you for each service tier.

7. For this quick start tutorial, select the **Standard** service tier and then use the slider to select **10 DTUs (S0)** and **1** GB of storage.

   ![create database-s1](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/create-database-s1.png)

8. Accept the preview terms to use the **Add-on Storage** option.

9. After selecting the server tier, the number of DTUs, and the amount of storage, click **Apply**.  

10. Now that you have completed the SQL Database form, click **Create** to provision the database. Provisioning takes a few minutes.

11. On the toolbar, click **Notifications** to monitor the deployment process.

     ![notification](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/notification.png)

## Create a server-level firewall rule

> [!NOTE]
> You do NOT need to create a firewall rule if you are using SQL Server Management Studio from within an Azure Virtual Machine to connect to the database, and you left the *Allow Azure services to access server* option selected during the database create screen.

The SQL Database service creates a firewall at the server-level that prevents external applications and tools from connecting to the server or any databases on the server unless a firewall rule is created to open the firewall for specific IP addresses. Follow these steps to create a [SQL Database server-level firewall rule](sql-database-firewall-configure.md) for your client's IP address and enable external connectivity through the SQL Database firewall for your IP address only.

> [!NOTE]
> SQL Database communicates over port 1433. If you are trying to connect from within a corporate network, outbound traffic over port 1433 may not be allowed by your network's firewall. If so, you cannot connect to your Azure SQL Database server unless your IT department opens port 1433.
>

1. After the deployment completes, click **SQL databases** from the left-hand menu and then click **mySampleDatabase** on the **SQL databases** page. The overview page for your database opens, showing you the fully qualified server name (such as **mynewserver-20170824.database.windows.net**) and provides options for further configuration.

2. Copy this fully qualified server name for use to connect to your server and its databases in subsequent quick starts.

   ![server name](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/server-name.png)

3. Click **Set server firewall** on the toolbar as shown in the previous image. The **Firewall settings** page for the SQL Database server opens.

   ![server firewall rule](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/server-firewall-rule.png)

4. Click **Add client IP** on the toolbar to add your current IP address to a new firewall rule. A firewall rule can open port 1433 for a single IP address or a range of IP addresses.

5. Click **Save**. A server-level firewall rule is created for your current IP address opening port 1433 on the logical server.

6. Click **OK** and then close the **Firewall settings** page.

You can now connect to the SQL Database server and its databases using SQL Server Management Studio or another tool of your choice from this IP address using the server admin account created previously.

## Query the SQL database

Now that you have created a sample database in Azure, let’s use the built-in query tool within the Azure portal to confirm that you can connect to the database and query the data.

1. On the SQL Database page for your database, click **Query editor (preview)** in the left-hand menu and then click **Login**.

   ![login](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/query-editor-login.png)

2. Select SQL server authentication, provide the required login information, and then click **OK** to log in.

3. After you are authenticated as admin, type the following query in the query editor pane.

```sql
SELECT @@VERSION
```

4. Click **Run** and then review the query results in the **Results** pane.

## Create another user with limited rights

Some sections of this workshop require the existence of a user which is not database owner, but will have data reading rights. Let's create this user right now.

1. Connect to the **AdventureWorks** database, using either the built-in portal query tool as above, or by using **SQL Server Management Studio**. 

2. If you used **SQL Server Management Studio**, after you are authenticated as admin, switch to the **AdventureWorks** database.

3. Type the following query in the query editor pane.

```sql
CREATE USER Mary WITH PASSWORD = 'SomePassword234%^&';
ALTER ROLE db_datareader ADD MEMBER Mary; 
```

3. Make sure the query has successfully completed.