---
title: 使用 PowerShell 來管理 Azure 服務匯流排資源 | Microsoft Docs
description: 使用 PowerShell 模組來建立及管理服務匯流排資源
services: service-bus-messaging
documentationcenter: .NET
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/21/2017
ms.author: spelluru
ms.openlocfilehash: 66475d9d59c8825eef7eb16e826d9813e819aa8a
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699622"
---
# <a name="use-powershell-to-manage-service-bus-resources"></a>使用 PowerShell 來管理服務匯流排資源

Microsoft Azure PowerShell 是一種指令碼環境，可讓您用來控制及自動化 Azure 服務的部署和管理。 本文說明如何使用本機 Azure PowerShell 主控台或指令碼，運用[服務匯流排 Resource Manager PowerShell 模組](/powershell/module/azurerm.servicebus)來佈建及管理服務匯流排實體 (命名空間、佇列、主題和訂用帳戶)。

您也可以使用 Azure Resource Manager 範本來管理服務匯流排實體。 如需詳細資訊，請參閱[使用 Azure Resource Manager 範本建立服務匯流排資源](service-bus-resource-manager-overview.md)文章。

## <a name="prerequisites"></a>必要條件

開始之前，您需要下列必要條件：

* Azure 訂用帳戶。 如需取得訂用帳戶的詳細資訊，請參閱[購買選項][purchase options]、[成員供應項目][member offers]或[免費帳戶][free account]。
* 具備 Azure PowerShell 的電腦。 如需指示，請參閱[開始使用 Azure PowerShell Cmdlet](/powershell/azure/get-started-azureps)。
* 大致了解 PowerShell 指令碼、NuGet 封裝和 .NET Framework。

## <a name="get-started"></a>開始使用

第一個步驟是使用 PowerShell 來登入 Azure 帳戶和 Azure 訂用帳戶。 遵循[開始使用 Azure PowerShell Cmdlet](/powershell/azure/get-started-azureps) 中的指示登入您的 Azure 帳戶，並擷取及存取 Azure 訂用帳戶中的資源。

## <a name="provision-a-service-bus-namespace"></a>佈建服務匯流排命名空間

使用服務匯流排命名空間時，您可以使用 [Get-AzureRmServiceBusNamespace](/powershell/module/azurerm.servicebus/get-azurermservicebusnamespace)、[New-AzureRmServiceBusNamespace](/powershell/module/azurerm.servicebus/new-azurermservicebusnamespace)、[Remove-AzureRmServiceBusNamespace](/powershell/module/azurerm.servicebus/remove-azurermservicebusnamespace) 和 [Set-AzureRmServiceBusNamespace](/powershell/module/azurerm.servicebus/set-azurermservicebusnamespace) Cmdlet。

這個範例會在指令碼中建立幾個區域變數：`$Namespace` 和 `$Location`。

* `$Namespace` 為我們想要使用之服務匯流排命名空間的名稱。
* `$Location` 會識別我們在其中佈建命名空間的資料中心。
* `$CurrentNamespace` 會儲存我們擷取 (或建立) 的參考命名空間。

在實際的指令碼中，`$Namespace` 和 `$Location` 可以參數的方式傳遞。

這部分的指令碼會執行下列作業：

1. 嘗試擷取具有指定名稱的服務匯流排命名空間。
2. 如果找到命名空間，它會回報找到的項目。
3. 如果找不到命名空間，它會建立命名空間，然後擷取新建立的命名空間。
   
    ``` powershell
    # Query to see if the namespace currently exists
    $CurrentNamespace = Get-AzureRMServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
   
    # Check if the namespace already exists or needs to be created
    if ($CurrentNamespace)
    {
        Write-Host "The namespace $Namespace already exists in the $Location region:"
        # Report what was found
        Get-AzureRMServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
    }
    else
    {
        Write-Host "The $Namespace namespace does not exist."
        Write-Host "Creating the $Namespace namespace in the $Location region..."
        New-AzureRmServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace -Location $Location
        $CurrentNamespace = Get-AzureRMServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
        Write-Host "The $Namespace namespace in Resource Group $ResGrpName in the $Location region has been successfully created."
                
    }
    ```

### <a name="create-a-namespace-authorization-rule"></a>建立命名空間授權規則

下列範例示範如何使用 [New-AzureRmServiceBusNamespaceAuthorizationRule](/powershell/module/azurerm.servicebus/new-azurermservicebusnamespaceauthorizationrule)、[Get-AzureRmServiceBusNamespaceAuthorizationRule](/powershell/module/azurerm.servicebus/get-azurermservicebusnamespaceauthorizationrule)、[Set-AzureRmServiceBusNamespaceAuthorizationRule](/powershell/module/azurerm.servicebus/set-azurermservicebusnamespaceauthorizationrule) 和 [Remove-AzureRmServiceBusNamespaceAuthorizationRule Cmdlet](/powershell/module/azurerm.servicebus/remove-azurermservicebusnamespaceauthorizationrule) 來管理命名空間授權規則。

```powershell
# Query to see if rule exists
$CurrentRule = Get-AzureRmServiceBusNamespaceAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

# Check if the rule already exists or needs to be created
if ($CurrentRule)
{
    Write-Host "The $AuthRule rule already exists for the namespace $Namespace."
}
else
{
    Write-Host "The $AuthRule rule does not exist."
    Write-Host "Creating the $AuthRule rule for the $Namespace namespace..."
    New-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule -Rights @("Listen","Send")
    $CurrentRule = Get-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule
    Write-Host "The $AuthRule rule for the $Namespace namespace has been successfully created."

    Write-Host "Setting rights on the namespace"
    $authRuleObj = Get-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

    Write-Host "Remove Send rights"
    $authRuleObj.Rights.Remove("Send")
    Set-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

    Write-Host "Add Send and Manage rights to the namespace"
    $authRuleObj.Rights.Add("Send")
    Set-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj
    $authRuleObj.Rights.Add("Manage")
    Set-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

    Write-Host "Show value of primary key"
    $CurrentKey = Get-AzureRmServiceBusKey -ResourceGroup $ResGrpName -NamespaceName $Namespace -Name $AuthRule
        
    Write-Host "Remove this authorization rule"
    Remove-AzureRmServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -Name $AuthRule
}
```

## <a name="create-a-queue"></a>建立佇列

若要建立佇列或主題，請使用上一節的指令碼來執行命名空間檢查。 然後，建立佇列：

```powershell
# Check if queue already exists
$CurrentQ = Get-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName

if($CurrentQ)
{
    Write-Host "The queue $QueueName already exists in the $Location region:"
}
else
{
    Write-Host "The $QueueName queue does not exist."
    Write-Host "Creating the $QueueName queue in the $Location region..."
    New-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -EnablePartitioning $True
    $CurrentQ = Get-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName
    Write-Host "The $QueueName queue in Resource Group $ResGrpName in the $Location region has been successfully created."
}
```

### <a name="modify-queue-properties"></a>修改佇列屬性

在執行上一節的指令碼後，您可以使用 [Set-AzureRmServiceBusQueue](/powershell/module/azurerm.servicebus/set-azurermservicebusqueue) Cmdlet 來更新佇列的屬性，如下列範例所示︰

```powershell
$CurrentQ.DeadLetteringOnMessageExpiration = $True
$CurrentQ.MaxDeliveryCount = 7
$CurrentQ.MaxSizeInMegabytes = 2048
$CurrentQ.EnableExpress = $True

Set-AzureRmServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -QueueObj $CurrentQ
```

## <a name="provisioning-other-service-bus-entities"></a>佈建其他服務匯流排實體

您可以使用[服務匯流排 PowerShell 模組](/powershell/module/azurerm.servicebus)來佈建其他實體，例如主題和訂用帳戶。 這些 Cmdlet 在語法上類似於上一節中所示範的佇列建立 Cmdlet。

## <a name="next-steps"></a>後續步驟

- 請在[這裡](/powershell/module/azurerm.servicebus)參閱完整的服務匯流排 Resource Manager PowerShell 模組文件。 此頁面會列出所有可用的 Cmdlet。
- 如需使用 Azure Resource Manager 範本的相關資訊，請參閱[使用 Azure Resource Manager 範本建立服務匯流排資源](service-bus-resource-manager-overview.md)文章。
- [服務匯流排 .NET 管理程式庫](service-bus-management-libraries.md)的相關資訊。

管理服務匯流排實體有一些替代方式，如這些部落格文章中所述︰

* [如何使用 PowerShell 指令碼來建立服務匯流排佇列、主題及訂閱](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
* [如何使用 PowerShell 指令碼來建立服務匯流排命名空間與事件中樞](http://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)
* [服務匯流排 PowerShell 指令碼](https://code.msdn.microsoft.com/Service-Bus-PowerShell-a46b7059)

<!--Anchors-->

[purchase options]: http://azure.microsoft.com/pricing/purchase-options/
[member offers]: http://azure.microsoft.com/pricing/member-offers/
[free account]: http://azure.microsoft.com/pricing/free-trial/
