---
title: 將 Microsoft Azure 應用程式閘道連線至 Azure 資訊安全中心 | Microsoft Docs
description: 了解如何整合應用程式閘道與 Azure 資訊安全中心以加強您資源的整體安全性。
services: security-center
documentationcenter: na
author: rkarlin
manager: mbaldwin
editor: ''
ms.assetid: 6af354da-f27a-467a-8b7e-6cbcf70fdbcb
ms.service: security-center
ms.topic: conceptual
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2018
ms.author: rkarlin
ms.openlocfilehash: 5638b71147592ae71c741ca86da68ddfec668af5
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299061"
---
# <a name="connecting-microsoft-azure-application-gateway-to-azure-security-center"></a>將 Microsoft Azure 應用程式閘道連線至 Azure 資訊安全中心
本文件可協助您設定以應用程式閘道 Web 應用程式防火牆 (WAF) 和資訊安全中心進行的整合。

## <a name="why-connect-application-gateway"></a>為何要連線應用程式閘道？
應用程式閘道中的 WAF 可保護 Web 應用程式，免於遭受常見的 Web 型攻擊，例如 SQL 插入式攻擊、跨網站指令碼攻擊和工作階段攔截。 資訊安全中心會與應用程式閘道整合，以防禦並偵測環境中未受保護之 Web 應用程式所受到的威脅。

## <a name="how-do-i-configure-this-integration"></a>如何設定此整合？
資訊安全中心會探索先前在訂用帳戶中部署的 WAF 執行個體。 將這些解決方案與資訊安全中心連線就能整合警示。

> [!NOTE]
> 您也可以從資訊安全中心的**建議**佈建應用程式閘道 WAF，如[新增 Web 應用程式防火牆](security-center-add-web-application-firewall.md)中所述。
>
>

1. 登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)。

2. 在 [Microsoft Azure] 功能表中，選取 [資訊安全中心]。

3. 在 [資源安全性檢查] 下方，選取 [安全性解決方案]。

  ![資訊安全中心概觀](./media/security-center-connect-application-gateway/overview.png)

4. 在 [探索到的解決方案] 底下尋找 Microsoft SaaS 型 Web 應用程式防火牆，然後選取 [連線]。

  ![探索到的解決方案](./media/security-center-connect-application-gateway/connect.png)

5. [連線 WAF 解決方案] 隨即開啟。  選取 [連線] 以整合 WAF 和資訊安全中心。

  ![連線 WAF 解決方案](./media/security-center-connect-application-gateway/waf-solution.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何在資訊安全中心內整合應用程式閘道 WAF。 如要深入了解資訊安全中心，請參閱下列文章：

* [在資訊安全中心內整合安全性解決方案](security-center-partner-integration.md)
* [將 Microsoft Advanced Threat Analytics 連線至資訊安全中心](security-center-ata-integration.md)
* [將 Azure Active Directory Identity Protection 連線至資訊安全中心](security-center-aadip-integration.md)
* [資訊安全中心的安全性健康情況監視](security-center-monitoring.md)。
* [使用資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md)。
* [Azure 資訊安全中心常見問題](security-center-faq.md)。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)。
