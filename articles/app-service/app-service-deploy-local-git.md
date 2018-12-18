---
title: 本機 Git 部署至 Azure App Service
description: 了解如何啟用本機 Git 部署至 Azure App Service。
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/05/2018
ms.author: dariagrigoriu;cephalin
ms.openlocfilehash: ae8739a65efbe7662a8f72e961d772fecaf4b527
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303688"
---
# <a name="local-git-deployment-to-azure-app-service"></a>本機 Git 部署至 Azure App Service

本使用說明指南示範如何將程式碼從本機電腦上的 Git 存放庫部署到 [Azure App Service](app-service-web-overview.md)。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

遵循本使用說明指南中的步驟：

* [安裝 Git](http://www.git-scm.com/downloads)。
* 使用您想要部署的程式碼維護本機 Git 存放庫。

若要範例存放庫以跟著執行，請在您的本機終端機視窗中執行下列命令：

```bash
git clone https://github.com/Azure-Samples/nodejs-docs-hello-world.git
```

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-from-local-git-with-kudu-builds"></a>使用 Kudu 組建從本機 Git 進行部署

使用 Kudu 組建伺服器為您的應用程式啟用本機 Git 部署的最簡單方式是使用 Cloud Shell。

### <a name="create-a-deployment-user"></a>建立部署使用者

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="enable-local-git-with-kudu"></a>使用 Kudu 啟用本機 Git

若要使用 Kudu 組建伺服器為您的應用程式啟用本機 Git 部署，在 Cloud Shell 中執行 [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/source?view=azure-cli-latest#az-webapp-deployment-source-config-local-git)。

```azurecli-interactive
az webapp deployment source config-local-git --name <app_name> --resource-group <group_name>
```

若要改為建立已啟用 Git 的應用程式，請在 Cloud Shell 中搭配 `--deployment-local-git` 參數執行 [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create)。

```azurecli-interactive
az webapp create --name <app_name> --resource-group <group_name> --plan <plan_name> --deployment-local-git
```

`az webapp create` 命令應可提供類似下列輸出的內容：

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

### <a name="deploy-your-project"></a>部署專案

回到「本機終端視窗」，將 Azure 遠端新增至本機 Git 存放庫。 使用從[為應用程式啟用 Git](#enable-git-for-you-app) 中取得的 Git 遠端 URL 來取代 _\<url>_。

```bash
git remote add azure <url>
```

推送到 Azure 遠端，使用下列命令來部署您的應用程式。 當系統提示輸入密碼時，務必輸入您在[設定部署使用者](#configure-a-deployment-user)中建立的密碼，而不是您用來登入 Azure 入口網站的密碼。

```bash
git push azure master
```

您可能會在輸出中看到執行階段特定的自動化，例如適用於 ASP.NET 的 MSBuild、適用於 Node.js 的 `npm install`，以及適用於 Python 的 `pip install`。 

瀏覽至您的應用程式以確認已部署內容。

## <a name="deploy-from-local-git-with-azure-devops-services-builds"></a>使用 Azure DevOps Services 組建從本機 Git 進行部署

> [!NOTE]
> 若要讓 App Service 在 Azure DevOps Services 組織中建立必要的 Azure Pipelines，您的 Azure 帳戶必須具備 Azure 訂用帳戶的**擁有者**角色。
>

若要使用 Kudu 組建伺服器為您的應用程式啟用本機 Git 部署，請在 [Azure 入口網站](https://portal.azure.com)中瀏覽至應用程式。

在應用程式頁面的左側導覽中，按一下 [部署中心] > [本機 Git] > [繼續]。 

![](media/app-service-deploy-local-git/portal-enable.png)

按一下 [Azure DevOps Services 持續傳遞] > [繼續]。

![](media/app-service-deploy-local-git/vsts-build-server.png)

在 [設定] 頁面中，設定新的 Azure DevOps Services 組織或指定現有的組織。 完成後，按一下 [繼續]。

> [!NOTE]
> 如果您想要使用未列出的現有 Azure DevOps Services 組織，則必須[將 Azure DevOps Services 組織連結至 Azure 訂用帳戶](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App)。

在 [測試] 頁面上，選擇是否要啟用負載測試，然後按一下 [繼續]。

視 App Service 方案的[定價層](https://azure.microsoft.com/pricing/details/app-service/plans/)而定，您可能也會看到 [部署至預備] 頁面。 選擇是否要啟用部署位置，然後按一下 [繼續]。

在 [摘要] 頁面上確認您的選項，然後按一下 [完成]。

Azure DevOps Services 組織需要幾分鐘才會準備就緒。 準備好時，複製部署中心內的 Git 儲存機制 URL。

![](media/app-service-deploy-local-git/vsts-repo-ready.png)

回到「本機終端視窗」，將 Azure 遠端新增至本機 Git 存放庫。 使用您從最後一個步驟取得的 URL 來取代 _\<url>_。

```bash
git remote add vsts <url>
```

推送到 Azure 遠端，使用下列命令來部署您的應用程式。 出現由 Git 認證管理員提供的提示時，請使用您的 visualstudio.com 使用者登入。 如需其他驗證方法，請參閱 [Azure DevOps Services 驗證概觀](/vsts/git/auth-overview?view=vsts)。

```bash
git push vsts master
```

完成部署時，您可以在 `https://<vsts_account>.visualstudio.com/<project_name>/_build` 上找到建置進度，並在 `https://<vsts_account>.visualstudio.com/<project_name>/_release` 上找到部署進度。

瀏覽至您的應用程式以確認已部署內容。

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshooting-kudu-deployment"></a>針對 Kudu 部署進行疑難排解

以下是使用 Git 發佈至 Azure 中的 App Service 應用程式時，會遇到的常見錯誤或問題：

---
**徵狀**：`Unable to access '[siteURL]': Failed to connect to [scmAddress]`

**原因**：如果應用程式尚未啟動並執行，就會發生這個錯誤。

**解決方式**：在 Azure 入口網站中啟動應用程式。 當 Web 應用程式停止時，無法使用 Git 部署。

---
**徵狀**：`Couldn't resolve host 'hostname'`

**原因**：如果建立 'azure' 遠端時所輸入的位址資訊不正確，就會發生這個錯誤。

**解決方式**：使用 `git remote -v` 命令，列出所有遠端以及相關聯的 URL。 驗證 'azure' 遠端的 URL 是否正確。 如有需要，移除此遠端並使用正確的 URL 重新建立。

---
**徵狀**：`No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'master'.`

**原因**：如果您在 `git push` 期間未指定分支，或未在 `.gitconfig` 中設定 `push.default` 值，就會發生這個錯誤。

**解決方式**：再次執行 `git push`，指定主分支。 例如︰

```bash
git push azure master
```

---
**徵狀**：`src refspec [branchname] does not match any.`

**原因**：如果您嘗試在 'azure' 遠端上推送至主分支以外的分支，就會發生這個錯誤。

**解決方式**：再次執行 `git push`，指定主分支。 例如︰

```bash
git push azure master
```

---
**徵狀**：`RPC failed; result=22, HTTP code = 5xx.`

**原因**︰如果您嘗試透過 HTTPS 推送大型 Git 存放庫，就會發生這個錯誤。

**解決方式**︰變更本機電腦上的 Git 組態以加大 postBuffer

```bash
git config --global http.postBuffer 524288000
```

---
**徵狀**：`Error - Changes committed to remote repository but your web app not updated.`

**原因**：如果您部署包含會指出需要額外模組之 _package.json_ 檔案的 Node.js 應用程式，就會發生這個錯誤。

**解決方式**︰其他包含 'npm ERR!' 的訊息 在發生這個錯誤之前應該就已記錄，並可提供關於此失敗的更多資訊。 下列是此錯誤的已知原因及其對應的 'npm ERR!' message:

* **格式錯誤的 package.json 檔案**：npm ERR! 無法讀取相依性。
* **沒有適用於 Windows 的二進位發佈原生模組**：

  * `npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1`

      或
  * `npm ERR! [modulename@version] preinstall: \make || gmake\`

## <a name="additional-resources"></a>其他資源

* [專案 Kudu 文件](https://github.com/projectkudu/kudu/wiki)
* [持續部署至 Azure App Service](app-service-continuous-deployment.md)
* [範例：建立 Web 應用程式並從本機 Git 存放庫部署程式碼 (Azure CLI)](./scripts/app-service-cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json)
* [範例：建立 Web 應用程式並從本機 Git 存放庫部署程式碼 (PowerShell)](./scripts/app-service-powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json)
