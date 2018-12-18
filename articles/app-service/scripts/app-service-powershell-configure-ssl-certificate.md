---
title: Azure PowerShell 指令碼範例 - 將自訂 SSL 憑證繫結至 Web 應用程式 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 將自訂 SSL 憑證繫結至 Web 應用程式
services: app-service\web
documentationcenter: ''
author: cephalin
manager: erikre
editor: ''
tags: azure-service-management
ms.assetid: 23e83b74-614a-49a0-bc08-7542120eeec5
ms.service: app-service-web
ms.workload: web
ms.devlang: na
ms.topic: sample
ms.date: 03/20/2017
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: 4ab6ad5e8a022c577e9382cde7bdcb3059a18a14
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39323801"
---
# <a name="bind-a-custom-ssl-certificate-to-a-web-app"></a>將自訂 SSL 憑證繫結至 Web 應用程式

此範例指令碼會在 App Service 中建立 Web 應用程式及其相關的資源，然後將自訂網域名稱的 SSL 憑證加以繫結。 

您可以視需要使用 [Azure PowerShell 指南](/powershell/azure/overview) \(英文\) 中的指示來安裝 Azure PowerShell，然後執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。 此外，請確定：

- 已使用 `az login` 命令建立與 Azure 的連線。
- 您可以存取網域註冊機構的 DNS 設定頁面。
- 對於想要上傳並繫結的 SSL 憑證，您具備有效的 .PFX 檔案和其密碼。

## <a name="sample-script"></a>範例指令碼

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom SSL certificate to a web app")]

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
| [Set-AzureRmAppServicePlan](/powershell/module/azurerm.websites/set-azurermappserviceplan) | 修改 App Service 方案來變更其定價層。 |
| [Set-AzureRmWebApp](/powershell/module/azurerm.websites/set-azurermwebapp) | 修改 Web 應用程式的組態。 |
| [New-AzureRmWebAppSSLBinding](/powershell/module/azurerm.websites/new-azurermwebappsslbinding) | 建立 Web 應用程式的 SSL 憑證繫結。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure PowerShell 範例](../app-service-powershell-samples.md)中找到適用於 App Service Web Apps 的其他 Azure PowerShell 範例。
