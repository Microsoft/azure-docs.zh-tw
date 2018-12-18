---
title: LUIS 的 Azure Cloud Shell 使用量資料
titleSuffix: Azure Cognitive Services
description: 了解如何在 Azure Cloud Shell 中取得 LUIS 的使用量資訊。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2017
ms.author: diberry
ms.openlocfilehash: 42bd8ef95ac1b55e76fe1cf0702b79de91832741
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48870174"
---
# <a name="usage-data-for-luis-service-from-azure-cloud-shell"></a>從 Azure Cloud Shell 管理 LUIS 服務的使用量資料
Azure 入口網站可讓您透過 PowerShell Cmdlet 來使用 LUIS 資源。 

這些 Cmdlet 可讓您[建立](https://docs.microsoft.com/powershell/module/azurerm.cognitiveservices/new-azurermcognitiveservicesaccount?view=azurermps-6.0.0) LUIS 訂用帳戶、取得訂用帳戶的相關資訊 (包括[使用量](https://docs.microsoft.com/powershell/module/azurerm.cognitiveservices/get-azurermcognitiveservicesaccountusage?view=azurermps-6.0.0))，以及[移除](https://docs.microsoft.com/powershell/module/azurerm.cognitiveservices/remove-azurermcognitiveservicesaccount?view=azurermps-6.0.0)訂用帳戶。 

## <a name="cloud-shell-storage-account-and-authentication"></a>Cloud Shell 儲存體帳戶和驗證
若要在 Azure 入口網站 [Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/quickstart-powershell) 中使用 PowerShell，您必須擁有 Azure 儲存體帳戶。 如果您沒有[儲存體帳戶](https://docs.microsoft.com/azure/cloud-shell/persisting-shell-storage)，系統會提示您建立帳戶。 儲存體帳戶可讓您將 PowerShell 指令碼儲存在 Cloud Shell 中。  

您也需要在 Cloud Shell 向 Azure 進行驗證，才能存取資源。 

在您擁有儲存體帳戶並通過驗證後，即可執行 PowerShell Cmdlet。

## <a name="open-cloud-shell"></a>開啟 Cloud Shell
當您使用 Azure 入口網站 Cloud Shell 時，您一律會保有最新的 PowerShell 版本。 

請使用 [啟動 Cloud Shell] 按鈕開啟 Cloud Shell，或透過 [https://shell.azure.com](https://shell.azure.com) 開啟瀏覽器。 

<a style="cursor:pointer" onclick='javascript:window.open("https://shell.azure.com", "_blank", "toolbar=no,scrollbars=yes,resizable=yes,menubar=no,location=no,status=no")'><image src="https://shell.azure.com/images/launchcloudshell.png" /></a>

## <a name="luis-endpoint-usage-information"></a>LUIS 端點使用量資訊

PowerShell 6.x Cmdlet `Get-AzureRmCognitiveServicesAccountUsage` 會針對包含 LUIS 的 Microsoft 認知服務提供使用量資訊。 [Get-AzureRmCognitiveServicesAccountUsage](https://docs.microsoft.com/powershell/module/azurerm.cognitiveservices/get-azurermcognitiveservicesaccountusage?view=azurermps-6.0.0) 需要服務的資源群組和資源名稱。 

命令語法為：

```
Get-AzureRmCognitiveServicesAccountUsage -ResourceGroupName my-resource-group -Name my-luis-service-name
```

在下列範例中，資源群組名稱為 `luis-westus-rg`，而 LUIS 服務訂用帳戶名稱為 `luis-westus-1`。 這兩個名稱都是在 LUIS 服務建立時所選擇的。 

此 Cmdlet 會傳回為期 30 天、結束日期為 6 月 7 日的期間內所使用的 16 次端點叫用 (共計 10,000 次) 的使用量資訊：

```
CurrentValue  : 16
Name          : LUIS.Calls
Limit         : 10000
Status        : Included
Unit          : Count
QuotaPeriod   : 30.00:00:00
NextResetTime : 2018-06-07T18:28:52Z
```

請在與 Cloud Shell 相關聯的 Azure 儲存體帳戶中將此命令儲存為 PowerShell 檔案 *.ps1，並隨時加以執行。 

![從儲存體執行指令碼](./media/luis-how-to-manage-from-powershell/run-script-from-storage.png)

當指令碼儲存到雲端磁碟機之後，您將可從 Azure 手機應用程式的 Cloud Shell 執行 PowerShell 指令碼。 

![從手機應用程式中的儲存體執行指令碼](./media/luis-how-to-manage-from-powershell/phone-app.png)