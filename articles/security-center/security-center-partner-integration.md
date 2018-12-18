---
title: 在 Azure 資訊安全中心整合安全性解決方案 | Microsoft Docs
description: 了解 Azure 資訊安全中心如何與夥伴整合，以提高您 Azure 資源的整體安全性。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: mbaldwin
editor: ''
ms.assetid: 6af354da-f27a-467a-8b7e-6cbcf70fdbcb
ms.service: security-center
ms.topic: hero-article
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/07/2018
ms.author: terrylan
ms.openlocfilehash: b0e674eb161af41a848f0456a033d615293a9947
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39622784"
---
# <a name="integrate-security-solutions-in-azure-security-center"></a>在 Azure 資訊安全中心整合安全性解決方案
這份文件可協助您管理已連線到 Azure 資訊安全中心的安全性解決方案，並且新增新的項目。

## <a name="integrated-azure-security-solutions"></a>整合式 Azure 安全性解決方案
資訊安全中心可以使得在 Azure 中啟用整合式安全性解決方案變得簡單。 優點包括：

- 
  **簡化部署**：資訊安全中心提供整合式合作夥伴解決方案的精簡佈建。 針對像是反惡意程式碼軟體和弱點評量的解決方案，資訊安全中心可以在您的虛擬機器上佈建所需的代理程式，針對防火牆應用裝置，資訊安全中心可以處理大部分所需的網路設定。
- **整合偵測**：來自合作夥伴解決方案的安全性事件會自動收集、彙總以及顯示為資訊安全中心警示和事件的一部分。 這些事件也會與來自其他來源的偵測整合，以提供進階的威脅偵測功能。
- **統一的健全狀況監視與管理**：客戶可以使用整合式的健康情況事件，一眼就能監視所有合作夥伴解決方案。 提供基本管理功能，而且可以讓您輕鬆使用夥伴解決方案存取進階設定。

目前整合式安全性解決方案包括：

- 端點保護 ([Trend Micro](https://help.deepsecurity.trendmicro.com/azure-marketplace-getting-started-with-deep-security.html)、[Symantec](https://www.symantec.com/products)、[McAfee](https://www.mcafee.com/us/products.aspx)、[Windows Defender](https://www.microsoft.com/windows/comprehensive-security) 和 [System Center Endpoint Protection](https://docs.microsoft.com/sccm/protect/deploy-use/endpoint-protection))
- Web 應用程式防火牆 ([Barracuda](https://www.barracuda.com/products/webapplicationfirewall)、[F5](https://support.f5.com/kb/en-us/products/big-ip_asm/manuals/product/bigip-ve-web-application-firewall-microsoft-azure-12-0-0.html)、[Imperva](https://www.imperva.com/Products/WebApplicationFirewall-WAF)、[Fortinet](https://www.fortinet.com/products.html)，以及 [Azure 應用程式閘道](https://azure.microsoft.com/blog/azure-web-application-firewall-waf-generally-available/))
- 新一代防火牆 ([Check Point](https://www.checkpoint.com/products/vsec-microsoft-azure/)、[Barracuda](https://campus.barracuda.com/product/nextgenfirewallf/article/NGF/AzureDeployment/)、[Fortinet](http://docs.fortinet.com/d/fortigate-fortios-handbook-the-complete-guide-to-fortios-5.2)、[Cisco](http://www.cisco.com/c/en/us/td/docs/security/firepower/quick_start/azure/ftdv-azure-qsg.html) 和 [Palo Alto Networks](https://www.paloaltonetworks.com/products))
- 弱點評量 ([Qualys](https://www.qualys.com/public-clouds/microsoft-azure/) 和 [Rapid7](https://www.rapid7.com/products/insightvm/))

端點保護整合體驗會根據解決方案而有所不同。 下表包含每個解決方案體驗的詳細資料：

| 端點保護               | 平台                             | 資訊安全中心安裝 | 資訊安全中心探索 |
|-----------------------------------|---------------------------------------|------------------------------|---------------------------|
| Windows Defender (Microsoft 反惡意程式碼軟體)                  | Windows Server 2016                   | 否，內建於 OS           | 是                       |
| System Center Endpoint Protection (Microsoft 反惡意程式碼軟體) | Windows Server 2012 R2、2012、2008 R2 | 透過延伸模組                | 是                       |
| Trend Micro – 所有版本         | Windows Server 系列                 | 否                           | 是                       |
| Symantec v12.1.1100+              | Windows Server 系列                 | 否                           | 是                       |
| McAfee v10+                       | Windows Server 系列                 | 否                           | 是                       |
| Kaspersky                         | Windows Server 系列                 | 否                           | 否                        |
| Sophos                            | Windows Server 系列                 | 否                           | 否                        |



## <a name="how-security-solutions-are-integrated"></a>安全性解決方案如何整合
從資訊安全中心部署的 Azure 安全性解決方案會自動連線。 您也可以連線其他安全性資料來源，包括：

- Azure AD Identity Protection
- 在內部部署或其他雲端中執行的電腦
- 支援常見事件格式 (CEF) 的安全性解決方案
- Microsoft Advanced Threat Analytics

![夥伴解決方案整合](./media/security-center-partner-integration/security-center-partner-integration-fig8.png)

## <a name="manage-integrated-azure-security-solutions-and-other-data-sources"></a>管理整合式 Azure 安全性解決方案和其他資料來源

1. 登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)。

2. 在 [Microsoft Azure] 功能表中，選取 [資訊安全中心]。 [資訊安全中心 - 概觀] 隨即開啟。

  ![資訊安全中心概觀](./media/security-center-partner-integration/overview.png)

3. 在 [概觀] 底下，選取 [安全性解決方案]。

在 [安全性解決方案] 之下，您可以檢視整合式 Azure 安全性解決方案的健康情況相關資訊，並且執行基本管理工作。 您也可以連線其他類型的安全性資料來源，例如常見事件格式 (CEF) 的 Azure Active Directory Identity Protection 警示和防火牆記錄。

### <a name="connected-solutions"></a>連線的解決方案

[連線的解決方案] 區段包含目前連線到資訊安全中心的安全性解決方案，和每個解決方案健康情況狀態的的資訊。  

![連線的解決方案](./media/security-center-partner-integration/security-center-partner-integration-fig4.png)

如需詳細資訊，請參閱[管理已連線的合作夥伴解決方案](security-center-partner-solutions.md)。

### <a name="discovered-solutions"></a>探索到的解決方案

資訊安全中心會自動探索在 Azure 中執行但未連線到資訊安全中心的安全性解決方案，並且在 [搜索到的解決方案] 區段中顯示解決方案。 這包括 Azure 解決方案，例如 [Azure AD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)，以及在合作夥伴解決方案。

> [!NOTE]
> 已探索解決方案功能的訂用帳戶層級需要資訊安全中心的標準層。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。
>
>

選取解決方案下方的 [連線] 來與資訊安全中心整合，並可收到安全性警示通知。

![探索到的解決方案](./media/security-center-partner-integration/security-center-partner-integration-fig5.png)

資訊安全中心也會探索訂用帳戶中能夠轉寄通用事件格式 (CEF) 記錄的已部署解決方案。 了解如何[將安全性解決方案連線](quick-security-solutions.md) 到資訊安全中心，而該解決方案會使用 CEF 記錄。

### <a name="add-data-sources"></a>新增資料來源

[新增資料來源] 區段包含可以連線的其他可用資料來源。 如需從這些來源新增資料的指示，請按一下 [新增]。

![資料來源](./media/security-center-partner-integration/security-center-partner-integration-fig7.png)


## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何在資訊安全中心中整合夥伴解決方案。 如要深入了解資訊安全中心，請參閱下列文章：

* [將 Microsoft Advanced Threat Analytics 連線至 Azure 資訊安全中心](security-center-ata-integration.md)
* [將 Azure Active Directory Identity Protection 連線至 Azure 資訊安全中心](security-center-aadip-integration.md)
* [資訊安全中心的安全性健康情況監視](security-center-monitoring.md)。 了解如何監視 Azure 資源的健全狀況。
* [使用資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md)。 了解如何監視合作夥伴解決方案的健全狀態。
* [Azure 資訊安全中心常見問題](security-center-faq.md)。 取得有關使用資訊安全中心常見問題的答案。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)。 尋找有關 Azure 安全性與相容性的部落格文章。
