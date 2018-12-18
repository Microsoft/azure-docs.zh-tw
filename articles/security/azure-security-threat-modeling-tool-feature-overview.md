---
title: Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: 深入了解威脅模型化工具中可用的所有功能
services: security
documentationcenter: na
author: RodSan
manager: RodSan
editor: RodSan
ms.assetid: na
ms.service: security
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: rodsan
ms.openlocfilehash: 76e49fc0e680acbc2b7b7c62b69fbf72d6690acf
ms.sourcegitcommit: 68aec76e471d677fd9a6333dc60ed098d1072cfc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/18/2017
ms.locfileid: "26754740"
---
# <a name="threat-modeling-tool-feature-overview"></a>威脅模型化工具功能概觀

威脅模型化工具能滿足您的威脅模型化需求。 如需這項工具的基本介紹，請參閱[開始使用威脅模型化工具](./azure-security-threat-modeling-tool-getting-started.md)。

> [!NOTE]
>威脅模型化工具會經常更新，因此請經常檢查本指南，以查看我們的最新功能和增強功能。

若要開啟空白頁面，請選取 [建立模型]。

![空白頁面](./media/azure-security-threat-modeling-tool-feature-overview/tmtstart.png)

若要查看工具目前提供哪些功能，請使用我們的小組在[開始使用](./azure-security-threat-modeling-tool-getting-started.md)範例中建立的威脅模型。

![基本威脅模型](./media/azure-security-threat-modeling-tool-feature-overview/basictmt.png)

## <a name="navigation"></a>導覽

在探討內建功能之前，我們先來看一下工具的主要元件。

### <a name="menu-items"></a>功能表項目

體驗與其他 Microsoft 產品相似。 我們來看看最上層的功能表項目。

![功能表項目](./media/azure-security-threat-modeling-tool-feature-overview/menuitems.png)

| 標籤                               | 詳細資料      |
| --------------------------------------- | ------------ |
| **檔案** | <ul><li>開啟、儲存及關閉檔案</li><li>登入及登出 OneDrive 帳戶。</li><li>共用連結 (檢視和編輯)。</li><li>檢視檔案資訊。</li><li>將新的範本套用至現有的模型。</li></ul> |
| **編輯** | 復原和重做動作，以及複製、貼上和刪除。 |
| **檢視** | <ul><li>在**分析**和**設計**檢視之間切換。</li><li>開啟已關閉的視窗 (例如，樣板、元素屬性和訊息)。</li><li>將版面配置重設為預設設定。</li></ul> |
| **圖表** | 新增和刪除圖表，以及切換圖表的索引標籤。 |
| **報告** | 建立要與其他人共用的 HTML 報告。 |
| **說明** | 尋找協助您使用工具的指南。 |

符號是最上層功能表的捷徑：

| 符號                               | 詳細資料      |
| --------------------------------------- | ------------ |
| **開啟** | 開啟新的檔案。 |
| **儲存** | 儲存目前的檔案。 |
| **設計** | 開啟**設計**檢視，以供您建立模型。 |
| **分析** | 顯示產生的威脅和其屬性。 |
| **新增圖表** | 新增圖表 (類似於 Excel 中的新增索引標籤)。 |
| **刪除圖表** | 刪除目前的圖表。 |
| **複製/剪下/貼上** | 複製、剪下及貼上元素。 |
| **復原/重做** | 復原和重做動作。 |
| **放大/縮小** | 放大和縮小圖表以獲得更清楚的檢視。 |
| **意見反應** | 開啟 MSDN 論壇。 |

### <a name="canvas"></a>畫布

畫布是供您拖放元素的空間。 拖放是建立模型最快且最有效率的方式。 您也能按一下滑鼠右鍵並從功能表選取項目，以新增泛型版本的元素，如下圖所示：

#### <a name="drop-the-stencil-on-the-canvas"></a>在畫布上置放樣板

![畫布置放](./media/azure-security-threat-modeling-tool-feature-overview/canvasdrop1.png)

#### <a name="select-the-stencil"></a>選取樣板

![元素屬性](./media/azure-security-threat-modeling-tool-feature-overview/canvasdrop2.png)

### <a name="stencils"></a>樣板

根據選取的範本，您可以找到所有可用的樣板。 如果您找不到正確的元素，請使用另一個範本。 您也可以修改範本以滿足需求。 一般而言，您能夠找到如下的類別組合：

| 樣板名稱                               | 詳細資料      |
| --------------------------------------- | ------------ |
| **處理程序** | 應用程式、瀏覽器外掛程式、執行緒、虛擬機器 |
| **外部互動者** | 驗證提供者、瀏覽器、使用者、Web 應用程式 |
| **資料存放區** | 快取、儲存體、設定檔、資料庫、登錄 |
| <bpt id="p1">**</bpt>Data flow<ept id="p1">**</ept> | 二進位、ALPC、HTTP、HTTPS/TLS/SSL、IOCTL、IPSec、具名管道、DCOM、SMB、UDP |
| **信任線路/邊界** | 公司網路、網際網路、機器、沙箱、使用者/核心模式 |

### <a name="notesmessages"></a>附註/訊息

| 元件                               | 詳細資料      |
| --------------------------------------- | ------------ |
| **訊息** | 每當發生錯誤時 (例如元素之間沒有資料流程) 警示使用者的內部工具邏輯。 |
| **注意事項** | 由工程小組在整個設計和檢閱流程中新增至檔案的手動附註。 |

### <a name="element-properties"></a>元素屬性

元素屬性會因您選取的元素不同而有所差異。 除了信任界限，其他所有元素會包含 3 個一般選取項目：

| 元素屬性                               | 詳細資料      |
| --------------------------------------- | ------------ |
| **名稱** | 用來為您的處理序、存放區、互動者及流程命名，以利辨識。 |
| **超出範圍** | 如果選取，元素會從威脅產生矩陣中移除 (不建議)。 |
| **超出範圍原因** | 理由欄位，讓使用者知道為什麼選取超出範圍。 |

每個元素類別下的屬性都會變更。 請選取每個元素來檢查可用選項。 您也可以開啟範本來取得更多資訊。 我們來看看有什麼功能。

## <a name="welcome-screen"></a>歡迎使用畫面

當您開啟應用程式時，會看到**歡迎使用**。

### <a name="open-a-model"></a>開啟模型

將游標移到 [開啟模型] 上方時會顯示 2 個選項：[從這部電腦開啟] 和 [從 OneDrive 開啟]。 第一個選項會開啟 [開啟舊檔] 畫面。 第二個選項會引導您完成 OneDrive 的登入程序。 成功通過驗證之後，您可以選取資料夾和檔案。

![開啟模型](./media/azure-security-threat-modeling-tool-feature-overview/openmodel.png)

![從電腦或 OneDrive 開啟](./media/azure-security-threat-modeling-tool-feature-overview/openmodel2.png)

### <a name="feedback-suggestions-and-issues"></a>意見反應、建議和問題

當您選取 [意見反應、建議和問題] 時，會前往 SDL 工具的 MSDN 論壇。 您可以閱讀其他人對工具有什麼看法 (包括因應措施和新的想法)。

![意見反應](./media/azure-security-threat-modeling-tool-feature-overview/feedback.png)

## <a name="design-view"></a>設計檢視

當您開啟或新建模型時，**設計**檢視隨即會開啟。

### <a name="add-elements"></a>新增元素

您可以利用兩種方法將元素加入格線：

- **拖放**：將需要的元素拖曳到格線中。 接著再使用元素屬性來提供其他資訊。
- **按一下滑鼠右鍵**：以滑鼠右鍵按一下格線上的任何位置，並從下拉式功能表選取項目。 選定元素的泛型表示法會出現在畫面上。

### <a name="connect-elements"></a>連接元素

您可以利用兩種方法連接元素：

- **拖放**：將所需的資料流程拖曳到格線中，然後將兩端連接到適當的元素。
- **按一下 + Shift**：按一下第一個元素 (傳送資料)，按住 Shift 鍵，然後選取第二個元素 (接收資料)。 按一下滑鼠右鍵，然後選取 [連接]。 如果您使用雙向資料流程，順序並不重要。

### <a name="properties"></a>properties

 若要查看可在樣板上修改的屬性，您只要選取樣板，系統就會據以填入資訊。 以下範例示範將**資料庫**樣板拖曳到圖表上的前後：

#### <a name="before"></a>先前

![先前](./media/azure-security-threat-modeling-tool-feature-overview/properties1.png)

#### <a name="after"></a>後續

![後續](./media/azure-security-threat-modeling-tool-feature-overview/properties2.png)

### <a name="messages"></a>訊息

如果您建立威脅模型，但是忘記將資料流程連接到元素，您會收到通知。 您可以忽略訊息，或遵循指示修正問題。 

![訊息](./media/azure-security-threat-modeling-tool-feature-overview/messages.png)

### <a name="notes"></a>注意

若要在圖表中加入附註，請從 [訊息] 索引標籤切換到 [附註]。

## <a name="analysis-view"></a>分析檢視

建置圖表後，請在捷徑工具列上選取**分析**符號 (放大鏡)，以切換為**分析**檢視。

![分析檢視](./media/azure-security-threat-modeling-tool-feature-overview/analysisview.png)

### <a name="generated-threat-selection"></a>產生的威脅選取項目

當您選取威脅時，可以使用三種不同的功能：

| 功能                               | 資訊      |
| --------------------------------------- | ------------ |
| **讀取指標** | <p>將威脅標示為讀取，以協助您追蹤檢閱過的項目。</p><p>![讀取/未讀取指標](./media/azure-security-threat-modeling-tool-feature-overview/readmode.png)</p> |
| **互動焦點** | <p>圖表中屬於威脅的互動會反白顯示。</p><p>![互動焦點](./media/azure-security-threat-modeling-tool-feature-overview/interactionfocus.png)</p> |
| **威脅屬性** | <p>威脅的其他資訊會顯示在 [威脅屬性] 視窗中。</p><p>![威脅屬性](./media/azure-security-threat-modeling-tool-feature-overview/threatproperties.png)</p> |

### <a name="priority-change"></a>優先順序變更

您可以變更每個產生之威脅的優先順序層級。 不同的色彩讓您能輕鬆辨識高、中和低優先順序的威脅。

![優先順序變更](./media/azure-security-threat-modeling-tool-feature-overview/prioritychange.png)

### <a name="threat-properties-editable-fields"></a>威脅屬性可編輯欄位

如前述影像所示，您可以變更工具所產生的資訊。 您也可以將資訊加入特定欄位 (例如，理由)。 這些欄位由範本產生。 如果您需要每個威脅的詳細資訊，可以進行修改。

![威脅屬性](./media/azure-security-threat-modeling-tool-feature-overview/threatproperties.png)

## <a name="reports"></a>報告

完成優先順序的變更及更新每個產生之威脅的狀態後，您可以儲存檔案和/或列印報告。 移至 [報告]  >  [建立完整報告]。 為報告命名，接著您應該會看到類似下圖的內容：

![報告](./media/azure-security-threat-modeling-tool-feature-overview/report.png)

## <a name="next-steps"></a>後續步驟

* 若要為社群貢獻範本，請移至我們的 [GitHub](https://github.com/Microsoft/threat-modeling-templates) 頁面。 
* 若要開始使用工具，請移至[下載](https://aka.ms/tmtpreview)頁面。
