---
title: 使用用於容器的 Web App 從 Docker 容器登錄進行持續部署 - Azure | Microsoft Docs
description: 如何在用於容器的 Web App 中設定從 Docker 容器登錄進行持續部署。
keywords: azure app service, linux, docker, acr,oss
services: app-service
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.assetid: a47fb43a-bbbd-4751-bdc1-cd382eae49f8
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/29/2018
ms.author: msangapu
ms.openlocfilehash: 0f2d4626308eed376b71f1b3df2f9e43f1b2a4f7
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37130956"
---
# <a name="continuous-deployment-with-web-app-for-containers"></a>使用用於容器的 Web 應用程式進行持續部署

在本教學課程中，您可以從受控 [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) 存放庫或 [Docker Hub](https://hub.docker.com) 設定自訂容器映像的持續部署。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="enable-the-continuous-deployment-feature"></a>啟用持續部署功能

您可以使用 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 並執行下列命令來啟用持續部署功能：

```azurecli-interactive
az webapp deployment container config --name name --resource-group myResourceGroup --enable-cd true
```

在 [Azure 入口網站](https://portal.azure.com/)中，選取頁面左側的 [App Service] 選項。

選取您要設定 Docker Hub 持續部署的應用程式名稱。

在 [Docker 容器] 頁面上，選取 [開啟]，然後選取 [儲存] 以啟用持續部署。

![應用程式設定的螢幕擷取畫面](./media/app-service-webapp-service-linux-ci-cd/step2.png)

## <a name="prepare-the-webhook-url"></a>準備 Webhook URL

使用 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 並執行下列命令來取得 Webhook URL：

```azurecli-interactive
az webapp deployment container show-cd-url --name sname1 --resource-group rgname
```

記下 Webhook URL。 您將會在下一節用到此 URL。
`https://<publishingusername>:<publishingpwd>@<sitename>.scm.azurewebsites.net/docker/hook`。

您可以取得 `publishingusername` 和 `publishingpwd`，方法是使用 Azure 入口網站來下載 Web 應用程式發佈設定檔。

![新增 Webhook 的螢幕擷取畫面 2](./media/app-service-webapp-service-linux-ci-cd/step3-3.png)

## <a name="add-a-webhook"></a>新增 Webhook

若要新增 Webhook，請遵循這些指南中的步驟：

- 使用 Webhook URL 的 [Azure Container Registry](../../container-registry/container-registry-webhook.md)
- [適用於 Docker Hub 的 Webhook](https://docs.docker.com/docker-hub/webhooks/)

## <a name="next-steps"></a>後續步驟

* [Linux 上的 Azure App Service 簡介](./app-service-linux-intro.md)
* [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)
* [在 Linux 上的 App Service 中建立.NET Core Web 應用程式](quickstart-dotnetcore.md)
* [在 Linux 上的 App Service 中建立 Ruby Web 應用程式](quickstart-ruby.md)
* [在用於容器的 Web App 中部署 Docker/Go Web 應用程式](quickstart-docker-go.md)
* [Linux 上的 Azure App Service 常見問題集](./app-service-linux-faq.md)
* [使用 Azure CLI 管理用於容器的 Web App](./app-service-linux-cli.md)
