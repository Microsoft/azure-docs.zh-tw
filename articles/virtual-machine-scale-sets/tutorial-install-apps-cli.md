---
title: 教學課程 - 使用 Azure CLI 在擴展集中安裝應用程式 | Microsoft Docs
description: 了解如何搭配使用 Azure CLI 與自訂指令碼延伸模組，將應用程式安裝到虛擬機器擴展集
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
ms.openlocfilehash: 21b85a3afa8e955661b3acbe5c7e4cb463a6d23f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986178"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-the-azure-cli"></a>教學課程 - 使用 Azure CLI 在虛擬機器擴展集中安裝應用程式
若要在擴展集的虛擬機器 (VM) 執行個體上執行應用程式，您需要先安裝應用程式元件和必要的檔案。 在先前的教學課程中，您已了解如何建立及使用自訂 VM 映像來部署您的 VM 執行個體。 此自訂映像已包含手動應用程式安裝和組態。 您也可以在部署好每個 VM 執行個體後，讓應用程式自動安裝到擴展集，或更新已在擴展集上執行的應用程式。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 自動將應用程式安裝到擴展集
> * 使用 Azure 自訂指令碼擴充功能
> * 更新在擴展集上執行的應用程式

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，則在本教學課程中，您必須執行 Azure CLI 2.0.29 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="what-is-the-azure-custom-script-extension"></a>Azure 自訂指令碼擴充功能是什麼？
自訂指令碼擴充功能會在 Azure VM 上下載並執行指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 您可以從 Azure 儲存體或 GitHub 下載指令碼，或是在擴充功能執行階段將指令碼提供給 Azure 入口網站。

自訂指令碼延伸模組會與 Azure Resource Manager 範本整合，而且可與 Azure CLI、Azure PowerShell、Azure 入口網站或 REST API 搭配使用。 如需詳細資訊，請參閱[自訂指令碼延伸模組概觀](../virtual-machines/linux/extensions-customscript.md)。

若要搭配使用自訂指令碼延伸模組與 Azure CLI，您可以建立 JSON 檔案來定義可取得的檔案及可執行的命令。 可跨擴展集部署重複使用這些 JSON 定義，以套用一致的應用程式安裝。


## <a name="create-custom-script-extension-definition"></a>建立自訂指令碼擴充功能的定義
若要查看作用中的自訂指令碼擴充功能，可建立擴展集來安裝 NGINX Web 伺服器，並輸出 VM 執行個體的主機名稱。 下列自訂指令碼擴充功能定義會從 GitHub 下載範例指令碼、安裝必要的套件，然後將 VM 執行個體主機名稱寫入基本的 HTML 頁面。

在您目前的殼層中，建立名為 *customConfig.json* 的檔案並貼上下列設定。 例如，在 Cloud Shell 中建立不在本機電腦上的檔案。 您可以使用任何您想要的編輯器。 在 Cloud Shell 中，輸入 `sensible-editor cloudConfig.json` 以建立檔案並查看可用的編輯器清單。

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],
  "commandToExecute": "./automate_nginx.sh"
}
```


## <a name="create-a-scale-set"></a>建立擴展集
使用 [az group create](/cli/azure/group#create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

現在使用 [az vmss create](/cli/azure/vmss#create) 建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集，以及產生 SSH 金鑰 (如果不存在)︰

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


## <a name="apply-the-custom-script-extension"></a>套用自訂指令碼擴充功能
使用 [az vmss extension set](/cli/azure/vmss/extension#set)，將自訂指令碼延伸組態套用到擴展集中的 VM 執行個體。 下列範例會將 customConfig.json 組態套用到 myResourceGroup 資源群組中的 myScaleSet VM 執行個體：

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings @customConfig.json
```

擴展集中的每個 VM 執行個體會下載並執行 GitHub 中的指令碼。 在更複雜的範例中，您可以安裝多個應用程式元件和檔案。 如果相應增加擴展集，新的 VM 執行個體就會自動套用相同的自訂指令碼擴充功能定義，並安裝必要的應用程式。


## <a name="test-your-scale-set"></a>測試您的擴展集
若要允許流量觸達 Web 伺服器，請使用 [az network lb rule create](/cli/azure/network/lb/rule#create) 建立負載平衡器規則。 下列範例會建立名為 myLoadBalancerRuleWeb 的規則：

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

若要查看作用中的 web 伺服器，可使用 [az network public-ip show](/cli/azure/network/public-ip#show) 取得負載平衡器的公用 IP 位址。 下列範例會取得建立作為擴展集一部分的 myScaleSetLBPublicIP IP 位址︰

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

將負載平衡器的公用 IP 位址輸入網頁瀏覽器中。 負載平衡器會將流量散發至您的其中一個 VM 執行個體，如下列範例所示：

![Nginx 中的基本網頁](media/tutorial-install-apps-cli/running-nginx.png)

讓網頁瀏覽器保持開啟，您就可以看到下一個步驟中的已更新版本。


## <a name="update-app-deployment"></a>更新應用程式部署
在擴展集的整個生命週期中，您可能需要部署更新版的應用程式。 透過自訂指令碼擴充功能，您可以參考已更新的部署指令碼，然後將擴充功能重新套用至擴展集。 在上一個步驟中建立擴展集時，`--upgrade-policy-mode` 已設定為「自動」。 此設定可讓擴展集中的 VM 執行個體自動更新並套用最新版的應用程式。

在您目前的殼層中，建立名為 customConfigv2.json 的檔案並貼上下列組態。 此定義會執行已更新的 v2 版應用程式安裝指令碼：

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx_v2.sh"],
  "commandToExecute": "./automate_nginx_v2.sh"
}
```

使用 [az vmss extension set](/cli/azure/vmss/extension#set)，再次將自訂指令碼擴充功能組態套用到擴展集中的 VM 執行個體。 customConfigv2.json 會用來套用更新版的應用程式：

```azurecli-interactive
az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroup \
    --vmss-name myScaleSet \
    --settings @customConfigv2.json
```

擴展集中的所有 VM 執行個體都會自動以最新版的範例網頁來進行更新。 若要查看更新後的版本，請重新整理您瀏覽器中的網站：

![Nginx 中的更新網頁](media/tutorial-install-apps-cli/running-nginx-updated.png)


## <a name="clean-up-resources"></a>清除資源
若要移除您的擴展集與其他資源，請使用 [az group delete](/cli/azure/group#az_group_delete) 刪除資源群組及其所有資源。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Azure CLI 自動安裝和更新擴展集上的應用程式：

> [!div class="checklist"]
> * 自動將應用程式安裝到擴展集
> * 使用 Azure 自訂指令碼擴充功能
> * 更新在擴展集上執行的應用程式

前往下一個教學課程，以了解如何自動調整擴展集。

> [!div class="nextstepaction"]
> [自動調整擴展集](tutorial-autoscale-cli.md)