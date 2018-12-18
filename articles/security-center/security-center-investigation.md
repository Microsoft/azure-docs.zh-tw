---
title: 在 Azure 資訊安全中心調查事件和警示 | Microsoft Docs
description: 這份文件可協助您使用 Azure 資訊安全中心的調查功能，以調查安全性事件和警示。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: mbaldwin
editor: ''
ms.assetid: a8e894a9-8781-4749-ae8f-8c8e01658566
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/29/2018
ms.author: terrylan
ms.openlocfilehash: 010c641c247552a8994c9d1c945cd585148fadf9
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37340785"
---
# <a name="investigate-incidents-and-alerts-in-azure-security-center-preview"></a>在 Azure 資訊安全中心調查事件和警示 (預覽)
這份文件可協助您使用 Azure 資訊安全中心的調查功能，以調查安全性事件和警示。

## <a name="what-is-investigation-in-security-center"></a>資訊安全中心的調查是什麼？
資訊安全中心的 [調查] 功能可讓您分級、了解範圍，並追蹤可能[安全性事件](https://docs.microsoft.com/azure/security-center/security-center-incident)的根本原因。

其目的是加速調查程序，方法是連結與您正在調查之事件有關的所有實體 ([安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)、使用者、電腦和事件)。  資訊安全中心可以藉由使相關資料與相關實體相互關聯，並且使用即時圖表 (協助您瀏覽物件及視覺化相關資訊) 公開此相互關聯，來完成這項工作。

> [!NOTE]
> 資訊安全中心的調查功能不支援[自訂警示](security-center-custom-alert.md)。
>
>


## <a name="how-investigation-works"></a>調查的運作方式？
調查是由佔用調查儀表板中央區域的圖表組成。 圖表永遠會將焦點放在特定實體，並且呈現與它相關的實體。 實體可以是安全性警示、使用者、電腦或事件。

![對應](./media/security-center-investigation/security-center-investigation-fig1.png)

使用者可以藉由在圖表中按一下某個實體，從該實體瀏覽到另一個實體。 圖表會自動將選取的實體及其相關實體置中。 不再相關的實體可能會從圖表中移除。

### <a name="investigation-path"></a>調查路徑
當使用者瀏覽至不同實體時，調查路徑會協助追蹤調查內容，並允許快速瀏覽。 包含調查結果的事件永遠是調查路徑中最左邊的事件。

![Path](./media/security-center-investigation/security-center-investigation-fig2.png)

### <a name="general-information"></a>一般資訊
當實體出現在圖表中時，索引標籤會顯示這個實體的其他資訊。 [資訊] 索引標籤會顯示各種可用資訊來源的實體一般資訊。

![一般資訊](./media/security-center-investigation/security-center-investigation-fig3.png)

[資訊] 索引標籤會顯示與對應中所選取之事件相關的資訊。 事件是一個容器，包括調查的結果。 每個調查會在事件內容中發生。

只有當使用者針對特定警示按一下 [開始調查] 按鈕時，才會建立事件。 調查的可用基本功能是標示實體，例如使用者、電腦或警示。 當實體標示為相關時，會提供原因。 從此時開始，這個實體會直接顯示在圖表和事件實體清單中的事件底下。

### <a name="entities"></a>實體

[實體] 索引標籤會顯示依據類型分組的所有相關實體。 它適用於兩種情況：圖表中有太多實體要顯示時，以及實體名稱太長時，使用表格式方式更容易檢查它們。

![實體](./media/security-center-investigation/security-center-investigation-fig4.png)

### <a name="search"></a>Search

[搜尋] 索引標籤會顯示實體可用的所有記錄類型。 您可以針對每個記錄類型，查看有多少記錄可用。 按一下每個記錄類型，系統就會帶領您前往搜尋畫面。 在搜尋畫面中，您可以調整您的搜尋，並使用不同的搜尋功能，例如設定警示。 在目前版本中，[搜尋] 索引標籤僅適用於使用者和電腦實體。

![Search](./media/security-center-investigation/security-center-investigation-fig5.png)

### <a name="exploration"></a>探索

[探索] 索引標籤可讓調查員檢查與各種問題 (與實體相關) 相關的資料。 例如，調查機器時，針對它執行的處理序清單會顯示在 [探索] 索引標籤中。在某些情況下，[探索] 索引標籤會呈現可能表示可疑問題的資料。 調查員可以檢查索引標籤內的資料，或在搜尋畫面中開啟以檢查大型資料集，以及使用進階搜尋選項，例如篩選和匯出至 Excel。

![探索](./media/security-center-investigation/security-center-investigation-fig6.png)

### <a name="timeline"></a>時間軸

圖表中顯示的大部分資料和各種索引標籤，是在特定時間週期相關。 此時間範圍是使用圖表左上方的時間範圍選取器進行設定。 調查員有各種方法可以選取時間範圍。

![時間軸](./media/security-center-investigation/security-center-investigation-fig7.png)

下列項目區分時間範圍：

- 圖表中的使用者電腦關聯性，只會顯示在此時間範圍內登入電腦的使用者。
- 當您檢查電腦和使用者時會顯示哪個警示：只會顯示在時間範圍內發生的警示。
- [實體] 索引標籤會遵循與圖表相同的邏輯。

不論選取的時間範圍，下列項目都會顯示：

- 顯示警示時，其中包含的所有實體 (例如使用者和電腦) 永遠都會顯示。
- 如果事件包含實體，則它會顯示。

> [!NOTE]
> [搜尋] 和 [探索] 索引標籤只會顯示這個時間範圍內的記錄。

## <a name="how-to-perform-an-investigation"></a>如何執行調查？

您可以從安全性事件或從警示開始調查，您所選擇的選項會根據您的需求而異。 從警示開始調查所採取的後續步驟：

1.  開啟 [資訊安全中心] 儀表板。
2.  按一下 [安全性警示]，然後選取您想要調查的事件。
3.  在事件的分頁上，按一下 [開始調查] 按鈕，[調查] 儀表板隨即出現。

    ![警示](./media/security-center-investigation/security-center-investigation-fig8.png)

4. 您可以從這個儀表板選取對應中的實體，此實體的相關資訊會出現在畫面右側。

    ![調查儀表板](./media/security-center-investigation/security-center-investigation-fig9.png)

從此刻起您可以探索此事件涉及的實體，並且探索更多詳細資料。

## <a name="see-also"></a>另請參閱
在本文件中，您已了解如何使用資訊安全中心的調查功能。 如要深入了解資訊安全中心，請參閱下列主題：

* [管理及回應 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)。 了解如何在資訊安全中心管理警示，以及回應安全性事件。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md)。 了解如何監視 Azure 資源的健全狀況。
* [了解 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 了解不同類型的安全性警示。
* [Azure 資訊安全中心疑難排解指南](https://docs.microsoft.com/azure/security-center/security-center-troubleshooting-guide)。 了解如何針對資訊安全中心的常見問題進行疑難排解。
* [Azure 資訊安全中心常見問題集](security-center-faq.md)。 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)。 尋找有關 Azure 安全性與相容性的部落格文章。
