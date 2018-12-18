---
title: 快速入門 - 使用 PowerShell 建立 Azure Analysis Services 伺服器 | Microsoft Docs
description: 了解如何使用 PowerShell 來建立 Azure Analysis Services 伺服器
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: quickstart
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 9149f15d0503b9a39ac67d9c3f6fc1ddde7e03bd
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37952761"
---
# <a name="quickstart-create-a-server---powershell"></a>快速入門：建立伺服器 - PowerShell

本快速入門說明如何從命令列使用 PowerShell 在 Azure 訂用帳戶中建立 Azure Analysis Services 伺服器。

## <a name="prerequisites"></a>先決條件

- **Azure 訂用帳戶**︰瀏覽 [Azure 免費試用](https://azure.microsoft.com/offers/ms-azr-0044p/)建立帳戶。
- **Azure Active Directory**：您的訂用帳戶必須與 Azure Active Directory 租用戶相關聯，而且您必須在該目錄中有一個帳戶。 若要深入了解，請參閱[驗證和使用者權限](analysis-services-manage-users.md)。
- **Azure PowerShell 模組 4.0 版或更新版本**。 若要尋找版本，請執行 ` Get-Module -ListAvailable AzureRM`。 若要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

## <a name="import-azurermanalysisservices-module"></a>Import AzureRm.AnalysisServices 模組

若要在您的訂用帳戶中建立伺服器，您可使用 [AzureRM.AnalysisServices](https://www.powershellgallery.com/packages/AzureRM.AnalysisServices) 元件模組。 將 AzureRm.AnalysisServices 模組載入 PowerShell 工作階段中。

```powershell
Import-Module AzureRM.AnalysisServices
```

## <a name="log-in-to-azure"></a>登入 Azure

使用 [Connect-AzureRmAccount](/powershell/module/azurerm.profile/connect-azurermaccount) 命令登入您的 Azure 訂用帳戶。 遵循螢幕上的指示進行。

```powershell
Connect-AzureRmAccount
```

## <a name="create-a-resource-group"></a>建立資源群組

[Azure 資源群組](../azure-resource-manager/resource-group-overview.md)是在其中以群組方式部署與管理 Azure 資源的邏輯容器。 當您建立伺服器時，必須指定您訂用帳戶中的資源群組。 如果您還沒有資源群組，可以使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令建立一個新的資源群組。 下列範例會在美國西部區域中建立一個名為 `myResourceGroup` 的資源群組。

```powershell
New-AzureRmResourceGroup -Name "myResourceGroup" -Location "WestUS"
```

## <a name="create-a-server"></a>建立伺服器

使用 [New-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/new-azurermanalysisservicesserver) 命令建立新的伺服器。 下列範例會在美國西部地區 (免費的 D1 層) 的 myResourceGroup 中建立名為 myServer 的伺服器，並將 philipc@adventureworks.com 指定為伺服器管理員。

```powershell
New-AzureRmAnalysisServicesServer -ResourceGroupName "myResourceGroup" -Name "myserver" -Location WestUS -Sku D1 -Administrator "philipc@adventure-works.com"
```

## <a name="clean-up-resources"></a>清除資源

您也可以使用 [Remove-AzureRmAnalysisServicesServer](/powershell/module/azurerm.analysisservices/new-azurermanalysisservicesserver) 命令，從您的訂用帳戶中移除伺服器。 如果您繼續進行本集合中的其他快速入門和教學課程，請勿移除您的伺服器。 下列範例會移除在前一個步驟中建立的伺服器。


```powershell
Remove-AzureRmAnalysisServicesServer -Name "myserver" -ResourceGroupName "myResourceGroup"
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何使用 PowerShell 在 Azure 訂用帳戶中建立伺服器。 既然您有伺服器，您可設定 (選擇性) 伺服器防火牆來協助保護其安全。 您也可以直接從入口網站，將基本範例資料模型新增到您的伺服器。 範例模型有助於您了解如何設定模型資料庫角色以及測試用戶端連線。 若要深入了解，請繼續進行新增範例模型的教學課程。

> [!div class="nextstepaction"]
> [快速入門：設定伺服器防火牆 - 入口網站](analysis-services-qs-firewall.md)      
> [!div class="nextstepaction"]
> [教學課程：將範例模型新增至您的伺服器](analysis-services-create-sample-model.md)