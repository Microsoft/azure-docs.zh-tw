---
title: 部署 Azure 虛擬網路容器網路功能 | Microsoft Docs
description: 了解如何為您自行部署、使用 ACS-Engine 部署的 Kubernetes 叢集和 Docker 容器，來部署 Azure 虛擬網路容器網路介面 (CNI) 外掛程式。
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/18/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: 62d19432cba431bce4485aaa2af3e0a23ad8b5f6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970969"
---
# <a name="deploy-the-azure-virtual-network-container-network-interface-plug-in"></a>部署 Azure 虛擬網路容器網路介面外掛程式

Azure 虛擬網路容器網路介面 (CNI) 外掛程式會安裝 Azure 虛擬機器，並將虛擬網路功能帶到 Kubernetes Pod 和 Docker 容器。 若要深入了解外掛程式，請參閱[讓容器使用 Azure 虛擬網路功能](container-networking-overview.md)。 此外，選擇[進階網路功能](../aks/networking-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)可讓外掛程式搭配 Azure Kubernetes Service (AKS)，因為該選項會自動將 AKS 放在虛擬網路中。

## <a name="deploy-plug-in-for-acs-engine-kubernetes-cluster"></a>部署 ACS-Engine Kubernetes 叢集外掛程式

ACS-Engine 使用 Azure Resource Manager 範本部署 Kubernetes 叢集。 叢集設定是在 JSON 檔案中設定，產生範本時會將該檔案傳遞到工具。 若要深入了解支援叢集設定及其說明的完整清單，請參閱 [Microsoft Azure Container Service 引擎 - 叢集定義](https://github.com/Azure/acs-engine/blob/master/docs/clusterdefinition.md)。 外掛程式是針對使用 ACS-Engine 建立的叢集的預設網路功能外掛程式。 設定外掛程式時，下列網路組態設定很重要：

  | 設定                              | 說明                                                                                                           |
  |--------------------------------------|------------------------------------------------------------------------------------------------------                 |
  | firstConsecutiveStaticIP             | 配置給主要節點的 IP 位址。 這是必要的設定。                                     |
  | kubernetesConfig 下的 clusterSubnet | 部署叢集的虛擬網路子網路的 CIDR，以及配置給 Pods 的 IP 位址   |
  | masterProfile 下的 vnetSubnetId     | 指定要部署叢集所在的子網路的 Azure Resource Manager 資源識別碼                    |
  | vnetCidr                             | 要部署叢集所在的虛擬網路的 CIDR                                                             |
  | kubeletConfig 下的 max-Pods         | 每個代理程式虛擬機器上的 Pod 數目上限。 外掛程式的預設值是 30 個。 最多可以指定到 250 個  |

### <a name="example-configuration"></a>設定範例

接下來的 json 範例是含有下列屬性的叢集：
-   1 個主要節點和 2 個代理程式節點 
-   部署在名為 *KubeClusterSubnet* (10.0.0.0/20) 的子網路，其中同時放置有主要和代理程式節點。

```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "kubernetesConfig": {
        "clusterSubnet": "10.0.0.0/20" --> Subnet allocated for the cluster
      }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "ACSKubeMaster",
      "vmSize": "Standard_A2",
      "vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<Vnet Name>/subnets/KubeClusterSubnet",
      "firstConsecutiveStaticIP": "10.0.1.50", --> IP address allocated to the Master node
"vnetCidr": "10.0.0.0/16" --> Virtual network address space
    },
    "agentPoolProfiles": [
      {
        "name": "k8sagentpoo1",
        "count": 2,
        "vmSize": "Standard_A2_v2",
"vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<VNet Name>/subnets/KubeClusterSubnet",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
    "linuxProfile": {
      "adminUsername": "KubeServerAdmin",
      "ssh": {
        "publicKeys": [
          {…}
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "dd438987-aa12-4754-b47d-375811889714",
      "secret": "azure123"
    }
  }
}
```

## <a name="deploy-plug-in-for-a-kubernetes-cluster"></a>部署 Kubernetes 叢集外掛程式

請完成下列步驟以在 Kubernetes 叢集中每個 Azure 虛擬機器上安裝外掛程式：

1. [下載並安裝外掛程式](#download-and-install-the-plug-in)。
2. 在每個虛擬機器上預先配置虛擬網路 IP 位址集區，這些 IP 位址將會指派給 Pod。 每個 Azure 虛擬機器在每個網路介面上都隨附一個主要虛擬網路私人 IP 位址。 Pod 的 IP 位址集區會透過下列其中一個選項新增到虛擬機器網路介面上做為次要位址 (*ipconfig*)：

   - **CLI**：[使用 Azure CLI 指派多個 IP 位址](virtual-network-multiple-ip-addresses-cli.md)
   - **PowerShell**：[使用 PowerShell 指派多個 IP 位址](virtual-network-multiple-ip-addresses-powershell.md)
   - **入口網站**：[使用 Azure 入口網站指派多個 IP 位址](virtual-network-multiple-ip-addresses-portal.md)
   - **Azure Resource Manager 範本**：[使用範本指派多個 IP　位址](virtual-network-multiple-ip-addresses-template.md)

   請確定您在虛擬機器上為所有要啟動的 Pod 新增足夠的 IP 位址。

3. 選取為叢集提供網路功能的外掛程式，方法是在建立叢集的時候對 Kubelet 傳遞 `–network-plugin=cni` 命令列選項。 根據預設，Kubernetes 會在已經安裝它們的目錄中尋找外掛程式和組態檔。
4. 如果您想要 Pod 存取網際網路，請將下列 *iptables* 規則新增到 Linux 虛擬機器以獲取 NAT 網際網路流量。 在下列範例中，指定的 IP 範圍會是 10.0.0.0/8。

   ```bash
   iptables -t nat -A POSTROUTING -m iprange ! --dst-range 168.63.129.16 -m
   addrtype ! --dst-type local ! -d 10.0.0.0/8 -j MASQUERADE
   ```

   不是要傳送到指定 IP 範圍的 NAT 流量規則。 假設是先前範圍之外的所有流量都是網際網路流量。 您可以選擇指定虛擬機器的虛擬網路、對等互連虛擬網路以及內部部署網路的 IP 範圍。

  Windows 虛擬機器會獲取 NAT 流量，該流量的目的地位於虛擬機器所屬子網路的外部。 您不能指定自訂 IP 範圍。

完成前面的步驟之後，Kubernetes 代理程式虛擬機器上啟動的 Pod 會自動從虛擬網路指派私人 IP 位址。

## <a name="deploy-plug-in-for-docker-containers"></a>部署 Docker 容器外掛程式

1. [下載並安裝外掛程式](#download-and-install-the-plug-in)。
2. 使用下列命令建立 Docker 容器：

   ```
   ./docker-run.sh \<container-name\> \<container-namespace\> \<image\>
   ```

容器會自動開始從配置的集區接收 IP 位址。 如果您想要平衡流量的負載到 Docker 容器，它們必須放置在軟體負載平衡器的後方，而且您必須設定負載平衡器探查，就和為虛擬機器建立原則和探查的方式一樣。

### <a name="cni-network-configuration-file"></a>CNI 網路組態檔

CNI 網路組態檔是以 JSON 格式描述。 根據預設，它出現在 Linux 的 `/etc/cni/net.d`和 Windows 的 `c:\cni\netconf` 中。 檔案會指定外掛程式的設定，而 Windows 和 Linux 是不同的。 下面的 json 是範例 Linux 組態檔，後面跟著一些重要設定的說明。 您完全不必對檔案進行任何變更：

```json
{
       "cniVersion":"0.3.0",
       "name":"azure",
       "plugins":[
          {
             "type":"azure-vnet",
             "mode":"bridge",
             "bridge":"azure0",
             "ipam":{
                "type":"azure-vnet-ipam"
             }
          },
          {
             "type":"portmap",
             "capabilities":{
                "portMappings":true
             },
             "snat":true
          }
       ]
}
```

#### <a name="settings-explanation"></a>設定說明

- **cniVersion**：Azure 虛擬網路 CNI 外掛程式支援版本 0.3.0 和 0.3.1 的 [CNI 規格](https://github.com/containernetworking/cni/blob/master/SPEC.md)。
- **name**：網路的名稱。 這個屬性可以設定為任何唯一值。
- **type**：網路外掛程式的名稱。 設定為 *azure-vnet*。
- **mode**：作業模式。 此為選擇性欄位。 唯一支援的模式是 "bridge"。 如需詳細資訊，請參閱[作業模式](https://github.com/Azure/azure-container-networking/blob/master/docs/network.md)。
- **bridge**：橋接器名稱，用來將容器連線到虛擬網路。 此為選擇性欄位。 如果省略，外掛程式會根據主要介面索引自動挑選一個唯一名稱。
- **ipam type**：IPAM 外掛程式的名稱。 一律設定為 *azure-vnet-ipam*。

## <a name="download-and-install-the-plug-in"></a>下載並安裝外掛程式

從 [GitHub](https://github.com/Azure/azure-container-networking/releases) 下載外掛程式。 下載適用於您使用平台的最新版本：

- **Linux**：[azure-vnet-cni-linux-amd64-\<version no.\>.tgz](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-linux-amd64-v1.0.12-rc3.tgz)
- **Windows**：[azure-vnet-cni-windows-amd64-\<version no.\>.zip](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-windows-amd64-v1.0.12-rc3.zip)

將 [Linux](https://github.com/Azure/azure-container-networking/blob/master/scripts/install-cni-plugin.sh) 或 [Windows](https://github.com/Azure/azure-container-networking/blob/master/scripts/Install-CniPlugin.ps1) 的安裝指令碼複製到您的電腦。 將指令碼儲存到電腦上的 `scripts` 目錄，並將檔案命名為 `install-cni-plugin.sh` (Linux) 或 `install-cni-plugin.ps1` (Windows)。 若要安裝外掛程式，請針對平台執行適當的指令碼，指定所使用外掛程式的版本。 例如，您可以指定 *v1.0.12-rc3*：

   ```bash
   \$scripts/install-cni-plugin.sh [version]
   ```

   ```powershell
   scripts\\ install-cni-plugin.ps1 [version]
   ```

指令碼會將外掛程式安裝在 `/opt/cni/bin` (Linux) 和 `c:\cni\bin` (Windows) 下。 安裝的外掛程式隨附一個在安裝之後即可運作的簡單網路組態檔，。 它不需要更新。 若要深入了解檔案中的設定，請參閱 [CNI 網路組態檔](#cni-network-configuration-file)。