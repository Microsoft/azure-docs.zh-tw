---
title: 在 Azure Site Recovery 中建立及自訂用於容錯移轉和復原的復原方案 | Microsoft Docs
description: 了解如何在 Azure Site Recovery 中建立和自訂復原方案。 本文說明如何容錯移轉及復原虛擬機器和實體伺服器。
services: site-recovery
documentationcenter: ''
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: 145b23aa7b8e48a0301676595c8d59a64bebdd0b
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39226663"
---
# <a name="create-and-customize-recovery-plans"></a>建立並自訂復原方案

本文說明如何在 [Azure Site Recovery](site-recovery-overview.md) 中建立和自訂復原方案。 開始之前，請[深入了解](recovery-plan-overview.md)復原方案。

## <a name="create-a-recovery-plan"></a>建立復原計畫

1. 在復原服務保存庫中，選取 [復原方案 (Site Recovery)] > [+ 復原方案]。
2. 在 [建立復原方案] 中，指定方案的名稱。
3. 根據方案中的機器選擇來源和目標，並針對部署模型選取 [資源管理員]。 來源位置必須有已啟用容錯移轉和復原功能的機器。 

   **容錯移轉** | **來源** | **目標** 
   --- | --- | ---
   Azure 至 Azure | Azure 區域 |Azure 區域
   VMware 至 Azure | 組態伺服器 | Azure
   實體機器至 Azure | 組態伺服器 | Azure   
   由 VMM 管理的 Hyper-V 至 Azure  | VMM 顯示名稱 | Azure
   不具 VMM 的 Hyper-V 至 Azure | Hyper-V 網站名稱 | Azure
   VMM 至 VMM |VMM 易記名稱 | VMM 顯示名稱 

   > [!NOTE]
   > 復原方案可以包含具有相同來源和目標的機器。 VMM 管理的 VMware 和 Hyper-V VM 不得位於相同的方案中。 VMware VM 和實體伺服器可位於相同的方案中，其中來源是組態伺服器。

2. 在 [選取項目虛擬機器] 中，選取您要新增至方案的機器 (或複寫群組)。 然後按一下 [確定] 。
    - 機器會新增至方案中的預設群組 (群組 1)。 容錯移轉之後，此群組中的所有機器會在相同的時間啟動。
    - 您只能選取您所指定之來源和目標位置中的機器。 
1. 按一下 [確定] 以建立方案。

## <a name="add-a-group-to-a-plan"></a>將群組新增至方案

您可以建立其他群組，並將機器新增至不同的群組，以便您可以為每個群組指定不同的行為。 例如，您可以指定群組中的機器在容錯移轉之後應該啟動的時間，或指定每個群組的自訂動作。

1. 在 [復原方案] 中，以滑鼠右鍵按一下方案 > [自訂]。 根據預設，建立方案之後，您新增至方案的所有機器都會位在預設群組 1 中。
2. 按一下 [+ 群組]。 依預設，新的群組會依其新增的順序進行編號。 最多可擁有七個群組。
3. 選取您需要移至新群組的機器，按一下 [變更群組]，然後選取新的群組。 或者，以滑鼠右鍵按一下群組名稱 > [受保護項目]，並將機器新增至該群組。 機器或複寫群組只能屬於復原方案中的一個群組。


## <a name="add-a-script-or-manual-action"></a>新增指令碼或手動動作

您可以透過新增指令碼或手動動作來自訂復原方案。 請注意：

- 如果您要複寫至 Azure，可以將 Azure 自動化 Runbook 整合至您的復原方案。 [深入了解](site-recovery-runbook-automation.md)。
- 如果您要複製 System Center VMM 所管理的 Hyper-V VM，可以在內部部署 VMM 伺服器上建立指令碼，並將它包含在復原方案中。
- 新增指令碼後，群組中會加入新的動作集。 例如，系統會為群組 1 建立一組前置步驟，並且命名為「群組 1：前置步驟」。 所有前置步驟都會列在此動作集內。 只有在部署 VMM 伺服器後，您才能在主要網站上新增指令碼。
- 如果您新增手動動作，在執行復原方案時，它會在您插入手動動作之處停止。 對話方塊會提示您指定手動動作已完成。
- 若要在 VMM 伺服器上建立指令碼，請依照[本文](hyper-v-vmm-recovery-script.md)中的指示。
- 在容錯移轉至次要站台期間，以及從次要站台容錯回復到主要站台期間，可以套用指令碼。 支援取決於您的複寫案例：
    
    **案例** | **容錯移轉** | **容錯回復**
    --- | --- | --- 
    Azure 至 Azure  | Runbook | Runbook
    VMware 至 Azure | Runbook | NA 
    具有 VMM 的 Hyper-V 至 Azure | Runbook | 指令碼
    從 Hyper-V 站台到 Azure | Runbook | NA
    VMM 至次要 VMM | 指令碼 | 指令碼

1. 在復原方案中，按一下要對其新增動作的步驟，並指定應該發生動作的時間：a. 如果您在群組中的機器於容錯移轉後啟動之前就需要動作發生，請選取 [新增前置動作]。
    b. 如果您在群組中的機器於容錯移轉後啟動之後才需要動作發生，請選取 [新增後置動作]。 若要移動動作的位置，請選取 [上移] 或 [下移] 按鈕。
2. 在 [插入動作] 中，選取 [指令碼] 或 [手動動作]。
3. 如果您需要新增手動動作，請執行下列動作：a. 輸入動作的名稱，然後輸入動作指示。 執行容錯移轉的使用者會看到這些指示。
    b. 指定您是否要針對所有類型的容錯移轉 (測試、容錯移轉、規劃的容錯移轉 (如果相關)) 新增手動動作。 然後按一下 [確定] 。
4. 如果您需要新增指令碼，請執行下列動作：a. 如果是新增 VMM 指令碼，請選取 [容錯移轉至 VMM 指令碼]，然後在 [指令碼路徑] 中輸入共用的相對路徑。 例如，如果共用的位置為 \\<VMMServerName>\MSSCVMMLibrary\RPScripts，則指定路徑：\RPScripts\RPScript.PS1。
    b. 如果是新增 Azure 自動化 Runbook，請指定 Runbook 所在的 [Azure 自動化帳戶]，並選取適當的 [Azure Runbook 指令碼]。
5. 請執行復原方案的測試容錯移轉，以確保指令碼會如預期般運作。

## <a name="watch-a-video"></a>觀看影片

觀看示範如何建立復原方案的影片。


> [!VIDEO https://www.youtube.com/embed/1KUVdtvGqw8]

## <a name="next-steps"></a>後續步驟

深入了解[如何執行容錯移轉](site-recovery-failover.md)。  

    
