---
title: Azure 中 IaaS 工作負載的安全性最佳作法 | Microsoft Docs
description: " 將工作負載移轉至 Azure IaaS 可帶來重新評估設計的機會 "
services: security
documentationcenter: na
author: barclayn
manager: MBaldwin
editor: TomSh
ms.assetid: 02c5b7d2-a77f-4e7f-9a1e-40247c57e7e2
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/18/2018
ms.author: barclayn
ms.openlocfilehash: 057c98d4bac87b4e43e5beb8268d3d3bdbe3ec85
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364251"
---
# <a name="security-best-practices-for-iaas-workloads-in-azure"></a>Azure 中 IaaS 工作負載的安全性最佳作法

在大部分的基礎結構即服務 (IaaS) 案例中，[Azure 虛擬機器 (VM)](https://docs.microsoft.com/azure/virtual-machines/) 對於使用雲端運算的組織來說是主要工作負載。 在組織想要慢慢地將工作負載移轉至雲端的[混合式案例](https://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx)中，這個情況相當明顯。 在這種情況下，請遵循 [IaaS 的一般安全性考量](https://social.technet.microsoft.com/wiki/contents/articles/3808.security-considerations-for-infrastructure-as-a-service-iaas.aspx)，並將安全性最佳作法套用到您所有的 VM。

您的安全性責任是根據雲端服務的類型。 下表摘要說明 Microsoft 與您所擔負之責任的平衡：

![責任範圍](./media/azure-security-iaas/sec-cloudstack-new.png)

安全性需求會取決於多項因素而有所不同，包括不同的工作負載類型。 任何其中一個最佳作法均無法獨自保護您的系統。 如同任何其他安全性項目，您必須選擇適當的選項，以及了解解決方案如何藉由滿足不足之處來彼此互補。

本文說明適用於 VM 和作業系統的最佳做法。

最佳作法是根據共識的意見，並使用目前的 Azure 平台功能及功能集。 由於意見和技術會隨著時間改變，因此我們會更新本文以反映這些變化。

## <a name="protect-vms-by-using-authentication-and-access-control"></a>使用驗證和存取控制來保護 VM
保護 VM 的第一個步驟是確保只有已獲授權的使用者能設定新的 VM 和存取 VM。

**最佳做法**：控制 VM 存取。   
**詳細資訊**：使用 [Azure 原則](../azure-policy/azure-policy-introduction.md)為組織資源制定慣例及建立自訂原則。 將這些原則套用到資源，例如[資源群組](../azure-resource-manager/resource-group-overview.md)。 屬於某資源群組的 VM 會繼承其原則。

如果貴組織有多個訂用帳戶，您可能需要一個方法來有效率地管理這些訂用帳戶的存取、原則和相容性。 [Azure 管理群組](../azure-resource-manager/management-groups-overview.md)可以提供訂用帳戶之上的範圍層級。 您可將訂用帳戶整理到管理群組 (容器) 中，並將治理條件套用至這些群組。 管理群組內的所有訂用帳戶都會自動繼承套用到該群組的條件。 無論具有何種類型的訂用帳戶，管理群組都可為您提供企業級的大規模管理功能。

**最佳做法**：降低 VM 設定和部署的變化性。   
**詳細資訊**：使用 [Azure Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md) 範本強化部署選項，更輕鬆了解及清查環境中的 VM。

**最佳做法**：特殊權限的安全存取。   
**詳細資訊**：使用[最低權限的方法](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models)和內建的 Azure 角色來讓使用者存取和設定 VM：

- [虛擬機器參與者](../role-based-access-control/built-in-roles.md#virtual-machine-contributor)：可以管理 VM，但無法管理他們連接的虛擬網路或儲存體帳戶。
- [傳統虛擬機器參與者](../role-based-access-control/built-in-roles.md#classic-virtual-machine-contributor)：可以管理使用傳統的部署模型建立的 VM，但無法管理 VM 連接的虛擬網路或儲存體帳戶。
- [安全性管理員](../role-based-access-control/built-in-roles.md#security-manager)：可以管理安全性元件、安全性原則及 VM。
- [DevTest Labs 使用者](../role-based-access-control/built-in-roles.md#devtest-labs-user)：可以檢視所有項目，並連接、啟動、重新啟動和關閉 VM。

訂用帳戶管理員和共同管理員可變更此設定，使其成為訂用帳戶中所有 VM 的系統管理員。 請確認您信任所有訂用帳戶管理員和共同管理員登入您的任何機器。

> [!NOTE]
> 我們建議將具有相同生命週期的 VM 合併到相同的資源群組中。 藉由使用資源群組，您可以部署、監視和彙總資源的計費成本。
>
>

控制 VM 存取及設定的組織可改善其整體 VM 安全性。

## <a name="use-multiple-vms-for-better-availability"></a>使用多部 VM 以提高可用性
如果您的 VM 會執行需要具備高可用性的重要應用程式，則強烈建議您使用多個 VM。 若要提高可用性，請使用[可用性設定組](../virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)。

可用性設定組是一種可在 Azure 中使用的邏輯群組，用以確保其中所放置的 VM 資源在部署到 Azure 資料中心時會彼此隔離。 Azure 可確保您在可用性設定組中所放置的 VM，會橫跨多部實體伺服器、計算機架、儲存體單位和網路交換器來執行。 如果硬體或 Azure 軟體發生故障，只有 VM 的子集會受到影響，整體的應用程式則可供客戶繼續使用。 如果您想要建置可靠的雲端解決方案，可用性設定組是不可或缺的重要功能。

## <a name="protect-against-malware"></a>抵禦惡意程式碼
您應安裝反惡意程式碼軟體，以協助識別及移除病毒、間諜軟體和其他惡意軟體。 您可安裝 [Microsoft Antimalware](azure-security-antimalware.md) 或 Microsoft 合作夥伴的端點保護解決方案 ([Trend Micro](https://help.deepsecurity.trendmicro.com/azure-marketplace-getting-started-with-deep-security.html)、[Symantec](https://www.symantec.com/products)、[McAfee](https://www.mcafee.com/us/products.aspx)、[Windows Defender](https://www.microsoft.com/search/result.aspx?q=Windows+defender+endpoint+protection) 及 [System Center Endpoint Protection](https://www.microsoft.com/search/result.aspx?q=System+Center+endpoint+protection))。

Microsoft Antimalware 包含下列功能：即時防護、排程掃描、惡意程式碼補救、簽章更新、引擎更新、範例報告和排除事件收集。 對於與您的生產環境分開裝載的環境，您可以使用反惡意程式碼擴充功能來協助保護 VM 和雲端服務。

您可以將 Microsoft Antimalware 和合作夥伴解決方案與 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/)整合，以方便部署和執行內建偵測 (警示與事件)。

**最佳做法**：安裝反惡意程式碼解決方案以抵禦惡意程式碼。   
**詳細資訊**：[安裝 Microsoft 合作夥伴解決方案或 Microsoft Antimalware](../security-center/security-center-install-endpoint-protection.md)

**最佳做法**：整合反惡意程式碼解決方案與資訊安全中心，確實監視您的防護狀態。   
**詳細資訊**：[透過資訊安全中心管理端點保護問題](../security-center/security-center-partner-integration.md)

## <a name="manage-your-vm-updates"></a>管理您的 VM 更新
Azure VM 就跟所有內部部署 VM 一樣，受控於使用者。 Azure 不會向使用者推送 Windows 更新。 您需自行管理 VM 更新。

**最佳做法**：讓 VM 保持最新狀態。   
**詳細資訊**：使用 Azure 自動化中的[更新管理](../automation/automation-update-management.md)解決方案，以便管理 Azure、內部部署環境或其他雲端提供者中所部署 Windows 和 Linux 電腦的作業系統更新。 您可以快速評估所有代理程式電腦上可用更新的狀態，並管理為伺服器安裝必要更新的程序。

「更新管理」所管理的電腦會使用下列設定來執行評估和更新部署：

- 適用於 Windows 或 Linux 的 Microsoft Monitoring Agent (MMA)
- 適用於 Linux 的 PowerShell 預期狀態組態 (DSC)
- 自動化 Hybrid Runbook Worker
- 適用於 Windows 電腦的 Microsoft Update 或 Windows Server Update Services (WSUS)

若您使用 Windows Update，請讓自動安裝 Windows Update 設定保持啟用狀態。

**最佳做法**：部署時確認您建置的映像包含最新一輪的 Windows 更新。   
**詳細資訊**：檢查並安裝所有 Windows 更新是每個部署的第一個步驟。 在部署來自您或您自己的程式庫之映像時，套用此量值特別重要。 雖然從 Azure Marketplace 取得的映像會自動更新，但根據預設，在公開發行之後可能會有一段延隔時間 (最多數週)。

**最佳做法**：定期重新部署 VM 以強制執行最新版的作業系統。   
**詳細資訊**：使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)來定義 VM，以便輕鬆重新部署。 使用範本可讓您在需要時取得經過修補的安全 VM。

**最佳做法**：安裝最新的安全性更新。   
**詳細資訊**：客戶移至 Azure 的第一批工作負載中包括實驗室和對外系統。 如果 Azure VM 會裝載需要開放網際網路存取的應用程式或服務，請對修補作業保持警戒。 作業系統之外的修補。 合作夥伴應用程式上未修補的弱點也可能造成問題，但只要妥善管理修補程式即可避免這類問題。

**最佳做法**：部署和測試備份解決方案。   
**詳細資訊**：備份的處理方式必須和任何其他作業的處理方式相同。 您生產環境中延伸至雲端的系統很適合採用這種作法。

測試和開發系統都必須遵循備份策略，而這些備份策略能夠根據使用者的內部部署環境經驗，為使用者提供他們已經習慣的類似還原功能。 可能的話，移至 Azure 的生產工作負載應該與現有的備份解決方案整合。 或者，您可以使用 [Azure 備份](../backup/backup-azure-vms-first-look-arm.md)來協助您滿足備份需求。

未強制執行軟體更新原則的組織會更容易遭受利用已知的先前已修正弱點威脅的攻擊。 為了遵守業界法規，公司必須證明自己盡心盡力，並使用正確的安全性控制項來協助確保其位於雲端上的工作負載安全無虞。

傳統資料中心的軟體更新最佳做法和 Azure IaaS 有許多相似之處。 建議您評估目前的軟體更新原則來包含位於 Azure 中的 VM。

## <a name="manage-your-vm-security-posture"></a>管理您的 VM 安全性狀態
網路威脅日新月異。 保護您的 VM 需仰賴監視功能，以便快速偵測到威脅、防止未經授權存取您的資源、觸發警示，以及減少誤判。

若要監視 [Windows](../security-center/security-center-virtual-machine.md) 和 [Linux VM](../security-center/security-center-linux-virtual-machine.md) 的安全狀態，請使用 [Azure 資訊安全中心](../security-center/security-center-intro.md)。 請利用資訊安全中心的下列功能保護您的 VM：

- 套用具有建議組態規則的 OS 安全性設定。
- 找出並下載可能遺失的系統安全性和重大更新。
- 部署針對端點反惡意程式碼保護的建議項目。
- 驗證磁碟加密。
- 評估和修復弱點。
- 偵測威脅。

資訊安全中心可以主動監視威脅，並將可能的威脅公開於安全性警示之下。 相互關聯的威脅將彙總於稱為安全性事件的單一檢視畫面中。

資訊安全中心會將資料儲存在 [Azure Log Analytics](../log-analytics/log-analytics-overview.md) 中。 Log Analytics 提供查詢語言和分析引擎，可讓您深入解析應用程式和資源的作業。 也會從 [Azure 監視器](../monitoring-and-diagnostics/monitoring-overview.md)、管理解決方案，以及安裝在雲端或內部部署環境中虛擬機器上的代理程式收集資料。 此共用功能可協助您完全了解整個環境。

組織若未針對其 VM 強制執行強式安全性，將一概無法得知未經授權的使用者可能嘗試規避安全性控制項。

## <a name="monitor-vm-performance"></a>監視 VM 效能
當 VM 程序比所應消耗更多的資源時，可能會產生資源不當使用的問題。 VM 的效能問題可能會導致服務中斷，違反可用性的安全性原則。 這對於裝載 IIS 或其他 Web 伺服器的 VM 而言尤其重要，因為高 CPU 或記憶體使用量可能表示發生拒絕服務 (DoS) 攻擊。 請務必了解，不只需要在發生問題時以反應性方式監視 VM 存取，還必須主動監視正常作業期間測量的基準效能。

建議您使用 [Azure 監視器](../monitoring-and-diagnostics/monitoring-overview-metrics.md)來查看您的資源健康狀態。 Azure 監視器功能：

- [資源診斷記錄檔](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)：監視 VM 資源，並找出可能影響效能和可用性的潛在問題。
- [Azure 診斷擴充功能](../monitoring-and-diagnostics/azure-diagnostics.md)：在 Windows VM 上提供監視和診斷功能。 您可以將此擴充功能納入為 Azure Resource Manager [範本](../virtual-machines/windows/extensions-diagnostics-template.md)的一部分，藉以啟用這些功能。

未監視 VM 效能的組織無法判斷效能模式中的特定變更為正常還是不正常。 若 VM 消耗的資源比平常還多，可能表示發生來自外部資源的攻擊，或是在此 VM 中執行的程序遭入侵。

## <a name="encrypt-your-virtual-hard-disk-files"></a>加密虛擬硬碟檔案
建議您加密虛擬硬碟 (VHD)，以協助保護開機磁碟區和儲存體中的待用資料磁碟區，還有加密金鑰與密碼。

[Azure 磁碟加密](azure-security-disk-encryption-overview.md)可協助您為 Windows 和 Linux IaaS 虛擬機器磁碟加密。 Azure 磁碟加密使用 Windows 的業界標準 [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) 功能和 Linux 的 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) 功能，為作業系統和資料磁碟提供磁碟區加密。 此解決方案與 [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) 整合，協助您控制及管理金鑰保存庫訂用帳戶中的磁碟加密金鑰與密碼。 此解決方案也可確保虛擬機器磁碟上的所有待用資料都會在 Azure 儲存體中加密。

以下是使用 Azure 磁碟加密的最佳做法：

**最佳做法**：在 VM 上啟用加密。   
**詳細資訊**：Azure Disk 磁碟加密會產生加密金鑰並將其寫入金鑰保存庫。 在金鑰保存庫中管理加密金鑰需要 Azure AD 驗證。 基於此目的，請建立 Azure AD 應用程式。 針對驗證目的，您可以使用用戶端密碼式驗證或[用戶端憑證式 Azure AD 驗證](../active-directory/active-directory-certificate-based-authentication-get-started.md)。

**最佳做法**：使用金鑰加密金鑰 (KEK) 來為加密金鑰額外添加一層安全性。 將 KEK 新增至金鑰保存庫。   
**詳細資訊**：使用 [Add-AzureKeyVaultKeyy](https://docs.microsoft.com/powershell/module/azurerm.keyvault/add-azurekeyvaultkey) Cmdlet 在金鑰保存庫中建立金鑰加密金鑰。 您也可以從內部部署硬體安全性模組 (HSM) 匯入 KEK 以管理金鑰。 如需詳細資訊，請參閱 [Key Vault 文件](../key-vault/key-vault-hsm-protected-keys.md)。 若指定了金鑰加密金鑰，Azure 磁碟加密會先使用該金鑰包裝加密祕密，再寫入 Key Vault。 將此金鑰的委付複本保存在內部部署金鑰管理 HSM，可提供額外保護，以防意外刪除金鑰。

**最佳做法**：在磁碟加密前製作[快照集](../virtual-machines/windows/snapshot-copy-managed-disk.md)及/或進行備份。 如果在加密期間發生非預期的失敗，備份可提供復原選項。   
**詳細資訊**：具有受控磁碟的 VM 需要有備份才能進行加密。 在建立備份後，您可以使用 **Set-AzureRmVMDiskEncryptionExtension** Cmdlet 並指定 *-skipVmBackup* 參數來加密受控磁碟。 如需如何備份和還原已加密 VM 的詳細資訊，請參閱 [Azure 備份](../backup/backup-azure-vms-encryption.md)一文。

**最佳做法**：為了確保加密密碼不會跨出區域界限，Azure 磁碟加密需要讓金鑰保存庫和 VM 共置於相同區域中。   
**詳細資訊**：在和所要加密 VM 相同的區域中建立並使用金鑰保存庫。

套用 Azure 磁碟加密時，可滿足下列業務需求：

- 系統會透過業界標準的加密技術保護 IaaS VM 安全無虞，解決組織的安全性與法務遵循需求。
- 開啟 IaaS VM 受到客戶控制的金鑰和原則限制，且您可以在金鑰保存庫中稽核其使用狀況。

## <a name="next-steps"></a>後續步驟
如需更多安全性最佳做法，請參閱 [Azure 安全性最佳做法與模式](security-best-practices-and-patterns.md)，以便在使用 Azure 設計、部署和管理雲端解決方案時使用。

下列資源可提供更多有關 Azure 安全性和相關 Microsoft 服務的一般資訊：
* [Azure 安全性部落格](https://blogs.msdn.microsoft.com/azuresecurity/) - Azure 安全性的最新資訊
* [Microsoft 安全性回應中心](https://technet.microsoft.com/library/dn440717.aspx) -- 可在其中回報 Microsoft 安全性弱點 (包括 Azure 的問題) 或透過電子郵件傳送給 secure@microsoft.com
