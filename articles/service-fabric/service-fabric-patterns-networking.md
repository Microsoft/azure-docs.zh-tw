---
title: Azure Service Fabric 的網路功能模式 | Microsoft Docs
description: 描述 Service Fabric 常見的網路功能模式，以及如何使用 Azure 網路功能建立叢集。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/19/2018
ms.author: ryanwi
ms.openlocfilehash: b180e62804b875ca4547a9d09f19efff32ae0cd9
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207218"
---
# <a name="service-fabric-networking-patterns"></a>Service Fabric 網路功能模式
您可以將 Azure Service Fabric 叢集與其他的 Azure 網路功能整合起來。 本文說明如何建立使用下列功能的叢集︰

- [現有虛擬網路或子網路](#existingvnet)
- [靜態公用 IP 位址](#staticpublicip)
- [僅內部負載平衡器](#internallb)
- [內部與外部負載平衡器](#internalexternallb)

Service Fabric 會在標準的虛擬機器擴展集內執行。 能在虛擬機器擴展集內使用的功能，就能在 Service Fabric 叢集內使用。 虛擬機器擴展集和 Service Fabric 之 Azure Resource Manager 範本中的網路區段完全相同。 在部署至現有虛擬網路後，即可輕鬆地納入其他網路功能，例如 Azure ExpressRoute、Azure VPN 閘道、網路安全性群組和虛擬網路對等互連。

Service Fabric 有一個方面是其他網路功能所沒有的。 [Azure 入口網站](https://portal.azure.com)在內部會使用 Service Fabric 資源提供者來呼叫叢集，以取得節點和應用程式的相關資訊。 Service Fabric 資源提供者對管理端點上的 HTTP 閘道連接埠 (預設為連接埠 19080) 需具備可公開存取的輸入存取權。 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 會使用此管理端點來管理您的叢集。 Service Fabric 資源提供者也會使用此連接埠來查詢您叢集的相關資訊，以顯示在 Azure 入口網站中。 

如果無法從 Service Fabric 資源提供者存取連接埠 19080，入口網站中會出現「找不到節點」之類的訊息，而且您的節點和應用程式清單會顯示為空白。 如果您想在 Azure 入口網站看到您的叢集，負載平衡器必須公開公用 IP 位址，且您的網路安全性群組必須允許連入的連接埠 19080 流量。 如果您的設定不符合這些需求，Azure 入口網站就不會顯示叢集的狀態。

## <a name="templates"></a>範本

所有 Service Fabric 範本都位於 [GitHub](https://github.com/Azure/service-fabric-scripts-and-templates/tree/master/templates/networking) 中。 使用下列 Powershell 命令應該可以依原樣部署範本。 如果您要部署現有 Azure 虛擬網路範本或靜態公用 IP 範本，請先閱讀本文的[初始設定](#initialsetup)一節。

<a id="initialsetup"></a>
## <a name="initial-setup"></a>初始設定

### <a name="existing-virtual-network"></a>現有的虛擬網路

在下列範例中，我們會從 **ExistingRG** 資源群組中名為 ExistingRG-vnet 的現有虛擬網路來開始。 子網路名為 default。 這些預設資源會在您使用 Azure 入口網站來建立標準虛擬機器 (VM) 時建立。 您可以建立虛擬網路和子網路而不建立 VM，但將叢集新增至現有虛擬網路的主要目標是要提供對其他 VM 的網路連線能力。 建立 VM 可為現有虛擬網路一般是如何使用的提供良好範例。 如果 Service Fabric 叢集只使用內部負載平衡器而不使用公用 IP 位址，您可以使用 VM 和其公用 IP 來做為安全的「跳躍箱」。

### <a name="static-public-ip-address"></a>靜態公用 IP 位址

靜態公用 IP 位址一般是會與 VM 或其指派到之 VM 分開管理的專用資源。 它會佈建在專用的網路資源群組中 (而非在 Service Fabric 叢集資源群組本身)。 請透過 Azure 入口網站或 Powershell，在同一個 ExistingRG 資源群組中建立名為 staticIP1 的靜態公用 IP 位址：

```powershell
PS C:\Users\user> New-AzureRmPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG -Location westus -AllocationMethod Static -DomainNameLabel sfnetworking

Name                     : staticIP1
ResourceGroupName        : ExistingRG
Location                 : westus
Id                       : /subscriptions/1237f4d2-3dce-1236-ad95-123f764e7123/resourceGroups/ExistingRG/providers/Microsoft.Network/publicIPAddresses/staticIP1
Etag                     : W/"fc8b0c77-1f84-455d-9930-0404ebba1b64"
ResourceGuid             : 77c26c06-c0ae-496c-9231-b1a114e08824
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Static
IpAddress                : 40.83.182.110
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : null
DnsSettings              : {
                             "DomainNameLabel": "sfnetworking",
                             "Fqdn": "sfnetworking.westus.cloudapp.azure.com"
                           }
```

### <a name="service-fabric-template"></a>Service Fabric 範本

在本文的範例中，我們會使用 Service Fabric template.json。 您可以先使用標準入口網站精靈從入口網站下載範本，再建立叢集。 您也可以使用其中一個[範例範本](https://github.com/Azure-Samples/service-fabric-cluster-templates)，例如[五個節點的安全 Service Fabric 叢集](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure)。

<a id="existingvnet"></a>
## <a name="existing-virtual-network-or-subnet"></a>現有虛擬網路或子網路

1. 將子網路參數變更為現有子網路的名稱，然後新增兩個新的參數以參考現有虛擬網路：

    ```
        "subnet0Name": {
                "type": "string",
                "defaultValue": "default"
            },
            "existingVNetRGName": {
                "type": "string",
                "defaultValue": "ExistingRG"
            },

            "existingVNetName": {
                "type": "string",
                "defaultValue": "ExistingRG-vnet"
            },
            /*
            "subnet0Name": {
                "type": "string",
                "defaultValue": "Subnet-0"
            },
            "subnet0Prefix": {
                "type": "string",
                "defaultValue": "10.0.0.0/24"
            },*/
    ```


2. 變更 `vnetID` 變數以指向現有虛擬網路︰

    ```
            /*old "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",*/
            "vnetID": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingVNetRGName'), '/providers/Microsoft.Network/virtualNetworks/', parameters('existingVNetName'))]",
    ```

3. 從資源中移除 `Microsoft.Network/virtualNetworks`，讓 Azure 不會建立新的虛擬網路︰

    ```
    /*{
    "apiVersion": "[variables('vNetApiVersion')]",
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[parameters('virtualNetworkName')]",
    "location": "[parameters('computeLocation')]",
    "properities": {
        "addressSpace": {
            "addressPrefixes": [
                "[parameters('addressPrefix')]"
            ]
        },
        "subnets": [
            {
                "name": "[parameters('subnet0Name')]",
                "properties": {
                    "addressPrefix": "[parameters('subnet0Prefix')]"
                }
            }
        ]
    },
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    }
    },*/
    ```

4. 從 `Microsoft.Compute/virtualMachineScaleSets` 的 `dependsOn` 屬性將虛擬網路註解化，以便不需依賴建立新的虛擬網路︰

    ```
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Computer/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType0Name')]",
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        /*"[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
        */
        "[Concat('Microsoft.Storage/storageAccounts/', variables('uniqueStringArray0')[0])]",

    ```

5. 部署範本：

    ```powershell
    New-AzureRmResourceGroup -Name sfnetworkingexistingvnet -Location westus
    New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingexistingvnet -TemplateFile C:\SFSamples\Final\template\_existingvnet.json
    ```

    在部署之後，您的虛擬網路應該會包含新的擴展集 VM。 虛擬機器擴展集節點類型應該會顯示現有虛擬網路和子網路。 您也可以使用遠端桌面通訊協定 (RDP) 來存取已存在於虛擬網路的 VM，以及 ping 新的擴展集 VM：

    ```
    C:>\Users\users>ping 10.0.0.5 -n 1
    C:>\Users\users>ping NOde1000000 -n 1
    ```

如需其他範例，請參閱[非 Service Fabric 專屬的範例](https://github.com/gbowerman/azure-myriad/tree/master/existing-vnet)。


<a id="staticpublicip"></a>
## <a name="static-public-ip-address"></a>靜態公用 IP 位址

1. 新增現有靜態 IP 資源群組名稱、名稱和完整網域名稱 (FQDN) 的參數︰

    ```
    "existingStaticIPResourceGroup": {
                "type": "string"
            },
            "existingStaticIPName": {
                "type": "string"
            },
            "existingStaticIPDnsFQDN": {
                "type": "string"
    }
    ```

2. 移除 `dnsName` 參數  (靜態 IP 位址已經有一個)。

    ```
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

3. 新增變數來參考現有靜態 IP 位址：

    ```
    "existingStaticIP": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingStaticIPResourceGroup'), '/providers/Microsoft.Network/publicIPAddresses/', parameters('existingStaticIPName'))]",
    ```

4. 從資源中移除 `Microsoft.Network/publicIPAddresses`，讓 Azure 不會建立新的 IP 位址︰

    ```
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

5. 從 `Microsoft.Network/loadBalancers` 的 `dependsOn` 屬性將 IP 位址註解化，以便不需依賴建立新的 IP 位址︰

    ```
    "apiVersion": "[variables('lbIPApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB', '-', parameters('clusterName'), '-', parameters('vmNodeType0Name'))]",
    "location": "[parameters('computeLocation')]",
    /*
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/', concat(parameters('lbIPName'), '-', '0'))]"
    ], */
    "properties": {
    ```

6. 在 `Microsoft.Network/loadBalancers` 資源中，變更 `frontendIPConfigurations` 的 `publicIPAddress` 元素來參考現有靜態 IP 位址，而非參考新建立的位址︰

    ```
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                "publicIPAddress": {
                                    /*"id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"*/
                                    "id": "[variables('existingStaticIP')]"
                                }
                            }
                        }
                    ],
    ```

7. 在 `Microsoft.ServiceFabric/clusters` 資源中，將 `managementEndpoint` 變更為靜態 IP 位址的 DNS FQDN。 如果您使用安全的叢集，請務必將 http:// 變更為 https://  (請注意，此步驟僅適用於 Service Fabric 叢集。 如果您使用虛擬機器擴展集，請略過此步驟)。

    ```
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',parameters('existingStaticIPDnsFQDN'),':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

8. 部署範本：

    ```powershell
    New-AzureRmResourceGroup -Name sfnetworkingstaticip -Location westus

    $staticip = Get-AzureRmPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG

    $staticip

    New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingstaticip -TemplateFile C:\SFSamples\Final\template\_staticip.json -existingStaticIPResourceGroup $staticip.ResourceGroupName -existingStaticIPName $staticip.Name -existingStaticIPDnsFQDN $staticip.DnsSettings.Fqdn
    ```

在部署後，您會看到負載平衡器繫結至另一個資源群組中的公用靜態 IP 位址。 Service Fabric 用戶端連線端點和 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 端點會指向靜態 IP 位址的 DNS FQDN。

<a id="internallb"></a>
## <a name="internal-only-load-balancer"></a>僅內部負載平衡器

此案例使用僅內部負載平衡器取代預設 Service Fabric 範本中的外部負載平衡器。 若要了解這對 Azure 入口網站和 Service Fabric 資源提供者的影響，請參閱上一節。

1. 移除 `dnsName` 參數  (不需要此參數)。

    ```
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

2. (選擇性) 如果您使用靜態配置方法，則可以新增靜態 IP 位址參數。 如果您使用動態配置方法，則不需要執行此步驟。

    ```
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

3. 從資源中移除 `Microsoft.Network/publicIPAddresses`，讓 Azure 不會建立新的 IP 位址︰

    ```
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

4. 從 `Microsoft.Network/loadBalancers` 的 `dependsOn` 屬性中移除 IP 位址，以便不需依賴建立新的 IP 位址。 新增虛擬網路 `dependsOn` 屬性，因為負載平衡器現在仰賴虛擬網路中的子網路︰

    ```
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'))]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /*"[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"*/
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
    ```

5. 將負載平衡器的 `frontendIPConfigurations` 設定從使用 `publicIPAddress` 變更為使用子網路和 `privateIPAddress`。 `privateIPAddress` 使用預先定義的靜態內部 IP 位址。 若要使用動態 IP 位址，請移除 `privateIPAddress` 元素，然後將 `privateIPAllocationMethod` 變更為 **Dynamic**。

    ```
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /*
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                } */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
                            }
                        }
                    ],
    ```

6. 在 `Microsoft.ServiceFabric/clusters` 資源中，變更 `managementEndpoint` 以指向內部負載平衡器位址。 如果您使用安全的叢集，請務必將 http:// **變更為 https://**  (請注意，此步驟僅適用於 Service Fabric 叢集。 如果您使用虛擬機器擴展集，請略過此步驟)。

    ```
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',reference(variables('lbID0')).frontEndIPConfigurations[0].properties.privateIPAddress,':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

7. 部署範本：

    ```powershell
    New-AzureRmResourceGroup -Name sfnetworkinginternallb -Location westus

    New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternallb -TemplateFile C:\SFSamples\Final\template\_internalonlyLB.json
    ```

在部署後，負載平衡器會使用私用靜態 IP 位址 10.0.0.250。 如果您在相同的虛擬網路中有另一部電腦，您可以移至內部 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 端點。 請注意，它會連線至負載平衡器後的其中一個節點。

<a id="internalexternallb"></a>
## <a name="internal-and-external-load-balancer"></a>內部與外部負載平衡器

在此案例中，您會從現有的單一節點類型外部負載平衡器來開始，然後新增同一節點類型的內部負載平衡器。 連結到後端位址集區的後端連接埠只能指派給單一負載平衡器。 選擇要讓哪個負載平衡器擁有您的應用程式連接埠，哪個負載平衡器擁有管理端點 (連接埠 19000 和 19080)。 如果您將管理端點放在內部負載平衡器，請記住本文稍早討論過的 Service Fabric 資源提供者限制。 在我們使用的範例中，管理端點會留在外部負載平衡器。 您也會新增連接埠 80 應用程式連接埠，並將它放在內部負載平衡器。

在雙節點類型的叢集中，一個節點類型位於外部負載平衡器。 另一個節點類型則位於內部負載平衡器。 若要使用雙節點類型的叢集，請在入口網站中建立雙節點類型的範本 (隨附兩個負載平衡器)，並將第二個負載平衡器切換至內部負載平衡器。 如需詳細資訊，請參閱[僅內部負載平衡器](#internallb)一節。

1. 新增靜態內部負載平衡器 IP 位址參數  (如需使用動態 IP 位址的相關注意事項，請參閱本文前面幾節)。

    ```
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

2. 新增應用程式連接埠 80 參數。

3. 若要新增現有網路變數的內部版本，請將變數複製並貼上，然後在名稱中新增「-Int」︰

    ```
    /* Add internal load balancer networking variables */
            "lbID0-Int": "[resourceId('Microsoft.Network/loadBalancers', concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal'))]",
            "lbIPConfig0-Int": "[concat(variables('lbID0-Int'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
            "lbPoolID0-Int": "[concat(variables('lbID0-Int'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
            "lbProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricGatewayProbe')]",
            "lbHttpProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricHttpGatewayProbe')]",
            "lbNatPoolID0-Int": "[concat(variables('lbID0-Int'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
            /* Internal load balancer networking variables end */
    ```

4. 如果您從使用應用程式連接埠 80 的入口網站所產生範本來開始，預設入口網站範本會在外部負載平衡器上新增 AppPort1 (連接埠 80)。 在此情況下，請將 AppPort1 從外部負載平衡器 `loadBalancingRules` 和探查中移除，以將它新增至內部負載平衡器：

    ```
    "loadBalancingRules": [
        {
            "name": "LBHttpRule",
            "properties":{
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[variables('lbHttpProbeID0')]"
                },
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortLBRule1",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('loadBalancedAppPort1')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concate(variables('lbID0'), '/probes/AppPortProbe1')]"
                },
                "protocol": "tcp"
            }
        }*/

    ],
    "probes": [
        {
            "name": "FabricGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricTcpGatewayPort')]",
                "protocol": "tcp"
            }
        },
        {
            "name": "FabricHttpGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricHttpGatewayPort')]",
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortProbe1",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('loadBalancedAppPort1')]",
                "protocol": "tcp"
            }
        } */

    ],
    "inboundNatPools": [
    ```

5. 新增第二個 `Microsoft.Network/loadBalancers` 資源。 此資源類似[僅內部負載平衡器](#internallb)一節中建立的內部負載平衡器，但它會使用「-Int」負載平衡器變數，並只實作應用程式連接埠 80。 這也會移除 `inboundNatPools`，以將 RDP 端點保留在公用負載平衡器上。 如果您想讓 RDP 位於內部負載平衡器上，請將 `inboundNatPools` 從外部負載平衡器移動到這個內部負載平衡器︰

    ```
            /* Add a second load balancer, configured with a static privateIPAddress and the "-Int" load balancer variables. */
            {
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                /* Add "-Internal" to the name. */
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal')]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /* Remove public IP dependsOn, add vnet dependsOn
                    "[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"
                    */
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
                "properties": {
                    "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /* Switch from Public to Private IP address
                                */
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                }
                                */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
                            }
                        }
                    ],
                    "backendAddressPools": [
                        {
                            "name": "LoadBalancerBEAddressPool",
                            "properties": {}
                        }
                    ],
                    "loadBalancingRules": [
                        /* Add the AppPort rule. Be sure to reference the "-Int" versions of backendAddressPool, frontendIPConfiguration, and the probe variables. */
                        {
                            "name": "AppPortLBRule1",
                            "properties": {
                                "backendAddressPool": {
                                    "id": "[variables('lbPoolID0-Int')]"
                                },
                                "backendPort": "[parameters('loadBalancedAppPort1')]",
                                "enableFloatingIP": "false",
                                "frontendIPConfiguration": {
                                    "id": "[variables('lbIPConfig0-Int')]"
                                },
                                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                                "idleTimeoutInMinutes": "5",
                                "probe": {
                                    "id": "[concat(variables('lbID0-Int'),'/probes/AppPortProbe1')]"
                                },
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "probes": [
                    /* Add the probe for the app port. */
                    {
                            "name": "AppPortProbe1",
                            "properties": {
                                "intervalInSeconds": 5,
                                "numberOfProbes": 2,
                                "port": "[parameters('loadBalancedAppPort1')]",
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "inboundNatPools": [
                    ]
                },
                "tags": {
                    "resourceType": "Service Fabric",
                    "clusterName": "[parameters('clusterName')]"
                }
            },
    ```

6. 在 `Microsoft.Compute/virtualMachineScaleSets` 資源的 `networkProfile` 上，新增內部後端位址集區︰

    ```
    "loadBalancerBackendAddressPools": [
                                                        {
                                                            "id": "[variables('lbPoolID0')]"
                                                        },
                                                        {
                                                            /* Add internal BE pool */
                                                            "id": "[variables('lbPoolID0-Int')]"
                                                        }
    ],
    ```

7. 部署範本：

    ```powershell
    New-AzureRmResourceGroup -Name sfnetworkinginternalexternallb -Location westus

    New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternalexternallb -TemplateFile C:\SFSamples\Final\template\_internalexternalLB.json
    ```

在部署後，您會看到資源群組中有兩個負載平衡器。 如果您瀏覽負載平衡器，您會看到公用 IP 位址和指派給公用 IP 位址的管理端點 (連接埠 19000 和 19080)。 您也會看到靜態內部 IP 位址和指派給內部負載平衡器的應用程式端點 (連接埠 80)。 這兩個負載平衡器會使用相同的虛擬機器擴展集後端集區。

## <a name="next-steps"></a>後續步驟
[建立叢集](service-fabric-cluster-creation-via-arm.md)
