---
title: 將 Azure Active Directory Identity Protection 連線至 Azure 資訊安全中心 | Microsoft Docs
description: 了解 Azure 資訊安全中心如何整合 Azure Active Directory Identity Protection。
services: security-center
documentationcenter: na
author: terrylan
manager: MBaldwin
editor: ''
ms.assetid: 0d4b77c2-dba4-4e46-8f55-ab04ddd92496
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/13/2017
ms.author: yurid
ms.openlocfilehash: 6ab3611d47b32eddd41736df69a3e7382c2b220d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298756"
---
# <a name="connecting-azure-active-directory-identity-protection-to-azure-security-center"></a>將 Azure Active Directory Identity Protection 連線至 Azure 資訊安全中心
本文件可協助您設定 Azure Active Directory (AD) Identity Protection 與 Azure 資訊安全中心之間的整合。

## <a name="why-connect-azure-ad-identity-protection"></a>為什麼要和 Azure AD Identity Protection 連線？
[Azure AD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 能協助偵測會影響組織身分識別的潛在弱點。 連線之後，您就能在資訊安全中心內檢視 Azure AD Identity Protection 的警示。 此整合可讓您在資訊安全中心內檢視、相互關聯和調查與混合式雲端工作負載相關的所有安全性警示。 

## <a name="how-do-i-configure-this-integration"></a>如何設定此整合？
如果您的組織已經使用 Azure AD Identity Protection，請遵循下列步驟來設定整合：

1. 開啟 [資訊安全中心] 儀表板。
2. 在左窗格上，按一下 [安全性解決方案]。 如果您的組織已經啟用 Azure AD Identity Protection，則資訊安全中心會自動探索。

    ![AADIP](./media/security-center-aadip-integration/security-center-aadip-integration-fig1.png)

3. 按一下 [連線]。
4. 在 [整合 Azure AD Identity Protection] 頁面上，向下捲動並選取適當的工作區：

    ![工作區](./media/security-center-aadip-integration/security-center-aadip-integration-fig2.png)

5. 按一下 [ **連接**]。

一旦您完成這項設定，Azure AD Identity Protection 解決方案就會出現在 [安全性解決方案] 頁面的 [已連接的解決方案] 之下。 

## <a name="next-steps"></a>後續步驟
在本文件中，您已了解如何將 Azure AD Identity Protection 連線至資訊安全中心。 如要深入了解資訊安全中心，請參閱下列文章：

* [將 Microsoft Advanced Threat Analytics 連線至 Azure 資訊安全中心](security-center-ata-integration.md)
* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) — 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) — 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) — 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) — 了解如何監視合作夥伴解決方案的健全狀況。
- [Azure 資訊安全中心資料安全性](security-center-data-security.md) - 了解資訊安全中心如何管理及保護其中的資料。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 取得最新的 Azure 安全性新聞和資訊。


