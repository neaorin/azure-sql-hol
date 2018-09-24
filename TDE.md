# Transparent data encryption for SQL Database and Data Warehouse

Transparent data encryption (TDE) helps protect Azure SQL Database and Azure Data Warehouse against the threat of malicious activity. It performs real-time encryption and decryption of the database, associated backups, and transaction log files at rest without requiring changes to the application. By default, TDE is enabled for all newly deployed Azure SQL databases. TDE cannot be used to encrypt the logical **master** database in SQL Database.  The **master** database contains objects that are needed to perform the TDE operations on the user databases.

TDE will need to be manually enabled for older databases or Azure SQL Data Warehouse.  

Transparent data encryption encrypts the storage of an entire database by using a symmetric key called the database encryption key. This database encryption key is protected by the transparent data encryption protector. The protector is either a service-managed certificate (service-managed transparent data encryption) or an asymmetric key stored in Azure Key Vault (Bring Your Own Key). You set the transparent data encryption protector at the server level. 

On database startup, the encrypted database encryption key is decrypted and then used for decryption and re-encryption of the database files in the SQL Server Database Engine process. Transparent data encryption performs real-time I/O encryption and decryption of the data at the page level. Each page is decrypted when it's read into memory and then encrypted before being written to disk. For a general description of transparent data encryption, see [Transparent data encryption](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption).

SQL Server running on an Azure virtual machine also can use an asymmetric key from Key Vault. The configuration steps are different from using an asymmetric key in SQL Database. For more information, see [Extensible key management by using Azure Key Vault (SQL Server)](https://docs.microsoft.com/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server).

## Service-managed transparent data encryption

In Azure, the default setting for transparent data encryption is that the database encryption key is protected by a built-in server certificate. The built-in server certificate is unique for each server. If a database is in a geo-replication relationship, both the primary and geo-secondary database are protected by the primary database's parent server key. If two databases are connected to the same server, they share the same built-in certificate. Microsoft automatically rotates these certificates at least every 90 days.

Microsoft also seamlessly moves and manages the keys as needed for geo-replication and restores. 

> [!IMPORTANT]
> All newly created SQL databases are encrypted by default by using service-managed transparent data encryption. Existing databases before May 2017 and databases created through restore, geo-replication, and database copy aren't encrypted by default.
>

To start using transparent data encryption with Bring Your Own Key support, see the how-to guide [Turn on transparent data encryption by using your own key from Key Vault by using PowerShell](https://docs.microsoft.com/en-us/azure/sql-database/transparent-data-encryption-byok-azure-sql-configure).


## Manage transparent data encryption in the Azure portal

To configure transparent data encryption through the Azure portal, you must be connected as the Azure Owner, Contributor, or SQL Security Manager. 

You set transparent data encryption on the database level. To enable transparent data encryption on a database, go to the [Azure portal](https://portal.azure.com) and sign in with your Azure Administrator or Contributor account. Find the transparent data encryption settings under your user database. By default, service-managed transparent data encryption is used. A transparent data encryption certificate is automatically generated for the server that contains the database. 

![Service-managed transparent data encryption](https://docs.microsoft.com/en-us/azure/sql-database/media/transparent-data-encryption-azure-sql/service-managed-tde.png)  

## Transparent Data Encryption with Bring Your Own Key (BYOK) support 

Bring Your Own Key (BYOK) support for [Transparent Data Encryption (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption) allows you to encrypt the Database Encryption Key (DEK) with an asymmetric key called TDE Protector.  The TDE Protector is stored under your control in [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault), Azure’s cloud-based external key management system. Azure Key Vault is the first key management service with which TDE has integrated support for BYOK. The TDE DEK, which is stored on the boot page of a database is encrypted and decrypted by the TDE protector. The TDE Protector is stored in Azure Key Vault and never leaves the key vault. If the server's access to the key vault is revoked, a database cannot be decrypted and read into memory.  The TDE protector is set at the logical server level and is inherited by all databases associated with that server. 

With BYOK support, users can now control key management tasks including key rotations, key vault permissions, deleting keys, and enable auditing/reporting on all TDE protectors using Azure Key Vault functionality. Key Vault provides central key management, leverages tightly monitored hardware security modules (HSMs), and enables separation of duties between management of keys and data to help meet regulatory compliance.  


TDE with BYOK provides the following benefits:
- Increased transparency and granular control with the ability to self-manage the TDE protector   
- Central management of TDE protectors (along with other keys and secrets used in other Azure services) by hosting them in Key Vault
- Separation of key and data management responsibilities within the organization, to support separation of duties
- Greater trust from your own clients, since Key Vault is designed so that Microsoft does not see or extract any encryption keys. 
- Support for key rotation

> [!IMPORTANT]
> For those using service-managed TDE who would like to start using Key Vault, TDE remains enabled during the process of switching over to a TDE protector in Key Vault. There is no downtime nor re-encryption of the database files. Switching from a service-managed key to a Key Vault key only requires re-encryption of the database encryption key (DEK), which is a fast and online operation. 
>

## How does TDE with BYOK support work?
 
![Authentication of the Server to the Key Vault](./media/transparent-data-encryption-byok-azure-sql/tde-byok-server-authentication-flow.PNG)

When TDE is first configured to use a TDE protector from Key Vault, the server sends the DEK of each TDE-enabled database to Key Vault for a wrap key request. Key Vault returns the encrypted database encryption key, which is stored in the user database.  

>[!IMPORTANT]
>It is important to note that **once a TDE Protector is stored in Azure Key Vault, it never leaves Azure Key Vault**. The logical server can only send key operation requests to the TDE protector key material within Key Vault, and **never accesses or caches the TDE protector**. The Key Vault administrator has the right to revoke Key Vault permissions of the server at any point, in which case all connections to the server are cut off. 
>

You set the transparent data encryption master key, also known as the transparent data encryption protector, on the server level. To use transparent data encryption with Bring Your Own Key support and protect your databases with a key from Key Vault, see the transparent data encryption settings under your server. 

![Transparent data encryption with Bring Your Own Key support](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/sql-database/media/transparent-data-encryption-azure-sql/tde-byok-support.png) 