---
title: 部署分割合併服務 | Microsoft Docs
description: 使用分割合併工具以在分區化資料庫之間移動資料。
services: sql-database
ms.service: sql-database
subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: e277e2fa5ca7062cde1c0061e585dfb092337d4a
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47159331"
---
# <a name="deploy-a-split-merge-service"></a>部署分割合併服務
分割合併工具可讓您在分區化資料庫之間移動資料。 請參閱 [在相應放大的雲端資料庫之間移動資料](sql-database-elastic-scale-overview-split-and-merge.md)

## <a name="download-the-split-merge-packages"></a>下載分割合併套件
1. 從 [NuGet](http://docs.nuget.org/docs/start-here/installing-nuget)下載最新的 NuGet 版本。
2. 開啟命令提示字元並瀏覽至您下載 nuget.exe 的目錄。 下載包含 PowerShell 命令。
3. 使用下列命令，將最新的分割合併封裝下載到目前的目錄：
   ```
   nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge
   ```  

檔案會放在名為 **Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge.x.x.xxx.x** 的目錄中，其中 *x.x.xxx.x* 反映版本號碼。 在 **content\splitmerge\service** 子目錄中找出分割合併服務檔案，在 **content\splitmerge\powershell** 子目錄中找出分割合併 PowerShell 指令碼 (和必要的用戶端 dll 檔)。

## <a name="prerequisites"></a>必要條件
1. 建立用來作為分割合併狀態資料庫的 Azure SQL DB 資料庫。 移至 [Azure 入口網站](https://portal.azure.com)。 建立新的 **SQL Database**。 提供資料庫名稱，並建立新的系統管理員和密碼。 請務必記錄名稱和密碼，以供稍後使用。
2. 請確定您的 Azure SQL DB 伺服器允許 Azure 服務進行連接。 在入口網站的 [防火牆設定] 中，確定 [允許存取 Azure 服務] 設定設為 [開啟]。 按一下儲存圖示。
3. 建立適用於診斷輸出的 Azure 儲存體帳戶。
4. 建立適用於分割合併服務的 Azure 雲端服務。

## <a name="configure-your-split-merge-service"></a>設定分割合併服務
### <a name="split-merge-service-configuration"></a>分割合併服務組態
1. 在您下載分割合併組件的資料夾中，建立 **ServiceConfiguration.Template.cscfg** 檔案 (隨 **SplitMergeService.cspkg** 一起提供) 的複本，然後重新命名為 **ServiceConfiguration.cscfg**。
2. 在文字編輯器中開啟 **ServiceConfiguration.cscfg** ，例如：會驗證憑證指紋格式等輸入的 Visual Studio。
3. 建立新的資料庫或選擇現有的資料庫，做為分割合併作業的狀態資料庫，並擷取該資料庫的連接字串。 
   
   > [!IMPORTANT]
   > 目前，狀態資料庫必須使用拉丁文定序 (SQL\_Latin1\_General\_CP1\_CI\_AS)。 如需詳細資訊，請參閱 [Windows 定序名稱 (Transact-SQL)](https://msdn.microsoft.com/library/ms188046.aspx)。
   >

   在 Azure SQL DB 中，連接字串的格式通常為：
      ```
      Server=myservername.database.windows.net; Database=mydatabasename;User ID=myuserID; Password=mypassword; Encrypt=True; Connection Timeout=30
      ```

4. 在 cscfg 檔案中，在 ElasticScaleMetadata 設定的 **SplitMergeWeb** 和 **SplitMergeWorker** 角色區段中，輸入此連接字串。
5. 針對 **SplitMergeWorker** 角色，在 **WorkerRoleSynchronizationStorageAccountConnectionString** 設定中輸入有效的連接字串以連接至 Azure 儲存體。

### <a name="configure-security"></a>設定安全性
如需有關設定服務安全性的詳細指示，請參閱 [分割合併安全性設定](sql-database-elastic-scale-split-merge-security-configuration.md)。

為了執行簡單的測試部署以完成本教學課程，將會執行一組最基本的設定步驟，讓服務啟動並執行。 這些步驟只會啟用一個電腦/帳戶來與服務通訊。

### <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證
使用 [Visual Studio 開發人員命令提示字元](http://msdn.microsoft.com/library/ms229859.aspx) 視窗建立新的目錄，並從這個目錄中執行下列命令：

   ```
    makecert ^
    -n "CN=*.cloudapp.net" ^
    -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1,1.3.6.1.5.5.7.3.2" ^
    -a sha1 -len 2048 ^
    -sr currentuser -ss root ^
    -sv MyCert.pvk MyCert.cer
   ```

將會要求您輸入密碼來保護私密金鑰。 輸入強式密碼並加以確認。 接著會提示您輸入之後要再次使用的密碼。 最後，按一下 [是]  ，將它匯入受信任的憑證授權單位根目錄存放區。

### <a name="create-a-pfx-file"></a>建立 PFX 檔案
從執行 makecert 所在的相同視窗執行下列命令；使用您用來建立憑證的相同密碼：

    pvk2pfx -pvk MyCert.pvk -spc MyCert.cer -pfx MyCert.pfx -pi <password>

### <a name="import-the-client-certificate-into-the-personal-store"></a>將用戶端憑證匯入個人存放區
1. 在 Windows 檔案總管中，按兩下 [MyCert.pfx] 。
2. 在 [憑證匯入精靈] 中，選取 [目前使用者]，然後按 [下一步]。
3. 確認檔案路徑，然後按 [下一步] 。
4. 輸入密碼，保持核取 [包含所有延伸內容]，然後按 [下一步]。
5. 保持核取 [自動選取憑證存放區…]，然後按 [下一步]。
6. 按一下 [完成] 和 [確定]。

### <a name="upload-the-pfx-file-to-the-cloud-service"></a>將 PFX 檔案上傳至雲端服務
1. 移至 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [雲端服務] 。
3. 選取您先前為分割/合併服務建立的雲端服務。
4. 按一下頂端功能表的 [憑證]  。
5. 按一下底列的 [上傳]  。
6. 選取 PFX 檔案，並輸入與上述相同的密碼。
7. 完成後，從清單中的新項目複製憑證指紋。

### <a name="update-the-service-configuration-file"></a>更新服務組態檔
將上面複製的憑證指紋，貼到這些設定的憑證指紋/值屬性中。
背景工作角色：
   ```
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
   ```

Web 角色：

   ```
    <Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />
    <Setting name="AllowedClientCertificateThumbprints" value="" />
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
   ```

請注意，在實際執行部署中，CA、加密、伺服器憑證和用戶端憑證應該使用個別憑證。 如需詳細指示，請參閱 [安全性設定](sql-database-elastic-scale-split-merge-security-configuration.md)。

## <a name="deploy-your-service"></a>部署您的服務
1. 移至 [Azure 入口網站](https://portal.azure.com)
2. 選取您稍早建立的雲端服務。
3. 按一下 [概觀] 。
4. 選擇預備環境，然後按一下 [上傳]。
5. 在對話方塊中，輸入部署的標籤。 在 [封裝] 和 [設定] 中，按一下 [從本機] 並選擇 **SplitMergeService.cspkg** 檔案和您稍早設定的 cscfg 檔案。
6. 確定已核取 [即使一或多個角色包含單一執行個體也請部署]  核取方塊。
7. 點按右下方的勾號按鈕，開始進行部署。 預期會需要幾分鐘才能完成。


## <a name="troubleshoot-the-deployment"></a>疑難排解部署
如果 Web 角色無法上線，安全性設定可能有問題。 請檢查 SSL 的設定如上所述。

如果背景工作角色無法上線，但 Web 角色成功上線，很可能是無法連接至您稍早建立的狀態資料庫。

* 請確定您的 cscfg 中的連接字串正確無誤。
* 請檢查伺服器和資料庫均存在，而且使用者識別碼與密碼皆正確無誤。
* 在 Azure SQL DB 中，連接字串的格式應該為：

   ```  
   Server=myservername.database.windows.net; Database=mydatabasename;User ID=myuserID; Password=mypassword; Encrypt=True; Connection Timeout=30
   ```

* 確定伺服器名稱不是以 **https://** 開頭。
* 請確定您的 Azure SQL DB 伺服器允許 Azure 服務進行連接。 若要這樣做，請在入口網站中開啟資料庫，並確定 [允許存取 Azure 服務] 設定已設為 [開啟]。

## <a name="test-the-service-deployment"></a>測試服務部署
### <a name="connect-with-a-web-browser"></a>使用網頁瀏覽器連接
決定分割合併服務的 Web 端點。 您可以在入口網站中找到此端點，請移至雲端服務的 [概觀]，查看右邊的 [網站 URL]。 由於預設安全性設定會停用 HTTP 端點，因此以 **https://** 取代 **http://**。 在瀏覽器中載入此 URL 的網頁。

### <a name="test-with-powershell-scripts"></a>使用 PowerShell 指令碼進行測試
執行內含的範例 PowerShell 指令碼即可測試部署和您的環境。

內含的指令碼檔案：

1. **SetupSampleSplitMergeEnvironment.ps1** - 設定分割/合併的測試資料層 (請參閱下表中的詳細說明)
2. **ExecuteSampleSplitMerge.ps1** - 在測試資料層執行測試作業 (請參閱下表中的詳細說明)
3. **GetMappings.ps1** - 可印出目前分區對應狀態的最上層範例指令碼。
4. **ShardManagement.psm1** - 包裝 ShardManagement API 的協助程式指令碼
5. **SqlDatabaseHelpers.psm1** - 用來建立及管理 SQL 資料庫的協助程式指令碼
   
   <table style="width:100%">
     <tr>
       <th>PowerShell 檔案</th>
       <th>步驟</th>
     </tr>
     <tr>
       <th rowspan="5">SetupSampleSplitMergeEnvironment.ps1</th>
       <td>1.    建立分區對應管理員資料庫</td>
     </tr>
     <tr>
       <td>2.    建立 2 個分區資料庫。
     </tr>
     <tr>
       <td>3.    建立這些資料庫的分區對應 (刪除這些資料庫上的任何現有分區對應)。 </td>
     </tr>
     <tr>
       <td>4.    在這兩個分區中建立小型範例資料表，並填入其中一個分區中的資料表。</td>
     </tr>
     <tr>
       <td>5.    宣告分區化資料表的 SchemaInfo。</td>
     </tr>
   </table>
   <table style="width:100%">
     <tr>
       <th>PowerShell 檔案</th>
       <th>步驟</th>
     </tr>
   <tr>
       <th rowspan="4">ExecuteSampleSplitMerge.ps1 </th>
       <td>1.    將分割要求傳送至分割合併服務 Web 前端，要求將第一個分區的資料分割一半給第二個分區。</td>
     </tr>
     <tr>
       <td>2.    向 Web 前端輪詢分割要求的狀態，並等待完成要求。</td>
     </tr>
     <tr>
       <td>3.    將合併要求傳送至分割合併服務 Web 前端，要求將第二個分區的資料移回到第一個分區。</td>
     </tr>
     <tr>
       <td>4.    向 Web 前端輪詢合併要求的狀態，並等待完成要求。</td>
     </tr>
   </table>
   
## <a name="use-powershell-to-verify-your-deployment"></a>使用 PowerShell 來驗證您的部署
1. 開啟新的 PowerShell 視窗，並瀏覽至您下載分割-合併套件的目錄，然後瀏覽到 "powershell" 目錄。
2. 建立 Azure SQL Database 伺服器 (或選擇現有的伺服器)，其中將會建立分區對應管理員和分區。
   
   > [!NOTE]
   > SetupSampleSplitMergeEnvironment.ps1 指令碼預設會在相同的伺服器上建立所有資料庫，以簡化指令碼。 這不是分割合併服務本身的限制。
   >
   
   需要有具備 DB 讀取/寫入存取權的 SQL 驗證登入，分割合併服務才能移動資料和更新分區對應。 因為分割合併服務是在雲端執行，目前不支援整合式驗證。
   
   請確定 Azure SQL server 設定為允許從執行這些指令碼的電腦 IP 位址存取。 您可以在 [Azure SQL server / 設定 / 允許的 IP 位址] 下找到這項設定。
3. 執行 SetupSampleSplitMergeEnvironment.ps1 指令碼以建立範例環境。
   
   執行這個指令碼將會從分區對應管理員資料庫和分區上，清除任何現有的分區對應管理資料結構。 如果您想要重新初始化分區對應或分區，最好重新執行此指令碼。
   
   範例命令列：

   ```   
     .\SetupSampleSplitMergeEnvironment.ps1 
   
         -UserName 'mysqluser' 
         -Password 'MySqlPassw0rd' 
         -ShardMapManagerServerName 'abcdefghij.database.windows.net'
   ```      
4. 執行 Getmappings.ps1 指令碼以檢視範例環境中目前存在的對應。
   
   ```
     .\GetMappings.ps1 
   
         -UserName 'mysqluser' 
         -Password 'MySqlPassw0rd' 
         -ShardMapManagerServerName 'abcdefghij.database.windows.net'

   ```         
5. 執行 ExecuteSampleSplitMerge.ps1 指令碼，以執行分割作業 (將第一個分區的一半資料移至第二個分區)，然後執行合併作業 (將資料移回至第一個分區)。 如果您設定 SSL，而且保持停用 http 端點，請確定您改為使用 https:// 端點。
   
   範例命令列：

   ```   
     .\ExecuteSampleSplitMerge.ps1
   
         -UserName 'mysqluser' 
         -Password 'MySqlPassw0rd' 
         -ShardMapManagerServerName 'abcdefghij.database.windows.net' 
         -SplitMergeServiceEndpoint 'https://mysplitmergeservice.cloudapp.net' 
         -CertificateThumbprint '0123456789abcdef0123456789abcdef01234567'
   ```      
   
   如果您收到下列錯誤，很可能是 Web 端點的憑證有問題。 使用您最愛的網頁瀏覽器嘗試連線到的 Web 端點，並檢查是否有憑證錯誤。
   
     ```
     Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLSsecure channel.
     ```
   
   如果成功，輸出應該看起來如下：
   
   ```
   > .\ExecuteSampleSplitMerge.ps1 -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net' -SplitMergeServiceEndpoint 'http://mysplitmergeservice.cloudapp.net' -CertificateThumbprint 0123456789abcdef0123456789abcdef01234567
   > Sending split request
   > Began split operation with id dc68dfa0-e22b-4823-886a-9bdc903c80f3
   > Polling split-merge request status. Press Ctrl-C to end
   > Progress: 0% | Status: Queued | Details: [Informational] Queued request
   > Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
   > Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
   > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
   > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Waiting for reference tables copy     completion.
   > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
   > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
   > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
   > ...
   > ...
   > Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
   > Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
   > Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
   > Sending merge request
   > Began merge operation with id 6ffc308f-d006-466b-b24e-857242ec5f66
   > Polling request status. Press Ctrl-C to end
   > Progress: 0% | Status: Queued | Details: [Informational] Queued request
   > Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
   > Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
   > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
   > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
   > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
   > ...
   > ...
   > Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
   > Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
   > Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
   > 
   ```
6. 試驗其他資料類型！ 所有這些指令碼都採用可供指定金鑰類型的選擇性 -ShardKeyType 參數。 預設值為 Int32，但您也可以指定 Int64、Guid 或二進位。

## <a name="create-requests"></a>建立要求
您可以透過 Web UI，或匯入並使用將透過 Web 角色提交需求的 SplitMerge.psm1 PowerShell 模組，以使用此服務。

此服務可以移動分區化資料表和參考資料表中的資料。 分區資料表具有分區化索引鍵資料行，且在每個分區上有不同的資料列資料。 參考資料表並未分區，所以每個分區包含相同的資料列資料。 參考資料表適用於不常變更的資料，且在查詢中用來「聯結」分區化資料表。

若要執行分割合併作業，您必須宣告想要移動的分區化資料表和參考資料表。 這是透過 **SchemaInfo** API 來達成。 此 API 位於 **Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.Schema** 命名空間中。

1. 對於每個分區資料表，建立 **ShardedTableInfo** 物件，以描述資料表的父結構描述名稱 (選擇性，預設值為 "dbo")、資料表名稱，以及該資料表中包含分區化索引鍵的資料行名稱。
2. 對於每個參考資料表，建立 **ReferenceTableInfo** 物件，以描述資料表的父結構描述名稱 (選擇性，預設為 "dbo") 和資料表名稱。
3. 將上述 TableInfo 物件新增至新的 **SchemaInfo** 物件。
4. 取得 **ShardMapManager** 物件的參考，並呼叫 **GetSchemaInfoCollection**。
5. 將 **SchemaInfo** 新增至 **SchemaInfoCollection**，並提供分區對應名稱。

在 SetupSampleSplitMergeEnvironment.ps1 指令碼中可以看見這個範例。

「分割-合併」服務不會為您建立目標資料庫 (或資料庫中任何資料表的結構描述)。 將要求傳送至服務之前，必須先建立它們。

## <a name="troubleshooting"></a>疑難排解
執行範例 powershell 指令碼時，您可能會看到以下訊息：

   ```
   Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.
   ```

此錯誤表示您的 SSL 憑證未正確設定。 請遵循＜使用網頁瀏覽器連接＞一節的指示。

若無法提交要求，您可能會看到：

```
[Exception] System.Data.SqlClient.SqlException (0x80131904): Could not find stored procedure 'dbo.InsertRequest'. 
```

在此情況下，請檢查組態檔，特別是 **WorkerRoleSynchronizationStorageAccountConnectionString**的設定。 這個錯誤通常表示背景工作角色無法在第一次使用時成功初始化中繼資料資料庫。 

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/allowed-services.png
[2]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/manage.png
[3]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/staging.png
[4]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/upload.png
[5]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/storage.png

