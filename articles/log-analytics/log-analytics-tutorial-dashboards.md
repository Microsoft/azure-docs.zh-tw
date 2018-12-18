---
title: 建立和共用 Azure Log Analytics 資料的儀表板 | Microsoft Docs
description: 本教學課程可幫助您了解 Log Analytics 儀表板如何以視覺化方式呈現所有儲存的記錄搜尋，讓您以單一方式檢視您的環境。
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: abb07f6c-b356-4f15-85f5-60e4415d0ba2
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/14/2017
ms.author: magoedte
ms.custom: mvc
ms.component: na
ms.openlocfilehash: 2c7c1e71830eb3fe4f7240195dac828ac9dfed3a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951556"
---
# <a name="create-and-share-dashboards-of-log-analytics-data"></a>建立和共用 Log Analytics 資料的儀表板

Log Analytics 儀表板可以視覺化方式呈現所有儲存的記錄搜尋，讓您可以尋找、相互關聯，並共用組織中的 IT 作業資料。  本教學課程涵蓋建立記錄搜尋，其將用來支援由您的 IT 作業支援團隊存取的共用儀表板。  您會了解如何：

> [!div class="checklist"]
> * 在 Azure 入口網站中建立共用儀表板
> * 以視覺化方式呈現效能記錄搜尋 
> * 將記錄搜尋新增至共用儀表板 
> * 在共用儀表板中自訂圖格

若要完成本教學課程中的範例，您目前必須有[連線至 Log Analytics 工作區](log-analytics-quick-collect-azurevm.md)的虛擬機器。  
 
## <a name="log-in-to-azure-portal"></a>登入 Azure 入口網站
在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。 

## <a name="create-a-shared-dashboard"></a>建立共用儀表板

您登入 Microsoft Azure 入口網站 之後最先看到的是[儀表板](../azure-portal/azure-portal-dashboards.md)。<br> ![Azure 入口網站儀表板](media/log-analytics-tutorial-dashboards/log-analytics-portal-dashboard.png)

您可以在這裡將所有 Azure 資源中對 IT 最重要的作業資料結合在一起，包括來自 Azure Log Analytics 的遙測資料。  在我們進入以視覺化方式呈現記錄搜尋之前，請先建立並共用儀表板。  完成此需求之後，才能利用範例效能記錄搜尋，其中將轉譯為折線圖並加入儀表板。  

若要建立儀表板，請選取目前的儀表板名稱旁的 [新增儀表板]  按鈕。<br> ![在 Azure 入口網站中建立新儀表板](media/log-analytics-tutorial-dashboards/log-analytics-create-dashboard-01.png)

這個動作會建立全新的空白私人儀表板並讓您進入自訂模式，供您為儀表板命名並新增或重新排列圖格。 編輯儀表板名稱，並指定此教學課程的「範例儀表板」，然後選取 [自訂完成]。<br><br> ![儲存自訂的 Azure 儀表板](media/log-analytics-tutorial-dashboards/log-analytics-create-dashboard-02.png)

當您建立儀表板時，它預設是私人用途，這表示只有您會看得到。 若要讓其他人也能看見，請使用與其他儀表板命令一同出現的 [共用]  按鈕。<br> ![在 Azure 入口網站中共用新儀表板](media/log-analytics-tutorial-dashboards/log-analytics-share-dashboard.png) 

系統會要求您選擇可將儀表板發佈至的訂用帳戶和資源群組。 為了方便起見，入口網站的發佈體驗會引導您朝向將儀表板放在稱為 **儀表板**的資源群組中的模式。  確認選取的訂用帳戶，然後按一下[發行]。  使用[Azure 資源存取控制](../role-based-access-control/role-assignments-portal.md)來控制對儀表板中顯示資訊的存取權。   

## <a name="visualize-a-log-search"></a>以視覺化方式呈現記錄搜尋

您可以在 Azure 入口網站中從記錄搜尋入口網站的任一行上建立基本查詢。 記錄搜尋入口網站不必啟動外部入口網站就可以使用，它可以透過記錄搜尋來執行各種函式，包括建立警示規則、建立電腦群組，以及匯出查詢結果。 

[Log Analytics 入口網站](../log-analytics/query-language/get-started-analytics-portal.md)是一個專用的入口網站，提供記錄搜尋入口網站中無法使用的進階功能。 功能包括在多行上編輯查詢、選擇性執行程式碼、區分內容的 Intellisense 和 Smart Analytics。 在進階 Analytics 入口網站中，您將會以圖形的形式建立效能檢視，儲存以用於未來搜尋，然後將其釘選在稍早建立的共用儀表板。   

您可從記錄搜尋入口網站中的連結啟動進階 Analytics 入口網站。<br> ![啟動進階 Analytics 入口網站](media/log-analytics-tutorial-dashboards/log-analytics-advancedportal-01.png)

在 Analytics 入口網站中，輸入下列查詢，只為 Windows 和 Linux 電腦傳回處理器使用率記錄，依電腦和 TimeGenerated 分組並顯示在視覺化圖表中：

```
Perf | where CounterName == "% Processor Time" and ObjectName == "Processor" and InstanceName == "_Total" | summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1m), Computer | render timechart
```

從右上角選取 [儲存查詢]按鈕來儲存查詢。<br> ![從進階 Analytics 入口網站儲存查詢](media/log-analytics-tutorial-dashboards/log-analytics-advancedportal-02.png)<br><br> 在 [儲存查詢] 控制台中，提供像是「Azure VM - 處理器使用率」的名稱，然後按一下 [儲存]。  如此一來，您可以建立要用以搜尋的常用查詢程式庫，或予以修改，而不用全部重寫。  最後，從頁面中間靠右的角落選取 [將圖表釘選到 Azure 儀表板]，將此釘選在稍早建立的共用儀表板。  

現在我們已將查詢釘選在儀表板，您會注意到其下方會有一般標題與註解。<br> ![Azure 儀表板範例](media/log-analytics-tutorial-dashboards/log-analytics-modify-dashboard-01.png)<br><br>  我們應將它重新命名為有意義的名稱，讓檢視者容易了解。  以滑鼠右鍵按一下圖格，並選取 [編輯圖格]。  當您完成自訂圖格的標題和副標題，請按一下 [更新]。  橫幅隨即出現，要求您發行變更或捨棄。  按一下 [發行變更]，然後關閉 [編輯圖格] 控制窗格。  

![已完成儀表板範例的設定](media/log-analytics-tutorial-dashboards/log-analytics-modify-dashboard-02.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何在 Azure 入口網站中建立儀表板，並加入記錄搜尋。  前進至下一個教學課程，以了解您可以根據記錄搜尋結果實作的不同回應。  

> [!div class="nextstepaction"]
> [使用 Log Analytics 警示來回應事件](log-analytics-tutorial-response.md)