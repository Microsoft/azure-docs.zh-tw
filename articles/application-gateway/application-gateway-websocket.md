---
title: Azure 應用程式閘道的 WebSocket 支援 | Microsoft Docs
description: 本頁面提供應用程式閘道 WebSocket 支援的概觀。
documentationcenter: na
services: application-gateway
author: amsriva
manager: rossort
editor: amsriva
ms.assetid: 8968dac1-e9bc-4fa1-8415-96decacab83f
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/08/2017
ms.author: amsriva
ms.openlocfilehash: 75b06ddd02da231b7813c609c848c75e42116da5
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22986723"
---
# <a name="overview-of-websocket-support-in-application-gateway"></a>應用程式閘道中的 WebSocket 支援概觀

應用程式閘道可跨所有閘道大小對 WebSocket 提供原生支援。 使用者無法進行設定來選擇要啟用或停用 WebSocket 支援。 

以 [RFC6455](https://tools.ietf.org/html/rfc6455) 標準化的 WebSocket 通訊協定可透過長時間執行的 TCP 連線讓伺服器和用戶端之間能夠進行全雙工通訊。 此功能可讓 Web 伺服器和用戶端之間進行互動性更高的通訊，此通訊可以是雙向的，而不需要像 HTTP 型實作所要求的進行輪詢。 不同於 HTTP，WebSocket 的負荷很低，而且可以對多個要求/回應重複使用相同的 TCP 連線，進而提升資源使用效率。 WebSocket 通訊協定設計為透過傳統 HTTP 連接埠 80 和 443 進行運作。

您可以在連接埠 80 或 443 上繼續使用標準 HTTP 接聽程式來接收 WebSocket 流量。 WebSocket 流量接著會使用應用程式閘道規則中指定的適當後端集區，來導向到已啟用 WebSocket 的後端伺服器。 後端伺服器必須回應應用程式閘道探查，[健康情況探查概觀](application-gateway-probe-overview.md)一節中會有所說明。 應用程式閘道健康情況探查僅限使用 HTTP/HTTPS。 每一部後端伺服器都必須回應 HTTP 探查，應用程式閘道才能將 WebSocket 流量路由傳送至伺服器。

## <a name="listener-configuration-element"></a>接聽程式組態元素

現有的 HTTP 接聽程式可用來支援 WebSocket 流量。 以下是來自範例範本檔案中 httpListeners 元素的程式碼片段。 您必須同時擁有 HTTP 和 HTTPS 接聽程式才能支援 WebSocket 和保護 WebSocket 流量。 同樣地，您可以使用[入口網站](application-gateway-create-gateway-portal.md)或 [PowerShell](application-gateway-create-gateway-arm.md) 在連接埠 80/443 上建立具有接聽程式的應用程式閘道，才能支援 WebSocket 流量。

```json
"httpListeners": [
        {
            "name": "appGatewayHttpsListener",
            "properties": {
                "FrontendIPConfiguration": {
                    "Id": "/subscriptions/{subscriptionId/resourceGroups/{resourceGroupName/providers/Microsoft.Network/applicationGateways/{applicationGatewayName/frontendIPConfigurations/DefaultFrontendPublicIP"
                },
                "FrontendPort": {
                    "Id": "/subscriptions/{subscriptionId/resourceGroups/{resourceGroupName/providers/Microsoft.Network/applicationGateways/{applicationGatewayName/frontendPorts/appGatewayFrontendPort443'"
                },
                "Protocol": "Https",
                "SslCertificate": {
                    "Id": "/subscriptions/{subscriptionId/resourceGroups/{resourceGroupName/providers/Microsoft.Network/applicationGateways/{applicationGatewayName/sslCertificates/appGatewaySslCert1'"
                },
            }
        },
        {
            "name": "appGatewayHttpListener",
            "properties": {
                "FrontendIPConfiguration": {
                    "Id": "/subscriptions/{subscriptionId/resourceGroups/{resourceGroupName/providers/Microsoft.Network/applicationGateways/{applicationGatewayName/frontendIPConfigurations/appGatewayFrontendIP'"
                },
                "FrontendPort": {
                    "Id": "/subscriptions/{subscriptionId/resourceGroups/{resourceGroupName/providers/Microsoft.Network/applicationGateways/{applicationGatewayName/frontendPorts/appGatewayFrontendPort80'"
                },
                "Protocol": "Http",
            }
        }
    ],
```

## <a name="backendaddresspool-backendhttpsetting-and-routing-rule-configuration"></a>BackendAddressPool、BackendHttpSetting 和路由規則組態

若後端集區具有已啟用 WebSocket 的伺服器，使用 backendAddressPool 加以定義。 backendHttpSetting 是以後端連接埠 80 和 443 加以定義。 Cookie 型同質性和 requestTimeouts 的屬性與 WebSocket 流量無關。 路由規則不需要任何變更，'Basic' 用於將適當的接聽程式繫結到對應的後端位址集區。 

```json
"requestRoutingRules": [{
    "name": "<ruleName1>",
    "properties": {
        "RuleType": "Basic",
        "httpListener": {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/httpListeners/appGatewayHttpsListener')]"
        },
        "backendAddressPool": {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/ContosoServerPool')]"
        },
        "backendHttpSettings": {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
        }
    }

}, {
    "name": "<ruleName2>",
    "properties": {
        "RuleType": "Basic",
        "httpListener": {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/httpListeners/appGatewayHttpListener')]"
        },
        "backendAddressPool": {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendAddressPools/ContosoServerPool')]"
        },
        "backendHttpSettings": {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/applicationGateways/{applicationGatewayName}/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
        }

    }
}]
```

## <a name="websocket-enabled-backend"></a>已啟用 WebSocket 的後端

後端必須在設定的連接埠 (通常是 80/443) 上執行 HTTP/HTTPS Web 伺服器，WebSocket 才能運作。 此需求是因為 WebSocket 通訊協定要求初始交握必須是 HTTP，並以升級為 WebSocket 通訊協定作為標頭欄位。 以下是標題的範例：

```
    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Origin: http://example.com
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13
```

另一個原因是該應用程式閘道後端健全狀態探查只支援 HTTP 和 HTTPS 通訊協定。 如果後端伺服器未回應 HTTP 或 HTTPS 探查，它被帶離後端集區。

## <a name="next-steps"></a>後續步驟

在了解 WebSocket 支援之後，請移至 [建立應用程式閘道](application-gateway-create-gateway.md) 以開始使用已啟用 WebSocket 的 Web 應用程式。

