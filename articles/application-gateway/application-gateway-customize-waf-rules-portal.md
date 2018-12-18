---
title: 在 Azure 應用程式閘道中自訂 Web 應用程式防火牆規則 - Azure 入口網站 | Microsoft Docs
description: 本文提供如何透過 Azure 入口網站，在應用程式閘道中自訂 Web 應用程式防火牆規則的相關資訊。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.assetid: 1159500b-17ba-41e7-88d6-b96986795084
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 03/28/2017
ms.author: victorh
ms.openlocfilehash: 30df26dc3a9697d3435779f91c32b2d99a747b88
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990462"
---
# <a name="customize-web-application-firewall-rules-through-the-azure-portal"></a>透過 Azure 入口網站自訂 Web 應用程式防火牆規則

> [!div class="op_single_selector"]
> * [Azure 入口網站](application-gateway-customize-waf-rules-portal.md)
> * [PowerShell](application-gateway-customize-waf-rules-powershell.md)
> * [Azure CLI](application-gateway-customize-waf-rules-cli.md)

Azure 應用程式閘道 Web 應用程式防火牆 (WAF) 提供 Web 應用程式的保護。 這些保護是由開放 Web 應用程式安全性專案 (OWASP) 的核心規則集 (CRS) 所提供。 某些規則可能會導致誤判，並封鎖真正的流量。 因此，應用程式閘道會提供功能以自訂規則群組與規則。 如需特定規則群組與規則的詳細資訊，請參閱 [Web 應用程式防火牆 CRS 規則群組與規則的清單](application-gateway-crs-rulegroups-rules.md)。

>[!NOTE]
> 如果您的應用程式閘道並未使用 WAF 層，右窗格中會出現將應用程式閘道升級至 WAF 層的選項。 

![啟用 WAF][fig1]

## <a name="view-rule-groups-and-rules"></a>檢視規則群組與規則

**檢視規則群組與規則**
   1. 瀏覽至應用程式閘道，然後選取 [Web 應用程式防火牆]。  
   2. 選取 [進階規則設定]。  
   這個檢視會在透過所選擇規則集提供之所有規則群組的頁面上顯示一個表格， 並選取所有的規則核取方塊。

![設定已停用的規則][1]

## <a name="search-for-rules-to-disable"></a>搜尋要停用的規則

[Web 應用程式防火牆設定] 刀鋒視窗提供透過文字搜尋篩選規則的功能。 結果只會顯示包含您搜尋之文字的規則群組與規則。

![搜尋規則][2]

## <a name="disable-rule-groups-and-rules"></a>停用規則群組與規則

當您停用規則時，可以停用整個規則群組，或停用一或多個規則群組下的特定規則。 

**停用規則群組或特定規則**

   1. 搜尋您要停用的規則或規則群組。
   2. 清除您要停用之規則的核取方塊。 
   2. 選取 [ **儲存**]。 

![儲存變更][3]

## <a name="next-steps"></a>後續步驟

設定已停用的規則之後，您可以了解如何檢視 WAF 記錄。 如需詳細資訊，請參閱[應用程式閘道診斷](application-gateway-diagnostics.md#diagnostic-logging)。

[fig1]: ./media/application-gateway-customize-waf-rules-portal/1.png
[1]: ./media/application-gateway-customize-waf-rules-portal/figure1.png
[2]: ./media/application-gateway-customize-waf-rules-portal/figure2.png
[3]: ./media/application-gateway-customize-waf-rules-portal/figure3.png
