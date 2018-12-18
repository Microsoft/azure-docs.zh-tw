---
title: 使用 PowerShell 建立 Azure API 管理執行個體 | Microsoft Docs
description: 請依照此教學課程的步驟來建立新的 Azure API 管理執行個體。
services: api-management
documentationcenter: ''
author: vladvino
manager: cflower
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: quickstart
ms.custom: mvc
ms.date: 11/15/2017
ms.author: apimpm
ms.openlocfilehash: 0bca8948e4afe156d468801f795c68c2143267f5
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225140"
---
# <a name="create-a-new-azure-api-management-service-instance"></a>建立新的 Azure API 管理服務執行個體

「Azure API 管理 (APIM)」可協助組織將 API 發佈給外部、合作夥伴及內部開發人員，以發揮其資料與服務的潛力。 「API 管理」提供的核心專長認證，透過開發人員參與、商務洞察力、分析、安全性和保護，可確保 API 程式獲致成功。 APIM 可讓您為裝載於任何位置的現有後端服務，建立和管理新式 API 閘道。 如需詳細資訊，請參閱[概觀](api-management-key-concepts.md)主題。

此快速入門描述使用 PowerShell 指令碼建立新的 API 管理執行個體的步驟。 快速入門會示範如何使用可以從 Azure 入口網站執行的 **Azure Cloud Shell**。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="log-in-to-azure"></a>登入 Azure

在 http://portal.azure.com 上登入 Azure 入口網站。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，此教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。


## <a name="create-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroup -Location WestUS
```

## <a name="create-an-api-management-service"></a>建立 API 管理服務

這是長時間執行的作業，而且可能需要最多 15 分鐘。

```azurepowershell-interactive
New-AzureRmApiManagement -ResourceGroupName "myResourceGroup" -Location "West US" -Name "apim-name" -Organization "myOrganization" -AdminEmail "myEmail" -Sku "Developer"
```

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 命令來移除資源群組和所有相關資源。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [匯入和發佈您的第一個 API](import-and-publish.md)
