---
title: 使用 Azure Site Recovery 將 VMware VM 或實體伺服器複寫至次要 VMware 網站的支援矩陣 | Microsoft Docs
description: 摘要說明使用 Azure Site Recovery 將 VMware/實體伺服器複寫至次要網站的支援
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: 2ea1c337b4193997039876a05e71fb0088d455b1
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37923406"
---
# <a name="support-matrix-for-replication-of-vmware-vms-and-physical-servers-to-a-secondary-site"></a>將 VMware VM 和實體伺服器複寫至次要網站的支援矩陣

本文摘要說明使用 [Azure Site Recovery ](site-recovery-overview.md) 服務將 VMware VM 或 Windows/Linux 實體伺服器複寫至次要 VMware 網站時所支援的項目。

- 如果您需要將 VMware VM 或實體伺服器複寫至 Azure，請檢閱[此支援矩陣](vmware-physical-azure-support-matrix.md)。
- 如果您需要將 Hyper-V VM 複寫至次要網站，請檢閱[此支援矩陣](hyper-v-azure-support-matrix.md)。

> [!NOTE]
> 內部部署 VMware VM 和實體伺服器的複寫是由 InMage Scout 提供。 InMage Scout 包含在 Azure Site Recovery 服務訂用帳戶中。


## <a name="host-servers"></a>主機伺服器

**作業系統** | **詳細資料**
--- | ---
vCenter 伺服器 | vCenter 5.5、6.0 和 6.5<br/><br/> 如果您是執行 6.0 或 6.5，請注意，僅支援 5.5 的功能。


## <a name="replicated-vm-support"></a>已複寫的 VM 支援

下表摘要說明使用 Site Recovery 複寫的機器支援的作業系統。 任何工作負載都可以在支援的作業系統上執行。

**作業系統** | **詳細資料**
--- | ---
Windows Server | 64 位元的 Windows Server 2016、Windows Server 2012 R2、Windows Server 2012、Windows Server 2008 R2 (至少含 SP1)。
Linux | Red Hat Enterprise Linux 6.7、6.8、6.9、7.1、7.2 <br/><br/> Centos 6.5、6.6、6.7、6.8、6.9、7.0、7.1、7.2 <br/><br/> Oracle Enterprise Linux 6.4、6.5 或 6.8，執行 Red Hat 相容核心或 Unbreakable Enterprise Kernel 第 3 版 (UEK3) <br/><br/> SUSE Linux Enterprise Server 11 SP3、11 SP4 


## <a name="linux-machine-storage"></a>Linux 機器儲存體

只能複寫有下列儲存體的 Linux 機器：

- 檔案系統 (EXT3、ETX4、ReiserFS、XFS)。
- 多重路徑軟體裝置對應程式。
- 磁碟區管理員 (LVM2)。
- 不支援使用 HP CCISS 控制站儲存體的實體伺服器。
- 只有在 SUSE Linux Enterprise Server 11 SP3 上才支援 ReiserFS 檔案系統。

## <a name="network-configuration---hostguest-vm"></a>網路設定 - 主機/客體 VM

**組態** | **支援**  
--- | --- 
主機 - NIC 小組 | yes 
主機 - VLAN | yes 
主機 - IPv4 | yes 
主機 - IPv6 | 否 
客體 VM - NIC 小組 | 否
客體 VM - IPv4 | yes
客體 VM - IPv6 | 否
客體 VM - Windows/Linux - 靜態 IP 位址 | yes
客體 VM - 多重 NIC | yes


## <a name="storage"></a>儲存體

### <a name="host-storage"></a>主機儲存體

**儲存體 (主機)** | **支援** 
--- | --- 
NFS | yes 
SMB 3.0 | N/A 
SAN (ISCSI) | yes 
多重路徑 (MPIO) | yes 

### <a name="guest-or-physical-server-storage"></a>客體或實體伺服器儲存體

**組態** | **支援** 
--- | --- 
VMDK | yes 
VHD/VHDX | N/A 
第 2 代 VM | N/A 
共用叢集磁碟 | yes 
已加密磁碟 | 否 
UEFI| yes 
NFS | 否 
SMB 3.0 | 否 
RDM | yes 
磁碟 > 1 TB | yes 
使用等量磁碟的磁碟區 > 1 TB<br/><br/> LVM | yes 
儲存空間 | 否 
熱新增/移除磁碟 | yes 
排除磁碟 | yes 
多重路徑 (MPIO) | N/A 

## <a name="vaults"></a>保存庫

**Action** | **支援** 
--- | --- 
跨資源群組間移動保存庫 (在訂用帳戶之內或跨訂用帳戶) | 否 
跨資源群組間移動儲存體、網路、Azure VM (在訂用帳戶之內或跨訂用帳戶) | 否 

## <a name="mobility-service-and-updates"></a>行動服務和更新

行動服務會協調內部部署 VMware 伺服器或實體伺服器和次要網站之間的複寫。 當您設定複寫時，必須確定您具有最新版本的行動服務，以及最新版本的其他元件。

更新 | **詳細資料** 
--- | --- 
Scout 更新 | [了解並下載](/vmware-physical-secondary-disaster-recovery.md#updates)最新的 Scout 更新 | Scout 更新是累計的。
元件更新 | Scout 更新包含所有元件的更新，包括 RX 伺服器、設定伺服器、處理序和主要目標伺服器、vContinuum 伺服器，以及您要保護的來源伺服器。<br/><br/> [深入了解](vmware-physical-secondary-disaster-recovery.md#download-and-install-component-updates)。


## <a name="next-steps"></a>後續步驟

下載 [InMage Scout 使用者指南](https://aka.ms/asr-scout-user-guide)

- [將 VMM 雲端中的 Hyper-V VM 複寫至次要網站](tutorial-vmm-to-vmm.md)
- [將 VMware VM 和實體伺服器複寫至次要網站](tutorial-vmware-to-vmware.md)
