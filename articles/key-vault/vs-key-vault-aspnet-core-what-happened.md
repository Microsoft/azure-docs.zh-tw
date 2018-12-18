---
title: 在您連線至 Azure Key Vault 時對 ASP.NET Core 專案所做的變更 | Microsoft Docs
description: 說明您使用 Visual Studio 連線服務連線至 Key Vault 時，ASP.NET Core 專案會發生什麼情形。
services: key-vault
author: ghogen
manager: douge
tags: azure-resource-manager
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 04/15/2018
ms.author: ghogen
ms.openlocfilehash: 8b6c590344db2997c1a987da14cabba76cdca83b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33781577"
---
# <a name="what-happened-to-my-aspnet-core-project-visual-studio-key-vault-connected-service"></a>我的 ASP.NET Core 專案 (Visual Studio Key Vault 連線服務) 發生什麼情形？

> [!div class="op_single_selector"]
> - [開始使用](vs-key-vault-aspnet-core-get-started.md)
> - [發生什麼情形](vs-key-vault-aspnet-core-what-happened.md)

本文說明在[使用 Visual Studio 新增 Key Vault 連線服務](vs-key-vault-add-connected-service.md)時，會對 ASP.NET 專案進行哪些明確的變更。

如需使用連線服務的相關資訊，請參閱[開始使用](vs-key-vault-aspnet-core-get-started.md)。

## <a name="added-references"></a>新增的參考

影響專案檔 *.NET 參考和 NuGet 套件參考。

| 類型 | 參考 |
| --- | --- |
| NuGet | Microsoft.AspNetCore.AzureKeyVault.HostingStartup |

## <a name="added-files"></a>已新增檔案

- 已新增 ConnectedService.json，其中記錄一些關於連線服務提供者、版本及文件連結的資訊。

## <a name="project-file-changes"></a>專案檔變更

- 已新增連線服務 ItemGroup 和 ConnectedServices.json 檔案。

## <a name="launchsettingsjson-changes"></a>launchsettings.json 變更

- 已將下列環境變數項目新增至 IIS Express 設定檔及符合您 Web 專案名稱的設定檔：

    ```json
      "environmentVariables": {
        "ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONENABLED": "true",
        "ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONVAULT": "<your keyvault URL>"
      }
    ```

## <a name="changes-on-azure"></a>Azure 上的變更

- 已建立資源群組 (或使用了現有的資源群組)。
- 已在指定的資源群組中建立 Key Vault。

