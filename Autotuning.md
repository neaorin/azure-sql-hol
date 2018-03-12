# Automatic tuning

Azure SQL Database is an automatically managed data service that constantly monitors your queries and identifies the action that you can perform to improve performance of your workload. You can review recommendations and manually apply them, or let Azure SQL Database automatically apply corrective actions - this is known as **automatic tuning mode**. Automatic tuning can be enabled at the server or the database level.

## Enable automatic tuning on server
On the server level you can choose to inherit automatic tuning configuration from "Azure Defaults" or not to inherit the configuration. Azure Defaults are FORCE_LAST_GOOD_PLAN enabled, CREATE_INDEX enabled, and DROP_INDEX disabled.

### Azure portal
To enable automatic tuning on Azure SQL Database server, navigate to the server in Azure portal and then select **Automatic tuning** in the menu. Select the automatic tuning options you want to enable and select **Apply**:

![Server](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-automatic-tuning-enable/server.png)

Automatic tuning options on server are applied to all databases on the server. By default, all databases inherit the configuration from their parent server, but this can be overridden and specified for each database individually.

## Enable automatic tuning on an individual database

The Azure SQL Database enables you to individually specify the automatic tuning configuration on each database. On the database level you can choose to inherit automatic tuning configuration from parent server, "Azure Defaults" or not to inherit the configuration. Azure Defaults are FORCE_LAST_GOOD_PLAN enabled, CREATE_INDEX enabled, and DROP_INDEX disabled.

> [!NOTE]
> The general recommendation is to manage the automatic tuning configuration at server level so the same configuration settings can be applied on every database automatically. Configure automatic tuning on an individual database if the database is different that others on the same server.
>

### Azure portal

To enable automatic tuning on a single database, navigate to the database in the Azure portal and then and select **Automatic tuning**. You can configure a single database to inherit the settings from the server by selecting the option or you can specify the configuration for a database individually.

![Database](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-automatic-tuning-enable/database.png)

Once you have selected appropriate configuration, click **Apply**.

Once Automatic tuning is enabled, you can view tuning operations in the [Recommendations](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-advisor-portal) section.

