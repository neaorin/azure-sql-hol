# Back up and restore an Azure SQL Database

SQL Database provides these options for database recovery using [automated database backups](sql-database-automated-backups.md) and [backups in long-term retention](sql-database-long-term-retention.md). You can restore from a database backup to:

* A new database on the same logical server recovered to a specified point in time within the retention period. 
* A database on the same logical server recovered to the deletion time for a deleted database.
* A new database on any logical server in any region recovered to the point of the most recent daily backups in geo-replicated blob storage (RA-GRS).

> [!IMPORTANT]
> You cannot overwrite an existing database during restore.
>

## Point-in-time restore using automated database backups

Point-in-Time Restore enables you to roll back an Azure SQL Database to a previous point in time (up to 35 days).

You can restore an existing database to an earlier point in time as a new database on the same logical server using the Azure portal, [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql/restore-azurermsqldatabase), or the [REST API](https://msdn.microsoft.com/library/azure/mt163685.aspx). 

The database can be restored to any service tier or performance level, and as a single database or into an elastic pool. Ensure you have sufficient resources on the logical server or in the elastic pool to which you are restoring the database. Once complete, the restored database is a normal, fully accessible, online database. The restored database is charged at normal rates based on its service tier and performance level. You do not incur charges until the database restore is complete.

You generally restore a database to an earlier point for recovery purposes. When doing so, you can treat the restored database as a replacement for the original database or use it to retrieve data from and then update the original database. 

* ***Database replacement:*** If the restored database is intended as a replacement for the original database, you should verify the performance level and/or service tier are appropriate and scale the database if necessary. You can rename the original database and then give the restored database the original name using the [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) command in T-SQL. 
* ***Data recovery:*** If you plan to retrieve data from the restored database to recover from a user or application error, you need to write and execute the necessary data recovery scripts to extract data from the restored database to the original database. Although the restore operation may take a long time to complete, the restoring database is visible in the database list throughout the restore process. If you delete the database during the restore, the restore operation is canceled and you are not charged for the database that did not complete the restore. 

### Azure portal

To recover to a point in time using the Azure portal, open the page for your database and click **Restore** on the toolbar.

![point-in-time-restore](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-recovery-using-backups/point-in-time-recovery.png)


# Configure and restore from Azure SQL Database long-term backup retention

You can configure the Azure Recovery Services vault to store Azure SQL database backups and then recover a database using backups retained in the vault using the Azure portal or PowerShell.

## Azure portal

The following sections show you how to use the Azure portal to configure the Azure Recovery Services vault, view backups in the vault, and restore from the vault.

### Configure the vault, register the server, and select databases

You [configure an Azure Recovery Services vault to retain automated backups](sql-database-long-term-retention.md) for a period longer than the retention period for your service tier. 

1. Open the **SQL Server** page for your server.

   ![sql server page](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/sql-server-blade.png)

2. Click **Long-term backup retention**.

   ![long-term backup retention link](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/long-term-backup-retention-link.png)

3. On the **Long-term backup retention** page for your server, review and accept the preview terms (unless you have already done so - or this feature is no longer in preview).

   ![accept the preview terms](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/accept-the-preview-terms.png)

4. To configure long-term backup retention, select that database in the grid and then click **Configure** on the toolbar.

   ![select database for long-term backup retention](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/select-database-for-long-term-backup-retention.png)

5. On the **Configure** page, click **Configure required settings** under **Recovery service vault**.

   ![configure vault link](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/configure-vault-link.png)

6. On the **Recovery services vault** page, select an existing vault, if any. Otherwise, if no recovery services vault found for your subscription, click to exit the flow and create a recovery services vault.

   ![create vault link](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/create-new-vault-link.png)

7. On the **Recovery Services vaults** page, click **Add**.

   ![add vault link](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/add-new-vault-link.png)
   
8. On the **Recovery Services vault** page, provide a valid name for the Recovery Services vault.

   ![new vault name](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/new-vault-name.png)

9. Select your subscription and resource group, and then select the location for the vault. When done, click **Create**.

   ![create vault](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/create-new-vault.png)

   > [!IMPORTANT]
   > The vault must be located in the same region as the Azure SQL logical server, and must use the same resource group as the logical server.
   >

10. After the new vault is created, execute the necessary steps to return to the **Recovery services vault** page.

11. On the **Recovery services vault** page, click the vault and then click **Select**.

   ![select existing vault](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/select-existing-vault.png)

12. On the **Configure** page, provide a valid name for the new retention policy, modify the default retention policy as appropriate, and then click **OK**.

   ![define retention policy](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/define-retention-policy.png)
   
   >[!NOTE]
   >Retention policy names don't allow some characters, including spaces.

13. On the **Long-term backup retention** page for your database, click **Save** and then click **OK** to apply the long-term backup retention policy to all selected databases.

   ![define retention policy](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/save-retention-policy.png)

14. Click **Save** to enable long-term backup retention using this new policy to the Azure Recovery Services vault that you configured.

   ![define retention policy](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/enable-long-term-retention.png)

> [!IMPORTANT]
> Once configured, backups show up in the vault within next seven days. Do not continue this tutorial until backups show up in the vault.
>

### View backups in long-term retention using Azure portal

View information about your database backups in [long-term backup retention](sql-database-long-term-retention.md). 

1. In the Azure portal, open your Azure Recovery Services vault for your database backups (go to **All resources** and select it from the list of resources for your subscription) to view the amount of storage used by your database backups in the vault.

   ![view recovery services vault with backups](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/view-recovery-services-vault-with-data.png)

2. Open the **SQL database** page for your database.

   ![new sample db page](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-portal/new-sample-db-blade.png)

3. On the toolbar, click **Restore**.

   ![restore toolbar](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/restore-toolbar.png)

4. On the Restore page, click **Long-term**.

5. Under Azure vault backups, click **Select a backup** to view the available database backups in long-term backup retention.

   ![backups in vault](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/view-backups-in-vault.png)

### Restore a database from a backup in long-term backup retention using the Azure portal

You restore the database to a new database from a backup in the Azure Recovery Services vault.

1. On the **Azure vault backups** page, click the backup to restore and then click **Select**.

   ![select backup in vault](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/select-backup-in-vault.png)

2. In the **Database name** text box, provide the name for the restored database.

   ![new database name](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/new-database-name.png)

3. Click **OK** to restore your database from the backup in the vault to the new database.

4. On the toolbar, click the notification icon to view the status of the restore job.

   ![restore job progress from vault](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/restore-job-progress-long-term.png)

5. When the restore job is completed, open the **SQL databases** page to view the newly restored database.

   ![restored database from vault](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-get-started-backup-recovery/restored-database-from-vault.png)

