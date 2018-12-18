---
title: 資料安全性和加密最佳做法 | Microsoft Docs
description: 此文章提供使用內建 Azure 功能的一些資料安全性和加密最佳做法。
services: security
documentationcenter: na
author: barclayn
manager: mbalwin
editor: TomSh
ms.assetid: 17ba67ad-e5cd-4a8f-b435-5218df753ca4
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/19/2018
ms.author: barclayn
ms.openlocfilehash: 263c04fd15240f365f2325c69d5cb25aa1a539f0
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465872"
---
# <a name="azure-data-security-and-encryption-best-practices"></a>Azure 資料安全性和加密最佳做法
若要協助保護雲端上的資料，您必須考慮您的資料可能會發生的狀態，以及哪些控制項適用於該狀態。 Azure 資料安全性和加密的最佳做法與下列資料狀態相關：

- 待用︰這包括實體媒體 (磁碟或光碟) 上以靜態方式存在的所有資訊儲存物件、容器和類型。
- 傳輸中：當資料在元件、位置或程式之間傳輸時，就為傳輸中。 範例會透過網路、跨服務匯流排 (從內部部署到雲端，反之亦然，包括諸如 ExpressRoute 的混合式連線)，或輸入/輸出過程期間傳輸。

此文章將討論 Azure 資料安全性和加密最佳做法的集合。 這些最佳做法衍生自我們的 Azure 資料安全性和加密經驗和客戶的經驗。

針對每個最佳做法，我們會說明︰

* 最佳做法是什麼
* 您為何想要啟用該最佳做法
* 如果無法啟用最佳做法，結果可能為何
* 最佳做法的可能替代方案
* 如何學習啟用最佳做法

這篇「Azure 資料安全性和加密最佳做法」是以共識意見以及 Azure 平台功能和特性集 (因為在撰寫此文章時已存在) 為基礎。 意見和技術會隨著時間改變，這篇文章將會定期進行更新以反映這些變更。

## <a name="choose-a-key-management-solution"></a>選擇金鑰管理解決方案
保護您的金鑰對於保護雲端中的資料至關重要。

[Azure Key Vault](../key-vault/key-vault-overview.md) 可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和祕密。 金鑰保存庫簡化了金鑰管理程序，並可讓您控管存取和加密資料的金鑰。 開發人員可以在幾分鐘內建立開發和測試的金鑰，然後將它們移轉至生產金鑰。 安全性系統管理員可以視需要授與 (和撤銷) 存取金鑰的權限。

您可以使用 Key Vault 建立多個安全的容器，稱之為保存庫。 這些保存庫由 HSM 支援。 保存庫透過集中儲存應用程式祕密，協助減少意外遺失安全性資訊的機會。 Key Vault 也會控制和記錄其中所儲存項目的存取權。 Azure Key Vault 可以處理要求及更新傳輸層安全性 (TLS) 憑證。 它為憑證生命週期管理提供了健全的解決方案功能。

Azure Key Vault 設計用來支援應用程式金鑰和祕密。 Key Vault 的用意並非作為使用者密碼的存放區。

以下是使用金鑰保存庫的安全性最佳做法。

**最佳做法**：在特定範圍對使用者、群組和應用程式授與存取權。   
**詳細資料**：使用 RBAC 的預先定義的角色。 例如，若要對使用者授與管理金鑰保存庫的權限，您會在特定範圍對此使用者指派預先定義的角色 [Key Vault 參與者](../role-based-access-control/built-in-roles.md)。 在此案例中，範圍會是訂用帳戶、資源群組或只是特定金鑰保存庫。 如果預先定義的角色不符合您的需求，您可以[定義您自己的角色](../role-based-access-control/custom-roles.md)。

**最佳做法**：控制哪些使用者具有存取權。   
**詳細資料**：金鑰保存庫的存取權可透過兩個不同介面來控制︰管理平面和資料平面。 管理平面和資料平面的存取控制在運作上互不相關。

使用 RBAC 控制使用者可以存取的內容。 例如，如果您想要對應用程式授與金鑰保存庫中金鑰的使用權限，您只需要使用金鑰保存庫存取原則對資料平面授與存取權限，此應用程式完全不需要管理平面的存取權。 相反地，如果您想要讓使用者能夠讀取保存庫屬性和標籤，但不讓他擁有金鑰、密碼或憑證的任何存取權，您可以使用 RBAC 對這位使用者授與「讀取」權限，但不需要授與資料平面的存取權。

**最佳做法**：將憑證儲存在金鑰保存庫。 您的憑證價值很高。 若陷入有心人的控制中，就可能危害您的應用程式安全性或資料的安全性。   
**詳細資料**：Azure Resource Manager 可以在部署 VM 時，將儲存在 Azure Key Vault 中的憑證安全地部署到 Azure VM。 透過為金鑰保存庫設定適當的存取原則，您也控制誰可以存取您的憑證。 另一個優點是您可以在 Azure Key Vault 中的單一位置管理所有憑證。 如需其他資訊，請參閱[將憑證從客戶管理的金鑰保存庫部署到 VM](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/) \(英文\)。

**最佳做法**：請確定您可以復原刪除的金鑰保存庫或金鑰保存庫物件。   
**詳細資料**：刪除金鑰保存庫或金鑰保存庫物件可能是不小心或惡意的。 啟用金鑰保存庫的虛刪除和清除保護功能，尤其是針對用來加密待用資料的金鑰。 刪除這些金鑰就相當於資料遺失，因此如有需要，您可以復原已刪除的保存庫和保存庫物件。 定期練習金鑰保存庫復原作業。

> [!NOTE]
> 如果使用者具有金鑰保存庫管理平面的參與者權限 (RBAC)，他們可以透過設定金鑰保存庫存取原則，對自己授與資料平面的存取權。 建議您嚴格控制擁有金鑰保存庫「參與者」權限的人員，以確保只有獲得授權的人員可以存取和管理您的金鑰保存庫、金鑰、密碼和憑證。
>
>

## <a name="manage-with-secure-workstations"></a>透過安全的工作站管理
> [!NOTE]
> 訂用帳戶管理員或擁有者應使用安全存取工作站或特殊權限存取工作站。
>
>

因為絕大多數的攻擊是以使用者為目標，所以端點會成為主要攻擊點之一。 攻擊者入侵端點，就可以利用使用者的認證來存取組織的資料。 大部分的端點攻擊可以利用使用者就是其本機工作站的系統管理員的這個事實。

**最佳做法**：使用安全管理工作站保護敏感性帳戶、工作和資料。   
**詳細資料**：使用[特殊權限存取工作站](https://technet.microsoft.com/library/mt634654.aspx)來減少工作站的受攻擊面。 這些安全的管理工作站可協助您減輕其中一些攻擊，以確保您的資料更為安全。

**最佳做法**：確保端點保護。   
**詳細資料**：在用來取用資料的所有裝置上強制執行安全性原則 (不論資料位置是雲端或內部部署)。

## <a name="protect-data-at-rest"></a>保護靜態資料的安全
[待用資料加密 (英文)](https://blogs.microsoft.com/cybertrust/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/) 是達到資料隱私性、合規性及資料主權的必要步驟。

**最佳做法**：套用磁碟加密來協助保護您的資料。   
**詳細資料**：使用 [Azure 磁碟加密](azure-security-disk-encryption.md)。 它可讓 IT 系統管理員加密 Windows 和 Linux IaaS VM 磁碟。 磁碟加密結合業界標準的 Windows BitLocker 功能和 Linux dm-crypt 功能，為 OS 和資料磁碟提供磁碟區加密。

Azure 儲存體和 Azure SQL Database 預設會加密待用資料，且許多服務會提供加密選項。 您可以使用 Azure Key Vault 控管存取和加密資料的金鑰。 若要深入了解，請參閱 [Azure 資源提供者加密模型支援](azure-security-encryption-atrest.md#azure-resource-providers-encryption-model-support)。

**最佳做法**：使用加密，協助降低與未經授權存取資料相關的風險。   
**詳細資料**：將機密資料寫入它們之前，先加密您的磁碟機。

未強制執行資料加密的組織會更容易遭受資料完整性問題的攻擊。 例如，未經授權或惡意使用者可能會竊取遭入侵帳戶中的資料，或未經授權存取以清除格式編碼的資料。 公司必須證明他們是十分用心，並使用正確的安全性控制項來增強資料安全性以遵守業界法規。

## <a name="protect-data-in-transit"></a>保護傳輸中的資料
保護傳輸中的資料應該是您的資料保護策略中不可或缺的部分。 因為資料會從許多位置來回移動，一般會建議您一律使用 SSL/TLS 通訊協定來交換不同位置的資料。 在某些情況下，您可能希望使用 VPN，隔離您的內部部署與雲端基礎結構之間的整個通訊通道。

對於在內部部署基礎結構與 Azure 之間移動的資料，請考慮適當的防護措施，例如 HTTPS 或 VPN。 當在 Azure 虛擬網路和內部部署位置之間傳送加密流量時，請使用 [Azure VPN 閘道](https://docs.microsoft.com/azure/vpn-gateway/)。

以下是特定於使用 Azure VPN 閘道、SSL/TLS，以及 HTTPS 的最佳做法。

**最佳做法**：從內部部署的多個工作站安全存取 Azure 虛擬網路。   
**詳細資料**：使用[站對站 VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)。

**最佳做法**：從內部部署的個別工作站安全存取 Azure 虛擬網路。   
**詳細資料**：[使用點對站 VPN](../vpn-gateway/vpn-gateway-point-to-site-create.md)。

**最佳做法**：通過專用的高速 WAN 連結移動大型資料集。   
**詳細資料**：使用 [ExpressRoute](../expressroute/expressroute-introduction.md)。 如果您選擇使用 ExpressRoute，您也可以透過使用 [SSL/TLS](https://support.microsoft.com/kb/257591) 或其他通訊協定，在應用程式層級加密資料，以提供額外的保護。

**最佳做法**：透過 Azure 入口網站與 Azure 儲存體互動。   
**詳細資料**：透過 HTTPS 發生的所有交易。 您也可以使用透過 HTTPS 的[儲存體 REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx) 來與 [Azure 儲存體](https://azure.microsoft.com/services/storage/)和 [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 互動。

無法保護傳輸中資料的組織比較容易遭受[攔截攻擊](https://technet.microsoft.com/library/gg195821.aspx)、[竊聽](https://technet.microsoft.com/library/gg195641.aspx)及工作階段攔截。 這些攻擊可能是取得機密資料存取權的第一步。

## <a name="secure-email-documents-and-sensitive-data"></a>保護電子郵件、 文件和敏感性資料
您想要控制及保護於公司外部共用的電子郵件、文件及敏感性資料。 [Azure 資訊保護](https://docs.microsoft.com/azure/information-protection/)為雲端式解決方案，可協助組織將其文件及電子郵件分類、加註標籤及進行保護。 定義規則和條件的系統管理員可自動完成此動作，使用者也可以手動方式完成，或在使用者取得建議的情況下組合完成。

不論資料儲存位置或共同對象為何，隨時都可識別分類。 標籤包括視覺標記，例如頁首、頁尾或浮水印。 中繼資料會以純文字新增至檔案和電子郵件標頭。 純文字可確保其他服務 (例如防止資料遺失的解決方案) 可以識別分類並採取適當動作。

此保護技術使用 Azure Rights Management (Azure RMS)。 此技術與其他 Microsoft 雲端服務和應用程式 (例如 Office 365 和 Azure Active Directory) 整合。 此保護技術使用加密、身分識別和授權原則。 透過 Azure RMS 套用的保護，不論位置是組織內部或外部、網路、檔案伺服器或應用程式，都可持續保護文件和電子郵件。

此資訊保護解決方案可讓您控制您的資料，即使資料與其他人共用也是如此。 您也可以將 Azure RMS 與您自己的特定業務應用程式和軟體廠商提供的資訊保護解決方案結合使用，不論這些應用程式和解決方案是在內部部署或雲端中。

建議您：

- 為您的組織[部署 Azure 資訊保護](https://docs.microsoft.com/azure/information-protection/deployment-roadmap)。
- 套用可反映您的業務需求的標籤。 例如：將名為「高度機密」的標籤套用於包含極機密資料的所有文件和電子郵件，來分類並保護此資料。 然後，只有授權的使用者可以存取此資料，並具有您指定的任何限制。
- 設定 [Azure RMS 的使用量記錄](https://docs.microsoft.com/azure/information-protection/log-analyze-usage)，以便您可以監視您的組織使用保護服務的方式。

[資料分類](http://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf)和檔案保護能力不佳的組織可能更容易受資料外洩或資料不當使用。 使用適當的檔案保護，您可以分析資料流程以深入了解您的業務、偵測風險行為並採取矯正措施、追蹤文件的存取等等。

## <a name="next-steps"></a>後續步驟
如需更多安全性最佳做法，請參閱 [Azure 安全性最佳做法與模式](security-best-practices-and-patterns.md)，以便在使用 Azure 設計、部署和管理雲端解決方案時使用。

下列資源可提供更多有關 Azure 安全性和相關 Microsoft 服務的一般資訊：
* [Azure 安全性小組部落格](https://blogs.msdn.microsoft.com/azuresecurity/) - Azure 安全性的最新資訊
* [Microsoft 安全性回應中心](https://technet.microsoft.com/library/dn440717.aspx) -- 可在其中回報 Microsoft 安全性弱點 (包括 Azure 的問題) 或透過電子郵件傳送給 secure@microsoft.com
