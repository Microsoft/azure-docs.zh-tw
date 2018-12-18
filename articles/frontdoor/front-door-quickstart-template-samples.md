---
title: Azure Resource Manager 範本範例 - Azure Front Door Service | Microsoft Docs
description: Azure Front Door Service 的 Azure Resource Manager 範本範例
services: frontdoor
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/13/2018
ms.author: sharadag
ms.openlocfilehash: 1e4571edb15bf1a06948e720e51c672f890d68b6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959509"
---
# <a name="azure-resource-manager-deployment-model-templates-for-front-door"></a>Front Door 的 Azure Resource Manager 部署模型範本

下表包含 Azure Front Door Service 的 Azure Resource Manager 部署模型範本連結。 

| | |
| ---| ---|
| [建立基本 Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-create-basic)| 建立具有單一後端的基本 Front Door 設定。 |
| [建立具有多個後端和後端集區及 URL 型路由的 Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-create-multiple-backends)| 建立已根據 URL 路徑為後端集區中及跨後端集區的多個後端設定負載平衡的 Front Door。 |
| [使用 Front Door 以 HTTPS (Front Door 受控憑證) 讓自訂網域上線](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-custom-domain)| 將自訂網域新增至您的 Front Door，並使用透過 DigiCert 產生的 Front Door 受控憑證為它啟用 HTTPS 流量。 |
| [建立具有地區篩選功能的 Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering)| 建立允許/封鎖來自特定國家/地區之流量的 Front Door。 |
| [在 Front Door 上控制後端的健康狀態探查](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-health-probes)| 藉由更新探查路徑及探查的傳送間隔，來更新 Front Door 以變更健康狀態探查設定。 |
| [建立具有作用中/待命後端設定的 Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-priority-lb)| 建立示範「作用中/待命」應用程式拓撲之優先順序型路由的 Front Door，也就是預設會將所有流量傳送至主要 (最高優先順序) 後端，直到它變成無法使用為止。 |
| [建立已針對特定路由啟用快取功能的 Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-create-caching)| 建立已針對所定義路由設定啟用快取功能的 Front Door，藉此為您的工作負載快取任何靜態資產。 |
| [為 Front Door 主機名稱設定工作階段親和性](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-session-affinity) | 更新 Front Door 來為您的前端主機啟用工作階段親和性，藉此將來自相同使用者工作階段的後續流量傳送至相同的後端。 |
| [使用自訂 SSL 憑證 (適用於 HTTPS) 讓 Front Door 的自訂網域上線](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-custom-byoc) | 使用自訂 SSL 憑證讓 Front Door 的自訂網域上線，也稱為「自攜憑證」案例。 |
| [設定 Front Door 以將用戶端 IP 列入白名單或黑名單](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-clientip)| 使用運用用戶端 IP 位址的自訂存取控制，設定讓 Front Door 限制來自特定用戶端 IP 的流量。 |
| [使用特定 http 參數來設定讓 Front Door 採取動作](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-http-params)| 藉由使用運用 http 參數的自訂存取控制規則，設定讓 Front Door 根據連入要求中的 http 參數來允許或封鎖特定流量。 |
| [設定 Front Door 速率限制](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-rate-limiting)| 設定讓 Front Door 限制所指定前端主機的連入流量速率。 |
| | |

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。