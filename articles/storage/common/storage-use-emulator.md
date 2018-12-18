---
title: 使用 Azure 儲存體模擬器進行開發和測試 | Microsoft Docs
description: Azure 儲存體模擬器提供免費的本機開發環境，針對 Azure 儲存體應用程式進行開發和測試。 了解如何授權要求、如何從應用程式連線到模擬器，以及如何使用命令列工具。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 08/10/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 529612aeecfcea1d775c2f4359c5135ca3c6885e
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44052538"
---
# <a name="use-the-azure-storage-emulator-for-development-and-testing"></a>使用 Azure 儲存體模擬器進行開發和測試

Microsoft Azure 儲存體模擬器提供了模擬 Azure Blob、佇列和資料表服務的本機環境，以供進行開發。 您可以使用儲存體模擬器，針對儲存體服務在本機測試您的應用程式，而不需建立 Azure 訂用帳戶，也不會產生任何費用。 如果您滿意應用程式在模擬器中的運作方式，就可以切換成使用雲端的 Azure 儲存體帳戶。

## <a name="get-the-storage-emulator"></a>取得儲存體模擬器
儲存體模擬器隨 [Microsoft Azure SDK](https://azure.microsoft.com/downloads/)提供。 您也可以使用[獨立安裝程式](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409) (直接下載) 來安裝儲存體模擬器。 若要安裝儲存體模擬器，您必須具有電腦上的系統管理權限。

儲存體模擬器目前只能在 Windows 上執行。 針對考慮適用於 Linux 之儲存體模擬器的使用者，有一個選擇是社群維護的開放原始碼儲存體模擬器 [Azurite](https://github.com/azure/azurite)。

> [!NOTE]
> 在某個儲存體模擬器版本中建立的資料不保證可在使用不同版本時加以存取。 如果您需要長期保存資料，建議您將該資料儲存於 Azure 儲存體帳戶中 (而不是儲存體模擬器中)。
> 
> 儲存體模擬器取決於特定的 OData 程式庫版本。 不支援將儲存體模擬器所使用的 OData DLL 更換為其他版本，而且可能會造成非預期的行為。 不過，儲存體服務支援的任何版本 OData 可能用來將要求傳送至模擬器。

## <a name="how-the-storage-emulator-works"></a>儲存體模擬器的運作方式
儲存體模擬器會使用本機 Microsoft SQL Server 執行個體及本機檔案系統來模擬 Azure 儲存體服務。 儲存體模擬器預設會使用 Microsoft SQL Server 2012 Express LocalDB 中的資料庫。 您可以選擇設定儲存體模擬器存取 SQL Server 本機執行個體，而非 LocalDB 執行個體。 如需詳細資訊，請參閱本文稍後的[啟動及初始化儲存體模擬器](#start-and-initialize-the-storage-emulator)一節。

儲存體模擬器會使用 Windows 驗證，連接到 SQL Server 或 LocalDB。

儲存體模擬器與 Azure 儲存體服務的功能有所差異。 如需有關這些差異的詳細資訊，請參閱本文稍後的[儲存體模擬器和 Azure 儲存體之間的差異](#differences-between-the-storage-emulator-and-azure-storage)一節。

## <a name="start-and-initialize-the-storage-emulator"></a>啟動及初始化儲存體模擬器

啟動 Azure 儲存體模擬器：
1. 選取 [開始] 按鈕或按下 [Windows] 鍵。
2. 開始輸入 `Azure Storage Emulator`。
3. 從顯示的應用程式清單中選取模擬器。

當儲存體模擬器啟動時，將會出現 [命令列提示字元] 視窗。 您可以使用此主控台視窗，來啟動和停止儲存體模擬器、清除資料、取得狀態，以及初始化模擬器。 如需詳細資訊，請參閱本文稍後的[儲存體模擬器命令列工具參考](#storage-emulator-command-line-tool-reference)一節。

如果模擬器正在執行，您就會在 Windows 工作列通知區域看到圖示。

當您關閉儲存體模擬器的 [命令提示字元] 視窗時，儲存體模擬器將會繼續執行。 若要再次顯示 [儲存體模擬器] 主控台視窗，請遵循先前步驟，如同啟動儲存體模擬器一樣。

當您第一次執行儲存體模擬器時，系統會為您初始化本機儲存體環境。 初始化處理程序會在 LocalDB 中建立資料庫，並為每個本機儲存體服務保留 HTTP 連接埠。

預設會將儲存體模擬器安裝至 `C:\Program Files (x86)\Microsoft SDKs\Azure\Storage Emulator`。

> [!TIP]
> 您可以使用 [Microsoft Azure 儲存體總管](http://storageexplorer.com)來使用本機儲存體模擬器資源。 當您安裝並啟動儲存體模擬器之後，在 [儲存體總管] 資源樹狀目錄中的 [儲存體帳戶] 下方尋找 [(開發)]。
>

### <a name="initialize-the-storage-emulator-to-use-a-different-sql-database"></a>初始化儲存體模擬器以使用不同的 SQL 資料庫

您可以使用儲存體模擬器命令列工具來初始化儲存體模擬器，以指向預設 LocalDB 執行個體以外的 SQL 資料庫執行個體：

1. 開啟 [儲存體模擬器] 主控台視窗，如[啟動及初始化儲存體模擬器](#start-and-initialize-the-storage-emulator)一節中所述。
1. 在主控台視窗中，輸入下列命令，其中 `<SQLServerInstance>` 是 SQL Server 執行個體的名稱。 若要使用 LocalDB，請指定 `(localdb)\MSSQLLocalDb` 做為 SQL Server 執行個體。

  `AzureStorageEmulator.exe init /server <SQLServerInstance>`

  您也可以使用下列命令，引導模擬器使用預設的 SQL Server 執行個體：

  `AzureStorageEmulator.exe init /server .`

  或者，您也可以使用下列命令，將資料庫重新初始化至預設 LocalDB 執行個體：

  `AzureStorageEmulator.exe init /forceCreate`

如需有關這些命令的詳細資訊，請參閱[儲存體模擬器命令列工具參考](#storage-emulator-command-line-tool-reference)。

> [!TIP]
> 您可以使用 [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) 來管理您的 SQL Server 執行個體，包括 LocalDB 安裝。 在 SMSS [連接到伺服器] 對話方塊中，於 [伺服器名稱:] 欄位中指定 `(localdb)\MSSQLLocalDb`，以連接到 LocalDB 執行個體。

## <a name="authenticating-requests-against-the-storage-emulator"></a>對儲存體模擬器的驗證要求
一旦您安裝並啟動儲存體模擬器之後，就可以針對它測試您的程式碼。 就像雲端中的 Azure 儲存體一樣，您傳送給儲存體模擬器的每個要求都必須經過授權，除非它是匿名的要求。 您可以使用共用金鑰驗證或共用存取簽章 (SAS)，授權傳送給儲存體模擬器的要求。

### <a name="authorize-with-shared-key-credentials"></a>使用共用金鑰認證進行授權
[!INCLUDE [storage-emulator-connection-string-include](../../../includes/storage-emulator-connection-string-include.md)]

如需連接字串的詳細資訊，請參閱[設定 Azure 儲存體連接字串](../storage-configure-connection-string.md)。

### <a name="authorize-with-a-shared-access-signature"></a>使用共用存取簽章進行授權
某些 Azure 儲存體用戶端程式庫 (例如 Xamarin 程式庫)，僅支援使用共用存取簽章 (SAS) 權杖進行的驗證。 您可以使用類似[儲存體總管](http://storageexplorer.com/)的工具或其他支援共用金鑰驗證的應用程式來建立 SAS 權杖。

您也可以使用 Azure PowerShell 來產生 SAS 權杖。 下列範例會產生 SAS 權杖且對 blob 容器具有完整權限：

1. 安裝 Azure PowerShell (如果您尚未安裝，建議使用 Azure PowerShell Cmdlet 的最新版本)。 如需安裝指示，請參閱[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。
2. 開啟 Azure PowerShell 並執行下列命令，使用您所選的名稱來取代 `CONTAINER_NAME`：

```powershell
$context = New-AzureStorageContext -Local

New-AzureStorageContainer CONTAINER_NAME -Permission Off -Context $context

$now = Get-Date

New-AzureStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri
```

針對新容器產生的共用存取簽章 URI 應該類似下列項目：

```
http://127.0.0.1:10000/devstoreaccount1/sascontainer?sv=2012-02-12&se=2015-07-08T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss
```

使用此範例建立的共用存取簽章的效期為一天。 簽章會將完整存取權限 (讀取、寫入、刪除、列出) 授與容器內的 Blob。

如需共用存取簽章的詳細資訊，請參閱[在 Azure 儲存體中使用共用存取簽章 (SAS)](../storage-dotnet-shared-access-signature-part-1.md)。

## <a name="addressing-resources-in-the-storage-emulator"></a>在儲存體模擬器中為資源定址
儲存體模擬器的服務端點與 Azure 儲存體帳戶的服務端點不同。 這項差別的原因是，本機電腦不會執行網域名稱解析，需要儲存體模擬器端點為本機位址。

當您在 Azure 儲存體帳戶中定址資源時，可使用下列配置。 帳戶名稱是 URI 主機名稱的一部分，而要定址的資源是 URI 路徑的一部分：

`<http|https>://<account-name>.<service-name>.core.windows.net/<resource-path>`

例如，下列 URI 是 Azure 儲存體帳戶中的有效 blob 位址：

`https://myaccount.blob.core.windows.net/mycontainer/myblob.txt`

但是，在儲存體模擬器中，因為本機電腦不會執行網域名稱解析，所以帳戶名稱是 URI 路徑的一部分，而不是主機名稱的一部分。 針對儲存體模擬器中的資源使用下列 URI 格式：

`http://<local-machine-address>:<port>/<account-name>/<resource-path>`

例如，下列位址可能會用於存取儲存體模擬器中的 Blob：

`http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt`

儲存體模擬器的服務端點包括：

* Blob 服務：`http://127.0.0.1:10000/<account-name>/<resource-path>`
* 佇列服務：`http://127.0.0.1:10001/<account-name>/<resource-path>`
* 表格服務：`http://127.0.0.1:10002/<account-name>/<resource-path>`

### <a name="addressing-the-account-secondary-with-ra-grs"></a>使用 RA-GRS 為帳戶次要位址定址
從 3.1 版開始，儲存體模擬器就支援讀取存取地理備援複寫 (RA-GRS)。 針對同時位於雲端和本機模擬器中的儲存體資源，您可以藉由將 -secondary 附加到帳戶名稱來存取次要位置。 例如，下列位址可能會用於在儲存體模擬器中使用唯讀的次要位置來存取 Blob：

`http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt`

> [!NOTE]
> 如需以程式設計方式使用儲存體模擬器來存取次要位置，請使用 Storage Client Library for.NET 3.2 版或更新版本。 如需詳細資訊，請參閱 [Microsoft Azure Storage Client Library for .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx) (適用於 .NET 的 Microsoft Azure 儲存體用戶端程式庫)。
>
>

## <a name="storage-emulator-command-line-tool-reference"></a>儲存體模擬器命令列工具參考
從 3.0 版開始，當您啟動儲存體模擬器時，就會顯示主控台視窗。 在主控台視窗中使用命令列，來啟動和停止模擬器，以及查詢狀態和執行其他作業。

> [!NOTE]
> 如果您已安裝 Microsoft Azure 計算模擬器，當您啟動儲存體模擬器時，就會出現系統匣圖示。 使用滑鼠右鍵按一下圖示即可顯示功能表，提供圖形化方式來啟動和停止儲存體模擬器。
>
>

### <a name="command-line-syntax"></a>命令列語法
`AzureStorageEmulator.exe [start] [stop] [status] [clear] [init] [help]`

### <a name="options"></a>選項
若要檢視選項清單，請在命令提示字元輸入 `/help` 。

| 選項 | 說明 | 命令 | 引數 |
| --- | --- | --- | --- |
| **啟動** |啟動儲存體模擬器。 |`AzureStorageEmulator.exe start [-inprocess]` |*-inprocess*：在目前的處理序中啟動模擬器，而不是建立新的處理序。 |
| **停止** |停止儲存體模擬器。 |`AzureStorageEmulator.exe stop` | |
| **狀態** |列印儲存體模擬器的狀態。 |`AzureStorageEmulator.exe status` | |
| **Clear** |清除命令列上指定的所有服務中的資料。 |`AzureStorageEmulator.exe clear [blob] [table] [queue] [all]                                                    ` |*blob*：清除 blob 資料。 <br/>*queue*：清除佇列資料。 <br/>*table*：清除資料表資料。 <br/>*all*：清除所有服務中的所有資料。 |
| **Init** |執行一次初始化以設定模擬器。 |<code>AzureStorageEmulator.exe init [-server serverName] [-sqlinstance instanceName] [-forcecreate&#124;-skipcreate] [-reserveports&#124;-unreserveports] [-inprocess]</code> |*-server serverName\instanceName*︰指定裝載 SQL 執行個體的伺服器。 <br/>*-sqlinstance instanceName*：指定在預設伺服器執行個體中使用之 SQL 執行個體的名稱。 <br/>*-forcecreate*：強制建立 SQL 資料庫，即使它已經存在。 <br/>*-skipcreate*︰略過建立 SQL 資料庫。 其優先順序高於 -forcecreate。<br/>*-reserveports*︰嘗試保留與服務相關聯的 HTTP 連接埠。<br/>*-unreserveports*︰嘗試移除服務相關聯 HTTP 連接埠的保留。 其優先順序高於 -reserveports。<br/>*-inprocess*：在目前的處理序中執行初始化，而不是繁衍新的處理序。 如果變更連接埠保留，必須以提高權限啟動目前的處理程序。 |

## <a name="differences-between-the-storage-emulator-and-azure-storage"></a>儲存體模擬器和 Azure 儲存體之間的差異
因為儲存體模擬器是在本機 SQL 執行個體中執行的模擬環境，所以模擬器和雲端 Azure 儲存體帳戶之間會有功能上的差異：

* 儲存體模擬器僅支援單一固定帳戶及已知的驗證金鑰。
* 儲存體模擬器不是可擴充的儲存體服務，也不支援大量並行用戶端。
* 如 [在儲存體模擬器中定址資源](#addressing-resources-in-the-storage-emulator)中所述，資源在儲存體模擬器與在 Azure 儲存體帳戶中的定址方式不同。 這項差別的原因是，雲端提供網域名稱解析，而本機電腦沒有提供。
* 從 3.1 版開始，儲存體模擬器帳戶就支援讀取存取地理備援複寫 (RA-GRS)。 在模擬器中，所有帳戶均已啟用 RA-GRS，而且主要和次要複本之間絕對不會有延遲。 次要帳戶支援 Get Blob Service Stats、Get Queue Service Stats 和 Get Table Service Stats 作業，且這些作業一律會根據基礎 SQL 資料庫傳回 `LastSyncTime` 回應元素的值當成目前的時間。
* 儲存體模擬器目前不支援檔案服務和 SMB 通訊協定服務端點。
* 如果模擬器尚不支援您使用的儲存體服務版本，該儲存體模擬器就會傳回 VersionNotSupportedByEmulator 錯誤 (HTTP 狀態碼 400-不正確的要求)。

### <a name="differences-for-blob-storage"></a>Blob 儲存體的差異
下列差異適用於模擬器中的 Blob 儲存體：

* 儲存體模擬器最多只支援 2 GB 的 blob 大小。
* 儲存體模擬器中的 Blob 名稱長度上限是 256 個字元，而 Azure 儲存體中的 Blob 名稱長度上限是 1024 個字元。
* 增量複本會讓來自覆寫 blob 的快照進行複製，其會在服務時傳回失敗。
* 取得頁面範圍差異無法在使用累加複製 Blob 複製的快照之間運作。
* 針對存在於儲存體模擬器中的 blob 進行的 Put Blob 作業會成功，並擁有作用中的租用 (即使要求中並未指定租用識別碼)。
* 模擬器不支援附加 Blob 作業。 在附加 Blob 上嘗試作業會傳回 FeatureNotSupportedByEmulator 錯誤 (HTTP 狀態碼 400-不正確的要求)。

### <a name="differences-for-table-storage"></a>資料表儲存體的差異
下列差異適用於模擬器中的資料表儲存體：

* 在儲存體模擬器中，表格服務的日期屬性只支援 SQL Server 2005 所支援的範圍 (它們必須晚於 1753 年 1 月 1 日)。 1753 年 1 月 1 日之前的所有日期都會變成此值。 日期的精確度受限於 SQL Server 2005 的精確度，也就是會精確度達到 1/300 秒。
* 儲存體模擬器支援屬性值小於 512 個位元組的資料分割索引鍵與資料列索引鍵。 此外，帳戶名稱、資料表名稱和索引鍵屬性名稱的大小總計不得超過 900 個位元組。
* 在儲存體模擬器中，資料表的資料列大小總計限制為小於 1 MB。
* 在儲存體模擬器中，資料類型 `Edm.Guid` 或 `Edm.Binary` 的屬性只支援查詢篩選字串中的 `Equal (eq)` 與 `NotEqual (ne)` 比較運算子。

### <a name="differences-for-queue-storage"></a>佇列儲存體的差異
模擬器中的佇列儲存體沒有特定差異。

## <a name="storage-emulator-release-notes"></a>儲存體模擬器版本資訊

### <a name="version-57"></a>5.7 版
已修正啟用記錄時會造成當機的錯誤。

### <a name="version-56"></a>5.6 版
* 儲存體模擬器現在支援 Blob、佇列和資料表服務端點上 2018-03-28 版的儲存體服務。

### <a name="version-55"></a>版本 5.5
* 儲存體模擬器現在支援 Blob、佇列和資料表服務端點上 2017-11-09 版的儲存體服務。
* 已新增 Blob **Created** 屬性的支援，它會傳回 Blob 建立時間。

### <a name="version-54"></a>版本 5.4
若要改善安裝穩定性，模擬器不會嘗試在安裝時保留連接埠。 如果想要保留連接埠，可以使用 **init** 命令的 *-reserveports* 選項來指定連接埠。

### <a name="version-53"></a>版本 5.3
儲存體模擬器現在支援 Blob、佇列和資料表服務端點上 2017-07-29 版的儲存體服務。

### <a name="version-52"></a>5.2 版
* 儲存體模擬器現在支援 Blob、佇列和表格服務端點上 2017-04-17 版的儲存體服務。
* 已修正資料表屬性值未正確編碼的錯誤。

### <a name="version-51"></a>版本 5.1
修正了儲存體模擬器在某些回應中傳回 `DataServiceVersion` 標頭 (但服務未傳回) 的問題。

### <a name="version-50"></a>版本 5.0
* 儲存體模擬器安裝程式不再會檢查現有的 MSSQL 和 .NET Framework 安裝。
* 儲存體模擬器安裝程式不再會隨著安裝建立資料庫。 必要時仍會隨著啟動建立資料庫。
* 建立資料庫不再需要提高權限。
* 啟動不再需要保留連接埠。
* 將下列選項新增至 `init`：`-reserveports` (需要提高權限)、`-unreserveports` (需要提高權限)、`-skipcreate`。
* 系統匣圖示上的儲存體模擬器 UI 上選項現在會啟動命令列介面。 已不再使用舊 GUI。
* 某些 DLL 已移除或重新命名。

### <a name="version-46"></a>版本 4.6
* 儲存體模擬器現在支援 Blob、佇列和資料表服務端點上的 2016-05-31 版儲存體服務。

### <a name="version-45"></a>版本 4.5
* 已修正在重新命名備份資料庫時，會導致儲存體模擬器的初始化與安裝失敗的錯誤。

### <a name="version-44"></a>4.4 版
* 儲存體模擬器現在支援 Blob、佇列和資料表服務端點上 2015-12-11 版的儲存體服務。
* 現在，處理大量 blob 時，儲存體模擬器的 blob 資料記憶體回收更有效率。
* 修正了造成容器 ACL XML 的驗證方式與儲存體服務的運作方式稍有不同的錯誤。
* 修正了有時造成報告的日期時間上限和下限值時區不正確的錯誤。

### <a name="version-43"></a>版本 4.3
* 儲存體模擬器現在支援 Blob、佇列和表格服務端點上的 2015-07-08 版儲存體服務。

### <a name="version-42"></a>4.2 版
* 儲存體模擬器現在支援 Blob、佇列和資料表服務端點上 2015-04-05 版的儲存體服務。

### <a name="version-41"></a>4.1 版
* 除了新的附加 Blob 功能，儲存體模擬器現在支援 Blob、佇列和表格服務端點上的 2015-02-21 版儲存體服務。
* 如果模擬器尚不支援您使用的儲存體服務版本，該模擬器會傳回有意義的錯誤訊息。 我們建議使用最新版本的模擬器。 如果您遇到 VersionNotSupportedByEmulator 錯誤 (HTTP 狀態碼 400-不正確的要求)，請下載最新版的儲存體模擬器。
* 修正競爭情形造成資料表實體資料在並行合併作業期間會不正確的 bug。

### <a name="version-40"></a>4.0 版
* 儲存體模擬器的可執行檔已重新命名為 *AzureStorageEmulator.exe*。

### <a name="version-32"></a>3.2 版
* 儲存體模擬器現在支援 Blob、佇列和資料表服務端點上的 2014-02-14 版儲存體服務。 儲存體模擬器目前不支援檔案服務端點。 如需有關 2014-02-14 版本的詳細資訊，請參閱 [為 Microsoft Azure 中的 Blob、佇列和表格服務進行版本設定](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) 。

### <a name="version-31"></a>3.1 版
* 儲存體模擬器現在支援讀取權限異地備援儲存體 (RA-GRS)。 針對次要帳戶支援 Get Blob Service Stats、Get Queue Service Stats 和 Get Table Service Stats API，且這些 API 一律會根據基礎 SQL Database 傳回 LastSyncTime 回應元素的值當成目前的時間。 如需以程式設計方式使用儲存體模擬器來存取次要位置，請使用 Storage Client Library for.NET 3.2 版或更新版本。 如需詳細資訊，請參閱「Microsoft Azure Storage Client Library for .NET」。

### <a name="version-30"></a>3.0 版
* Azure 儲存體模擬器不再隨附於計算模擬器封裝。
* 儲存體模擬器圖形化使用者介面已由可編寫指令碼的命令列介面取代。 如需有關命令列介面的詳細資訊，請參閱儲存體模擬器命令列工具參考。 3.0 版仍將提供圖形化介面，除非在系統匣圖示按一下滑鼠右鍵並選取 [顯示儲存體模擬器 UI] 的方式安裝計算模擬器，否則無法存取圖形化介面。
* 現在完全支援 2013-08-15 版的 Azure 儲存體服務。 (先前只有儲存體模擬器 2.2.1 版預覽才支援此版本)。

## <a name="next-steps"></a>後續步驟

* 評估跨平台的社群維護開放原始碼儲存體模擬器 [Azurite](https://github.com/arafato/azurite)。 
* [使用 .NET 的 Azure 儲存體範例](../storage-samples-dotnet.md)包含開發應用程式時您可以使用的數個程式碼範例的連結。
* 您可以使用 [Microsoft Azure 儲存體總管](http://storageexplorer.com)，來使用雲端儲存體帳戶和儲存體模擬器中的資源。
