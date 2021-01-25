---
title: 管理與監視安全性功能-Microsoft Azure |Microsoft Docs
description: 本文涵蓋 Azure 提供來協助管理和監視 Azure 雲端服務和虛擬機器的安全性功能和服務概觀。
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 5cf2827b-6cd3-434d-9100-d7411f7ed424
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/24/2021
ms.author: terrylan
ms.openlocfilehash: d85b1fdd433c372bb41adec6e3d33013f19363f0
ms.sourcegitcommit: 3c8964a946e3b2343eaf8aba54dee41b89acc123
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98747168"
---
# <a name="azure-security-management-and-monitoring-overview"></a>Azure 安全性管理和監視概觀
本文涵蓋 Azure 提供來協助管理和監視 Azure 雲端服務和虛擬機器的安全性功能和服務概觀。

## <a name="azure-role-based-access-control"></a>Azure 角色型存取控制

Azure 角色型存取控制 (Azure RBAC) 提供適用于 Azure 資源的詳細存取管理。 藉由使用 Azure RBAC，您可以只授與使用者執行其作業所需的存取權數量。 Azure RBAC 也可以協助您確保當使用者離開組織時，他們會失去雲端資源的存取權。

深入了解：

* [Azure RBAC 上的 Active Directory team blog](https://cloudblogs.microsoft.com/enterprisemobility/?product=azure-active-directory)
* [Azure 角色型存取控制 (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md)

## <a name="antimalware"></a>反惡意程式碼

運用 Azure，您可以使用來自各大安全性廠商 (例如 Microsoft、Symantec、Trend Micro、McAfee 和 Kaspersky) 的反惡意程式碼軟體。 此軟體可協助保護您的虛擬機器來抵禦惡意檔案、廣告軟體和其他威脅。

適用於 Azure 雲端服務和虛擬機器的 Microsoft Antimalware 可讓您針對 PaaS 角色和虛擬機器安裝反惡意程式碼代理程式。 根據 System Center Endpoint Protection，這項功能會將經證實的內部部署安全性技術帶入雲端。

我們也會在 Azure 平臺中提供趨勢的 [深度安全性](https://www.trendmicro.com/us/enterprise/cloud-solutions/deep-security/) 與 [>securecloud](https://www.trendmicro.com/us/enterprise/cloud-solutions/secure-cloud/) 產品的深度整合。 Deep Security 是一種防毒解決方案，而 SecureCloud 則是一種加密解決方案。 Deep Security 是透過延伸模組模型部署於 VM 內的。 透過使用 Azure 入口網站 UI 和 PowerShell，您可以選擇在所啟動的新 VM 內或已部署的現有 VM 內使用 Deep Security。

Azure 也支援 Symantec Endpoint Protection (SEP)。 透過入口網站整合，您可以指定想要在 VM 上使用 SEP。 SEP 可以透過 Azure 入口網站安裝在新的 VM 上，或透過 PowerShell 安裝在現有的 VM 上。

深入了解：

* [在 Azure 虛擬機器上部署反惡意程式碼解決方案](https://azure.microsoft.com/blog/deploying-antimalware-solutions-on-azure-virtual-machines/)
* [適用於 Azure 雲端服務和虛擬機器的 Microsoft Antimalware](antimalware.md)
* [如何在 Windows VM 上安裝和設定 Trend Micro Deep Security as a Service](../../virtual-machines/extensions/trend.md)
* [如何在 Windows VM 上安裝和設定 Symantec Endpoint Protection](../../virtual-machines/extensions/symantec.md)
* [可用於保護 Azure 虛擬機器的新反惡意程式碼選項](https://azure.microsoft.com/blog/new-antimalware-options-for-protecting-azure-virtual-machines/) \(英文\)

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication

Azure AD Multi-Factor Authentication 是一種需要使用一個以上驗證方法的驗證方法。 它可以為使用者登入和交易新增重要的第二層安全性。

Multi-Factor Authentication 有助於保護對資料與應用程式的存取，同時滿足使用者對簡單登入程序的需求。 它可以透過一系列的驗證選項 (例如電話、簡訊，或是行動應用程式通知或驗證碼) 和第三方 OATH 權杖來提供強大的驗證功能。

深入了解：

* [Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/)
* [什麼是 Azure AD Multi-Factor Authentication？](../../active-directory/authentication/concept-mfa-howitworks.md)
* [Azure AD Multi-Factor Authentication 的作用](../../active-directory/authentication/concept-mfa-howitworks.md)

## <a name="expressroute"></a>ExpressRoute

您可以使用 Azure ExpressRoute，透過連線提供者所提供的專用私人連線，將內部部署網路延伸到 Microsoft Cloud。 使用 ExpressRoute，您可以建立與 Microsoft 雲端服務（例如 Azure、Microsoft 365 和 CRM Online）的連線。 連線可以來自：

* 任意點對任意點 (IP VPN) 網路。
* 點對點乙太網路。
* 共置設備上透過連線提供者的虛擬交叉連線。

ExpressRoute 連線不會越過公用網際網路。 相較於透過網際網路的一般連線，它們可以提供更為可靠、速度更快、延遲更低且安全性更高的網際網路連線。

深入了解：

* [ExpressRoute 技術概觀](../../expressroute/expressroute-introduction.md)

## <a name="virtual-network-gateways"></a>虛擬網路閘道

VPN 閘道 (也稱為 Azure 虛擬網路閘道) 可用來在虛擬網路與內部部署位置之間傳送網路流量。 它們也用來在 Azure 內多個虛擬網路之間傳送流量 (網路對網路)。 VPN 閘道提供 Azure 與基礎結構之間的跨單位安全連線。

深入了解：

* [關於 VPN 閘道](../../vpn-gateway/vpn-gateway-about-vpngateways.md)
* [Azure 網路安全性概觀](network-overview.md)

## <a name="privileged-identity-management"></a>Privileged Identity Management

使用者有時候需要在 Azure 資源或其他 SaaS 應用程式中執行特殊權限的作業。 這通常表示組織會授與他們永久的 Azure Active Directory (Azure AD) 特殊存取權限。

這會提高雲端資源的安全性風險，因為組織無法滴水不漏地監視這些使用者利用其特殊存取權限的所作所為。 此外，如果具特殊存取權限的使用者帳戶遭到入侵，這個缺口就會影響組織的整體雲端安全性。 Azure AD Privileged Identity Management 有助於解決這項風險，方法是降低權限的曝光時間，並增加使用情形的可見性。  

Privileged Identity Management 引進了角色的臨時系統管理員或「即時」系統管理員存取權的概念。 這種系統管理員是需要針對該指派的角色完成啟用程序的使用者。 啟用程序會在指定的期間，將 Azure AD 中角色的使用者指派從非作用中變更為作用中。

深入了解：

* [Azure AD 特殊權限身分識別管理](../../active-directory/privileged-identity-management/pim-configure.md)
* [開始使用 Azure AD Privileged Identity Management](../../active-directory/privileged-identity-management/pim-getting-started.md)

## <a name="identity-protection"></a>身分識別保護

Azure AD Identity Protection 提供可疑登入活動和潛在弱點的合併檢視，以協助保護您的企業。 Identity Protection 會根據如下的訊號，來偵測使用者和具特殊權限 (系統管理員) 身分識別的可疑活動：

* 暴力密碼破解攻擊。
* 認證外洩。
* 從不熟悉的位置和受感染的裝置登入。

藉由提供通知和建議的補救，Identity Protection 有助於即時降低風險。 它會計算使用者風險嚴重性。 您可以設定以風險為基礎的原則，自動協助保護應用程式存取，以免未來受到威脅。

深入了解：

* [Azure Active Directory Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md)
* [第 9 頻道：Azure AD 和身分識別展示：Identity Protection 預覽](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)

## <a name="security-center"></a>資訊安全中心

Azure 資訊安全中心可協助您保護、偵測威脅並採取相應的措施。 「安全性中心」可讓您更深入瞭解及控制您的 Azure 資源及混合式雲端環境中的安全性。 

資訊安全中心會對您的已連線資源執行持續的安全性評估，並將其設定和部署與 [Azure 安全性基準測試](../benchmarks/introduction.md) 進行比較，以提供針對您的環境量身打造的詳細安全性建議。

資訊安全中心藉由下列方式來協助您最佳化和監視 Azure 資源安全性︰

- 讓您能夠根據下列各項來定義適用於 Azure 訂用帳戶資源的原則：
    - 您組織的安全性需求。
    - 每個訂用帳戶中應用程式的類型或資料的敏感度。
    - 任何適用于您訂用帳戶的產業或法規標準或基準測試。 
- 監視 Azure 虛擬機器、網路和應用程式的狀態。
- 提供排列優先順序的安全性警示清單，包括來自整合式協力廠商解決方案的警示。 它也會提供讓您用來快速調查攻擊所需資訊，以及如何補救它的建議事項。

深入了解：

* [Azure 資訊安全中心簡介](../../security-center/security-center-introduction.md)
* [改善您 Azure 資訊安全中心的安全分數](../../security-center/secure-score-security-controls.md)

## <a name="intelligent-security-graph"></a>Intelligent Security Graph

Intelligent Security Graph 能在 Microsoft 產品及服務中提供即時威脅保護。 它會使用能連結大量威脅情報和安全性資料的進階分析，來提供可強化組織安全性的見解。 Microsoft 會採用進階分析來提供更豐富的見解，這些分析每月可處理 4500 億筆驗證、掃描 4000 億封電子郵件是否有惡意程式碼與網路釣魚，並更新十億部裝置。 這些見解可協助您的組織迅速地偵測並回應攻擊。

* [Intelligent Security Graph](https://www.microsoft.com/security/intelligence)

## <a name="next-steps"></a>後續步驟
瞭解共用的 [責任模型](shared-responsibility.md) ，以及由 Microsoft 處理哪些安全性工作，以及由您處理哪些工作。

如需安全性管理的詳細資訊，請參閱 [Azure 的安全性管理](management.md)。