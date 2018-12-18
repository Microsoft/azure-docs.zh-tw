---
title: 包含檔案
description: 包含檔案
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 3275fb0e40e7cdd6f02ec96fb949ec4e4e673f26
ms.sourcegitcommit: 902aae1b5f872993185cbf1e35a4f7bb93748226
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/15/2018
ms.locfileid: "35656312"
---
在 `myAppServicePlan` App Service 方案中建立 [Web 應用程式](../articles/app-service/containers/app-service-linux-intro.md)。 

在 Cloud Shell 中，您可以使用 [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az_webapp_create) 命令。 在下列範,了中，使用全域唯一的應用程式名稱 (有效的字元為 `a-z`、`0-9` 和 `-`) 取代 `<app_name>`。 

```azurecli-interactive
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --deployment-local-git
```

建立 Web 應用程式後，Azure CLI 會顯示類似下列範例的輸出：

```json
Local git is configured with url of 'https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "deploymentLocalGitUrl": "https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

> [!NOTE]
> Git 遠端的 URL 會顯示在 `deploymentLocalGitUrl` 屬性中，其格式為 `https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git`。 儲存此 URL，稍後您會用到此資訊。
>
