---
title: Azure 資料目錄的常見問題集
description: 與 Azure 資料目錄相關的常見問題集，包括資料來源探索、註解和管理功能。
services: data-catalog
author: steelanddata
ms.author: maroche
ms.assetid: 5c7e209a-458c-4bb4-96bb-7ed178f9528a
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: bb9223e2eef11d3378c228f8cd5242e6ebbbc74d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43053116"
---
# <a name="azure-data-catalog-frequently-asked-questions"></a>Azure 資料目錄的常見問題集
這篇文章提供與 Azure 資料目錄服務相關的常見問題集解答。

## <a name="what-is-azure-data-catalog"></a>什麼是 Azure 資料目錄？
資料目錄是完全受控的雲端服務，託管在 Microsoft Azure 中，可作為企業資料來源的註冊系統和探索系統。 不論是分析師、資料科學家或資料開發人員，所有使用者都可利用資料目錄來註冊、探索、了解及取用資料來源。

## <a name="what-customer-challenges-does-it-solve"></a>它解決了哪些客戶面臨的挑戰？
資料目錄可解決資料來源探索和「黑暗資料」的挑戰，讓使用者能夠探索並了解企業資料來源。

## <a name="what-are-its-target-audiences"></a>其目標對象為何？
資料目錄的設計目標為技術性和非技術性使用者，包括：

* 資料開發人員、BI 和分析專業人員：這些人負責產生資料和分析內容供他人使用。
* 資料負責人：這些人了解資料、其意義和用法。
* 資料取用者：這些人需要能夠透過他們所選擇的工具，輕鬆地探索、了解並連線至工作所需的資料。
* 中央 IT：這些人需要提供數百個資料來源供商務使用者探索，也需要持續監督資料的使用情況和由誰使用。

## <a name="what-is-its-availability-by-region"></a>其依區域的可用性為何？
目前只有下列資料中心提供資料目錄服務：

* 美國西部
* 美國東部
* 西歐
* 北歐
* 澳洲東部
* 東南亞

## <a name="what-are-its-limits-on-the-number-of-data-assets"></a>其資料資產的數目限制有哪些？
免費版資料目錄僅限於 5,000 個已註冊的資料資產。

標準版資料目錄支援多達 100,000 個已註冊的資料資產。

任何已在資料目錄中註冊的物件，例如資料表、檢視、檔案、報表，都算作一個資料資產。

## <a name="what-are-its-supported-data-source-and-asset-types"></a>其支援的資料來源和資產類型為何？
請參閱[資料目錄 DSR](data-catalog-dsr.md) 以取得目前支援的資料來源清單。

## <a name="how-do-i-request-support-for-another-data-source"></a>如何要求另一個資料來源的支援？
若要提交功能要求與其他意見，請前往 [Azure 意見論壇 - 資料目錄](https://feedback.azure.com/forums/906052-data-catalog/category/320788-data-sources)。

## <a name="how-do-i-get-started-with-data-catalog"></a>如何開始使用資料目錄？
開始使用的最佳方式是前往[開始使用資料目錄](data-catalog-get-started.md)。 本文是服務中各項功能的端對端概觀。

## <a name="how-do-i-register-my-data"></a>如何註冊我的資料？
在資料目錄中註冊您的資料：
1. 在 Azure 資料目錄入口網站的 [發佈] 區域中，啟動 Azure 資料目錄註冊工具。 
2. 在資料目錄的資料來源註冊工具中，使用用以存取資料目錄入口網站的相同認證登入。
3. 選取您要註冊的資料來源和特定資產。

## <a name="what-properties-does-it-extract-for-data-assets-that-are-registered"></a>從已註冊的資料資產中會擷取哪些屬性？
具體的屬性隨資料來源而不同，一般而言，資料目錄發佈服務會擷取下列資訊：

* 資產名稱
* 資產類型
* 資產描述
* 屬性/資料行名稱
* 屬性/資料行資料類型
* 屬性/資料行描述

> [!IMPORTANT]
> 使用資料目錄註冊資料資產並不會將資料移動或複製至雲端。 從資料來源註冊資產會將資產的中繼資料複製到 Azure，但資料仍保留在現有的資料來源位置。 此規則的例外狀況在於您註冊資產時，選擇上傳預覽記錄或資料設定檔。 包含預覽時，最多會從每個資產複製 20 筆記錄，並在資料目錄中儲存為快照。 包含資料設定檔時，會計算彙總資訊，並將其併入儲存在目錄中的中繼資料。 彙總資訊可包含資料表大小、每個資料行 null 值的百分比，或者資料行的最小值、最大值和平均值。 
>
>

> [!NOTE]
> 若資料來源 (例如 SQL Server Analysis Services) 具有第一級的 **Description** 屬性，資料目錄的資料來源註冊工具就會擷取該屬性值。 若 SQL Server 關聯式資料庫缺少第一級的 **Description** 屬性，資料目錄的資料來源註冊工具則會從物件和資料行的 **ms_description** 擴充屬性擷取值。 如需詳細資訊，請參閱[使用資料庫物件的擴充屬性](https://technet.microsoft.com/library/ms190243%28v=sql.105%29.aspx)。
>
>

## <a name="how-long-should-it-take-for-newly-registered-assets-to-appear-in-the-catalog"></a>新註冊的資產經過多久才會出現在目錄中？
使用資料目錄註冊資產之後，可能需要經過 5-10 秒才會在資料目錄入口網站中顯示這些資產。

## <a name="how-do-i-annotate-and-enrich-the-metadata-for-my-registered-data-assets"></a>如何標註和充實已註冊資料資產的中繼資料？
若要為已註冊的資產提供中繼資料，最簡單的方法是在資料目錄入口網站中選取資產，然後在所選物件的屬性窗格或結構描述窗格中，輸入中繼資料值。

在註冊過程中，您也可以提供一些中繼資料，例如專家和標記。 在資料目錄發佈服務中提供的值，將會套用到當時註冊的所有資產。 若要在入口網站中檢視最近註冊的物件以另外標註，請在資料目錄資料來源註冊工具的最後一個畫面中選取 [檢視入口網站] 按鈕。

## <a name="how-do-i-delete-my-registered-data-objects"></a>如何刪除我已註冊的資料物件？
您可以從資料目錄中刪除物件，只要在入口網站中選取物件，然後按一下 [刪除] 按鈕即可。 移除物件會從資料目錄中移除其中繼資料，但不會影響基礎資料來源。

## <a name="what-is-an-expert"></a>什麼是專家？
專家是指對資料物件具有獨特見解的人。 一個物件可能有多個專家。 專家不見得是物件的「擁有者」，而只是知道應該如何運用資料的人。

## <a name="how-do-i-share-information-with-the-data-catalog-team-if-i-encounter-problems"></a>如果我遇到問題，如何與資料目錄小組共用資訊？
若要報告問題、共用資訊及發問，請前往 [Azure 資料目錄論壇](http://go.microsoft.com/fwlink/?LinkID=616424&clcid=0x409)。

## <a name="does-the-catalog-work-with-another-data-source-that-im-interested-in"></a>目錄是否可與我感興趣的其他資料來源搭配運作？
我們正積極將更多資料來源新增至資料目錄。 如果您希望我們支援某個特定的資料來源，請前往 [Azure 資料目錄意見反應論壇](https://feedback.azure.com/forums/906052-data-catalog)提出建議 (如果已有人建議，請表達支持)。

## <a name="how-is-azure-data-catalog-related-to-the-data-catalog-in-power-bi-for-office-365"></a>Azure 資料目錄與 Power BI for Office 365 中的資料目錄有何關係？
您可將 Azure 資料目錄視為 Power BI 中進化版的「資料目錄」。 從 2017 年春季開始，Azure 資料目錄用來在 Excel 2016 和 Power Query for Excel 中共用和探索查詢。 Excel 中的資料目錄功能適用於具有 Power BI Pro 授權的使用者。

## <a name="what-permissions-do-i-need-to-register-assets-with-data-catalog"></a>向資料目錄註冊資產需要什麼權限？
若要執行資料目錄註冊工具，您在資料來源上需要有可讓您從來源讀取中繼資料的權限。 若要一併包含預覽，您必須有可讓您從註冊的物件讀取資料的權限。

資料目錄也允許目錄系統管理員限制哪些使用者和群組可以將中繼資料新增至目錄。 如需詳細資訊，請參閱[如何保護對資料目錄及資料資產的存取](data-catalog-how-to-secure-catalog.md)。

## <a name="will-data-catalog-be-made-available-for-on-premises-deployment-as-well"></a>資料目錄也會支援內部部署嗎？
資料目錄是可以同時使用雲端和內部部署資料來源的雲端服務，提供混合式資料來源探索解決方案。 目前並未規劃推出可採內部部署執行的資料目錄服務版本。

## <a name="can-i-extract-more-or-richer-metadata-from-the-data-sources-i-register"></a>我是否可以從我所註冊的資料來源中擷取更多或更豐富的中繼資料？
我們正積極擴充資料目錄的功能。 如果您希望在註冊期間從資料來源擷取其他中繼資料，請在 [Azure 意見論壇 - 資料目錄](https://feedback.azure.com/forums/906052-data-catalog)提出建議 (如果已有人建議，請投票支持)。 

針對資料來源註冊工具未擷取此中繼資料的資料來源，如果您想要包含資料行/結構描述中繼資料、預覽或資料設定檔，可以使用資料目錄 API 來新增此中繼資料。 如需詳細資訊，請參閱 [Azure 資料目錄 REST API](https://docs.microsoft.com/rest/api/datacatalog/)。

## <a name="how-do-i-restrict-the-visibility-of-registered-data-assets-so-that-only-certain-people-can-discover-them"></a>如何限制已註冊之資料資產的可見性，以限制只有特定人員才能探索它們？
在資料目錄中選取資料資產，然後按一下 [取得擁有權] 按鈕。 資料目錄中的資料資產擁有者可以變更可見性設定，可能變更為允許所有使用者探索擁有的資產，或限定只有特定使用者才能看見。 如需詳細資訊，請參閱[在 Azure 資料目錄中管理資料資產](data-catalog-how-to-manage.md)。

## <a name="how-do-i-update-the-registration-for-a-data-asset-so-that-changes-in-the-data-source-are-reflected-in-the-catalog"></a>如何更新資料資產的註冊，讓資料來源中的變更反映在目錄中？
若要更新已在目錄中註冊之資料資產的中繼資料，只要重新註冊包含這些資產的資料來源即可。 資料來源中的任何變更 (例如在資料表或檢視中新增或移除資料行) 都會在目錄中更新，但會保留使用者提供的任何註解。

## <a name="my-question-isnt-answered-here-where-can-i-go-for-answers"></a>這裡沒有回答我的問題。 可以在何處尋求解答？
請前往 [Azure 資料目錄論壇](http://go.microsoft.com/fwlink/?LinkID=616424&clcid=0x409)。 那裡提出的問題會在這裡找到答案。
