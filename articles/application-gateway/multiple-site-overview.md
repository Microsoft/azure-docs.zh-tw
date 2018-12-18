---
title: 在 Azure 應用程式閘道上裝載多個網站 | Microsoft Docs
description: 本頁面提供「應用程式閘道」多站台支援的概觀。
documentationcenter: na
services: application-gateway
author: amsriva
manager: rossort
editor: ''
ms.assetid: 49993fd2-87e5-4a66-b386-8d22056a616d
ms.service: application-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2017
ms.author: amsriva
ms.openlocfilehash: df98559a9476190d683812bf9f63d8ad9c4d3f0e
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32160506"
---
# <a name="application-gateway-multiple-site-hosting"></a>應用程式閘道多站台裝載

多站台裝載可讓您在相同的應用程式閘道執行個體上設定多個 Web 應用程式。 此功能可讓您將最多 20 個網站新增到一個應用程式閘道，為您的部署設定更有效率的拓撲。 每個網站都可以導向到自己的後端集區。 在下列範例中，應用程式閘道會從兩個後端伺服器集區 (名為 ContosoServerPool 和 FabrikamServerPool) 為 contoso.com 和 fabrikam.com 的流量提供服務。

![imageURLroute](./media/multiple-site-overview/multisite.png)

> [!IMPORTANT]
> 規則會依照其列在入口網站中的順序進行處理。 強烈建議纖設定多站台接聽程式，再設定基本接聽程式。  這可確保流量路由傳送到右邊後端。 如果先列出了基本接聽程式，且該接聽程式符合傳入的要求，就會由該接聽程式處理。

對 http://contoso.com 的要求會路由傳送至 ContosoServerPool，而對 http://fabrikam.com 的要求則會路由傳送至 FabrikamServerPool。

同樣地，相同父系網域的兩個子網域也可以裝載在相同的應用程式閘道部署上。 使用子網域的範例可能包括單一應用程式閘道部署上裝載的 http://blog.contoso.com 和 http://app.contoso.com。

## <a name="host-headers-and-server-name-indication-sni"></a>主機標頭和伺服器名稱指示 (SNI)

有三個常見的機制可允許在相同的基礎結構上進行多站台裝載。

1. 將多個 Web 應用程式分別裝載在一個唯一的 IP 位址上。
2. 使用主機名稱將多個 Web 應用程式裝載在相同的 IP 位址上。
3. 使用不同的連接埠將多個 Web 應用程式裝載在相同的 IP 位址上。

目前應用程式閘道會取得它用來接聽流量的單一公用 IP 位址。 因此，目前不支援讓多個應用程式分別擁有自己的 IP 位址。 「應用程式閘道」支援裝載多個分別在不同連接埠上進行接聽的應用程式，但此案例會需要應用程式在非標準連接埠上接受流量，而這樣的組態通常不是理想的組態。 「應用程式閘道」需依賴 HTTP 1.1 主機標頭，才能在相同的公用 IP 位址和連接埠上裝載多個網站。 裝載在應用程式閘道上的網站也可以使用「伺服器名稱指示」(SNI) TLS 擴充功能來支援 SSL 卸載。 此案例表示用戶端瀏覽器和後端 Web 伺服陣列必須支援 RFC 6066 中所定義的 HTTP/1.1 和 TLS 擴充功能。

## <a name="listener-configuration-element"></a>接聽程式組態元素

現有的 HTTPListener 組態元素已增強，可以支援應用程式閘道用來將流量路由傳送至適當後端集區的主機名稱和伺服器名稱指示元素。 下列程式碼範例是來自範本檔案的 HttpListeners 元素程式碼片段。

```json
"httpListeners": [
    {
        "name": "appGatewayHttpsListener1",
        "properties": {
            "FrontendIPConfiguration": {
                "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendIPConfigurations/DefaultFrontendPublicIP"
            },
            "FrontendPort": {
                "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendPorts/appGatewayFrontendPort443'"
            },
            "Protocol": "Https",
            "SslCertificate": {
                "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/sslCertificates/appGatewaySslCert1'"
            },
            "HostName": "contoso.com",
            "RequireServerNameIndication": "true"
        }
    },
    {
        "name": "appGatewayHttpListener2",
        "properties": {
            "FrontendIPConfiguration": {
                "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendIPConfigurations/appGatewayFrontendIP'"
            },
            "FrontendPort": {
                "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendPorts/appGatewayFrontendPort80'"
            },
            "Protocol": "Http",
            "HostName": "fabrikam.com",
            "RequireServerNameIndication": "false"
        }
    }
],
```

您可以瀏覽[使用多站台裝載的 Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/blob/master/201-application-gateway-multihosting)，以了解以範本為基礎的端對端部署。

## <a name="routing-rule"></a>路由規則

無須在路由規則中進行任何變更。 應該繼續選擇路由規則 'Basic'，以將適當的站台接聽程式繫結到對應的後端位址集區。

```json
"requestRoutingRules": [
{
    "name": "<ruleName1>",
    "properties": {
        "RuleType": "Basic",
        "httpListener": {
            "id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/httpListeners/appGatewayHttpsListener1')]"
        },
        "backendAddressPool": {
            "id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/ContosoServerPool')]"
        },
        "backendHttpSettings": {
            "id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
        }
    }

},
{
    "name": "<ruleName2>",
    "properties": {
        "RuleType": "Basic",
        "httpListener": {
            "id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/httpListeners/appGatewayHttpListener2')]"
        },
        "backendAddressPool": {
            "id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/FabrikamServerPool')]"
        },
        "backendHttpSettings": {
            "id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
        }
    }

}
]
```

## <a name="next-steps"></a>後續步驟

了解多站台裝載之後，請移至 [使用多站台裝載建立應用程式閘道](tutorial-multiple-sites-powershell.md) ，以建立能夠支援多個 Web 應用程式的應用程式閘道。

