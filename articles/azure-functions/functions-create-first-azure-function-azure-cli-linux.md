---
title: 在 Linux 上從 Azure CLI 建立您的第一個函式 (預覽) | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立您的第一個在預設 Linux 映像上執行的 Azure 函式。
services: functions
keywords: ''
author: ggailey777
ms.author: glenga
ms.date: 11/15/2017
ms.topic: quickstart
ms.service: azure-functions
ms.custom: mvc
ms.devlang: azure-cli
manager: jeconnoc
ms.openlocfilehash: 1cf20a4a93ef1b5bfb9c7818f35be5e75e45a3d2
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901085"
---
# <a name="create-your-first-function-running-on-linux-using-the-azure-cli-preview"></a>使用 Azure CLI 建立第一個在 Linux 上執行的函式 (預覽)

Azure Functions 可讓您在 Linux 上預設的 Azure App Service 容器中裝載函式。 您也可以[自備自訂容器](functions-create-function-linux-custom-image.md)。 這項功能目前為預覽狀態並且需要 [Functions 2.0 執行階段](functions-versions.md)。

本快速入門主題會逐步解說如何使用 Azure Functions 與 Azure CLI，在預設 App Service 容器裝載的 Linux 上建立第一個函式應用程式。 函式程式碼本身是從 GitHub 範例存放庫部署到映像。    

下列步驟適用於 Mac、Windows 或 Linux 電腦。 

## <a name="prerequisites"></a>必要條件 

若要完成本快速入門，您需要：

+ 有效的 Azure 訂用帳戶。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本主題需要 Azure CLI 2.0.21 版或更新版本。 執行 `az --version` 以尋找您擁有的版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

[!INCLUDE [functions-create-resource-group](../../includes/functions-create-resource-group.md)]

[!INCLUDE [functions-create-storage-account](../../includes/functions-create-storage-account.md)]

## <a name="create-a-linux-app-service-plan"></a>建立 Linux App Service 方案

使用於 Functions 的 Linux 裝載目前僅支援 App Service 方案。 尚未支援使用情況方案裝載。 若要深入了解裝載，請參閱 [Azure Functions 裝載方案比較](functions-scale.md)。 

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

## <a name="create-a-function-app-on-linux"></a>在 Linux 上建立函式應用程式

您必須擁有函式應用程式以便在 Linux 上主控函式的執行。 函式應用程式會提供環境來讓您的函式程式碼進行執行。 它可讓您將多個函式群組為邏輯單位，以方便您管理、部署和共用資源。 使用 [az functionapp create](/cli/azure/functionapp#az-functionapp-create) 命令與 Linux App Service 方案來建立函式應用程式。 

在下列命令中，使用唯一函式應用程式名稱來替代您看見 `<app_name>` 預留位置的地方，並使用儲存體帳戶名稱來替代 `<storage_name>`。 `<app_name>` 會作為函式應用程式的預設 DNS 網域，所以此名稱在 Azure 的所有應用程式中都必須是唯一的名稱。 _deployment-source-url_ 參數是 GitHub 中的範例存放庫，其中包含 "Hello World" HTTP 觸發函式。

```azurecli-interactive
az functionapp create --name <app_name> --storage-account  <storage_name>  --resource-group myResourceGroup \
--plan myAppServicePlan --deployment-source-url https://github.com/Azure-Samples/functions-quickstart-linux
```
建立及部署函式應用程式後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 1536,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "quickstart.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "quickstart.azurewebsites.net",
    "quickstart.scm.azurewebsites.net"
  ],
   ....
    // Remaining output has been truncated for readability.
}
```

因為 `myAppServicePlan` 是 Linux 方案，內建 Docker 映像會用來建立容器 (在 Linux 上執行函式應用程式)。 

>[!NOTE]  
>範例存放庫目前包含兩個指令碼檔案，[deploy.sh](https://github.com/Azure-Samples/functions-quickstart-linux/blob/master/deploy.sh) 和 [.deployment](https://github.com/Azure-Samples/functions-quickstart-linux/blob/master/.deployment)。 .deployment 檔案會告知部署程序使用 deploy.sh 作為[自訂部署指令碼](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script)。 在目前的預覽版本中，需要指令碼才能在 Linux 映像上部署函式應用程式。  

## <a name="configure-the-function-app"></a>設定函式應用程式

GitHub 存放庫中的專案需要 1.x 版的 Functions 執行階段。 將 `FUNCTIONS_WORKER_RUNTIME` 應用程式設定設為 `~1`，以將函式應用程式釘選至最新的 1.x 版本。 使用 [az functionapp config appsettings set](https://docs.microsoft.com/cli/azure/functionapp/config/appsettings#set) 命令設定應用程式設定。

在下列 Azure CLI 命令中，`<app_name> 是函式應用程式的名稱。

```azurecli-interactive
az functionapp config appsettings set --name <app_name> \
--resource-group myResourceGroup \
--settings FUNCTIONS_WORKER_RUNTIME=~1
```

[!INCLUDE [functions-test-function-code](../../includes/functions-test-function-code.md)]

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

[!INCLUDE [functions-quickstart-next-steps-cli](../../includes/functions-quickstart-next-steps-cli.md)]
