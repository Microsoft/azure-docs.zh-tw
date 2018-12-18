---
title: 包含適用於 Azure DNS 的 PowerShell 檔案
description: 包含適用於 Azure DNS 的 PowerShell 檔案
services: dns
author: subsarma
ms.service: dns
ms.topic: include file for PowerShell for Azure DNS
ms.date: 03/21/2018
ms.author: subsarma
ms.custom: include file for PowerShell for Azure DNS
ms.openlocfilehash: 388bea528b138d78b9ec23ceea295108306c61e9
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31613597"
---
## <a name="set-up-azure-powershell-for-azure-dns"></a>針對 Azure DNS 設定 Azure PowerShell SDK

### <a name="before-you-begin"></a>開始之前

在開始設定之前，請確認您具備下列項目。

* Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
* 您必須安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。

此外，若要使用私人區域 (公開預覽)，您必須確定您有下列 PowerShell 模組和版本。 
* AzureRM.Dns - [4.1.0 版](https://www.powershellgallery.com/packages/AzureRM.Dns/4.1.0)或更新版本
* AzureRM.Network - [5.4.0 版](https://www.powershellgallery.com/packages/AzureRM.Network/5.4.0)或更新版本

```powershell 
Find-Module -Name AzureRM.Dns 
``` 
 
```powershell 
Find-Module -Name AzureRM.Network 
``` 
 
上述命令的輸出必須顯示 AzureRM.Dns 的版本是 4.1.0 或更新版本，而 AzureRM.Network 是 5.4.0 或更新版本。  

如果您的系統使用較舊的版本，您可以安裝最新版的 Azure PowerShell，或使用上方位於模組版本旁的連結，從 PowerShell 資源庫下載並安裝上述模組。 接著，您可以使用下列命令加以安裝。 這兩個都是必要的模組，且都具有完整的回溯相容性。 

```powershell
Install-Module -Name AzureRM.Dns -Force
```

```powershell
Install-Module -Name AzureRM.Network -Force
```

### <a name="sign-in-to-your-azure-account"></a>登入您的 Azure 帳戶

開啟 PowerShell 主控台並連接到您的帳戶。 如需詳細資訊，請參閱[搭配使用 PowerShell 與 Resource Manager](../articles/azure-resource-manager/powershell-azure-resource-manager.md)。

```powershell
Connect-AzureRmAccount
```

### <a name="select-the-subscription"></a>選取訂用帳戶
 
檢查帳戶的訂用帳戶。

```powershell
Get-AzureRmSubscription
```

選擇其中一個要使用的 Azure 訂用帳戶。

```powershell
Select-AzureRmSubscription -SubscriptionName "your_subscription_name"
```

### <a name="create-a-resource-group"></a>建立資源群組

Azure 資源管理員需要所有的資源群組指定一個位置。 此位置用來作為該資源群組中資源的預設位置。 然而，因為所有 DNS 資源是全球性，而非區域性，資源群組位置的選擇不會對 Azure DNS 造成影響。

如果您使用現有的資源群組，則可略過此步驟。

```powershell
New-AzureRmResourceGroup -Name MyAzureResourceGroup -location "West US"
```

### <a name="register-resource-provider"></a>註冊資源提供者

Azure DNS 服務由 Microsoft.Network 資源提供者管理。 您的 Azure 訂用帳戶必須註冊為使用此資源提供者，您才能使用 Azure DNS。 每個訂用帳戶只需執行一次此作業。

```powershell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```
