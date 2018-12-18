---
title: 將 Azure Log Analytics 資料匯入至 Power BI | Microsoft Docs
description: Power BI 是 Microsoft 的雲端架構商務分析服務，能提供豐富的視覺效果和不同資料集的分析報告。  本文說明如何設定將 Log Analytics 資料匯入 Power BI，並將它設定為自動重新整理。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: 83edc411-6886-4de1-aadd-33982147b9c3
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: fb05ddabab3702299df0e81e8dda5af5cb676c1a
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37127504"
---
# <a name="import-azure-log-analytics-data-into-power-bi"></a>將 Azure Log Analytics 資料匯入至 Power BI


[Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/) 是 Microsoft 的雲端架構商務分析服務，能提供豐富的視覺效果和不同資料集的分析報告。  您可以將 Log Analytics 記錄搜尋結果匯入 Power BI 資料集，以便運用其功能，例如可合併不同來源的資料，以及共用網路和行動裝置上的資料。

## <a name="overview"></a>概觀
若要將 Log Analytics 工作區的資料匯入到 Power BI，您必須按照 Log Analytics 的記錄搜尋查詢在 Power BI 中建立資料集。  每次資料集重新整理時，就會執行查詢。  然後，您即可使用資料集中的資料建立 Power BI 報表。  若要在 Power BI 中建立資料集，您必須從 Log Analytics 將您的查詢匯出到 [Power Query (M) 語言](https://msdn.microsoft.com/library/mt807488.aspx)。  接著可以用它在 Power BI Desktop 中建立查詢，然後將它發佈至 Power BI 做為資料集。  以下將詳細說明此流程。

![Log Analytics 到 Power BI](media/log-analytics-powerbi/overview.png)

## <a name="export-query"></a>匯出查詢
首先建立[記錄搜尋](log-analytics-log-search-new.md)，傳回您想要填入 Power BI 資料集的 Log Analytics 資料。  您接著可以將該查詢匯出到可以由 Power BI Desktop 使用的 [Power Query (M) 語言](https://msdn.microsoft.com/library/mt807488.aspx)。

1. 在 Log Analytics 中建立記錄搜尋，擷取您資料集的資料。
2. 如果您使用記錄搜尋入口網站，請按一下 **Power BI**。  如果您使用 Analytics 入口網站，請選取**匯出** > **Power BI 查詢 (M)**。  這兩個選項都會將查詢匯出到稱為 **PowerBIQuery.txt** 的文字檔。 

    ![匯出記錄搜尋](media/log-analytics-powerbi/export-logsearch.png) ![匯出記錄搜尋](media/log-analytics-powerbi/export-analytics.png)

3. 開啟該文字檔，然後複製其中的內容。

## <a name="import-query-into-power-bi-desktop"></a>將查詢匯入至 Power BI Desktop
Power BI Desktop 是桌面應用程式，可讓您建立可發行至 Power BI 的資料集和報表。  您也可以用它來建立查詢，使用從 Log Analytics 匯出的 Power Query 語言。 

1. 請安裝 [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (如果尚未安裝)，然後開啟該應用程式。
2. 選取**取得資料** > **空白查詢**開啟新查詢。  然後選取**進階編輯器**，並在查詢中貼上匯出的檔案內容。 按一下 [完成] 。

    ![Power BI Desktop 查詢](media/log-analytics-powerbi/desktop-new-query.png)

5. 執行查詢並顯示其結果。  可能會提示您提供連接到 Azure 的認證。  
6. 輸入查詢的描述性名稱。  預設值是 **Query1**。 按一下**關閉並套用**，將資料集新增至報表。

    ![Power BI Desktop 名稱](media/log-analytics-powerbi/desktop-results.png)



## <a name="publish-to-power-bi"></a>發行至 Power BI
您發行至 Power BI 時，會建立資料集和報表。  如果您在 Power BI Desktop 中建立報表，將連同您的資料一起發佈。  如果未建立，則將建立空白報表。  您可以在 Power BI 中修改該報表，也建立依照資料集建立新的報表。

8. 依照資料建立報表。  如果您不熟悉如何進行，請參閱 [Power BI Desktop 文件](https://docs.microsoft.com/power-bi/desktop-report-view)。  您準備將它傳送至 Power BI 時時，請按一下**發行**。  出現提示時，請在 Power BI 帳戶中選取目的地。  除非您有特定的目的地，否則請使用**我的工作區**。

    ![Power BI Desktop 發佈](media/log-analytics-powerbi/desktop-publish.png)

3. 發佈完成時，按一下**在 Power BI 中開啟**，以新的資料集開啟 Power BI。


### <a name="configure-scheduled-refresh"></a>設定排定的重新整理
在 Power BI 中建立的資料集將具有與您先前在 Power BI Desktop 看見的相同資料。  您需要定期重新整理資料集，以便再次執行查詢，並填入 Log Analytics 的最新資料。  

1. 按一下您上傳報表的工作區，並選取**資料集**功能表。 選取新的資料集旁邊的快顯功能表，並選取**設定**。 在**資料來源認證**下，應該會出現認證無效的訊息。  這是因為您尚未提供資料集重新整理其資料時使用的認證。  按一下**編輯認證**，並指定存取 Log Analytics 的認證。

    ![Power BI 排程](media/log-analytics-powerbi/powerbi-schedule.png)

5. 在**排定的重新整理**選項下，開啟**維持資料的最新狀態**。  您可以選擇變更**重新整理頻率**，以及執行重新整理的一個或多個特定時間。

    ![Power BI 重新整理](media/log-analytics-powerbi/powerbi-schedule-refresh.png)



## <a name="next-steps"></a>後續步驟
* 深入了解 [記錄檔搜尋](log-analytics-log-searches.md) ，建置可以匯出至 Power BI 的查詢。
* 深入了解 [Power BI](http://powerbi.microsoft.com)，建置以 Log Analytics 匯出為基礎的視覺效果。
