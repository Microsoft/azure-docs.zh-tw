---
title: 設定 Azure Service Fabric 容器服務的網路模式 | Microsoft Docs
description: 了解如何設定 Azure Service Fabric 所支援的不同網路模式。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: d552c8cd-67d1-45e8-91dc-871853f44fc6
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: 869b87b8df3b1f532a33e943e728681b358ed8b4
ms.sourcegitcommit: d8ffb4a8cef3c6df8ab049a4540fc5e0fa7476ba
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36287621"
---
# <a name="service-fabric-container-networking-modes"></a>Service Fabric 容器網路模式

適用於容器服務的 Azure Service Fabric 叢集預設會使用 **nat** 網路模式。 當有多個容器服務在相同的連接埠上接聽，且正在使用 nat 模式時，可能會發生部署錯誤。 為了支援在相同連接埠上進行多個容器服務接聽，Service Fabric 會提供 **Open** 網路模式 (5.7 版及更新版本)。 在 Open 模式下，每個容器服務都具一個以動態方式指派的內部 IP 位址，此 IP 位址可支援在相同連接埠上進行多個服務接聽。  

如果您的其中一個容器服務在服務資訊清單中含有靜態端點，可以使用 Open 模式來建立與刪除新的服務，而不會發生任何部署錯誤。 相同的 docker-compose.yml 檔案也可以搭配靜態連接埠對應使用，以建立多個服務。

當某個容器服務重新啟動或移到叢集中的另一個節點時，IP 位址便會變更。 因此，不建議使用以動態方式指派的 IP 位址來探索容器服務。 只有 Service Fabric 命名服務或 DNS 服務可用於服務探索。 

>[!WARNING]
>Azure 針對每個虛擬網路皆允許最多 4,096 個 IP。 節點數目和 (使用 Open 模式的) 容器服務執行個體數目的總和，在虛擬網路內不能超過 4,096 個 IP。 針對高密度的情況，建議使用 nat 網路模式。
>

## <a name="set-up-open-networking-mode"></a>設定 Open 網路模式

1. 設定 Azure Resource Manager 範本。 在 [fabricSettings] 區段中，啟用 DNS 服務及 IP 提供者： 

    ```json
    "fabricSettings": [
                {
                    "name": "DnsService",
                    "parameters": [
                       {
                            "name": "IsEnabled",
                            "value": "true"
                      }
                    ]
                },
                {
                    "name": "Hosting",
                    "parameters": [
                      { 
                            "name": "IPProviderEnabled",
                            "value": "true"
                      }
                    ]
                },
                {
                    "name":  "Trace/Etw", 
                    "parameters": [
                    {
                            "name": "Level",
                            "value": "5"
                    }
                    ]
                },
                {
                    "name": "Setup",
                    "parameters": [
                    {
                            "name": "ContainerNetworkSetup",
                            "value": "true"
                    }
                    ]
                }
            ],
    ```

2. 設定網路設定檔區段，以允許在叢集的每個節點上設定多個 IP 位址。 下列範例會針對 Windows/Linux Service Fabric 叢集的每個節點，設定五個 IP 位址。 您可以在每個節點的連接埠上接聽五個服務執行個體。

    ```json
    "variables": {
        "nicName": "NIC",
        "vmName": "vm",
        "virtualNetworkName": "VNet",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
        "vmNodeType0Name": "[toLower(concat('NT1', variables('vmName')))]",
        "subnet0Name": "Subnet-0",
        "subnet0Prefix": "10.0.0.0/24",
        "subnet0Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet0Name'))]",
        "lbID0": "[resourceId('Microsoft.Network/loadBalancers',concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType0Name')))]",
        "lbIPConfig0": "[concat(variables('lbID0'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
        "lbPoolID0": "[concat(variables('lbID0'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
        "lbProbeID0": "[concat(variables('lbID0'),'/probes/FabricGatewayProbe')]",
        "lbHttpProbeID0": "[concat(variables('lbID0'),'/probes/FabricHttpGatewayProbe')]",
        "lbNatPoolID0": "[concat(variables('lbID0'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]"
    }
    "networkProfile": {
                "networkInterfaceConfigurations": [
                  {
                    "name": "[concat(parameters('nicName'), '-0')]",
                    "properties": {
                      "ipConfigurations": [
                        {
                          "name": "[concat(parameters('nicName'),'-',0)]",
                          "properties": {
                            "primary": "true",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "loadBalancerInboundNatPools": [
                              {
                                "id": "[variables('lbNatPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 1)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 2)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 3)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 4)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 5)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        }
                      ],
                      "primary": true
                    }
                  }
                ]
              }
   ```
 
3. 僅針對 Windows 叢集，使用下列值來設定能針對虛擬網路開啟連接埠 UDP/53 的 Azure 網路安全性群組 (NSG) 規則：

   |設定 |值 | |
   | --- | --- | --- |
   |優先順序 |2000 | |
   |Name |Custom_Dns  | |
   |來源 |VirtualNetwork | |
   |目的地 | VirtualNetwork | |
   |服務 | DNS (UDP/53) | |
   |動作 | 允許  | |
   | | |

4. 在每個服務的應用程式資訊清單中指定網路模式：`<NetworkConfig NetworkType="Open">`。 **Open** 網路模式會使服務取得專用 IP 位址。 如果未指定模式，服務會預設為 **nat** 模式。 在下列資訊清單範例中，`NodeContainerServicePackage1` 和 `NodeContainerServicePackage2` 服務可以分別在相同的連接埠上接聽 (這兩個服務都會在 `Endpoint1` 上接聽)。 指定 Open 網路模式之後，就無法指定 `PortBinding` 設定。

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <ApplicationManifest ApplicationTypeName="NodeJsApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <Description>Calculator Application</Description>
      <Parameters>
        <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
        <Parameter Name="MyCpuShares" DefaultValue="3"></Parameter>
      </Parameters>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeContainerServicePackage1" ServiceManifestVersion="1.0"/>
        <Policies>
          <ContainerHostPolicies CodePackageRef="NodeContainerService1.Code" Isolation="hyperv">
           <NetworkConfig NetworkType="Open"/>
          </ContainerHostPolicies>
        </Policies>
      </ServiceManifestImport>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeContainerServicePackage2" ServiceManifestVersion="1.0"/>
        <Policies>
          <ContainerHostPolicies CodePackageRef="NodeContainerService2.Code" Isolation="default">
            <NetworkConfig NetworkType="Open"/>
          </ContainerHostPolicies>
        </Policies>
      </ServiceManifestImport>
    </ApplicationManifest>
    ```

    您可以跨 Windows 叢集之應用程式內的服務，混合和比對不同網路模式。 有些服務可以使用 Open 模式，有些服務則是使用 nat 模式。 當服務設定為使用 nat 模式時，服務接聽的連接埠必須是唯一的。

    >[!NOTE]
    >在 Linux 叢集上，不支援針對不同服務混合網路模式。 
    >

5. 選取 [開啟] 模式時，服務資訊清單中的 [端點] 定義應該明確地指向對應至端點的程式碼套件，即使服務套件中只有一個程式碼套件也是一樣。 
   
   ```xml
   <Resources>
     <Endpoints>
       <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" CodePackageRef="Code"/>
     </Endpoints>
   </Resources>
   ```
   
6. 就 Windows 而言、VM 重新開機將會導致開放式網路重新建立。 這是為了緩解網路堆疊中潛藏的問題。 預設行為是重新建立網路。 如果必須關閉此行為，您可以在進行組態升級之前使用下列組態。

```json
"fabricSettings": [
                {
                    "name": "Setup",
                    "parameters": [
                    {
                            "name": "SkipContainerNetworkResetOnReboot",
                            "value": "true"
                    }
                    ]
                }
            ],          
 ``` 
 
## <a name="next-steps"></a>後續步驟
* [了解 Service Fabric 應用程式模型](service-fabric-application-model.md)
* [深入了解 Service Fabric 服務資訊清單資源](https://docs.microsoft.com/azure/service-fabric/service-fabric-service-manifest-resources)
* [將 Windows 容器部署至 Windows Server 2016 上的 Service Fabric](service-fabric-get-started-containers.md)
* [將 Docker 容器部署至 Linux 上的 Service Fabric](service-fabric-get-started-containers-linux.md)
