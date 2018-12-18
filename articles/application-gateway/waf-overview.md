---
title: 適用於 Azure 應用程式閘道的 Web 應用程式防火牆 (WAF) 簡介 | Microsoft Docs
description: 本頁面提供適用於應用程式閘道的 Web 應用程式防火牆 (WAF) 的概觀。
documentationcenter: na
services: application-gateway
author: amsriva
manager: rossort
editor: amsriva
ms.assetid: 04b362bc-6653-4765-86f6-55ee8ec2a0ff
ms.service: application-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/03/2017
ms.author: amsriva
ms.openlocfilehash: 9e04f69410251b5748facf44e9f2947b1415bc19
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32160829"
---
# <a name="web-application-firewall-waf"></a>Web 應用程式防火牆 (WAF)

Web 應用程式防火牆 (WAF) 是一項應用程式閘道功能，可提供 Web 應用程式的集中式保護，免於遭遇常見的攻擊和弱點。 

Web 應用程式防火牆會根據 [OWASP 核心規則集](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 或 2.2.9 中的規則提供保護。 Web 應用程式已逐漸成為利用常見已知弱點的惡意攻擊目標。 這些攻擊中最常見的是 SQL 插入式攻擊、跨網站指令碼攻擊等等。 想要防止應用程式的程式碼受到這類攻擊會非常困難，而且可能需要對多層次的應用程式拓撲執行嚴格的維護、修補和監視工作。 集中式 Web 應用程式防火牆有助於簡化安全性管理作業，且更加確保應用程式管理員能夠對抗威脅或入侵。 相較於保護每個個別的 Web 應用程式，WAF 方案還可透過在中央位置修補已知弱點，更快地因應安全性威脅。 現有的應用程式閘道可以輕易地轉換成已啟用 Web 應用程式防火牆的應用程式閘道。

![imageURLroute](./media/waf-overview/WAF1.png)

應用程式閘道可做為應用程式傳遞控制器來運作，並提供 SSL 終止、Cookie 型工作階段同質、循環配置資源負載分配、內容型路由，以及裝載多個網站和安全性增強功能的能力。 應用程式閘道提供的安全性增強功能包括 SSL 原則管理和端對端 SSL 支援。 直接整合到 ADC 供應項目的 WAF (Web 應用程式防火牆) 現已增強應用程式安全性。 這可讓您輕鬆地設定要管理的中央位置，並保護 Web 應用程式來對抗常見的 Web 弱點。

## <a name="benefits"></a>優點

應用程式閘道和 Web 應用程式防火牆提供的核心優勢如下︰

### <a name="protection"></a>保護

* 不需修改後端程式碼就能保護 Web 應用程式不受 Web 弱點和攻擊的威脅。

* 在應用程式閘道背後同時保護多個 Web 應用程式。 應用程式閘道可在單一閘道背後支援裝載最多 20 個網站，而且全都利用 WAF 受到免於 Web 攻擊的保護。

### <a name="monitoring"></a>監視

* 使用即時 WAF 記錄來監視 Web 應用程式對抗攻擊。 此記錄已經與 [Azure 監視器](../monitoring-and-diagnostics/monitoring-overview.md)整合，可追蹤 WAF 警示和記錄並輕鬆地監視趨勢。

* WAF 即將與 Azure 資訊安全中心整合。 Azure 資訊安全中心可供集中檢閱所有 Azure 資源的安全性狀態。

### <a name="customization"></a>自訂

* 能夠自訂 WAF 規則和規則群組，以符合您的應用程式需求並消除誤判。

## <a name="features"></a>特性

Web 應用程式防火牆已預先設定為使用 CRS 3.0，或者您可以選擇使用 2.2.9。 CRS 3.0 的誤判情形低於 2.2.9。 能夠[自訂規則以符合您需求](application-gateway-customize-waf-rules-portal.md)。 Web 應用程式防火牆防禦的一些常見 Web 弱點包括︰

* SQL 插入式攻擊保護
* 跨網站指令碼保護
* 常見 Web 攻擊保護，例如命令插入式攻擊、HTTP 要求走私、HTTP 回應分割和遠端檔案包含攻擊
* 防範 HTTP 通訊協定違規
* 防範 HTTP 通訊協定異常行為，例如遺漏主機使用者代理程式和接受標頭
* 防範 Bot、編目程式和掃描器
* 偵測一般應用程式錯誤組態 (也就是 Apache、IIS 等)

如需更詳細的規則清單及其保護功能，請參閱下列[核心規則集](#core-rule-sets)。

### <a name="core-rule-sets"></a>核心規則集

應用程式閘道支援兩個規則集：CRS 3.0 和 CRS 2.2.9。 這些核心規則集是可保護 Web 應用程式免於遭遇惡意活動的規則集合。

#### <a name="owasp30"></a>OWASP_3.0

所提供的 3.0 核心規則集有 13 個規則群組，如下表所示。 每個規則群組包含多個規則 (可加以停用)。

|RuleGroup|說明|
|---|---|
|**[REQUEST-911-METHOD-ENFORCEMENT](application-gateway-crs-rulegroups-rules.md#crs911)**|包含規則，可鎖定方法 (PUT、PPATCH< ..)|
|**[REQUEST-913-SCANNER-DETECTION](application-gateway-crs-rulegroups-rules.md#crs913)**| 包含規則，可防禦連接埠和環境掃描器。|
|**[REQUEST-920-PROTOCOL-ENFORCEMENT](application-gateway-crs-rulegroups-rules.md#crs920)**|包含規則，可防範通訊協定和編碼問題。|
|**[REQUEST-921-PROTOCOL-ATTACK](application-gateway-crs-rulegroups-rules.md#crs921)**|包含規則，可防止標頭插入、要求走私和回應分割|
|**[REQUEST-930-APPLICATION-ATTACK-LFI](application-gateway-crs-rulegroups-rules.md#crs930)**|包含規則，可防禦檔案和路徑攻擊。|
|**[REQUEST-931-APPLICATION-ATTACK-RFI](application-gateway-crs-rulegroups-rules.md#crs931)**|包含規則，可防範遠端檔案引入 (RFI)|
|**[REQUEST-932-APPLICATION-ATTACK-RCE](application-gateway-crs-rulegroups-rules.md#crs932)**|包含規則，可防止遠端執行程式碼。|
|**[REQUEST-933-APPLICATION-ATTACK-PHP](application-gateway-crs-rulegroups-rules.md#crs933)**|包含規則，可防禦 PHP 插入式攻擊。|
|**[REQUEST-941-APPLICATION-ATTACK-XSS](application-gateway-crs-rulegroups-rules.md#crs941)**|包含規則，可防範跨網站指令碼。|
|**[REQUEST-942-APPLICATION-ATTACK-SQLI](application-gateway-crs-rulegroups-rules.md#crs942)**|包含規則，可防禦 SQL 插入式攻擊。|
|**[REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION](application-gateway-crs-rulegroups-rules.md#crs943)**|包含規則，可防禦工作階段固定攻擊。|

#### <a name="owasp229"></a>OWASP_2.2.9

所提供的 2.2.9 核心規則集有 10 個規則群組，如下表所示。 每個規則群組包含多個規則 (可加以停用)。

|RuleGroup|說明|
|---|---|
|**[crs_20_protocol_violations](application-gateway-crs-rulegroups-rules.md#crs20)**|包含規則，可防範通訊協定違規 (無效的字元，使用 GET 搭配要求本文等等)|
|**[crs_21_protocol_anomalies](application-gateway-crs-rulegroups-rules.md#crs21)**|包含規則，可防範不正確的標頭資訊。|
|**[crs_23_request_limits](application-gateway-crs-rulegroups-rules.md#crs23)**|包含規則，可防範超出限制的引數或檔案。|
|**[crs_30_http_policy](application-gateway-crs-rulegroups-rules.md#crs30)**|包含規則，可防範限制的方法、標頭和檔案類型。 |
|**[crs_35_bad_robots](application-gateway-crs-rulegroups-rules.md#crs35)**|包含規則，可防禦 Web 編目程式和掃描器。|
|**[crs_40_generic_attacks](application-gateway-crs-rulegroups-rules.md#crs40)**|包含規則，可防禦一般攻擊 (工作階段固定、遠端檔案引入、PHP 插入等等)|
|**[crs_41_sql_injection_attacks](application-gateway-crs-rulegroups-rules.md#crs41sql)**|包含規則，可防禦 SQL 插入式攻擊。|
|**[crs_41_xss_attacks](application-gateway-crs-rulegroups-rules.md#crs41xss)**|包含規則，可防範跨網站指令碼。|
|**[crs_42_tight_security](application-gateway-crs-rulegroups-rules.md#crs42)**|包含規則，可防禦路徑周遊攻擊|
|**[crs_45_trojans](application-gateway-crs-rulegroups-rules.md#crs45)**|包含規則，可防禦特洛伊木馬後門程式。|

### <a name="waf-modes"></a>WAF 模式

應用程式閘道 WAF 可以設定為在下列兩種模式中執行︰

* **偵測模式** – 當設定為在偵測模式中執行時，應用程式閘道 WAF 會監視所有威脅警示並記錄到記錄檔。 應使用 [診斷] 區段開啟應用程式閘道的記錄診斷。 您也必須確保已選取並開啟 WAF 記錄檔。 在偵測模式執行的 Web 應用程式防火牆不會封鎖連入要求。
* **防止模式** – 當設定為在防止模式中執行時，應用程式閘道會主動封鎖其規則偵測到的入侵和攻擊。 攻擊者會收到 403 未經授權存取例外狀況，且連線會終止。 防止模式會繼續將這類攻擊記錄在 WAF 記錄檔中。

### <a name="application-gateway-waf-reports"></a>WAF 監視

監視您應用程式閘道的健康狀態非常重要。 透過記錄並與 Azure 監視器、Azure 資訊安全中心 (即將推出) 和 Log Analytics 整合，以監視 Web 應用程式防火牆及其保護之應用程式的健康狀態。

![診斷](./media/waf-overview/diagnostics.png)

#### <a name="azure-monitor"></a>Azure 監視器

每個應用程式閘道記錄都會與 [Azure 監視器](../monitoring-and-diagnostics/monitoring-overview.md)整合。  這可讓您追蹤包括 WAF 警示和記錄的診斷資訊。  這項功能提供於入口網站的 [診斷] 索引標籤之下的應用程式閘道資源內，或直接透過 Azure 監視器服務提供。 若要深入了解如何啟用應用程式閘道的診斷記錄，請瀏覽[應用程式閘道診斷](application-gateway-diagnostics.md)。

#### <a name="azure-security-center"></a>Azure 資訊安全中心

[Azure 資訊安全中心](../security-center/security-center-intro.md)利用加強對您 Azure 資源的能見度及安全性控制權，來協助您預防、偵測及回應威脅。 應用程式閘道現在會[整合到 Azure 資訊安全中心](application-gateway-integration-security-center.md)。 Azure 資訊安全中心會掃描您的環境，以偵測未受保護的 Web 應用程式。 它現在可以建議應用程式閘道 WAF 保護這些易受攻擊的資源。 您可以直接從 Azure 資訊安全中心建立應用程式閘道 WAF。  這些 WAF 執行個體已與 Azure 資訊安全中心整合，並且會將警示和健康情況資訊傳回 Azure 資訊安全中心進行報告。

![圖 1](./media/waf-overview/figure1.png)

#### <a name="logging"></a>記錄

應用程式閘道 WAF 提供其偵測到之每個威脅的詳細報告。 記錄會與 Azure 診斷記錄整合，而且警示會以 JSON 格式來記錄。 這些記錄可以與 [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md) 整合。

![imageURLroute](./media/waf-overview/waf2.png)

```json
{
  "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupId}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{appGatewayName}",
  "operationName": "ApplicationGatewayFirewall",
  "time": "2017-03-20T15:52:09.1494499Z",
  "category": "ApplicationGatewayFirewallLog",
  "properties": {
    "instanceId": "ApplicationGatewayRole_IN_0",
    "clientIp": "104.210.252.3",
    "clientPort": "4835",
    "requestUri": "/?a=%3Cscript%3Ealert(%22Hello%22);%3C/script%3E",
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "ruleId": "941320",
    "message": "Possible XSS Attack Detected - HTML Tag Handler",
    "action": "Blocked",
    "site": "Global",
    "details": {
      "message": "Warning. Pattern match \"<(a|abbr|acronym|address|applet|area|audioscope|b|base|basefront|bdo|bgsound|big|blackface|blink|blockquote|body|bq|br|button|caption|center|cite|code|col|colgroup|comment|dd|del|dfn|dir|div|dl|dt|em|embed|fieldset|fn|font|form|frame|frameset|h1|head|h ...\" at ARGS:a.",
      "data": "Matched Data: <script> found within ARGS:a: <script>alert(\\x22hello\\x22);</script>",
      "file": "rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf",
      "line": "865"
    }
  }
} 

```

## <a name="application-gateway-waf-sku-pricing"></a>應用程式閘道 WAF SKU 價格

只有在新的 WAF SKU 之下才能使用 Web 應用程式防火牆。 此 SKU 只適用於 Azure Resource Manager 佈建模型，而不適用於傳統部署模型。 此外 WAF SKU 僅以中型和大型應用程式閘道執行個體大小提供。 應用程式閘道的所有限制也適用於 WAF SKU。 價格是以每小時的閘道器執行個體費用和資料處理費用為基礎。 WAF SKU 的每小時閘道價格不同於標準 SKU 費用，您可在[應用程式閘道價格詳細資料](https://azure.microsoft.com/pricing/details/application-gateway/)找到。 資料處理費用維持不變。 不會依照每個規則或規則群組計算費用。 您可以保護相同 Web 應用程式防火牆後面的多個 Web 應用程式，因此支援多個應用程式不需額外付費。 

## <a name="next-steps"></a>後續步驟

深入了解 WAF 的功能之後，請瀏覽[如何在應用程式閘道上設定 Web 應用程式防火牆](tutorial-restrict-web-traffic-powershell.md)。

