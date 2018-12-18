---
title: 使用 ZIP 或 WAR 檔案將您的應用程式部署至 Azure App Service | Microsoft Docs
description: 了解如何使用 ZIP 檔案 (若為 Java 開發人員，則是 WAR 檔案) 將您的應用程式部署至 Azure App Service。
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/07/2018
ms.author: cephalin;sisirap
ms.openlocfilehash: b628ae0806febb3ffd4edaf71be45841aff38516
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39423023"
---
# <a name="deploy-your-app-to-azure-app-service-with-a-zip-or-war-file"></a>使用 ZIP 或 WAR 檔案將您的應用程式部署至 Azure App Service

本文說明如何使用 ZIP 檔案或 WAR 檔案將您的 Web 應用程式部署到 [Azure App Service](app-service-web-overview.md)。 

此 ZIP 檔案部署所使用的是支援持續整合式部署的同一個 Kudu 服務。 Kudu 為 ZIP 檔案部署支援下列功能： 

- 刪除先前部署所留下的檔案。
- 用來開啟預設建置程序的選項，其中包含套件還原。
- [部署自訂](https://github.com/projectkudu/kudu/wiki/Configurable-settings#repository-and-deployment-related-settings)，包括執行中的部署指令碼。  
- 部署記錄。 

如需詳細資訊，請參閱 [Kudu 文件](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)。

WAR 檔案部署會將您的 [WAR](https://wikipedia.org/wiki/WAR_(file_format)) 檔案部署至 App Service，以執行您的 Java Web 應用程式。 請參閱[部署 WAR 檔案](#deploy-war-file)。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本文中的步驟：

* [建立 App Service 應用程式](/azure/app-service/)，或使用您針對另一個教學課程建立的應用程式。

## <a name="create-a-project-zip-file"></a>建立專案 ZIP 檔案

>[!NOTE]
> 如果您在 ZIP 檔案中下載了檔案，請先將檔案解壓縮。 例如，如果您從 GitHub 下載了 ZIP 檔案，則無法以原狀部署該檔案。 GitHub 會新增其他巢狀目錄，其不適用於 App Service。 
>

在本機終端機視窗中，瀏覽至應用程式專案的根目錄。 

此目錄應該包含您的 web 應用程式的項目檔案，例如 index.html、index.php 和 app.js。 也可以包含套件管理檔案，像是 project.json、composer.json、package.json、bower.json 和 requirements.txt。

在專案中建立所有項目的 ZIP 封存。 下列命令會使用您終端機中的預設工具：

```
# Bash
zip -r <file-name>.zip .

# PowerShell
Compress-Archive -Path * -DestinationPath <file-name>.zip
``` 

[!INCLUDE [Deploy ZIP file](../../includes/app-service-web-deploy-zip.md)]

## <a name="deploy-zip-file-with-azure-cli"></a>使用 Azure CLI 部署 ZIP 檔案

請確定您的 Azure CLI 是 2.0.21 版或更新版本。 若要查看您有哪個版本，請在您的終端機視窗中執行 `az --version` 命令。

使用 [az webapp deployment source config-zip](/cli/azure/webapp/deployment/source?view=azure-cli-latest#az-webapp-deployment-source-config-zip) 命令，將上傳的 ZIP 檔案部署至 Web 應用程式。  

下列範例會部署您上傳的 ZIP 檔案。 使用 Azure CLI 的本機安裝時，請針對 `--src` 指定您的本機 ZIP 檔案路徑。   

```azurecli-interactive
az webapp deployment source config-zip --resource-group myResourceGroup --name <app_name> --src clouddrive/<filename>.zip
```

此命令會將檔案和目錄從 ZIP 檔案部署到預設的 App Service 應用程式資料夾 (`\home\site\wwwroot`)，並重新啟動應用程式。 如果已設定任何其他的自訂建置程序，它也會執行。 如需詳細資訊，請參閱 [Kudu 文件](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)。

[!INCLUDE [app-service-deploy-zip-push-rest](../../includes/app-service-deploy-zip-push-rest.md)]  

## <a name="deploy-war-file"></a>部署 WAR 檔案

若要將 WAR 檔案部署至 App Service，請將 POST 要求傳送至 https://<app_name>.scm.azurewebsites.net/api/wardeploy。 POST 要求必須在訊息本文中包含 .war 檔案。 系統會使用 HTTP 基本驗證，在要求中提供應用程式的部署認證。 

針對 HTTP 基本驗證，您需要 App Service 部署的認證。 若要了解如何設定部署認證，請參閱[設定及重設使用者層級的認證](app-service-deployment-credentials.md#userscope)。

### <a name="with-curl"></a>使用 cURL

下列範例會使用 cURL 工具來部署 .war 檔案。 取代預留位置 `<username>`、`<war_file_path>` 和 `<app_name>`。 當 cURL 顯示提示時，請輸入密碼。

```bash
curl -X POST -u <username> --data-binary @"<war_file_path>" https://<app_name>.scm.azurewebsites.net/api/wardeploy
```

### <a name="with-powershell"></a>透過 PowerShell

下列範例使用 [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) 來傳送包含 .war 檔案的要求。 取代預留位置 `<deployment_user>`、`<deployment_password>`、`<zip_file_path>` 和 `<app_name>`。

```PowerShell
$username = "<deployment_user>"
$password = "<deployment_password>"
$filePath = "<war_file_path>"
$apiUrl = "https://<app_name>.scm.azurewebsites.net/api/wardeploy"
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username, $password)))
Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -Method POST -InFile $filePath -ContentType "multipart/form-data"
```

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="next-steps"></a>後續步驟

如需更多的進階部署案例，請嘗試[使用 Git 部署至 Azure ](app-service-deploy-local-git.md)。 Git 型部署至 Azure 可啟用版本控制、封裝還原、MSBuild 等等。

## <a name="more-resources"></a>其他資源

* [Kudu：從 zip 檔案部署](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)
* [Azure App Service 部署認證](app-service-deploy-ftp.md)
