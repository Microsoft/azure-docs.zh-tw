---
title: 教學課程 - 透過 Azure CLI 使用擴展集中的自訂 VM 映像 | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立可用於部署虛擬機器擴展集的自訂 VM 映像
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 5eee55846bd6f5821be1e40b969a35f5e50bd205
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967365"
---
# <a name="tutorial-create-and-use-a-custom-image-for-virtual-machine-scale-sets-with-the-azure-cli"></a>教學課程：使用 Azure CLI 建立及使用虛擬機器擴展集的自訂映像
當您建立擴展集時，您會指定部署 VM 執行個體時所要使用的映像。 若要減少部署 VM 執行個體後的工作數量，您可以使用自訂的 VM 映像。 此自訂 VM 映像包括任何必要的應用程式安裝或組態。 在擴展集中建立的任何 VM 執行個體都會使用自訂 VM 映像，並已可以處理您的應用程式流量。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立及自訂 VM
> * 取消佈建及一般化 VM
> * 建立自訂的 VM 映像
> * 部署使用自訂 VM 映像的擴展集

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，則在本教學課程中，您必須執行 Azure CLI 2.0.29 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。


## <a name="create-and-configure-a-source-vm"></a>建立並設定來源 VM

>[!NOTE]
> 本教學課程將逐步說明建立及使用一般化 VM 映像的程序。 不支援從特製化 VM 映像建立擴展集。

首先，請使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組，然後使用 [az vm create](/cli/azure/vm#az_vm_create)來建立 VM。 接著，此 VM 會用來當作自訂 VM 映像的來源。 下列範例會在名為 myResourceGroup 的資源群組中建立名為 myVM 的 VM：

```azurecli-interactive
az group create --name myResourceGroup --location eastus

az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image ubuntults \
  --admin-username azureuser \
  --generate-ssh-keys
```

VM 的公用 IP 位址會顯示在 [az vm create](/cli/azure/vm#az_vm_create)命令的輸出中。 透過 SSH 連線至 VM 的公用 IP 位址，如下所示：

```azurecli-interactive
ssh azureuser@<publicIpAddress>
```

若要自訂您的 VM，請安裝基本 Web 伺服器。 部署擴展集中的 VM 執行個體時，其中就會有執行 Web 應用程式所需的所有必要套件。 請依下列方式使用 `apt-get` 安裝 NGINX：

```bash
sudo apt-get install -y nginx
```

準備您的 VM 以作為自訂映像使用的最後一個步驟是取消佈建 VM。 此步驟會從 VM 移除機器專屬的資訊，並讓您能夠從單一映像部署多個 VM。 取消佈建 VM 時，主機名稱會重設為 localhost.localdomain。 SSH 主機金鑰、名稱伺服器設定、根密碼及快取的 DHCP 租用也會一併刪除。

若要取消佈建 VM，請使用 Azure VM 代理程式 (waagent)。 Azure VM 代理程式會安裝在每個 VM 上，用來與 Azure 平台通訊。 `-force` 參數會指示代理程式接受提示，以重設機器專屬的資訊。

```bash
sudo waagent -deprovision+user -force
```

關閉與 VM 的 SSH 連線：

```bash
exit
```


## <a name="create-a-custom-vm-image-from-the-source-vm"></a>從來源 VM 建立自訂 VM 映像
您現在可使用安裝的 Nginx Web 伺服器來自訂來源 VM。 讓我們來建立自訂 VM 映像，以搭配使用擴展集。

若要建立映像，必須解除配置 VM。 使用 [az vm deallocate](/cli//azure/vm#az_vm_deallocate) 解除配置 VM。 然後，使用 [az vm generalize](/cli//azure/vm#az_vm_generalize) 將 VM 的狀態設定為一般化，讓 Azure 平台知道 VM 已準備好要使用自訂映像。 您只能從一般化的 VM 建立映像：

```azurecli-interactive
az vm deallocate --resource-group myResourceGroup --name myVM
az vm generalize --resource-group myResourceGroup --name myVM
```

解除配置及一般化 VM 可能需要幾分鐘的時間。

使用 [az image create](/cli//azure/image#az_image_create) 建立 VM 的映像。 下列範例會從您的 VM 建立名為 myImage 的映像：

```azurecli-interactive
az image create \
  --resource-group myResourceGroup \
  --name myImage \
  --source myVM
```


## <a name="create-a-scale-set-from-the-custom-vm-image"></a>從自訂 VM 映像建立擴展集
使用 [az vmss create](/cli/azure/vmss#az-vmss-create) 建立擴展集。 請勿使用 UbuntuLTS 或 CentOS 這類平台映像，而是指定自訂 VM 映像的名稱。 下列範例會建立名為 myScaleSet 的擴展集，其使用前一步驟中名為 myImage 的自訂映像：

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image myImage \
  --admin-username azureuser \
  --generate-ssh-keys
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。


## <a name="test-your-scale-set"></a>測試您的擴展集
若要允許流量觸達擴展集，並確認 Web 伺服器正常運作，請使用 [az network lb rule create](/cli/azure/network/lb/rule#create) 建立負載平衡器規則。 下列範例會建立名為 myLoadBalancerRuleWeb 的規則，其會允許 TCP 連接埠 80 上的流量：

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

若要查看作用中的擴展集，可使用 [az network public-ip show](/cli/azure/network/public-ip#show) 取得負載平衡器的公用 IP 位址。 下列範例會取得建立為擴展集一部分的 myScaleSetLBPublicIP IP 位址︰

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

在您的 Web 瀏覽器中輸入公用 IP 位址。 預設 NGINX 網頁即會顯示，如下列範例所示：

![從自訂 VM 映像執行的 Nginx](media/tutorial-use-custom-image-cli/default-nginx-website.png)


## <a name="clean-up-resources"></a>清除資源
若要移除您的擴展集與其他資源，請使用 [az group delete](/cli/azure/group#az_group_delete) 刪除資源群組及其所有資源。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Azure CLI 來建立及使用擴展集的自訂 VM 映像：

> [!div class="checklist"]
> * 建立及自訂 VM
> * 取消佈建及一般化 VM
> * 建立自訂的 VM 映像
> * 部署使用自訂 VM 映像的擴展集

前往下一個教學課程，以了解如何將應用程式部署至擴展集。

> [!div class="nextstepaction"]
> [將應用程式部署至擴展集](tutorial-install-apps-cli.md)