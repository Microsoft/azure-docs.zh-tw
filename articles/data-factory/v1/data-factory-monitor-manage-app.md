---
title: 監視和管理資料管線 - Azure |Microsoft Docs
description: 了解如何使用監視及管理應用程式來監視及管理 Azure Data Factory 及管線。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: f3f07bc4-6dc3-4d4d-ac22-0be62189d578
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 3f234e49f1a28fd0881e47ede13ae72483ed31f3
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37053111"
---
# <a name="monitor-and-manage-azure-data-factory-pipelines-by-using-the-monitoring-and-management-app"></a>使用監視及管理應用程式，以監視和管理 Azure Data Factory 管線
> [!div class="op_single_selector"]
> * [使用 Azure 入口網站/Azure PowerShell](data-factory-monitor-manage-pipelines.md)
> * [使用監視及管理應用程式](data-factory-monitor-manage-app.md)
>
>

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[監視和管理 Data Factory 管線](../monitor-visually.md)。

本文說明如何使用監視及管理應用程式來監視、管理與偵錯您的 Data Factory 管線。 您可以藉由觀賞下列影片開始使用應用程式：

> [!NOTE]
> 影片中顯示的使用者介面可能不完全符合您在入口網站中看到的使用者介面。 它比較舊，但是概念相同。 

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Azure-Data-Factory-Monitoring-and-Managing-Big-Data-Piplines/player]
>

## <a name="launch-the-monitoring-and-management-app"></a>啟動監視及管理應用程式
若要啟動監視及管理應用程式，針對您的 Data Factory 按一下 [Data Factory] 刀鋒視窗上的 [監視及管理] 圖格。

![Data Factory 首頁上的監視磚](./media/data-factory-monitor-manage-app/MonitoringAppTile.png)

您應該會看到監視及管理應用程式在個別視窗中開啟。  

![監視及管理應用程式](./media/data-factory-monitor-manage-app/AppLaunched.png)

> [!NOTE]
> 如果您看到網頁瀏覽器停滯在「授權中...」，請清除 [封鎖第三方 Cookie 和站台資料] 核取方塊，或保留選取該核取方塊但建立 **login.microsoftonline.com** 的例外狀況，然後再試一次開啟應用程式。


在中間窗格的 [活動時段] 清單中，您會在每次執行活動時看到活動時段。 例如，如果您有活動排程在五個小時內每小時執行，您會看到與五個資料配量與相關聯的五個活動時段。 如果您沒有在底部的清單中看到活動時段，請執行下列作業：
 
- 更新頂端的**開始時間**和**結束時間**篩選以符合管線的開始和結束時間，然後按一下 [套用] 按鈕。  
- [活動時段] 清單不會自動重新整理。 按一下 [活動時段] 清單中工具列上的 [重新整理] 按鈕。  

如果您還沒有用來測試這些步驟的 Data Factory 應用程式，請進行教學課程：[使用 Data Factory 將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

## <a name="understand-the-monitoring-and-management-app"></a>了解監視及管理應用程式
左側有三個索引標籤︰[資源總管]、[監視檢視] 以及 [警示]。 預設會選取第一個索引標籤 ([資源總管])。

### <a name="resource-explorer"></a>資源總管
這時會顯示下列項目：

* 左窗格中的資源總管**樹狀檢視**。
* 中間窗格頂端的**圖表檢視**。
* 中間窗格底部的 [活動時段] 清單。
* 右窗格中的 [屬性]、[活動時段總管] 和 [指令碼] 索引標籤。

在資源總管中，您可以在樹狀檢視的 Data Factory 看到所有資源 (管線、資料集、連結的服務)。 當您在 [資源總管] 中選取物件時：

* 在 [圖表檢視] 中，相關聯的 Data Factory 實體會反白顯示。
* 在底部的 [活動時段] 清單中，[相關聯的活動時段](data-factory-scheduling-and-execution.md)已反白顯示。  
* 右窗格的 [屬性] 視窗中顯示已選取物件的屬性。
* 顯示已選取物件的 JSON 定義 (如果適用的話)。 例如︰連結的服務、資料集或管線。

![資源總管](./media/data-factory-monitor-manage-app/ResourceExplorer.png)

請參閱[排程和執行](data-factory-scheduling-and-execution.md)文章，了解活動時段的詳細概念性資訊。

### <a name="diagram-view"></a>圖表檢視
Data Factory 的圖表檢視提供單一窗格，可用來監視和管理 Data Factory 及其資產。 當您在 [圖表檢視] 中選取 Data Factory 實體 (資料集/管線) 時：

* 在樹狀檢視中，Data Factory 實體已選取。
* 在 [活動時段] 清單中，相關聯的活動時段已反白顯示。
* 在 [屬性] 視窗中，會顯示已選取物件的屬性。

當管線已啟用時 (不在暫停狀態)，它會顯示綠色線條：

![管線執行中](./media/data-factory-monitor-manage-app/PipelineRunning.png)

您可以暫停、繼續或終止管線，方法是在圖表檢視中選取它，以及使用命令列上的按鈕。

![命令列上的暫停/繼續](./media/data-factory-monitor-manage-app/SuspendResumeOnCommandBar.png)
 
[圖表檢視] 中的管線有三個命令列按鈕。 第二個按鈕可以讓您暫停管線。 暫停不會終止正在執行的活動，還會讓活動繼續完成。 第三個按鈕會暫停管線，並終止管線正在執行的活動。 第一個按鈕會讓管線繼續執行。 您的管線暫停時，管線的色彩會變更。 例如，暫停管線看起來像下圖所示： 

![管線已暫停](./media/data-factory-monitor-manage-app/PipelinePaused.png)

您可以使用 Ctrl 鍵，以多重選取兩個以上的管線。 您可以使用命令列按鈕來同時暫停/繼續多個管線。

您也可以用滑鼠右鍵按一下管線，並選取選項來暫停、繼續或終止管線。 

![管線的內容功能表](./media/data-factory-monitor-manage-app/right-click-menu-for-pipeline.png)

按一下 [開啟管線] 選項，查看管線中的所有活動。 

![開啟管線功能表](./media/data-factory-monitor-manage-app/OpenPipelineMenu.png)

在開啟的管線檢視中，您會看到管線裡的所有活動。 在這個範例中只有一個活動：複製活動。 

![已開啟的管線](./media/data-factory-monitor-manage-app/OpenedPipeline.png)

如要回到上一個檢視，請按一下頂端階層連結功能表中的 Data Factory 名稱。

在管線檢視中，當您選取某個輸出資料集時，或當您把滑鼠游標移動到輸出資料集上時，您會看到該資料集的 [活動時段] 快顯視窗。

![活動時段快顯視窗](./media/data-factory-monitor-manage-app/ActivityWindowsPopup.png)

只要按一下某個活動時段，即可在右窗格的 [屬性] 視窗中看到該活動時段的詳細資料。

![活動時段屬性](./media/data-factory-monitor-manage-app/ActivityWindowProperties.png)

在右窗格中，切換到 [活動時段總管] 索引標籤，即可看到更多詳細資料。

![活動時段總管](./media/data-factory-monitor-manage-app/ActivityWindowExplorer.png)

您也可以在 [嘗試次數] 區段中看到每個活動執行嘗試的**解析的變數**。

![解析的變數](./media/data-factory-monitor-manage-app/ResolvedVariables.PNG)

切換至 [指令碼]  索引標籤，以查看所選物件的 JSON 指令碼定義。   

![指令碼索引標籤](./media/data-factory-monitor-manage-app/ScriptTab.png)

您可以在三個地方看到活動時段：

* [圖表檢視] \(中間窗格) 中的 [活動時段] 快顯視窗。
* 右窗格中的 [活動時段總管]。
* 底部窗格中的 [活動時段] 清單。

在 [活動時段] 快顯視窗和 [活動時段總管] 中，您可以使用向左箭號和向右箭號來捲動至前一週及下一週。

![活動時段總管的向左/向右箭號](./media/data-factory-monitor-manage-app/ActivityWindowExplorerLeftRightArrows.png)

在 [圖表檢視] 的底部，您會看到 [放大]、[縮小]、[縮放至適當比例]、[顯示比例 100%]、[鎖定配置] 等按鈕。 [鎖定配置] 按鈕會防止您在 [圖表檢視] 中不小心移動了資料表和管線。 此按鈕預設為開啟。 但您可以關閉該設定，以便移動圖表中的實體。 當您關閉該設定，您可以使用上一個按鈕來自動把資料表和管線移動到適當的地方。 您也可以使用滑鼠滾輪來放大或縮小。

![圖表檢視縮放命令](./media/data-factory-monitor-manage-app/DiagramViewZoomCommands.png)

### <a name="activity-windows-list"></a>活動時段清單
中間窗格底端的 [活動時段] 清單會顯示您在 [資源總管] 或 [圖表檢視] 中所選取資料集的所有活動時段。 根據預設，清單是以遞減的順序排列，這代表清單頂端會是最新的活動時段。

![活動時段清單](./media/data-factory-monitor-manage-app/ActivityWindowsList.png)

這份清單不會自動重新整理，因此請使用工具列上 [重新整理] 按鈕來手動重新整理。  

下列為活動時段的狀態：

<table>
<tr>
    <th align="left">狀態</th><th align="left">子狀態</th><th align="left">說明</th>
</tr>
<tr>
    <td rowspan="8">等候</td><td>ScheduleTime</td><td>活動時段執行的時間還沒到。</td>
</tr>
<tr>
<td>DatasetDependencies</td><td>上游相依項目尚未就緒。</td>
</tr>
<tr>
<td>ComputeResources</td><td>計算資源無法使用。</td>
</tr>
<tr>
<td>ConcurrencyLimit</td> <td>所有活動執行個體都在執行其他的活動時段。</td>
</tr>
<tr>
<td>ActivityResume</td><td>活動已暫停，除非活動繼續，否則無法執行活動時段。</td>
</tr>
<tr>
<td>Retry</td><td>正在重試活動執行。</td>
</tr>
<tr>
<td>驗證</td><td>驗證尚未啟動。</td>
</tr>
<tr>
<td>ValidationRetry</td><td>驗證正在等待重試。</td>
</tr>
<tr>
<tr>
<td rowspan="2">InProgress</td><td>Validating</td><td>驗證正在進行中。</td>
</tr>
<td>-</td>
<td>系統正在處理該活動時段。</td>
</tr>
<tr>
<td rowspan="4">Failed</td><td>TimedOut</td><td>活動執行超過活動所允許的時間。</td>
</tr>
<tr>
<td>Canceled</td><td>使用者動作已取消活動時段。</td>
</tr>
<tr>
<td>驗證</td><td>驗證失敗。</td>
</tr>
<tr>
<td>-</td><td>無法產生或驗證活動時段。</td>
</tr>
<td>就緒</td><td>-</td><td>活動時段已可供使用。</td>
</tr>
<tr>
<td>Skipped</td><td>-</td><td>活動時段未處理。</td>
</tr>
<tr>
<td>None</td><td>-</td><td>曾經以不同的狀態存在，但目前已重設的活動時段。</td>
</tr>
</table>


當您按一下清單中的某個活動時段時，您會在右側的 [活動時段總管] 或 [屬性] 視窗中看到其詳細資料。

![活動時段總管](./media/data-factory-monitor-manage-app/ActivityWindowExplorer-2.png)

### <a name="refresh-activity-windows"></a>重新整理活動時段
詳細資料並不會自動重新整理，因此您必須使用命令列上的 [重新整理] 按鈕 (第二個按鈕) 來手動重新整理活動時段清單。  

### <a name="properties-window"></a>[屬性] 視窗
[屬性] 視窗在監視及管理應用程式最右側的窗格中。

![[屬性] 視窗](./media/data-factory-monitor-manage-app/PropertiesWindow.png)

它會顯示您在 [資源總管] \(樹狀檢視)、[圖表檢視] 或 [活動時段清單] 中所選取項目的屬性。

### <a name="activity-window-explorer"></a>活動時段總管
[活動時段總管] 視窗在監視及管理應用程式最右側的窗格中。 它會顯示您在 [活動時段] 快顯視窗或 [活動時段清單] 中所選取活動時段的詳細資料。

![活動時段總管](./media/data-factory-monitor-manage-app/ActivityWindowExplorer-3.png)

您可以切換至另一個活動時段，方法是按一下頂端行事曆檢視中的其他活動時段。 而頂端的 [向左箭號]/[向右箭號] 按鈕可讓您查看上一週/下一週的活動時段。

您可以使用底部窗格中的工具列按鈕重新執行活動時段，或是重新整理窗格中的詳細資料。

### <a name="script"></a>指令碼
您可以使用 [指令碼] 索引標籤，檢視所選取 Data Factory 實體 (連結服務、資料集或管線) 的 JSON 定義。

![指令碼索引標籤](./media/data-factory-monitor-manage-app/ScriptTab.png)

## <a name="use-system-views"></a>使用系統檢視
監視及管理應用程式包含預先建置的系統檢視 (**最近的活動時段**、**失敗的活動時段**、**進行中的活動時段**)，可讓您檢視 Data Factory 最近/失敗/進行中的活動時段。

只要按一下左側的 [監視檢視]  索引標籤，就能切換到 [監視檢視]。

![[監視檢視] 索引標籤](./media/data-factory-monitor-manage-app/MonitoringViewsTab.png)

目前，有三個支援的系統檢視。 請選取某個選項，以便在 [活動時段] 清單 (位於中間窗格底部) 中查看最近的活動時段、失敗的活動時段，或進行中的活動時段。

當您選取 [最近的活動時段] 選項時，您會看到以 [上次嘗試時間] 的遞減順序排列之所有最近的活動時段。

您可以使用 [失敗的活動時段]  檢視，查看清單中所有失敗的活動時段。 選取清單中某個失敗的活動時段，就能在 [屬性] 視窗或 [活動時段總管] 中看到該活動時段的詳細資料。 您也可以下載失敗的活動時段的任何記錄檔。

## <a name="sort-and-filter-activity-windows"></a>排序與篩選活動時段
在命令列中變更**開始時間**及**結束時間**設定來篩選活動時段。 當您變更開始時間及結束時間之後，請按一下結束時間旁邊的按鈕來重新整理 [活動時段] 清單。

![開始及結束時間](./media/data-factory-monitor-manage-app/StartAndEndTimes.png)

> [!NOTE]
> 目前，監視及管理應用程式中所有時間都是以 UTC 格式來顯示。
>
>

請在 [活動時段清單] 中，按一下某個資料行的名稱 (例如：[狀態])。

![活動時段清單的資料行功能表](./media/data-factory-monitor-manage-app/ActivityWindowsListColumnMenu.png)

您可以執行下列動作：

* 以遞增順序排序。
* 以遞減順序排序。
* 根據一或多個值 (Ready、Waiting 等) 來篩選。

當您在某個資料行上指定篩選條件時，該資料行的篩選器按鈕就會啟用，以指出該資料行中的值為已經過篩選的值。

![篩選活動時段清單的資料行](./media/data-factory-monitor-manage-app/ActivityWindowsListFilterInColumn.png)

您可以使用相同的快顯視窗來清除篩選條件。 如要清除活動時段清單的所有篩選條件，請按一下命令列上的 [清除篩選條件] 按鈕。

![清除活動時段清單的所有篩選條件](./media/data-factory-monitor-manage-app/ClearAllFiltersActivityWindowsList.png)

## <a name="perform-batch-actions"></a>執行批次動作
### <a name="rerun-selected-activity-windows"></a>重新執行已選取的活動時段
選取某個活動時段、按一下第一個命令列按鈕旁的向下箭號，然後選取 [重新執行] / [搭配管線上游來重新執行]。 當您選取 [搭配管線上游來重新執行] 時，系統也會傳回所有上游的活動時段。
    ![重新執行某個活動時段](./media/data-factory-monitor-manage-app/ReRunSlice.png)

您也可以選取清單中的數個活動時段，然後同時重新執行這些活動時段。 您可能會想要根據狀態 (例如 [失敗]) 來篩選活動時段，然後在修正導致活動時段執行失敗的問題之後，重新執行失敗的活動時段。 請參閱下一節來取得如何篩選清單中活動時段的詳細資料。  

### <a name="pauseresume-multiple-pipelines"></a>暫停/繼續多個管線
您可以使用 Ctrl 鍵，以多重選取兩個以上的管線。 您可以使用命令列按鈕 (在下圖以紅色矩形反白顯示) 來暫停/繼續這些管線。

![命令列上的暫停/繼續](./media/data-factory-monitor-manage-app/SuspendResumeOnCommandBar.png)
