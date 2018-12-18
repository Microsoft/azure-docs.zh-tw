---
title: Azure PowerShell 指令碼範例 - 建立 Web 應用程式並從 GitHub 部署程式碼 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 建立 Web 應用程式並從 GitHub 部署程式碼
services: app-service\web
documentationcenter: ''
author: cephalin
manager: erikre
editor: ''
tags: azure-service-management
ms.assetid: 0f9c8bc5-3789-4eb3-8deb-ae6e2200795a
ms.service: app-service-web
ms.workload: web
ms.devlang: na
ms.topic: sample
ms.date: 03/20/2017
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: d60503e195c1a8f32992cfb1978ad7a181c069b0
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39323937"
---
# <a name="create-a-web-app-and-deploy-code-from-github"></a>建立 Web 應用程式並從 GitHub 部署程式碼

此範例指令碼會在 App Service 中建立 Web 應用程式及其相關資源，然後從公用 GitHub 存放庫部署 Web 應用程式程式碼 (不使用連續部署)。 如需進行使用連續部署的 GitHub 部署，請參閱[建立可從 GitHub 連續部署的 Web 應用程式](app-service-powershell-continuous-deployment-github.md)。

您可以視需要使用 [Azure PowerShell 指南](/powershell/azure/overview) \(英文\) 中的指示來安裝 Azure PowerShell，然後執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。 此外，您需要包含 Web 應用程式程式碼的 GitHub 儲存機制的連結。

## <a name="sample-script"></a>範例指令碼

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/deploy-github/deploy-github.ps1?highlight=1-2 "Create a web app and deploy code from GitHub")]

## <a name="clean-up-deployment"></a>清除部署 

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組、Web 應用程式和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmAppServicePlan](/powershell/module/azurerm.websites/new-azurermappserviceplan) | 建立 App Service 方案。 |
| [New-AzureRmWebApp](/powershell/module/azurerm.websites/new-azurermwebapp) | 建立 Web 應用程式。 |
| [Set-AzureRmResource](/powershell/module/azurerm.resources/set-azurermresource) | 修改資源群組中的資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure PowerShell 範例](../app-service-powershell-samples.md)中找到適用於 App Service Web Apps 的其他 Azure PowerShell 範例。
