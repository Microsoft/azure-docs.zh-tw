---
title: 使用 Azure CLI 選取 Linux VM 映像 | Microsoft Docs
description: 了解如何使用 Azure CLI 來判斷發行者、供應項目、SKU 和 Marketplace VM 映像的版本。
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 7a858e38-4f17-4e8e-a28a-c7f801101721
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/28/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 33ffb9d1685f3d76e884ae0d90545f659b5ec87c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953340"
---
# <a name="how-to-find-linux-vm-images-in-the-azure-marketplace-with-the-azure-cli"></a>如何使用 Azure CLI 在 Azure Marketplace 中尋找 Linux VM 映像
本主題描述如何在 Azure Marketplace 中使用 Azure CLI 尋找 VM 映像。 當您使用 CLI、Resource Manager 範本或其他工具以程式設計方式建立 VM 時，使用此資訊來指定 Marketplace 映像。

也請透過 [Azure Marketplace](https://azuremarketplace.microsoft.com/) 店面、[Azure 入口網站](https://portal.azure.com)或 [Azure PowerShell](../windows/cli-ps-findimage.md) 瀏覽可用的映像和供應項目。 

請確定您已安裝最新的 [Azure CLI](/cli/azure/install-azure-cli) 並登入 Azure 帳戶 (`az login`)。

[!INCLUDE [virtual-machines-common-image-terms](../../../includes/virtual-machines-common-image-terms.md)]

## <a name="list-popular-images"></a>列出常用的映像

執行 [az vm image list](/cli/azure/vm/image#az_vm_image_list) 命令，而不包含 `--all` 選項，以查看 Azure Marketplace 中的常用 VM 映像清單。 例如，執行下列命令，以資料表格式顯示常用映像的快取清單：

```azurecli
az vm image list --output table
```

輸出會包含映像 URN ([Urn] 欄中的值)。 使用其中一個常用 Marketplace 映像建立 VM 時，您也可以指定 UrnAlias，例如 UbuntuLTS 的縮短格式。

```
You are viewing an offline list of images, use --all to retrieve an up-to-date list
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
CentOS         OpenLogic               7.3                 OpenLogic:CentOS:7.3:latest                                     CentOS               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
openSUSE-Leap  SUSE                    42.2                SUSE:openSUSE-Leap:42.2:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7.3                 RedHat:RHEL:7.3:latest                                          RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
...
```

## <a name="find-specific-images"></a>尋找特定映像

若要在 Marketplace 中尋找特定 VM 映像，請使用 `az vm image list` 命令搭配 `--all` 選項。 這個版本的命令需要一些時間才能完成，而且可能會傳回冗長的輸出，因此您通常會依 `--publisher` 或其他參數篩選清單。 

例如，以下命令會顯示所有的 Debian 供應項目 (請記住，如果沒有 `--all` 參數，則只會搜尋通用映像的本機快取)：

```azurecli
az vm image list --offer Debian --all --output table 

```

部分輸出： 
```
Offer    Publisher    Sku                Urn                                              Version
-------  -----------  -----------------  -----------------------------------------------  --------------
Debian   credativ     7                  credativ:Debian:7:7.0.201602010                  7.0.201602010
Debian   credativ     7                  credativ:Debian:7:7.0.201603020                  7.0.201603020
Debian   credativ     7                  credativ:Debian:7:7.0.201604050                  7.0.201604050
Debian   credativ     7                  credativ:Debian:7:7.0.201604200                  7.0.201604200
Debian   credativ     7                  credativ:Debian:7:7.0.201606280                  7.0.201606280
Debian   credativ     7                  credativ:Debian:7:7.0.201609120                  7.0.201609120
Debian   credativ     7                  credativ:Debian:7:7.0.201611020                  7.0.201611020
Debian   credativ     8                  credativ:Debian:8:8.0.201602010                  8.0.201602010
Debian   credativ     8                  credativ:Debian:8:8.0.201603020                  8.0.201603020
Debian   credativ     8                  credativ:Debian:8:8.0.201604050                  8.0.201604050
Debian   credativ     8                  credativ:Debian:8:8.0.201604200                  8.0.201604200
Debian   credativ     8                  credativ:Debian:8:8.0.201606280                  8.0.201606280
Debian   credativ     8                  credativ:Debian:8:8.0.201609120                  8.0.201609120
Debian   credativ     8                  credativ:Debian:8:8.0.201611020                  8.0.201611020
Debian   credativ     8                  credativ:Debian:8:8.0.201701180                  8.0.201701180
Debian   credativ     8                  credativ:Debian:8:8.0.201703150                  8.0.201703150
Debian   credativ     8                  credativ:Debian:8:8.0.201704110                  8.0.201704110
Debian   credativ     8                  credativ:Debian:8:8.0.201704180                  8.0.201704180
Debian   credativ     8                  credativ:Debian:8:8.0.201706190                  8.0.201706190
Debian   credativ     8                  credativ:Debian:8:8.0.201706210                  8.0.201706210
Debian   credativ     8                  credativ:Debian:8:8.0.201708040                  8.0.201708040
...
```

使用 `--location`、`--publisher` 和 `--sku` 選項套用類似的篩選條件。 您甚至可以執行篩選的部份相符，例如搜尋 `--offer Deb` 以尋找所有 Debian 映像。

如果您未使用 `--location` 選項指定特定的位置，就會傳回預設位置的值。 (執行 `az configure --defaults location=<location>` 以設定不同的預設位置。)

例如，下列命令會列出西歐位置中所有的 Debian 8 個 SKU：

```azurecli
az vm image list --location westeurope --offer Deb --publisher credativ --sku 8 --all --output table
```

部分輸出：

```
Offer    Publisher    Sku                Urn                                              Version
-------  -----------  -----------------  -----------------------------------------------  -------------
Debian   credativ     8                  credativ:Debian:8:8.0.201602010                  8.0.201602010
Debian   credativ     8                  credativ:Debian:8:8.0.201603020                  8.0.201603020
Debian   credativ     8                  credativ:Debian:8:8.0.201604050                  8.0.201604050
Debian   credativ     8                  credativ:Debian:8:8.0.201604200                  8.0.201604200
Debian   credativ     8                  credativ:Debian:8:8.0.201606280                  8.0.201606280
Debian   credativ     8                  credativ:Debian:8:8.0.201609120                  8.0.201609120
Debian   credativ     8                  credativ:Debian:8:8.0.201611020                  8.0.201611020
Debian   credativ     8                  credativ:Debian:8:8.0.201701180                  8.0.201701180
Debian   credativ     8                  credativ:Debian:8:8.0.201703150                  8.0.201703150
Debian   credativ     8                  credativ:Debian:8:8.0.201704110                  8.0.201704110
Debian   credativ     8                  credativ:Debian:8:8.0.201704180                  8.0.201704180
Debian   credativ     8                  credativ:Debian:8:8.0.201706190                  8.0.201706190
Debian   credativ     8                  credativ:Debian:8:8.0.201706210                  8.0.201706210
...
```

## <a name="navigate-the-images"></a>瀏覽映像 
要在位置中找到映像的另一個方法是在序列中執行 [az vm image list-publishers](/cli/azure/vm/image#az_vm_image_list_publishers)、[az vm image list-offers](/cli/azure/vm/image#az_vm_image_list_offers) 和 [az vm image list-skus](/cli/azure/vm/image#az_vm_image_list_skus) 命令。 您可以使用這些命令來判斷下列的值：

1. 列出映像發行者。
2. 針對指定的發行者，列出其供應項目。
3. 針對指定的供應項目，列出其 SKU。

然後，對於選取的 SKU，您可以選擇要部署的版本。

例如，下列命令會列出美國西部位置中的映像發行者：

```azurecli
az vm image list-publishers --location westus --output table
```

部分輸出：

```
Location    Name
----------  ----------------------------------------------------
westus      1e
westus      4psa
westus      7isolutions
westus      a10networks
westus      abiquo
westus      accellion
westus      Acronis
westus      Acronis.Backup
westus      actian_matrix
westus      actifio
westus      activeeon
westus      adatao
...
```
使用這項資訊從特定的發行者尋找供應項目。 例如，如果 Canonical 是位於美國西部的映像發行者，執行 `azure vm image list-offers` 可找到其供應項目。 傳遞位置和發行者，如下列範例所示：

```azurecli
az vm image list-offers --location westus --publisher Canonical --output table
```

輸出：

```
Location    Name
----------  -------------------------
westus      Ubuntu15.04Snappy
westus      Ubuntu15.04SnappyDocker
westus      UbunturollingSnappy
westus      UbuntuServer
westus      Ubuntu_Core
westus      Ubuntu_Snappy_Core
westus      Ubuntu_Snappy_Core_Docker
```
您看到在美國西部區域中，Canonical 在 Azure 上發佈 *UbuntuServer* 供應項目。 但是，是什麼 SKU？ 若要取得這些值，請執行 `azure vm image list-skus` 並設定您探索到的位置、發行者和供應項目：

```azurecli
az vm image list-skus --location westus --publisher Canonical --offer UbuntuServer --output table
```

輸出：

```
Location    Name
----------  -----------------
westus      12.04.3-LTS
westus      12.04.4-LTS
westus      12.04.5-DAILY-LTS
westus      12.04.5-LTS
westus      12.10
westus      14.04.0-LTS
westus      14.04.1-LTS
westus      14.04.2-LTS
westus      14.04.3-LTS
westus      14.04.4-LTS
westus      14.04.5-DAILY-LTS
westus      14.04.5-LTS
westus      16.04-beta
westus      16.04-DAILY-LTS
westus      16.04-LTS
westus      16.04.0-LTS
westus      16.10
westus      16.10-DAILY
westus      17.04
westus      17.04-DAILY
westus      17.10-DAILY
```

最後，使用 `az vm image list` 命令來尋找您需要的 SKU 特定版本，例如，*16.04-LTS*：

```azurecli
az vm image list --location westus --publisher Canonical --offer UbuntuServer --sku 16.04-LTS --all --output table
```

輸出：

```
Offer         Publisher    Sku        Urn                                               Version
------------  -----------  ---------  ------------------------------------------------  ---------------
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201611220  16.04.201611220
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201611300  16.04.201611300
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201612050  16.04.201612050
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201612140  16.04.201612140
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201612210  16.04.201612210
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201701130  16.04.201701130
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201702020  16.04.201702020
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201702200  16.04.201702200
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201702210  16.04.201702210
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201702240  16.04.201702240
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201703020  16.04.201703020
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201703030  16.04.201703030
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201703070  16.04.201703070
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201703270  16.04.201703270
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201703280  16.04.201703280
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201703300  16.04.201703300
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201705080  16.04.201705080
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201705160  16.04.201705160
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201706100  16.04.201706100
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201706191  16.04.201706191
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201707210  16.04.201707210
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201707270  16.04.201707270
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201708030  16.04.201708030
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201708110  16.04.201708110
UbuntuServer  Canonical    16.04-LTS  Canonical:UbuntuServer:16.04-LTS:16.04.201708151  16.04.201708151
```

現在，您可以記下 URN 值，精確地選擇想要使用的映像。 當您使用 [az vm create](/cli/azure/vm#az_vm_create) 命令建立 VM 時，請傳遞此值與 `--image` 參數。 請記住，您可以使用 "latest" 來取代 URN 中的版本號碼。 此版本一律為最新的映像版本。 

如果您使用 Resource Manager 範本來部署 VM，請在 `imageReference` 屬性中個別設定映像參數。 請參閱[範本參考](/azure/templates/microsoft.compute/virtualmachines)。

[!INCLUDE [virtual-machines-common-marketplace-plan](../../../includes/virtual-machines-common-marketplace-plan.md)]

### <a name="view-plan-properties"></a>檢視方案屬性
若要檢視映像的購買方案資訊，請執行 [az vm image show](/cli/azure/image#az_image_show) 命令。 如果輸出中的 `plan` 屬性不是 `null`，此映像具有您必須在以程式設計方式部署之前接受的條款。

例如，Canonical Ubuntu Server 16.04 LTS 映像沒有其他條款，因為 `plan` 資訊為 `null`：

```azurecli
az vm image show --location westus --publisher Canonical --offer UbuntuServer --sku 16.04-LTS --version 16.04.201801260
```

輸出：

```
{
  "dataDiskImages": [],
  "id": "/Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/16.04-LTS/Versions/16.04.201801260",
  "location": "westus",
  "name": "16.04.201801260",
  "osDiskImage": {
    "operatingSystem": "Linux"
  },
  "plan": null,
  "tags": null
}
```

針對 RabbitMQ Certified by Bitnami 映像執行類似的命令會顯示下列 `plan` 屬性：`name`、`product` 和 `publisher`。 (某些映像也有 `promotion code` 屬性。)若要部署此映像，請參閱下列各節，以接受條款並啟用以程式設計方式部署。

```azurecli
az vm image show --location westus --publisher bitnami --offer rabbitmq --sku rabbitmq --version 3.7.1807171506
```
輸出：

```
{
  "dataDiskImages": [],
  "id": "/Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/bitnami/ArtifactTypes/VMImage/Offers/rabbitmq/Skus/rabbitmq/Versions/3.7.1807171506",
  "location": "westus",
  "name": "3.7.1807171506",
  "osDiskImage": {
    "operatingSystem": "Linux"
  },
  "plan": {
    "name": "rabbitmq",
    "product": "rabbitmq",
    "publisher": "bitnami"
  },
  "tags": null
}
```

### <a name="accept-the-terms"></a>接受條款
若要檢視並接受授權條款，請使用 [az vm image accept-terms](/cli/azure/vm/image?#az_vm_image_accept_terms) 命令。 當您接受條款時，您會在訂用帳戶中啟用以程式設計方式部署。 您只需針對映像的每個訂用帳戶接受一次條款。 例如︰

```azurecli
az vm image accept-terms --urn bitnami:rabbitmq:rabbitmq:latest
``` 

輸出會包含授權條款的 `licenseTextLink`，並指出 `accepted` 的值為 `true`：

```
{
  "accepted": true,
  "additionalProperties": {},
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.MarketplaceOrdering/offertypes/bitnami/offers/rabbitmq/plans/rabbitmq",
  "licenseTextLink": "https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_BITNAMI%253a24RABBITMQ%253a24RABBITMQ%253a24IGRT7HHPIFOBV3IQYJHEN2O2FGUVXXZ3WUYIMEIVF3KCUNJ7GTVXNNM23I567GBMNDWRFOY4WXJPN5PUYXNKB2QLAKCHP4IE5GO3B2I.txt",
  "name": "rabbitmq",
  "plan": "rabbitmq",
  "privacyPolicyLink": "https://bitnami.com/privacy",
  "product": "rabbitmq",
  "publisher": "bitnami",
  "retrieveDatetime": "2018-02-22T04:06:28.7641907Z",
  "signature": "WVIEA3LAZIK7ZL2YRV5JYQXONPV76NQJW3FKMKDZYCRGXZYVDGX6BVY45JO3BXVMNA2COBOEYG2NO76ONORU7ITTRHGZDYNJNKLNLWI",
  "type": "Microsoft.MarketplaceOrdering/offertypes"
}
```

### <a name="deploy-using-purchase-plan-parameters"></a>使用購買方案的參數進行部署
接受映像的條款之後，您可以在訂用帳戶中部署 VM。 若要使用 `az vm create` 命令來部署映像，除了映像的 URN 之外，請提供購買方案的參數。 例如，若要使用 RabbitMQ Certified by Bitnami 映像部署 VM：

```azurecli
az group create --name myResourceGroupVM --location westus

az vm create --resource-group myResourceGroupVM --name myVM --image bitnami:rabbitmq:rabbitmq:latest --plan-name rabbitmq --plan-product rabbitmq --plan-publisher bitnami

```



## <a name="next-steps"></a>後續步驟
若要使用映像資訊來快速建立虛擬機器，請參閱[使用 Azure CLI 來建立和管理 Linux VM](tutorial-manage-vm.md)。
