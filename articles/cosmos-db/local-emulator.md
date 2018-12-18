---
title: 使用 Azure Cosmos DB 模擬器在本機開發 | Microsoft Docs
description: 您也可以免費使用 Azure Cosmos DB 模擬器在本機開發及測試應用程式，不需建立 Azure 訂用帳戶。
services: cosmos-db
keywords: Azure Cosmos DB 模擬器
author: David-Noble-at-work
manager: kfile
editor: ''
ms.service: cosmos-db
ms.devlang: na
ms.topic: tutorial
ms.date: 04/20/2018
ms.author: danoble
ms.openlocfilehash: ce42d30b816599f7eaf90ce5a92164c6b85cfa36
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094168"
---
# <a name="use-the-azure-cosmos-db-emulator-for-local-development-and-testing"></a>使用 Azure Cosmos DB 模擬器進行本機開發和測試

<table>
<tr>
  <td><strong>二進位檔</strong></td>
  <td>[下載 MSI](https://aka.ms/cosmosdb-emulator)</td>
</tr>
<tr>
  <td><strong>Docker</strong></td>
  <td>[Docker Hub](https://hub.docker.com/r/microsoft/azure-cosmosdb-emulator/)</td>
</tr>
<tr>
  <td><strong>Docker 來源</strong></td>
  <td>[Github](https://github.com/Azure/azure-cosmos-db-emulator-docker)</td>
</tr>
</table>
  
Azure Cosmos DB 模擬器提供一個模擬 Azure Cosmos DB 服務的本機環境做為開發之用。 您可以使用 Azure Cosmos DB 模擬器在本機開發及測試應用程式，不需建立 Azure 訂用帳戶，也不會產生任何費用。 如果您滿意應用程式在 Azure Cosmos DB 模擬器中的運作方式，就可以切換成使用雲端的 Azure Cosmos DB 帳戶。 

模擬器中的資料總管目前只完整支援 SQL API 集合和 MongoDB 集合。 不會完整支援資料表、Graph 和 Cassandra 容器。 

本文涵蓋下列工作： 

> [!div class="checklist"]
> * 安裝模擬器
> * 驗證要求
> * 在模擬器中使用資料總管
> * 匯出 SSL 憑證
> * 從命令列呼叫模擬器
> * 在適用於 Windows 的 Docker 執行模擬器
> * 收集追蹤檔案
> * 疑難排解

## <a name="how-the-emulator-works"></a>模擬器的運作方式

Azure Cosmos DB 模擬器提供 Azure Cosmos DB 服務的高逼真度模擬。 它支援與 Azure Cosmos DB 完全相同的功能，包括支援建立和查詢 JSON 文件、佈建和擴充集合，以及執行預存程序和觸發程序。 您可以使用 Azure Cosmos DB 模擬器來開發及測試應用程式，並且只需對 Azure Cosmos DB 的連接端點進行單一組態變更，就能將應用程式部署到範圍遍及全球的 Azure。

雖然 Azure Cosmos DB 服務的模擬很可靠，但模擬器的實作會與服務有所不同。 例如，模擬器會使用標準的作業系統元件，比如使用本機檔案系統以獲得持續性，以及使用 HTTPS 通訊協定堆疊進行連線。 依賴 Azure 基礎結構的功能，例如全域複寫、讀取/寫入的個位數毫秒延遲，以及可調式的一致性層級等，都無法使用。

## <a name="differences-between-the-emulator-and-the-service"></a>模擬器和服務之間的差異 
因為 Azure Cosmos DB 模擬器是在本機開發人員工作站上提供一個執行的模擬環境，所以模擬器和雲端 Azure Cosmos DB 帳戶之間會有一些功能上的差異：

* 模擬器中的資料總管目前只支援 SQL API 集合和 MongoDB 集合。 尚未支援資料表、圖形和 Cassandra API。  
* Azure Cosmos DB 模擬器僅支援單一固定帳戶及已知的主要金鑰。  在 Azure Cosmos DB 模擬器中無法重新產生金鑰。
* Azure Cosmos DB 模擬器服務無法擴充，也不支援大量集合。
* Azure Cosmos DB 模擬器不會模擬不同的 [Azure Cosmos DB 一致性層級](consistency-levels.md)。
* Azure Cosmos DB 模擬器不會模擬[多重區域複寫](distribute-data-globally.md)。
* Azure Cosmos DB 模擬器不支援 Azure Cosmos DB 服務中可用的服務配額覆寫 (例如文件大小限制、增加的分割集合儲存體)。
* 因為 Azure Cosmos DB 服務最新的變更，您的 Azure Cosmos DB 模擬器複本可能不是最新狀態，請使用 [Azure Cosmos DB 容量規劃工具](https://www.documentdb.com/capacityplanner)，準確地評估應用程式的生產輸送量 (RU) 需求。

## <a name="system-requirements"></a>系統需求
Azure Cosmos DB 模擬器的硬體和軟體需求如下︰

* 軟體需求
  * Windows Server 2012 R2、Windows Server 2016 或 Windows 10
*   最低硬體需求
  * 2-GB RAM
  * 10-GB 可用硬碟空間

## <a name="installation"></a>安裝
您可以從 [Microsoft 下載中心](https://aka.ms/cosmosdb-emulator)下載並安裝 Azure Cosmos DB 模擬器，或執行 Emulator on Docker for Windows。 如需在適用於 Window 的 Docker 使用模擬器的指示，請參閱[在 Docker 上執行](#running-on-docker)。 

> [!NOTE]
> 若要安裝、設定和執行 Azure Cosmos DB 模擬器，您必須具備電腦的系統管理權限。

## <a name="running-on-windows"></a>在 Windows 上執行

若要啟動 Azure Cosmos DB 模擬器，請選取 [開始] 按鈕或按下 Windows 鍵。 先輸入 **Azure Cosmos DB 模擬器**，再從應用程式清單選取模擬器。 

![選取 [開始] 按鈕或按下 Windows 鍵，先輸入 **Azure Cosmos DB 模擬器**，再從應用程式清單選取模擬器](./media/local-emulator/database-local-emulator-start.png)

如果模擬器正在執行，您就會在 Windows 工作列通知區域看到圖示。 ![Azure Cosmos DB 本機模擬器的工作列通知](./media/local-emulator/database-local-emulator-taskbar.png)

Azure Cosmos DB 模擬器預設會在接聽連接埠 8081 的本機電腦 ("localhost") 上執行。

Azure Cosmos DB 模擬器預設會安裝到 `C:\Program Files\Azure Cosmos DB Emulator`。 您也可以從命令列啟動和停止模擬器。 如需詳細資訊，請參閱[命令列工具參考](#command-line)。

## <a name="start-data-explorer"></a>啟動資料總管

當 Azure Cosmos DB 模擬器啟動時，會自動在瀏覽器中開啟 Azure Cosmos DB 資料總管。 位址會顯示為 [https://localhost:8081/_explorer/index.html](https://localhost:8081/_explorer/index.html)。 如果您關閉資料總管，想要稍後重新開啟，可以在瀏覽器中開啟 URL 或從 Windows 系統匣圖示中的 Azure Cosmos DB 模擬器啟動，如下所示。

![Azure Cosmos DB 本機模擬器的資料總管啟動程式](./media/local-emulator/database-local-emulator-data-explorer-launcher.png)

## <a name="checking-for-updates"></a>檢查更新
資料總管會表示是否有新的更新可供下載。 

> [!NOTE]
> 在某個 Azure Cosmos DB 模擬器版本中建立的資料不保證可以使用不同的版本存取。 如果您需要長期保存資料，建議您將資料儲存於 Azure Cosmos DB 帳戶中，而不是 Azure Cosmos DB 模擬器中。 

## <a name="authenticating-requests"></a>驗證要求
就像雲端的 Azure Cosmos DB 一樣，您對 Azure Cosmos DB 模擬器的每個要求都必須經過驗證。 Azure Cosmos DB 模擬器支援主要金鑰驗證的單一固定帳戶及已知驗證金鑰。 此帳戶和金鑰都是唯一允許搭配 Azure Cosmos DB 模擬器使用的認證， 如下：

    Account name: localhost:<port>
    Account key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==

> [!NOTE]
> Azure Cosmos DB 模擬器所支援的主要金鑰僅能與模擬器搭配使用。 您無法將生產用的 Azure Cosmos DB 帳戶和金鑰與 Azure Cosmos DB 模擬器搭配使用。 

> [!NOTE] 
> 如果您使用 /Key 選項啟動模擬器，則請使用產生的金鑰，而不要使用 "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="

就像 Azure Cosmos DB 服務一樣，Azure Cosmos DB 模擬器僅支援透過 SSL 的安全通訊。

## <a name="running-on-a-local-network"></a>在區域網路上執行

您可以在本機網路上執行模擬器。 若要啟用網路存取，請在[命令列](#command-line-syntax)指定 /AllowNetworkAccess 選項，這也需要您指定 /Key=key_string 或 /KeyFile=file_name。 您可以使用 /GenKeyFile=file_name 來產生具有預先隨機金鑰的檔案。  然後您可以將其傳遞至 /KeyFile=file_name 或 /Key=contents_of_file。

若是第一次啟用網路存取，使用者應該關閉模擬器，並且刪除模擬器的資料目錄 (C:\Users\user_name\AppData\Local\CosmosDBEmulator)。

## <a name="developing-with-the-emulator"></a>使用模擬器進行開發
在桌面上執行 Azure Cosmos DB 模擬器之後，就可以使用任何支援的 [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) 或 [Azure Cosmos DB REST API](/rest/api/cosmos-db/) 與模擬器互動。 Azure Cosmos DB 模擬器也包含內建的資料總管，可讓您建立 SQL 和 MongoDB API 集合、檢視及編輯文件，而不需要撰寫任何程式碼。   

    // Connect to the Azure Cosmos DB Emulator running locally
    DocumentClient client = new DocumentClient(
        new Uri("https://localhost:8081"), 
        "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==");

如果您使用 [MongoDB 的 Azure Cosmos DB 通訊協定支援](mongodb-introduction.md)，請使用下列連接字串︰

    mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true

您可以使用現有的工具，像是 [Azure DocumentDB Studio](https://github.com/mingaliu/DocumentDBStudio) 連線到 Azure Cosmos DB 模擬器。 您也可以使用 [Azure Cosmos DB 資料移轉工具](https://github.com/azure/azure-documentdb-datamigrationtool)在 Azure Cosmos DB 模擬器與 Azure Cosmos DB 服務之間移轉資料。

> [!NOTE] 
> 如果您使用 /Key 選項啟動模擬器，則請使用產生的金鑰，而不要使用 "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="

根據預設，您可以使用 Azure Cosmos DB 模擬器建立最多 25 個單一分割區的集合，或 1 個分割集合。 如需變更此值的詳細資訊，請參閱[設定 PartitionCount 值](#set-partitioncount)。

## <a name="export-the-ssl-certificate"></a>匯出 SSL 憑證

.NET 語言和執行階段使用 Windows 憑證存放區來安全地連線到 Azure Cosmos DB 本機模擬器。 其他語言則有自己管理和使用憑證的方法。 Java 使用自己的[憑證存放區](https://docs.oracle.com/cd/E19830-01/819-4712/ablqw/index.html)，而 Python 使用[通訊端包裝函式](https://docs.python.org/2/library/ssl.html)。

若要取得憑證來搭配未整合 Windows 憑證存放區的語言和執行階段使用，您必須使用 Windows 憑證管理員將它匯出。 您可以執行 certlm.msc 或依照[匯出 Azure Cosmos DB 模擬器憑證](./local-emulator-export-ssl-certificates.md)中的逐步指示開始。 憑證管理員執行之後時，開啟 [個人憑證] \(如下所示)，並將有 "DocumentDBEmulatorCertificate" 易記名稱的憑證匯出為 BASE-64 編碼的 X.509 (.cer) 檔案。

![Azure Cosmos DB 本機模擬器的 SSL 憑證](./media/local-emulator/database-local-emulator-ssl_certificate.png)

遵循[新增憑證至 Java CA 憑證存放區](https://docs.microsoft.com/azure/java-add-certificate-ca-store)中的指示，X.509 憑證就可以匯入 Java 憑證存放區。 將憑證匯入憑證存放區之後，Java 和 MongoDB 應用程式就可以連線到 Azure Cosmos DB 模擬器。

從 Python 和 Node.js SDK 連接到模擬器時，會停用 SSL 驗證。

## <a id="command-line"></a>命令列工具參考
您可以從安裝位置使用命令列來啟動和停止模擬器、設定選項，以及執行其他作業。

### <a name="command-line-syntax"></a>命令列語法

    CosmosDB.Emulator.exe [/Shutdown] [/DataPath] [/Port] [/MongoPort] [/DirectPorts] [/Key] [/EnableRateLimiting] [/DisableRateLimiting] [/NoUI] [/NoExplorer] [/?]

若要檢視選項清單，請在命令提示字元輸入 `CosmosDB.Emulator.exe /?` 。

<table>
<tr>
  <td><strong>選項</strong></td>
  <td><strong>說明</strong></td>
  <td><strong>命令</strong></td>
  <td><strong>引數</strong></td>
</tr>
<tr>
  <td>[無引數]</td>
  <td>使用預設設定啟動 Azure Cosmos DB 模擬器。</td>
  <td>CosmosDB.Emulator.exe</td>
  <td></td>
</tr>
<tr>
  <td>[說明]</td>
  <td>顯示支援的命令列引數清單。</td>
  <td>CosmosDB.Emulator.exe /?</td>
  <td></td>
</tr>
<tr>
  <td>GetStatus</td>
  <td>取得 Azure Cosmos DB 模擬器的狀態。 狀態會以結束代碼表示：1 = 啟動，2 = 執行中，3 = 已停止。 負數的結束代碼表示發生錯誤。 不會產生其他輸出。</td>
  <td>CosmosDB.Emulator.exe /GetStatus</td>
  <td></td>
<tr>
  <td>Shutdown</td>
  <td>關閉 Azure Cosmos DB 模擬器。</td>
  <td>CosmosDB.Emulator.exe /Shutdown</td>
  <td></td>
</tr>
<tr>
  <td>資料路徑</td>
  <td>指定用來儲存資料檔案的路徑。 預設值為 %LocalAppdata%\CosmosDBEmulator。</td>
  <td>CosmosDB.Emulator.exe /DataPath=&lt;datapath&gt;</td>
  <td>&lt;datapath&gt;︰可存取的路徑</td>
</tr>
<tr>
  <td>Port</td>
  <td>指定用於模擬器的連接埠號碼。  預設值為 8081。</td>
  <td>CosmosDB.Emulator.exe /Port=&lt;port&gt;</td>
  <td>&lt;port&gt;︰單一連接埠號碼</td>
</tr>
<tr>
  <td>MongoPort</td>
  <td>指定要用於 MongoDB 相容性 API 的連接埠號碼。 預設值為 10255。</td>
  <td>CosmosDB.Emulator.exe /MongoPort=&lt;mongoport&gt;</td>
  <td>&lt;mongoport&gt;︰單一連接埠號碼</td>
</tr>
<tr>
  <td>DirectPorts</td>
  <td>指定要用於直接連線的連接埠。 預設值為 10251、10252、10253、10254。</td>
  <td>CosmosDB.Emulator.exe /DirectPorts:&lt;directports&gt;</td>
  <td>&lt;directports&gt;︰以逗號分隔的 4 個連接埠清單</td>
</tr>
<tr>
  <td>Key</td>
  <td>模擬器的授權金鑰。 金鑰必須是 64 位元組向量的 base-64 編碼方式。</td>
  <td>CosmosDB.Emulator.exe /Key:&lt;key&gt;</td>
  <td>&lt;key&gt;：金鑰必須是 64 位元組向量的 base-64 編碼方式</td>
</tr>
<tr>
  <td>EnableRateLimiting</td>
  <td>指定要啟用要求率限制行為。</td>
  <td>CosmosDB.Emulator.exe /EnableRateLimiting</td>
  <td></td>
</tr>
<tr>
  <td>DisableRateLimiting</td>
  <td>指定要停用要求率限制行為。</td>
  <td>CosmosDB.Emulator.exe /DisableRateLimiting</td>
  <td></td>
</tr>
<tr>
  <td>NoUI</td>
  <td>不要顯示模擬器使用者介面。</td>
  <td>CosmosDB.Emulator.exe /NoUI</td>
  <td></td>
</tr>
<tr>
  <td>NoExplorer</td>
  <td>啟動時不要顯示資料總管。</td>
  <td>CosmosDB.Emulator.exe /NoExplorer</td>
  <td></td>
</tr>
<tr>
  <td>PartitionCount</td>
  <td>指定已分割集合的數目上限。 如需詳細資訊，請參閱[變更集合的數目](#set-partitioncount)。</td>
  <td>CosmosDB.Emulator.exe /PartitionCount=&lt;partitioncount&gt;</td>
  <td>&lt;partitioncount&gt;：允許的單一分割區集合數目上限。 預設值為 25。 允許的上限為 250。</td>
</tr>
<tr>
  <td>DefaultPartitionCount</td>
  <td>指定分割集合之分割數目的預設值。</td>
  <td>CosmosDB.Emulator.exe /DefaultPartitionCount=&lt;defaultpartitioncount&gt;</td>
  <td>&lt;defaultpartitioncount&gt; 預設值為 25。</td>
</tr>
<tr>
  <td>AllowNetworkAccess</td>
  <td>讓模擬器可以透過網路存取。 您也必須傳遞 /Key=&lt;key_string&gt; 或 /KeyFile=&lt;file_name&gt; 以啟用網路存取。</td>
  <td>CosmosDB.Emulator.exe /AllowNetworkAccess /Key=&lt;key_string&gt;<br><br>或<br><br>CosmosDB.Emulator.exe /AllowNetworkAccess /KeyFile=&lt;file_name&gt;</td>
  <td></td>
</tr>
<tr>
  <td>NoFirewall</td>
  <td>當 /AllowNetworkAccess 在使用中時請勿調整防火牆規則。</td>
  <td>CosmosDB.Emulator.exe /NoFirewall</td>
  <td></td>
</tr>
<tr>
  <td>GenKeyFile</td>
  <td>產生新的授權金鑰並儲存到指定的檔案。 產生的金鑰可以搭配 /Key 或 /KeyFile 選項使用。</td>
  <td>CosmosDB.Emulator.exe  /GenKeyFile=&lt;金鑰檔案路徑&gt;</td>
  <td></td>
</tr>
<tr>
  <td>一致性</td>
  <td>設定帳戶的預設一致性層級。</td>
  <td>CosmosDB.Emulator.exe /Consistency=&lt;consistency&gt;</td>
  <td>&lt;一致性&gt;：值必須是下列[一致性層級](consistency-levels.md)其中之一：Session、Strong、Eventual 或 BoundedStaleness。  預設值為 Session。</td>
</tr>
<tr>
  <td>?</td>
  <td>顯示說明訊息。</td>
  <td></td>
  <td></td>
</tr>
</table>

## <a id="set-partitioncount"></a>變更集合的數目

根據預設，您可以使用 Azure Cosmos DB 模擬器建立最多 25 個單一分割區的集合，或 1 個分割集合。 藉由修改 **PartitionCount** 值，您可以建立最多 250 個單一分割區集合或是 10 個已分割的集合，或是這兩種的任何組合 (不超過 250 個單一分割區，其中 1 個已分割集合 = 25 個單一分割區集合)。

如果您在目前的分割計數超過限制後嘗試建立集合，模擬器會擲回 ServiceUnavailable 例外狀況，並隨附下列訊息。

    Sorry, we are currently experiencing high demand in this region, 
    and cannot fulfill your request at this time. We work continuously 
    to bring more and more capacity online, and encourage you to try again. 
    Please do not hesitate to email askcosmosdb@microsoft.com at any time or
    for any reason. ActivityId: 29da65cc-fba1-45f9-b82c-bf01d78a1f91

若要變更 Azure Cosmos DB 模擬器可用的集合數目，請執行下列動作：

1. 刪除所有的本機 Azure Cosmos DB 模擬器資料，方法是以滑鼠右鍵按一下系統匣上的 [Azure Cosmos DB 模擬器] 圖示，然後按一下 [重設資料...]。
2. 刪除以下資料夾中所有的模擬器資料：C:\Users\user_name\AppData\Local\CosmosDBEmulator。
3. 結束所有開啟的執行個體，方法是以滑鼠右鍵按一下系統匣上的 [Azure Cosmos DB 模擬器] 圖示，然後按一下 [結束]。 結束所有執行個體可能需要數分鐘的時間。
4. 安裝最新版的 [Azure Cosmos DB Emulator 模擬器](https://aka.ms/cosmosdb-emulator)。
5. 啟動具有 PartitionCount 旗標的模擬器，方法是設定值 <= 250。 例如： `C:\Program Files\Azure Cosmos DB Emulator> CosmosDB.Emulator.exe /PartitionCount=100` 。

## <a name="controlling-the-emulator"></a>控制模擬器

模擬器隨附 PowerShell 模組，可用於執行服務的啟動、停止、解除安裝和狀態擷取。 使用方式：

```powershell
Import-Module "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules\Microsoft.Azure.CosmosDB.Emulator"
```

或將 `PSModules` 目錄置於 `PSModulesPath` 上，並匯入如下：

```powershell
$env:PSModulesPath += "$env:ProgramFiles\Azure Cosmos DB Emulator\PSModules"
Import-Module Microsoft.Azure.CosmosDB.Emulator
```

以下是從 PowerShell 控制模擬器的命令摘要：

### `Get-CosmosDbEmulatorStatus`

#### <a name="syntax"></a>語法

`Get-CosmosDbEmulatorStatus`

#### <a name="remarks"></a>備註

傳回這些 ServiceControllerStatus 值的其中一個：ServiceControllerStatus.StartPending、ServiceControllerStatus.Running 或 ServiceControllerStatus.Stopped。

### `Start-CosmosDbEmulator`

#### <a name="syntax"></a>語法

`Start-CosmosDbEmulator [-DataPath <string>] [-DefaultPartitionCount <uint16>] [-DirectPort <uint16[]>] [-MongoPort <uint16>] [-NoUI] [-NoWait] [-PartitionCount <uint16>] [-Port <uint16>]  [<CommonParameters>]`

#### <a name="remarks"></a>備註

啟動模擬器。 根據預設，命令會等到模擬器準備好接受要求。 如果您希望啟動模擬器時立即傳回 Cmdlet，請使用 -NoWait 選項。

### `Stop-CosmosDbEmulator`

#### <a name="syntax"></a>語法

 `Stop-CosmosDbEmulator [-NoWait]`

#### <a name="remarks"></a>備註

停止模擬器。 根據預設，此命令會等到模擬器完全關機。 如果您希望模擬器開始關閉時立即傳回 Cmdlet，請使用 -NoWait 選項。

### `Uninstall-CosmosDbEmulator`

#### <a name="syntax"></a>語法

`Uninstall-CosmosDbEmulator [-RemoveData]`

#### <a name="remarks"></a>備註

解除安裝模擬器，並選擇性地移除 $env 的完整內容：LOCALAPPDATA\CosmosDbEmulator。
Cmdlet 可確保在解除安裝之前停止模擬器。

## <a name="running-on-docker"></a>在 Docker 上執行

Azure Cosmos DB 模擬器可以在 Docker for Windows 上執行。 模擬器無法在 Docker for Oracle Linux 上運作。

安裝 [Docker for Windows](https://www.docker.com/docker-windows) 後，以滑鼠右鍵按一下工具列上的 Docker 圖示，然後選取 [切換到 Windows 容器]，即可切換到 Windows 容器。

接下來，從最喜愛的殼層執行下列命令，即可從 Docker Hub 提取模擬器映像。

```     
docker pull microsoft/azure-cosmosdb-emulator 
```
若要啟動映像，請執行下列命令。

從命令列執行：
```cmd 
md %LOCALAPPDATA%\CosmosDBEmulatorCert 2>null
docker run -v %LOCALAPPDATA%\CosmosDBEmulatorCert:C:\CosmosDB.Emulator\CosmosDBEmulatorCert -P -t -i -m 2GB microsoft/azure-cosmosdb-emulator 
```

從 PowerShell：
```powershell
md $env:LOCALAPPDATA\CosmosDBEmulatorCert 2>null
docker run -v $env:LOCALAPPDATA\CosmosDBEmulatorCert:C:\CosmosDB.Emulator\CosmosDBEmulatorCert -P -t -i -m 2GB microsoft/azure-cosmosdb-emulator 
```

回應如下所示：

```
Starting emulator
Emulator Endpoint: https://172.20.229.193:8081/
Master Key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
Exporting SSL Certificate
You can import the SSL certificate from an administrator command prompt on the host by running:
cd /d %LOCALAPPDATA%\CosmosDBEmulatorCert
powershell .\importcert.ps1
--------------------------------------------------------------------------------------------------
Starting interactive shell
``` 

現在，在您的用戶端使用回應中的端點和主要金鑰，並將 SSL 憑證匯入您的主機。 若要匯入 SSL 憑證，請從系統管理員命令提示字元中執行下列命令︰

從命令列執行：
```cmd 
cd %LOCALAPPDATA%\CosmosDBEmulatorCert
powershell .\importcert.ps1
```

從 PowerShell：
```powershell
cd $env:LOCALAPPDATA\CosmosDBEmulatorCert
.\importcert.ps1
```

模擬器啟動之後，關閉互動式殼層將會關閉模擬器的容器。

若要開啟資料總管，請在瀏覽器中瀏覽至下列 URL。 如上所示，回應訊息中提供模擬器端點。

    https://<emulator endpoint provided in response>/_explorer/index.html


## <a name="troubleshooting"></a>疑難排解

使用下列秘訣可協助您對使用 Azure Cosmos DB 模擬器時遇到的問題進行疑難排解︰

- 如果您已安裝新版本的模擬器但發生了錯誤，請務必重設您的資料。 您可以在系統匣上的 [Azure Cosmos DB 模擬器] 圖示上按一下滑鼠右鍵，然後按一下 [重設資料...]，以重設資料。 如果無法修正錯誤，您可以解除安裝，然後重新安裝應用程式。 如需相關指示，請參閱[解除安裝本機模擬器](#uninstall)。

- 如果 Azure Cosmos DB 模擬器當機，請從 c:\Users\user_name\AppData\Local\CrashDumps 資料夾中收集傾印檔案，壓縮檔案後附加到電子郵件，然後寄到 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com)。

- 如果 CosmosDB.StartupEntryPoint.exe 發生損毀，請從系統管理員命令提示字元執行下列命令︰`lodctr /R` 

- 如果您遇到連線問題，請[收集追蹤檔案](#trace-files)，壓縮檔案後，再附加到電子郵件寄至[askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com)。

- 如果您收到**服務無法使用**訊息，模擬器可能無法初始化網路堆疊。 由於 Pulse 安全用戶端或 Juniper 網路用戶端的網路篩選驅動程式可能會造成問題，因此請檢查是否已安裝這些驅動程式。 解除安裝協力廠商網路篩選驅動程式通常便會修正問題。

- 當模擬器執行時，如果您的電腦進入睡眠模式或執行任何作業系統更新，您應該會看見**服務目前無法使用**的訊息。 請重設模擬器，方法是以滑鼠右鍵按一下出現在視窗通知匣的圖示，然後選取 [重設資料]。

### <a id="trace-files"></a>收集追蹤檔案

若要收集偵錯追蹤，請從系統管理命令提示字元執行下列命令︰

1. `cd /d "%ProgramFiles%\Azure Cosmos DB Emulator"`
2. `CosmosDB.Emulator.exe /shutdown` 。 監看系統匣，確認程式已經關閉，這可能需要一分鐘的時間。 您也可以直接按一下 Azure Cosmos DB 模擬器使用者介面中的 [結束]。
3. `CosmosDB.Emulator.exe /starttraces`
4. `CosmosDB.Emulator.exe`
5. 重現問題。 如果資料總管無法運作，您只需要等候數秒鐘的時間，等到瀏覽器開啟即可攔截錯誤。
5. `CosmosDB.Emulator.exe /stoptraces`
6. 瀏覽至 `%ProgramFiles%\Azure Cosmos DB Emulator` 並尋找 docdbemulator_000001.etl 檔案。
7. 將 .etl 檔案以及重現步驟傳送至 [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com) 進行偵錯。

### <a id="uninstall"></a>將本機模擬器解除安裝

1. 結束本機模擬器所有已開啟的執行個體，方法是以滑鼠右鍵按一下系統匣上的 Azure Cosmos DB 模擬器圖示，然後按一下 [結束]。 結束所有執行個體可能需要數分鐘的時間。
2. 在 Windows 搜尋方塊中，輸入 **App 與功能**，然後按一下 [App 與功能 (系統設定)] 結果。
3. 在應用程式清單中，捲動至 [Azure Cosmos DB 模擬器] 並將其選取，按一下 [解除安裝]，然後確認並再按一下 [解除安裝]。
4. 當應用程式已解除安裝時，請瀏覽至 `C:\Users\<user>\AppData\Local\CosmosDBEmulator` 然後刪除該資料夾。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列操作：

> [!div class="checklist"]
> * 已安裝本機模擬器
> * 已在適用於 Windows 的 Docker 上執行模擬器
> * 已驗證要求
> * 已在模擬器中使用資料總管
> * 已匯出 SSL 憑證
> * 已從命令列呼叫模擬器
> * 已收集追蹤檔案

在本教學課程中，您學習到如何使用免費的本機模擬器在本機開發。 現在您可以繼續進行下一個教學課程，了解如何匯出模擬器 SSL 憑證。 

> [!div class="nextstepaction"]
> [匯出 Azure Cosmos DB 模擬器憑證](local-emulator-export-ssl-certificates.md)
