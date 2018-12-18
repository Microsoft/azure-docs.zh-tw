---
title: 使用 Azure CLI 在 Linux VM 上安裝 MongoDB | Microsoft Docs
description: 了解如何使用 Azure CLI 在 Linux 虛擬機器上安裝及設定 MongoDB
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: 3f55b546-86df-4442-9ef4-8a25fae7b96e
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: a6228290eb5f35d687c26042c87551471462ab85
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987827"
---
# <a name="how-to-install-and-configure-mongodb-on-a-linux-vm"></a>如何在 Linux VM 上安裝及設定 MongoDB

[MongoDB](http://www.mongodb.org) 是受歡迎的高效能開放原始碼 NoSQL 資料庫。 本文說明如何使用 Azure CLI 在 Linux VM 上安裝及設定 MongoDB。 範例會詳細說明如何︰

* [手動安裝及設定基本 MongoDB 執行個體](#manually-install-and-configure-mongodb-on-a-vm)
* [使用 Resource Manager 範本建立基本 MongoDB 執行個體](#create-basic-mongodb-instance-on-centos-using-a-template)
* [使用 Resource Manager 範本建立複雜的 MongoDB 分區化叢集與複本集](#create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template)


## <a name="manually-install-and-configure-mongodb-on-a-vm"></a>在 VM 上手動安裝及設定 MongoDB
MongoDB [提供 Linux 散發版本的安裝指示](https://docs.mongodb.com/manual/administration/install-on-linux/)，包括 Red Hat / CentOS、SUSE、Ubuntu 和 Debian。 下列範例會建立名為 CentOS 的 VM。 若要建立此環境，您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會建立名為 *myVM* 的 VM，其中具有使用 SSH 公開金鑰驗證、名為 *azureuser* 的使用者

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image CentOS \
    --admin-username azureuser \
    --generate-ssh-keys
```

使用您自己的使用者名稱和上一個步驟輸出中列出的 `publicIpAddress`，透過 SSH 連接到 VM：

```bash
ssh azureuser@<publicIpAddress>
```

若要新增 MongoDB 的安裝來源，請建立 yum 存放庫檔案，如下所示︰

```bash
sudo touch /etc/yum.repos.d/mongodb-org-3.6.repo
```

開啟 MongoDB 儲存機制檔案以進行編輯，例如使用 `vi` 或 `nano`。 加入下列幾行：

```sh
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

使用 yum 安裝 MongoDB，如下所示：

```bash
sudo yum install -y mongodb-org
```

依預設，會在可防止您存取 MongoDB 的 CentOS 映像上強制採用 SELinux。 安裝原則管理工具及設定 SELinux，以允許 MongoDB 在其預設 TCP 通訊埠 27017 上運作，如下所示：

```bash
sudo yum install -y policycoreutils-python
sudo semanage port -a -t mongod_port_t -p tcp 27017
```

啟動 MongoDB 服務，如下所示︰

```bash
sudo service mongod start
```

藉由使用本機 `mongo` 用戶端連線來確認 MongoDB 安裝︰

```bash
mongo
```

現在，新增一些資料然後進行搜尋，以測試 MongoDB 執行個體︰

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```

如有需要，設定 MongoDB 以在系統重新開機期間自動啟動：

```bash
sudo chkconfig mongod on
```


## <a name="create-basic-mongodb-instance-on-centos-using-a-template"></a>使用範本在 CentOS 上建立基本 MongoDB 執行個體
您可以使用下列來自 GitHub 的 Azure 快速入門範本，在單一 CentOS VM 上建立基本的 MongoDB 執行個體。 這個範本會使用 Linux 適用的自訂指令碼延伸模組將 yum 存放庫新增至您新建立的 CentOS VM，然後安裝 MongoDB。

* [CentOS 上的基本 MongoDB 執行個體](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-on-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json

若要建立此環境，您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。 首先，使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

接著，使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create) 來佈建 MongoDB 範本。 出現提示時，針對 newStorageAccountName、dnsNameForPublicIP和管理員使用者名稱與密碼輸入您自己唯一的值：

```azurecli
az group deployment create --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json
```

使用您 VM 的公用 DNS 位址來登入 VM。 您可以使用 [az vm show](/cli/azure/vm#az_vm_show) 來檢視公用 DNS 位址：

```azurecli
az vm show -g myResourceGroup -n myLinuxVM -d --query [fqdns] -o tsv
```

使用您自己的使用者名稱和公用 DNS 位址來透過 SSH 連線到 VM：

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

藉由使用本機 `mongo` 用戶端連線來確認 MongoDB 安裝，如下所示︰

```bash
mongo
```

現在，新增一些資料並進行搜尋來測試執行個體，如下所示︰

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```


## <a name="create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template"></a>使用範本在 CentOS 上建立複雜的 MongoDB 分區化叢集
您可以使用下列來自 GitHub 的 Azure 快速入門範本，建立複雜的 MongoDB 分區化叢集。 此範本遵循 [MongoDB 分區化叢集最佳作法](https://docs.mongodb.com/manual/core/sharded-cluster-components/)提供備援和高可用性。 範本會建立兩個分區，其中每個複本集中有三個節點。 還會建立具有三個節點的組態伺服器複本集，加上兩個 mongos 路由器伺服器，以提供跨分區應用程式的一致性。

* [CentOS 上的 MongoDB 分區化叢集](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json

> [!WARNING]
> 部署這個複雜的 MongoDB 分區化叢集需要超過 20 個核心，通常是每個訂用帳戶區域的預設核心計數。 開啟 Azure 支援要求，以增加核心計數。

若要建立此環境，您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。 首先，使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

接著，使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create) 來佈建 MongoDB 範本。 在需要的地方定義您自己的資源名稱與大小，例如 mongoAdminUsername、sizeOfDataDiskInGB 和 configNodeVmSize：

```azurecli
az group deployment create --resource-group myResourceGroup \
  --parameters '{"adminUsername": {"value": "azureuser"},
    "adminPassword": {"value": "P@ssw0rd!"},
    "mongoAdminUsername": {"value": "mongoadmin"},
    "mongoAdminPassword": {"value": "P@ssw0rd!"},
    "dnsNamePrefix": {"value": "mypublicdns"},
    "environment": {"value": "AzureCloud"},
    "numDataDisks": {"value": "4"},
    "sizeOfDataDiskInGB": {"value": 20},
    "centOsVersion": {"value": "7.0"},
    "routerNodeVmSize": {"value": "Standard_DS3_v2"},
    "configNodeVmSize": {"value": "Standard_DS3_v2"},
    "replicaNodeVmSize": {"value": "Standard_DS3_v2"},
    "zabbixServerIPAddress": {"value": "Null"}}' \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json \
  --name myMongoDBCluster \
  --no-wait
```

這項部署可能需要超過一小時的時間來部署及設定所有 VM 執行個體。 `--no-wait` 旗標是用在上述命令的結尾，可在 Azure 平台接受範本部署之後，將控制權交回給命令提示字元。 您可以接著使用 [az group deployment show](/cli/azure/group/deployment#az_group_deployment_show) 來檢視部署狀態。 下列範例會檢視 myResourceGroup 資源群組中 myMongoDBCluster 部署的狀態：

```azurecli
az group deployment show \
    --resource-group myResourceGroup \
    --name myMongoDBCluster \
    --query [properties.provisioningState] \
    --output tsv
```

## <a name="next-steps"></a>後續步驟
在這些範例中，您會從 VM 本機連線到 MongoDB 執行個體。 如果您想要從另一個 VM 或網路連線到 MongoDB 執行個體，請確定建立適當的[網路安全性群組規則](nsg-quickstart.md)。

這些範例會部署核心 MongoDB 環境以用於開發用途。 請為您的環境套用必要的安全性設定選項。 如需詳細資訊，請參閱 [MongoDB 安全性文件](https://docs.mongodb.com/manual/security/) \(英文\)。

如需關於建立範本的詳細資訊，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。

Azure Resource Manager 範本會使用自訂指令碼延伸模組，在您的 VM 上下載並執行指令碼。 如需詳細資訊，請參閱[搭配 Linux 虛擬機器使用 Azure 自訂指令碼擴充功能](extensions-customscript.md)。

