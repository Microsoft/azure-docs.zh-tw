---
title: 從 Azure Container Registry 部署至 Azure 容器執行個體
description: 了解如何使用 Azure Container Registry 中的容器映像，在 Azure 容器執行個體中部署容器。
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: article
ms.date: 03/30/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: bbdf9a88c19e8006ffa9669b0c6d95d85506b256
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854451"
---
# <a name="deploy-to-azure-container-instances-from-azure-container-registry"></a>從 Azure Container Registry 部署至 Azure 容器執行個體

Azure Container Registry 是以 Azure 為基礎的私人登錄，用於裝載 Docker 容器映像。 本文說明如何將儲存於 Azure Container Registry 的容器映像部署至 Azure Container Instances。

## <a name="prerequisites"></a>必要條件

**Azure Container Registry**：您需要 Azure 容器登錄，且登錄中至少一個容器映像，才能完成本篇文章中的步驟。 如果您需要登錄，請參閱[使用 Azure CLI 建立容器登錄](../container-registry/container-registry-get-started-azure-cli.md)。

**Azure CLI**：本文中的命令列範例使用 [Azure CLI](/cli/azure/)，並使用 Bash 殼層適用的格式。 您可以在本機[安裝 Azure CLI](/cli/azure/install-azure-cli)，或使用 [Azure Cloud Shell][cloud-shell-bash]。

## <a name="configure-registry-authentication"></a>設定登錄驗證

在任何生產案例中，應該使用[服務主體](../container-registry/container-registry-auth-service-principal.md)提供 Azure 容器登錄的存取。 服務主體可讓您針對容器映像提供角色型存取控制。 例如，您可以設定服務主體具有僅限提取登錄的存取權。

本節中，您會建立 Azure 金鑰保存庫和服務主體，並將服務主體的認證儲存在保存庫中。

### <a name="create-key-vault"></a>建立金鑰保存庫

如果您在 [Azure Key Vault](/azure/key-vault/) 中還沒有保存庫，使用 Azure CLI 以下列命令建立一個。

將 `RES_GROUP` 變數更新為您將在其中建立金鑰保存庫的資源群組名稱，將 `ACR_NAME` 更新為容器登錄的名稱。 在 `AKV_NAME` 指定新金鑰保存庫的名稱。 保存庫名稱在 Azure 內必須是唯一的，長度介於 3 到 24 個英數字元之間，以字母開頭、以字母或數字作為結尾，且不可包含連續的連字號。

```azurecli
RES_GROUP=myresourcegroup # Resource Group name
ACR_NAME=myregistry       # Azure Container Registry registry name
AKV_NAME=mykeyvault       # Azure Key Vault vault name

az keyvault create -g $RES_GROUP -n $AKV_NAME
```

### <a name="create-service-principal-and-store-credentials"></a>建立服務主體並儲存認證

您現在需要建立服務主體，並將它的認證儲存在金鑰保存庫中。

下列命令使用 [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] 建立服務主體，以及使用 [az keyvault secret set][az-keyvault-secret-set] 將服務主體的**密碼**儲存在保存庫中。

```azurecli
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role reader \
                --query password \
                --output tsv)
```

在前面的命令中，`--role` 引數設定服務主體具有 reader 角色，授與主體僅限提取登錄的存取權。 若要同時授與發送和提取存取權，請將 `--role` 引數變更為 contributor。

接下來，在保存庫中儲存服務主體的 appId，也就是您傳遞給 Azure Container Registry 進行驗證的**使用者名稱**。

```azurecli
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

您已建立 Azure Key Vault，並在其中儲存兩個祕密：

* `$ACR_NAME-pull-usr`：服務主體識別碼，作為容器登錄的**使用者名稱**。
* `$ACR_NAME-pull-pwd`：服務主體密碼，作為容器登錄的**密碼**。

現在，當您或應用程式和服務從登錄提取映像時，您可以依名稱參考這些祕密。

## <a name="deploy-container-with-azure-cli"></a>使用 Azure CLI 部署容器

現在，服務主體認證儲存在 Azure Key Vault 祕密中，應用程式和服務可以使用它們來存取您的私人登錄。

執行下列 [az container create][az-container-create] 命令來部署容器執行個體。 此命令使用儲存在 Azure Key Vault 中的服務主體認證來向容器登錄驗證，並假設您已事先將 [aci-helloworld](container-instances-quickstart.md) 映像推送到登錄。 如果您想要使用不同於登錄的映像，請更新 `--image` 的值。

```azurecli
az container create \
    --name aci-demo \
    --resource-group $RES_GROUP \
    --image $ACR_NAME.azurecr.io/aci-helloworld:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label aci-demo-$RANDOM \
    --query ipAddress.fqdn
```

`--dns-name-label` 值在 Azure 內必須是唯一的，上述命令才能將一個隨機數字附加至容器的 DNS 名稱標籤。 命令的輸出會顯示容器的完整網域名稱 (FQDN)，例如：

```console
$ az container create --name aci-demo --resource-group $RES_GROUP --image $ACR_NAME.azurecr.io/aci-helloworld:v1 --registry-login-server $ACR_NAME.azurecr.io --registry-username $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-usr --query value -o tsv) --registry-password $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-pwd --query value -o tsv) --dns-name-label aci-demo-$RANDOM --query ipAddress.fqdn
"aci-demo-25007.eastus.azurecontainer.io"
```

一旦容器啟動成功，您可以在瀏覽器中瀏覽到容器的 FQDN，以確認應用程式順利執行。

## <a name="deploy-with-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本進行部署

您可以在容器群組定義中加入 `imageRegistryCredentials` 屬性，以在 Azure Resource Manager 範本中指定 Azure Container Registry 的屬性：

```JSON
"imageRegistryCredentials": [
  {
    "server": "imageRegistryLoginServer",
    "username": "imageRegistryUsername",
    "password": "imageRegistryPassword"
  }
]
```

如需參考 Resource Manager 範本中 Azure Key Vault 祕密的詳細資訊，請參閱[在部署期間使用 Azure Key Vault 傳遞安全的參數值](../azure-resource-manager/resource-manager-keyvault-parameter.md)。

## <a name="deploy-with-azure-portal"></a>使用 Azure 入口網站進行部署

如果您在 Azure Container Registry 中保有容器映像，您就可以使用 Azure 入口網站在 Azure Container Instances 中輕鬆地建立容器。

1. 在 Azure 入口網站中，瀏覽到您的容器登錄。

1. 選取 [存放庫]，接著選取您想要從中部署的存放庫、以滑鼠右鍵按一下您想要部署之容器映像的標籤，然後選取 [執行執行個體]。

    ![Azure 入口網站內 Azure Container Registry 中的 [執行執行個體]][acr-runinstance-contextmenu]

1. 分別為容器和資源群組輸入名稱。 如有需要，您也可以變更預設值。

    ![建立 Azure Container Instances 的功能表][acr-create-deeplink]

1. 部署完成之後，您可以從 [通知] 窗格瀏覽至容器群組，以尋找其 IP 位址和其他屬性。

    ![Azure Container Instances 容器群組的詳細資料檢視][aci-detailsview]

## <a name="next-steps"></a>後續步驟

如需有關 Azure Container Registry 驗證的詳細資訊，請參閱[向 Azure Container Registry 進行驗證](../container-registry/container-registry-authentication.md)。

<!-- IMAGES -->
[acr-create-deeplink]: ./media/container-instances-using-azure-container-registry/acr-create-deeplink.png
[aci-detailsview]: ./media/container-instances-using-azure-container-registry/aci-detailsview.png
[acr-runinstance-contextmenu]: ./media/container-instances-using-azure-container-registry/acr-runinstance-contextmenu.png

<!-- LINKS - External -->
[cloud-shell-bash]: https://shell.azure.com/bash
[cloud-shell-powershell]: https://shell.azure.com/powershell

<!-- LINKS - Internal -->
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-create]: /cli/azure/container#az-container-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set