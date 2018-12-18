---
title: 在您連線至 Azure Key Vault 時對 ASP.NET 專案所做的變更 | Microsoft Docs
description: 說明您使用 Visual Studio 連線服務連線至 Key Vault 時，ASP.NET 專案會發生什麼情形。
services: key-vault
author: ghogen
manager: douge
tags: azure-resource-manager
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.topic: conceptual
ms.date: 04/15/2018
ms.author: ghogen
ms.openlocfilehash: a74e4e10681f6b91e028067d8985408b0745dcd2
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140278"
---
# <a name="what-happened-to-my-aspnet-project-visual-studio-key-vault-connected-service"></a>我的 ASP.NET 專案 (Visual Studio Key Vault 連線服務) 發生什麼情形？

> [!div class="op_single_selector"]
> - [開始使用](vs-key-vault-aspnet-get-started.md)
> - [發生什麼情形](vs-key-vault-aspnet-what-happened.md)

本文說明在[使用 Visual Studio 新增 Key Vault 連線服務](vs-key-vault-add-connected-service.md)時，會對 ASP.NET 專案進行哪些明確的變更。

如需使用連線服務的相關資訊，請參閱[開始使用](vs-key-vault-aspnet-get-started.md)。

## <a name="added-references"></a>新增的參考

影響專案檔 *.NET 參考和 `packages.config` (NuGet 參考)。

| 類型 | 參考 |
| --- | --- |
| .NET; NuGet | Microsoft.Azure.KeyVault |
| .NET; NuGet | Microsoft.Azure.KeyVault.WebKey |
| .NET; NuGet | Microsoft.Rest.ClientRuntime |
| .NET; NuGet | Microsoft.Rest.ClientRuntime.Azure |

## <a name="added-files"></a>已新增檔案

- 已新增 ConnectedService.json，其中記錄一些關於連線服務提供者、版本及文件連結的資訊。

## <a name="project-file-changes"></a>專案檔變更

- 已新增連線服務 ItemGroup 和 ConnectedServices.json 檔案。
- 對[新增的參考](#added-references)一節中所述 .NET 組件的參考。

## <a name="webconfig-or-appconfig-changes"></a>web.config 或 app.config 的變更

- 已新增下列組態項目：

    ```xml
    <appSettings>
       <add key="vaultName" value="<your Key Vault name>" />
       <add key="vaultUri" value="<the URI to your Key Vault in Azure>" />
    </appSettings>
    ```

## <a name="changes-on-azure"></a>Azure 上的變更

- 已建立資源群組 (或使用了現有的資源群組)。
- 已在指定的資源群組中建立 Key Vault。

