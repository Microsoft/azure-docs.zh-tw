---
title: 取得應用程式驗證的值 - Azure SQL Database | Microsoft Docs
description: 建立服務主體以從程式碼存取 SQL Database。
services: sql-database
author: stevestein
manager: craigg
ms.service: sql-database
ms.custom: develop apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sstein
ms.openlocfilehash: f6d3599425669a6717a58bb1e5ac0af4b291d4d6
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34646321"
---
# <a name="get-the-required-values-for-authenticating-an-application-to-access-sql-database-from-code"></a>取得驗證應用程式以從程式碼存取 SQL Database 的所需值
若要從程式碼建立和管理 SQL Database，您必須在建立 Azure 資源所使用的訂用帳戶的 Azure Active Directory (AAD) 網域中註冊您的應用程式。

## <a name="create-a-service-principal-to-access-resources-from-an-application"></a>從應用程式建立用來存取資源的服務主體
您必須安裝並執行最新的 [Azure PowerShell](https://msdn.microsoft.com/library/mt619274.aspx)。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。

下列 PowerShell 指令碼會建立 Active Directory (AD) 應用程式以及驗證 C# 應用程式所需的服務主體。 指令碼會輸出先前 C# 範例所需的值。 如需詳細資訊，請參閱 [使用 Azure PowerShell 建立用來存取資源的服務主體](../azure-resource-manager/resource-group-authenticate-service-principal.md)。

    # Sign in to Azure.
    Connect-AzureRmAccount

    # If you have multiple subscriptions, uncomment and set to the subscription you want to work with.
    #$subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}"
    #Set-AzureRmContext -SubscriptionId $subscriptionId

    # Provide these values for your new AAD app.
    # $appName is the display name for your app, must be unique in your directory.
    # $uri does not need to be a real uri.
    # $secret is a password you create.

    $appName = "{app-name}"
    $uri = "http://{app-name}"
    $secret = "{app-password}"

    # Create a AAD app
    $azureAdApplication = New-AzureRmADApplication -DisplayName $appName -HomePage $Uri -IdentifierUris $Uri -Password $secret

    # Create a Service Principal for the app
    $svcprincipal = New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

    # To avoid a PrincipalNotFound error, I pause here for 15 seconds.
    Start-Sleep -s 15

    # If you still get a PrincipalNotFound error, then rerun the following until successful. 
    $roleassignment = New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $azureAdApplication.ApplicationId.Guid


    # Output the values we need for our C# application to successfully authenticate

    Write-Output "Copy these values into the C# sample app"

    Write-Output "_subscriptionId:" (Get-AzureRmContext).Subscription.SubscriptionId
    Write-Output "_tenantId:" (Get-AzureRmContext).Tenant.TenantId
    Write-Output "_applicationId:" $azureAdApplication.ApplicationId.Guid
    Write-Output "_applicationSecret:" $secret




## <a name="see-also"></a>另請參閱
* [以 C# 建立 SQL Database](sql-database-get-started-csharp.md)
* [使用 Azure Active Directory 驗證連線到 SQL Database](sql-database-aad-authentication.md)

