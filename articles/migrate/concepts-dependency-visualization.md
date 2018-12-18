---
title: Azure Migrate 的相依性視覺效果 | Microsoft Docs
description: 概括介紹 Azure Migrate 服務中的評量計算。
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: raynew
ms.openlocfilehash: 923a2a137bb4510e9490ce4077f744a43619a2c6
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165019"
---
# <a name="dependency-visualization"></a>相依性視覺效果

[Azure Migrate](migrate-overview.md) 服務會評估要移轉至 Azure 的內部部署機器群組。 您可以使用 Azure Migrate 的相依性視覺效果功能來建立群組。 此文章提供此功能的相關資訊。


## <a name="overview"></a>概觀

Azure Migrate 的相依性視覺效果可建立用於移轉評估且值得高度信賴的群組。 使用相依性視覺效果，您可以檢視機器的網路相依性，並找出有必要一起移轉到 Azure 的相關機器。 當您並不完全了解構成應用程式和需要一起移轉到 Azure 的機器時，此功能會很實用。

## <a name="how-does-it-work"></a>運作方式

為提供相依性的視覺效果，Azure Migrate 採用了 [Log Analytics](../log-analytics/log-analytics-overview.md) 的[服務對應](../operations-management-suite/operations-management-suite-service-map.md)解決方案。
- 若要利用相依性視覺效果，您需要將新增或現有的 Log Analytics 工作區與 Azure Migrate 專案建立關聯。
- 您只能在建立移轉專案的相同訂用帳戶中，建立或連結工作區。
- 若要將 Log Analytics 工作區連結至專案，請移至專案 [概觀] 頁面的 [基本資訊] 區段，按一下 [需要設定]

    ![與 Log Analytics 工作區建立關聯](./media/concepts-dependency-visualization/associate-workspace.png)

- 建立新工作區時，您必須指定工作區的名稱。 然後會在和移轉專案相同之 [Azure 地理區](https://azure.microsoft.com/global-infrastructure/geographies/)的區域中建立工作區。
- 相關聯的工作區會以索引鍵 **Migration Project** 和**專案名稱**的值標記，您可用來在 Azure 入口網站中搜尋。
- 若要瀏覽到與專案相關聯的工作區，您可以移至專案 [概觀] 頁面的 [基本資訊] 區段，然後存取該工作區

    ![瀏覽 Log Analytics 工作區](./media/concepts-dependency-visualization/oms-workspace.png)

若要使用相依性視覺效果，您需要在待分析的每個內部部署機器上，下載及安裝代理程式。  

## <a name="do-i-need-to-pay-for-it"></a>我需要支付多少費用？

不須額外費用即可使用 Azure Migrate。 使用 Azure Migrate 的相依性視覺效果功能需要服務對應，並需要將 Log Analytics 工作區與全新或現有 Azure Migrate 專案建立關聯。 Azure Migrate 的相依性視覺效果功能前 180 天不收取任何費用。

1. 在此 Log Analytics 工作區內使用服務對應以外的任何解決方案，將會產生[標準 Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) 費用。
2. 為了支援不需額外成本的移轉案例，自 Log Analytics 工作區與 Azure Migrate 專案建立關聯當天算起的前 180 天，服務對應解決方案將不會產生任何費用，而 180 天則適用標準收費方式。 經過 180 天後，會套用標準 Log Analytics 費用。

當您將代理程式註冊至工作區時，請使用專案在安裝代理程式步驟頁面上提供的識別碼和金鑰。

刪除 Azure Migrate 專案時，工作區不會隨之刪除。 在專案刪除後，服務對應就無法免費使用，系統會根據 Log Analytics 工作區的付費層向每個節點收取費用。

> [!NOTE]
> 相依性視覺效果功能會透過 Log Analytics 工作區使用服務對應。 Azure Migrate 於 2018 年 2 月 28 日 宣告正式運作，現在可以免費使用該功能。 您必須建立新的專案，才能使用免費的工作區。 在正式運作之前存在的工作區仍需收費，因此建議您移至新的專案。

[在此](https://azure.microsoft.com/pricing/details/azure-migrate/)深入了解 Azure Migrate 定價。

## <a name="how-do-i-manage-the-workspace"></a>如何管理工作區？

您可以使用 Azure Migrate 外部的 Log Analytics 工作區。 如果您刪除建立工作區的移轉專案，並不會刪除工作區。 如果不再需要工作區中，請手動[刪除工作區](../log-analytics/log-analytics-manage-access.md)。

除非您刪除移轉專案，否則請勿刪除 Azure Migrate 建立的工作區。 如果這樣做，相依性視覺效果功能不會如預期般運作。

## <a name="next-steps"></a>後續步驟
- [使用機器相依性的群組機器](how-to-create-group-machine-dependencies.md)
- [深入了解](https://docs.microsoft.com/azure/migrate/resources-faq#dependency-visualization)相依性視覺效果常見問題集。
