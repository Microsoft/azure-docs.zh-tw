---
title: 從 Azure Kubernetes Service 對 Azure Container Registry 進行驗證
description: 了解如何使用 Azure Active Directory 服務主體，提供從 Azure Kubernetes Service 對您私人容器登錄中的映像的存取。
services: container-service
author: mmacy
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 08/08/2018
ms.author: marsma
ms.openlocfilehash: c9ade4d61a1b95d5041a13f9436f0d02a7951758
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981642"
---
# <a name="authenticate-with-azure-container-registry-from-azure-kubernetes-service"></a>從 Azure Kubernetes Service 對 Azure Container Registry 進行驗證

當您搭配使用 Azure Container Service (AKS) 與 Azure Kubernetes Registry (ACR) 時，必須建立驗證機制。 本文詳細說明在這兩個 Azure 服務之間進行驗證所建議的組態。

## <a name="grant-aks-access-to-acr"></a>將 AKS 存取權授與 ACR

當您建立 AKS 叢集時，Azure 也會建立服務主體來支援其他 Azure 資源的叢集可操作性。 您可以使用此自動產生的服務主體，向 ACR 登錄進行驗證。 若要這麼做，您需要建立 Azure AD [角色指派](../role-based-access-control/overview.md#role-assignments)，授與叢集的服務主體對容器登錄的存取權。

使用以下指令碼，將 AKS 產生的服務主體存取權授與 Azure Container Registry。 請先為您的環境修改 `AKS_*` 和 `ACR_*` 變數，然後再執行指令碼。

```bash
#!/bin/bash

AKS_RESOURCE_GROUP=myAKSResourceGroup
AKS_CLUSTER_NAME=myAKSCluster
ACR_RESOURCE_GROUP=myACRResourceGroup
ACR_NAME=myACRRegistry

# Get the id of the service principal configured for AKS
CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)

# Get the ACR registry resource id
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)

# Create role assignment
az role assignment create --assignee $CLIENT_ID --role Reader --scope $ACR_ID
```

## <a name="access-with-kubernetes-secret"></a>使用 Kubernetes 祕密進行存取

在某些情況下，您可能無法將所需角色指派給自動產生的 AKS 服務主體，授與其存取 ACR 的權限。 例如，由於貴組織的安全性模型，您在 Azure AD 目錄中可能沒有足夠權限為 AKS 產生的服務主體指派角色。 在此情況下，您可以建立新的服務主體，然後使用 Kubernetes 映像提取密碼，授與其容器登錄的存取權。

使用下列指令碼來建立新的服務主體 (您將使用其認證進行 Kubernetes 映像提取祕密)。 請先為您的環境修改 `ACR_NAME` 變數，然後再執行指令碼。

```bash
#!/bin/bash

ACR_NAME=myacrinstance
SERVICE_PRINCIPAL_NAME=acr-service-principal

# Populate the ACR login server and resource id.
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

# Create a 'Reader' role assignment with a scope of the ACR resource.
SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --role Reader --scopes $ACR_REGISTRY_ID --query password --output tsv)

# Get the service principal client id.
CLIENT_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

# Output used when creating Kubernetes secret.
echo "Service principal ID: $CLIENT_ID"
echo "Service principal password: $SP_PASSWD"
```

您現在可以將服務主體認證儲存在 Kubernetes [映像提取密碼][image-pull-secret]中，而且您的 AKS 叢集會在執行容器時予以參考。

使用下列 **kubectl** 命令以建立 Kubernetes 密碼。 將 `<acr-login-server>` 取代為 Azure 容器登錄的完整格式名稱 (其格式為「acrname.azurecr.io」)。 將 `<service-principal-ID>` 和 `<service-principal-password>` 取代為執行先前指令碼獲取的值。 以任何正確格式的電子郵件地址取代 `<email-address>`。

```bash
kubectl create secret docker-registry acr-auth --docker-server <acr-login-server> --docker-username <service-principal-ID> --docker-password <service-principal-password> --docker-email <email-address>
```

您現在可以藉由使用 `imagePullSecrets` 參數指定其名稱，在 Pod 部署中使用 Kubernetes 密碼 (在此情況下為「acr-auth」)：

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: acr-auth-example
spec:
  template:
    metadata:
      labels:
        app: acr-auth-example
    spec:
      containers:
      - name: acr-auth-example
        image: myacrregistry.azurecr.io/acr-auth-example
      imagePullSecrets:
      - name: acr-auth
```

<!-- LINKS - external -->
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[image-pull-secret]: https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets
