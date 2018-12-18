---
title: 搭配 AKS 使用 Azure 檔案
description: 搭配 AKS 使用 Azure 磁碟
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 03/08/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: b35d0e33009f76e0b2d6f90c52c98ce5f317792d
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436755"
---
# <a name="volumes-with-azure-files"></a>包含 Azure 檔案的磁碟區

容器型應用程式常常需要存取和保存外部資料磁碟區中的資料。 Azure 檔案可用來作為這個外部資料存放區。 本文將詳細說明如何在 Azure Kubernetes Service 中使用 Azure 檔案作為 Kubernetes 磁碟區。

如需有關 Kubernetes 磁碟區的詳細資訊，請參閱 [Kubernetes 磁碟區][kubernetes-volumes]。

## <a name="create-an-azure-file-share"></a>建立 Azure 檔案共用

在使用 Azure 檔案共用作為 Kubernetes 磁碟區之前，您必須建立 Azure 儲存體帳戶與檔案共用。 下列指令碼可用來完成這些工作。 記下或更新參數值，建立 Kubernetes 磁碟區時需要其中一些參數值。

```azurecli-interactive
#!/bin/bash

# Change these four parameters
AKS_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
AKS_PERS_RESOURCE_GROUP=myAKSShare
AKS_PERS_LOCATION=eastus
AKS_PERS_SHARE_NAME=aksshare

# Create the Resource Group
az group create --name $AKS_PERS_RESOURCE_GROUP --location $AKS_PERS_LOCATION

# Create the storage account
az storage account create -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -l $AKS_PERS_LOCATION --sku Standard_LRS

# Export the connection string as an environment variable, this is used when creating the Azure file share
export AZURE_STORAGE_CONNECTION_STRING=`az storage account show-connection-string -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -o tsv`

# Create the file share
az storage share create -n $AKS_PERS_SHARE_NAME

# Get storage account key
STORAGE_KEY=$(az storage account keys list --resource-group $AKS_PERS_RESOURCE_GROUP --account-name $AKS_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv)

# Echo storage account name and key
echo Storage account name: $AKS_PERS_STORAGE_ACCOUNT_NAME
echo Storage account key: $STORAGE_KEY
```

## <a name="create-kubernetes-secret"></a>建立 Kubernetes 祕密

Kubernetes 必須要有認證才能存取檔案共用。 這些認證會儲存在 [Kubernetes 密碼][kubernetes-secret]，它會在建立 Kubernetes Pod 時參考。

使用下列命令來建立祕密。 請使用您的儲存體帳戶名稱來取代 `STORAGE_ACCOUNT_NAME`，並使用您的儲存體金鑰來取代 `STORAGE_ACCOUNT_KEY`。

```console
kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=STORAGE_ACCOUNT_KEY
```

## <a name="mount-file-share-as-volume"></a>將檔案共用掛接成磁碟區

藉由在 Pod 的規格中設定磁碟區，將「Azure 檔案服務」共用掛接到 Pod 中。建立一個名為 `azure-files-pod.yaml` 且含有下列內容的新檔案。 將 `aksshare` 更新成提供給「Azure 檔案」共用的名稱。

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: azure-files-pod
spec:
 containers:
  - image: microsoft/sample-aks-helloworld
    name: azure
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
 volumes:
  - name: azure
    azureFile:
      secretName: azure-secret
      shareName: aksshare
      readOnly: false
```

使用 kubectl 來建立 Pod。

```azurecli-interactive
kubectl apply -f azure-files-pod.yaml
```

您現在有一個 Azure 檔案共用掛接在 `/mnt/azure` 目錄中的執行中容器。  您可以在透過 `kubectl describe pod azure-files-pod`檢查 Pod 時，查看磁碟區掛接。

## <a name="next-steps"></a>後續步驟

使用「Azure 檔案」來深入了解 Kubernetes 磁碟區。

> [!div class="nextstepaction"]
> [適用於 Azure 檔案的 Kubernetes 外掛程式][kubernetes-files] \(英文\)

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/user-guide/kubectl/v1.8/#create
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-storage-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-key-list]: /cli/azure/storage/account/keys#az-storage-account-keys-list
[az-storage-share-create]: /cli/azure/storage/share#az-storage-share-create
