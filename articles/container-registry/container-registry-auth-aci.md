---
title: 使用來自 Azure 容器執行個體的 Azure Container Registry 進行驗證
description: 了解如何使用 Azure Active Directory 服務主體，從 Azure 容器執行個體提供私人容器登錄中映像的存取權。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: article
ms.date: 04/23/2018
ms.author: danlep
ms.openlocfilehash: 32b1788d73e1c323d93b40b778bc64a1ba45c4ad
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855828"
---
# <a name="authenticate-with-azure-container-registry-from-azure-container-instances"></a>使用來自 Azure 容器執行個體的 Azure Container Registry 進行驗證

您可以使用 Azure Active Directory (Azure AD) 服務主體，來提供 Azure Container Registry 中私人容器登錄的存取權。

在本文中，您將了解如何建立及設定 Azure AD 服務主體，並使其具備您登錄的「提取」權限。 然後，您會使用服務主體進行驗證，來啟動 Azure 容器執行個體 (ACI) 中的容器，以從您的私人登錄中提取其映像。

## <a name="when-to-use-a-service-principal"></a>何時使用服務主體

您應該在**無周邊案例**中從 ACI 使用服務主體進行驗證，例如，在以自動化或其他自動方式建立容器執行個體的應用程式或服務中。

例如，如果您有每晚都會執行的自動化指令碼，並建立一個[以工作為基礎的容器執行個體](../container-instances/container-instances-restart-policy.md)來處理某些資料，它可以使用具有僅提取 (讀取器) 權限的服務主體來向登錄進行驗證。 您接著可以替換服務主體的認證，或完全撤銷其存取權，而不會影響其他服務和應用程式。

停用登錄[管理使用者](container-registry-authentication.md#admin-account)時，也應使用服務主體。

[!INCLUDE [container-registry-service-principal](../../includes/container-registry-service-principal.md)]

## <a name="authenticate-using-the-service-principal"></a>使用服務主體進行驗證

若要在 Azure 容器執行個體中使用服務主體來啟動容器，請針對 `--registry-username` 指定它的識別碼，並針對 `--registry-password` 指定它的密碼。

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerregistry.azurecr.io/myimage:v1 \
    --registry-login-server mycontainerregistry.azurecr.io \
    --registry-username <service-principal-ID> \
    --registry-password <service-principal-password>
```

## <a name="sample-scripts"></a>範例指令碼

您可以找到前述適用於 GitHub 上的 Azure CLI 的範例指令碼，以及適用於 Azure PowerShell 的版本：

* [Azure CLI][acr-scripts-cli]
* [Azure PowerShell][acr-scripts-psh]

## <a name="next-steps"></a>後續步驟

下列文章包含運用服務主體與 ACR 的其他詳細資料：

* [使用服務主體進行 Azure Container Registry 驗證](container-registry-auth-service-principal.md)
* [從 Azure Kubernetes Service 對 Azure Container Registry 進行驗證 (AKS)](container-registry-auth-aks.md)

<!-- IMAGES -->

<!-- LINKS - External -->
[acr-scripts-cli]: https://github.com/Azure/azure-docs-cli-python-samples/tree/master/container-registry
[acr-scripts-psh]: https://github.com/Azure/azure-docs-powershell-samples/tree/master/container-registry

<!-- LINKS - Internal -->
