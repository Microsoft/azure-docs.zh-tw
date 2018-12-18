---
title: Azure Service Fabric 安裝反向 Proxy | Microsoft Docs
description: 了解如何安裝及設定 Service Fabric 的反向 Proxy。
services: service-fabric
documentationcenter: na
author: jimacoMS2
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 07/27/201
ms.author: v-jamebr
ms.openlocfilehash: c590c9d1ccbbb84a76ba09021a97464ec85c5784
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39507209"
---
# <a name="set-up-and-configure-reverse-proxy-in-azure-service-fabric"></a>安裝及設定 Azure Service Fabric 中的反向 Proxy
反向 Proxy 是選用的 Azure Service Fabric 服務，可協助在 Service Fabric 叢集中執行的微服務進行探索，並與其他擁有 HTTP 端點的服務通訊。 如需詳細資訊，請參閱 [Azure Service Fabric 中的反向 Proxy](service-fabric-reverseproxy.md)。 這篇文章會示範如何在叢集中安裝及設定反向 Proxy。 

## <a name="enable-reverse-proxy-using-azure-portal"></a>使用 Azure 入口網站啟用反向 Proxy

Azure 入口網站提供選項，以在您建立新的 Service Fabric 叢集時啟用反向 Proxy。 您無法升級現有的叢集以透過入口網站使用反向 Proxy。 

若要在[使用 Azure 入口網站建立叢集](./service-fabric-cluster-creation-via-portal.md)時設定反向 Proxy，請確定您執行下列動作：

1. 在 [步驟 2：叢集設定] 中的 [節點類型設定] 下，選取 [啟用反向 Proxy]。

   ![在入口網站上啟用反向 Proxy](./media/service-fabric-reverseproxy-setup/enable-rp-portal.png)
2. (選擇性) 若要設定安全反向 Proxy，需要設定 SSL 憑證。 在 [步驟 3：安全性] 中的 [設定叢集安全性設定] 上，選取 [組態類型] 下的 [自訂]。 然後，在 [反向 Proxy SSL 憑證] 下，選取 [包含反向 Proxy 的 SSL 憑證] 並輸入您的憑證詳細資料。

   ![在入口網站上設定安全反向 Proxy](./media/service-fabric-reverseproxy-setup/configure-rp-certificate-portal.png)

   如果在建立叢集時選擇不使用憑證設定反向 Proxy，則可以稍後透過叢集之資源群組的 Resource Manager 範本執行此操作。 若要深入了解，請參閱[透過 Azure Resource Manager 範本啟用反向 Proxy](#enable-reverse-proxy-via-azure-resource-manager-templates)。

## <a name="enable-reverse-proxy-via-azure-resource-manager-templates"></a>透過 Azure Resource Manager 範本啟用反向 Proxy

針對 Azure 上的叢集，您可以使用 Azure Resource Manager 範本啟用 Service Fabric 中的反向 Proxy。 您可以在建立叢集時啟用反向 Proxy，也可以在稍後藉由更新叢集來啟用它。 

針對新的叢集，您可以[建立自訂的 Resource Manager 範本](service-fabric-cluster-creation-via-arm.md)，也可以使用範例範本。 

您可以在 GitHub 上的[安全反向 Proxy 範例範本](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/ReverseProxySecureSample) \(英文\) 中，找到可幫助您為 Azure 叢集設定安全反向 Proxy 的範例 Resource Manager 範本。 請參閱 README 檔案中的[在安全的叢集中設定 HTTPS 反向 Proxy](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/ReverseProxySecureSample/README.md#configure-https-reverse-proxy-in-a-secure-cluster) \(英文\)，以取得使用憑證設定安全反向 Proxy 和處理憑證變換的指示。

針對現有的叢集，您可以使用 [Azure 入口網站](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-export-template#export-the-template-from-resource-group)、[PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-export-template-powershell#export-resource-group-as-template) 或 [Azure CLI](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-export-template-cli#export-resource-group-as-template) 匯出叢集之資源群組的 Resource Manager 範本。

擁有 Resource Manager 範本後，您可以使用下列步驟來啟用反向 Proxy：

1. 在範本的 [參數區段](../azure-resource-manager/resource-group-authoring-templates.md) 定義反向 Proxy 的連接埠。

    ```json
    "SFReverseProxyPort": {
        "type": "int",
        "defaultValue": 19081,
        "metadata": {
            "description": "Endpoint for Service Fabric Reverse proxy"
        }
    },
    ```
2. 在 [**Microsoft.ServiceFabric/clusters**](https://docs.microsoft.com/azure/templates/microsoft.servicefabric/clusters) 的[資源類型區段](../azure-resource-manager/resource-group-authoring-templates.md)中為每個節點類型物件指定連接埠。

    連接埠是以參數名稱 reverseProxyEndpointPort 識別。

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
       "nodeTypes": [
          {
           ...
           "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]",
           ...
          },
        ...
        ],
        ...
    }
    ```
3. 若要在連接埠上設定反向 Proxy 的 SSL 憑證，請將憑證新增至 **Microsoft.ServiceFabric/clusters** 的[資源類型區段](../resource-group-authoring-templates.md)中的 ***reverseProxyCertificate*** 屬性。

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
        ],
        "properties": {
            ...
            "reverseProxyCertificate": {
                "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                "x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
            },
            ...
            "clusterState": "Default",
        }
    }
    ```

### <a name="supporting-a-reverse-proxy-certificate-thats-different-from-the-cluster-certificate"></a>支援不同於叢集憑證的反向 Proxy 憑證
 如果反向 Proxy 憑證不同於保護叢集的憑證，則先前指定的憑證應該安裝在虛擬機器上，並新增至存取控制清單 (ACL)，讓 Service Fabric 可以存取它。 這可以在 [**Microsoft.Compute/virtualMachineScaleSets**](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachinescalesets) 的[資源類型區段](../resource-group-authoring-templates.md)來完成。 若要進行安裝，請將憑證新增至 osProfile。 範本的延伸模組可以更新 ACL 中的憑證。

  ```json
  {
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    ....
      "osProfile": {
          "adminPassword": "[parameters('adminPassword')]",
          "adminUsername": "[parameters('adminUsername')]",
          "computernamePrefix": "[parameters('vmNodeType0Name')]",
          "secrets": [
            {
              "sourceVault": {
                "id": "[parameters('sfReverseProxySourceVaultValue')]"
              },
              "vaultCertificates": [
                {
                  "certificateStore": "[parameters('sfReverseProxyCertificateStoreValue')]",
                  "certificateUrl": "[parameters('sfReverseProxyCertificateUrlValue')]"
                }
              ]
            }
          ]
        }
   ....
   "extensions": [
          {
              "name": "[concat(parameters('vmNodeType0Name'),'_ServiceFabricNode')]",
              "properties": {
                      "type": "ServiceFabricNode",
                      "autoUpgradeMinorVersion": false,
                      ...
                      "publisher": "Microsoft.Azure.ServiceFabric",
                      "settings": {
                        "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                        "nodeTypeRef": "[parameters('vmNodeType0Name')]",
                        "dataPath": "D:\\\\SvcFab",
                        "durabilityLevel": "Bronze",
                        "testExtension": true,
                        "reverseProxyCertificate": {
                          "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                          "x509StoreName": "[parameters('sfReverseProxyCertificateStoreValue')]"
                        },
                  },
                  "typeHandlerVersion": "1.0"
              }
          },
      ]
    }
  ```
> [!NOTE]
> 當您使用不同於叢集憑證的憑證在現有叢集上啟用反向 Proxy 時，請先在叢集上安裝反向 Proxy 憑證並更新 ACL，再啟用反向 Proxy。 先使用先前所述設定完成 [Azure Resource Manager 範本](service-fabric-cluster-creation-via-arm.md)部署，再開始於步驟 1-3 中啟用反向 Proxy 的部署。

## <a name="enable-reverse-proxy-on-standalone-clusters"></a>在獨立叢集上啟用反向 Proxy

針對獨立叢集，您要在 ClusterConfig.json 檔案中啟用反向 Proxy。 您可以在建立叢集時啟用反向 Proxy，也可以透過升級現有叢集的設定來啟用。 若要深入了解 ClusterConfig.json 檔案中可用的設定，請參閱[獨立叢集設定](./service-fabric-cluster-manifest.md)。

下列步驟會顯示啟用反向 Proxy 使用的設定，以及 (選擇性) 使用 X.509 憑證保護反向 Proxy的設定。 

1. 若要啟用反向 Proxy，請在叢集組態中的 **properties** 下為節點類型設定 **reverseProxyEndpointPort** 值。下列 JSON 顯示為類型為 "NodeType0" 的節點，將反向 Proxy 端點連接埠設定為 19081：

   ```json
       "properties": {
          ... 
           "nodeTypes": [
               {
                   "name": "NodeType0",
                   ...
                   "reverseProxyEndpointPort": "19081",
                   ...
               }
           ],
          ...
       }
   ```
2. (選擇性) 針對安全反向 Proxy，請在 **properties** 下的 **security** 區段中設定憑證。 
   - 針對開發或測試環境，您可以使用 **ReverseProxyCertificate** 設定：

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificate": {
                          "Thumbprint": "[Thumbprint]",
                          "ThumbprintSecondary": "[Thumbprint]",
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```
   - 針對生產環境，建議使用 **ReverseProxyCertificateCommonNames** 設定：

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificateCommonNames": {
                        "CommonNames": [
                            {
                              "CertificateCommonName": "[CertificateCommonName]"
                            }
                          ],
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```

   若要深入了解為獨立叢集設定和管理憑證的相關資訊，以及設定用來保護反向 Proxy 安全之憑證的相關詳細資訊，請參閱 [X509 憑證型安全性](./service-fabric-windows-cluster-x509-security.md)。

在您修改 ClusterConfig.json 檔案以啟用反向 Proxy 後，請按照[升級叢集設定](./service-fabric-cluster-upgrade-windows-server.md#upgrade-the-cluster-configuration)中的說明將更改推送到叢集。


## <a name="expose-reverse-proxy-on-a-public-port-through-azure-load-balancer"></a>透過 Azure 負載平衡器在公用連接埠上公開反向 Proxy

若要從 Azure 叢集外部定址反向 Proxy，請針對反向 Proxy 連接埠設定 Azure Load Balancer 規則與 Azure 健康情況探查。 建立叢集之後，可以隨時使用 Azure 入口網站或 Resource Manager 範本執行這些步驟。 

> [!WARNING]
> 當您在 Load Balancer 中設定反向 Proxy 的連接埠時，叢集中公開 HTTP 端點的所有微服務就能從叢集外部尋址。 這表示已判定的惡意使用者可以探索必定是內部的微服務。 這可能代表可以利用的嚴重弱點，例如：
>
> * 惡意使用者可重複呼叫沒有充分強化攻擊面的內部服務，以啟動拒絕服務的攻擊。
> * 惡意使用者可以將格式不正確的封包，傳遞至導致非預期行為的內部服務。
> * 必定是內部的服務可能會傳回不打算對叢集外部服務公開的私人或敏感性資訊，因而對惡意使用者公開此敏感性資訊。 
>
> 在公開反向 Proxy 連接埠之前，確定您完全了解您的叢集以及在其上執行的應用程式，並減輕其潛在安全性分歧。 
>

如果您想要公開地為獨立叢集公開 (expose) 反向 Proxy，則執行此操作的方式將取決於裝載叢集的系統，這已超出本文的範圍。 然而，關於公開反向 Proxy 的上述警告仍然適用。

### <a name="expose-the-reverse-proxy-using-azure-portal"></a>使用 Azure 入口網站公開反向 Proxy 

1. 在 Azure 入口網站上，按一下叢集的資源群組，然後按一下叢集的負載平衡器。
2. 若要為反向 Proxy 連接埠新增健全狀況探查，請在 [負載平衡器] 視窗的左窗格中，於 [設定] 下，按一下 [健全狀況探查]。 然後，按一下 [健全狀況探查] 視窗頂端的 [新增] 並輸入反向 Proxy 連接埠的詳細資料，然後按一下 [確定]。 根據預設，反向 Proxy 連接埠是 19081，除非您在建立叢集時變更它。

   ![設定反向 Proxy 健全狀況探查](./media/service-fabric-reverseproxy-setup/lb-rp-probe.png)
3. 若要新增負載平衡器規則以公開反向 Proxy 連接埠，請在 [負載均衡器] 視窗的左窗格中，於 [設定] 下按一下 [負載平衡規則]。 然後，按一下 [負載平衡規則] 視窗頂端的 [新增] 並輸入反向 Proxy 連接埠的詳細資料。 請確定您將 [連接埠] 值設定為您想要公開反向 Proxy 的連接埠，[後端連接埠] 值設定為啟用反向 Proxy 時設定的連接埠，以及 [健全狀況探查] 值設定為您在上一個步驟中設定的健全狀況探查。 根據需要設定其他欄位，然後按一下 [確定]。

   ![設定反向 Proxy 的負載平衡器規則](./media/service-fabric-reverseproxy-setup/lb-rp-rule.png)

### <a name="expose-the-reverse-proxy-via-resource-manager-templates"></a>透過 Resource Manager 範本公開反向 Proxy

下列 JSON 參考[透過 Azure Resource Manager 範本啟用反向 Proxy](#enable-reverse-proxy-via-azure-resource-manager-templates) 中使用的相同範本。 如需有關如何建立 Resource Manager 範本或匯出現有叢集之範本的詳細資訊，請參閱文件的該小節。  對 [**Microsoft.Network/loadBalancers**](https://docs.microsoft.com/azure/templates/microsoft.network/loadbalancers) [資源類型區段](../resource-group-authoring-templates.md) 進行了變更。

    ```json
    {
        "apiVersion": "[variables('lbApiVersion')]",
        "type": "Microsoft.Network/loadBalancers",
        ...
        ...
        "loadBalancingRules": [
            ...
            {
                "name": "LBSFReverseProxyRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID0')]"
                    },
                    "backendPort": "[parameters('SFReverseProxyPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig0')]"
                    },
                    "frontendPort": "[parameters('SFReverseProxyPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID0'),'/probes/SFReverseProxyProbe')]"
                    },
                    "protocol": "tcp"
                }
            }
        ],
        "probes": [
            ...
            {
                "name": "SFReverseProxyProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port":     "[parameters('SFReverseProxyPort')]",
                    "protocol": "tcp"
                }
            }  
        ]
    }
    ```


## <a name="customize-reverse-proxy-behavior-using-fabric-settings"></a>使用網狀架構設定自訂反向 Proxy 行為

您可以透過 Resource Manager 範本中的網狀架構設定，為裝載於 Azure 中的叢集或 ClusterConfig.json 檔案中的獨立叢集，自訂反向 Proxy 的行為。 控制反向 Proxy 行為的設定，位於叢集 **properties** 區段下 **fabricSettings** 區段中的 [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp) 區段內。 

例如，您可以設定 **DefaultHttpRequestTimeout** 的值，以將對反向 Proxy 之要求的逾時設為 180 秒，如以下 JSON 所示：

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "DefaultHttpRequestTimeout",
                   "value": "180"
                 }
               ]
             }
           ],
           ...
   }
   ``` 

如需更新 Azure 叢集之網狀架構設定的相關詳細資訊，請參閱[使用 Resource Manager 範本自訂叢集設定](./service-fabric-cluster-fabric-settings.md#customize-cluster-settings-using-resource-manager-templates)。 針對獨立叢集，請參閱[自訂獨立叢集的叢集設定](./service-fabric-cluster-fabric-settings.md#customize-cluster-settings-for-standalone-clusters)。 

多個網狀架構設定用來協助建立反向 Proxy 與服務之間的安全通訊。 如需這些設定的詳細資訊，請參閱[連線到安全服務與反向 Proxy](service-fabric-reverseproxy-configure-secure-communication.md)。

## <a name="next-steps"></a>後續步驟
* [設定透過反向 Proxy 轉送到安全的 HTTP 服務](service-fabric-reverseproxy-configure-secure-communication.md)
* 如需反向 Proxy 組態選項，請參閱[自訂 Service Fabric 叢集設定中的 ApplicationGateway/Http 區段](service-fabric-cluster-fabric-settings.md#applicationgatewayhttp)。
