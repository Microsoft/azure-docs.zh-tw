---
title: 使用 FreeBSD 的封包篩選器在 Azure 中建立防火牆 | Microsoft Docs
description: 了解如何使用 FreeBSD 的 PF 在 Azure 中部署 NAT 防火牆。
services: virtual-machines-linux
documentationcenter: ''
author: KylieLiang
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/20/2017
ms.author: kyliel
ms.openlocfilehash: 8400fa12e8776834f71740f809df18753291601f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992300"
---
# <a name="how-to-use-freebsds-packet-filter-to-create-a-secure-firewall-in-azure"></a>如何使用 FreeBSD 的「封包篩選器」在 Azure 中建立安全防火牆
本文介紹如何透過 Azure Resource Manager 範本，針對常見的 Web 伺服器案例，使用 FreeBSD 的「封包篩選器」來部署 NAT 防火牆。

## <a name="what-is-pf"></a>什麼是 PF？
PF (封包篩選器，也寫成 pf) 是一個 BSD 授權的具狀態封包篩選器，是防火牆軟體的一個核心部分。 PF 一直以來快速發展，現在已有數個超越其他可用防火牆的優點。 PF 從一開始便包含「網路位址轉譯」(NAT)，接著，藉由整合 ALTQ 並將它變成可透過 PF 組態來進行設定，整合了封包排程器和作用中佇列管理。 此外，一些功能也將 PF 加以延伸，例如用來進行容錯移轉和備援的 pfsync 和 CARP、用來進行工作階段驗證的 authpf，以及可輕鬆為難以設定的 FTP 通訊協定建立防火牆的 ftp-proxy 等功能。 簡單說來，PF 就是一個強大且功能豐富的防火牆。 

## <a name="get-started"></a>開始使用
如果您對於在雲端為您的 Web 伺服器設定安全的防火牆感興趣，就讓我們開始吧！ 您也可以套用這個 Azure Resource Manager 範本中使用的指令碼來設定您的網路拓撲。
Azure Resource Manager 範本會設定一部 FreeBSD 虛擬機器，此虛擬機器會使用 PF 和兩部已安裝並設定 Nginx Web 伺服器的 FreeBSD 虛擬機器，來執行 NAT/重新導向。 除了為兩部 Web 伺服器的輸出流量執行 NAT 之外，NAT/重新導向虛擬機器還會攔截 HTTP 要求，然後以循環方式將這些要求重新導向到兩部 Web 伺服器。 VNet 會使用無法經路由傳送的私人 IP 位址空間 10.0.0.2/24，而您可以修改此範本的參數。 Azure Resource Manager 範本也為整個 VNet 定義了路由表，此表格是個別路由的集合，可用來根據目的地 IP 位址覆寫 Azure 預設路由。 

![pf_topology](./media/freebsd-pf-nat/pf_topology.jpg)
    
### <a name="deploy-through-azure-cli"></a>透過 Azure CLI 進行部署
您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。 使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 `West US` 位置建立名為 `myResourceGroup` 的資源群組。

```azurecli
az group create --name myResourceGroup --location westus
```

接著，使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create) 來部署 [pf-freebsd-setup](https://github.com/Azure/azure-quickstart-templates/tree/master/pf-freebsd-setup) 範本。 下載相同路徑下的 [azuredeploy.parameters.json](https://github.com/Azure/azure-quickstart-templates/blob/master/pf-freebsd-setup/azuredeploy.parameters.json)，並定義您自己的資源值，例如 `adminPassword`、`networkPrefix` 及 `domainNamePrefix`。 

```azurecli
az group deployment create --resource-group myResourceGroup --name myDeploymentName \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/pf-freebsd-setup/azuredeploy.json \
    --parameters '@azuredeploy.parameters.json' --verbose
```

在大約 5 分鐘後，您將會收到 `"provisioningState": "Succeeded"` 的資訊。 接著，您可以透過 SSH 連線到前端 VM (NAT)，或在瀏覽器中使用公用 IP 位址或前端 VM (NAT) 的 FQDN 來存取 Nginx Web 伺服器。 下列範例會列出指派給 `myResourceGroup` 資源群組中前端 VM (NAT) 的 FQDN 和公用 IP 位址。 

```azurecli
az network public-ip list --resource-group myResourceGroup
```
    
## <a name="next-steps"></a>後續步驟
您想要在 Azure 中設定自己的 NAT 嗎？ 需要是開放原始碼、免費但具有強大功能嗎？ 那麼，PF 會是一個理想的選擇。 藉由使用 [pf-freebsd-setup](https://github.com/Azure/azure-quickstart-templates/tree/master/pf-freebsd-setup) 範本，您只需 5 分鐘的時間，即可使用 FreeBSD 的 PF，在 Azure 中針對常見的 Web 伺服器案例，設定具備循環配置資源負載平衡功能的 NAT 防火牆。 

如果您想要了解 Azure 中的 FreeBSD 供應項目，請參閱[Azure 上的 FreeBSD 簡介](freebsd-intro-on-azure.md)。

如果您想要深入了解 PF，請參考 [FreeBSD 手冊 (英文)](https://www.freebsd.org/doc/handbook/firewalls-pf.html) 或 [PF 使用者指南 (英文)](https://www.freebsd.org/doc/handbook/firewalls-pf.html)。
