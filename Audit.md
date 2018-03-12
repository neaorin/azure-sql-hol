# Audit an Azure SQL database
Azure SQL database auditing tracks database events and writes them to an audit log in your Azure storage account. Auditing also:

* Helps you maintain regulatory compliance, understand database activity, and gain insight into discrepancies and anomalies that could indicate business concerns or suspected security violations.

* Enables and facilitates adherence to compliance standards, although it doesn't guarantee compliance. For more information about Azure programs that support standards compliance, see the [Azure Trust Center](https://azure.microsoft.com/support/trust-center/compliance/).


## <a id="subheading-1"></a>Azure SQL database auditing overview
You can use SQL database auditing to:


* **Retain** an audit trail of selected events. You can define categories of database actions to be audited.
* **Report** on database activity. You can use preconfigured reports and a dashboard to get started quickly with activity and event reporting.
* **Analyze** reports. You can find suspicious events, unusual activity, and trends.

You can configure auditing for different types of event categories, as explained in the [Set up auditing for your database](#subheading-2) section.

> [!IMPORTANT]
> Audit logs are written to **Append Blobs** in an Azure Blob storage on your Azure subscription. 

## <a id="subheading-8"></a>Define server-level vs. database-level auditing policy

An auditing policy can be defined for a specific database or as a default server policy:

* A server policy applies to all existing and newly created databases on the server.

* If *server blob auditing is enabled*, it *always applies to the database*. The database will be audited, regardless of the database auditing settings.

* Enabling blob auditing on the database, in addition to enabling it on the server, does *not* override or change any of the settings of the server blob auditing. Both audits will exist side by side. In other words, the database is audited twice in parallel; once by the server policy and once by the database policy.

   > [!NOTE]
   > You should avoid enabling both server blob auditing and database blob auditing together, unless:
    > * You want to use a different *storage account* or *retention period* for a specific database.
    > * You want to audit event types or categories for a specific database that differ from the rest of the databases on the server. For example, you might have table inserts that need to be audited only for a specific database.
   >
   > Otherwise, we recommended that you enable only server-level blob auditing and leave the database-level auditing disabled for all databases.


## <a id="subheading-2"></a>Set up auditing for your database
The following section describes the configuration of auditing using the Azure portal.

1. Go to the [Azure portal](https://portal.azure.com).
2. Go to the **Settings** blade of the SQL database/SQL server you want to audit. In the **Settings** blade, select **Auditing & Threat detection**.

    <a id="auditing-screenshot"></a>
    ![Navigation pane][1]
3. If you prefer to set up a server auditing policy, you can select the **View server settings** link in the database auditing blade. You can then view or modify the server auditing settings. Server auditing policies  apply to all existing and newly created databases on this server.

    ![Navigation pane][2]
4. If you prefer to enable blob auditing on the database level, for **Auditing**, select **ON**, and for **Auditing type**, select  **Blob**.

    If server blob auditing is enabled, the database-configured audit will exist side by side with the server blob audit.

    ![Navigation pane][3]
5. To open the **Audit Logs Storage** blade, select **Storage Details**. Select the Azure storage account where logs will be saved, and then select the retention period. The old logs will be deleted. Then click **OK**.
   >[!TIP]
   >To get the most out of the auditing reports templates, use the same storage account for all audited databases.

    <a id="storage-screenshot"></a>
    ![Navigation pane][4]
6. If you want to customize the audited events, you can do this via PowerShell or the REST API.
7. After you've configured your auditing settings, you can turn on the new threat detection feature and configure emails to receive security alerts. When you use threat detection, you receive proactive alerts on anomalous database activities that can indicate potential security threats. For more information, see [Getting started with threat detection](sql-database-threat-detection-get-started.md).
8. Click **Save**.





## <a id="subheading-3"></a>Analyze audit logs and reports
Audit logs are aggregated in the Azure storage account you chose during setup. You can explore audit logs by using a tool such as [Azure Storage Explorer](http://storageexplorer.com/).

Blob auditing logs are saved as a collection of blob files within a container named **sqldbauditlogs**.

For further details about the hierarchy of the storage folder, naming conventions, and log format, see the [Blob Audit Log Format Reference](https://go.microsoft.com/fwlink/?linkid=829599).

There are several methods you can use to view blob auditing logs:

* Use the [Azure portal](https://portal.azure.com).  Open the relevant database. At the top of the database's **Auditing & Threat detection** blade, click **View audit logs**.

    ![Navigation pane][7]

    An **Audit records** blade opens, from which you'll be able to view the logs.

    - You can view specific dates by clicking **Filter** at the top of the **Audit records** blade.
    - You can switch between audit records that were created by a server policy or database policy audit.

       ![Navigation pane][8]

* Use the system function **sys.fn_get_audit_file** (T-SQL) to return the audit log data in tabular format. For more information on using this function, see the [sys.fn_get_audit_file documentation](https://docs.microsoft.com/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql). Below is a sample query which is reading the data from Azure Blob Storage:

```sql
SELECT TOP 20 * FROM sys.fn_get_audit_file ('https://demoazuresqlaudit.blob.core.windows.net/sqldbauditlogs/sqlhol-sorin/AdventureWorks/SqlDbAuditing_Audit/2018-03-12/16_52_07_372_0.xel',default,default);
```


* Use **Merge Audit Files** in SQL Server Management Studio (starting with SSMS 17):
    1. From the SSMS menu, select **File** > **Open** > **Merge Audit Files**.

        ![Navigation pane][9]
    2. The **Add Audit Files** dialog box opens. Select one of the **Add** options to
     choose whether to merge audit files from a local disk or import them from Azure Storage. You are required to provide your Azure Storage details and account key.

    3. After all files to merge have been added, click **OK** to complete the merge operation.

    4. The merged file opens in SSMS, where you can view and analyze it, as well as export it to an XEL or CSV file or to a table.

<!--Image references-->
[1]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/1_auditing_get_started_settings.png
[2]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/2_auditing_get_started_server_inherit.png
[3]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/3_auditing_get_started_turn_on.png
[4]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/4_auditing_get_started_storage_details.png
[5]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/5_auditing_get_started_storage_key_regeneration.png
[6]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/6_auditing_get_started_regenerate_key.png
[7]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/7_auditing_get_started_blob_view_audit_logs.png
[8]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/8_auditing_get_started_blob_audit_records.png
[9]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/9_auditing_get_started_ssms_1.png
[10]: https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-auditing-get-started/10_auditing_get_started_ssms_2.png