---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: scottwhi-msft
ms.service: cognitive-services
ms.topic: include
ms.date: 04/09/2018
ms.author: scottwhi
ms.custom: include file
ms.openlocfilehash: 4e19c1afefdc5bcacebcb0d495193b48c7a6d724
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36313705"
---
Bing 統計資料可為 Bing 搜尋 API 提供分析。 Analytics 包括呼叫量、熱門查詢字串、地理分佈等等。 若要在 Bing 搜尋付費訂用帳戶中啟用 Bing 統計資料，請瀏覽至 [Azure 儀表板](https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7)，選取您的付費訂用帳戶，然後按一下 [啟用 Bing 統計資料]。 啟用 Bing 統計資料會稍微增加訂用帳戶的費率 (請參閱[定價](https://aka.ms/bingstatisticspricing))。


> [!NOTE]
> Bing 統計資料僅適用於付費訂用帳戶，免費的試用訂用帳戶無法使用。 

> [!NOTE]
> 您可能無法使用透過 Bing 統計資料儀表板提供的任何資料，來建立向第三方散發的應用程式。

Bing 會每隔 24 小時更新分析資料一次，並可維護長達 13 個月的歷程記錄。

## <a name="accessing-your-analytics"></a>存取您的分析

若要存取分析儀表板，請移至 https://bingapistatistics.com。 請確定您用來登入的 Microsoft 帳戶 (MSA) 是用來取得付費訂用帳戶的相同帳戶。


## <a name="filtering-the-data"></a>篩選資料

根據預設，圖表與圖形會反映您有權存取的所有計量資料。 您可以選取資源、市場、端點，以及感興趣的報告週期，以篩選圖表與圖形中所顯示的資料。 圖表與圖形會變更以反映您所套用的篩選條件。 以下說明您可能會變更的篩選條件。

- **資源識別碼**：可識別 Azure 訂用帳戶的唯一資源識別碼。 如果您訂閱一個以上的 Bing 搜尋 API 層，此清單會包含多個識別碼。 預設會選取所有資源。  
  
- **市場**：結果來自的市場。 例如，en-US (英文，美國)。 預設會選取所有市場。 請注意，如果呼叫未指定市場，而且 Bing 無法判斷使用者的市場，Bing 會使用 en-WW 市場。  
  
- **端點**：Bing 搜尋 API 端點。 此清單包含您有付費訂用帳戶的所有端點。 預設會選取所有端點。  

- **時間範圍**：報告期間。 您可以指定：  
  
  - 全部&mdash;包含長達 13 個月的資料量  
  - 過去 24 小時&mdash;包含過去 24 小時內的分析  
  - 過去一週&mdash;包含過去七天內的分析  
  - 過去一個月&mdash;包含過去 30 天內的分析  
  - 自訂日期範圍&mdash;包含指定日期範圍內的分析 (如果有的話)  
  
  > [!NOTE]  
  > 計量最多可能需要 24 小時才會呈現在儀表板上。 儀表板會顯示上次更新資料的時間與日期。  
  
  > [!NOTE]  
  > 從啟用 Bing 統計資料增益集起，就可以取得計量。 


## <a name="charts-and-graphs"></a>圖表和圖形

儀表板會顯示所選端點可用計量的圖表和圖形。 並非所有端點都能取得所有計量。 每個端點的圖表和圖形都是靜態的 (您無法選取要顯示的圖表和圖表)。 儀表板只會顯示有資料可用的圖表和圖形。 

<!--
For example, if you don't include the User-Agent header in your calls, the dashboard will not include device-related graphs.
-->

以下是一些可能的計量。 每個計量都會指明端點限制。 

- **呼叫量**：顯示在報告期間發出的呼叫次數。 如果報告期間為一天，圖表會顯示每小時發出的呼叫次數。 否則，圖表會顯示報告期間內每日發出的呼叫次數。  
  
  > [!NOTE]
  > 呼叫量可能會和帳單報告有所不同，帳單報告通常只包括成功的呼叫。 
  
-  **熱門查詢**：顯示報告期間內的熱門查詢，以及每個查詢的發生次數。 您可以設定顯示的查詢數目。 例如，您可以顯示前 25、50 或 75 個查詢。 熱門查詢不適用於下列端點：  
  
  - /images/trending
  - /images/details
  - /images/visualsearch
  - /videos/trending
  - /videos/details
  - /news
  - /news/trendingtopics
  - /suggestions  
  
  > [!NOTE]  
  > 某些查詢詞彙可能隱藏起來以移除機密資訊，例如電子郵件、電話號碼、SSN 等。  

- **地理分佈**：結果所來自的市場。 例如，en-US (英文，美國)。 Bing 使用 `mkt` 查詢參數來判斷市場 (如果指定的話)。 否則，Bing 會使用訊號 (例如呼叫端的 IP 位址) 來判斷市場。  
  
- **回應碼分佈**：在報告期間內所有呼叫的 HTTP 狀態碼。  
  
- **呼叫來源分佈**：使用者使用的瀏覽器類型。 例如，Edge、Chrome、Safari 及 FireFox。 從 bot、Postman 等瀏覽器外部或從主控台應用程式使用 curl 發出的呼叫，都會分組歸類在程式庫之下。 來源是使用要求的使用者代理程式標頭值來判斷的。 如果要求不包含使用者代理程式標頭，Bing 就會嘗試從其他訊號衍生來源。  
  
- **安全搜尋分佈**：安全搜尋值的分佈。 例如，關閉、中等或嚴格。 `safeSearch` 查詢參數會包含值 (如果指定的話)。 否則，Bing 會預設該值為中等。  
  
- **要求的答案分佈**：您在 `responseFilter` 查詢參數中要求的 Web 搜尋 API 答案。  
  
- **傳回的答案分佈**：Web 搜尋 API 在回應中傳回的答案。  
  
- **回應伺服器分佈**：處理 API 要求的應用程式伺服器。 可能的值為 Bing.com (針對從桌上型與膝上型電腦裝置提供的流量) 和 Bing.com-mobile (針對從行動裝置提供的流量)。 伺服器是使用要求的使用者代理程式標頭值來判斷的。 如果要求不包含使用者代理程式標頭，Bing 會嘗試從其他訊號衍生伺服器。  
  


下圖顯示每個端點可用的分析。

![依端點支援矩陣的分佈情況](./media/cognitive-services-bing-statistics/bing-statistics-matrix.PNG)


