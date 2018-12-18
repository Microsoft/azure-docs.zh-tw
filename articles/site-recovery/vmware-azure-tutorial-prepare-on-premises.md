---
title: 準備內部部署 VMware 伺服器以進行 VMware VM 至 Azure 的災害復原| Microsoft Docs
description: 了解如何準備內部部署 VMware 伺服器，以使用 Azure Site Recovery 服務來進行 Azure 的災害復原。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/06/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: facf8895770f890bfbbef946a32cc681f685e998
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37915197"
---
# <a name="prepare-on-premises-vmware-servers-for-disaster-recovery-to-azure"></a>準備內部部署 VMware 伺服器以進行 Azure 的災害復原

[Azure Site Recovery](site-recovery-overview.md) 藉由確保您的商務應用程式可在計劃性與非計劃性中斷期間持續啟動並執行，來提供商務持續性和災害復原 (BCDR) 策略。 Site Recovery 會管理並協調內部部署機器和 Azure 虛擬機器 (VM) 的災害復原，包括複寫、容錯移轉和復原。

- 這是一系列中的第二個教學課程，說明如何為內部部署 VMware VM 設定 Azure 的災害復原。 在第一個教學課程中，我們針對 VMware 災害復原需求[設定 Azure 元件](tutorial-prepare-azure.md)。
- 這些教學課程接設計成顯示案例的最簡單部署路徑。 可能的話，會使用預設選項，而不會顯示所有可能的設定與路徑。 

在本文中，我們會說明在您想要使用 Azure Site Recovery 將 VMware VM 複寫至 Azure 時，如何備妥內部部署 VMware 基礎結構。 您會了解如何：

> [!div class="checklist"]
> * 在 vCenter Server 或 vSphere ESXi 主機上，準備一個用來將 VM 探索自動化的帳戶
> * 準備一個用來在 VMware VM 上自動安裝行動服務的帳戶
> * 檢閱 VMware 伺服器和 VM 需求
> * 準備在容錯移轉後連接到 Azure VM



## <a name="prepare-an-account-for-automatic-discovery"></a>準備帳戶以進行自動探索

Site Recovery 需要存取 VMware 伺服器才能：

- 自動探索 VM。 需要至少一個唯讀帳戶。
- 協調複寫、容錯移轉和容錯回復。 您需要可執行建立和移除磁碟以及開啟 VM 電源等作業的帳戶。

遵循下列方式建立此帳戶：

1. 若要使用專用帳戶，請在 vCenter 層級建立一個角色。 指定角色的名稱，例如 **Azure_Site_Recovery**。
2. 將下表摘要說明的權限指派給角色。
3. 在 vCenter 伺服器或 vSphere 主機上建立使用者。 將角色指派給使用者。

### <a name="vmware-account-permissions"></a>VMware 帳戶權限

**Task** | **角色/權限** | **詳細資料**
--- | --- | ---
**VM 探索** | 至少是唯讀使用者<br/><br/> 資料中心物件 –> 傳播至子物件、role=Read-only | 在資料中心層級指派的使用者，且能夠存取資料中心內的所有物件。<br/><br/> 如果要限制存取權，請將具備 [傳播至子物件] 權限的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、VM 及網路)。
**完整複寫、容錯移轉、容錯回復** |  建立具有必要權限的角色 (Azure_Site_Recovery)，然後將角色指派給 VMware 使用者或群組<br/><br/> 資料中心物件 –> 傳播至子物件、role=Azure_Site_Recovery<br/><br/> 資料存放區 -> 配置空間、瀏覽資料存放區、底層檔案作業、移除檔案、更新虛擬機器檔案<br/><br/> 網路 -> 網路指派<br/><br/> 資源 -> 指派 VM 至資源集區、移轉已關閉電源的 VM、移轉已開啟電源的 VM<br/><br/> 工作 -> 建立工作、更新工作<br/><br/> 虛擬機器 -> 組態<br/><br/> 虛擬機器 -> 互動 -> 回答問題、裝置連線、設定 CD 媒體、設定磁碟片媒體、電源關閉、電源開啟、VMware 工具安裝<br/><br/> 虛擬機器 -> 清查 -> 建立、註冊、取消註冊<br/><br/> 虛擬機器 -> 佈建 -> 允許虛擬機器下載、允許虛擬機器檔案上傳<br/><br/> 虛擬機器 -> 快照 -> 移除快照 | 在資料中心層級指派的使用者，且能夠存取資料中心內的所有物件。<br/><br/> 如果要限制存取權，請將具備 [傳播至子物件] 權限的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、VM 及網路)。

## <a name="prepare-an-account-for-mobility-service-installation"></a>準備一個用來安裝行動服務的帳戶

行動服務必須安裝在您要複寫的電腦上。 Site Recovery 可以在您為電腦啟用複寫時，進行此服務的推送安裝，您也可以手動進行安裝，或使用安裝工具。

- 在本教學課程中，我們即將使用推送安裝來安裝行動服務。
- 針對此推送安裝，就必須準備一個可供 Site Recovery 用來存取 VM 的帳戶。 當您在 Azure 主控台中設定災害復原時，您會指定此帳戶。

遵循下列方式準備此帳戶：

準備有權限可以在 VM 上安裝的網域或本機帳戶。

- **Windows VMs**：若要在 Windows VM 上安裝，如果您不使用網域帳戶，請停用本機電腦上的遠端使用者存取控制。 若要執行此動作，請在登錄的 **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System** 下，新增 DWORD 登錄 **LocalAccountTokenFilterPolicy**，其值為 1。
- **Linux VMs**：若要在 Linux VM 上安裝，請在來源 Linux 伺服器上準備根帳戶。


## <a name="check-vmware-requirements"></a>檢查 VMware 需求

確定 VMware 伺服器和 VM 符合下列需求。

1. [檢查](vmware-physical-azure-support-matrix.md#on-premises-virtualization-servers) VMware 伺服器需求。
2. 針對 Linux VM，[檢查](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage)檔案系統和儲存體需求。 
3. 檢查內部部署[網路](vmware-physical-azure-support-matrix.md#network)和[儲存體](vmware-physical-azure-support-matrix.md#storage)支援。 
4. 檢查在容錯移轉之後，[Azure 網路](vmware-physical-azure-support-matrix.md#azure-vm-network-after-failover)、[儲存體](vmware-physical-azure-support-matrix.md#azure-storage)和[計算](vmware-physical-azure-support-matrix.md#azure-compute)支援的項目。
5. 您複寫到 Azure 的內部部署 VM 必須符合 [Azure VM 需求](vmware-physical-azure-support-matrix.md#azure-vm-requirements)。


## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

在容錯移轉後，您可以從內部部署網路連線至 Azure VM。

若要在容錯移轉後使用 RDP 連線到 Windows VM，請執行下列作業：

- **網際網路存取**。 在容錯移轉前，請在內部部署 VM 上啟用 RDP。 確定已針對 [公用] 設定檔新增 TCP 和 UDP 規則，且在 [Windows 防火牆] > [允許的應用程式] 中已針對所有設定檔允許 RDP。
- **站對站 VPN 存取**：
    - 在容錯移轉前，請在內部部署電腦上啟用 RDP。
    - 您應該在 [Windows 防火牆] -> [允許的應用程式與功能] 中，針對 [網域] 和 [私人] 網路允許 RDP。
    - 確認作業系統的 SAN 原則已設為 [OnlineAll]。 [深入了解](https://support.microsoft.com/kb/3031135)。
- 觸發容錯移轉時，VM 上不應該有任何擱置的 Windows 更新。 如果有，在更新完成之前，您將無法登入虛擬機器。
- 在容錯移轉之後，於 Windows Azure VM 上，勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。 如果您無法連線，請檢查 VM 是否正在執行，並檢閱這些[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。

若要在容錯移轉後使用 SSH 連線到 Linux VM，請執行下列作業：

- 在容錯移轉之前，於內部部署機器上，確認安全殼層服務已設定為在系統開機時自動啟動。
- 確認防火牆規則允許 SSH 連線。
- 在容錯移轉之後，於 Azure VM 上，針對已容錯移轉之 VM 上的網路安全性群組規則及其所連線的 Azure 子網路，允許 SSH 連接埠的連入連線。
- [新增 VM 的公用 IP 位址](site-recovery-monitoring-and-troubleshooting.md)。
- 您可以勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。

## <a name="useful-links"></a>實用的連結

如果您要複寫多個 VM，您應該在開始前規劃容量和部署。 [深入了解](site-recovery-deployment-planner.md)。



## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [為 VMware VM 設定 Azure 的災害復原](vmware-azure-tutorial.md)
