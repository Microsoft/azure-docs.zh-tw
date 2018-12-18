---
title: 用於從 Azure 複寫至 Azure 的 Azure Site Recovery 支援矩陣 | Microsoft Docs
description: 摘要說明支援的作業系統，以及 Azure 虛擬機器 (VM) 的 Azure Site Recovery 複寫設定 (對於嚴重損壞修復 (DR) 需求，從一個區域複寫至另一個區域)。
services: site-recovery
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.date: 09/10/2018
ms.author: sujayt
ms.openlocfilehash: 0e9c5c7ebaaa0a51d723340751e8ea82bfb96b5e
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433662"
---
# <a name="support-matrix-for-replicating-from-one-azure-region-to-another"></a>從一個 Azure 區域複寫至另一個區域的支援矩陣



此文章摘要說明使用 [Azure Site Recovery](site-recovery-overview.md) 服務將 Azure 虛擬機器從一個區域複寫及復原至另一個區域時，所支援的設定和元件。

## <a name="user-interface-options"></a>使用者介面選項

**使用者介面** |  **支援 / 不支援**
--- | ---
**Azure 入口網站** | 支援
**PowerShell** | [使用 PowerShell 進行 Azure 到 Azure 複寫](azure-to-azure-powershell.md)
**REST API** | 目前不支援
**CLI** | 目前不支援


## <a name="resource-support"></a>資源支援

**資源移動類型** | **詳細資料**
--- | --- | ---
**在資源群組間移動保存庫** | 不支援<br/><br/> 您無法跨資源群組移動復原服務保存庫。
**跨資源群組移動計算/儲存體/網路資源** | 不支援。<br/><br/> 如果您在 VM 或是相關聯的元件 (例如儲存體/網路) 複寫之後移動它們，您必須停用複寫，然後重新啟用該 VM 的複寫。
**將 Azure VM 從某個訂用帳戶複寫至另一個以進行災害復原** | 在相同的 Azure Active Directory 租用戶中支援。 傳統 VM 不支援。
**在支援的地理叢集 (內及跨訂用帳戶) 內移轉區域之間的 VM** | 在支援的「Resource Manager 部署模型」VM 的相同 Azure Active Directory 租用戶內支援。 「傳統部署模型」VM 則不支援。
**在相同區域內移轉 VM** | 不支援。


## <a name="support-for-replicated-machine-os-versions"></a>支援多種複寫機器作業系統版本

下列支援適用於在上述 OS 中執行的任何工作負載。

#### <a name="windows"></a>Windows

- Windows Server 2016 (伺服器核心、含有桌面體驗的伺服器)*
- Windows Server 2012 R2
- Windows Server 2012
- Windows Server 2008 R2 (至少加裝 SP1)

>[!NOTE]
>
> 不支援 \* Windows Server 2016 Nano Server。

#### <a name="linux"></a>Linux

- Red Hat Enterprise Linux 6.7、6.8、6.9、6.10、7.0、7.1、7.2、7.3、7.4、7.5   
- CentOS 6.5、6.6、6.7、6.8、6.9、6.10、7.0、7.1、7.2、7.3、7.4、7.5
- Ubuntu 14.04 LTS 伺服器 [(支援的核心版本)](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
- Ubuntu 16.04 LTS 伺服器 [(支援的核心版本)](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
- Debian 7 [ (支援的核心版本)](#supported-debian-kernel-versions-for-azure-virtual-machines)
- Debian 8 [ (支援的核心版本)](#supported-debian-kernel-versions-for-azure-virtual-machines)
- SUSE Linux Enterprise Server 12 SP1、SP2、SP3 [ (支援的核心版本)](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines)
- SUSE Linux Enterprise Server 11 SP3
- SUSE Linux Enterprise Server 11 SP4
- Oracle Enterprise Linux 6.4、6.5、6.6、6.7，執行 Red Hat 相容核心或 Unbreakable Enterprise Kernel 第 3 版 (UEK3)

(不支援 SLES 11 SP3 至 SLES 11 SP4 的複寫電腦升級。 若已將複寫電腦從 SLES 11SP3 升級至 SLES 11 SP4，則您必須停用複寫以在升級後重新提供電腦防護。)

>[!NOTE]
>
> 使用密碼型驗證和登入並使用雲端 init 套件來設定雲端虛擬機器的 Ubuntu 伺服器，可能對容錯移轉停用密碼型登入 (取決於 cloudinit 組態)。從 Azure 入口網站上容錯移轉的虛擬機器本身的設定功能表 (在 [支援 + 疑難排解] 區段下) 重設密碼，即可重新啟用密碼型登入。

### <a name="supported-ubuntu-kernel-versions-for-azure-virtual-machines"></a>Azure 虛擬機器支援的 Ubuntu 核心版本

**版本** | **行動服務版本** | **核心版本** |
--- | --- | --- |
14.04 LTS | 9.19 | 3.13.0-24-generic 至 3.13.0-153-generic、<br/>3.16.0-25-generic 至 3.16.0-77-generic、<br/>3.19.0-18-generic 至 3.19.0-80-generic、<br/>4.2.0-18-generic 至 4.2.0-42-generic、<br/>4.4.0-21-generic 至 4.4.0-131-generic |
14.04 LTS | 9.18 | 3.13.0-24-generic 至 3.13.0-151-generic、<br/>3.16.0-25-generic 至 3.16.0-77-generic、<br/>3.19.0-18-generic 至 3.19.0-80-generic、<br/>4.2.0-18-generic 至 4.2.0-42-generic、<br/>4.4.0-21-generic 至 4.4.0-128-generic |
14.04 LTS | 9.17 | 3.13.0-24-generic 至 3.13.0-147-generic、<br/>3.16.0-25-generic 至 3.16.0-77-generic、<br/>3.19.0-18-generic 至 3.19.0-80-generic、<br/>4.2.0-18-generic 至 4.2.0-42-generic、<br/>4.4.0-21-generic 至 4.4.0-124-generic |
14.04 LTS | 9.16 | 3.13.0-24-generic 至 3.13.0-144-generic、<br/>3.16.0-25-generic 至 3.16.0-77-generic、<br/>3.19.0-18-generic 至 3.19.0-80-generic、<br/>4.2.0-18-generic 至 4.2.0-42-generic、<br/>4.4.0-21-generic 至 4.4.0-119-generic |
|||
16.04 LTS | 9.19 | 4.4.0-21-generic 至 4.4.0-131-generic、<br/>4.8.0-34-generic 至 4.8.0-58-generic、<br/>4.10.0-14-generic 至 4.10.0-42-generic、<br/>4.11.0-13-generic 至 4.11.0-14-generic、<br/>4.13.0-16-generic 至 4.13.0-45-generic、<br/>4.15.0-13-generic 至 4.15.0-30-generic<br/>4.11.0-1009-azure 至 4.11.0-1016-azure、<br/>4.13.0-1005-azure 至 4.13.0-1018-azure <br/>4.15.0-1012-azure 至 4.15.0-1019-azure|
16.04 LTS | 9.18 | 4.4.0-21-generic 至 4.4.0-128-generic、<br/>4.8.0-34-generic 至 4.8.0-58-generic、<br/>4.10.0-14-generic 至 4.10.0-42-generic、<br/>4.11.0-13-generic 至 4.11.0-14-generic、<br/>4.13.0-16-generic 至 4.13.0-45-generic、<br/>4.11.0-1009-azure 至 4.11.0-1016-azure、<br/>4.13.0-1005-azure 至 4.13.0-1018-azure |
16.04 LTS | 9.17 | 4.4.0-21-generic 至 4.4.0-124-generic、<br/>4.8.0-34-generic 至 4.8.0-58-generic、<br/>4.10.0-14-generic 至 4.10.0-42-generic、<br/>4.11.0-13-generic 至 4.11.0-14-generic、<br/>4.13.0-16-generic 至 4.13.0-41-generic、<br/>4.11.0-1009-azure 至 4.11.0-1016-azure、<br/>4.13.0-1005-azure 至 4.13.0-1016-azure |
16.04 LTS | 9.16 | 4.4.0-21-generic 至 4.4.0-119-generic、<br/>4.8.0-34-generic 至 4.8.0-58-generic、<br/>4.10.0-14-generic 至 4.10.0-42-generic、<br/>4.11.0-13-generic 至 4.11.0-14-generic、<br/>4.13.0-16-generic 至 4.13.0-38-generic、<br/>4.11.0-1009-azure 至 4.11.0-1016-azure、<br/>4.13.0-1005-azure 至 4.13.0-1012-azure |


### <a name="supported-debian-kernel-versions-for-azure-virtual-machines"></a>Azure 虛擬機器支援的 Debian 核心版本

**版本** | **行動服務版本** | **核心版本** |
--- | --- | --- |
Debian 7 | 9.17、9.18、9.19 | 3.2.0-4-amd64 至 3.2.0-6-amd64、3.16.0-0.bpo.4-amd64 |
Debian 7 | 9.16 | 3.2.0-4-amd64 至 3.2.0-5-amd64、3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | 9.19 | 3.16.0-4-amd64 至 3.16.0-6-amd64、4.9.0-0.bpo.4-amd64 至 4.9.0-0.bpo.7-amd64 |
Debian 8 | 9.17、9.18 | 3.16.0-4-amd64 至 3.16.0-6-amd64、4.9.0-0.bpo.4-amd64 至 4.9.0-0.bpo.6-amd64 |
Debian 8 | 9.16 | 3.16.0-4-amd64 至 3.16.0-5-amd64、4.9.0-0.bpo.4-amd64 至 4.9.0-0.bpo.5-amd64 |

### <a name="supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines"></a>Azure 虛擬機器支援的 SUSE Linux Enterprise Server 12 核心版本

**版本** | **行動服務版本** | **核心版本** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1、SP2、SP3) | 9.19 | SP1 3.12.49-11-default 至 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 至 3.12.74-60.64.93-default</br></br> SP2 4.4.21-69-default 至 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default 至 4.4.121-92.80-default</br></br>SP3 4.4.73-5-default 至 4.4.140-94.42-default |
SUSE Linux Enterprise Server 12 (SP1、SP2、SP3) | 9.18 | SP1 3.12.49-11-default 至 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 至 3.12.74-60.64.93-default</br></br> SP2 4.4.21-69-default 至 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default 至 4.4.121-92.80-default</br></br>SP3 4.4.73-5-default 至 4.4.138-94.39-default |
SUSE Linux Enterprise Server 12 (SP1、SP2、SP3) | 9.17 | SP1 3.12.49-11-default 至 3.12.74-60.64.40-default</br></br> SP1(LTSS) 3.12.74-60.64.45-default 至 3.12.74-60.64.88-default</br></br> SP2 4.4.21-69-default 至 4.4.120-92.70-default</br></br>SP2(LTSS) 4.4.121-92.73-default</br></br>SP3 4.4.73-5-default 至 4.4.126-94.22-default |

## <a name="supported-file-systems-and-guest-storage-configurations-on-azure-virtual-machines-running-linux-os"></a>Azure 虛擬機器 (執行 Linux OS) 上的支援檔案系統與客體儲存體組態

* 檔案系統：ext3、ext4、ReiserFS (僅 Suse Linux Enterprise Server)、XFS
* 磁碟區管理員：LVM2
* 多重路徑軟體：裝置對應程式

## <a name="region-support"></a>區域支援

您可以複製和復原相同的地理叢集內任何兩個區域之間的 VM。

**地理叢集** | **Azure 區域**
-- | --
美洲 | 加拿大東部、加拿大中部、美國中南部、美國中西部、美國東部、美國東部 2、美國西部、美國西部 2、美國中部、美國中北部
歐洲 | 英國西部、英國南部、北歐、西歐、法國中部、法國南部
亞洲 | 印度南部、印度中部、東南亞、東亞、日本東部、日本西部、南韓中部、南韓南部
澳大利亞   | 澳大利亞東部、澳大利亞東南部、澳大利亞中部、澳大利亞中部 2
Azure Government    | US Gov 維吉尼亞州、US Gov 愛荷華州、US Gov 亞利桑那州、US Gov 德克薩斯州、US DoD 東部、US DoD 中部
德國 | 德國中部、德國東北部
中國 | 中國東部、中國北部

>[!NOTE]
>
> 對於巴西南部區域，您只能複寫及容錯移轉至美國中南部、美國中西部、美國東部、美國東部 2、美國西部、美國西部 2 和美國中北部區域的其中一個區域，並進行容錯回復。

## <a name="support-for-vmdisk-management"></a>VM/磁碟管理的支援

**Action** | **詳細資料**
-- | ---
在複寫的 VM 上調整磁碟大小 | 支援
在複寫的 VM 上新增磁碟 | 不支援。 您需要停用 VM 的複寫、新增磁碟，然後重新啟用複寫。


## <a name="support-for-compute-configuration"></a>計算設定的支援

**組態** | **支援/不支援** | **備註**
--- | --- | ---
大小 | 至少 2 顆 CPU 核心和 1 GB RAM 的任何 Azure VM 大小 | 請參閱 [Azure 虛擬機器大小](../virtual-machines/windows/sizes.md)
可用性設定組 | 支援 | 您在入口網站中進行「啟用複寫」步驟期間，如果使用預設選項，將根據來源地區設定自動建立可用性設定組。 您可以隨時變更 [複寫項目] > [設定] > [計算與網路] > [可用性設定組] 中的目標可用性設定組。
可用性區域 | 不支援 | 目前不支援可用性區域中部署的 VM。
Hybrid Use Benefit | 支援 | 如果來源 VM 已啟用 HUB 授權，測試容錯移轉或容錯移轉 VM 也會使用 HUB 授權。
虛擬機器擴展集 | 不支援 |
Azure 資源庫映像 - Microsoft 發行 | 支援 | 只要 VM 在 Site Recovery 支援的作業系統上執行即支援
Azure 資源庫映像 - 第三方發行 | 支援 | 只要 VM 在 Site Recovery 支援的作業系統上執行即支援。
自訂映像 - 第三方發行 | 支援 | 只要 VM 在 Site Recovery 支援的作業系統上執行即支援。
使用 Site Recovery 移轉 VM | 支援 | 如果使用 Site Recovery 將 VMware/實體機器移轉到 Azure，您需要解除安裝舊版的行動服務，然後重新啟動電腦，再複寫到另一個 Azure 區域。

## <a name="support-for-storage-configuration"></a>儲存體設定的支援

**組態** | **支援/不支援** | **備註**
--- | --- | ---
最大的作業系統磁碟大小 | 2048 GB | 請參閱 [VM 使用的磁碟](../virtual-machines/windows/about-disks-and-vhds.md#disks-used-by-vms)。
最大的資料磁碟大小 | 4095 GB | 請參閱 [VM 使用的磁碟](../virtual-machines/windows/about-disks-and-vhds.md#disks-used-by-vms)。
資料磁碟數量 | 依指定的 Azure VM 大小而定，最多支援 64 個 | 請參閱 [Azure 虛擬機器大小](../virtual-machines/windows/sizes.md)
暫存磁碟 | 一律排除在複寫之外 | 暫存磁碟排除在複寫之外。 根據 Azure 指導方針，您不應該在暫存磁碟上放置任何持續性資料。 如需詳細資訊，請參閱 [Azure VM 上的暫存磁碟](../virtual-machines/windows/about-disks-and-vhds.md#temporary-disk)。
磁碟上的資料變更率 | 進階儲存體的上限為每個磁碟 10 MBps，標準儲存體的上限為每個磁碟 2 MBps | 如果磁碟的平均資料變更率持續超過 10 MBps (適用於進階儲存體) 和 2 MBps (適用於標準儲存體)，複寫將會趕不上進度。 不過，如果是偶發性的資料高載，資料變更率超過 10 MBps (適用於進階儲存體) 和 2 MBps (適用於標準儲存體) 的情況只持續一段時間便下降，複寫便可趕上進度。 在此情況下，您可能會發現復原點稍微延遲。
標準儲存體帳戶上的磁碟 | 支援 |
進階儲存體帳戶上的磁碟 | 支援 | 如果 VM 的磁碟分散於進階和標準儲存體帳戶，您可以對於各個磁碟選取不同的目標儲存體帳戶，以確保目標區域有相同的儲存體設定
標準受控磁碟 | 在支援 Azure Site Recovery 的 Azure 區域中會支援。 |  
進階受控磁碟 | 在支援 Azure Site Recovery 的 Azure 區域中會支援。 |
儲存體空間 | 支援 |         
待用加密 (SSE) | 支援 | SSE 是儲存體帳戶上的預設設定。   
適用於 Windows OS 的 Azure 磁碟加密 (ADE) | 支援啟用[使用 Azure AD 應用程式加密](https://aka.ms/ade-aad-app)的 VM |
適用於 Linux OS 的 Azure 磁碟加密 (ADE) | 不支援 |
熱新增/移除磁碟 | 不支援 | 如果您在 VM 上新增或移除資料磁碟，需要停用複寫，然後再次為 VM 啟用複寫。
排除磁碟 | 不支援|   預設排除暫存磁碟。
儲存空間直接存取  | 不支援|
向外延展檔案伺服器  | 不支援|
LRS | 支援 |
GRS | 支援 |
RA-GRS | 支援 |
ZRS | 不支援 |  
非經常性和經常性儲存體 | 不支援 | 非經常性和經常性儲存體不支援虛擬機器磁碟
適用於虛擬網路的 Azure 儲存體防火牆  | 否 | 不支援在用來儲存複寫資料之快取儲存體帳戶上存取特定的 Azure 虛擬網路。
一般用途 V2 儲存體帳戶 (經常性存取層和非經常性存取層) | 否 | 與一般用途 V1 儲存體帳戶相比，交易成本大幅增加

>[!IMPORTANT]
> 請務必遵守 [Linux](../virtual-machines/linux/disk-scalability-targets.md) 或 [Windows](../virtual-machines/windows/disk-scalability-targets.md) 虛擬機器的 VM 磁碟延展性和效能目標，以避免任何效能問題。 如果您遵循預設設定，Site Recovery 會根據來源設定建立所需的磁碟和儲存體帳戶。 如果您自訂並選取您自己的設定，請務必遵循您的來源 VM 磁碟延展性和效能目標。

## <a name="support-for-network-configuration"></a>網路組態的支援
**組態** | **支援/不支援** | **備註**
--- | --- | ---
網路介面 (NIC) | 指定 Azure VM 大小所支援的 NIC 數目上限 | 在測試容錯移轉或容錯移轉作業中建立 VM 時，會建立 NIC。 VM 容錯移轉的 NIC 數目取決於啟用複寫時來源 VM 具有的 NIC 數量。 啟用複寫後，如果您新增/移除 NIC，不會影響容錯移轉 VM 上的 NIC 計數。
網際網路負載平衡器 | 支援 | 您需要使用復原方案中的 azure 自動化指令碼，使預先設定的負載平衡器產生關聯。
內部負載平衡器 | 支援 | 您需要使用復原方案中的 azure 自動化指令碼，使預先設定的負載平衡器產生關聯。
公用 IP| 支援 | 您需要使現有公用 IP 與 NIC 產生關聯，或建立一個 IP 並使用復原方案中的 azure 自動化指令碼與 NIC 產生關聯。
NIC 上的 NSG (資源管理員)| 支援 | 您需要使用復原方案中的 azure 自動化指令碼，使 NSG 與 NIC 產生關聯。  
子網路上的 NSG (資源管理員與傳統)| 支援 | 您需要使用復原方案中的 azure 自動化指令碼，使 NSG 與子網路產生關聯。
VM 上的 NSG (傳統)| 支援 | 您需要使用復原方案中的 azure 自動化指令碼，使 NSG 與 NIC 產生關聯。
保留的 IP (靜態 IP) / 保留來源 IP | 支援 | 如果來源 VM 上的 NIC 有靜態 IP 組態，而目標子網路有相同的 IP，則會將它指派給容錯移轉 VM。 如果目標子網路沒有相同的 IP，子網路中一個可用的 IP 會被保留供 VM 使用。 您可以在 [複寫項目] > [設定] > [計算與網路] > [網路介面] 中指定選擇的固定 IP。 您可以選取 NIC，並指定您選擇的子網路和 IP。
動態 IP| 支援 | 如果來源 VM 上的 NIC 有動態 IP 組態，則容錯移轉 VM 上的 NIC 預設也是動態。 您可以在 [複寫項目] > [設定] > [計算與網路] > [網路介面] 中指定選擇的固定 IP。 您可以選取 NIC，並指定您選擇的子網路和 IP。
流量管理員整合 | 支援 | 您可以預先設定流量管理員，定期將流量傳輸到來源區域中的端點，如果發生容錯移轉，則傳輸到目標區域中的端點。
Azure 受控 DNS | 支援 |
自訂 DNS  | 支援 |    
未經驗證的 Proxy | 支援 | 請參閱[網路指導方針文件](site-recovery-azure-to-azure-networking-guidance.md)。    
經驗證的 Proxy | 不支援 | 如果 VM 對於輸出連線能力使用經驗證的 Proxy，則無法使用 Azure Site Recovery 複寫它。    
內部部署的網站對網站 VPN 與 (不論是否有 ExpressRoute)| 支援 | 請確定設定 UDR 和 NSG 時，站台復原流量不是傳送到內部部署的裝置。 請參閱[網路指導方針文件](site-recovery-azure-to-azure-networking-guidance.md)。  
VNET 對 VNET 連線 | 支援 | 請參閱[網路指導方針文件](site-recovery-azure-to-azure-networking-guidance.md)。  
虛擬網路服務端點 | 支援 | 不支援適用於虛擬網路的 Azure 儲存體防火牆。 不支援在用來儲存複寫資料之快取儲存體帳戶上存取特定的 Azure 虛擬網路。
加速網路 | 支援 | 必須在來源 VM 上啟用加速網路。 [深入了解](azure-vm-disaster-recovery-with-accelerated-networking.md)。


## <a name="next-steps"></a>後續步驟
- 深入了解[複寫 Azure VM 的網路指導方針](site-recovery-azure-to-azure-networking-guidance.md)
- [複寫 Azure VM](site-recovery-azure-to-azure.md) 開始保護您的工作負載
