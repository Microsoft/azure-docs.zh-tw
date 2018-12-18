---
title: 使用 Azure CLI 在多個 IP 組態上進行負載平衡 | Microsoft Docs
description: 了解如何使用 Azure CLI 對虛擬機器指派多個 IP 位址。
services: virtual-network
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2018
ms.author: kumud
ms.openlocfilehash: f9cd6405f5c3c87cdf004f8a71b9e72d58532a12
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37108919"
---
# <a name="load-balancing-on-multiple-ip-configurations-using-azure-cli"></a>使用 Azure CLI 在多個 IP 組態上進行負載平衡

本文說明如何對次要網路介面 (NIC) 上的多個 IP 位址使用 Azure Load Balancer。 在此案例中，我們有兩部執行 Windows 的 VM，每部各有一個主要和次要 NIC。 每個次要 NIC 都有兩個 IP 組態。 每部 VM 都裝載 contoso.com 和 fabrikam.com 兩個網站。 每個網站繫結到次要 NIC 上的其中一個 IP 組態。 我們使用 Azure Load Balancer 來公開兩個前端 IP 位址，每個網站各使用其中一個，以將流量分散給網站的個別 IP 組態。 此案例會在兩個前端以及兩個後端集區 IP 位址使用相同的連接埠號碼。

![LB 案例影像](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>在多個 IP 組態上進行負載平衡的步驟

請完成下列步驟來達成本文所述案例︰

1. [安裝和設定 Azure CLI]((https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest))，方法是遵循所連結文章內的步驟並登入 Azure 帳戶。
2. 如下所述[建立資源群組](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-resource-group) (稱為 *contosofabrikam*)。

    ```azurecli
    az group create contosofabrikam westcentralus
    ```

3. 為兩個 VM [建立可用性集合](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-an-availability-set)。 在此案例中，請使用下列命令：

    ```azurecli
    az vm availability-set create --resource-group contosofabrikam --location westcentralus --name myAvailabilitySet
    ```

4. [建立虛擬網路](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-virtual-network-and-subnet) (稱為 myVNet) 和子網路 (稱為 mySubnet)：

    ```azurecli
    az network vnet create --resource-group contosofabrikam --name myVnet --address-prefixes 10.0.0.0/16  --location westcentralus --subnet-name MySubnet --subnet-prefix 10.0.0.0/24

    ```

5. [建立負載平衡器](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (稱為 mylb)：

    ```azurecli
    az network lb create --resource-group contosofabrikam --location westcentralus --name mylb
    ```

6. 針對負載平衡器的前端 IP 組態建立兩個動態公用 IP 位址︰

    ```azurecli
    az network public-ip create --resource-group contosofabrikam --location westcentralus --name PublicIp1 --domain-name-label contoso --allocation-method Dynamic

    az network public-ip create --resource-group contosofabrikam --location westcentralus --name PublicIp2 --domain-name-label fabrikam --allocation-method Dynamic
    ```

7. 分別建立兩個前端 IP 組態 contosofe 和 fabrikamfe︰

    ```azurecli
    az network lb frontend-ip create --resource-group contosofabrikam --lb-name mylb --public-ip-name PublicIp1 --name contosofe
    az network lb frontend-ip create --resource-group contosofabrikam --lb-name mylb --public-ip-name PublicIp2 --name fabrkamfe
    ```

8. 建立後端位址集區 - contosopool 和 fabrikampool、[探查](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json) - HTTP，和負載平衡規則 - HTTPc 和 HTTPf：

    ```azurecli
    az network lb address-pool create --resource-group contosofabrikam --lb-name mylb --name contosopool
    azure network lb address-pool create --resource-group contosofabrikam --lb-name mylb --name fabrikampool

    az network lb probe create --resource-group contosofabrikam --lb-name mylb --name HTTP --protocol "http" --interval 15 --count 2 --path index.html

    az network lb rule create --resource-group contosofabrikam --lb-name mylb --name HTTPc --protocol tcp --probe-name http--frontend-port 5000 --backend-port 5000 --frontend-ip-name contosofe --backend-address-pool-name contosopool
    az network lb rule create --resource-group contosofabrikam --lb-name mylb --name HTTPf --protocol tcp --probe-name http --frontend-port 5000 --backend-port 5000 --frontend-ip-name fabrkamfe --backend-address-pool-name fabrikampool
    ```

9. 執行下列命令，檢查輸出以[確認負載平衡器](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json)已正確建立︰

    ```azurecli
    az network lb show --resource-group contosofabrikam --name mylb
    ```

10. 如下所示為第一個虛擬機器 VM1 [建立公用 IP](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-public-ip-address) (*myPublicIp*) 和[儲存體帳戶](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (*mystorageaccont1*)︰

    ```azurecli
    az network public-ip create --resource-group contosofabrikam --location westcentralus --name myPublicIP --domain-name-label mypublicdns345 --allocation-method Dynamic

    az storage account create --location westcentralus --resource-group contosofabrikam --kind Storage --sku-name GRS mystorageaccount1
    ```

11. 如下所示為 VM1 [建立網路介面](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-virtual-nic)與新增第二個 IP 組態 (*VM1-ipconfig2*)，然後[建立 VM](../virtual-machines/linux/create-cli-complete.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-vm)︰

    ```azurecli
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM1Nic1 --ip-config-name NIC1-ipconfig1
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM1Nic2 --ip-config-name VM1-ipconfig1 --public-ip-name myPublicIP --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/contosopool"
    az network nic ip-config create --resource-group contosofabrikam --nic-name VM1Nic2 --name VM1-ipconfig2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/fabrikampool"
    az vm create --resource-group contosofabrikam --name VM1 --location westcentralus --os-type linux --nic-names VM1Nic1,VM1Nic2  --vnet-name VNet1 --vnet-subnet-name Subnet1 --availability-set myAvailabilitySet --vm-size Standard_DS3_v2 --storage-account-name mystorageaccount1 --image-urn canonical:UbuntuServer:16.04.0-LTS:latest --admin-username <your username>  --admin-password <your password>
    ```

12. 為第二個 VM 重複步驟 10 至 11：

    ```azurecli
    az network public-ip create --resource-group contosofabrikam --location westcentralus --name myPublicIP2 --domain-name-label mypublicdns785 --allocation-method Dynamic
    az storage account create --location westcentralus --resource-group contosofabrikam --kind Storage --sku-name GRS mystorageaccount2
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM2Nic1
    az network nic create --resource-group contosofabrikam --location westcentralus --subnet-vnet-name myVnet --subnet-name mySubnet --name VM2Nic2 --ip-config-name VM2-ipconfig1 --public-ip-name myPublicIP2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/contosopool"
    az network nic ip-config create --resource-group contosofabrikam --nic-name VM2Nic2 --name VM2-ipconfig2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/fabrikampool"
    az vm create --resource-group contosofabrikam --name VM2 --location westcentralus --os-type linux --nic-names VM2Nic1,VM2Nic2 --vnet-name VNet1 --vnet-subnet-name Subnet1 --availability-set myAvailabilitySet --vm-size Standard_DS3_v2 --storage-account-name mystorageaccount2 --image-urn canonical:UbuntuServer:16.04.0-LTS:latest --admin-username <your username>  --admin-password <your password>
    ```

13. 最後，您必須將 DNS 資源記錄設定為指向 Load Balancer 的個別前端 IP 位址。 您可以在 Azure DNS 中裝載網域。 如需搭配使用 Azure DNS 與 Load Balancer 的詳細資訊，請參閱[使用 Azure DNS 搭配其他 Azure 服務](../dns/dns-for-azure-services.md)。

## <a name="next-steps"></a>後續步驟
- 若要深入了解如何在 Azure 中合併負載平衡服務，請參閱[在 Azure 中使用負載平衡服務](../traffic-manager/traffic-manager-load-balancing-azure.md)。
- 若要深入了解如何在 Azure 中使用不同類型的 記錄檔來管理負載平衡器和針對其問題進行疑難排解，請參閱 [Azure Load Balancer 的 Log Analytics](../load-balancer/load-balancer-monitor-log.md)。
