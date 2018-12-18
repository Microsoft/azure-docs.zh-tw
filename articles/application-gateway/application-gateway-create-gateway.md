---
title: 建立、啟動或刪除應用程式閘道 | Microsoft Docs
description: 本頁面提供建立、設定、啟動和刪除 Azure 應用程式閘道的指示。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.assetid: 577054ca-8368-4fbf-8d53-a813f29dc3bc
ms.service: application-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 07/31/2017
ms.author: victorh
ms.openlocfilehash: a1cfd42f5b1c31b911005b2539047630c6d320dd
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32771184"
---
# <a name="create-start-or-delete-an-application-gateway-with-powershell"></a>使用 PowerShell 建立、啟動或刪除應用程式閘道 

> [!div class="op_single_selector"]
> * [Azure 入口網站](application-gateway-create-gateway-portal.md)
> * [Azure Resource Manager PowerShell](application-gateway-create-gateway-arm.md)
> * [Azure 傳統 PowerShell](application-gateway-create-gateway.md)
> * [Azure Resource Manager 範本](application-gateway-create-gateway-arm-template.md)
> * [Azure CLI](application-gateway-create-gateway-cli.md)

Azure 應用程式閘道是第 7 層負載平衡器。 不論是在雲端或內部部署中，此閘道均提供在不同伺服器之間進行容錯移轉及效能路由傳送 HTTP 要求。 應用程式閘道提供許多應用程式傳遞控制器 (ADC) 功能，包括 HTTP 負載平衡、以 Cookie 為基礎的工作階段同質性、安全通訊端層 (SSL) 卸載、自訂健全狀態探查、多網站支援，以及許多其他功能。 若要尋找完整的支援功能清單，請瀏覽 [應用程式閘道概觀](application-gateway-introduction.md)

本文將逐步引導您完成建立、設定、啟動及刪除應用程式閘道的步驟。

## <a name="before-you-begin"></a>開始之前

1. 使用 Web Platform Installer 安裝最新版的 Azure PowerShell Cmdlet。 您可以從 **下載頁面** 的 [Windows PowerShell](https://azure.microsoft.com/downloads/)區段下載並安裝最新版本。
2. 如果您有現有的虛擬網路，請選取現有的空白子網路，或在現有的虛擬網路中建立新的子網路，僅供應用程式閘道使用。 除非使用 VNet 對等互連，否則，您無法將應用程式閘道部署到與您打算部署於應用程式閘道後方的資源不同的虛擬網路。 如需深入了解，請參閱 [VNet 對等互連](../virtual-network/virtual-network-peering-overview.md)
3. 請確認您的運作中虛擬網路具有有效子網路。 請確定沒有虛擬機器或是雲端部署正在使用子網路。 應用程式閘道必須單獨位於虛擬網路子網路中。
4. 您要設定來使用應用程式閘道的伺服器必須存在，或是在虛擬網路中建立其端點，或是已指派公用 IP/VIP。

## <a name="what-is-required-to-create-an-application-gateway"></a>建立應用程式閘道需要什麼？

當您使用 `New-AzureApplicationGateway` 命令來建立應用程式閘道時，此時不需進行任何設定，並使用 XML 或組態物件來設定新建立的資源。

值如下：

* **後端伺服器集區：** 後端伺服器的 IP 位址清單。 列出的 IP 位址應屬於虛擬網路子網路或是公用 IP/VIP。
* **後端伺服器集區設定：** 每個集區都包括一些設定，例如連接埠、通訊協定和以 Cookie 為基礎的同質性。 這些設定會繫結至集區，並套用至集區內所有伺服器。
* **前端連接埠：** 此連接埠是在應用程式閘道上開啟的公用連接埠。 流量會達到此連接埠，然後重新導向至其中一個後端伺服器。
* **接聽程式：** 接聽程式具有前端連接埠、通訊協定 (Http 或 Https，這些值都區分大小寫) 和 SSL 憑證名稱 (如果已設定 SSL 卸載)。
* **規則：** 規則會繫結接聽程式和後端伺服器集區，並定義當流量到達特定接聽程式時，應該導向到哪個後端伺服器集區。

## <a name="create-an-application-gateway"></a>建立應用程式閘道

建立應用程式閘道：

1. 建立應用程式閘道資源。
2. 建立設定 XML 檔案或設定物件。
3. 認可新建立應用程式閘道資源的設定。

> [!NOTE]
> 如果您需要設定應用程式閘道的自訂探查，請參閱 [使用 PowerShell 建立具有自訂探查的應用程式閘道](application-gateway-create-probe-classic-ps.md)。 請參閱 [自訂探查和健全狀況監視](application-gateway-probe-overview.md) 以取得詳細資訊。

![案例範例][scenario]

### <a name="create-an-application-gateway-resource"></a>建立應用程式閘道資源

若要建立閘道，請使用 `New-AzureApplicationGateway` Cmdlet，並以您自己的值來取代這些值。 此時還不會開始對閘道計費。 會在稍後的步驟中於成功啟動閘道之後開始計費。

下列範例會使用名為 "testvnet1" 的虛擬網路和名為 "subnet-1" 的子網路來建立應用程式閘道：

```powershell
New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")
```

Description、InstanceCount 和 GatewaySize 為選擇性參數。

若要驗證已建立閘道，您可以使用 `Get-AzureApplicationGateway` Cmdlet。

```powershell
Get-AzureApplicationGateway AppGwTest
```

```
Name          : AppGwTest
Description   :
VnetName      : testvnet1
Subnets       : {Subnet-1}
InstanceCount : 2
GatewaySize   : Medium
State         : Stopped
VirtualIPs    : {}
DnsName       :
```

> [!NOTE]
> *InstanceCount* 的預設值是 2，且最大值是 10。 *GatewaySize* 的預設值是 Medium。 您可以選擇 Small、Medium 和 Large。

因為尚未啟動閘道，所以 VirtualIPs 和 DnsName 會顯示為空白。 閘道處於執行中狀態之後，就會建立這些項目。

## <a name="configure-the-application-gateway"></a>設定應用程式閘道

您可以使用 XML 或設定物件來設定應用程式閘道。

### <a name="configure-the-application-gateway-by-using-xml"></a>使用 XML 設定應用程式閘道

在下列範例中，您將使用 XML 檔案來設定所有應用程式閘道設定，並將它們認可到應用程式閘道資源。  

#### <a name="step-1"></a>步驟 1

將下列文字複製到 [記事本]。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
    <FrontendPorts>
        <FrontendPort>
            <Name>(name-of-your-frontend-port)</Name>
            <Port>(port number)</Port>
        </FrontendPort>
    </FrontendPorts>
    <BackendAddressPools>
        <BackendAddressPool>
            <Name>(name-of-your-backend-pool)</Name>
            <IPAddresses>
                <IPAddress>(your-IP-address-for-backend-pool)</IPAddress>
                <IPAddress>(your-second-IP-address-for-backend-pool)</IPAddress>
            </IPAddresses>
        </BackendAddressPool>
    </BackendAddressPools>
    <BackendHttpSettingsList>
        <BackendHttpSettings>
            <Name>(backend-setting-name-to-configure-rule)</Name>
            <Port>80</Port>
            <Protocol>[Http|Https]</Protocol>
            <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        </BackendHttpSettings>
    </BackendHttpSettingsList>
    <HttpListeners>
        <HttpListener>
            <Name>(name-of-the-listener)</Name>
            <FrontendPort>(name-of-your-frontend-port)</FrontendPort>
            <Protocol>[Http|Https]</Protocol>
        </HttpListener>
    </HttpListeners>
    <HttpLoadBalancingRules>
        <HttpLoadBalancingRule>
            <Name>(name-of-load-balancing-rule)</Name>
            <Type>basic</Type>
            <BackendHttpSettings>(backend-setting-name-to-configure-rule)</BackendHttpSettings>
            <Listener>(name-of-the-listener)</Listener>
            <BackendAddressPool>(name-of-your-backend-pool)</BackendAddressPool>
        </HttpLoadBalancingRule>
    </HttpLoadBalancingRules>
</ApplicationGatewayConfiguration>
```

編輯設定項目括號間的值。 以 .xml 副檔名儲存檔案。

> [!IMPORTANT]
> 通訊協定項目 Http 或 Https 會區分大小寫。

下列範例示範如何使用組態檔來設定應用程式閘道。 此範例會平衡公用連接埠 80 上的 HTTP 流量負載，並將網路流量傳送至兩個 IP 位址之間的後端連接埠 80。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationGatewayConfiguration xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
    <FrontendPorts>
        <FrontendPort>
            <Name>FrontendPort1</Name>
            <Port>80</Port>
        </FrontendPort>
    </FrontendPorts>
    <BackendAddressPools>
        <BackendAddressPool>
            <Name>BackendPool1</Name>
            <IPAddresses>
                <IPAddress>10.0.0.1</IPAddress>
                <IPAddress>10.0.0.2</IPAddress>
            </IPAddresses>
        </BackendAddressPool>
    </BackendAddressPools>
    <BackendHttpSettingsList>
        <BackendHttpSettings>
            <Name>BackendSetting1</Name>
            <Port>80</Port>
            <Protocol>Http</Protocol>
            <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        </BackendHttpSettings>
    </BackendHttpSettingsList>
    <HttpListeners>
        <HttpListener>
            <Name>HTTPListener1</Name>
            <FrontendPort>FrontendPort1</FrontendPort>
            <Protocol>Http</Protocol>
        </HttpListener>
    </HttpListeners>
    <HttpLoadBalancingRules>
        <HttpLoadBalancingRule>
            <Name>HttpLBRule1</Name>
            <Type>basic</Type>
            <BackendHttpSettings>BackendSetting1</BackendHttpSettings>
            <Listener>HTTPListener1</Listener>
            <BackendAddressPool>BackendPool1</BackendAddressPool>
        </HttpLoadBalancingRule>
    </HttpLoadBalancingRules>
</ApplicationGatewayConfiguration>
```

#### <a name="step-2"></a>步驟 2

接下來，設定應用程式閘道。 使用 `Set-AzureApplicationGatewayConfig` Cmdlet 搭配組態 XML 檔案。

```powershell
Set-AzureApplicationGatewayConfig -Name AppGwTest -ConfigFile "D:\config.xml"
```

### <a name="configure-the-application-gateway-by-using-a-configuration-object"></a>使用設定物件設定應用程式閘道

下列範例示範如何使用設定物件來設定應用程式閘道。 必須個別設定所有組態項目，然後再將其新增至應用程式閘道組態物件。 建立組態物件之後，您會使用 `Set-AzureApplicationGateway` 命令，將組態認可到先前建立的應用程式閘道資源。

> [!NOTE]
> 在將值指派到各個設定物件之前，您必須宣告 PowerShell 要用來儲存的物件種類。 建立個別項目的第一行會定義要使用哪些 `Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model(object name)`。

#### <a name="step-1"></a>步驟 1

建立所有的個別設定項目。

建立前端 IP，如下列範例所示。

```powershell
$fip = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.FrontendIPConfiguration
$fip.Name = "fip1"
$fip.Type = "Private"
$fip.StaticIPAddress = "10.0.0.5"
```

建立前端連接埠，如下列範例所示。

```powershell
$fep = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.FrontendPort
$fep.Name = "fep1"
$fep.Port = 80
```

建立後端伺服器集區。

定義加入至後端伺服器集區的 IP 位址，如下列範例所示。

```powershell
$servers = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.BackendServerCollection
$servers.Add("10.0.0.1")
$servers.Add("10.0.0.2")
```

使用 $server 物件，將值加入至後端集區物件 ($pool)。

```powershell
$pool = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.BackendAddressPool
$pool.BackendServers = $servers
$pool.Name = "pool1"
```

建立後端伺服器集區設定。

```powershell
$setting = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.BackendHttpSettings
$setting.Name = "setting1"
$setting.CookieBasedAffinity = "enabled"
$setting.Port = 80
$setting.Protocol = "http"
```

建立接聽程式。

```powershell
$listener = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.HttpListener
$listener.Name = "listener1"
$listener.FrontendPort = "fep1"
$listener.FrontendIP = "fip1"
$listener.Protocol = "http"
$listener.SslCert = ""
```

建立規則。

```powershell
$rule = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.HttpLoadBalancingRule
$rule.Name = "rule1"
$rule.Type = "basic"
$rule.BackendHttpSettings = "setting1"
$rule.Listener = "listener1"
$rule.BackendAddressPool = "pool1"
```

#### <a name="step-2"></a>步驟 2

將所有的個別設定項目指派給應用程式閘道設定物件 ($appgwconfig)。

將前端 IP 加入設定。

```powershell
$appgwconfig = New-Object Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.ApplicationGatewayConfiguration
$appgwconfig.FrontendIPConfigurations = New-Object "System.Collections.Generic.List[Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.FrontendIPConfiguration]"
$appgwconfig.FrontendIPConfigurations.Add($fip)
```

將前端連接埠加入設定。

```powershell
$appgwconfig.FrontendPorts = New-Object "System.Collections.Generic.List[Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.FrontendPort]"
$appgwconfig.FrontendPorts.Add($fep)
```
將後端伺服器集區加入設定。

```powershell
$appgwconfig.BackendAddressPools = New-Object "System.Collections.Generic.List[Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.BackendAddressPool]"
$appgwconfig.BackendAddressPools.Add($pool)
```

將後端集區設定加入設定。

```powershell
$appgwconfig.BackendHttpSettingsList = New-Object "System.Collections.Generic.List[Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.BackendHttpSettings]"
$appgwconfig.BackendHttpSettingsList.Add($setting)
```

將接聽程式加入設定。

```powershell
$appgwconfig.HttpListeners = New-Object "System.Collections.Generic.List[Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.HttpListener]"
$appgwconfig.HttpListeners.Add($listener)
```

將規則加入設定。

```powershell
$appgwconfig.HttpLoadBalancingRules = New-Object "System.Collections.Generic.List[Microsoft.WindowsAzure.Commands.ServiceManagement.Network.ApplicationGateway.Model.HttpLoadBalancingRule]"
$appgwconfig.HttpLoadBalancingRules.Add($rule)
```

### <a name="step-3"></a>步驟 3
使用 `Set-AzureApplicationGatewayConfig`，將組態物件認可到應用程式閘道資源。

```powershell
Set-AzureApplicationGatewayConfig -Name AppGwTest -Config $appgwconfig
```

## <a name="start-the-gateway"></a>啟動閘道

設定閘道之後，請使用 `Start-AzureApplicationGateway` Cmdlet 來啟動閘道。 成功啟動閘道之後，會開始應用程式閘道計費。

> [!NOTE]
> `Start-AzureApplicationGateway` Cmdlet 最多可能需要 15 到 20 分鐘才能完成。

```powershell
Start-AzureApplicationGateway AppGwTest
```

## <a name="verify-the-gateway-status"></a>確認閘道狀態

使用 `Get-AzureApplicationGateway` Cmdlet 檢查閘道狀態。 如果上一個步驟中的 `Start-AzureApplicationGateway` 成功，則 *State* 應該是 Running，而且 *Vip* 和 *DnsName* 應該具備有效的輸入。

下列範例示範已啟動、正在執行且準備好將流量傳送到 `http://<generated-dns-name>.cloudapp.net`的應用程式閘道。

```powershell
Get-AzureApplicationGateway AppGwTest
```

```
VERBOSE: 8:09:28 PM - Begin Operation: Get-AzureApplicationGateway
VERBOSE: 8:09:30 PM - Completed Operation: Get-AzureApplicationGateway
Name          : AppGwTest
Description   :
VnetName      : testvnet1
Subnets       : {Subnet-1}
InstanceCount : 2
GatewaySize   : Medium
State         : Running
Vip           : 138.91.170.26
DnsName       : appgw-1b8402e8-3e0d-428d-b661-289c16c82101.cloudapp.net
```

## <a name="delete-the-application-gateway"></a>刪除應用程式閘道

若要刪除應用程式閘道：

1. 使用 `Stop-AzureApplicationGateway` Cmdlet 停止閘道。
2. 使用 `Remove-AzureApplicationGateway` Cmdlet 移除閘道。
3. 使用 `Get-AzureApplicationGateway` Cmdlet 確認已移除閘道。

下列範例會在第一行顯示 `Stop-AzureApplicationGateway` Cmdlet，後面接著輸出。

```powershell
Stop-AzureApplicationGateway AppGwTest
```

```
VERBOSE: 9:49:34 PM - Begin Operation: Stop-AzureApplicationGateway
VERBOSE: 10:10:06 PM - Completed Operation: Stop-AzureApplicationGateway
Name       HTTP Status Code     Operation ID                             Error
----       ----------------     ------------                             ----
Successful OK                   ce6c6c95-77b4-2118-9d65-e29defadffb8
```

當應用程式閘道處於已停止狀態之後，使用 `Remove-AzureApplicationGateway` Cmdlet 來移除服務。

```powershell
Remove-AzureApplicationGateway AppGwTest
```

```
VERBOSE: 10:49:34 PM - Begin Operation: Remove-AzureApplicationGateway
VERBOSE: 10:50:36 PM - Completed Operation: Remove-AzureApplicationGateway
Name       HTTP Status Code     Operation ID                             Error
----       ----------------     ------------                             ----
Successful OK                   055f3a96-8681-2094-a304-8d9a11ad8301
```

若要確認已移除服務，您可以使用 `Get-AzureApplicationGateway` Cmdlet。 這不是必要步驟。

```powershell
Get-AzureApplicationGateway AppGwTest
```

```
VERBOSE: 10:52:46 PM - Begin Operation: Get-AzureApplicationGateway

Get-AzureApplicationGateway : ResourceNotFound: The gateway does not exist.
.....
```

## <a name="next-steps"></a>後續步驟

如果您想要設定 SSL 卸載，請參閱 [設定應用程式閘道以進行 SSL 卸載](application-gateway-ssl.md)。

如果您想要將應用程式閘道設為與內部負載平衡器搭配使用，請參閱 [建立具有內部負載平衡器 (ILB) 的應用程式閘道](application-gateway-ilb.md)。

如果您想進一步了解一般負載平衡選項，請參閱：

* [Azure 負載平衡器](https://azure.microsoft.com/documentation/services/load-balancer/)
* [Azure 流量管理員](https://azure.microsoft.com/documentation/services/traffic-manager/)

[scenario]: ./media/application-gateway-create-gateway/scenario.png
