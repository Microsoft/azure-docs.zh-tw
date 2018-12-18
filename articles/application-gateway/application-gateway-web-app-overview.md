---
title: 採用 Azure 應用程式閘道的多租用戶後端概觀
description: 本頁面提供多租用戶後端的應用程式閘道支援概觀。
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 8/1/2018
ms.author: victorh
ms.openlocfilehash: c0084580a2e4860f24aecd37232f38da2e55ccc8
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578427"
---
# <a name="application-gateway-support-for-multi-tenant-back-ends"></a>多租用戶後端的應用程式閘道支援

Azure 應用程式閘道支援以虛擬機器擴展集、網路介面、公用/私人 IP 或完整的網域名稱 (FQDN) 作為其後端集區的一部分。 根據預設，應用程式閘道不會變更來自用戶端的連入 HTTP 主機標頭，而會將標頭原封不動地傳送回後端。 有許多服務 (像是 [Azure Web Apps](../app-service/app-service-web-overview.md)) 本質上為多租用戶，且依賴特定主機標頭或 SNI 擴充功能來解析為正確的端點。 應用程式閘道現在支援使用者根據後端 HTTP 設定覆寫傳入 HTTP 主機標頭的能力。 這項功能可支援多租用戶後端進行 Azure Web 應用程式和 API 管理。 這項功能同時適用於標準和 WAF SKU。 多租用戶後端支援也適用於 SSL 終止和端對端 SSL 案例。

> [!NOTE]
> 受信任的 Azure 服務 (例如 Azure Web Apps) 不需進行驗證憑證設定。

![Web 應用程式案例](./media/application-gateway-web-app-overview/scenario.png)

指定主機覆寫的功能定義於 HTTP 設定，並可在規則建立期間套用至任何後端集區的功能。 多租用戶後端支援下列兩種覆寫主機標頭和 SNI 擴充功能的方法。

1. 將主機名稱設定為 HTTP 設定中固定值的功能。 對於套用 HTTP 設定之後端集區的所有流量，這項功能可確保主機標頭覆寫為此值。 使用端對端 SSL 時，這個覆寫的主機名稱會使用於 SNI 擴充功能。 這項功能可促成以下案例：其中的後端集區伺服器陣列預期會有不同於連入客戶主機標頭的主機標頭。

2. 從後端集區成員的 IP 或 FQDN 衍生主機名稱的功能。 如果使用從個別後端集區成員衍生主機名稱的選項進行設定，HTTP 設定也會提供從後端集區成員的 FQDN 中挑選主機名稱的選項。 使用端對端 SSL 時，此主機名稱衍生自 FQDN 且使用於 SNI 擴充功能。 這項功能可促成以下案例：其中的後端集區可以有兩個或多個多租用戶 PaaS 服務，例如 Azure Web 應用程式，而且每個成員的要求主機標頭包含衍生自其 FQDN 的主機名稱。

> [!NOTE]
> 在上述兩個情況下，設定只會影響即時流量行為，而不會影響健康情況探查行為。 自訂探查已支援在探查組態中指定主機標頭的功能。 自訂探查現在也支援從目前設定的 HTTP 設定衍生主機標頭行為的功能。 在探查組態中使用 `PickHostNameFromBackendHttpSettings` 參數即可指定此組態。 若要讓端對端功能運作，必須修改探查和 HTTP 設定，以反映正確的組態。

使用這項功能，客戶可將 HTTP 設定中的選項和自訂探查指定為適當的組態。 這項設定會接著使用規則繫結至接聽程式和後端集區。

## <a name="next-steps"></a>後續步驟

了解如何設定以 Web 應用程式作為後端集區成員的應用程式閘道，請瀏覽：[透過應用程式閘道設定 App Service Web Apps](application-gateway-web-app-powershell.md)
