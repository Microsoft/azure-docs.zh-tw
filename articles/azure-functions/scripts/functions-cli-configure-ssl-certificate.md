---
title: Azure CLI 指令碼範例 - 將自訂 SSL 憑證繫結至函式應用程式 | Microsoft Docs
description: Azure CLI 指令碼範例 - 在 Azure 中將自訂 SSL 憑證繫結至函式應用程式
services: functions
documentationcenter: ''
author: ggailey777
manager: jeconnoc
ms.assetid: eb95d350-81ea-4145-a1e2-6eea3b7469b2
ms.service: azure-functions
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/03/2013
ms.author: glenga
ms.custom: mvc
ms.openlocfilehash: 689764543f5d927273f92deecbfd43e282fc028c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961360"
---
# <a name="bind-a-custom-ssl-certificate-to-a-function-app"></a>將自訂 SSL 憑證繫結至函式應用程式

此範例指令碼會在 App Service 中建立函式應用程式及其相關資源，然後將自訂網域名稱的 SSL 憑證與其繫結。 針對此範例，您需要：

* 存取網域註冊機構的 DNS 設定頁面。
* 對於想要上傳並繫結的 SSL 憑證，您具備有效的 .PFX 檔案和其密碼。
* 已在指向您 Web 應用程式預設網域名稱的自訂網域中設定 A 記錄。 如需詳細資訊，請參閱[將現有的自訂 DNS 名稱對應至 Azure Web Apps](https://aka.ms/appservicecustomdns)。

若要繫結 SSL 憑證，必須在 App Service 方案中建立函式應用程式，而不是在取用方案中建立。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，則必須執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom SSL certificate to a web app")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account#az-storage-account-create) | 建立函式應用程式所需的儲存體帳戶。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#az-appservice-plan-create) | 建立繫結 SSL 憑證所需的 App Service 方案。 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az-functionapp-create) | 在 App Service 方案中建立函式應用程式。 |
| [az functionapp config hostname add](https://docs.microsoft.com/cli/azure/functionapp/config/hostname#az-functionapp-config-hostname-add) | 將自訂網域對應至函式應用程式。 |
| [az functionapp config ssl upload](https://docs.microsoft.com/cli/azure/functionapp/config/ssl#az-functionapp-config-ssl-upload) | 將 SSL 憑證上傳至函式應用程式。 |
| [az functionapp config ssl bind](https://docs.microsoft.com/cli/azure/functionapp/config/ssl#az-functionapp-config-ssl-bind) | 將上傳的 SSL 憑證繫結至函式應用程式。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure App Service 文件](../functions-cli-samples.md)中找到其他的 App Service CLI 指令碼範例。
