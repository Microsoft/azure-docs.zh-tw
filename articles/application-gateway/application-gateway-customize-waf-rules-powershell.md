---
title: 在 Azure 應用程式閘道中自訂 Web 應用程式防火牆規則 - PowerShell | Microsoft Docs
description: 本文提供如何透過 PowerShell，在應用程式閘道中自訂 Web 應用程式防火牆規則的相關資訊。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 07/26/2017
ms.author: victorh
ms.openlocfilehash: b341bdbe6611187b158f353d00077d33f317f374
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365419"
---
# <a name="customize-web-application-firewall-rules-through-powershell"></a>透過 PowerShell 自訂 Web 應用程式防火牆規則

> [!div class="op_single_selector"]
> * [Azure 入口網站](application-gateway-customize-waf-rules-portal.md)
> * [PowerShell](application-gateway-customize-waf-rules-powershell.md)
> * [Azure CLI](application-gateway-customize-waf-rules-cli.md)

Azure 應用程式閘道 Web 應用程式防火牆 (WAF) 提供 Web 應用程式的保護。 這些保護是由開放 Web 應用程式安全性專案 (OWASP) 的核心規則集 (CRS) 所提供。 某些規則可能會導致誤判，並封鎖真正的流量。 因此，應用程式閘道會提供功能以自訂規則群組與規則。 如需特定規則群組與規則的詳細資訊，請參閱 [Web 應用程式防火牆 CRS 規則群組與規則的清單](application-gateway-crs-rulegroups-rules.md)。

## <a name="view-rule-groups-and-rules"></a>檢視規則群組與規則

以下程式碼範例顯示如何檢視可在啟用 WAF 的應用程式閘道上設定的規則和規則群組。

### <a name="view-rule-groups"></a>檢視規則群組

下列範例顯示如何檢視規則群組：

```powershell
Get-AzureRmApplicationGatewayAvailableWafRuleSets
```

以下輸出是上述範例的截斷回應：

```
OWASP (Ver. 3.0):

    REQUEST-910-IP-REPUTATION:
        Description:
            
        Rules:
            RuleId     Description
            ------     -----------
            910011     Rule 910011
            910012     Rule 910012
            ...        ...

    REQUEST-911-METHOD-ENFORCEMENT:
        Description:
            
        Rules:
            RuleId     Description
            ------     -----------
            911011     Rule 911011
            ...        ...

OWASP (Ver. 2.2.9):

    crs_20_protocol_violations:
        Description:
            
        Rules:
            RuleId     Description
            ------     -----------
            960911     Invalid HTTP Request Line
            981227     Apache Error: Invalid URI in Request.
            960000     Attempted multipart/form-data bypass
            ...        ...
```

## <a name="disable-rules"></a>停用規則

下列範例會停用應用程式閘道上的規則 `910018` 和 `910017`：

```powershell
$disabledrules=New-AzureRmApplicationGatewayFirewallDisabledRuleGroupConfig -RuleGroupName REQUEST-910-IP-REPUTATION -Rules 910018,910017
Set-AzureRmApplicationGatewayWebApplicationFirewallConfiguration -ApplicationGateway $gw -Enabled $true -FirewallMode Detection -RuleSetVersion 3.0 -RuleSetType OWASP -DisabledRuleGroups $disabledrules
Set-AzureRmApplicationGateway -ApplicationGateway $gw
```

## <a name="next-steps"></a>後續步驟

設定已停用的規則之後，您可以了解如何檢視 WAF 記錄。 如需詳細資訊，請參閱[應用程式閘道診斷](application-gateway-diagnostics.md#diagnostic-logging)。

[fig1]: ./media/application-gateway-customize-waf-rules-portal/1.png
[1]: ./media/application-gateway-customize-waf-rules-portal/figure1.png
[2]: ./media/application-gateway-customize-waf-rules-portal/figure2.png
[3]: ./media/application-gateway-customize-waf-rules-portal/figure3.png
