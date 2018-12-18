---
title: 使用者入門 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: 這是更深入的概觀，反白顯示動作中的威脅模型化工具。
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
ms.openlocfilehash: 340c92d2830069a9d957f4ece79416a707062629
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096769"
---
# <a name="getting-started-with-the-threat-modeling-tool"></a>威脅模型化工具使用者入門

雲端和企業安全性工具小組今年稍早將威脅模型化工具預覽發行為免費的**[按一下即可下載](https://aka.ms/tmtpreview)**。 傳遞機制中的變更可讓我們在客戶每次開啟它時，將最新的增強功能和錯誤修正傳送給客戶，讓它更容易維護及使用。
此文章會引導您開始使用 Microsoft SDL 威脅模型化方法的程序，並且示範如何使用工具來開發很棒的威脅模型，作為安全性程序的骨幹。

這篇文章是根據 SDL 威脅模型化方法的現有知識所建置的。 如需快速檢閱，請參閱**[威脅模型化 Web 應用程式](https://msdn.microsoft.com/library/ms978516.aspx)** 和 2006 年發佈的**[使用 STRIDE 方法發現安全性缺陷](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy)** MSDN 文件的封存版本。

若要快速摘要，方法牽涉到建立圖表、識別威脅、緩和它們及驗證每個風險降低。 以下反白顯示此程序的圖表：

![SDL 程序](./media/azure-security-threat-modeling-tool-feature-overview/sdlapproach.png)

## <a name="starting-the-threat-modeling-process"></a>啟動威脅模型化程序

當您啟動威脅模型化工具時，您會發現一些事情，如在圖片中所見：

![空白起始分頁](./media/azure-security-threat-modeling-tool-feature-overview/tmtstart.png)

### <a name="threat-model-section"></a>威脅模型區段

| 元件                                   | 詳細資料                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **意見反應、建議和問題按鈕** | 針對所有 SDL 的事項，將您導向  **[MSDN 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=sdlprocess)**。 它會讓您閱讀其他使用者做了什麼，以及因應措施和建議。 如果您仍然找不到您要尋找的項目，請傳送電子郵件至 tmtextsupport@microsoft.com，讓我們的支援小組協助您                                                                                                                            |
| **建立模型**                          | 開啟空白的畫布，讓您繪製圖表。 請務必選取您想要用於模型的範本                                                                                                                                                                                                                                                                                                                                                                       |
| **新模型的範本**                 | 您必須在建立模型之前選取要使用哪個範本。 我們的主要範本是 Azure 威脅模型範本範本，其中包含 Azure 特定樣板、威脅和風險降低。 對於一般模型，請從下拉式功能表選取 SDL TM 知識庫。 想要建立您自己的範本或為所有使用者提交新範本嗎？ 請參閱我們的**[範本存放庫](https://github.com/Microsoft/threat-modeling-templates)** GitHub 分頁，以進一步了解                              |
| **開啟模型**                            | <p>會開啟先前儲存的威脅模型。 如果您需要開啟最新的檔案，「最近開啟的模型」功能是很好的選項。 當您將滑鼠停留在選取項目時，您會發現有 2 種方法可以開啟模型：</p><p><ul><li>從這部電腦開啓 – 使用本機儲存體開啟檔案的傳統方法</li><li>從 OneDrive 開啟 – 小組可以使用 OneDrive 中的資料夾，在單一位置中儲存及共用他們的所有威脅模型，以協助增進生產力和共同作業</li></ul></p> |
| **使用者入門指南**                   | 開啟 **[Microsoft 威脅模型化工具](./azure-security-threat-modeling-tool.md)** 主要分頁                                                                                                                                                                                                                                                                                                                                                                                            |

### <a name="template-section"></a>範本區段

| 元件               | 詳細資料                                                                                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **建立新的範本** | 開啟空白的範本讓您據以建置。 除非您有從頭建置範本的廣泛知識，否則我們建議您從現有範本建置 |
| **開啟範本**       | 開啟現有的範本供您進行變更                                                                                                              |

威脅模型化工具小組持續努力改善工具功能和體驗。 在過去一年進行了一些微幅變更，但是所有主要變更需要在指南中重寫。 請經常參閱它以確保您取得最新的宣告。

## <a name="building-a-model"></a>建置模型

在本節中，我們會跟隨：

- Cristina (開發人員)
- Ricardo (程式經理) 和
- Ashish (測試人員)

他們會進行開發其第一個威脅模型的程序。

> Ricardo：Hi Cristina，我在威脅模型圖表上工作，而且想要確定我們的詳細資料正確。 您可以協助查看嗎？
> Cristina：當然。 讓我看看。
> Ricardo 開啟工具，並且與 Cristina 共用他的畫面。

![基本威脅模型](./media/azure-security-threat-modeling-tool-feature-overview/basictmt.png)

> Cristina：好的，看起來直接明瞭，但是可以為我逐步示範嗎？
> Ricardo：當然！ 分解如下：
> - 我們的人類使用者是繪製為外部實體 — 正方形
> - 他們將命令傳送給我們的 Web 伺服器 — 圓形
> - Web 伺服器已諮詢資料庫 (兩條平行線)

Ricardo 剛剛為 Cristina 顯示的是 DFD，**[資料流程圖表](https://en.wikipedia.org/wiki/Data_flow_diagram)** 的簡稱。 威脅模型化工具可讓使用者指定信任界限，以紅色虛線表示，來顯示控制不同實體的位置。 例如，IT 系統管理員需要 Active Directory 系統用於驗證目的，因此 Active Directory 會在其控制之外。

> Cristina：就我看來是正確的。 威脅呢？
> Ricardo：我來示範給您看。

## <a name="analyzing-threats"></a>分析威脅

當他從圖示功能表選取項目按一下分析檢視 (具有放大鏡的檔案) 時，系統將他導向產生之威脅的清單，這些威脅是威脅模型化工具根據預設範本找到的，使用稱為 **[STRIDE (詐騙、竄改、資訊洩漏、否認、阻斷服務以及權限提高)](https://en.wikipedia.org/wiki/STRIDE_(security))** 的 SDL 方法。 這個概念是軟體來自可預測的一組威脅底下，可以使用這 6 個類別找到。

這個方法就像在新增警報系統或追趕竊賊之前，確定每個門窗都有就地鎖定機制，籍以保護您的家園。

![基本威脅](./media/azure-security-threat-modeling-tool-feature-overview/basicthreats.png)

Ricardo 從選取清單上的第一個項目開始。 排程狀況如下：

首先，兩個樣板之間的互動已增強

![互動](./media/azure-security-threat-modeling-tool-feature-overview/interaction.png)

第二，威脅的其他資訊已顯示在威脅屬性視窗中

![互動資訊](./media/azure-security-threat-modeling-tool-feature-overview/interactioninfo.png)

產生的潛在威脅可協助他了解可能的設計缺陷。 STRIDE 分類讓他了解潛在攻擊媒介的概念，而額外描述則確切地告訴他什麼是錯誤的，以及緩和這種情況的可能方式。 他可以使用可編輯欄位在理由詳細資料中寫入附註，或根據其組織的 錯誤列來變更優先順序評等。

Azure 範本有額外的詳細資料，可協助使用者不只了解什麼是錯誤的，同時知道如何修正此問題，方法是將描述、範例和超連結新增至 Azure 特定文件。

描述讓他理解新增驗證機制以防止使用者被詐騙的重要性，揭露第一個要進行處理的威脅。 與 Cristina 幾分鐘的討論之後，他們了解實作存取控制和角色的重要性。 Ricardo 填入一些快速附註，以確定實作這些項目。

Ricardo 進入 [資訊洩漏] 底下的威脅時，他理解到存取控制計劃需要一些唯讀帳戶オ能進行稽核和報告產生。 他設想這是否為新的威脅，但是風險降低都相同，所以他據以附註威脅。
他也更深入一些思考資訊洩漏，並且理解到備份磁帶需要加密，這是作業小組的作業。

威脅不適用於設計，因為現有的風險降低或安全性保證可以從 [狀態] 下拉式清單變更為「不適用」。 有其他三個選擇：未啟動 – 預設選項、需要調查 – 用來追蹤項目，以及緩和 – 完全處理之後。

## <a name="reports--sharing"></a>報告與共用

一旦 Ricardo 說明完整份清單且 Cristina 新增重要注意事項，風險降低/理由、優先順序和狀態會變更，他選取 [報告] -> [建立完整報告] -> [儲存報告]，這樣會列印出良好的清單供他與同事瀏覽，確保已實作適當的安全性工作。

![互動資訊](./media/azure-security-threat-modeling-tool-feature-overview/report.png)

如果 Ricardo 想要改為共用檔案，他可以籍由在組織的 OneDrive 帳戶中儲存，來完成此操作。 一旦他那樣做，就可以複製文件連結，並且與同事共用。 

## <a name="threat-modeling-meetings"></a>威脅模型化會議

當 Ricardo 使用 OneDrive 將威脅模型傳送給他的同事時，測試人員 Ashish 並未留下深刻印象。 Ricardo 和 Cristina 似乎遺漏一些重要的極端案例，這些案例很容易遭到入侵。 他的懷疑論對於威脅模型是一種補充。

在此案例中，Ashish 瀏覽過威脅模型之後，他召開兩個威脅模型會議：一個會議是同步處理程序並且逐步進行圖表，而第二個會議則是針對威脅檢閱和登出。

在第一個會議中，Ashish 花費 10 分鐘讓每個人了解 SDL 威脅模型流程。 然後，他會提取威脅模型圖表並且開始詳細說明。 五分鐘內就識別重要的遺漏元件。

幾分鐘之後，Ashish 和 Ricardo 進入 Web 伺服器建置方式的延伸討論。 它不是會議進行的理想方式，但是每個人最終都會同意，及早發現差異可以在未來節省他們的時間。

在第二個會議中，小組瀏覽威脅、討論一些解決方法，然後登出威脅模型。 他們將文件簽入原始檔控制，然後繼續進行開發。

## <a name="thinking-about-assets"></a>思考資產

某些有威脅模型化的讀者可能會注意到，我們完全未討論到資產。 我們發現許多軟體工程師對於軟體的理解，比對於資產概念的理解以及了解攻擊者對哪個資產感到興趣來得好。

如果您打算對一個家庭塑造威脅模型，您可能要從您的家族、無法取代的相片或重要的藝術品開始思考。 也許您可以從誰有可能侵入目前的安全性系統開始思考。 或者，您可以從考量實際功能，例如游泳池和前廊開始。 這些類似於思考資產、攻擊者或軟體設計。 三種方法的任何一個都有作用。

我們在這裡呈現的威脅模型化方法比 Microsoft 過去完成的方法簡單許多。 我們發現軟體設計方法對於許多小組都運作良好。 我們希望這也包括您。

## <a name="next-steps"></a>後續步驟

將您的問題、註解和考量傳送到 tmtextsupport@microsoft.com。 **[下載](https://aka.ms/tmtpreview)** 威脅模型化工具來開始。
