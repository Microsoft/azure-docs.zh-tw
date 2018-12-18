---
title: 快速入門 - 使用 Azure CLI 建立虛擬機器擴展集 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 快速建立虛擬機器縮放
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: quickstart
ms.custom: mvc
ms.date: 03/27/18
ms.author: cynthn
ms.openlocfilehash: 348074592da8465762b178b49af4713aadcc8c8d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970649"
---
# <a name="quickstart-create-a-virtual-machine-scale-set-with-the-azure-cli"></a>快速入門：使用 Azure CLI 建立虛擬機器擴展集
虛擬機器擴展集可讓您部署和管理一組相同、自動調整的虛擬機器。 您可以手動調整擴展集中的 VM 數目，或定義規則以根據 CPU、記憶體需求或網路流量等資源使用量進行自動調整。 其後，Azure 負載平衡器會將流量分配到擴展集中的多個 VM 執行個體。 在本快速入門中，您會使用 Azure CLI 建立虛擬機器擴展集，並部署範例應用程式。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，則在本教學課程中，您必須執行 Azure CLI 2.0.29 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="create-a-scale-set"></a>建立擴展集
請先使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組，才可以建立擴展集。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

現在使用 [az vmss create](/cli/azure/vmss#az_vmss_create) 建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集，其已設定為在套用變更時自動更新，並在 ~/.ssh/id_rsa 中沒有 SSH 金鑰時產生 SSH 金鑰。 如果您需要登入 VM 執行個體，就會用到這些 SSH 金鑰。 若要使用現有的 SSH 金鑰組，請改為使用 `--ssh-key-value` 參數，並指定您的金鑰所在位置。

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。


## <a name="deploy-sample-application"></a>部署範例應用程式
若要測試您的擴展集，請安裝基本的 Web 應用程式。 您可以使用 Azure 自訂指令碼擴充功能來下載及執行會在 VM 執行個體上安裝應用程式的指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 如需詳細資訊，請參閱[自訂指令碼延伸模組概觀](../virtual-machines/linux/extensions-customscript.md)。

使用自訂指令碼擴充功能安裝基本的 NGINX Web 伺服器。 使用 [az vmss extension set](/cli/azure/vmss/extension#az_vmss_extension_set) 套用安裝 NGINX 的自訂指令碼擴充功能，如下所示：

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'
```


## <a name="allow-traffic-to-application"></a>允許流量流向應用程式
建立擴展集後，系統會自動部署 Azure 負載平衡器。 負載平衡器會將流量分配到擴展集中的多個 VM 執行個體。 若要允許流量觸達範例 Web 應用程式，請使用 [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create) 建立負載平衡器規則。 下列範例會建立名為 myLoadBalancerRuleWeb 的規則：

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```


## <a name="test-your-scale-set"></a>測試您的擴展集
若要查看有效的擴展集，請在網頁瀏覽器中存取範例 Web 應用程式。 使用 [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show) 取得負載平衡器的公用 IP 位址。 下列範例會取得建立作為擴展集一部分的 myScaleSetLBPublicIP IP 位址︰

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query '[ipAddress]' \
  --output tsv
```

將負載平衡器的公用 IP 位址輸入網頁瀏覽器中。 負載平衡器會將流量散發至您的其中一個 VM 執行個體，如下列範例所示：

![NGINX 中的預設網頁](media/virtual-machine-scale-sets-create-cli/running-nginx-site.png)


## <a name="clean-up-resources"></a>清除資源
若不再需要，您可以使用 [az group delete](/cli/azure/group#az_group_delete) 移除資源群組、擴展集和所有相關資源，如下所示。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>後續步驟
在此快速入門中，您建立了基本的擴展集，並使用自訂指令碼擴充功能，在 VM 執行個體上安裝基本的 NGINX Web 伺服器。 若要深入了解，請繼續執行有關於如何建立和管理 Azure 虛擬機器擴展集的教學課程。

> [!div class="nextstepaction"]
> [建立和管理 Azure 虛擬機器擴展集](tutorial-create-and-manage-cli.md)

