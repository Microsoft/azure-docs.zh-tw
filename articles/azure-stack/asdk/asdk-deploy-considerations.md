---
title: Azure Stack 開發套件部署 (ASDK) 先決條件 | Microsoft Docs
description: 檢閱 Azure Stack 開發套件 (ASDK) 的環境和硬體需求。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 41d27b9a173cc019549f1dea0c66ffcaa51fbb10
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719438"
---
# <a name="azure-stack-deployment-planning-considerations"></a>Azure Stack 部署規劃考量
在部署 Azure Stack 開發套件 (ASDK) 前，請確定您的開發套件主機電腦符合本文中所述的需求。


## <a name="hardware"></a>硬體
| 元件 | 最小值 | 建議 |
| --- | --- | --- |
| 磁碟機：作業系統 |1 個 OS 磁碟 (SSD 或 HDD)，最少有 200 GB 供系統磁碟分割使用 |1 個 OS 磁碟 (SSD 或 HDD)，最少有 200 GB 供系統磁碟分割使用 |
| 磁碟機：一般開發套件資料<sup>*</sup>  |4 個磁碟。 每個磁碟 (SSD 或 HDD) 提供最少 140 GB 的容量。 會使用所有可用的磁碟。 |4 個磁碟。 每個磁碟 (SSD 或 HDD) 至少提供 250 GB 的容量。 會使用所有可用的磁碟。 |
| 計算：CPU |雙插槽：12 個實體核心 (總計) |雙插槽：16 個實體核心 (總計) |
| 計算：記憶體 |96 GB RAM |128 GB RAM (這是支援 PaaS 資源提供者的下限)。|
| 計算：BIOS |啟用 Hyper-V (支援 SLAT) |啟用 Hyper-V (支援 SLAT) |
| 網路：NIC |NIC 所需的 Windows Server 2012 R2 憑證；不需特殊的功能 |NIC 所需的 Windows Server 2012 R2 憑證；不需特殊的功能 |
| HW 標誌憑證 |[Windows Server 2012 R2 認證](http://windowsservercatalog.com/results.aspx?&chtext=&cstext=&csttext=&chbtext=&bCatID=1333&cpID=0&avc=79&ava=0&avq=0&OR=1&PGS=25&ready=0) |[Windows Server 2016 認證](http://windowsservercatalog.com/results.aspx?&chtext=&cstext=&csttext=&chbtext=&bCatID=1333&cpID=0&avc=79&ava=0&avq=0&OR=1&PGS=25&ready=0) |

<sup>*</sup>如果您打算從 Azure 新增許多[市集項目](asdk-marketplace-item.md)，則您所需的容量會比這個建議的容量多。

**資料磁碟機組態：** 所有資料磁碟機都必須具有相同的類型 (全部都是 SAS、SATA 或 NVMe) 和容量。 如果使用 SAS 磁碟機，則該磁碟機必須透過單一路徑連結 (不支援 MPIO 多重路徑)。

**HBA 組態選項**

* (慣用) 簡單 HBA
* RAID HBA – 介面卡必須設定為「穿通」模式
* RAID HBA – 磁碟應設定為單一磁碟、RAID 0

**支援的匯流排和媒體類型組合**

* SATA HDD
* SAS HDD
* RAID HDD
* RAID SSD (如果媒體類型為未指定/不明<sup>*</sup>)
* SATA SSD + SATA HDD
* SAS SSD + SAS HDD
* NVMe

<sup>*</sup> 不具備穿通功能的 RAID 控制器無法辨識媒體類型。 這類控制器會將 HDD 和 SSD 皆標示為未指定。 在此情況下，SSD 會當作永續性儲存體使用，而非快取裝置。 因此，您可以在那些 SSD 上部署開發套件。

**範例 HBA**：傳遞模式中的 LSI 9207-8i、LSI-9300-8i 或 LSI-9265-8i

範例 OEM 設定可供使用。

## <a name="operating-system"></a>作業系統
|  | **需求** |
| --- | --- |
| **作業系統版本** |Windows Server 2016 或更新版本。 作業系統版本在部署開始前並不重要，因為您將會讓主機電腦開機進入 Azure Stack 安裝所包含的 VHD。 作業系統和所有必要的修補程式都已經整合到該映像中。 請勿使用任何金鑰來啟用開發套件中使用的任何 Windows Server 執行個體。 |

> [!TIP]
> 安裝作業系統之後，您可以使用[適用於 Azure Stack 的部署檢查程式](https://gallery.technet.microsoft.com/Deployment-Checker-for-50e0f51b)來確認您的硬體符合所有需求。

## <a name="account-requirements"></a>帳戶需求
通常，您會在有網際網路連線能力的情況下部署開發套件，以便連線到 Microsoft Azure。 在此情況下，您必須設定 Azure Active Directory (Azure AD) 帳戶以部署開發套件。

如果您的環境未連線到網際網路，或您不想要使用 Azure AD，則可以使用「Active Directory 同盟服務」(AD FS) 來部署 Azure Stack。 開發套件包含自己的 AD FS 和 Active Directory Domain Services 執行個體。 如果使用這個選項來部署，就不需要事先設定帳戶。

>[!NOTE]
如果您使用 AD FS 選項來部署，則必須重新部署 Azure Stack 以切換到 Azure AD。

### <a name="azure-active-directory-accounts"></a>Azure Active Directory 帳戶
若要使用 Azure AD 帳戶來部署 Azure Stack，您必須在執行部署 PowerShell 指令碼之前，先備妥 Azure AD 帳戶。 這個帳戶會成為 Azure AD 租用戶的「全域系統管理員」。 它可用來為與 Azure Active Directory 和「圖形 API」互動的所有 Azure Stack 服務佈建及委派應用程式和服務主體。 它也用來作為預設提供者訂用帳戶 (您可以稍後變更此訂用帳戶) 的擁有者。 您可以使用這個帳戶來登入 Azure Stack 系統的系統管理員入口網站。

1. 建立至少是一個 Azure AD 之目錄系統管理員的 Azure AD 帳戶。 如果您已經有一個這樣的帳戶，則可以使用該帳戶。 否則，您可以在 [https://azure.microsoft.com/free/](http://azure.microsoft.com/pricing/free/) 免費建立一個帳戶 (若在中國，請改為造訪 <http://go.microsoft.com/fwlink/?LinkID=717821>)。 如果您打算稍後[向 Azure 註冊 Azure Stack](asdk-register.md)，則也必須在這個新建立的帳戶中有一個訂用帳戶。
   
    儲存這些認證以供作為服務管理員。 這個帳戶可以設定和管理資源雲端、使用者帳戶、租用戶方案、配額及價格。 在此入口網站中，這些管理員可以建立網站雲端、虛擬機器私人雲端、建立方案，以及管理使用者訂用帳戶。
1. 在 Azure AD 中建立至少一個測試使用者帳戶，以便您能夠以租用戶身分登入開發套件。
   
   | **Azure Active Directory 帳戶** | **是否支援？** |
   | --- | --- |
   | 具備有效公用 Azure 訂用帳戶的公司或學校帳戶 |是 |
   | 具備有效的公用 Azure 訂用帳戶之 Microsoft 帳戶 |是 |
   | 具備有效中國 Azure 訂用帳戶的公司或學校帳戶 |是 |
   | 具備有效美國政府 Azure 訂用帳戶的公司或學校帳戶 |是 |

部署之後，就不需要 Azure Active Directory 全域管理員權限。 不過，某些作業可能需要全域管理員認證。 例如，需要獲派權限的資源提供者安裝程式指令碼或新功能。 您可以暫時恢復帳戶的全域管理員權限，或使用擁有「預設提供者訂用帳戶」的個別全域管理員帳戶。

## <a name="network"></a>網路
### <a name="switch"></a>Switch
一個在開發套件機器交換器上的可用連接埠。  

開發套件機器支援連線到交換器存取連接埠或主幹連接埠。 此交換器不需要有任何特殊的功能。 如果您使用主幹連接埠，或如果您需要設定 VLAN ID，就必須提供 VLAN ID 作為部署參數。

### <a name="subnet"></a>子網路
請勿將開發套件機器連線到下列子網路：

* 192.168.200.0/24
* 192.168.100.0/27
* 192.168.101.0/26
* 192.168.102.0/24
* 192.168.103.0/25
* 192.168.104.0/25

這些子網路是保留給在開發套件環境中的內部網路使用的。

### <a name="ipv4ipv6"></a>IPv4/IPv6
僅支援 IPv4。 您無法建立 IPv6 網路。

### <a name="dhcp"></a>DHCP
請確定該網路上有可用的 DHCP 伺服器，供 NIC 與其連線。 如果無法使用 DHCP，則除了主機所使用的網路之外，您還必須準備其他靜態 IPv4 網路。 您必須提供該 IP 位址和閘道作為部署參數。

### <a name="internet-access"></a>網際網路存取
Azure Stack 需要能夠直接或透過 Transparent Proxy 存取網際網路。 Azure Stack 不支援設定 Web Proxy 來啟用網際網路存取。 指派給 MAS-BGPNAT01 (透過 DHCP 或靜態 IP) 的主機 IP 和新 IP 都必須能夠存取網際網路。 連接埠 80 和 443 會用在 graph.windows.net 和 login.microsoftonline.com 網域下。


## <a name="next-steps"></a>後續步驟
[下載 ASDK 部署套件](asdk-download.md)
