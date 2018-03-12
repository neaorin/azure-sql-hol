# Configure active geo-replication for Azure SQL Database

This article shows you how to configure active geo-replication for SQL Database in the [Azure portal](http://portal.azure.com) and to initiate failover.

To initiate failover with the Azure portal, see [Initiate a planned or unplanned failover for Azure SQL Database with the Azure portal](sql-database-geo-replication-portal.md).

To configure active geo-replication by using the Azure portal, you need the following resource:

* An Azure SQL database: The primary database that you want to replicate to a different geographical region.

> [!Note]
Active geo-replication must be between databases in the same subscription.

## Add a secondary database
The following steps create a new secondary database in a geo-replication partnership.  

To add a secondary database, you must be the subscription owner or co-owner.

The secondary database has the same name as the primary database and has, by default, the same service level. The secondary database can be a single database or a database in an elastic pool. For more information, see [Service tiers](sql-database-service-tiers.md).
After the secondary is created and seeded, data begins replicating from the primary database to the new secondary database.

> [!NOTE]
> If the partner database already exists (for example, as a result of terminating a previous geo-replication relationship) the command fails.
> 

1. In the [Azure portal](http://portal.azure.com), browse to the database that you want to set up for geo-replication.
2. On the SQL database page, select **geo-replication**, and then select the region to create the secondary database. You can select any region other than the region hosting the primary database, but we recommend the [paired region](../best-practices-availability-paired-regions.md). For this tutorial, select **South Central US** as the region for the secondary database.

   
    ![Configure geo-replication](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-geo-replication-portal/configure-geo-replication.png)

3. Select or configure the server and pricing tier for the secondary database. For this tutorial, use the name **sqlholgeo[XX]**.
   
    ![Configure secondary](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-geo-replication-portal/create-secondary.png)

5. Click **Create** to add the secondary.
6. The secondary database is created and the seeding process begins.
   
    ![Configure secondary](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-geo-replication-portal/seeding0.png)
7. When the seeding process is complete, the secondary database displays its status.
   
    ![Seeding complete](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-geo-replication-portal/seeding-complete.png)

8. You can optionally connect to the secondary database and verify that it responds to read-only queries.

For example, this query will complete successfully on the secondary replica:

```sql
select top 1 * from Person.Person
```

However, this query will fail as the database is read-only:

```sql
update Person.Person set MiddleName = 'K' where BusinessEntityID = 1
```

## Initiate a failover (optional)

The secondary database can be switched to become the primary.  

1. In the [Azure portal](http://portal.azure.com), browse to the primary database in the geo-replication partnership.
2. On the SQL Database blade, select **All settings** > **geo-replication**.
3. In the **SECONDARIES** list, select the database you want to become the new primary and click **Failover**.
   
    ![failover](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/sql-database-geo-replication-failover-portal/secondaries.png)
4. Click **Yes** to begin the failover.

The command immediately switches the secondary database into the primary role. 

There is a short period during which both databases are unavailable (on the order of 0 to 25 seconds) while the roles are switched. If the primary database has multiple secondary databases, the command automatically reconfigures the other secondaries to connect to the new primary. The entire operation should take less than a minute to complete under normal circumstances. 

> [!NOTE]
> This command is designed for quick recovery of the database in case of an outage. It triggers failover without data synchronization (forced failover).  If the primary is online and committing transactions when the command is issued some data loss may occur. 
> 
> 

