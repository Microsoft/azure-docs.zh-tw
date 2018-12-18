---
title: 使用 Azure 時間序列深入解析總管來探索資料 | Microsoft Docs
description: 本文說明如何在網頁瀏覽器中使用 Azure 時間序列深入解析總管，快速查看您巨量資料的全域檢視，並驗證您的 IoT 環境。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 11/30/2017
ms.openlocfilehash: dfdc538719b0c7571ba04f4134819d7142f109d3
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39629135"
---
# <a name="azure-time-series-insights-explorer"></a>Azure 時間序列深入解析總管
本文探討時間序列深入解析總管 Web 應用程式中的各種功能和可用的選項。 在網頁瀏覽器中使用時間序列深入解析總管來建立資料的視覺效果。
 
Azure 時間序列深入解析是完全受控的分析、儲存體及視覺效果服務，讓您可輕易同時探索及分析數十億筆的 IoT 事件。 它可為您提供資料的全域檢視，從而讓您快速驗證 IoT 解決方案，並避免關鍵任務裝置成本高昂的停機時間。 您可以幾近即時地探索隱藏趨勢、找出異常狀況，並進行根本原因分析。 時間序列深入解析總管目前處於公開預覽狀態。

## <a name="prerequisites"></a>必要條件

在使用時間序列深入解析總管之前，您必須先：
- 建立時間序列深入解析環境
- 提供您環境中帳戶的存取權
- 將事件來源新增至內嵌資料並加以儲存

## <a name="explore-and-query-data"></a>探索及查詢資料
在將事件來源連線至時間序列深入解析環境的幾分鐘內，您可以探索並查詢時間序列資料。

1. 若要開始，請在您的網頁瀏覽器中開啟[時間序列深入解析總管](https://insights.timeseries.azure.com/)，並在視窗的左邊選取環境。 您可以存取的所有環境都會依字母順序列出。

2. 一旦您選取環境後，請使用頂端的 **FROM** 和 **TO** 組態，或按一下並拖曳您所需的時間範圍。  按一下右上方的放大鏡，或以滑鼠右鍵按一下選取的範圍，然後選取 [搜尋]。  

3. 您也可以每隔一分鐘自動重新整理可用性，方法為選取 [自動開啟] 按鈕。  請注意，[自動開啟] 按鈕只適用於可用性圖表，不適用於主要視覺效果的內容。

4. 請注意，Azure 雲端圖示會帶您前往 Azure 入口網站中的環境。

   ![時間序列深入解析環境](media/time-series-insights-explorer/explorer1.png)

5. 接下來，您會看到一個圖表，顯示所選時間範圍期間所有事件的計數。  這裡您有許多控制項：

    **字詞編輯器面板**：字詞空間是您用來查詢環境的位置。  可在畫面左邊找到，啟用 
      - **量值**：這個下拉式清單會顯示所有的數值資料行 (雙精度浮點數)
      - **分割依據**：這個下拉式清單會顯示類別資料行 (字串)
      - 您可以啟用步驟插補、顯示最小和最大值，並從量值旁的控制台調整 y 軸。  此外，您可以調整所顯示的資料為計數、平均值還是資料的總和。
      - 您可以新增最多五個要在相同 x 軸上檢視的字詞。  使用 [copy-down] 按鈕以新增其他字詞，或按一下 [新增] 按鈕來新增新的字詞。
     
        ![字詞編輯器面板](media/time-series-insights-explorer/explorer2.png)

      - **述詞**：述詞可讓您使用下面所列的運算元集合來快速篩選事件。 如果您選取/按一下以進行搜尋，述詞會以該搜尋作為基礎自動更新。      支援的運算元類型包括：

         |作業  |支援的類型  |注意  |
         |---------|---------|---------|
         |<, >, <=, >=     |  Double、DateTime、TimeSpan       |         |
         |=, !=, <>     | String、Bool、Double、DateTime、TimeSpan、NULL        |         |
         |IN     | String、Bool、Double、DateTime、TimeSpan、NULL        |  所有的運算元都應該是相同的類型或是 NULL 常數。        |
         |HAS     | 字串        |  右側只允許常數字串常值。 不允許空字串和 NULL。       |

      - **查詢的範例**
      
         ![查詢範例](media/time-series-insights-explorer/explorer9.png)

6. **間隔大小**滑桿工具可讓您透過相同的時間範圍縮放間隔。  這可在大型配量時間 (會顯示向下配量最小至毫秒的平滑趨勢) 之間提供更精確的移動控制，從而讓您查看資料的細微、高解析度的剪輯。 滑桿的預設起始點會從您所選取項目中設定為資料的最佳檢視；平衡解析度、查詢速度與細微性。

7. **時間筆刷**工具可讓您輕鬆地從一個時間範圍瀏覽至另一個時間範圍，在時間範圍之間放置直覺式 UX 前端和中心以進行無縫移動。

8. **儲存**命令可讓您儲存目前的查詢，並加以啟用以供與環境的其他使用者共用。 您可以使用**開啟**來查看您可存取之環境中所有已儲存的查詢和其他使用者的任何共用查詢。 

   ![查詢](media/time-series-insights-explorer/explorer3.png)

9. **透視圖**工具能提供同時檢視最多四個唯一的查詢。 您可以在圖表右上角找到 [透視圖] 按鈕。  

   ![透視圖](media/time-series-insights-explorer/explorer4.png)

10. **圖表**可讓您以視覺化方式探索資料。 圖表工具包括：

   - 選取/按一下，可讓您選取特定時間範圍或單一資料序列。  
   - 在定時間範圍選取項目內，您可以放大或探索事件。  
   - 在資料序列內，您可以由另一個資料行分割序列、將序列新增為新的字詞、只顯示選取的序列、排除選取的序列、釘選該序列或從選取的序列探索事件。
   - 在圖表左側的篩選區域中，您可以查看所有顯示的資料序列並依值或名稱重新排列、檢視所有資料序列或特別是已釘選或取消取消釘選的序列。  您也可以選取單一資料序列，並由另一個資料行分割序列、將序列新增為新的字詞、只顯示選取的序列、排除選取的序列、釘選該序列或從選取的序列探索事件。
   - 當您同時檢視多個字詞時，可以堆疊、取消堆疊、查看關於資料序列的其他資料，以及使用圖表右上角的按鈕跨所有字詞使用相同的 Y 軸。
 
   ![圖表工具](media/time-series-insights-explorer/explorer5.png) 

11. **熱度圖**可用來在給定查詢中快速找出唯一或異常的資料序列。 只有一個搜尋字詞可視覺化為熱度圖。    

   ![熱度圖](media/time-series-insights-explorer/explorer6.png)

12. **事件**：當您在上方選取或以滑鼠右鍵按一下來選擇探索事件時，事件面板可供使用。  在這裡，您可以看到所有未經處理的事件，並將您的事件匯出為 JSON 或 CSV 檔案。 請注意，時間序列深入解析會儲存所有未經處理的資料。

   ![活動](media/time-series-insights-explorer/explorer7.png)

13. 探索事件之後請按一下 [STATS] 索引標籤，以公開模式和資料行統計資料。  

   - **模式**：這項功能會主動瀏覽所選取資料區中最具統計顯著性的模式。 這可以讓您不必查看數千個事件，就能了解哪些模式最需要時間和精力。 此外，時間序列深入解析可讓您直接跳至這些最具統計顯著性的模式，以繼續進行分析。 這項功能對於事後調查歷史資料也很有幫助。 

   - **資料行統計資料**：資料行統計資料會提供圖表及資料表，將所選時間範圍內、已選取資料序列中的每個資料行的資料加以分割。  
 
      ![STATS](media/time-series-insights-explorer/explorer8.png) 

現在您已經了解時間序列深入解析總管 Web 應用程式中的各種功能和可用的選項。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
>[在 Time Series Insights 環境中診斷與解決問題](time-series-insights-diagnose-and-solve-problems.md)
