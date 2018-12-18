---
title: 使用 Azure CLI 來建立 Azure 媒體服務帳戶 | Microsoft Docs
description: 按照此快速入門的步驟來建立 Azure 媒體服務帳戶。
services: media-services
documentationcenter: ''
author: Juliako
manager: cflower
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 03/27/2018
ms.author: juliako
ms.openlocfilehash: ca01f32709ce7c9fc49629415cd8697a9d9ba43a
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47091944"
---
# <a name="create-an-azure-media-services-account"></a>建立 Azure 媒體服務帳戶

若要在 Azure 中開始加密、編碼、分析、管理和串流處理媒體內容，您需要建立 Media Services 帳戶。 當您建立媒體服務帳戶時，您也會在與媒體服務帳戶相同的地理區域中建立相關聯的儲存體帳戶 (或使用現有儲存體帳戶)。

此主題說明使用 Azure CLI 來建立新「Azure 媒體服務」帳戶的步驟。  

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="log-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](http://portal.azure.com)，然後啟動 **CloudShell** 來執行 CLI 命令，如後續步驟所示。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 CLI，此主題需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找您擁有的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="set-the-azure-subscription"></a>設定 Azure 訂用帳戶

在下列命令中，提供您要用於媒體服務帳戶的 Azure 訂用帳戶識別碼。 您可以瀏覽至[訂閱](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)，以查看可以存取的訂用帳戶。

```azurecli-interactive
az account set --subscription mySubscriptionId
```
 
[!INCLUDE [media-services-cli-create-v3-account-include](../../../includes/media-services-cli-create-v3-account-include.md)]
 
## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [串流處理檔案](stream-files-dotnet-quickstart.md)
