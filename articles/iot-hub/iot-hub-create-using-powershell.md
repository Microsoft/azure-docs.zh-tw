---
title: 使用 PowerShell Cmdlet 建立 Azure IoT 中樞 | Microsoft Docs
description: 如何使用 PowerShell Cmdlet 建立 IoT 中樞。
author: robinsh
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.author: robinsh
ms.openlocfilehash: 7ecd35ba33d2860ba052aa27286c69985c2f7dd9
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190225"
---
# <a name="create-an-iot-hub-using-the-new-azurermiothub-cmdlet"></a>使用 New-AzureRmIotHub Cmdlet 建立 IoT 中樞

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>簡介

您可以使用 Azure PowerShell Cmdlet 建立及管理 Azure IoT 中樞。 此教學課程會示範如何使用 PowerShell 建立 IoT 中樞。

若要完成此操作說明，您需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="connect-to-your-azure-subscription"></a>連接到 Azure 訂用帳戶

如果您使用 Cloud Shell，表示您已登入訂用帳戶。 但如果您是在本機執行 PowerShell，請輸入下列命令來登入您的 Azure 訂用帳戶：

```powershell
# Log into Azure account.
Login-AzureRMAccount
```

## <a name="create-a-resource-group"></a>建立資源群組

您需要一個資源群組來部署 IoT 中樞。 您可以使用現有的資源群組，或建立一個新的群組。

若要為您的 IoT 中樞建立資源群組，請使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/AzureRM.Resources/New-AzureRmResourceGroup) 命令。 此範例會在**美國東部**區域中建立一個稱為 **MyIoTRG1** 的資源群組：

```azurepowershell-interactive
New-AzureRmResourceGroup -Name MyIoTRG1 -Location "East US"
```

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

若要在您於上一個步驟中建立的資源群組中建立 IoT 中樞，請使用 [New-AzureRmIotHub](https://docs.microsoft.com/powershell/module/AzureRM.IotHub/New-AzureRmIotHub) 命令。 此範例會在**美國東部**區域中建立一個稱為 **MyTestIoTHub** 的 **S1** 中樞：

```azurepowershell-interactive
New-AzureRmIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub `
    -SkuName S1 -Units 1 `
    -Location "East US"
```

IoT 中樞名稱必須是全域唯一的。

[!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

您可以使用 [Get-AzureRmIotHub](https://docs.microsoft.com/powershell/module/AzureRM.IotHub/Get-AzureRmIotHub) 命令來列出您訂用帳戶中的所有 IoT 中樞︰

```azurepowershell-interactive
Get-AzureRmIotHub
```

此範例會顯示您在上一個步驟中建立的 S1 標準 IoT 中樞。

您可以使用 [Remove-AzureRmIotHub](https://docs.microsoft.com/powershell/module/azurerm.iothub/remove-azurermiothub) 命令刪除該 IoT 中樞：

```azurepowershell-interactive
Remove-AzureRmIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub
```

或者，您可以使用 [Remove-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/AzureRM.Resources/Remove-AzureRmResourceGroup) 命令移除資源群組，及其包含的所有資源：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name MyIoTRG1
```

## <a name="next-steps"></a>後續步驟

現在您已經使用 PowerShell Cmdlet 部署 IoT 中樞，若要進一步探索，請參閱下列文章︰

* [搭配 IoT 中樞使用的 PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.iothub/)。

* [IoT 中樞資源提供者 REST API](https://docs.microsoft.com/rest/api/iothub/iothubresource)。

若要深入了解如何開發 IoT 中樞，請參閱以下文章︰

* [C SDK 簡介](iot-hub-device-sdk-c-intro.md)

* [Azure IoT SDK](iot-hub-devguide-sdks.md)

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置](../iot-edge/tutorial-simulate-device-linux.md)