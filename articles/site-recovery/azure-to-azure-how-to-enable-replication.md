---
title: 在 Azure Site Recovery 中設定 Azure VM 的複寫 | Microsoft Docs
description: 本文說明如何使用 Site Recovery，將 Azure VM 從一個 Azure 區域複寫到另一個區域。
services: site-recovery
author: asgang
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: asgang
ms.openlocfilehash: e7cd3032053b3628b94f93f3c7e00b6890afd4ca
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37916277"
---
# <a name="replicate-azure-virtual-machines-to-another-azure-region"></a>將 Azure 虛擬機器複寫到另一個 Azure 區域



本文說明如何將 Azure VM 從一個 Azure 區域複寫到另一個區域。

## <a name="prerequisites"></a>先決條件

本文假設您已經設定適用於此案例的 Site Recovery，如 [Azure 至 Azure 教學課程](azure-to-azure-tutorial-enable-replication.md)中所述。 確定您已備妥必要條件，並建立了復原服務保存庫。



## <a name="enable-replication"></a>啟用複寫

啟用複寫。 此程序假設主要 Azure 區域為東亞，而次要區域為東南亞。

1. 在保存庫中，按一下 [+複寫]。
2. 請注意以下欄位：
    - **來源**：VM 的起點，在此案例中為 **Azure**。
    - **來源位置**：您想要保護虛擬機器的 Azure 區域。 在此圖例中，來源位置是 [東亞]
    - **部署模型**：來源機器的 Azure 部署模型。
    - **資源群組**：來源虛擬機器所屬的資源群組。 下一個步驟會列出所選取資源群組下的所有 VM，以供保護。

    ![啟用複寫](./media/site-recovery-replicate-azure-to-azure/enabledrwizard1.png)

3. 在 [虛擬機器] > [選取虛擬機器] 中，按一下並選取您要複寫的每部 VM。 您只能選取可以啟用複寫的機器。 然後按 [下一步] 。
    ![啟用複寫](./media/site-recovery-replicate-azure-to-azure/virtualmachine_selection.png)

4. 在 [設定] 中，您可以選擇性地設定目標網站設定：

    - **目標位置**：將複寫來源虛擬機器資料的位置。 端視您選取的機器位置而定，Site Recovery 將提供適當目標區域的清單。 建議您讓目標位置與復原服務保存庫位置保持相同。
    - **目標資源群組**：所有已複寫虛擬機器所屬的資源群組。 根據預設，Azure Site Recovery 會在目標區域中建立名稱尾碼為 "asr" 的新資源群組。 如果 Azure Site Recovery 建立的資源群組已經存在，則會重複使用。 您也可以選擇對它進行自訂，如下所示。 目標資源群組的位置可以是任何 Azure 區域，但是裝載您來源虛擬機器所在的區域除外。
    - **目標虛擬網路**：根據預設，Site Recovery 將在目標區域中建立名稱尾碼為 "asr" 的新虛擬網路。 這會對應至您的來源網路，並用來進行任何未來的保護。 [深入了解](site-recovery-network-mapping-azure-to-azure.md)網路對應。
    - **目標儲存體帳戶 (如果您的來源 VM 不使用受控磁碟)**：根據預設，Site Recovery 會建立新的目標儲存體帳戶，以模擬您的來源 VM 儲存體設定。 如果儲存體帳戶已經存在，就會重複使用。
    - **複本受控磁碟 (如果您的來源 VM 使用受控磁碟)**：Site Recovery 會在目標區域中建立新的複本受控磁碟，建立與來源 VM 的受控磁碟相同儲存類型 (標準或進階) 之來源 VM 受控磁碟的鏡像。
    - **快取儲存體帳戶**：Site Recovery 需要在來源區域中有額外的儲存體帳戶 (稱為快取儲存體)。 在來源 VM 上發生的所有變更都會受到追蹤，並傳送到快取儲存體帳戶，然後複寫到目標位置。
    - **可用性設定組**：根據預設，Azure Site Recovery 會在目標區域中建立名稱尾碼為 "asr" 的新可用性設定組。 如果 Azure Site Recovery 建立的可用性設定組已經存在，則會重複使用。
    - **複寫原則**：這會定義復原點保留期歷程記錄和應用程式一致快照集頻率的設定。 根據預設，Azure Site Recovery 會對於復原點保留期使用「24 小時」的預設設定來建立新的複寫原則，並對於應用程式一致快照集頻率使用「60 分鐘」。

    ![啟用複寫](./media/site-recovery-replicate-azure-to-azure/enabledrwizard3.PNG)

## <a name="customize-target-resources"></a>自訂目標資源

您可以修改 Site Recovery 所使用的預設目標設定。

1. 按一下 [自訂:] 以修改預設設定：
    - 在 [目標資源群組] 中，從訂用帳戶目標位置上的所有資源群組清單中選取資源群組。
    - 在 [目標虛擬網路] 中，從目標位置上的所有虛擬網路清單中選取網路。
    - 在 [可用性設定組] 中，如果 VM 是來源區域中可用性設定組的一部分，即可將可用性設定組新增至其中。
    - 在 [目標儲存體帳戶] 中，選取您要使用的帳戶。

        ![啟用複寫](./media/site-recovery-replicate-azure-to-azure/customize.PNG)

2. 按一下 [建立目標資源] > [啟用複寫]。
3. 啟用 VM 以進行複寫之後，您就可以在 [複寫的項目] 下方檢查 VM 健康情況的狀態。

>[!NOTE]
>在初始複寫期間，狀態可能需要一些時間才會重新整理，期間不會有任何進展。 按一下 [重新整理] 按鈕來取得最新狀態。
>

# <a name="next-steps"></a>後續步驟

[深入了解](site-recovery-test-failover-to-azure.md)執行測試容錯移轉。
