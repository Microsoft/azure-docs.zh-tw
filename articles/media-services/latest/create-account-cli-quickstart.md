---
title: 快速入門 - 使用 Azure CLI 建立 Azure 媒體服務帳戶 |Microsoft Docs
description: 按照本快速入門的步驟來建立 Azure 媒體服務帳戶。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: quickstart
ms.custom: mvc
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: de54571308b737b9160a39ee4ba5d4b2d9f15775
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376528"
---
# <a name="quickstart-create-an-azure-media-services-account"></a>快速入門：建立 Azure 媒體服務帳戶

不論您是開發人員或是媒體內容建立者，若要在 Azure 中儲存、加密、編碼、管理及串流媒體內容，都必須建立媒體服務帳戶。 當建立媒體服務帳戶時，您需要提供 Azure 儲存體帳戶資源的識別碼。 指定的儲存體帳戶會附加到您的媒體服務帳戶。 此儲存體帳戶資源必須和媒體服務帳戶位在相同的地理區域。  

本快速入門說明使用 Azure CLI 建立新的 Azure 媒體服務帳戶的步驟。  

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="log-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](http://portal.azure.com)，然後啟動 **CloudShell** 來執行 CLI 命令，如後續步驟所示。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 CLI，本主題需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找您擁有的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="set-the-azure-subscription"></a>設定 Azure 訂用帳戶

在下列命令中，提供您要用於媒體服務帳戶的 Azure 訂用帳戶識別碼。 您可以瀏覽至[訂閱](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)，以查看可以存取的訂用帳戶。

```azurecli-interactive
az account set --subscription <mySubscriptionId>
```

## <a name="create-an-azure-resource-group"></a>建立 Azure 資源群組

下列命令會建立其中您想要擁有儲存體和媒體服務帳戶的資源群組。 使用您想用於資源群組的名稱，替代 *myresourcegroup* 預留位置。

```azurecli-interactive
az group create -n <myresourcegroup> -l westus2
```

## <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶

當建立媒體服務帳戶時，您需要提供 Azure 儲存體帳戶資源的識別碼。 指定的儲存體帳戶會附加到您的媒體服務帳戶。 

您只能有一個**主要**儲存體帳戶，而與您的媒體服務帳戶相關聯的**次要**儲存體帳戶可以有任意數量。 「媒體服務」支援**一般用途 v2** 或 **一般用途 v1** 帳戶。 Blob 儲存體帳戶不允許作為**主要**帳戶。 如需有關儲存體帳戶的詳細資訊，請參閱 [Azure 儲存體帳戶概觀](../../storage/common/storage-account-overview.md)。 

以下命令會建立要與媒體服務帳戶 (主要) 相關聯的儲存體帳戶。 在以下的指令碼中，請取代 *storageaccountforams* 預留位置。 'account_name' 的長度必須少於 24。

```azurecli-interactive
az storage account create -n <storageaccountforams> -g <myresourcegroup>
```

## <a name="create-an-azure-media-services-account"></a>建立 Azure 媒體服務帳戶

您可以在下面找到建立新的媒體服務帳戶的 Azure CLI 命令。 您只需要取代下列反白顯示的值：

* *myamsaccountname*
* *myresourcegroup*
* *storageaccountforams*

```azurecli-interactive
az ams create -n <myamsaccountname> -g <myresourcegroup> --storage-account <storageaccountforams>
```

## <a name="clean-up-resources"></a>清除資源

如果您不再需要資源群組中的任何資源 (包括在此快速入門中建立的媒體服務帳戶)，請將資源群組刪除。

在 **CloudShell** 中，執行以下命令：

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [串流處理檔案](stream-files-dotnet-quickstart.md)
