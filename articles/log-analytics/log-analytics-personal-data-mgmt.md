---
title: Azure Log Analytics 中所儲存個人資料的指引 | Microsoft Docs
description: 本文說明如何管理 Azure Log Analytics 中所儲存的個人資料，以及用以識別和移除個人資料的方法。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/18/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 9ea004a35f739a8c4f7ee1ed320bd6657ed4e820
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957909"
---
# <a name="guidance-for-personal-data-stored-in-log-analytics-and-application-insights"></a>儲存在 Log Analytics 和 Application Insights 中的個人資料指引

Log Analytics 是有望找到個人資料的資料存放區。 Application Insights 會將其資料儲存在 Log Analytics 分割區中。 本文會討論這類資料一般是放在 Log Analytics 和 Application Insights 中的哪個位置，以及可供您處理這類資料的功能。

> [!NOTE]
> 根據本文的用途，_記錄資料_是指傳送至 Log Analytics 工作區的資料，而_應用程式資料_則是指 Application Insights 所收集的資料。

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="strategy-for-personal-data-handling"></a>個人資料處理策略

雖然您和您的公司對於處理私人資料 (如果有的話) 的策略擁有最終決定權，以下提供一些可採行的方法。 這些方法是從技術觀點，依照推薦順序由高至低來列出：

* 如果可以，請停止收集、模糊處理、匿名處理或調整所收集的資料，讓資料不致於成為「私人」資料。 這是截至目前為止最推薦的方法，可讓您不必建立成本極高、影響層面極大的資料處理策略。
* 不行的話，請嘗試將資料標準化，以減少對資料平台和效能所造成的影響。 例如，不是記錄明確的使用者識別碼，而是建立查閱資料，將使用者名稱和其詳細資料關聯到某個可於隨後記錄到別處的內部識別碼。 這樣一來，如果某位使用者要求您刪除其個人資訊，您只要刪除查閱表中與該使用者對應的資料列可能便已足夠。 
* 最後，如果您必須收集私人資料，請圍繞清除 API 路徑和現有查詢 API 路徑建置程序，以符合您在匯出和刪除使用者相關私人資料方面所該承擔的責任。 

## <a name="where-to-look-for-private-data-in-log-analytics"></a>Log Analytics 中的私人資料位於何處？

Log Analytics 是彈性的存放區，在指定資料結構描述的同時，允許您覆寫每個含有自訂值的欄位。 此外，也能擷取任何自訂結構描述。 因此，很難確實指出私人資料在特定工作區中的存放位置。 不過，您可以先從以下位置開始清查：

### <a name="log-data"></a>記錄資料

* IP 位址：Log Analytics 會從許多不同的資料表收集各種 IP 資訊。 例如，下列查詢會顯示過去 24 小時內已收集過 IPv4 位址的所有資料表：
    ```
    search * 
    | where * matches regex @'\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b' //RegEx originally provided on https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp
    | summarize count() by $table
    ```
* 使用者識別碼：眾多解決方案和資料表中都可以發現使用者識別碼。 您可以使用搜尋命令來尋找整個資料集內的特定使用者名稱：
    ```
    search "[username goes here]"
    ```
請記住，不要只是尋找人類看得懂的使用者名稱，還要尋找可以直接回溯追蹤到特定使用者的 GUID！
* 裝置識別碼：有時候，使用者識別碼、裝置識別碼等資料也會視為是「私人」資料。 請使用和上述針對使用者識別碼所列出的相同方法，來識別可能有這方面疑慮的資料表。 
* 自訂資料：Log Analytics 允許以各種方法收集資料：自訂記錄和自訂欄位、[HTTP 資料收集器 API](log-analytics-data-collector-api.md)，以及隨系統事件記錄一起收集的自訂資料。 這些項目全都可能包含私人資料，所以應該加以檢查以確認是否有這類資料存在。
* 解決方案所擷取的資料：因為解決方案採開放式機制，建議您檢閱解決方案所產生的所有資料表以確保合規性。

### <a name="application-data"></a>應用程式資料

* *IP 位址*：雖然 Application Insights 依預設會將所有 IP 位址欄位模糊處理為「0.0.0.0」，常見模式為以實際的使用者 IP 覆寫此數值，維護工作階段資訊。 以下的 Analytics 查詢可用來尋找資料表，其 IP 位址欄在過去 24 小時內包含「0.0.0.0」以外的值：
    ```
    search client_IP != "0.0.0.0"
    | where timestamp > ago(1d)
    | summarize numNonObfuscatedIPs_24h = count() by $table
    ```
* *使用者識別碼*：依預設，Application Insights 會使用隨機產生的識別碼來追蹤使用者和工作階段。 不過，通常這些欄位會經過覆寫以儲存與應用程式更相關的識別碼。 例如：使用者名稱，AAD GUID 等等。這些識別碼經常會歸類在個人資料範圍內，因此應妥善處理。 建議一律嘗試將這些識別碼模糊或匿名處理。 通常包含這些值的欄位有 session_Id、user_Id、user_AuthenticatedId、user_AccountId 以及 customDimensions。
* *自訂資料*：Application Insights 能讓您將一組自訂維度附加至任何資料類型。 這些維度可以是*任何*資料。 您可以使用以下查詢來識別過去 24 小時內收集的任何自訂維度：
    ```
    search * 
    | where isnotempty(customDimensions)
    | where timestamp > ago(1d)
    | project $table, timestamp, name, customDimensions 
    ```
* *記憶體中和傳輸中的資料*：Application Insights 會追蹤例外狀況、要求、相依性呼叫及追蹤。 私人資料通常會在程式碼和 HTTP 呼叫層級收集。 檢閱例外狀況、要求、相依性及追蹤資料表來識別任何這類資料。 使用[遙測初始設定式](https://docs.microsoft.com/azure/application-insights/app-insights-api-filtering-sampling)，盡可能模糊處理這項資料。
* *快照偵錯工具擷取*：Application Insights 中的[快照偵錯工具](https://docs.microsoft.com/azure/application-insights/app-insights-snapshot-debugger)功能可讓您在應用程式的生產執行個體上攔截到例外狀況時收集偵錯快照集。 快照集會公開導致例外狀況的完整堆疊追蹤，以及在堆疊中每個步驟的本機變數的值。 遺憾的是，這項功能不允許選擇性刪除快照點，或以程式設計的方式來存取快照集內的資料。 因此，如果預設快照集的保留率無法滿足您的合規性需求，則建議關閉此功能。

## <a name="how-to-export-and-delete-private-data"></a>如何匯出及刪除私人資料

如先前的[個人資料處理策略](#strategy-for-personal-data-handling)一節所述，如果可以，__強烈__建議您重新制定資料收集政策以停止收集私人資料，或者模糊、匿名處理或修改私人資料，以免資料成為「私人」資料。 處理資料起先會導致您和團隊需要花費心力來定義並自動執行策略，建置介面供客戶與其資料互動，並產生長期維護成本。 此外，Log Analytics 和 Application Insights 的計算成本很高，而且大量的並行查詢或清除 API 呼叫可能會對 Log Analytics 功能的所有其他互動產生負面影響。 話雖如此，但在某些合理情況下，確實有必要收集私人資料。 在這些情況下，就應該採用本節所述的方式來處理資料。

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

### <a name="view-and-export"></a>檢視和匯出

針對檢視和匯出資料的要求，均應使用 [Log Analytics 查詢 API](https://dev.loganalytics.io/) 或 [Application Insights 查詢 API](https://dev.applicationinsights.io/quickstart)。 至於要如何將資料轉換為適當形式以提供給使用者，其實作邏輯則由您自行決定。 [Azure Functions](https://azure.microsoft.com/services/functions/) 很適合用來裝載這類邏輯。

### <a name="delete"></a>刪除

> [!WARNING]
> Log Analytics 中的刪除動作具有破壞性，且將無法復原！ 進行這方面的作業時請格外小心。

我們已將處理「清除」API 路徑的功能納入到隱私權中。 請謹慎使用此路徑，原因是這項操作會引發相關風險、可能影響效能，而且可能會扭曲整個彙總、量測和 Log Analytics 的其他方面。 如需替代的私人資料處理方法，請參閱[個人資料處理策略](#strategy-for-personal-data-handling)一節。

清除作業需要極高的特殊權限，若未對 Azure 中的應用程式或使用者 (甚至包括資源擁有者) 明確授與 Azure Resource Manager 角色，其將無權執行此作業。 這個角色便是「資料清除者」，由於可能會遺失資料，委派此角色時請務必小心。 

獲派 Azure Resource Manager 角色後，就可使用兩個新的 API 路徑： 

#### <a name="log-data"></a>記錄資料

* [POST 清除] (https://docs.microsoft.com/rest/api/loganalytics/workspaces%202015-03-20/purge) - 採用物件來指定要刪除的資料參數，並傳回參考 GUID 
* GET 清除狀態 - POST 清除呼叫會傳回 'x-ms-status-location' 標頭，其中包含可供您呼叫的 URL，以便判斷清除 API 的狀態。 例如︰

    ```
    x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/Microsoft.OperatonalInsights/workspaces/[WorkspaceName]/operations/purge-[PurgeOperationId]?api-version=2015-03-20
    ```

> [!IMPORTANT]
>  雖然絕大多數清除作業的完成速度應該都遠遠超過 SLA，但由於這些作業會對 Log Analytics 所使用的資料平台產生重大影響，所以**清除作業的正式完成 SLA 是設定為 30 天**。 

#### <a name="application-data"></a>應用程式資料

* [POST 清除](https://docs.microsoft.com/rest/api/application-insights/components/purge) - 採用物件來指定要刪除的資料參數，並傳回參考 GUID
* GET 清除狀態 - POST 清除呼叫會傳回 'x-ms-status-location' 標頭，其中包含可供您呼叫的 URL，以便判斷清除 API 的狀態。 例如︰

   ```
   x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/microsoft.insights/components/[ComponentName]/operations/purge-[PurgeOperationId]?api-version=2015-05-01
   ```

> [!IMPORTANT]
>  雖然絕大多數清除作業的完成速度應該都遠遠超過 SLA，但由於這些作業會對 Application Insights 所使用的資料平台產生重大影響，所以**清除作業的正式完成 SLA 是設定為 30 天**。

## <a name="next-steps"></a>後續步驟
- 若要深入了解 Log Analytics 資料的收集、處理和保護方式，請參閱 [Log Analytics 資料安全性](log-analytics-data-security.md)。
- 若要深入了解 Application Insights 資料的收集、處理和保護方式，請參閱 [Application Insights 資料安全性](../application-insights/app-insights-data-retention-privacy.md)。