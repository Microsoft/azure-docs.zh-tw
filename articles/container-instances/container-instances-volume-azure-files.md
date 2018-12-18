---
title: 在 Azure 容器執行個體中掛接 Azure 檔案磁碟區
description: 了解如何掛接 Azure 檔案磁碟區來保存 Azure 容器執行個體的狀態
services: container-instances
author: seanmck
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 02/20/2018
ms.author: seanmck
ms.custom: mvc
ms.openlocfilehash: 83c86d8310aff80f148e878261ba33b01846006b
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39441318"
---
# <a name="mount-an-azure-file-share-in-azure-container-instances"></a>在 Azure 容器執行個體中掛接 Azure 檔案共用

根據預設，Azure 容器執行個體都是無狀態的。 如果容器損毀或停止，其所有狀態都會遺失。 若要在容器超過存留期後保存其狀態，您必須從外部存放區掛接磁碟區。 本文說明如何掛接 Azure 檔案共用以便與 Azure 容器執行個體搭配使用。

> [!NOTE]
> 目前只有 Linux 容器才能掛接 Azure 檔案共用。 雖然我們致力於將所有功能帶入 Windows 容器，但是您可以在 [Azure 容器執行個體配額和區域可用性](container-instances-quotas.md)中找到目前的平台差異。

## <a name="create-an-azure-file-share"></a>建立 Azure 檔案共用

在搭配使用 Azure 檔案共用與 Azure 容器執行個體前，您必須先建立 Azure 檔案共用。 請執行下列指令碼來建立儲存體帳戶，以裝載檔案共用和共用本身。 儲存體帳戶名稱必須為全域獨有，指令碼才能為基礎字串加上隨機值。

```azurecli-interactive
# Change these four parameters as needed
ACI_PERS_RESOURCE_GROUP=myResourceGroup
ACI_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
ACI_PERS_LOCATION=eastus
ACI_PERS_SHARE_NAME=acishare

# Create the storage account with the parameters
az storage account create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --location $ACI_PERS_LOCATION \
    --sku Standard_LRS

# Export the connection string as an environment variable. The following 'az storage share create' command
# references this environment variable when creating the Azure file share.
export AZURE_STORAGE_CONNECTION_STRING=`az storage account show-connection-string --resource-group $ACI_PERS_RESOURCE_GROUP --name $ACI_PERS_STORAGE_ACCOUNT_NAME --output tsv`

# Create the file share
az storage share create -n $ACI_PERS_SHARE_NAME
```

## <a name="get-storage-credentials"></a>取得儲存體認證

若要在 Azure 容器執行個體中掛接 Azure 檔案共用來作為磁碟區，您需要三個值：儲存體帳戶名稱、共用名稱和儲存體存取金鑰。

如果您使用上述指令碼，所建立的儲存體帳戶名稱尾端會加上隨機值。 若要查詢最終字串 (包括隨機部分)，請使用下列命令：

```azurecli-interactive
STORAGE_ACCOUNT=$(az storage account list --resource-group $ACI_PERS_RESOURCE_GROUP --query "[?contains(name,'$ACI_PERS_STORAGE_ACCOUNT_NAME')].[name]" --output tsv)
echo $STORAGE_ACCOUNT
```

我們已知道共用名稱 (在上述指令碼中定義為 acishare)，因此只差儲存體帳戶金鑰，此金鑰可以使用下列命令來找到：

```azurecli-interactive
STORAGE_KEY=$(az storage account keys list --resource-group $ACI_PERS_RESOURCE_GROUP --account-name $STORAGE_ACCOUNT --query "[0].value" --output tsv)
echo $STORAGE_KEY
```

## <a name="deploy-container-and-mount-volume"></a>部署容器和掛接磁碟區

若要掛接 Azure 檔案共用以作為容器中的磁碟區，請在您使用 [az container create][az-container-create] 建立容器時，指定共用和磁碟區掛接點。 如果您已遵循上述步驟，您可以使用下列命令來掛接先前建立的共用並建立容器：

```azurecli-interactive
az container create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name hellofiles \
    --image microsoft/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $ACI_PERS_SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/
```

`--dns-name-label` 值在您建立容器執行個體所在的 Azure 區域中必須是唯一的。 如果您在執行命令時收到 **DNS 名稱標籤**錯誤訊息，請更新上方命令中的值。

## <a name="manage-files-in-mounted-volume"></a>管理已掛接磁碟區中的檔案

容器啟動後，即可使用透過 [microsoft/aci-hellofiles][aci-hellofiles] 映像部署的簡單 Web 應用程式，管理位於指定掛接路徑之 Azure 檔案共用中的檔案。 使用 [az container show][az-container-show] 命令取得 Web 應用程式的完整網域名稱 (FQDN)：

```azurecli-interactive
az container show --resource-group $ACI_PERS_RESOURCE_GROUP --name hellofiles --query ipAddress.fqdn
```

您可以使用 [Azure 入口網站][portal]或 [Microsoft Azure 儲存體總管][storage-explorer]之類的工具擷取和檢查寫入至檔案共用的檔案。

## <a name="mount-multiple-volumes"></a>掛接多個磁碟區

若要在容器執行個體中掛接多個磁碟區，您必須使用 [Azure Resource Manager 範本](/azure/templates/microsoft.containerinstance/containergroups)進行部署。

首先，提供共用詳細資料，並填入範本之 `properties` 區段中的 `volumes` 陣列來定義磁碟區。 例如，如果您已在儲存體帳戶 *myStorageAccount* 中建立名為 *share1* 和 *share2* 的兩個 Azure 檔案共用，則會顯示與下列類似的 `volumes` 陣列：

```json
"volumes": [{
  "name": "myvolume1",
  "azureFile": {
    "shareName": "share1",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
},
{
  "name": "myvolume2",
  "azureFile": {
    "shareName": "share2",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
}]
```

接下來，針對您想要掛接磁碟區所在容器群組中的每個容器，填入容器定義之 `properties` 區段中的 `volumeMounts` 陣列。 例如，這會掛接兩個先前定義的磁碟區：*myvolume1* 和 *myvolume2*：

```json
"volumeMounts": [{
  "name": "myvolume1",
  "mountPath": "/mnt/share1/"
},
{
  "name": "myvolume2",
  "mountPath": "/mnt/share2/"
}]
```

若要使用 Azure Resource Manager 範本來查看容器執行個體部署範例，請參閱[在 Azure 容器執行個體中部署多個容器群組](container-instances-multi-container-group.md)。

## <a name="next-steps"></a>後續步驟

了解如何在 Azure 容器執行個體中掛接其他類型的磁碟區：

* [在 Azure 容器執行個體中掛接 emptyDir 磁碟區](container-instances-volume-emptydir.md)
* [在 Azure 容器執行個體中掛接 gitRepo 磁碟區](container-instances-volume-gitrepo.md)
* [在 Azure 容器執行個體中掛接秘密磁碟區](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-hellofiles]: https://hub.docker.com/r/microsoft/aci-hellofiles/
[portal]: https://portal.azure.com
[storage-explorer]: https://storageexplorer.com

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show