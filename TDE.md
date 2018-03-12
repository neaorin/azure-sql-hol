# Transparent data encryption for Azure SQL Database 

Transparent data encryption helps protect Azure SQL Database and Azure Data Warehouse against the threat of malicious activity. It performs real-time encryption and decryption of the database, associated backups, and transaction log files at rest without requiring changes to the application.

Transparent data encryption encrypts the storage of an entire database by using a symmetric key called the database encryption key. This database encryption key is protected by the transparent data encryption protector. The protector is either a service-managed certificate (service-managed transparent data encryption) or an asymmetric key stored in Azure Key Vault (Bring Your Own Key). You set the transparent data encryption protector at the server level. 

On database startup, the encrypted database encryption key is decrypted and then used for decryption and re-encryption of the database files in the SQL Server Database Engine process. Transparent data encryption performs real-time I/O encryption and decryption of the data at the page level. Each page is decrypted when it's read into memory and then encrypted before being written to disk. For a general description of transparent data encryption, see [Transparent data encryption](transparent-data-encryption.md).

SQL Server running on an Azure virtual machine also can use an asymmetric key from Key Vault. The configuration steps are different from using an asymmetric key in SQL Database. For more information, see [Extensible key management by using Azure Key Vault (SQL Server)](extensible-key-management-using-azure-key-vault-sql-server.md).

## Service-managed transparent data encryption

In Azure, the default setting for transparent data encryption is that the database encryption key is protected by a built-in server certificate. The built-in server certificate is unique for each server. If a database is in a geo-replication relationship, both the primary and geo-secondary database are protected by the primary database's parent server key. If two databases are connected to the same server, they share the same built-in certificate. Microsoft automatically rotates these certificates at least every 90 days.

Microsoft also seamlessly moves and manages the keys as needed for geo-replication and restores. 

> [!IMPORTANT]
> All newly created SQL databases are encrypted by default by using service-managed transparent data encryption. Existing databases before May 2017 and databases created through restore, geo-replication, and database copy aren't encrypted by default.
>

## Bring Your Own Key (preview)

With Bring Your Own Key (in preview) support, you can take control over your transparent data encryption keys and control who can access them and when. Key Vault, which is the Azure cloud-based external key management system, is the first key management service that transparent data encryption has integrated with Bring Your Own Key support. With Bring Your Own Key support, the database encryption key is protected by an asymmetric key stored in Key Vault. The asymmetric key never leaves Key Vault. After the server has permissions to a key vault, the server sends basic key operation requests to it through Key Vault. You set the asymmetric key at the server level, and all databases under that server inherit it.

With Bring Your Own Key support, you now can control key management tasks such as key rotations and key vault permissions. You also can delete keys and enable auditing/reporting on all encryption keys. Key Vault provides central key management and uses tightly monitored hardware security modules. Key Vault promotes separation of management of keys and data to help meet regulatory compliance. To learn more about Key Vault, see the [Key Vault documentation page](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault).

To learn more about transparent data encryption with Bring Your Own Key support for SQL Database and Data Warehouse, see [Transparent data encryption with Bring Your Own Key support](transparent-data-encryption-byok-azure-sql.md).

To start using transparent data encryption with Bring Your Own Key support, see the how-to guide [Turn on transparent data encryption by using your own key from Key Vault by using PowerShell](transparent-data-encryption-byok-azure-sql-configure.md).

## Manage transparent data encryption in the Azure portal

To configure transparent data encryption through the Azure portal, you must be connected as the Azure Owner, Contributor, or SQL Security Manager. 

You set transparent data encryption on the database level. To enable transparent data encryption on a database, go to the [Azure portal](https://portal.azure.com) and sign in with your Azure Administrator or Contributor account. Find the transparent data encryption settings under your user database. By default, service-managed transparent data encryption is used. A transparent data encryption certificate is automatically generated for the server that contains the database. 

![Service-managed transparent data encryption](https://raw.githubusercontent.com/MicrosoftDocs/sql-docs/live/docs/relational-databases/security/encryption/media/transparent-data-encryption-azure-sql/service-managed-tde.PNG)  

You set the transparent data encryption master key, also known as the transparent data encryption protector, on the server level. To use transparent data encryption with Bring Your Own Key support and protect your databases with a key from Key Vault, see the transparent data encryption settings under your server. 

![Transparent data encryption with Bring Your Own Key support](https://raw.githubusercontent.com/MicrosoftDocs/sql-docs/live/docs/relational-databases/security/encryption/media/transparent-data-encryption-azure-sql/tde-byok-support.png) 
