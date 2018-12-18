---
title: 從 Application Insights 連續匯出遙測 | Microsoft Docs
description: 匯出診斷和使用量資料至 Microsoft Azure 中的儲存體，並從那裡下載。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 5b859200-b484-4c98-9d9f-929713f1030c
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 08/20/2018
ms.author: mbullwin
ms.openlocfilehash: a960ace141d5d71559c39c627f96746a25bf5207
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42142333"
---
# <a name="export-telemetry-from-application-insights"></a>從 Application Insights 匯出遙測
想要讓遙測保留比標準保留期限還久的時間？ 或以某些特殊方式處理它？ 連續匯出很適合此用途。 在 Application Insights 入口網站中看見的事件，可以使用 JSON 格式匯出到 Microsoft Azure 中的儲存體。 從那裡，您可以下載資料並編寫處理所需的任何程式碼。  

在設定連續匯出之前，您可能要考慮某些替代作法︰

* 計量或搜尋刀鋒視窗頂端的 [匯出] 按鈕，可讓您傳送資料表和圖表到 Excel 試算表。

* [分析](app-insights-analytics.md) 可提供功能強大的遙測查詢語言。 它也可以匯出結果。
* 如果您想要 [在 Power BI 中探索資料](app-insights-export-power-bi.md)，不需要用到「連續匯出」也可以這麼做。
* [資料存取 REST API](https://dev.applicationinsights.io/) 可讓您以程式設計方式存取您的遙測。
* 您也可以[透過 Powershell 存取連續匯出設定](https://docs.microsoft.com/powershell/module/azurerm.applicationinsights/new-azurermapplicationinsightscontinuousexport?view=azurermps-5.7.0)。

在「連續匯出」將您的資料複製到儲存體 (資料可在此依您喜好的時間長短存放) 之後，資料仍然會在 Application Insights 中依一般的[保留期間](app-insights-data-retention-privacy.md)可供使用。

## <a name="setup"></a> 建立連續匯出
1. 在您應用程式的 Application Insights 資源中，開啟 [連續匯出]，然後選擇 [新增]：

    ![向下捲動並按一下 [連續匯出]](./media/app-insights-export-telemetry/01-export.png)

2. 選擇您想要匯出的遙測資料類型。

3. 建立或選取要用來儲存資料的 [Azure 儲存體帳戶](../storage/common/storage-introduction.md)。

    > [!Warning]
    > 根據預設，儲存體位置將設為與您 Application Insights 資源相同的地理區域。 如果您儲存在不同的區域中，可能會產生傳輸費用。

    ![按一下 [加入]、[匯出目的地]、[儲存體帳戶]，然後建立新儲存區或選擇現有儲存區](./media/app-insights-export-telemetry/02-add.png)

4. 建立或選取儲存體中的容器︰

    ![按一下 [選擇事件類型]](./media/app-insights-export-telemetry/create-container.png)

建立匯出之後，就會開始進行。 您只會取得建立匯出之後送抵的資料。

資料出現在儲存體中之前可能有大約一小時的延遲。

### <a name="to-edit-continuous-export"></a>若要編輯連續匯出

如果稍後想要變更事件類型，只需要編輯匯出：

![按一下 [選擇事件類型]](./media/app-insights-export-telemetry/05-edit.png)

### <a name="to-stop-continuous-export"></a>若要停止連續匯出

若要停止匯出，請按一下 [停用]。 再次按一下 [啟用] 時，匯出將會以新資料重新啟動。 您無法取得在停用匯出時送抵入口網站的資料。

若要永久停止匯出，請刪除它。 這麼做不會將您的資料從儲存體刪除。

### <a name="cant-add-or-change-an-export"></a>無法加入或變更匯出？
* 若要加入或變更匯出，您需要擁有者、參與者或 Application Insights 參與者存取權。 [了解角色][roles]。

## <a name="analyze"></a> 您取得什麼事件？
匯出的資料是我們從您的應用程式接收的原始遙測，只不過我們加入了從用戶端 IP 位址計算的位置資料。

[取樣](app-insights-sampling.md) 已捨棄的資料不會包含在匯出的資料中。

未包含其他計算的度量。 例如，我們不會匯出平均 CPU 使用率，但我們會匯出用以計算平均的原始遙測。

該資料也包含您曾設定之 [可用性 Web 測試](app-insights-monitor-web-app-availability.md) 的任何結果。

> [!NOTE]
> **取樣** 如果應用程式會傳送大量資料，取樣功能或許會運作，並只傳送一小部分產生的遙測。 [深入了解取樣。](app-insights-sampling.md)
>
>

## <a name="get"></a> 檢查資料
您可以直接在入口網站中檢查儲存體。 按一下 [瀏覽]、選取您的儲存體帳戶，然後開啟 [容器]。

若要在 Visual Studio 中檢查 Azure 儲存體，請依序開啟 [檢視]、[Cloud Explorer]。 (如果您沒有該功能表命令，則必須安裝 Azure SDK：開啟 [新增專案] 對話方塊，展開 [Visual C#]/[Cloud]，然後選擇 [取得 Microsoft Azure SDK for .NET]。)

當您開啟 Blob 存放區時，您會看到含有一組 Blob 檔案的容器。 衍生自您 Application Insights 的資源名稱、其檢測金鑰、遙測-類型/日期/時間之每個檔案的 URI。 (資源名稱全部小寫，而檢測金鑰會省略連字號。)

![使用適合的工具檢查 Blob 儲存區](./media/app-insights-export-telemetry/04-data.png)

日期和時間為 UTC，並且是遙測存放在儲存區的時間 - 而不是產生的時間。 因此，如果您編寫程式碼來下載資料，它可以透過資料線性地移動。

以下是路徑的格式︰

    $"{applicationName}_{instrumentationKey}/{type}/{blobDeliveryTimeUtc:yyyy-MM-dd}/{ blobDeliveryTimeUtc:HH}/{blobId}_{blobCreationTimeUtc:yyyyMMdd_HHmmss}.blob"

Where

* `blobCreationTimeUtc` 是在內部暫存儲存體中建立 Blob 的時間
* `blobDeliveryTimeUtc` 是將 Blob 複製到匯出目的地儲存體的時間

## <a name="format"></a> 資料格式
* 每個 Blob 是包含多個以 '\n' 分隔的列的文字檔案。 它包含大約半分鐘的時間內所處理的遙測。
* 每個資料列都代表遙測資料點，例如要求或頁面檢視。
* 每列是未格式化的 JSON 文件。 如果您想要靜靜地仔細觀看，請在 Visual Studio 中開啟，並依序選擇 [編輯]、[進階]、[格式檔案]：

![使用合適的工具檢視遙測](./media/app-insights-export-telemetry/06-json.png)

時間期間依刻度為單位，10000 刻度 = 1 毫秒。 例如，這些值顯示從瀏覽器傳送要求用了 1 毫秒的時間，接收它用了 3 毫秒，以及在瀏覽器中處理頁面用了 1.8 秒：

    "sendRequest": {"value": 10000.0},
    "receiveRequest": {"value": 30000.0},
    "clientProcess": {"value": 17970000.0}

[屬性類型和值的詳細資料模型參考。](app-insights-export-data-model.md)

## <a name="processing-the-data"></a>處理資料
就小型規模而言，您可以編寫一些程式碼來取出您的資料，將它讀取為試算表等等。 例如︰

    private IEnumerable<T> DeserializeMany<T>(string folderName)
    {
      var files = Directory.EnumerateFiles(folderName, "*.blob", SearchOption.AllDirectories);
      foreach (var file in files)
      {
         using (var fileReader = File.OpenText(file))
         {
            string fileContent = fileReader.ReadToEnd();
            IEnumerable<string> entities = fileContent.Split('\n').Where(s => !string.IsNullOrWhiteSpace(s));
            foreach (var entity in entities)
            {
                yield return JsonConvert.DeserializeObject<T>(entity);
            }
         }
      }
    }

如需較大型的程式碼範例，請參閱[使用背景工作角色][exportasa]。

## <a name="delete"></a>刪除舊資料
請注意，您負責管理儲存容量，以及在必要時刪除舊資料。

## <a name="if-you-regenerate-your-storage-key"></a>如果您重新產生儲存體金鑰...
如果您變更儲存體的金鑰，連續匯出將停止運作。 您將在 Azure 帳戶中看到通知。

開啟 [連續匯出] 分頁，並編輯您的匯出。 編輯 [匯出目的地]，但只保留選取相同的儲存體。 按一下 [確定] 以確認。

![編輯連續匯出，開啟並關閉匯出目的地。](./media/app-insights-export-telemetry/07-resetstore.png)

連續匯出將重新開始。

## <a name="export-samples"></a>匯出範例

* [使用串流分析匯出至 SQL][exportasa]
* [串流分析範例 2](app-insights-export-stream-analytics.md)

就更大型規模而言，請考慮 [HDInsight](https://azure.microsoft.com/services/hdinsight/) - 雲端中的 Hadoop 叢集。 HDInsight 提供各種管理和分析巨量資料的技術，您可以使用它來處理已從 Application Insights 匯出的資料。

## <a name="q--a"></a>問答集
* *但我想要的只是一次性下載圖表。*  

    是的，您可以這麼做。 在刀鋒視窗頂端，按一下 [ **匯出資料**]。
* *我設定匯出，但我的儲存區中沒有資料。*

    自從設定匯出之後，Application Insights 是否從您的應用程式收到任何遙測？ 您將只會收到新資料。
* *我嘗試設定匯出，但被拒絕存取*

    如果帳戶是組織所擁有，您必須是擁有者或參與者群組的成員。
* *我是否能直接匯出到我自己的內部部署儲存區？*

    否，抱歉。 我們的匯出引擎目前僅適用於 Azure 儲存體。  
* *放置在我的儲存區中的資料量有任何限制？*

    否。 我們將持續送入資料，直到刪除匯出為止。 如果我們到達 Blob 儲存體的外部限制，將會停止，但那個限制很大。 您可以自行控制使用的儲存體數量。  
* *應該在儲存體中看到多少 Blob？*

  * 針對您選取要匯出的每個資料類型，會每分鐘建立一個新的 Blob (如果有可用的資料)。
  * 此外，針對具有高流量的應用程式，則會配置額外的分割單位。 在此情況下，每個單位會每分鐘建立一個 Blob。
* *我對我的儲存體重新產生了金鑰，或變更了容器的名稱，現在匯出沒有作用。*

    編輯匯出並開啟匯出目的地分頁。 照舊保留選取相關的儲存體，並按一下 [確定] 來確認。 匯出將重新開始。 如果變更是在最近幾天內，您不會遺失資料。
* *我可以暫停匯出嗎？*

    是。 按一下 [停用]。

## <a name="code-samples"></a>程式碼範例

* [串流分析範例](app-insights-export-stream-analytics.md)
* [使用串流分析匯出至 SQL][exportasa]
* [屬性類型和值的詳細資料模型參考。](app-insights-export-data-model.md)

<!--Link references-->

[exportasa]: app-insights-code-sample-export-sql-stream-analytics.md
[roles]: app-insights-resources-roles-access-control.md
