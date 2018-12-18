---
title: 建立、變更或刪除虛擬網路 TAP - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立、變更或刪除虛擬網路 TAP。
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: kaanan
ms.openlocfilehash: 36de5ec6f7384663106bfb88ee9f236cced6930a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997942"
---
# <a name="work-with-a-virtual-network-tap-using-the-azure-cli"></a>使用 Azure CLI 來處理虛擬網路 TAP

Azure 虛擬網路 TAP (終端機存取點) 可讓您持續將您的虛擬機器網路流量串流到網路封包收集器或分析工具。 收集器或分析工具是由[網路虛擬設備](https://azure.microsoft.com/solutions/network-appliances/)合作夥伴所提供。 如需經驗證能與虛擬網路 TAP 相容的合作夥伴解決方案清單，請參閱[合作夥伴解決方案](virtual-network-tap-overview.md#virtual-network-tap-partner-solutions)。 

## <a name="create-a-virtual-network-tap-resource"></a>建立虛擬網路 TAP 資源

建立虛擬網路 TAP 資源之前，請先閱讀[必要條件](virtual-network-tap-overview.md#prerequisites)。 您可以執行 [Azure Cloud Shell](https://shell.azure.com/bash) 中採用的命令，或從您的電腦執行 Azure 命令列介面 (CLI)。 Azure Cloud Shell 是免費的互動式殼層，而且不會要求您必須在電腦上安裝 Azure CLI。 您必須使用具有適當[權限](virtual-network-tap-overview.md#permissions)的帳戶登入 Azure。 此文章需要 Azure CLI 2.0.46 版或更新版本。 執行 `az --version` 來了解安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI 2.0](/cli/azure/install-azure-cli)。 如果您在本機執行 Azure CLI，則也需要執行 `az login` 以建立與 Azure 的連線。

1. 將您訂用帳戶的識別碼擷取到在稍後步驟使用的變數中：

   ```azurecli-interactive
   subscriptionId=$(az account show \
   --query id \
   --out tsv)
   ```

2. 設定您將用來建立虛擬網路 TAP 資源的訂用帳戶識別碼。

   ```azurecli-interactive
   az account set --subscription $subscriptionId
   ```

3. 重新註冊您將用來建立虛擬網路 TAP 資源的訂用帳戶識別碼。 若建立 TAP 資源時發生註冊錯誤，請執行下列命令：

   ```azurecli-interactive
   az provider register --namespace Microsoft.Network --subscription $subscriptionId
   ```

4. 若虛擬網路 TAP 的目的地是收集器或分析工具之網路虛擬設備上的網路介面 -

   - 將網路虛擬設備之網路介面的 IP 設定擷取到在稍後步驟使用的變數中。 識別碼是將會彙總 TAP 流量的端點。 下列範例會擷取名為 *myResourceGroup* 之資源群組中名為 *myNetworkInterface* 的網路介面 *ipconfig1* IP 設定識別碼：

      ```azurecli-interactive
       IpConfigId=$(az network nic ip-config show \
       --name ipconfig1 \
       --nic-name myNetworkInterface \
       --resource-group myResourceGroup \
       --query id \
       --out tsv)
      ```

   - 使用 IP 設定的識別碼做為目的地與選擇性的連接埠屬性，在美國中西部 Azure 區域中建立虛擬網路 TAP。 連接埠會指定將會接收 TAP 流量之網路介面 IP 設定上的目的地連接埠：  

      ```azurecli-interactive
       az network vnet tap create \
       --resource-group myResourceGroup \
       --name myTap \
       --destination $IpConfigId \
       --port 4789 \
       --location westcentralus
      ```

5. 若虛擬網路 TAP 的目的地是 Azure 內部負載平衡器：
  
   - 將 Azure 內部負載平衡器的前端 IP 設定擷取到在稍後步驟使用的變數中。 識別碼是將會彙總 TAP 流量的端點。 下列範例會擷取名為 *myResourceGroup* 之資源群組中名為 *myInternalLoadBalancer* 的負載平衡器 *frontendipconfig1* 前端 IP 設定的識別碼：

      ```azurecli-interactive
      FrondendIpConfigId=$(az network lb fronend-ip show \
      --name frontendipconfig1 \
      --lb-name myInternalLoadBalancer \
      --resource-group myResourceGroup \
      --query id \
      --out tsv)
      ```
   - 使用前端 IP 設定的識別碼做為目的地與選擇性的連接埠屬性，建立虛擬網路 TAP。 連接埠會指定將會接收 TAP 流量之前端 IP 設定上的目的地連接埠：  

      ```azurecli-interactive
      az network vnet tap create \
      --resource-group myResourceGroup \
      --name myTap \
      --destination $FrontendIpConfigId \
      --port 4789 \
     --location westcentralus
     ```

6. 確認已建立虛擬網路 TAP：

   ```azurecli-interactive
   az network vnet tap show \
   --resource-group myResourceGroup
   --name myTap
   ```

## <a name="add-a-tap-configuration-to-a-network-interface"></a>新增 TAP 設定到網路介面

1. 擷取現有虛擬網路 TAP 資源的識別碼。 下列範例會擷取名為 *myResourceGroup* 之資源群組中名為 *myTap* 的虛擬網路 TAP：

   ```azurecli-interactive
   tapId=$(az network tap show show \
   --name myTap \
   --resource-group myResourceGroup \
   --query id \
   --out tsv)
   ```

2. 在監視之虛擬機器的網路介面上建立 TAP 設定。 下列範例會為名為 *myNetworkInterface* 的網路介面建立 TAP 設定：

   ```azurecli-interactive
   az network nic vtap-config create \
   --resource-group myResourceGroup \
   --nic myNetworkInterface \
   --vnet-tap $tapId \
   --name mytapconfig \
   --subscription subscriptionId
   ```

3. 確認已建立 TAP 設定：

   ```azurecli-interactive
   az network nic vtap-config show \
   --resource-group myResourceGroup \
   --nic-name myNetworkInterface \
   --name mytapconfig \
   --subscription subscriptionId
   ```

## <a name="delete-the-tap-configuration-on-a-network-interface"></a>刪除網路介面上的 TAP 設定

   ```azure-cli-interactive
   az network nic vtap-config delete \
   --resource-group myResourceGroup \
   --nic myNetworkInterface \
   --tap-configuration-name myTapConfig \
   --subscription subscriptionId
   ```

## <a name="list-virtual-network-taps-in-a-subscription"></a>列出訂用帳戶中的虛擬網路 TAP

   ```azurecli-interactive
   az network vnet tap list
   ```

## <a name="delete-a-virtual-network-tap-in-a-resource-group"></a>刪除資源群組中的虛擬網路 TAP

   ```azurecli-interactive
   az network vnet tap delete \
   --resource-group myResourceGroup \
   --name myTap
   ```