---
title: Azure 監視器計量瀏覽器
description: 了解 Azure 監視器計量瀏覽器中的新功能
author: vgorbenko
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/17/2017
ms.author: vitaly.gorbenko
ms.component: metrics
ms.openlocfilehash: 21b0029ff12915c8416ad2366fbf6c45ddfaa288
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978410"
---
# <a name="azure-monitor-metrics-explorer"></a>Azure 監視器計量瀏覽器

此操作說明描述目前屬於公開預覽版的新一代 Azure 監視器計量圖表製作體驗。 新的體驗支援同時針對多維度計量與不含維度的基本計量呈現圖表。 您可以繪製會重疊顯示來自不同資源類型、多個資源群組與不同訂用帳戶之計量的圖表。 您可以套用維度篩選器與分組功能來自訂多維度計量圖表。 任何圖表 (包括自訂圖表) 都可以釘選到儀表板。

若您要尋找只支援不含任何維度之基本計量的舊體驗相關資訊，請參閱 [Microsoft Azure 計量概觀指南](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics)中的＜透過入口網站存取計量＞一節。

## <a name="what-is-azure-monitor-metrics-explorer"></a>Azure 監視器計量瀏覽器是什麼？

Azure 監視器計量瀏覽器是 Microsoft Azure 入口網站的一個元件，它可讓您繪製圖表、以視覺方式串連趨勢，以及調查計量值中的突增值與突降值。 計量瀏覽器是調查由 Azure 裝載或由 Azure 監視服務監視之應用程式與基礎結構中各種效能與可用性問題的起點。 

## <a name="what-are-metrics-in-azure"></a>Azure 中的計量是什麼？

Microsoft Azure 中的計量是隨時間收集並儲存的一系列度量與計數值。 計量包括標準 (或稱為「平台」) 計量與自訂計量。 標準計量是由 Azure 平台本身提供給您使用。 標準計量反映您 Azure 資源的健康情況與使用情形統計資料。 而自訂計量則是由您的應用程式使用[適用於自訂事件的 Application Insights API ](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics) 傳送給 Azure。 自訂計量是隨著其他應用程式特定計量儲存在 Application Insights 資源中。



## <a name="how-do-i-create-a-new-chart"></a>如何建立新圖表？

   > [!NOTE]
   > 舊計量體驗的某些功能在新的計量瀏覽器中還無法使用。 雖然新體驗目前處於預覽狀態，但您可以繼續使用 Azure 監視器的舊 (無維度) 計量檢視。 

1. 開啟 Azure 入口網站
2. 瀏覽到新的 [監視器] 索引標籤，然後選取 [計量 (預覽)]。

   ![計量預覽影像](./media/monitoring-metric-charts/0001.png)

3. 系統將會自動為您開啟計量選取器。 從清單選擇資源以檢視其關聯計量。 清單中只會顯示具有計量的資源。

   ![計量預覽影像](./media/monitoring-metric-charts/0002.png)

   > [!NOTE]
   >若您有多個 Azure 訂用帳戶，計量瀏覽器會提取 [入口網站設定] > [依訂用帳戶篩選] 清單中選取之所有訂用帳戶的資源。 若要變更它，請按一下畫面頂端的入口網站設定齒輪圖示，然後選取您要使用的訂用帳戶。

4. 針對某些資源類型 (例如儲存體帳戶與虛擬機器)，您必須先選擇 [命名空間] 再選取計量。 每個命名空間都具有自己的計量組 (只與此命名空間相關，未與其他命名空間相關)。

   例如，每個 Azure 儲存體都有子服務「Blob」、「檔案」、「佇列」與「資料表」(這些都是儲存體帳戶的一部分) 的計量。 不過，「佇列訊息計數」計量本質上即適用於「佇列」子服務，而非任何其他儲存體帳戶子服務。

   ![計量預覽影像](./media/monitoring-metric-charts/0003.png)

5. 從清單選取計量。 若您知道您要尋找之計量的部分名稱，您可以開始輸入以查看已篩選的可用計量清單：

   ![計量預覽影像](./media/monitoring-metric-charts/0004.png)

6. 選取計量之後，圖表將會以所選計量的預設彙總來呈現。 此時，您只能按一下計量選取器以外的位置以將它關閉。 您也可以將圖表切換為不同的彙總。 針對某些計量，切換彙總可讓您選擇要在圖表上看到的值。 例如，您可以在平均、最小值與最大值之間切換。 

7. 透過按一下 [加入計量] 圖示 ![計量圖示](./media/monitoring-metric-charts/icon001.png) 並重複步驟 3-6 即可在相同圖表上加入更多計量。

   > [!NOTE]
   > 您一般不會想要有具有不同計量單位的計量 (例如「毫秒」與 “KB”) 或其刻度在單一圖表上會呈現大幅差異的計量。 如果是這樣，您應該考慮使用多個圖表。 按一下 [新增圖表] 按鈕以在計量瀏覽器中建立多個圖表。

## <a name="how-do-i-apply-filters-to-the-charts"></a>如何將篩選器套用到圖表？

您可以將篩選器套用到以多個維度顯示計量的圖表。 例如，若「交易計數」計量有「回應類型」維度 (指出來自交易的回應成功或失敗)，則在此維度上篩選將只為成功 (或失敗) 的交易繪製圖表線條。 

### <a name="to-add-a-filter"></a>加入篩選器：

1. 按一下 [加入篩選] 圖示 ![篩選器圖示](./media/monitoring-metric-charts/icon002.png) (位於圖表上方)

2. 選取您要篩選的維度 (屬性)

   ![計量影像](./media/monitoring-metric-charts/0006.png)

3. 選取當繪製圖表時要包括的維度值 (此範例會顯示篩選出成功的交易)：

   ![計量影像](./media/monitoring-metric-charts/0007.png)

4. 選取篩選器值之後，請按一下 [篩選器選取器] 以外的位置以將它關閉。 現在圖表會顯示失敗的儲存體交易數目：

   ![計量影像](./media/monitoring-metric-charts/0008.png)

5. 您可以重複步驟 1-4 以將多個篩選器套用到相同的圖表。

## <a name="how-do-i-segment-a-chart"></a>如何將圖表劃分為不同的部分？

您可以依維度來分割計量，以視覺化方式顯示維度的各區段與彼此之間的差異，並識別維度中與其他區段差異較大的區段。 

### <a name="to-segment-a-chart"></a>將圖表劃分為不同的區段：

1. 按一下 [加入群組] 圖示  ![計量影像](./media/monitoring-metric-charts/icon003.png) (位於圖表上方)。
 
   > [!NOTE]
   > 一個圖表上可以有多個篩選器，但只能有一個分組。

2. 選擇要據以將圖表分割為不同區段的維度： 

   ![計量影像](./media/monitoring-metric-charts/0010.png)

   現在圖表會顯示多行，每個維度區段一行：

   ![計量影像](./media/monitoring-metric-charts/0012.png)

3. 按一下 [分組選取器] 以外的位置以將它關閉。

   > [!NOTE]
   > 在相同維度上同時使用篩選與分組，以隱藏與您的案例不相關的區段，讓圖表更容易閱讀。

## <a name="how-do-i-pin-charts-to-dashboards"></a>如何將圖表釘選到儀表板？

設定圖表之後，您可以將它加入到儀表板，以便您可以在其他監視遙測情況下重新檢視它，或與您的小組共用。 

將已設定的圖表釘選到儀表板：

設定您的圖表之後，按一下圖表右上角的 [圖表動作] 功能表，然後按一下 [釘選到儀表板]。

   ![計量影像](./media/monitoring-metric-charts/0013.png)

## <a name="next-steps"></a>後續步驟

  閱讀[建立自訂 KPI 儀表板](https://docs.microsoft.com/azure/application-insights/app-insights-tutorial-dashboards)以了解使用計量建立可採取動作之儀表板的最佳做法。