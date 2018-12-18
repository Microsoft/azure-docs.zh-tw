---
title: Azure Functions 的 ZIP 推送部署 | Microsoft Docs
description: 使用 Kudu 部署服務的 .zip 檔案部署工具來發佈 Azure Functions。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 08/12/2018
ms.author: glenga
ms.openlocfilehash: 03cb9384beba1afcb97a765824e63e79ca34aa62
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44717213"
---
# <a name="zip-deployment-for-azure-functions"></a>Azure Functions 的 ZIP 部署

本文會說明如何從 .zip (壓縮) 檔案將函式應用程式的專案檔部署至 Azure。 您將了解如何藉由使用 Azure CLI 和使用 REST API 來進行推送部署。 [Azure Functions Core Tools](functions-run-local.md) 也會在將本機專案發佈至 Azure 時，使用這些部署 API。

Azure Functions 擁有 Azure App Service 所提供的全套持續部署與整合選項。 如需詳細資訊，請參閱 [Azure Functions 的持續部署](functions-continuous-deployment.md)。

為了加快開發速度，您可能會發現直接從 .zip 檔案部署函式應用程式的專案檔會比較容易。 .zip 部署 API 會取得 .zip 檔案的內容，並將內容擷取至函式應用程式的 `wwwroot` 資料夾。 此 .zip 檔案部署所使用的是支援持續整合式部署的同一個 Kudu 服務，包括：

+ 刪除較舊部署所留下的檔案。
+ 部署自訂，包括執行中的部署指令碼。
+ 部署記錄。
+ 同步處理[取用方案](functions-scale.md)函式應用程式中的函式觸發程序。

如需詳細資訊，請參閱 [.zip 部署參考](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)。

## <a name="deployment-zip-file-requirements"></a>.zip 部署檔案需求

您用於推送部署的 .zip 檔案必須包含執行函式所需的所有檔案。

>[!IMPORTANT]
> 當您使用 .zip 部署時，.zip 檔案中找不到的現有部署檔案，都會從函式應用程式中刪除。  

[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

函式應用程式包含 `wwwroot` 目錄中所有檔案和資料夾。 .zip 檔案部署包含 `wwwroot` 目錄的內容，但不是目錄本身。 在部署 C# 類別庫專案時，您必須將已編譯的類別庫檔案和相依性納入到 .zip 套件中的 `bin` 子資料夾。

## <a name="download-your-function-app-files"></a>下載函式應用程式檔案

在本機電腦上進行開發時，您不用費多少功夫就能在開發電腦上建立函式應用程式專案資料夾的 .zip 檔案。

不過，您可能已在 Azure 入口網站中使用編輯器建立好您的函式。 您可以使用下列其中一個方式來下載現有函式應用程式專案：

+ **從 Azure 入口網站：**

    1. 登入 [Azure 入口網站](https://portal.azure.com)，然後移至您的函式應用程式。

    2. 在 [概觀] 索引標籤上，選取 [下載應用程式內容]。 選取下載選項，然後選取 [下載]。

        ![下載函式應用程式專案](./media/deployment-zip-push/download-project.png)

    所下載的 .zip 檔案會採用正確格式，以便您可以使用 .zip 推送部署加以重新發佈至函式應用程式。 入口網站下載也可新增直接在 Visual Studio 中開啟您函式應用程式所需的檔案。

+ **使用 REST API：**

    使用下列部署 GET API 從您的 `<function_app>` 專案下載檔案： 

        https://<function_app>.scm.azurewebsites.net/api/zip/site/wwwroot/

    包括 `/site/wwwroot/` 可確保您的 zip 檔案僅包含函式應用程式專案檔案，而不是整個站台。 如果您尚未登入 Azure，系統會要求您登入。  

您也可以從 GitHub 存放庫下載 .zip 檔案。 當您將 GitHub 存放庫下載為 .zip 檔案時，GitHub 會為分支新增額外的資料夾層級。 這個額外的資料夾層級表示，由於您是從 GitHub 下載 .zip 檔案，所以無法直接部署該檔案。 如果您使用 GitHub 存放庫來維護函式應用程式，請使用[持續整合](functions-continuous-deployment.md)來部署應用程式。  

## <a name="cli"></a>使用 Azure CLI 進行部署

您可以使用 Azure CLI 來觸發推送部署。 請使用 [az functionapp deployment source config-zip](/cli/azure/functionapp/deployment/source#az-functionapp-deployment-source-config-zip) 命令，將 .zip 檔案推送部署至函式應用程式。 若要使用此命令，您所使用的 Azure CLI 必須是 2.0.21 版或更新版本。 若要了解您所使用的 Azure CLI 版本，請使用 `az --version` 命令。

在下列命令中，使用 .zip 檔案的位置路徑取代 `<zip_file_path>` 預留位置。 也請使用函式應用程式的唯一名稱取代 `<app_name>`。 

```azurecli-interactive
az functionapp deployment source config-zip  -g myResourceGroup -n \
<app_name> --src <zip_file_path>
```

此命令會將專案檔從所下載的 .zip 檔案部署到您在 Azure 中的函式應用程式。 然後重新啟動應用程式。 若要檢視此函式應用程式的部署清單，您必須使用 REST API。

當您在本機電腦上使用 Azure CLI 時，`<zip_file_path>` 是 .zip 檔案在您電腦上的路徑。 您也可以在 [Azure Cloud Shell](../cloud-shell/overview.md) 中執行 Azure CLI。 當您使用 Cloud Shell 時，必須先將部署 .zip 檔案上傳到與 Cloud Shell 相關聯的 Azure 檔案服務帳戶。 在此情況下，`<zip_file_path>` 會是 Cloud Shell 帳戶所使用的儲存體位置。 如需詳細資訊，請參閱[在 Azure Cloud Shell 中保存檔案](../cloud-shell/persisting-shell-storage.md)。

[!INCLUDE [app-service-deploy-zip-push-rest](../../includes/app-service-deploy-zip-push-rest.md)]

## <a name="run-functions-from-the-deployment-package"></a>從部署套件執行函式

您也可以選擇直接從部署套件檔案執行函式。 這個方法會略過將套件中的檔案複製到函式應用程式 `wwwroot` 目錄的部署步驟。 相反地，套件檔案會由 Functions 執行階段掛接，而且 `wwwroot` 目錄的內容會變成唯讀狀態。  

ZIP 部署會與這項功能整合，藉由將函式應用程式的設定 `WEBSITE_RUN_FROM_PACKAGE` 設定為值 `1` 即可加以啟用。 如需詳細資訊，請參閱[從部署套件檔案執行函式](run-functions-from-deployment-package.md)。

[!INCLUDE [app-service-deploy-zip-push-custom](../../includes/app-service-deploy-zip-push-custom.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Functions 的持續部署](functions-continuous-deployment.md)

[.zip push deployment reference topic]: https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file
