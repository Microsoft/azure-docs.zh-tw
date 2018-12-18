---
title: Azure 資訊安全中心中的資源安全性檢查 | Microsoft Docs
description: 此文章可協助您開始在「Azure 資訊安全中心」中監視您的資源安全性檢查。
services: security-center
documentationcenter: na
author: rkarlin
manager: mbaldwin
editor: ''
ms.assetid: 3bd5b122-1695-495f-ad9a-7c2a4cd1c808
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 9/21/2018
ms.author: rkarlin
ms.openlocfilehash: 361fc2c6931dcfaed062bd39b75be7bef059b467
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46122057"
---
# <a name="monitoring-resource-security-hygiene-in-azure-security-center"></a>在 Azure 資訊安全中心監視資源安全性檢查
此文章可協助您使用「Azure 資訊安全中心」中的監視功能來確定您的資源安全性保護機制十分嚴密並監視原則合規性。

## <a name="what-is-resource-security-hygiene"></a>什麼是資源安全性檢查？
我們通常都將監視想像成監看及等候事件發生，以便對該情況作出反應。 資源安全性檢查是指擁有一個主動式策略來稽核您的資源，以識別出不符合組織標準或最佳做法的系統。

## <a name="resource-security-hygiene"></a>資源安全性檢查
在您為訂用帳戶的資源啟用[安全性原則](security-center-policies.md)之後，資訊安全中心會分析您資源的安全性狀態，以找出潛在的弱點。 您可以立即取得網路組態的相關資訊。 視您已安裝代理程式的 VM 和電腦數目而定，可能需要一個小時以上的時間才能收集有關 VM 和電腦的組態資訊，例如安全性更新狀態和作業系統組態。 您可以在 [建議] 圖格中檢視問題與強化網路與補救風險的完整清單。

您可以在檢視每個資源類型的資源安全性狀態與任何問題：

- 若要監視您電腦資源與應用程式的健康情況，並接收有關如何改進其安全性的建議，請參閱[保護 Azure 資訊安全中心內的機器和應用程式](security-center-virtual-machine-protection.md)
- 若要監視您的網路資源 (例如虛擬機器、網路安全性群組與端點) 並接收有關如何改進其安全性的建議，請參閱[保護 Azure 資訊安全中心內的網路](security-center-network-recommendations.md)以取得詳細資訊。 
- 若要監視您的資料與儲存體資源 (例如 SQL 伺服器與儲存體帳戶) 並接收有關如何改進其安全性的建議，請參閱[保護 Azure 資訊安全中心內的 Azure SQL 服務和資料](security-center-sql-service-recommendations.md)以取得詳細資訊。 
- 若要監視您的身分識別與存取資源 (包括 MFA 與帳戶權限) 並接收有關如何改進其安全性的建議，請參閱[在 Azure 資訊安全中心監視身分識別和存取](security-center-identity-access.md)以取得詳細資訊。 
- 若要監視對您資源的 Just-In-Time 存取，請參閱[使用 Just-In-Time 管理虛擬機器存取](security-center-just-in-time.md)以取得詳細資訊。 


如需如何套用建議的詳細資訊，請參閱[實作 Azure 資訊安全中心的安全性建議](security-center-recommendations.md)。



![資源安全性健康情況圖格](./media/security-center-monitoring/security-center-monitoring-fig1-newUI-2017.png)



## <a name="see-also"></a>另請參閱
在此文章中，您已來了解如何使用「Azure 資訊安全中心」的監視功能。 若要深入了解「Azure 資訊安全中心」，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md)：了解如何在 Azure 資訊安全中心設定安全性設定。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md)：了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md)：了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md)：尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)：尋找有關 Azure 安全性與相容性的部落格文章。
