---
title: 在 Azure 資訊安全中心新增 Web 應用程式防火牆 | Microsoft Docs
description: 本文件說明如何實作「Azure 資訊安全中心」的「新增 Web 應用程式防火牆」和「完成應用程式保護」建議。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 8f56139a-4466-48ac-90fb-86d002cf8242
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/15/2018
ms.author: terrylan
ms.openlocfilehash: e28a1f6b865dae3abe2cb9dfac2921c6a2034491
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34203517"
---
# <a name="add-a-web-application-firewall-in-azure-security-center"></a>在 Azure 資訊安全中心新增 Web 應用程式防火牆
Azure 資訊安全中心可能會建議您從 Microsoft 合作夥伴新增 Web 應用程式防火牆 (WAF)，以保護您 Web 應用程式的安全。 本文件逐步解說如何套用此建議的範例。

系統會針對任何具有相關聯網路安全性群組 (包含開放輸入 Web 連接埠 (80,443)) 的公開 IP (執行個體層級 IP 或負載平衡 IP)，顯示 WAF 建議。

資訊安全中心建議您佈建 WAF，協助對抗以虛擬機器和外部 App Service 環境 (ASE) (部署於[隔離式](https://azure.microsoft.com/pricing/details/app-service/windows/)服務方案下) 上的 Web 應用程式為目標的攻擊。 隔離式方案會在私人專用 Azure 環境中託管您的應用程式，適合需要與內部部署網路安全連線或額外效能和規模的應用程式使用。 應用程式除了會在隔離環境外，還必須有外部 IP 位址負載平衡器。 若要深入了解 ASE，請參閱 [App Service 環境的文件](../app-service/environment/intro.md)。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  本文件不是一份逐步解說指南。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 之下，選取 [使用 Web 應用程式防火牆保護 Web 應用程式]。
   ![保護 Web 應用程式][1]
2. 在 [使用 Web 應用程式防火牆保護 Web 應用程式] 之下，選取 Web 應用程式。 [新增 Web 應用程式防火牆] 隨即開啟。
   ![新增 Web 應用程式防火牆][2]
3. 您可以選擇使用現有的 Web 應用程式防火牆 (如果有的話)，或者您可以建立一個新的 Web 應用程式防火牆。 此範例中沒有任何可用的現有 WAF，因此我們會建立一個 WAF。
4. 若要建立 WAF，請從整合式合作夥伴的清單中選取一個解決方案。 在此範例中，我們會選取 [Barracuda Web 應用程式防火牆]。
5. [Barracuda Web 應用程式防火牆]  隨即開啟，為您提供合作夥伴解決方案的相關資訊。 選取 [建立] 。

   ![防火牆資訊刀鋒視窗][3]

6. [新增 Web 應用程式防火牆] 隨即開啟，您可以在此視窗中執行 [VM 組態] 步驟並提供 [WAF 資訊]。 選取 [VM 組態]。
7. 在 [VM 組態] 之下，輸入啟動要執行 WAF 之虛擬機器所需的資訊。
   ![VM configuration][4]
8. 返回 [新增 Web 應用程式防火牆]，然後選取 [WAF 資訊]。 在 [WAF 資訊] 之下，設定 WAF 本身。 步驟 7 可讓您設定要執行 WAF 的虛擬機器，而步驟 8 則可讓您佈建 WAF 本身。

## <a name="finalize-application-protection"></a>完成應用程式保護
1. 返回**建議**。 在您建立 WAF 之後會產生一個新項目，稱為 [完成應用程式保護] 。 此項目可讓您知道您需要完成實際串聯起 Azure 虛擬網路內 WAF 的程序，讓它可以保護應用程式。

   ![完成應用程式保護][5]

2. 選取 [完成應用程式保護] 。 此時會開啟新的分頁。 您會看到有一個需要重新路由流量的 Web 應用程式。
3. 選取 Web 應用程式。 將會開啟一個刀鋒視窗，其中提供完成 Web 應用程式防火牆設定的步驟。 完成這些步驟，然後選取 [限制流量] 。 接著，資訊安全中心會為您進行串聯。

   ![限制流量][6]

> [!NOTE]
> 您可以將這些應用程式加入現有的 WAF 部署，以保護資訊安全中心的多個 Web 應用程式。
>
>

現在已將來自該 WAF 的記錄完全整合。 「資訊安全中心」可以開始自動收集並分析記錄，以便對您顯示重要的安全性警示。

## <a name="next-steps"></a>後續步驟
本文件說明如何實作資訊安全中心建議的「新增 Web 應用程式」。 若要深入了解如何設定 Web 應用程式防火牆，請參閱下列各項：

* [設定 App Service 環境的 Web 應用程式防火牆 (WAF)](../app-service/environment/app-service-app-service-environment-web-application-firewall.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) --了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助您保護您的 Azure 資源。
* [Azure 安全性中心常見問題集](security-center-faq.md) -- 尋找使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-add-web-application-firewall/secure-web-application.png
[2]:./media/security-center-add-web-application-firewall/add-a-waf.png
[3]: ./media/security-center-add-web-application-firewall/info-blade.png
[4]: ./media/security-center-add-web-application-firewall/select-vm-config.png
[5]: ./media/security-center-add-web-application-firewall/finalize-waf.png
[6]: ./media/security-center-add-web-application-firewall/restrict-traffic.png
