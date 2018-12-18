---
title: 使用 Azure Site Recovery 在 Azure 區域之間移轉 Azure VM | Microsoft Docs
description: 使用 Azure Site Recovery，將 Azure IaaS VM 從一個 Azure 區域移轉至另一個區域。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 9bd5c1b2bad475dbb2c8ce216ed7d3f57ace0696
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49067609"
---
# <a name="migrate-azure-vms-to-another-region"></a>將 Azure VM 移轉到另一個區域

除了針對商務持續性和災害復原 (BCDR) 使用 [Azure Site Recovery](site-recovery-overview.md) 服務來管理和協調內部部署電腦和 Azure VM 的災害復原之外，您也可以使用 Site Recovery 來管理 Azure VM 到次要區域的移轉。 若要移轉 Azure VM，您可以為其啟用複寫，並將它們從主要區域容錯移轉至您選擇的次要區域。

本教學課程示範如何將 Azure VM 移轉至另一個區域。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立復原服務保存庫
> * 啟用 VM 複寫
> * 執行容錯移轉以移轉 VM

本教學課程假設您已有 Azure 訂用帳戶。 如果沒有，請在開始前建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。





## <a name="prerequisites"></a>必要條件

- 請確定 Azure 區域中有 Azure VM，以便您從中進行遷移。
- 請確定您了解[情節架構和元件](azure-to-azure-architecture.md)。
- 檢閱[支援限制和需求](azure-to-azure-support-matrix.md)。



## <a name="before-you-start"></a>開始之前

設定複寫之前，請完成這些步驟。


### <a name="verify-target-resources"></a>驗證目標資源

1. 確認您的 Azure 訂用帳戶允許您在用於災害復原的目標區域中建立 VM。 請連絡支援人員啟用所需的配額。

2. 確定您的訂用帳戶具有足夠的資源，可支援大小與來源 VM 相符的 VM。 Site Recovery 會為目標 VM 挑選相同的大小或最接近的大小。


### <a name="verify-account-permissions"></a>確認帳戶權限

如果您剛建立免費的 Azure 帳戶，則您會是訂用帳戶的系統管理員。 如果您不是訂用帳戶系統管理員，請與系統管理員合作以指派您所需的權限。 若要啟用新 VM 的複寫，您必須具有：

1. 在 Azure 資源中建立 VM 的權限。 「虛擬機器參與者」內建角色具有這些權限，包含：
    - 在所選取資源群組中建立 VM 的權限
    - 在所選取虛擬網路中建立 VM 的權限
    - 寫入所選取儲存體帳戶的權限

2. 您也需要管理 Azure Site Recovery 作業的權限。 「Site Recovery 參與者」角色具有在復原服務保存庫中管理 Site Recovery 作業所需的所有權限。


### <a name="verify-vm-outbound-access"></a>驗證 VM 輸出存取

1. 請確定您未使用驗證 Proxy 來控制您想要移轉之 VM 的網路連線能力。 
2. 基於本教學課程的用途，假設您想要移轉的 VM 可以存取網際網路，而且未使用防火牆 Proxy 來控制輸出存取。 如果您使用的話，請在[這裡](azure-to-azure-tutorial-enable-replication.md#configure-outbound-network-connectivity)檢查需求。

### <a name="verify-vm-certificates"></a>驗證 VM 憑證

檢查您要移轉的 Azure VM 上具有所有最新根憑證。 如果沒有最新的根憑證，VM 會因安全性條件限制而無法註冊至 Site Recovery。

- 若為 Windows VM，請安裝所有最新的 Windows 更新，讓所有的受信任根憑證都在機器上。 在中斷連線的環境中，請遵循您組織的標準 Windows Update 和憑證更新程序。
- 若為 Linux VM，請遵循 Linux 散發者提供的指引，以取得最新的受信任根憑證及 VM 的憑證撤銷清單。



## <a name="create-a-vault"></a>建立保存庫

在來源區域以外的任何區域中建立保存庫。

1. 登入 [Azure 入口網站](https://portal.azure.com) > [復原服務]。
2. 按一下 [建立群組] > [監視和管理] > [備份和 Site Recovery]。
3. 在 [名稱] 中，指定易記名稱 [ContosoVMVault]。 如果您有多個訂用帳戶，請選取適當的一個。
4. 建立資源群組 **ContosoRG**。
5. 指定 Azure 區域。 若要查看支援的區域，請參閱 [Azure Site Recovery 定價詳細資料](https://azure.microsoft.com/pricing/details/site-recovery/)。
6. 若要從儀表板快速存取保存庫，請按一下 [釘選到儀表板]，然後按一下 [建立]。

   ![新增保存庫](./media/tutorial-migrate-azure-to-azure/azure-to-azure-vault.png)

新的保存庫會新增到主要 [復原服務保存庫] 頁面上 [所有資源] 之下的 [儀表板]。






## <a name="select-the-source"></a>選取來源

1. 在復原服務保存庫中，按一下 [ConsotoVMVault] > [+複寫]。
2. 在 [來源] 中，選取 [Azure]。
3. 在 [來源位置] 中，選取 VM 目前執行所在的來源 Azure 區域。
4. 選取 Resource Manager 部署模型。 然後選取 [來源資源群組]。
5. 按一下 [確定]  來儲存設定。


## <a name="enable-replication-for-azure-vms"></a>啟用 Azure VM 的複寫

Site Recovery 會擷取與訂用帳戶和資源群組建立關聯的 VM 清單。


1. 在 Azure 入口網站中，按一下 [虛擬機器]。
2. 選取您要移轉的 VM。 然後按一下 [確定] 。
3. 在 [設定] 中，按一下 [災害復原]。
4. 在 [設定災害復原] >  [目標區域] 中，選取您要複寫至的目標區域。
5. 在本教學課程中，接受其他預設設定。
6. 按一下 [啟用複寫]。 這會開始一項作業來啟用 VM 的複寫。

    ![啟用複寫](media/tutorial-migrate-azure-to-azure/settings.png)

 

## <a name="run-a-failover"></a>執行容錯移轉

1. 在 [設定] > [複寫的項目] 中，按一下電腦，然後按一下 [容錯移轉]。
2. 在 [容錯移轉] 中，選取 [最新]。 加密金鑰設定不適用於此案例。
3. 選取 [Shut down machine before beginning failover] \(先將機器關機再開始容錯移轉)。 Site Recovery 嘗試先關閉來源 VM，再觸發容錯移轉。 即使關機失敗，仍會繼續容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
4. 確認 Azure VM 如預期般出現在 Azure 中。
5. 在 [複寫的項目] 中，以滑鼠右鍵按一下 VM > [認可]。 這會完成移轉程序。
6. 認可完成之後，請按一下 [停用複寫]。  這會停止 VM 的複寫。



## <a name="next-steps"></a>後續步驟

在本教學課程中，您已將 Azure VM 移轉至不同的 Azure 區域。 您現在可以設定已移轉 VM 的災害復原。

> [!div class="nextstepaction"]
> [在移轉之後設定災害復原](azure-to-azure-quickstart.md)

