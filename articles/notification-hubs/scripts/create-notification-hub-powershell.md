---
title: PowerShell 指令碼： 建立 Azure 通知中樞 |Microsoft Docs
description: 這個 PowerShell 指令碼會建立 Azure 通知中樞範例。
services: data-factory
author: dimazaid
manager: kpiteira
editor: spelluru
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 747d743a0573bd959b4d3c7100be8ae9451c5ed5
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33786727"
---
# <a name="use-powershell-to-create-an-azure-notification-hub"></a>使用 PowerShell 建立 Azure 通知中樞

這個 PowerShell 指令碼範例會建立 Azure 通知中樞範例。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>先決條件
* **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/notification-hubs/create-notification-hub/create-notification-hub.ps1 "Create a notification hub")]


## <a name="clean-up-deployment"></a>清除部署

執行範例指令碼之後，您可以使用下列命令以移除資源群組及與其相關聯的所有資源：

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令： 

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmNotificationHubsNamespace](/powershell/module//azurerm.notificationhubs/new-azurermnotificationhubsnamespace) | 建立通知中樞的命名空間。 |
| [New-AzureRmNotificationHub](/powershell/module//azurerm.notificationhubs/new-azurermnotificationhubsnamespace) | 建立通知中樞。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |
|||

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。
