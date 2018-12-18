---
title: 在向外延展的雲端資料庫之間移動資料 | Microsoft Docs
description: 說明如何使用彈性資料庫 API 透過自我託管服務操作分區和移動資料。
services: sql-database
ms.service: sql-database
ms.subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 518d7659df603ed0fcab4aebf5f35c3b92e75ccf
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162623"
---
# <a name="moving-data-between-scaled-out-cloud-databases"></a>在向外延展的雲端資料庫之間移動資料
如果您是軟體服務開發人員，您的應用程式突然出現巨量需求，您需要因應這種成長。 所以，您加入了更多的資料庫 (分區)。 您該如何將資料重新發佈到新的資料庫，卻不打斷資料的完整性？ 使用 **分割合併工具** 將資料從受條件約束的資料庫移到新的資料庫。  

分割合併工具執行的方式如同 Azure Web 服務。 系統管理員或開發人員使用工具在不同的資料庫 (分區) 之間移動 Shardlet (分區的資料)。 此工具會使用分區對應管理來維護服務中繼資料資料庫，並確保一致的對應。

![概觀][1]

## <a name="download"></a>下載
[Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge/)

## <a name="documentation"></a>文件
1. [彈性資料庫分割合併工具教學課程](sql-database-elastic-scale-configure-deploy-split-and-merge.md)
2. [Split-Merge 安全性設定](sql-database-elastic-scale-split-merge-security-configuration.md)
3. [分割合併安全性考量](sql-database-elastic-scale-split-merge-security-configuration.md)
4. [分區對應管理](sql-database-elastic-scale-shard-map-management.md)
5. [轉換現有的資料庫以使用彈性資料庫工具](sql-database-elastic-convert-to-use-elastic-tools.md)
6. [彈性資料庫功能概觀](sql-database-elastic-scale-introduction.md)
7. [彈性資料庫工具字彙](sql-database-elastic-scale-glossary.md)

## <a name="why-use-the-split-merge-tool"></a>為何使用分割合併工具？
**彈性**

應用程式需要靈活地伸展，才能超越單一 Azure SQL DB 資料庫的限制。 使用工具依需要將資料移至新的資料庫，同時保有完整性。

**分割以成長** 

您需要增加整體容量，處理爆炸性的成長。 若要達到這個目標，請將資料分區，並分散到越來越多的資料庫上，來提供更多的容量，直到滿足容量需求為止。 這是主要的「分割」功能範例。 

**合併以縮減**

容量因為業務的季節特性需要縮減。 當業務減緩時，此工具可讓您相應減少至較少的縮放單位。 Elastic Scale 分割合併服務的「合併」功能可滿足此需求。 

**移動 Shardlet 來管理作用區**

在每個資料庫有多個租用戶的情況下，分區的 Shardlet 配置可能造成某些分區出現容量瓶頸。 這需要重新配置 Shardlet，或將忙碌的 Shardlet 移到新的或較少使用的分區。 

## <a name="concepts--key-features"></a>概念和重要功能
**客戶主控式服務**

分割合併提供為客戶主控式服務。 您必須將服務部署並裝載於 Microsoft Azure 訂用帳戶中。 您從 NuGet 下載的封裝包含設定範本，可加入您特定部署的資訊而使之完備。 如需詳細資訊，請參閱 [分割合併教學課程](sql-database-elastic-scale-configure-deploy-split-and-merge.md) 。 因為服務是在您的 Azure 訂用帳戶中執行，您可以控制和設定服務的大部分安全性層面。 預設範本包含設定 SSL、以憑證為基礎的用戶端驗證、加密儲存的認證、DoS 防護及 IP 限制等選項。 您可以在下列文件中找到安全性層面的詳細資訊： [分割合併安全性組態](sql-database-elastic-scale-split-merge-security-configuration.md)。

預設部署服務以一個背景工作角色和一個 Web 角色執行。 各角色在 Azure 雲端服務中使用 A1 VM 大小。 雖然您無法在部署封裝時修改這些設定，但可以在部署成功後在執行中的雲端服務中變更它們 (透過 Azure 入口網站)。 請注意，基於技術原因，不可以將背景工作角色設定多個執行個體。 

**分區對應整合**

分割合併服務會與應用程式的分區對應互動。 如果使用分割合併服務來分割或合併範圍，或在分區之間移動 Shardlet，則服務會自動將分區對應保持在最新狀態。 為了這樣做，服務會連接到應用程式的分區對應管理員資料庫，並於分割/合併/移動要求進行時維護範圍和對應。 這可確保當分割合併作業持續進行時，分區對應一律呈現最新的檢視。 分割、合併和 Shardlet 移動作業的實作方式是將一批 Shardlet 從來源分區移至目標分區。 在 Shardlet 移動作業期間，屬於目前批次中的 Shardlet 在分區對應中會標示為離線，也無法供使用 **OpenConnectionForKey** API 的資料相依路由連線使用。 

**一致的 Shardlet 連線**

當新一批 Shardlet 的資料移動開始時，分區對應提供指向分區 (儲存 Shardlet) 的任何資料相依路由連線會終止，而當資料移動正在進行時，也會封鎖後續從分區對應 API 至 Shardlet 的連線，以避免不一致。 對同一個分區的其他 Shardlet 的連接也會終止，但重試就會立即成功。 移動批次後，目標分區的 Shardlet 會再次標示為線上，並從來源分區移除來源資料。 服務會對每個批次執行這些步驟，直到所有 Shardlet 都已經移動為止。 在整個分割/合併/移動作業的過程中，這將會產生數個連接終止作業。  

**管理 Shardlet 可用性**

如上所述，只限制終止目前 Shardlet 批次的連接，可將無法使用的範圍限制在一次一批 Shardlet。 這比分割或合併作業過程中整個分區對其所有 Shardlet 仍保持離線的作法更好。 批次大小 (定義為一次要移動的相異 Shardlet 數目) 是組態參數。 每個分割和合併作業中都可定義它，視應用程式的可用性和效能需求而定。 請注意，分區對應中鎖定的範圍可能大於指定的批次大小。 這是因為服務挑選範圍大小時，都希望資料中的分區化索引鍵值的實際數大約符合批次大小。 對於稀疏填入的分區化索引鍵，尤其要記住這一點。 

**中繼資料儲存體**

分割合併服務使用資料庫來維護其狀態，並在要求處理期間保留記錄檔。 使用者在其訂用帳戶中建立此資料庫，並在服務部署的組態檔中提供連接字串。 使用者組織的系統管理員也可以連接到此資料庫來檢閱要求進度，並調查關於潛在失敗的詳細資訊。

**分區化感知**

分割合併服務會區別 (1) 分區化資料表、(2) 參考資料表和 (3) 一般資料表。 分割/合併移動作業的語意視使用的資料表類型而定，定義如下： 

* **分區化資料表**：分割、合併和移動作業會將 Shardlet 從來源移至目標分區。 成功完成整體要求之後，這些 Shardlet 就不存在於來源。 請注意，目標資料表必須存在於目標分區，而且在處理作業之前不能包含目標範圍中的資料。 
* **參考資料表**：對於參考資料表，分割、合併和移動作業會將資料從來源複製到目標分區。 但是請注意，若目標上給定資料表中已存在任何資料列，則此資料表的目標分區不會發生任何變更。 資料表必須是空的，才會處理任何參考資料表複製作業。
* **其他資料表**：分割和合併作業的來源或目標上可以存在其他資料表。 在任何資料移動或複製作業中，分割合併服務會忽略這些資料表。 但是請注意，在有條件約束時，它們會干擾這些作業。

分區對應的 **SchemaInfo** API 提供參考資料表與分區化資料表的比較資訊。 下列範例說明如何在給定的分區對應管理員物件 smm 上使用這些 API： 

    // Create the schema annotations 
    SchemaInfo schemaInfo = new SchemaInfo(); 

    // Reference tables 
    schemaInfo.Add(new ReferenceTableInfo("dbo", "region")); 
    schemaInfo.Add(new ReferenceTableInfo("dbo", "nation")); 

    // Sharded tables 
    schemaInfo.Add(new ShardedTableInfo("dbo", "customer", "C_CUSTKEY")); 
    schemaInfo.Add(new ShardedTableInfo("dbo", "orders", "O_CUSTKEY")); 

    // Publish 
    smm.GetSchemaInfoCollection().Add(Configuration.ShardMapName, schemaInfo); 

資料表 '‘region’ 及 ‘nation’ 定義為參考資料表，將透過分割/合併/移動作業來複製。 ‘customer’ 和 ‘orders’ 接著定義為分區化資料表。 C_CUSTKEY 和 O_CUSTKEY 作為分區化索引鍵。 

**參考完整性**

分割合併服務會分析資料表之間的相依性，並使用外部主索引鍵關聯性來接移作業，以移動參考資料表和 Shardlet。 一般而言會先根據相依性順序複製參考資料表，然後在每個批次中再根據相依性順序複製 Shardlet。 這是必要的，當新資料到達時，才能符合目標分區的 FK PK 條件約束。 

**分區對應一致性與最終完成**

失敗時，分割合併服務會在任何中斷之後繼續作業，以完成任何進行中的要求。 不過，可能有無法復原的情況，例如，當目標分區遺失或損壞到無法修復的程度。 在這些情況下，原本要移動的一些 Shardlet 可能繼續留在來源分區。 服務可以確保只有當必要的資料成功複製到目標之後，才會更新 Shardlet 對應。 只有當所有資料都已複製到目標，並已成功更新對應的對應之後，才會在來源刪除 Shardlet。 當範圍在目標分區上已上線時，刪除作業會在幕後執行。 分割合併服務永遠確保儲存在分區對應中的對應的正確性。

## <a name="the-split-merge-user-interface"></a>分割合併使用者介面
分割合併 Service Pack 包含背景工作角色和 Web 角色。 Web 角色用互動方式提交分割合併要求。 使用者介面的主要元件如下：

* 作業類型：作業類型是選項按鈕，控制服務針對此要求所執行的作業類型。 您可以選擇分割、合併和移動案例。 您也可以取消先前提交的作業。 您可以在範圍分區對應中使用分割、合併和移動等要求。 清單分區對應僅支援移動作業。
* 分區對應：下一節的要求參數討論分區對應和主控分區對應之資料庫的相關資訊。 特別的是，您需要提供 Azure SQL Database 伺服器名稱和裝載 shardmap 的資料庫名稱、用以連接到分區對應資料庫的認證，以及分區對應名稱的名稱。 目前，此作業只接受一組認證。 這些認證必須有足夠權限來變更分區對應及分區上的使用者資料。
* 來源範圍 (分割和合併)：分割及合併作業會採用其低和高索引鍵處理該範圍。 若要指定無限制高索引鍵值的作業，請勾選 [高索引鍵為最大值] 核取方塊，並將高索引鍵欄位留空。 指定的範圍索引鍵值不需要精確地符合分區對應中的對應及其界限。 如果您未指定任何範圍界限，此服務將會自動為您推斷最接近的範圍。 您可以使用 GetMappings.ps1 PowerShell 指令碼來擷取給定分區對應中目前的對應。
* 分割來源行為 (分割)：在分割作業中，定義來源範圍中要分割的點。 您可以提供想要從何處分割的分區化索引鍵。 請使用選項按鈕指定要移動範圍的下半部 (不含分割索引鍵)，還是要移動上半部 (包括分割索引鍵)。
* 來源 Shardlet (移動)：移動作業不同於分割或合併作業，它們不需要範圍來描述來源。 使用您打算移動的分區化索引鍵值，即可識別移動的來源。
* 目標分區 (分割)：提供分割作業的來源相關資訊之後，您需要提供目標的 Azure SQL Db 伺服器和資料庫名稱，以定義您想要將資料複製到何處。
* 目標範圍 (合併)：合併作業會將 Shardlet 移至現有的分區。 您可以提供想要合併的現有範圍的範圍界限，以識別現有的分區。
* 批次大小：批次大小控制資料移動期間會一次離線的 Shardlet 數目。 這是整數值，當您無法忍受長時間沒有 Shardlet 可用時，您可以使用較小的值。 較大的值會延長給定 Shardlet 離線的時間，但可改善效能。
* 作業識別碼 (取消)：如果您不再需要某個進行中的作業，您可以在此欄位中提供其作業識別碼來取消作業。 您可以從要求狀態資料表 (請參閱 8.1 節)，或從您提交要求的網頁瀏覽器中的輸出，擷取作業識別碼。

## <a name="requirements-and-limitations"></a>需求和限制
目前的分割合併服務實作有下列需求和限制： 

* 分區必須存在且在分區對應中註冊，才能對這些分區執行分割合併作業。 
* 服務不會在其作業中自動建立資料表或其他任何資料庫物件。 這表示執行任何分割/合併/移動作業之前，所有分區化資料表和參考資料表的結構描述必須存在於目標分區。 尤其，在由分割/合併/移動作業加入新 Shardlet 的範圍中，分區化資料表必須是空的。 否則，作業無法通過目標分區上的初始一致性檢查。 另請注意，只有當參考資料表是空的時，才會複製參考資料，而對於參考資料表上的其他並行寫入作業，也無法保證一致性。 我們建議 – 在執行分割/合併作業時，沒有其他寫入作業在變更參考資料表。
* 服務依賴唯一索引或索引鍵 (包含分區化索引鍵) 所建立的資料列識別，以改善大型 Shardlet 的效能和可靠性。 這可讓服務移動比分區化索引鍵值更精細的資料。 這有助於減少作業期間所需的記錄檔空間和鎖定數目上限。 如果您想要透過分割/合併/移動要求來使用特定的資料表，請考慮在此資料表上建立唯一索引或主索引鍵 (包含分區化索引鍵)。 基於效能考量，分區化索引鍵應該為索引鍵或索引中的開頭資料行。
* 要求處理期間，某些 Shardlet 資料可能會同時出現在來源和目標分區。 為了防止 Shardlet 移動期間失敗，這是必要的。 分割合併與分區對應的整合，可確保在分區對應上透過資料相依路由 API 使用 **OpenConnectionForKey** 方法來連線時，不會出現任何不一致的過渡狀態。 不過，當連線到來源或目標分區不是使用 **OpenConnectionForKey** 方法時，則在分割/合併/移動要求進行時，可能會出現不一致的過渡狀態。 這些連接可能會顯示不完整或重複的結果，視連接之下的時間或分區而定。 這項限制目前包括 Elastic Scale 多分區查詢所建立的連接。
* 不同角色之間不可共用分割合併服務的中繼資料資料庫。 例如，在預備環境中執行分割合併服務的角色，必須指向與生產角色不同的中繼資料資料庫。

## <a name="billing"></a>計費
分割合併服務是以 Microsoft Azure 訂用帳戶中的雲端服務執行。 因此會對您的服務執行個體收取雲端服務的費用。 除非您經常執行分割/合併/移動作業，否則建議您刪除分割合併雲端服務。 這可以節省執行中或已部署的雲端服務執行個體的成本。 每當您需要執行分割或合併作業時，您可以重新部署和啟動可立即運作的組態。 

## <a name="monitoring"></a>監視
### <a name="status-tables"></a>狀態資料表
分割合併服務在中繼資料儲存資料庫中提供 **RequestStatus** 資料表，以監視已完成和進行中的要求。 已提交至這個分割合併服務執行個體的每一個分割合併要求，在此資料表中都會列出一個資料列。 它針對每個要求提供以下的資訊：

* **Timestamp**：要求開始的時間和日期。
* **OperationId**：唯一識別要求的 GUID。 這項要求也可用來取消仍在進行中的作業。
* **Status**：要求的目前狀態。 對於持續進行的要求，它也列出要求目前所處的階段。
* **CancelRequest**：旗標，指出是否已取消要求。
* **Progress**：作業完成的估計百分比。 值為 50 表示作業已完成大約 50%。
* **Details**：XML 值，提供更詳細的進度報表。 隨著資料列從來源複製到目標，進度報表會定期更新。 如果發生錯誤或例外狀況，此資料行也包含失敗的詳細資訊。

### <a name="azure-diagnostics"></a>Azure 診斷
分割合併服務會根據 Azure SDK 2.5 使用 Azure 診斷來監控與診斷。 您可以如這裡所述控制診斷組態： [在 Azure 雲端服務和虛擬機器中啟用診斷](../cloud-services/cloud-services-dotnet-diagnostics.md)。 下載封裝包含兩個診斷組態 - 一個用於 Web 角色，另一個用於背景工作角色。 這些服務診斷組態遵循 [Microsoft Azure 雲端服務基本概念](https://code.msdn.microsoft.com/windowsazure/Cloud-Service-Fundamentals-4ca72649)中的指引。 其中包含定義來記錄效能計數器、IIS 記錄檔、Windows 事件記錄檔，以及分割合併應用程式事件記錄檔。 

## <a name="deploy-diagnostics"></a>部署診斷
針對 NuGet 封裝所提供的 Web 和背景工作角色，若要使用診斷組態啟用監視和診斷，請使用 Azure PowerShell 執行下列命令： 

    $storage_name = "<YourAzureStorageAccount>" 

    $key = "<YourAzureStorageAccountKey" 

    $storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key  


    $config_path = "<YourFilePath>\SplitMergeWebContent.diagnostics.xml" 

    $service_name = "<YourCloudServiceName>" 

    Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name -Slot Production -Role "SplitMergeWeb" 


    $config_path = "<YourFilePath>\SplitMergeWorkerContent.diagnostics.xml" 

    $service_name = "<YourCloudServiceName>" 

    Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name -Slot Production -Role "SplitMergeWorker" 

您可以在這裡找到有關如何設定和部署診斷設定的詳細資訊： [在 Azure 雲端服務和虛擬機器中啟用診斷](../cloud-services/cloud-services-dotnet-diagnostics.md)。 

## <a name="retrieve-diagnostics"></a>擷取診斷
從 Visual Studio 伺服器總管，您可以在伺服器總管樹狀目錄的 Azure 部分中輕鬆存取診斷。 開啟 Visual Studio 執行個體，在功能表列按一下 [檢視] 和 [伺服器總管]。 按一下 [Azure] 圖示，連接至您的 Azure 訂用帳戶。 然後瀏覽至 [Azure] -> [儲存體] -> <your storage account> -> [資料表] -> [WADLogsTable]。 如需詳細資訊，請參閱 [使用伺服器總管瀏覽儲存體資源](http://msdn.microsoft.com/library/azure/ff683677.aspx)。 

![WADLogsTable][2]

在上圖中反白顯示的 WADLogsTable 包含分割合併服務應用程式記錄檔中的詳細事件。 請注意，下載的封裝的預設組態是特別針對生產部署而設計。 因此，從服務執行個體提取記錄檔和計數器的間隔時間很大 (5 分鐘)。 在測試和開發時，請依需要調整 Web 或背景工作角色的診斷設定，以縮短間隔。 請以滑鼠右鍵按一下 Visual Studio 伺服器總管 (請參閱上圖) 中的角色，然後調整 [診斷組態] 設定對話方塊中的 [傳輸期間]： 

![組態][3]

## <a name="performance"></a>效能
一般而言，Azure SQL Database 中越高階、越有效能的服務層，預期會有較佳的效能。 越高的服務層使用越高的 IO、CPU 和記憶體配置，有利於分割合併服務使用的大量複製和刪除作業。 基於這個理由，請只針對這些資料庫，在一段已定義的有限期間內增加服務層。

服務在其正常作業中也會執行驗證查詢。 這些驗證查詢會檢查目標範圍中是否存在非預期的資料，並確保任何分割/合併/移動作業是從一致的狀態開始。 這些查詢全部都會檢查作業領域所定義的分區化索引鍵範圍，以及要求定義中所提供的批次大小。 當索引存在且以分區化索引鍵做為開頭資料行時，這些查詢的表現最好。 

此外，以分區化索引鍵做為開頭資料行的唯一性屬性，可讓服務使用最佳化方法來限制記錄檔空間和記憶體方面的資源消耗。 移動大型資料時 (通常超過 1 GB) 需要這個唯一性屬性。 

## <a name="how-to-upgrade"></a>如何升級
1. 請遵循 [部署分割-合併服務](sql-database-elastic-scale-configure-deploy-split-and-merge.md)中的步驟執行。
2. 變更分割合併部署的雲端服務組態檔，以反映新的組態參數。 新的必要參數是用於加密的憑證相關資訊。 若要這樣做，一個簡單方法是將下載的新組態範本檔案與現有組態進行比較。 請確定您新增 Web 和背景工作角色的 "DataEncryptionPrimaryCertificateThumbprint" 和 "DataEncryptionPrimary" 設定。
3. 將更新部署至 Azure 之前，請確定目前執行的所有分割合併作業都已完成。 作法很簡單，您可以針對進行中的要求，查詢分割合併中繼資料資料庫中的 RequestStatus 和 PendingWorkflows 資料表。
4. 使用新的封裝和更新的服務組態檔，在您的 Azure 訂用帳戶中更新分割合併的現有雲端服務部署。

您無需佈建新的中繼資料資料庫，即可升級分割合併。 新的版本會自動將現有的中繼資料資料庫升級到新的版本。 

## <a name="best-practices--troubleshooting"></a>最佳作法和疑難排解
* 定義測試租用戶，並在數個分區上對測試租用戶練習您最重要的分割/合併/移動作業。 確定分區對應中定義的所有中繼資料都正確，且作業未違反條件約束或外部索引鍵。
* 將測試租用戶資料大小保持大於您最大租用戶的最大資料大小，以確保不會發生資料大小的相關問題。 這有助於您評估移動單一租用戶所花費的時間上限。 
* 請確定您的結構描述允許刪除動作。 一旦資料成功複製到目標，分割合併服務必須能夠從來源分區移除資料。 例如， **刪除觸發程序** 可能防止服務刪除來源上的資料，也可能造成作業失敗。
* 分區化索引鍵應該為主索引鍵或唯一索引定義中的開頭資料行。 如此可確保分割或合併驗證查詢，以及永遠在分區化索引鍵範圍上執行的實際資料移動和刪除作業，獲得最佳效能。
* 將分割合併服務共置在您的資料庫所在的區域和資料中心。 

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->
[1]:./media/sql-database-elastic-scale-overview-split-and-merge/split-merge-overview.png
[2]:./media/sql-database-elastic-scale-overview-split-and-merge/diagnostics.png
[3]:./media/sql-database-elastic-scale-overview-split-and-merge/diagnostics-config.png

