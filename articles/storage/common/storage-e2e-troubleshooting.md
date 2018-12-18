---
title: 使用診斷與 Message Analyzer 進行 Azure 儲存體的疑難排解 | Microsoft Docs
description: 示範使用 Azure Storage Analytics、AzCopy 和 Microsoft Message Analyzer 進行端對端疑難排解的教學課程
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 03/15/2017
ms.author: tamram
ms.component: common
ms.openlocfilehash: 41e7f5b4c36ad0bfed0ef5a9a31565474cf4d823
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038013"
---
# <a name="end-to-end-troubleshooting-using-azure-storage-metrics-and-logging-azcopy-and-message-analyzer"></a>使用 Azure 儲存體計量和記錄、AzCopy 和 Message Analyzer 進行端對端疑難排解
[!INCLUDE [storage-selector-portal-e2e-troubleshooting](../../../includes/storage-selector-portal-e2e-troubleshooting.md)]

診斷與疑難排解是透過 Microsoft Azure 儲存體建置和支援用戶端應用程式的關鍵技術。 由於 Azure 應用程式的分散式本質，診斷和疑難排解錯誤和效能問題可能會比在傳統環境中更為複雜。

在本教學課程中，我們將示範如何使用 Microsoft 提供的工具和 Azure 儲存體，識別可能會影響效能的錯誤，以及針對這些錯誤進行疑難排解的完整步驟，以便最佳化用戶端應用程式。

本教學課程提供端對端疑難排解案例的實際操作探勘。 如需針對 Azure 儲存體應用程式進行疑難排解的深度概念性指南，請參閱 [監控、診斷和疑難排解 Microsoft Azure 儲存體](storage-monitoring-diagnosing-troubleshooting.md)。

## <a name="tools-for-troubleshooting-azure-storage-applications"></a>對 Azure 儲存體應用程式進行疑難排解的工具
若要使用 Microsoft Azure 儲存體進行用戶端應用程式的疑難排解，您可以使用一些工具組合來判斷何時發生問題和問題的可能原因。 這些工具包括：

* **Azure Storage Analytics**。 [Azure Storage Analytics](/rest/api/storageservices/Storage-Analytics) 提供 Azure 儲存體的度量和記錄。
  
  * **儲存體度量** 可追蹤儲存體帳戶的交易度量和容量度量。 您可以使用度量，判斷如何根據各種不同的量值來執行您的應用程式。 如需 Storage Analytics 追蹤的度量類型詳細資訊，請參閱 [Storage Analytics 度量資料表結構描述](/rest/api/storageservices/Storage-Analytics-Metrics-Table-Schema) 。
  * **儲存體記錄** 可將對 Azure 儲存體服務的每項要求記錄至伺服器端記錄檔。 記錄檔可追蹤每項要求的詳細資料，包括執行的作業、作業的狀態及延遲資訊。 如需 Storage Analytics 寫入記錄檔的要求和回應資料詳細資訊，請參閱 [Storage Analytics 記錄檔格式](/rest/api/storageservices/Storage-Analytics-Log-Format) 。

* **Azure 入口網站**。 您可以在 [Azure 入口網站](https://portal.azure.com)中設定儲存體帳戶的計量和記錄。 您也可以檢視圖表和圖形 (其中顯示應用程式在一段時間內的執行狀況)，並設定警示，通知應用程式是否對指定的度量以超乎預期的方式執行。
  
    如需在 Azure 入口網站中設定監視的相關資訊，請參閱[在 Azure 入口網站中監視儲存體帳戶](storage-monitor-storage-account.md)。
* **AzCopy**。 Azure 儲存體的伺服器記錄檔會儲存為 Blob，因此您可以使用 AzCopy，將記錄檔 Blob 複製到本機目錄，以便使用 Microsoft Message Analyzer 分析。 如需 AzCopy 的詳細資訊，請參閱[使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)。
* **Microsoft Message Analyzer**。 Message Analyzer 是一種可取用記錄檔並以視覺化格式顯示記錄檔資料的工具，讓您輕鬆地篩選、搜尋記錄檔資料並分組為實用的資料集，以便用來分析錯誤和效能問題。 如需 Message Analyzer 的詳細資訊，請參閱 [Microsoft Message Analyzer 操作指南](http://technet.microsoft.com/library/jj649776.aspx)。

## <a name="about-the-sample-scenario"></a>關於範例案例
在本教學課程中，我們將檢驗一個案例，其中的 Azure 儲存體度量表示應用程式呼叫 Azure 儲存體的成功率很低。 低百分比成功速率計量 (在 [Azure 入口網站](https://portal.azure.com)和計量資料表中顯示為 **PercentSuccess**) 會追蹤成功的作業，但是會傳回大於 299 的 HTTP 狀態碼。 在伺服器端的儲存體記錄檔中，這些作業會加上 **ClientOtherErrors**的交易狀態記錄下來。 如需低成功百分比度量的詳細資訊，請參閱 [度量顯示低 PercentSuccess 或分析記錄檔項目的交易狀態為 ClientOtherErrors](storage-monitoring-diagnosing-troubleshooting.md#metrics-show-low-percent-success)。

Azure 儲存體作業可能會傳回大於 299 的 HTTP 狀態碼為其正常功能的一部分。 但是在某些情況下，這些錯誤表示您可能可以最佳化您的用戶端應用程式，以改善效能。

在此案例中，我們會將低百分比成功率視為任何低於 100%的項目。 不過，您可以根據自己的需求，選擇不同的度量層級。 我們建議您在應用程式的測試期間，建立關鍵效能指標的基準容錯。 例如，您可能會根據測試，決定您的應用程式應具有 90% 或 85% 的穩定成功率。 如果度量資料顯示應用程式偏離該數字，您可以調查造成增加的可能原因。

在我們的範例案例中，一旦建立了低於 100% 的百分比成功率度量，我們就會檢查記錄檔，以找出度量相關的錯誤，並利用它們來找出造成較低百分比成功率的原因。 我們會特別關注 400 範圍中的錯誤。 然後我們將更仔細調查 404 (找不到) 錯誤。

### <a name="some-causes-of-400-range-errors"></a>400 範圍錯誤的某些原因
下列範例顯示 Azure Blob 儲存體要求的某些 400 範圍錯誤的取樣，以及其可能原因。 任何這些錯誤，以及 300 範圍和 500 範圍中的錯誤，都可能導致低百分比成功率。

請注意，下面所列並不完整。 如需一般 Azure 儲存體錯誤和每項儲存體服務特定錯誤的詳細資料，請參閱 MSDN 上的 [狀態和錯誤碼](http://msdn.microsoft.com/library/azure/dd179382.aspx) 。

**狀態碼 404 (找不到) 範例**

發生於因為找不到 Blob 或容器而使對容器或 Blob 的讀取作業失敗時。

* 如果在其他用戶端此要求前刪除了容器或 Blob，就會發生。
* 如果您在檢查容器或 Blob 是否存在之後，使用可建立容器或 Blob 的 API 呼叫，就會發生。 CreateIfNotExists API 會先進行 HEAD 呼叫，以檢查容器或 Blob 是否存在；如果不存在，寔會傳回 404 錯誤，然後進行第二次 PUT 呼叫，以寫入容器或 Blob。

**狀態碼 409 (衝突) 範例**

* 如果您使用建立 API 來建立新的容器或 Blob，但未先檢查其存在與否，而且已存在具有該名稱的容器或 Blob，就會發生。
* 如果正在刪除容器，而您嘗試在刪除作業完成之前，使用相同名稱建立新的容器，就會發生。
* 如果您在容器或 Blob 上指定租用，而且已經有租用存在，就會發生。

**狀態碼 412 (先決條件失敗) 範例**

* 發生於不符合條件式標頭指定的條件時。
* 發生於指定的租用識別碼與容器或 Blob 上的租用識別碼不相符時。

## <a name="generate-log-files-for-analysis"></a>產生記錄檔以供分析
在本教學課程中，我們將使用 Message Analyzer 來處理三種不同類型的記錄檔，雖然您可以選擇使用任何其中一種：

* 當您啟用 Azure 儲存體記錄時建立的 **伺服器記錄檔**。 伺服器記錄檔包含對其中一項 Azure 儲存體服務 (Blob、佇列、資料表和檔案) 呼叫的每項作業的相關資料。 伺服器記錄檔指出已呼叫的作業和傳回的狀態碼，以及有關要求和回應的其他詳細資料。
* 當您在 .NET 應用程式中啟用用戶端記錄時建立的 **.NET 用戶端記錄檔**。 用戶端記錄檔包含用戶端如何準備要求及如何接收和處理回應的詳細資訊。
* **HTTP 網路追蹤記錄檔**，用以收集 HTTP/HTTPS 要求和回應資料，包括對 Azure 儲存體的作業。 在本教學課程中，我們將透過 Message Analyzer 產生網路追蹤。

### <a name="configure-server-side-logging-and-metrics"></a>設定伺服器端記錄和度量
首先，我們必須設定 Azure 儲存體記錄和度量，如此我們才有用戶端應用程式提供的資料可分析。 您可以用各種方式設定記錄和計量 - 透過 [Azure 入口網站](https://portal.azure.com)，或使用 PowerShell，或以程式設計的方式。 如需設定記錄和度量的詳細資訊，請參閱 MSDN 上的[啟用儲存體度量和檢視度量資料](http://msdn.microsoft.com/library/azure/dn782843.aspx)和[啟用儲存體記錄和存取記錄檔資料](http://msdn.microsoft.com/library/azure/dn782840.aspx)。

**透過 Azure 入口網站**

若要使用 [Azure 入口網站](https://portal.azure.com)設定儲存體帳戶的記錄和計量，請遵循[在 Azure 入口網站中監視儲存體帳戶](storage-monitor-storage-account.md)上的指示執行。

> [!NOTE]
> 使用 Azure 入口網站無法設定分鐘計量。 不過，我們建議您不要為了本教學課程的目的，以及為了調查您的應用程式的效能問題，而設定這類度量。 您可以使用 PowerShell (如下所示)，或以程式設計方式使用儲存體用戶端程式庫，來設定分鐘度量。
> 
> 請注意，Azure 入口網站無法顯示分鐘計量，只能顯示每小時計量。
> 
> 

**透過 PowerShell**

若要開始使用 Azure PowerShell，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。

1. 使用 [Add-AzureAccount](/powershell/module/servicemanagement/azure/add-azureaccount?view=azuresmps-3.7.0) Cmdlet 將您的 Azure 使用者帳戶新增至 PowerShell 視窗：
   
    ```powershell
    Add-AzureAccount
    ```

2. 在 [登入 Microsoft Azure]  視窗中，輸入電子郵件地址以及與您的帳戶相關聯的密碼。 Azure 會驗證並儲存認證資訊，然後關閉視窗。
3. 在 PowerShell 視窗中執行下列命令，將預設儲存體帳戶設定為您在本教學課程中使用的儲存體帳戶：
   
    ```powershell
    $SubscriptionName = 'Your subscription name'
    $StorageAccountName = 'yourstorageaccount'
    Set-AzureSubscription -CurrentStorageAccountName $StorageAccountName -SubscriptionName $SubscriptionName
    ```

4. 啟用 Blob 服務的儲存體記錄：
   
    ```powershell
    Set-AzureStorageServiceLoggingProperty -ServiceType Blob -LoggingOperations Read,Write,Delete -PassThru -RetentionDays 7 -Version 1.0
    ```

5. 啟用 Blob 服務的儲存體度量，並確定將 -**-MetricsType`Minute` 設定為** ：
   
    ```powershell
    Set-AzureStorageServiceMetricsProperty -ServiceType Blob -MetricsType Minute -MetricsLevel ServiceAndApi -PassThru -RetentionDays 7 -Version 1.0
    ```

### <a name="configure-net-client-side-logging"></a>設定 .NET 用戶端記錄
若要設定 .NET 應用程式的用戶端的記錄，請在應用程式的組態檔 (web.config 或 app.config) 中啟用 .NET 診斷。 如需詳細資訊，請參閱 MSDN 上的[使用 .NET Storage Client Library 的用戶端記錄](http://msdn.microsoft.com/library/azure/dn782839.aspx)和[使用 Microsoft Azure Storage SDK for Java 的用戶端記錄](http://msdn.microsoft.com/library/azure/dn782844.aspx)。

用戶端記錄檔包含用戶端如何準備要求及如何接收和處理回應的詳細資訊。

儲存體用戶端程式庫會將用戶端記錄檔資料儲存在應用程式的組態檔 (web.config 或 app.config) 中指定的位置。

### <a name="collect-a-network-trace"></a>收集網路追蹤
您可以使用 Message Analyzer，在用戶端應用程式執行時收集 HTTP/HTTPS 網路追蹤。 Message Analyzer 會在後端使用 [Fiddler](http://www.telerik.com/fiddler) 。 收集網路追蹤之前，我們建議您設定 Fiddler 以記錄未加密的 HTTPS 流量：

1. 安裝 [Fiddler](http://www.telerik.com/download/fiddler)。
2. 啟動 Fiddler。
3. 選取 [工具] | [Fiddler 選項]。
4. 在 [選項] 對話方塊中，確定 [擷取 HTTPS CONNECT] 和 [解密 HTTPS 流量] 都已選取，如下所示。

![設定 Fiddler 選項](./media/storage-e2e-troubleshooting/fiddler-options-1.png)

在本教學課程中，先在 Message Analyzer 中收集並儲存網路追蹤，然後建立分析工作階段以分析追蹤和記錄檔。 若要在 Message Analyzer 中收集網路追蹤：

1. 在 Message Analyzer 中，選取 [檔案] | [快速追蹤] | [未加密的 HTTPS]。
2. 追蹤會立即開始。 選取 [停止]  以停止追蹤，我們即可將它設定為只追蹤儲存體流量。
3. 選取 [編輯]  以編輯追蹤工作階段。
4. 選取 [Microsoft-Pef-WebProxy] ETW 提供者右邊的 [設定] 連結。
5. 在 [進階設定] 對話方塊中，按一下 [提供者] 索引標籤。
6. 在 [主機名稱篩選條件]  欄位中，指定您的儲存體端點 (以空格分隔)。 例如，您可以如下指定您的端點；將 `storagesample` 變更為儲存體帳戶的名稱：

    ```   
    storagesample.blob.core.windows.net storagesample.queue.core.windows.net storagesample.table.core.windows.net
    ```

7. 結束對話方塊，然後按一下 [重新啟動]  以開始使用主機名稱篩選條件來收集追蹤，以便追蹤中只包含 Azure 儲存體網路流量。

> [!NOTE]
> 完成網路追蹤收集之後，強烈建議您還原您在 Fiddler 中變更的設定，以解密 HTTPS 流量。 在 [Fiddler 選項] 對話方塊中，取消選取 [擷取 HTTPS CONNECT] 和 [解密 HTTPS 流量] 核取方塊。
> 
> 

如需詳細資訊，請參閱 Technet 上的 [使用網路追蹤功能](http://technet.microsoft.com/library/jj674819.aspx) 。

## <a name="review-metrics-data-in-the-azure-portal"></a>在 Azure 入口網站中檢閱計量資料
一旦您的應用程式執行了一段時間，您即可檢閱 [Azure 入口網站](https://portal.azure.com)中顯示的計量圖表，以觀察服務的執行狀況。

首先，在 Azure 入口網站中瀏覽至您的儲存體帳戶。 根據預設，包含 [成功百分比] 計量的監視圖表會顯示在 [帳戶] 刀鋒視窗上。 如果您之前曾修改圖表以顯示不同的計量，請新增 [成功百分比] 計量。

您會立即在監控圖表中看到 [成功百分比]，以及您所新增的其他計量。 在接著要調查的案例中，藉由分析 Message Analyzer 中的記錄檔，百分比成功率稍微低於 100%。

如需新增與自訂計量圖表的詳細資訊，請參閱[自訂計量圖表](storage-monitor-storage-account.md#customize-metrics-charts)。

> [!NOTE]
> 在您啟用儲存體計量之後，計量資料可能需要一些時間才會出現在 Azure 入口網站中。 這是因為過了目前這個小時之後，前一個小時的每小時計量才會顯示在 Azure 入口網站中。 此外，分鐘計量目前不會顯示在 Azure 入口網站中。 因此視您啟用度量的時間而定，有可能需要兩個小時才看得見度量資料。
> 
> 

## <a name="use-azcopy-to-copy-server-logs-to-a-local-directory"></a>使用 AzCopy 將伺服器記錄檔複製到本機目錄
Azure 儲存體會將伺服器記錄檔資料寫入至 Blob，而度量會寫入至資料表。 記錄檔 Blob 位於您儲存體帳戶已知名稱的 `$logs` 容器中。 記錄檔 Blob 會以階層方式依照年、月、日和小時命名，讓您輕鬆地找出您要調查的時間範圍。 例如，在 `storagesample` 帳戶中，01/02/2015 上午 8-9 點的記錄檔 Blob 容器為 `https://storagesample.blob.core.windows.net/$logs/blob/2015/01/08/0800`。 此容器中的個別 Blob 會循序命名，並以 `000000.log`開頭。

您可以使用 AzCopy 命令列工具，將這些伺服器端記錄檔下載至您在本機電腦上選擇的位置。 例如，您可以使用下列命令，將發生於 2015 年 1 月 2 日的 Blob 作業的記錄檔下載至 `C:\Temp\Logs\Server` 資料夾；以您的儲存體帳戶名稱取代 `<storageaccountname>`，並以您的帳戶存取金鑰取代 `<storageaccountkey>`：

```azcopy
AzCopy.exe /Source:http://<storageaccountname>.blob.core.windows.net/$logs /Dest:C:\Temp\Logs\Server /Pattern:"blob/2015/01/02" /SourceKey:<storageaccountkey> /S /V
```
在 [Azure 下載](https://azure.microsoft.com/downloads/) 頁面可以下載 AzCopy。 如需使用 AzCopy 的詳細資訊，請參閱 [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)。

如需下載伺服器端記錄檔的詳細資訊，請參閱 [下載儲存體記錄記錄檔資料](http://msdn.microsoft.com/library/azure/dn782840.aspx#DownloadingStorageLogginglogdata)。

## <a name="use-microsoft-message-analyzer-to-analyze-log-data"></a>使用 Microsoft Message Analyzer 來分析記錄檔資料
Microsoft Message Analyzer 是在疑難排解與診斷案例中用於擷取、顯示及分析通訊協定訊息流量、事件和其他系統或應用程式訊息的工具。 Message Analyzer 也可讓您載入、彙總和分析記錄檔和儲存的追蹤檔案中的資料。 如需 Message Analyzer 的詳細資訊，請參閱 [Microsoft Message Analyzer 操作指南](http://technet.microsoft.com/library/jj649776.aspx)。

Message Analyzer 包含 Azure 儲存體資產，可協助您分析伺服器、用戶端和網路記錄檔。 在本節中，我們將討論如何使用這些工具來解決儲存記錄檔中低成功百分比的問題。

### <a name="download-and-install-message-analyzer-and-the-azure-storage-assets"></a>下載並安裝 Message Analyzer 和 Azure 儲存體資產
1. 從 Microsoft 下載中心下載 [Message Analyzer](http://www.microsoft.com/download/details.aspx?id=44226) ，並執行安裝程式。
2. 啟動 Message Analyzer。
3. 從 [工具] 功能表，選取 [資產管理員]。 在 [資產管理員] 對話方塊中，選取 [下載]，然後依據 [Azure 儲存體] 篩選。 您會看到 Azure 儲存體資產，如下圖所示。
4. 按一下 [同步處理所有顯示的項目]  以安裝 Azure 儲存體資產。 可用的資產包括：
   * **Azure 儲存體色彩規則** ：Azure 儲存體色彩規則可讓您定義特殊篩選條件，以使用色彩、文字和字型樣式來反白顯示在追蹤中包含特定資訊的訊息。
   * **Azure 儲存體圖表** ：Azure 儲存體圖表是圖形伺服器記錄資料的預先定義的圖表。 請注意，若要在此時使用 Azure 儲存體圖表，您可能只要將伺服器記錄檔載入到 [分析方格]。
   * **Azure 儲存體剖析器** ：Azure 儲存體剖析器可剖析 Azure 儲存體用戶端、伺服器和 HTTP 記錄檔，以便顯示在 [分析方格] 中。
   * **Azure 儲存體篩選條件** ：Azure 儲存體篩選條件是預先定義的準則，您可以在 [分析方格] 中用來查詢資料。
   * **Azure 儲存體檢視版面配置** ：Azure 儲存體檢視版面配置是 [分析方格] 中預先定義的資料行版本配置和群組。
5. 在您安裝資產之後，請重新啟動 Message Analyzer。

![訊息分析器資產管理員](./media/storage-e2e-troubleshooting/mma-start-page-1.png)

> [!NOTE]
> 安裝針對此教學課程的目的而顯示的所有 Azure 儲存體資產。
> 
> 

### <a name="import-your-log-files-into-message-analyzer"></a>將記錄檔匯入 Message Analyzer 中
您可以將所有已儲存的記錄檔 (伺服器端、用戶端和網路) 匯入 Microsoft Message Analyzer 的單一工作階段中進行分析。

1. 在 Microsoft Message Analyzer 的 [檔案] 功能表中，按一下 [新增工作階段]，然後按一下 [空白工作階段]。 在 [新增工作階段]  對話方塊中，輸入您的分析工作階段名稱。 在 [工作階段詳細資料] 面板中，按一下 [檔案] 按鈕。
2. 若要載入 Message Analyzer 所產生的網路追蹤資料，請按一下 [新增檔案]，瀏覽至您從 Web 追蹤工作階段儲存的 .matp 檔案所在位置，選取 .matp 檔案，然後按一下 [開啟]。
3. 若要載入伺服器端記錄檔資料，請按一下 [新增檔案]，瀏覽至您下載伺服器端記錄檔的位置，選取您要分析的時間範圍內的記錄檔，然後按一下 [開啟]。 然後，在 [工作階段詳細資料] 面板中，將每個伺服器端記錄檔的 [文字記錄檔設定] 下拉式清單設定為 [AzureStorageLog]，以確保 Microsoft Message Analyzer 可以正確剖析記錄檔。
4. 若要載入用戶端記錄檔資料，請按一下 [新增檔案]，瀏覽至您儲存用戶端記錄檔的位置，選取您要分析的記錄檔，然後按一下 [開啟]。 然後，在 [工作階段詳細資料] 面板中，將每個用戶端記錄檔的 [文字記錄檔設定] 下拉式清單設定為 [AzureStorageClientDotNetV4]，以確保 Microsoft Message Analyzer 可以正確剖析記錄檔。
5. 按一下 [新增工作階段] 對話方塊中的 [開始]，以載入和剖析記錄檔資料。 記錄檔資料會顯示在 Message Analyzer 分析方格中。

下圖顯示使用伺服器、用戶端與網路追蹤記錄檔設定的範例工作階段。

![設定 Message Analyzer 工作階段](./media/storage-e2e-troubleshooting/configure-mma-session-1.png)

請注意，Message Analyzer 會將記錄檔載入記憶體中。 如果您有大量記錄檔資料，您會想要進行篩選，以便 Message Analyzer 發揮最佳效能。

首先，判斷您有興趣檢閱的時間範圍，而此時間範圍越小越好。 在許多情況下，您會想檢閱數分鐘或數小時 (最多) 的期間。 匯入可符合您需求的最小記錄檔集合。

如果您仍有大量的記錄檔資料，您可能會想要指定工作階段篩選條件，以在載入記錄檔資料前進行篩選。 在 [工作階段篩選條件] 方塊中，選取 [程式庫] 按鈕來選擇預先定義的篩選條件。例如，從 Azure 儲存體篩選條件中選擇 [全域時間篩選 I]，以篩選時間間隔。 然後，您可以編輯篩選準則，針對您要查看的間隔指定開始和結束時間戳記。 您也可以篩選特定狀態碼 ；例如，您可以選擇只載入狀態碼為 404 的記錄檔項目。

如需將記錄檔資料匯入 Microsoft Message Analyzer 的詳細資訊，請參閱 TechNet 上的 [擷取訊息資料](http://technet.microsoft.com/library/dn772437.aspx) 。

### <a name="use-the-client-request-id-to-correlate-log-file-data"></a>使用用戶端要求識別碼讓記錄檔資料相互關聯
Azure 儲存體用戶端程式庫會自動為每一項要求產生唯一的用戶端要求識別碼。 這個值會寫入至用戶端記錄檔、伺服器記錄檔和網路追蹤，以便您使用它讓 Message Analyzer 內全部三個記錄檔的資料相互關聯。 如需用戶端要求識別碼的其他資訊，請參閱 [用戶端要求識別碼](storage-monitoring-diagnosing-troubleshooting.md#client-request-id) 。

下列各節說明如何使用預先設定和自訂的版面配置檢視，根據用戶端要求識別碼讓資料相互關聯和分組。

### <a name="select-a-view-layout-to-display-in-the-analysis-grid"></a>選取要顯示在分析方格中的檢視版本配置
適用於 Message Analyzer 的儲存體資產包括 Azure 儲存體檢視版面配置，這些是預先設定的檢視，可用來顯示在不同案例中具有實用群組和資料行的資料。 您也可以建立自訂檢視版面配置並儲存起來以便重複使用。

下圖顯示 [檢視版面配置] 功能表，此功能表可從工具列功能區選取 [檢視版本配置] 取得。 Azure 儲存體的檢視版本配置位於功能表的 [Azure 儲存體]  節點之下。 您可以在 [搜尋] 方塊中搜尋 `Azure Storage` ，只篩選 Azure 儲存體檢視版面配置。 您也可以選取檢視版面配置旁邊的星號，使其成為我的最愛並顯示在功能表的頂端。

![檢視版面配置功能表](./media/storage-e2e-troubleshooting/view-layout-menu.png)

首先選取 [ **依 ClientRequestID 和模組分組**]。 此檢視版面配置會先依用戶端要求識別碼，再依來源記錄檔 (或 Message Analyzer 中的 [模組]  )，將全部三個記錄檔中的記錄檔資料分組。 與此檢視中，您可以向下切入到特定用戶端要求識別碼，並查看所有三個記錄檔中該用戶端要求識別碼的資料。

下圖顯示套用至範例記錄檔資料的這個版面配置檢視，其中顯示部分的資料行。 您可以看見針對特定用戶端要求識別碼，分析方格會顯示來自用戶端記錄檔、伺服器記錄檔和網路追蹤的資料。

![Azure 儲存體檢視版面配置](./media/storage-e2e-troubleshooting/view-layout-client-request-id-module.png)

> [!NOTE]
> 不同的記錄檔有不同的資料行，因此當分析方格中顯示來自多個記錄檔的資料時，某些資料行可能不會包含指定資料列的資料。 例如，在上圖中，用戶端記錄檔資料列不會顯示 **Timestamp**、**TimeElapsed**、**Source** 和 **Destination** 資料行的任何資料，因為這些資料行不存在於用戶端記錄檔中，但存在於網路追蹤中。 同樣地，**Timestamp** 資料行會顯示伺服器記錄檔中的時間戳記資料，但不會顯示 **TimeElapsed**、**Source** 和 **Destination** 資料行 (不屬於伺服器記錄檔) 的資料。
> 
> 

除了使用 Azure 儲存體檢視版面配置，您也可以定義和儲存自己的檢視版面配置。 您可以選取其他所需的欄位來進行資料分組，並將群組儲存在您自訂的版面配置中。

### <a name="apply-color-rules-to-the-analysis-grid"></a>將色彩規則套用至分析方格
儲存體資產也包含色彩規則，以便以視覺化方式識別分析方格中不同類型的的錯誤。 預先定義的色彩規則會套用至 HTTP 錯誤，所以只會針對伺服器記錄檔和網路追蹤顯示。

若要套用色彩規則，請從工具列功能區中選取 [色彩規則]  。 您會在功能表中看到 Azure 儲存體色彩規則。 針對本教學課程，選取 [用戶端錯誤 (400 與 499 之間的 StatusCode)] ，如下圖所示。

![Azure 儲存體檢視版面配置](./media/storage-e2e-troubleshooting/color-rules-menu.png)

除了使用 Azure 儲存體色彩規則，您也可以定義並儲存自己的色彩規則。

### <a name="group-and-filter-log-data-to-find-400-range-errors"></a>群組及篩選記錄檔資料以尋找 400 範圍的錯誤
接下來，我們將群組及篩選記錄檔資料，以找出 400 範圍內的所有錯誤。

1. 找出 [分析方格] 中的 **StatusCode** 資料行，以滑鼠右鍵按一下資料行標題，並選取 [群組]。
2. 接著，在 **ClientRequestId** 資料行上進行分組。 您會看到分析方格中的資料現在依狀態碼和用戶端要求識別碼排列。
3. 如果 [檢視篩選條件] 工具視窗尚未顯示，請予以顯示 。 在工具列功能區中，選取 [工具視窗]，然後選取 [檢視篩選條件]。
4. 若要篩選記錄檔資料而僅顯示 400 範圍的錯誤，請將下列篩選準則加入至 [檢視篩選條件] 視窗，然後按一下 [套用]：

    ```   
    (AzureStorageLog.StatusCode >= 400 && AzureStorageLog.StatusCode <=499) || (HTTP.StatusCode >= 400 && HTTP.StatusCode <= 499)
    ```

下圖顯示這個群組和篩選的結果。 展開狀態碼 409 的群組下方的 **ClientRequestID** 欄位，例如，顯示導致該狀態碼的作業。

![Azure 儲存體檢視版面配置](./media/storage-e2e-troubleshooting/400-range-errors1.png)

套用此篩選條件之後，您會看到用戶端記錄檔中的資料列遭到排除，因為用戶端記錄檔不包含 **StatusCode** 資料行。 首先，我們會檢閱伺服器和網路追蹤記錄檔以找出 404 錯誤，然後我們會返回用戶端記錄檔，以檢查導致這些錯誤的用戶端作業。

> [!NOTE]
> 您可以篩選 **StatusCode** 資料行，而如果將運算式加入至可納入狀態碼為 null 之記錄檔項目的篩選條件，則仍會顯示來自所有三個記錄檔 (包括用戶端記錄檔) 的資料。 若要建構此篩選運算式，請使用：
> 
> <code>&#42;StatusCode >= 400 or !&#42;StatusCode</code>
> 
> 此篩選條件會傳回用戶端記錄檔的所有資料列，以及僅限狀態碼大於 400 之伺服器記錄檔和 HTTP 記錄檔的資料列。 如果您將此篩選條件套用到依用戶端要求識別碼和模組分組的檢視版面配置，即可搜尋或捲視所有記錄檔項目，以尋找所有三個記錄檔表示的地方。   
> 
> 

### <a name="filter-log-data-to-find-404-errors"></a>篩選記錄檔資料以尋找 404 錯誤
儲存體資產包括預先定義的篩選條件，您可用來縮小記錄檔資料的範圍，以尋找您所尋找的錯誤或趨勢。 接下來，我們將會套用兩個預先定義的篩選條件：一個篩選 404 錯誤的伺服器和網路追蹤記錄檔，一個篩選指定時間範圍內的資料。

1. 如果 [檢視篩選條件] 工具視窗尚未顯示，請予以顯示 。 在工具列功能區中，選取 [工具視窗]，然後選取 [檢視篩選條件]。
2. 在 [檢視篩選條件] 視窗中，選取 [程式庫]，然後搜尋 `Azure Storage` 以尋找 Azure 儲存體篩條件。 選取**所有記錄檔中的 404 (找不到) 訊息**的篩選條件。
3. 再次顯示 [程式庫] 功能表，然後找出並選取 [全域時間篩選條件]。
4. 將篩選條件中顯示的時間戳記編輯成您想要檢視的範圍。 這有助於縮小要分析的資料範圍。
5. 您的篩選條件看起來應類似下列範例。 按一下 [套用]  將篩選條件套用到 [分析方格]。

    ```   
    ((AzureStorageLog.StatusCode == 404 || HTTP.StatusCode == 404)) And
    (#Timestamp >= 2014-10-20T16:36:38 and #Timestamp <= 2014-10-20T16:36:39)
    ```

    ![Azure 儲存體檢視版面配置](./media/storage-e2e-troubleshooting/404-filtered-errors1.png)

### <a name="analyze-your-log-data"></a>分析記錄檔資料
您現已將資料分組並進行篩選，即可檢查產生 404 錯誤的個別要求詳細資料。 在目前的檢視版面配置中，資料會依用戶端要求識別碼分組，再依記錄檔來源分組。 因為我們是在篩選 StatusCode 欄位包含 404 的要求，所以只會看到伺服器和網路追蹤資料，而不會看到用戶端記錄檔資料。

下圖顯示「取得 Blob」作業因為 Blob 不存在而產生 404 的特定要求。 請注意，某些資料行已從標準檢視中移除，以便顯示相關資料。

![已篩選的伺服器和網路追蹤記錄檔](./media/storage-e2e-troubleshooting/server-filtered-404-error.png)

接下來，我們會讓此用戶端要求識別碼與用戶端記錄檔資料相互關聯，以查看用戶端在錯誤發生時所採取的動作。 您可以顯示此工作階段的新分析方格檢視，以檢視在第二個索引標籤中開啟的用戶端記錄檔資料：

1. 首先，將 **ClientRequestId** 欄位的值複製到剪貼簿。 您可以選取任一個資料列、找出 **ClientRequestId** 欄位，以滑鼠右鍵按一下資料值，然後選擇 [複製 'ClientRequestId']。
2. 在工具列功能區中，選取 [新增檢視器]，然後選取 [分析方格] 以開啟新的索引標籤。新索引標籤會顯示您記錄檔中的所有資料，而不分組、篩選或套用色彩規則。
3. 在工具列功能區中，選取 [檢視版面配置]，然後選取 [Azure 儲存體] 區段下的 [所有 .NET 用戶端資料行]。 此檢視版面配置會顯示用戶端記錄檔中的資料，以及伺服器和網路追蹤記錄檔中的資料。 預設會依 **MessageNumber** 資料行排序。
4. 接下來，搜尋用戶端記錄檔中的用戶端要求識別碼。 在工具列功能區中，選取 [尋找訊息]，然後在 [尋找] 欄位中指定用戶端要求識別碼的自訂篩選條件。 將此語法用於篩選，並指定自己的用戶端要求識別碼：

    ```
    *ClientRequestId == "398bac41-7725-484b-8a69-2a9e48fc669a"
    ```

Message Analyzer 會找出並選取搜尋準則符合用戶端要求識別碼的第一個記錄檔項目。 在用戶端記錄檔中，每個用戶端要求識別碼都有數個項目，您可以依照 **ClientRequestId** 欄位加以分組，以便輕鬆地一起查看這些項目。 下圖顯示用戶端記錄檔中指定之用戶端要求識別碼的所有訊息。

![顯示 404 錯誤的用戶端記錄檔](./media/storage-e2e-troubleshooting/client-log-analysis-grid1.png)

使用這兩個索引標籤的檢視版面配置中顯示的資料，即可分析要求資料，以判斷造成錯誤的可能原因。 您也可以查看這一個要求之前的要求，看看前一個事件是否可能造成 404 錯誤。 例如，您可以檢閱此用戶端要求識別碼之前的用戶端記錄檔項目，以判斷是否可能已刪除 Blob，或是否因為用戶端應用程式在容器或 Blob 上呼叫 CreateIfNotExists API 而造成。 在用戶端記錄檔中，您可以在 [描述] 欄位中找到 Blob 的位址；在伺服器和網路追蹤記錄檔中，此資訊會出現在 [摘要] 欄位中。

一旦知道造成 404 錯誤之 Blob 的位址，即可進一步調查。 如果您在記錄檔項目中搜尋其他與相同 Blob 上的作業相關聯的訊息，您可以檢查用戶端先前是否刪除了實體。

## <a name="analyze-other-types-of-storage-errors"></a>分析其他類型的儲存體錯誤
您現已熟悉使用 Message Analyzer 來分析記錄檔資料，便可利用檢視版面配置、色彩規則及搜尋/篩選功能，分析其他類型的錯誤。 下表列出一些您可能會遇到的問題以及您可用來尋找問題的篩選準則。 如需建構篩選條件與 Message Analyzer 篩選語言的詳細資訊，請參閱 [篩選訊息資料](http://technet.microsoft.com/library/jj819365.aspx)。

| 若要調查... | 使用篩選運算式… | 運算式套用到記錄檔 (用戶端、伺服器、網路、全部) |
| --- | --- | --- |
| 佇列上未預期的訊息傳遞延遲 |AzureStorageClientDotNetV4.Description 包含「正在重試失敗的作業」。 |用戶端 |
| PercentThrottlingError 的 HTTP 增加 |HTTP.Response.StatusCode   == 500 &#124;&#124; HTTP.Response.StatusCode == 503 |網路 |
| PercentTimeoutError 增加 |HTTP.Response.StatusCode   == 500 |網路 |
| PercentTimeoutError 增加 (全部) |*StatusCode   == 500 |全部 |
| PercentNetworkError 增加 |AzureStorageClientDotNetV4.EventLogEntry.Level   < 2 |用戶端 |
| HTTP 403 (禁止) 訊息 |HTTP.Response.StatusCode   == 403 |網路 |
| HTTP 404 (找不到) 訊息 |HTTP.Response.StatusCode   == 404 |網路 |
| 404 (全部) |*StatusCode   == 404 |全部 |
| 共用存取簽章 (SAS) 授權問題 |AzureStorageLog.RequestStatus ==  "SASAuthorizationError" |網路 |
| HTTP 409 (衝突) 訊息 |HTTP.Response.StatusCode   == 409 |網路 |
| 409 (全部) |*StatusCode   == 409 |全部 |
| 低 PercentSuccess，或是分析記錄項目內含具有 ClientOtherErrors 交易狀態的作業 |AzureStorageLog.RequestStatus ==   "ClientOtherError" |伺服器 |
| Nagle 警告 |((AzureStorageLog.EndToEndLatencyMS   - AzureStorageLog.ServerLatencyMS) > (AzureStorageLog.ServerLatencyMS *   1.5)) 和 (AzureStorageLog.RequestPacketSize <1460) 和 (AzureStorageLog.EndToEndLatencyMS -   AzureStorageLog.ServerLatencyMS >= 200) |伺服器 |
| 伺服器和網路記錄檔中的時間範圍 |#Timestamp   >= 2014-10-20T16:36:38 and #Timestamp <= 2014-10-20T16:36:39 |伺服器、網路 |
| 伺服器記錄檔中的時間範圍 |AzureStorageLog.Timestamp   >= 2014-10-20T16:36:38 和 AzureStorageLog.Timestamp <=   2014-10-20T16:36:39 |伺服器 |

## <a name="next-steps"></a>後續步驟
如需在 Azure 儲存體中進行端對端案例疑難排解的詳細資訊，請參閱下列資源：

* [監視、診斷與疑難排解 Microsoft Azure 儲存體](storage-monitoring-diagnosing-troubleshooting.md)
* [Storage Analytics](http://msdn.microsoft.com/library/azure/hh343270.aspx)
* [在 Azure 入口網站中監視儲存體帳戶](storage-monitor-storage-account.md)
* [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)
* [Microsoft Message Analyzer 操作指南](http://technet.microsoft.com/library/jj649776.aspx)
