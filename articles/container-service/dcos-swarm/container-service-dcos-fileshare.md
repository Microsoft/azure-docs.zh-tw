---
title: Azure DC/OS 叢集的檔案共用
description: 建立檔案共用並將它掛接到 Azure Container Service 中的 DC/OS 叢集
services: container-service
author: julienstroheker
manager: dcaro
ms.service: container-service
ms.topic: tutorial
ms.date: 06/07/2017
ms.author: juliens
ms.custom: mvc
ms.openlocfilehash: 4e03a0b450c9806edfb81a867fba97052659ec44
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973491"
---
# <a name="create-and-mount-a-file-share-to-a-dcos-cluster"></a>建立檔案共用並將它掛接到 DC/OS 叢集

本教學課程詳細說明如何在 Azure 上建立檔案共用，並將它掛接到 DC/OS 叢集的每個代理程式和主機。 設定檔案共用能使在整個叢集上共用檔案 (例如設定、存取、記錄等等) 變得更容易。 在本教學課程中，會完成下列工作：

> [!div class="checklist"]
> * 建立 Azure 儲存體帳戶
> * 建立檔案共用
> * 在 DC/OS 叢集中掛接共用

您需要 ACS DC/OS 叢集，才能完成本教學課程中的步驟。 如有需要，[此指令碼範例](./../kubernetes/scripts/container-service-cli-deploy-dcos.md)可為您建立一個叢集。

本教學課程需要 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-file-share-on-microsoft-azure"></a>在 Microsoft Azure 上建立檔案共用

搭配使用 Azure 檔案共用與 DC/OS ACS 叢集之前，您必須建立儲存體帳戶和檔案共用。 執行下列指令碼可建立存放區和檔案共用。 使用您環境中的來更新參數。

```azurecli-interactive
# Change these four parameters
DCOS_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
DCOS_PERS_RESOURCE_GROUP=myResourceGroup
DCOS_PERS_LOCATION=eastus
DCOS_PERS_SHARE_NAME=dcosshare

# Create the storage account with the parameters
az storage account create -n $DCOS_PERS_STORAGE_ACCOUNT_NAME -g $DCOS_PERS_RESOURCE_GROUP -l $DCOS_PERS_LOCATION --sku Standard_LRS

# Export the connection string as an environment variable, this is used when creating the Azure file share
export AZURE_STORAGE_CONNECTION_STRING=`az storage account show-connection-string -n $DCOS_PERS_STORAGE_ACCOUNT_NAME -g $DCOS_PERS_RESOURCE_GROUP -o tsv`

# Create the share
az storage share create -n $DCOS_PERS_SHARE_NAME
```

## <a name="mount-the-share-in-your-cluster"></a>在叢集中掛接共用

接下來，必須在您叢集內的每部虛擬機器上掛接檔案共用。 已使用 cifs 工具/通訊協定完成這項工作。 可在叢集的每個節點上以手動方式完成掛接作業，或針對叢集中的每個節點執行指令碼完成掛接作業。

在此範例中，會執行兩個指令碼，其中一個掛接 Azure 檔案共用，第二個是在 DC/OS 叢集的每個節點上執行這個指令碼。

首先，需要 Azure 儲存體帳戶名稱和存取金鑰。 執行下列命令來取得此資訊。 請將每一個記下，稍後步驟中將會使用這些值。

儲存體帳戶名稱︰

```azurecli-interactive
STORAGE_ACCT=$(az storage account list --resource-group $DCOS_PERS_RESOURCE_GROUP --query "[?contains(name, '$DCOS_PERS_STORAGE_ACCOUNT_NAME')].[name]" -o tsv)
echo $STORAGE_ACCT
```

儲存體帳戶存取金鑰︰

```azurecli-interactive
az storage account keys list --resource-group $DCOS_PERS_RESOURCE_GROUP --account-name $STORAGE_ACCT --query "[0].value" -o tsv
```

接下來，取得 DC/OS 主機的 FQDN，並將它儲存在變數中。

```azurecli-interactive
FQDN=$(az acs list --resource-group $DCOS_PERS_RESOURCE_GROUP --query "[0].masterProfile.fqdn" --output tsv)
```

將您的私密金鑰複製到主要節點。 必須擁有此金鑰，才能建立 ssh 與叢集中所有節點的連線。 如果建立叢集時已使用非預設值，請更新使用者名稱。 

```azurecli-interactive
scp ~/.ssh/id_rsa azureuser@$FQDN:~/.ssh
```

建立 SSH 與以 DC/OS 為基礎的叢集主機 (或主要主機) 的連線。 如果建立叢集時已使用非預設值，請更新使用者名稱。

```azurecli-interactive
ssh azureuser@$FQDN
```

建立名為 **cifsMount.sh** 的檔案，並將下列內容複製到其中。 

此指令碼會用來裝載 Azure 檔案共用。 使用稍早收集到的資訊來更新 `STORAGE_ACCT_NAME` 和 `ACCESS_KEY` 變數。

```azurecli-interactive
#!/bin/bash

# Azure storage account name and access key
SHARE_NAME=dcosshare
STORAGE_ACCT_NAME=mystorageaccount
ACCESS_KEY=mystorageaccountKey

# Install the cifs utils, should be already installed
sudo apt-get update && sudo apt-get -y install cifs-utils

# Create the local folder that will contain our share
if [ ! -d "/mnt/share/$SHARE_NAME" ]; then sudo mkdir -p "/mnt/share/$SHARE_NAME" ; fi

# Mount the share under the previous local folder created
sudo mount -t cifs //$STORAGE_ACCT_NAME.file.core.windows.net/$SHARE_NAME /mnt/share/$SHARE_NAME -o vers=3.0,username=$STORAGE_ACCT_NAME,password=$ACCESS_KEY,dir_mode=0777,file_mode=0777
```
建立名為 **getNodesRunScript.sh** 的第二個檔案，並將下列內容複製到檔案中。 

此指令碼會探索所有叢集節點，然後再執行 **cifsMount.sh** 指令碼，以將檔案共用掛接在每個叢集節點上。

```azurecli-interactive
#!/bin/bash

# Install jq used for the next command
sudo apt-get install jq -y

# Get the IP address of each node using the mesos API and store it inside a file called nodes
curl http://leader.mesos:1050/system/health/v1/nodes | jq '.nodes[].host_ip' | sed 's/\"//g' | sed '/172/d' > nodes

# From the previous file created, run our script to mount our share on each node
cat nodes | while read line
do
  ssh `whoami`@$line -o StrictHostKeyChecking=no < ./cifsMount.sh
  done
```

執行指令碼可在叢集的所有節點上掛接 Azure 檔案共用。

```azurecli-interactive
sh ./getNodesRunScript.sh
```  

現在可在叢集每個節點上的 `/mnt/share/dcosshare` 存取檔案共用。

## <a name="next-steps"></a>後續步驟

本教學課程中，Azure 檔案共用已使用下列步驟開放供 DC/OS 叢集使用：

> [!div class="checklist"]
> * 建立 Azure 儲存體帳戶
> * 建立檔案共用
> * 在 DC/OS 叢集中掛接共用

請前往下一個教學課程，了解如何在 Azure 中將 Azure Container Registry 與 DC/OS 進行整合。  

> [!div class="nextstepaction"]
> [負載平衡應用程式](container-service-dcos-acr.md)
