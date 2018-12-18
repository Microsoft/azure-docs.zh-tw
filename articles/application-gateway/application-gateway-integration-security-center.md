---
title: 應用程式閘道與 Azure 資訊安全中心整合 | Microsoft Docs
description: 此頁面提供如何將應用程式閘道整合到 Azure 資訊安全中心的相關資訊。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
editor: ''
ms.assetid: e5ea5cf9-3b41-4b85-a12c-e758bff7f3ec
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 06/07/2017
ms.author: victorh
ms.openlocfilehash: b3a4abf4d0f408cdb49020d831b50d943c3467dd
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2018
ms.locfileid: "33205038"
---
# <a name="overview-of-integration-between-application-gateway-and-azure-security-center"></a>應用程式閘道與 Azure 資訊安全中心整合概觀

了解如何使用應用程式閘道和資訊安全中心如何保護您的 Web 應用程式資源。 應用程式閘道 Web 應用程式防火牆 (WAF) 與[資訊安全中心](../security-center/security-center-intro.md)整合可提供無落差的檢視，以防止、偵測及回應環境中針對未受保護 Web 應用程式的威脅。

## <a name="overview"></a>概觀

建議在資訊安全中心使用應用程式閘道 WAF，以保護 Web 應用程式議抵禦入侵和弱點。 已啟用 Web 的資源若沒有受到 WAF 的保護，資訊安全中心會建議它是高嚴重性。 使用 Web 應用程式防火牆的建議會出現在 [概觀] 頁面的 [應用程式] 底下。

![與資訊安全中心整合][1]

按一下任何與 Web 應用程式防火牆相關的建議會開啟新的頁面，其中顯示建議的詳細資料。

## <a name="add-a-web-application-firewall-to-an-existing-resource"></a>將 Web 應用程式防火牆新增至現有資源

瀏覽至 [所有服務]  >  [安全性 + 身分識別]  >  [資訊安全中心]，在 [資訊安全中心-概觀] 中按一下 [應用程式]。 在 [資訊安全中心 - 應用程式] 中，資料表包含資訊安全中心在您的訂用帳戶中偵測到的應用程式清單。

![Web 應用程式][3]

按一下有嚴重問題的 Web 應用程式，您會看到 [應用程式安全性健康情況] 頁面。 在下圖中，Web 應用程式沒有 Web 應用程式防火牆的保護。 

![未受保護的 Web 資源][2]

按一下 [建議] 底下的 [新增 Web 應用程式防火牆] 以開啟 [新增 Web 應用程式防火牆] 頁面。

如果您還沒有應用程式閘道或想要建立新的，按一下 [新建]，在 [建立新的 Web 應用程式防火牆] 中，按一下 [Microsoft - 應用程式閘道]。 這會帶您開始建立應用程式閘道的步驟。 此時，您的 Web 應用程式已新增為受保護的資源，資訊安全中心現在會追蹤這個受到 Web 應用程式防火牆保護的資源。 這不會將它新增為後端集區的成員。

如果您有現有的應用程式閘道，您可以在 [使用現有的解決方案] 中選擇它。

![新增 Web 應用程式防火牆頁面][4]

透過資訊安全中心將 Web 應用程式新增至應用程式閘道，不會將此資源新增為後端集區成員。 這必須在應用程式閘道資源上直接設定。

## <a name="add-a-resource-to-an-existing-web-application-firewall"></a>將資源新增至現有 Web 應用程式防火牆

瀏覽至 [所有服務]  >  [安全性 + 身分識別]  >  [資訊安全中心]，在 [資訊安全中心-概觀] 中按一下 [合作夥伴解決方案]。 在 [合作夥伴解決方案] 頁面中會顯示現有的資訊安全中心感知應用程式閘道。

![合作夥伴解決方案][7]

按一下 [連結應用程式] 開啟 [連結應用程式]，您可以在此選取現有的應用程式。 選擇要保護的應用程式，按一下 [確定]。 這不會將 Web 應用程式新增至應用程式閘道的後端集區。 這會將資源設定為受保護的資源，讓資訊安全中心能夠追蹤它。 若要將資源新增為後端集區成員，必須在應用程式閘道上完成，在眼前的頁面中按一下 [解決方案主控台] 會帶您前往此應用程式閘道資源，讓您將 Web 應用程式新增至後端集區。

![合作夥伴解決方案應用程式][6]

## <a name="finalize-configuration"></a>完成設定

資訊安全中心會追蹤已新增至應用程式閘道成為受保護資源的應用程式。  它會監視此資源的健康情況，確保它受到應用程式閘道的保護。 下一個步驟是將虛耗網路的私人 IP、公用 IP 或 NIC 新增至應用程式閘道的後端集區。 若不這麼做，會顯示另一個 [完成應用程式保護] 建議，直到新增資源。

![新增 Web 應用程式防火牆頁面][5]

## <a name="security-alerts"></a>安全性警示

在資訊安全中心內瀏覽至 [偵測]  >  [安全性警示]。  在這裡可以找到您的應用程式閘道的 WAF 警示。 警示是依 WAF 規則細分。

![安全性警示][8]

按一下規則會提供該特定 WAF 規則的警示清單。 每個警示會顯示找到項目的其他詳細資料。 詳細資料會提供連結至應用程式閘道的連結。
 
![警示詳細資料][9]

## <a name="next-steps"></a>後續步驟

若要了解如何在現有應用程式閘道上啟用 Web 應用程式防火牆，請參閱[建立或更新具有 Web 應用程式防火牆的 Azure 應用程式閘道](application-gateway-web-application-firewall-portal.md)。

[1]: ./media/application-gateway-integration-security-center/figure1.png
[2]: ./media/application-gateway-integration-security-center/figure2.png
[3]: ./media/application-gateway-integration-security-center/figure3.png
[4]: ./media/application-gateway-integration-security-center/figure4.png
[5]: ./media/application-gateway-integration-security-center/figure5.png
[6]: ./media/application-gateway-integration-security-center/figure6.png
[7]: ./media/application-gateway-integration-security-center/figure7.png
[8]: ./media/application-gateway-integration-security-center/securitycenter.png
[9]: ./media/application-gateway-integration-security-center/figure9.png