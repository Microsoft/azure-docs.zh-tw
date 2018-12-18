## <a name="next-steps"></a>後續步驟

啟用 Azure 金鑰保存庫整合之後，您可以在您的 SQL VM 上啟用 SQL Server 加密。 首先，您必須在金鑰保存庫內建立非對稱金鑰，以及在 VM 上的 SQL Server 內建立對稱金鑰。 然後，您可以執行 T-SQL 陳述式來啟用您的資料庫和備份的加密。

有數種形式的加密可供您利用：

* [透明資料加密 (TDE)](https://msdn.microsoft.com/library/bb934049.aspx)
* [加密的備份](https://msdn.microsoft.com/library/dn449489.aspx)
* [資料行層級加密 (CLE)](https://msdn.microsoft.com/library/ms173744.aspx)

下列 Transact-SQL 指令碼為每個區域提供範例。

### <a name="prerequisites-for-examples"></a>範例的必要條件

每個範例是根據兩個必要條件：您的金鑰保存庫的非對稱金鑰 (稱為 **CONTOSO_KEY**)，和 AKV 整合功能所建立的認證 (稱為 **Azure_EKM_TDE_cred**)。 下列 Transact-SQL 命令會設定這些必要條件，以用於執行範例。

``` sql
USE master;
GO

--create credential
--The <<SECRET>> here requires the <Application ID> (without hyphens) and <Secret> to be passed together without a space between them.
CREATE CREDENTIAL sysadmin_ekm_cred
    WITH IDENTITY = 'keytestvault', --keyvault
    SECRET = '<<SECRET>>'
FOR CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM_Prov;


--Map the credential to a SQL login that has sysadmin permissions. This allows the SQL login to access the key vault when creating the asymmetric key in the next step.
ALTER LOGIN [SQL_Login]
ADD CREDENTIAL sysadmin_ekm_cred;


CREATE ASYMMETRIC KEY CONTOSO_KEY
FROM PROVIDER [AzureKeyVault_EKM_Prov]
WITH PROVIDER_KEY_NAME = 'KeyName_in_KeyVault',  --The key name here requires the key we created in the key vault
CREATION_DISPOSITION = OPEN_EXISTING;
```

### <a name="transparent-data-encryption-tde"></a>透明資料加密 (TDE)

1. 建立 SQL Server 登入，讓資料庫引擎用於 TDE，然後新增認證。

   ``` sql
   USE master;
   -- Create a SQL Server login associated with the asymmetric key
   -- for the Database engine to use when it loads a database
   -- encrypted by TDE.
   CREATE LOGIN EKM_Login
   FROM ASYMMETRIC KEY CONTOSO_KEY;
   GO

   -- Alter the TDE Login to add the credential for use by the
   -- Database Engine to access the key vault
   ALTER LOGIN EKM_Login
   ADD CREDENTIAL Azure_EKM_cred;
   GO
   ```

1. 建立將用於 TDE 的資料庫加密金鑰。

   ``` sql
   USE ContosoDatabase;
   GO

   CREATE DATABASE ENCRYPTION KEY 
   WITH ALGORITHM = AES_128 
   ENCRYPTION BY SERVER ASYMMETRIC KEY CONTOSO_KEY;
   GO

   -- Alter the database to enable transparent data encryption.
   ALTER DATABASE ContosoDatabase
   SET ENCRYPTION ON;
   GO
   ```

### <a name="encrypted-backups"></a>加密的備份

1. 建立 SQL Server 登入，讓資料庫引擎用於加密備份，然後新增認證。

   ``` sql
   USE master;
   -- Create a SQL Server login associated with the asymmetric key
   -- for the Database engine to use when it is encrypting the backup.
   CREATE LOGIN EKM_Login
   FROM ASYMMETRIC KEY CONTOSO_KEY;
   GO

   -- Alter the Encrypted Backup Login to add the credential for use by
   -- the Database Engine to access the key vault
   ALTER LOGIN EKM_Login
   ADD CREDENTIAL Azure_EKM_cred ;
   GO
   ```

1. 備份資料庫會指定加密，具有儲存在金鑰保存庫中的非對稱金鑰。

   ``` sql
   USE master;
   BACKUP DATABASE [DATABASE_TO_BACKUP]
   TO DISK = N'[PATH TO BACKUP FILE]'
   WITH FORMAT, INIT, SKIP, NOREWIND, NOUNLOAD,
   ENCRYPTION(ALGORITHM = AES_256, SERVER ASYMMETRIC KEY = [CONTOSO_KEY]);
   GO
   ```

### <a name="column-level-encryption-cle"></a>資料行層級加密 (CLE)

此指令碼會建立受到金鑰保存庫中非對稱金鑰保護的對稱金鑰，然後使用對稱金鑰來加密資料庫中的資料。

``` sql
CREATE SYMMETRIC KEY DATA_ENCRYPTION_KEY
WITH ALGORITHM=AES_256
ENCRYPTION BY ASYMMETRIC KEY CONTOSO_KEY;

DECLARE @DATA VARBINARY(MAX);

--Open the symmetric key for use in this session
OPEN SYMMETRIC KEY DATA_ENCRYPTION_KEY
DECRYPTION BY ASYMMETRIC KEY CONTOSO_KEY;

--Encrypt syntax
SELECT @DATA = ENCRYPTBYKEY(KEY_GUID('DATA_ENCRYPTION_KEY'), CONVERT(VARBINARY,'Plain text data to encrypt'));

-- Decrypt syntax
SELECT CONVERT(VARCHAR, DECRYPTBYKEY(@DATA));

--Close the symmetric key
CLOSE SYMMETRIC KEY DATA_ENCRYPTION_KEY;
```

## <a name="additional-resources"></a>其他資源

如需有關如何使用這些加密功能的詳細資訊，請參閱 [以 SQL Server 加密功能使用 EKM](https://msdn.microsoft.com/library/dn198405.aspx#UsesOfEKM)。

請注意，本文中的步驟假設您已在 Azure 虛擬機器上執行 SQL Server。 如果沒有，請參閱[在 Azure 中佈建 SQL Server 虛擬機器](../articles/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision.md)。 如需在 Azure VM 中執行 SQL Server 的其他指引，請參閱[Azure 虛擬機器上的 SQL Server 概觀](../articles/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview.md)。
