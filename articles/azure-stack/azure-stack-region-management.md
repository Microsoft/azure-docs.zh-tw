---
title: Azure Stack 中的區域管理 | Microsoft Docs
description: Azure Stack 中的區域管理概觀。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: e94775d5-d473-4c03-9f4e-ae2eada67c6c
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/05/2018
ms.author: sethm
ms.reviewer: efemmano
ms.openlocfilehash: 401b81ceb7ab71528a4ad11bc7d8944b4d732933
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078849"
---
# <a name="region-management-in-azure-stack"></a>Azure Stack 中的區域管理

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 使用區域概念，也就是由邏輯實體組成硬體資源以形成 Azure Stack 基礎結構。 在區域管理內，您可以找到成功操作 Azure Stack 基礎結構所需的所有資源。

一個整合系統部署 (稱為「Azure Stack 雲端」) 可構成一個區域。 每個「Azure Stack 開發套件」都有一個名為 **local**的區域。 如果您部署第二個 Azure Stack 整合系統，或是在個別的硬體上設定另一個開發套件執行個體，則此 Azure Stack 雲端為不同的區域。

## <a name="information-available-through-the-region-management-tile"></a>透過區域管理圖格提供的資訊

Azure Stack 具有一組可用於 [區域管理] 磚的區域管理功能。 Azure Stack 操作員可以在系統管理員入口網站中的預設儀表板上存取此圖格。 您可以透過此磚來監視和更新特定區域的 Azure Stack 區域及其元件。

 ![區域管理磚](media/azure-stack-manage-region/image1.png)

 如果您按一下 [區域管理] 磚中的區域，則可以存取下列資訊：

  ![[區域管理] 刀鋒視窗上的窗格說明](media/azure-stack-manage-region/image2.png)

1. [資源功能表]。 您可以在這裡存取特定的基礎結構管理領域，以及檢視和管理使用者資源，例如儲存體帳戶和虛擬網路。

2. **警示**。 此項目會列出全系統警示，並提供每個警示的詳細資料。

3. **更新**。 您可以在此檢視 Azure Stack 基礎結構的目前版本、可用的更新，以及更新記錄。 您也可以更新您的整合式系統。

4. **資源提供者**。 資源提供者是可讓您管理使用者功能的位置，這些功能是由執行 Azure Stack 時所需的元件所提供。 每個資源提供者皆隨附系統管理體驗。 這項體驗可能包含特定提供者的警示、度量以及其他資源提供者專屬的管理功能。

5. **基礎結構角色**。 基礎結構角色是執行 Azure Stack 時所需的元件。 僅報告警示的基礎結構角色會列出。 您可以選取角色來檢視與角色相關聯的警示，以及此角色執行所在的角色執行個體。

## <a name="next-steps"></a>後續步驟

- [在 Azure Stack 中監視健康情況和警示](azure-stack-monitor-health.md)
- [在 Azure Stack 中管理更新](azure-stack-updates.md)
