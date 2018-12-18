---
title: 在使用 Azure Migrate 評估之後移轉機器 | Microsoft Docs
description: 說明如何在使用 Azure Migrate 服務執行評估之後，取得機器的移轉建議。
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: article
ms.date: 09/17/2018
ms.author: raynew
ms.openlocfilehash: 0b02ae4b75426b379ad7c124f5ddeb053c142ce6
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730289"
---
# <a name="migrate-machines-after-assessment"></a>評量之後移轉機器


[Azure Migrate](migrate-overview.md) 會評估內部部署機器，以確認機器是否適合移轉至 Azure，並提供在 Azure 中執行機器的大小調整建議和成本估計。 Azure Migrate 目前只會評估要移轉的機器。 系統目前是使用其他 Azure 服務來執行移轉作業本身。

本文會說明如何在執行移轉評估之後，取得移轉工具的建議。

## <a name="migration-tool-suggestion"></a>移轉工具的建議

若要取得有關移轉工具的建議，您必須深入探索內部部署環境。 若要深入探索，您必須在內部部署機器上安裝代理程式。  

1. 建立 Azure Migrate 專案、探索內部部署機器，然後建立移轉評估。 [深入了解](tutorial-assessment-vmware.md)。
2. 在您想要看到建議移轉方法的每個內部部署機器上，下載並安裝 Azure Migrate 代理程式。 [遵循此程序](how-to-create-group-machine-dependencies.md#prepare-for-dependency-visualization)來安裝代理程式。
2. 識別適合進行原形 (Lift and shift) 移轉的內部部署機器。 這些是不需要對其上執行的應用程式進行任何變更，可依原狀移轉的 VM。
3. 針對原形移轉，我們會建議使用 Azure Site Recovery。 [深入了解](../site-recovery/tutorial-migrate-on-premises-to-azure.md)。 或者，您可以使用支援移轉至 Azure 的第三方工具。
4. 如果您的內部部署機器不適合原形移轉，也就是說，如果您想要移轉特定應用程式，而不是整個 VM，您可以使用其他移轉工具。 例如，如果您想要移轉 SQL Server、MySQL 或 Oracle 之類的內部部署資料庫到 Azure，我們會建議 [Azure 資料庫移轉服務](https://azure.microsoft.com/campaigns/database-migration/)。


## <a name="review-suggested-migration-methods"></a>檢閱建議的移轉方法

1. 您必須先建立 Azure Migrate 專案、探索內部部署機器，然後執行移轉評估，然後才能取得建議的移轉方法。 [深入了解](tutorial-assessment-vmware.md)。
2. 建立評估之後，在 [專案] > [概觀] > [儀表板] 中檢視該評估。 按一下 [評估整備]

    ![評估整備](./media/tutorial-assessment-vmware/assessment-report.png)  

3. 在 [建議工具] 中，檢閱可用於移轉之工具的建議。

    ![建議的工具](./media/tutorial-assessment-vmware/assessment-suitability.png)




## <a name="next-steps"></a>後續步驟

[深入了解](concepts-assessment-calculation.md)評定的計算方式。
