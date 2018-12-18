---
title: 適用於車輛健全狀況與駕駛習慣的 Power BI 儀表板 - Azure | Microsoft Docs
description: 使用 Cortana Intelligence 具備的強大功能，取得關於車輛健全狀態與駕駛習慣的即時預測情資。
services: machine-learning
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: aaeb29a5-4a13-4eab-bbf1-885690d86c56
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2018
ms.author: deguhath
ms.openlocfilehash: e6601093577eb9e3dfba4ed27e1e0510cad17de7
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421044"
---
# <a name="vehicle-telemetry-analytics-solution-template-power-bi-dashboard-setup-instructions"></a>車輛遙測分析方案範本 Power BI 儀表板安裝指示
此功能表連結至此腳本的章節： 

[!INCLUDE [cap-vehicle-telemetry-playbook-selector](../../../includes/cap-vehicle-telemetry-playbook-selector.md)]

本車輛遙測分析解決方案示範汽車經銷商、汽車製造商及保險公司如何使用 Cortana Intelligence 的功能。 取得車輛健全狀況和駕駛習慣的即時和預測性深入解析，從而改善客戶經驗、研究和開發以及行銷活動。 這些逐步指示說明在您的訂用帳戶中部署方案之後，如何設定 Power BI 報告和儀表板。 

## <a name="prerequisites"></a>必要條件
* 部署[車輛遙測分析](https://gallery.cortanaintelligence.com/Solution/5bdb23f3abb448268b7402ab8907cc90) /(英文/) 解決方案。 
* [安裝 Power BI Desktop](http://www.microsoft.com/download/details.aspx?id=45331)。
* 取得 [Azure 訂用帳戶](https://azure.microsoft.com/pricing/free-trial/)。 如果您沒有 Azure 訂用帳戶，請立即取得免費的 Azure 訂用帳戶。
* 開啟 Power BI 帳戶。

## <a name="cortana-intelligence-suite-components"></a>Cortana Intelligence Suite 元件
車輛遙測分析解決方案範本會在您的訂用帳戶中部署下列 Cortana Intelligence 服務：

* **Azure 事件中樞**可將數百萬計的車輛遙測事件擷取至 Azure。
* **Azure 串流分析**可提供關於車輛健全況的即時深入解析，同時以長期儲存的方式保存這些資料，供日後進行更豐富廣泛的批次分析之用。
* **Azure Machine Learning** 可即時偵測異常狀況，並使用批次處理來提供預測性深入解析。
* **Azure HDInsight** 可大規模轉換資料。
* **Azure Data Factory** 可執行批次處理管線的協調流程、排程、資源管理及監控工作。

**Power BI** 為此解決方案提供具備資料與預測性分析視覺效果等豐富功能的儀表板。 

此方案使用兩種不同的資料來源：

* 模擬車輛訊號和診斷資料集
* 車輛目錄

此方案包含車輛遠程資訊服務模擬器。 它會在指定時間點發出對應於車輛狀態與駕駛模式的診斷資訊和訊號。 

車輛目錄是將 VIN 對應至模型的參考資料集。

## <a name="power-bi-dashboard-preparation"></a>Power BI 儀表板準備
### <a name="set-up-the-power-bi-real-time-dashboard"></a>設定 Power BI 即時儀表板

#### <a name="start-the-real-time-dashboard-application"></a>啟動即時儀表板應用程式
完成部署之後，請遵循手動操作指示進行。

1. 下載即時儀表板應用程式 RealtimeDashboardApp.zip，並將它解壓縮。

1.  在解壓縮的資料夾中，開啟 應用程式組態檔 RealtimeDashboardApp.exe.config。以手動操作指示中的值取代事件中樞的 appSettings、Azure Blob 儲存體及 Azure Machine Learning 服務連線。 儲存您的變更。

1. 執行應用程式 RealtimeDashboardApp.exe。 在登入視窗中，輸入有效的 Power BI 認證。 

   ![Power BI 登入視窗](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/5-sign-into-powerbi.png)
   
1. 選取 [接受]。 應用程式隨即開始執行。

   ![Power BI 儀表板權限](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/6-powerbi-dashboard-permissions.png)

1. 登入 Power BI 網站，並建立即時儀表板。

您現在已經準備好設定 Power BI 儀表板。  

### <a name="configure-power-bi-reports"></a>設定 Power BI 報告
即時報告和儀表板需要大約 30 至 45 分鐘的時間才能完成。 

1. 瀏覽至 [Power BI](http://powerbi.com) 網頁並登入。

    ![Power BI 登入頁面](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/6-1-powerbi-signin.png)

1. Power BI 中會產生新的資料集。 選取 **ConnectedCarsRealtime** 資料集。

    ![ConnectedCarsRealtime 資料集](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/7-select-connected-cars-realtime-dataset.png)

1. 按 Ctrl+S，儲存空白報告。

    ![空白報告](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/8-save-blank-report.png)

1. 輸入報告名稱「即時車輛遙測分析 - 報告」。

    ![報告名稱](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/9-provide-report-name.png)

## <a name="real-time-reports"></a>即時報告
此解決方案有三份即時報告：

* 行駛中的車輛
* 需要維修的車輛
* 車輛健全狀況統計資料

您可以設定上述這三種報告，或在任何階段後停止。 您接著可以進入下一節，盡如如何設定批次報告。 建議您建立上述三份報表，以將解決方案即時路徑的完整深入解析視覺化。  

### <a name="vehicles-in-operation-report"></a>行駛中的車輛報告
1. 按兩下 [第 1 頁]，重新命名為**行駛中的車輛**。

    ![行駛中的車輛](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4a.png)  

1. 在 [欄位] 索引標籤上，選取 **vin**。 在 [視覺效果] 索引標籤上，選取 [卡片] 視覺效果。  

    建立的**卡片**視覺效果如下圖所示：

    ![選取 vin](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4b.png)

1. 選取空白區域以加入新的視覺效果。  

1. 在 [欄位] 索引標籤上，選取 **city** 和 **vin**。 在 [視覺效果] 索引標籤上，選取 [地圖] 視覺效果。 將 **vin** 拖曳至 [值] 區域。 將 **city** 拖曳至 [圖例] 區域。 

    ![卡片視覺效果](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4c.png)

1. 在 [視覺效果] 索引標籤上，選取 [格式] 區段。 選取 [標題]，然後將 [文字] 變更為**行駛中的車輛 (依城市)**。

    ![行駛中的車輛 (依城市)](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4d.png)   

    最終的視覺效果看起來如下列範例所示：

    ![最終的視覺效果](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4e.png)

1. 選取空白區域以加入新的視覺效果。  

1. 在 [欄位] 索引標籤上，選取 **city** 和 **vin**。 在 [視覺效果] 索引標籤上，選取 [群組直條圖] 視覺效果。 將 **city** 拖曳至 [軸] 區域。 將 **vin** 拖曳至 [值] 區域。

1. 依 [vin 的計數] 排序圖表。

    ![vin 的計數](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4f.png)  

1. 將圖表 [標題] 變更為**行駛中的車輛 (依城市)**。 

1. 選取 [格式] 區段，然後選取 [資料色彩]。 將 [全部顯示] 變更為 [開啟]。

    ![資料色彩](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4g.png)  

1. 依照選取的色彩符號變更個別城市的色彩。

    ![色彩變化](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4h.png)  

1. 選取空白區域以加入新的視覺效果。  

1. 在 [視覺效果] 索引標籤上，選取 [群組直條圖] 視覺效果。 在 [欄位] 索引標籤上，將 **city** 拖曳至 [軸] 區域。 將 **Model** 拖曳至 [圖例] 區域。 將 **vin** 拖曳至 [值] 區域。

    ![群組直條圖](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4i.png)

    圖表看起來如下圖所示：

    ![轉譯](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4j.png)

1. 重新排列所有視覺效果，讓頁面看起來如下列範例所示：

    ![儀表板與視覺效果](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4k.png)

您已成功設定「行駛中的車輛」報告。 您可以建立下一份即時報告，或可以在此停止並設定儀表板。 

### <a name="vehicles-requiring-maintenance-report"></a>需要維修的車輛報告

1. 選取 [新增]![](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4add.png) 來加入新報告。 將名稱重新命名為**需要維修的車輛**。

    ![需要維修的車輛](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4l.png)  

1. 在 [欄位] 索引標籤上，選取 **vin**。 在 [視覺效果] 索引標籤上，選取 [卡片] 視覺效果。

    ![Vin 卡片視覺效果](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4m.png)  

    資料集中含有一個名為 **MaintenanceLabel** 的欄位。 此欄位的值可以是 "0" 或 "1"。 此值是隨解決方案佈建的機器學習模型所設定。 與即時路徑整合。 值為 "1" 表示車輛需要維修。 

1. 若要加入 [頁面層級篩選] 以顯示需要維修的車輛資料： 

   a. 將 **MaintenanceLabel** 欄位拖曳到 [頁面層級篩選]。
  
      ![頁面層級篩選](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4n1.png)

    b. 在 [頁面層級篩選 MaintenanceLabel] 底部，選取 [基本篩選]。

      ![基本篩選](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4n2.png) 

    c. 將篩選值設為 **1**。

      ![篩選值](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4n3.png)  

1. 選取空白區域以加入新的視覺效果。  

1. 在 [視覺效果] 索引標籤上，選取 [群組直條圖] 視覺效果。 

    ![Vin 卡片](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4o.png)

    ![群組直條圖](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4p.png)

1. 在 [欄位] 索引標籤上，將 **Model** 拖曳至 [軸] 區域。 將 **vin** 拖曳至 [值] 區域。 然後，依 [vin 的計數] 排序視覺效果。 將圖表 [標題] 變更為**需要維修的車輛 (依車型)**。 

1. 在 [視覺效果] 索引標籤的 [欄位] ![欄位圖像](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4field.png) 區段上，將 **vin** 拖曳至 [色彩飽和度]。

    ![色彩飽和度](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4q.png)  

1. 在 [格式] 區段上，變更視覺效果中的 [資料色彩]： 

    a. 將 [最小值] 色彩變更為 **F2C812**。

    b. 將 [最大值] 色彩變更為 **FF6300**。

    ![新的資料色彩](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4r.png)

    新的視覺效果色彩看起來如下範例所示：

    ![新的視覺效果色彩](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4s.png)  

1. 選取空白區域以加入新的視覺效果。  

1. 在 [視覺效果] 索引標籤上，選取 [群組直條圖]。 將 **vin** 拖曳至 [值] 區域。 將 **city** 拖曳至 [軸] 區域。 依 [vin 的計數] 排序圖表。 將圖表 [標題] 變更為**需要維修的車輛 (依城市)**。

    ![需要維修的車輛 (依城市)](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4t.png)  

1. 選取空白區域以加入新的視覺效果。  

1. 在 [視覺效果] 索引標籤上，選取 [多列卡片] 視覺效果。 將 **Model** 和 **vin** 拖曳至 [欄位] 區域。

    ![多列卡片](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4u.png)    

1. 重新排列所有視覺效果，讓最終的報告看起來如下列範例所示： 

    ![重新排列最終的報告](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4v.png)  

您已成功設定「需要維修的車輛」即時報告。 您可以建立下一份即時報告，或可以在此停止並設定儀表板。 

### <a name="vehicle-health-statistics-report"></a>車輛健全狀況統計資料報告

1. 選取 [新增]![](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4add.png) 來加入新報告。 將名稱重新命名為**車輛健全狀況統計資料**。 

1. 在 [視覺效果] 索引標籤上，選取 [量測計] 視覺效果。 將 **speed** 拖曳至 [值]、[最小值] 及 [最大值] 區域。

   ![車輛健全狀況統計資料](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4w.png)  

1. 在 [值] 區域中，將 **speed** 的預設彙總變更為 [平均值]。

1. 在 [最小值] 區域中，將 **speed** 的預設彙總變更為 [最小值]。

1. 在 [最大值] 區域中，將 **speed** 的預設彙總變更為 [最大值]。

   ![車速值](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4x.png)  

1. 將 [量測計標題] 重新命名為**平均車速**。

   ![量測計](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4y.png)  

1. 選取空白區域以加入新的視覺效果。  

    同樣地，加入**平均機油**、**平均油量**和**平均引擎溫度**的 [量測計]。  

1. 如上述在**平均車速**量測計的步驟，變更各個量測計欄位的預設彙總。

    ![其他量測計](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4z.png)

1. 選取空白區域以加入新的視覺效果。

1. 在 [視覺效果] 索引標籤上，選取 [折線與群組直條圖] 視覺效果。 將 **city** 拖曳至 [共用軸]。 將 **tirepressure**、**engineoil** 及 **speed** 拖曳至 [資料行值] 區域。 將其彙總類型變更為 [平均值]。 

1. 將 **engineTemperature** 拖曳至 [折線圖值] 區域。 將彙總類型變更為 [平均值]。 

    ![資料行和折線圖值](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4aa.png)

1. 將圖表**標題**變更為**平均車速、胎壓、機油和引擎溫度**。  

    ![折線圖和群組直條圖標題](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4bb.png)

1. 選取空白區域以加入新的視覺效果。

1. 在 [視覺效果] 索引標籤上，選取 [樹狀圖] 視覺效果。 將 **Model** 拖曳至 [群組] 區域。 將 **MaintenanceProbability** 拖曳至 [值] 區域。

1. 將圖表**標題**變更為**需要維修的車型**。

    ![樹狀圖標題](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4cc.png)

1. 選取空白區域以加入新的視覺效果。

1. 在 [視覺效果] 索引標籤上，選取 [100% 堆疊橫條圖] 視覺效果。 將 **city** 拖曳至 [軸] 區域。 將 **MaintenanceProbability** 和 **RecallProbability** 拖曳至 [值] 區域。

    ![軸和值區域](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4dd.png)

1. 在 [格式] 區段上，選取 [資料色彩]。 將 **MaintenanceProbability** 色彩設定為值 **F2C80F**。

1. 將圖表**標題**變更為**車輛維修和召回的機率 (依城市)**。

    ![100% 堆疊橫條圖標題](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4ee.png)

1. 選取空白區域以加入新的視覺效果。

1. 在 [視覺效果] 索引標籤上，選取 [區域圖] 視覺效果。 將 **Model** 拖曳至 [軸] 區域。 將 **engineOil**、**tirepressure**、**speed** 及 **MaintenanceProbability** 拖曳至 [值] 區域。 將其彙總類型變更為 [平均值]。 

    ![彙總類型](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4ff.png)

1. 將圖表的**標題**變更為**平均機油、胎壓、車速和維修機率 (依車型)**。

    ![區域圖標題](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4gg.png)

1. 選取空白區域以加入新的視覺效果。

1. 在 [視覺效果] 索引標籤上，選取 [散佈圖] 視覺效果。 將 **Model** 拖曳至 [詳細資料] 和 [圖例] 區域。 將 **fuel** 拖曳至 [X 軸] 區域。 將彙總變更為 [平均值]。 將 **engineTemperature** 拖曳至 [Y 軸] 區域。 將彙總變更為 [平均值]。 將 **vin** 拖曳至 [大小] 區域。

    ![詳細資料、圖例、軸和大小區域](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4hh.png)

1. 將圖表**標題**變更為**平均油量、平均引擎溫度和 vin 的計數 (依車型)**。

    ![散佈圖標題](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4ii.png)

    最終的報告看起來如下列範例所示：

    ![最終的報告](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.4jj.png)

### <a name="pin-visualizations-from-the-reports-to-the-real-time-dashboard"></a>將報告中的視覺效果釘選到即時儀表板
1. 選取 [儀表板] 旁邊的加號符號，以建立空白儀表板。 將名稱輸入為**車輛遙測分析儀表板**。

    ![儀表板加號符號](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.5.png)

1. 將先前報告中的視覺效果釘選到儀表板。 

    ![儀表板釘選符號](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-3.6.png)

    三份報告都釘選到儀表板時，看起來應該如下列範例所示。 如果您尚未建立所有報告，儀表板看起來可能有所不同。 

    ![儀表板與報告](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/connected-cars-4.0.png)

您已成功建立即時儀表板。 隨著您繼續執行 CarEventGenerator.exe 和 RealtimeDashboardApp.exe，便會在儀表板上看見即時更新。 下列步驟需要約 10 至 15 分鐘的時間才能完成。

## <a name="set-up-the-power-bi-batch-processing-dashboard"></a>設定 Power BI 批次處理儀表板
> [!NOTE]
> 端對端批次處理管線大約需要兩小時 (從成功完成部署開始算起) 的時間才能完成執行並處理一年份的已產生資料。 請等處理完成後再繼續進行下列步驟：
> 
> 

### <a name="download-the-power-bi-designer-file"></a>下載 Power BI 設計工具檔案

1. 預先設定的 Power BI 設計工具檔案包含在部署手動操作說明中。 尋找「2. 設定 Power BI 批次處理儀表板。」

1. 下載批次處理儀表板的 Power BI 範本，在此稱為 **ConnectedCarsPbiReport.pbix**。

1. 儲存在本機。

### <a name="configure-power-bi-reports"></a>設定 Power BI 報告

1. 使用 Power BI Desktop，開啟設計工具檔案 **ConnectedCarsPbiReport.pbix**。 如果您還沒有安裝 Power BI Desktop，請從 [Power BI Desktop 安裝](http://www.microsoft.com/download/details.aspx?id=45331)網站進行安裝。

1. 選取 [編輯查詢]。

    ![編輯查詢](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/10-edit-powerbi-query.png)

1. 按兩下 [來源]。

    ![來源](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/11-set-powerbi-source.png)

1. 使用在部署時佈建的 Azure SQL Server 更新伺服器連接字串。 在 Azure SQL Database 查看手動操作說明：

    * 伺服器︰somethingsrv.database.windows.net
    * 資料庫︰connectedcar
    * 使用者名稱︰使用者名稱
    * 密碼︰您可以從 Azure 入口網站管理您的 SQL Server 密碼。

1. 將 [資料庫]  保留為 **connectedcar**。

    ![資料庫](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/12-set-powerbi-database.png)

1. 選取 [確定] 。

1. 預設會選取 [Windows 認證] 索引標籤。 選取右側的 [資料庫] 索引標籤來變更為 [資料庫認證]。

1. 輸入在 Azure SQL Database 部署安裝過程中提供的 [使用者名稱] 和 [密碼]。

    ![資料庫認證](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/13-provide-database-credentials.png)

1. 選取 [ **連接**]。

1. 針對出現在右窗格中的其餘三個查詢逐一重複前述步驟。 然後更新資料來源連接詳細資料。

1. 選取 [關閉並載入]。 Power BI Desktop 檔案資料集會連接到 SQL 資料庫資料表。

1. 選取 [關閉] 以關閉 Power BI Desktop 檔案。

    ![關閉](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/14-close-powerbi-desktop.png)

1. 按一下 [儲存] 以儲存變更。 

您現在已設定對應至解決方案中批次處理路徑的所有報告。 

## <a name="upload-to-powerbicom"></a>上傳至 powerbi.com
1. 前往 [Power BI 網頁入口網站](http://powerbi.com)並登入。

1. 選取 [取得資料]。

1. 上傳 Power BI Desktop 檔案。 選取 [取得資料] > [檔案取得] > [本機檔案]。

1. 移至 **ConnectedCarsPbiReport.pbix**。

1. 上傳檔案之後，返回您的 Power BI 工作區。 隨即為您建立資料集、報告及空白儀表板。  

1. 將圖表釘選至 Power BI 中的新儀表板，名為**車輛遙測分析儀表板**。 選取先前建立的空白儀表板，然後移至 [報告] 區段。 選取新上傳的報告。  

    ![新的 Power BI 儀表板](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard1.png) 

    報告共有六頁：

    第 1 頁︰車輛密度  
    第 2 頁︰即時車輛健全狀況  
    第 3 頁︰危險駕駛的車輛   
    第 4 頁︰召回的車輛  
    第 5 頁︰省油的車輛  
    第 6 頁：Contoso Motors 標誌  

    ![共有六頁的 Power BI 報告](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard2.png)

1. 釘選**第 3 頁**的下列內容：  

    a. **vin 的計數**  

   ![第 3 頁 vin 的計數](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard3.png)

    b. **危險駕駛的車輛 (依車型) – 瀑布圖** 

   ![第 3 頁圖表 4](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard4.png)

1. 釘選**第 5 頁**的下列內容： 

    a. **vin 的計數**

   ![第 5 頁圖表 5](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard5.png)

    b. **省油的車輛 (依車型)：群組直條圖**

   ![第 5 頁圖表 6](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard6.png)

1. 釘選**第 4 頁**的下列內容：  

    a. **vin 的計數** 

   ![第 4 頁圖表 7](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard7.png) 

    b. **召回的車輛 (依城市、車型)：樹狀圖**

   ![第 4 頁圖表 8](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard8.png)  

1. 釘選**第 6 頁**的下列內容：  

    * **Contoso Motors 標誌**

    ![Contoso Motors 標誌 ](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard9.png)

### <a name="organize-the-dashboard"></a>組織儀表板  

1. 移至儀表板。

1. 將游標停留在每個圖表上。 根據下列已完成儀表板範例提供的名稱，重新命名每個圖表：

   ![儀表板組織](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-organize-dashboard2.png) 
   
1. 四處移動圖表以如下列儀表板範例所示來排列：

    ![重新排列儀表板](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-dashboard.png)

1. 本文所述的報告全部建立之後，最後完成的儀表板看起來會如下列範例所示： 

   ![最終的儀表板](./media/cortana-analytics-playbook-vehicle-telemetry-powerbi-dashboard/vehicle-telemetry-organize-dashboard3.png)

您已成功建立報告和儀表板，可取得車輛健全狀況和駕駛習慣的即時、預測性和批次深入解析。  
