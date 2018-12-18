---
title: IaaS VM 適用的 Azure 磁碟加密概觀 | Microsoft Docs
description: 本文提供 IaaS VM 適用的 Microsoft Azure 磁碟加密概觀。
author: mestew
ms.service: security
ms.subservice: Azure Disk Encryption
ms.topic: article
ms.author: mstewart
ms.date: 09/14/2018
ms.openlocfilehash: 193aa8f87a90eb7bbf1e2c49132ad480881d41fe
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45633464"
---
# <a name="azure-disk-encryption-for-iaas-vms"></a>IaaS VM 適用的 Azure 磁碟加密 
Microsoft Azure 致力於確保您的資料隱私權、資料主權，並透過一系列進階技術來加密、控制和管理加密金鑰、控制和稽核資料存取，讓您控制您的 Azure 託管資料。 此控制提供 Azure 客戶靈活度，可選擇最符合其商務需求的解決方案。 本文為您介紹技術解決方案「Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密」，以協助保護及保障您的資料，以便符合組織的安全性和符合性的承諾。 

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]


## <a name="overview"></a>概觀
Azure 磁碟加密 (ADE) 是協助您加密 Windows 和 Linux IaaS 虛擬機器磁碟的功能。 ADE 運用 Windows 的業界標準 [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview) 功能和 Linux 的 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) 功能，為 OS 和資料磁碟提供磁碟區加密。 此解決方案與 [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) 整合，協助您控制及管理磁碟加密金鑰和祕密。 此解決方案也可確保虛擬機器磁碟上的所有待用資料都會在您的 Azure 儲存體中加密。

Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密已在所有 Azure 公用區域和 AzureGov 區域**正式推出**，可供用於標準 VM 和具有高階儲存體的 VM。 當您套用 Azure 磁碟加密管理解決方案時，可滿足下列商務需求：

* IaaS VM 會使用業界標準的加密技術輕鬆加以保護，解決組織的安全性與法務遵循的需求。
* IaaS VM 會在客戶控制的金鑰和原則下開機，且您可以在金鑰保存庫中稽核其使用狀況。


如果您使用 Azure 資訊安全中心，其會在您有未加密的虛擬機器時對您發出警示。 這些警示會顯示為高嚴重性，因此建議您加密這些虛擬機器。
![Azure 資訊安全中心磁碟加密警示](media/azure-security-disk-encryption/security-center-disk-encryption-fig1.png)

> [!NOTE]
> 某些建議可能會增加資料、網路或計算資源的使用量，導致額外的授權或訂用帳戶成本。


## <a name="encryption-scenarios"></a>加密案例
Azure 磁碟加密解決方案支援下列客戶案例：

* 對透過預先加密的 VHD 和加密金鑰建立的新 Windows IaaS VM 啟用加密 
* 對透過受支援 Azure 資源庫映像建立的新 IaaS VM 啟用加密
* 在 Azure 中執行的現有 IaaS VM 上啟用加密
* 在 Windows 虛擬機器擴展集上啟用加密
* 在 Linux 虛擬機器擴展集的資料磁碟機上啟用加密
* 在 Windows IaaS VM 上停用加密
* 在 Linux IaaS VM 的資料磁碟機上停用加密
* 在 Windows 虛擬機器擴展集上停用加密
* 在 Linux 虛擬機器擴展集的資料磁碟機上停用加密
* 允許加密受控磁碟 VM
* 更新現有已加密進階與非進階儲存體 VM 的加密設定
* 備份與還原加密的 VM

在 Microsoft Azure 中啟用時，解決方案會對 IaaS VM 支援下列案例：

* 與 Azure 金鑰保存庫整合
* 標準層 VM：[A、D、DS、G、GS、F 等系列 IaaS VM](https://azure.microsoft.com/pricing/details/virtual-machines/)
    * 這幾層內的 [Linux VM](azure-security-disk-encryption-faq.md#bkmk_LinuxOSSupport) 必須符合 7GB 的最小記憶體需求
* 在來自受支援 Azure 資源庫映像的 Windows 和 Linux IaaS VM、受控磁碟以及擴展集 VM 上啟用加密
* 在 Windows IaaS VM、擴展集 VM 與受控磁碟 VM 的作業系統與資料磁碟機上停用加密
* 在 Linux IaaS VM、擴展集 VM 與受控磁碟 VM 的資料磁碟機上停用加密
* 在執行 Windows Client OS 的 IaaS VM 上啟用加密
* 在具有掛接路徑的磁碟區上啟用加密
* 使用 mdadm 在設定了磁碟串接 (RAID) 的 Linux VM 上啟用加密
* 使用資料磁碟適用的 LVM 在 Linux VM 上啟用加密
* 在 Linux VM 作業系統和資料磁碟上啟用加密 
* 在設定了儲存空間的 Windows VM 上啟用加密
* 更新現有已加密進階與非進階儲存體 VM 的加密設定
* 針對非 KEK 與 KEK 案例 (KEK，金鑰加密金鑰) 備份與還原已加密的 VM
* 支援所有 Azure 公用區域與 AzureGov 區域

此解決方案不支援下列案例、功能和技術：

* 基本層 IaaS VM
* 在 Linux IaaS VM 的 OS 磁碟機上停用加密
* 如果已加密 Linux IaaS VM 的 OS 磁碟機，請將資料磁碟機上的加密停用
* 使用傳統 VM 建立方法所建立的 IaaS VM
* 在 Linux IaaS VM 客戶自訂映像上啟用加密
* 與您的內部部署金鑰管理服務整合
* Azure 檔案 (共用檔案系統)
* 網路檔案系統 (NFS)
* 動態磁碟區
* 使用以軟體為基礎之 RAID 系統設定的 Windows VM

## <a name="encryption-features"></a>加密功能
為 Azure IaaS VM 啟用並部署 Azure 磁碟加密時，會啟用下列功能，視提供的組態而定：

* 加密 OS 磁碟區以保護儲存體中的待用開機磁碟區
* 加密資料磁碟區以保護儲存體中的待用資料磁碟區
* 在 Windows IaaS VM 的 OS 和資料磁碟機上停用加密
* 在 Linux IaaS VM 的資料磁碟機上停用加密 (僅在 OS 磁碟機「並未」加密時)
* 保護金鑰保存庫訂用帳戶中的加密金鑰和密碼
* 報告已加密 IaaS VM 的加密狀態
* 從 IaaS 虛擬機器移除磁碟加密組態設定
* 使用 Azure 備份服務來備份和還原已加密的 VM

Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密解決方案包含：

* Windows 適用的磁碟加密擴充功能。
* Linux 適用的磁碟加密擴充功能。
* 磁碟加密 PowerShell Cmdlet。
* 磁碟加密 Azure 命令列介面 (CLI) Cmdlet。
* 磁碟加密 Azure Resource Manager 範本。

執行 Windows 或 Linux OS 的 IaaS VM 支援 Azure 磁碟加密解決方案。 如需支援的作業系統相關詳細資訊，請參閱[必要條件](azure-security-disk-encryption-prerequisites.md)一文。

> [!NOTE]
> 使用 Azure 磁碟加密來加密 VM 磁碟完全免費。 標準[金鑰保存庫定價](https://azure.microsoft.com/pricing/details/key-vault/)適用於用來儲存加密金鑰的金鑰保存庫。 


## <a name="encryption-workflow"></a>加密工作流程

 若要啟用 Windows 和 Linux VM 的磁碟加密，請執行下列步驟︰

1. 從先前的加密案例中選擇一個加密案例。
2. 選擇透過 Azure 磁碟加密 Resource Manager 範本、PowerShell Cmdlet 或 CLI 命令啟用磁碟加密，並指定加密組態。

   * 針對客戶加密的 VHD 案例，請將加密的 VHD 上傳至儲存體帳戶並將加密金鑰資料上傳至金鑰保存庫。 接著，提供加密組態資訊以在新的 IaaS VM 上啟用加密。
   * 針對透過 Marketplace 所建立的新 VM 和已在 Azure 中執行的現有 VM，提供加密組態以在 IaaS VM 上啟用加密。

3. 授與存取至 Azure 平台，以從您的金鑰保存庫讀取加密金鑰資料 (Windows 系統的 BitLocker 加密金鑰和 Linux 的複雜密碼)，藉以在 IaaS VM 上啟用加密。

4. Azure 會使用加密、金鑰保存庫組態更新 VM 服務模型，並設定加密的 VM。

 ![Azure 中的 Microsoft Antimalware](./media/azure-security-disk-encryption/disk-encryption-fig1.png)

## <a name="decryption-workflow"></a>解密工作流程
若要停用 IaaS VM 的磁碟加密，請完成下列高階步驟：

1. 選擇在 Azure 中的執行中 IaaS VM 上停用加密 (解密)，並指定解密組態。 您可以透過 Azure 磁碟加密、Resource Manager 範本、PowerShell cmdlet 或 Azure CLI 停用。

 此步驟會在執行中 Windows IaaS VM 上停用 OS 或資料磁碟區或兩者的加密。 但是如前一節所述，並不支援停用 Linux 的 OS 磁碟加密。 只要 OS 磁碟機未加密，就只允許對 Linux VM 上的資料磁碟機執行解密步驟。
2. Azure 會更新 VM 服務模型，且 IaaS VM 會標示為已解密。 VM 的待用內容不會再加密。

> [!NOTE]
> 停用加密作業並不會刪除您的金鑰保存庫和加密金鑰資料 (Windows 系統的 BitLocker 加密金鑰或 Linux 的複雜密碼)。
 > 不支援停用 Linux 適用的作業系統磁碟加密。 只允許對 Linux VM 上的資料磁碟機執行解密步驟。
如果 OS 磁碟機已加密，就不支援將 Linux 的資料磁碟加密停用。


## <a name="encryption-workflow-previous-release"></a>加密工作流程 (舊版)

新版的 Azure 磁碟加密不需要提供 Azure AD 應用程式參數，即可啟用 VM 磁碟加密。 若使用新版本，您就不再需要在啟用加密步驟期間提供 Azure AD 認證。 若使用新版本，所有新的 VM 必須經過加密，而不需要 Azure AD 應用程式參數。 已經使用 Azure AD 應用程式參數進行加密的 VM 仍然受支援，應該繼續使用 AAD 語法進行維護。 若要啟用 Windows 和 Linux VM (舊版) 的磁碟加密，請執行下列步驟︰

1. 從先前的加密案例中選擇一個加密案例。
2. 選擇透過 Azure 磁碟加密 Resource Manager 範本、PowerShell Cmdlet 或 CLI 命令啟用磁碟加密，並指定加密組態。

   * 針對客戶加密的 VHD 案例，請將加密的 VHD 上傳至儲存體帳戶並將加密金鑰資料上傳至金鑰保存庫。 接著，提供加密組態資訊以在新的 IaaS VM 上啟用加密。
   * 針對透過 Marketplace 所建立的新 VM 和已在 Azure 中執行的現有 VM，提供加密組態以在 IaaS VM 上啟用加密。

3. 授與存取至 Azure 平台，以從您的金鑰保存庫讀取加密金鑰資料 (Windows 系統的 BitLocker 加密金鑰和 Linux 的複雜密碼)，藉以在 IaaS VM 上啟用加密。

4. 提供 Azure Active Directory (Azure AD) 應用程式身分識別，以將加密金鑰資料寫入至金鑰保存庫。 如此可針對步驟 2 中所述的案例在 IaaS VM 上啟用加密。

5. Azure 會使用加密和金鑰保存庫組態更新 VM 服務模型，並建立加密的 VM。


## <a name="terminology"></a>術語
若要了解此技術所使用的一些常用術語，請使用以下術語表：

| 術語 | 定義 |
| --- | --- |
| Azure AD | Azure AD 是 [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)。 Azure AD 帳戶用於從金鑰保存庫驗證、儲存和擷取祕密。 |
| Azure 金鑰保存庫 | Key Vault 是密碼編譯金鑰管理服務，它是基於美國聯邦資訊處理標準 (FIPS) 驗證的硬體安全性模組，可協助您保護密碼編譯金鑰和敏感性祕密。 如需詳細資訊，請參閱 [Key Vault](https://azure.microsoft.com/services/key-vault/) 文件。 |
| BitLocker |[BitLocker](https://technet.microsoft.com/library/hh831713.aspx) 是一種業界認可的 Windows 磁碟區加密技術，用來在 Windows IaaS VM 上啟用磁碟加密。 |
| BEK | BitLocker 加密金鑰可用來加密作業系統開機磁碟區和資料磁碟區。 BitLocker 金鑰會在金鑰保存庫中以密碼形式保護。 |
| CLI | 請參閱 [Azure 命令列介面](/cli/azure/install-azure-cli)。|
| DM-Crypt |[DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) 是基於 Linux 的透明磁碟加密子系統，用來在 Linux IaaS VM 上啟用磁碟加密。 |
| KEK | 金鑰加密金鑰是非對稱金鑰 (RSA 2048)，可用來保護或包裝密碼。 您可以提供硬體安全性模組 (HSM) 保護的金鑰或軟體保護的金鑰。 如需詳細資訊，請參閱 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 文件。 |
| PS Cmdlet | 請參閱 [Azure PowerShell Cmdlet](/powershell/azure/overview)。 |

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [Azure 磁碟加密的先決條件](azure-security-disk-encryption-prerequisites.md)
