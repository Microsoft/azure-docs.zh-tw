---
services: virtual-machines
title: 設定 PowerShell
author: JoeDavies-MSFT
solutions: ''
manager: timlt
editor: tysonn
ms.service: virtual-machines
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 05/12/2015
ms.author: rasquill
ms.openlocfilehash: 19c704d965ff3e2fc9ac8c5b623aeb386cb0b974
ms.sourcegitcommit: e19f6a1709b0fe0f898386118fbef858d430e19d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/16/2018
ms.locfileid: "26730519"
---
## <a name="setting-up-powershell"></a>設定 PowerShell
請先遵循下列步驟，您才可以使用 Azure PowerShell。

### <a name="verify-powershell-versions"></a>確認 PowerShell 版本
您必須具備 Windows PowerShell 3.0 版或 4.0 版，才可以使用 Windows PowerShell。 若要尋找 Windows PowerShell 版本，請在 Windows PowerShell 命令提示字元中輸入這個命令。

    $PSVersionTable

您應該會看到如下的結果。

    Name                           Value
    ----                           -----
    PSVersion                      3.0
    WSManStackVersion              3.0
    SerializationVersion           1.1.0.1
    CLRVersion                     4.0.30319.18444
    BuildVersion                   6.2.9200.16481
    PSCompatibleVersions           {1.0, 2.0, 3.0}
    PSRemotingProtocolVersion      2.2

確認 **PSVersion** 的值是 3.0 或 4.0。 若要安裝相容版本，請參閱 [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855)。

您應該也有 Azure PowerShell 0.8.0 版或更新版本。 您可以在 Azure PowerShell 命令提示字元下使用這個命令來檢查已安裝的 Azure PowerShell 版本。

    Get-Module azure | format-table version

您應該會看到如下的結果。

    Version
    -------
    0.8.16.1

如需最新版本的指示與連結，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。

### <a name="set-your-azure-account-and-subscription"></a>設定 Azure 帳戶和訂用帳戶
如果您還沒有 Azure 訂用帳戶，則可以啟用 [MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或申請[免費試用](https://azure.microsoft.com/pricing/free-trial/)。

使用這個命令，開啟 Azure PowerShell 命令提示字元，並登入 Azure。

    Add-AzureAccount

如果您有多個 Azure 訂用帳戶，則可以使用這個命令列出 Azure 訂用帳戶。

    Get-AzureSubscription

您會收到下列類型的資訊：

    SubscriptionId            : fd22919d-eaca-4f2b-841a-e4ac6770g92e
    SubscriptionName          : Visual Studio Ultimate with MSDN
    Environment               : AzureCloud
    SupportedModes            : AzureServiceManagement,AzureResourceManager
    DefaultAccount            : johndoe@contoso.com
    Accounts                  : {johndoe@contoso.com}
    IsDefault                 : True
    IsCurrent                 : True
    CurrentStorageAccountName : 
    TenantId                  : 32fa88b4-86f1-419f-93ab-2d7ce016dba7

您可以在 Azure PowerShell 命令提示字元下執行這些命令，設定目前 Azure 訂用帳戶。 以正確的名稱取代括號中 (包括 < 和 > 字元) 的所有內容。

    $subscr="<SubscriptionName from the display of Get-AzureSubscription>"
    Select-AzureSubscription -SubscriptionName $subscr -Current    

如需 Azure 訂用帳戶和帳戶的詳細資訊，請參閱[如何：連線至訂用帳戶](/powershell/azureps-cmdlets-docs#Connect)。

