---
title: Azure 服務匯流排管理程式庫| Microsoft Docs
description: 從 .NET 管理服務匯流排命名空間和傳訊實體。
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/05/2018
ms.author: spelluru
ms.openlocfilehash: a959687fbf6e296cab7e0d8ca49ae97a005622cf
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696570"
---
# <a name="service-bus-management-libraries"></a>服務匯流排管理程式庫

Azure 服務匯流排管理程式庫可以動態佈建服務匯流排命名空間和實體。 這適合複雜的部署和傳訊案例，且可讓您以程式設計方式決定要佈建的實體。 這些程式庫目前適用於 .NET。

## <a name="supported-functionality"></a>支援的功能

* 建立、更新、刪除命名空間
* 建立、更新、刪除佇列
* 建立、更新、刪除主題
* 建立、更新、刪除訂用帳戶

## <a name="prerequisites"></a>必要條件

若要開始使用服務匯流排管理程式庫，您必須使用 Azure Active Directory (Azure AD) 服務來驗證。 Azure AD 會要求您以提供 Azure 資源存取權的服務主體來進行驗證。 如需建立服務主體的詳細資訊，請參閱以下其中一篇文章：  

* [使用 Azure 入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](/azure/azure-resource-manager/resource-group-create-service-principal-portal)
* [使用 Azure PowerShell 建立用來存取資源的服務主體](/azure/azure-resource-manager/resource-group-authenticate-service-principal)
* [使用 Azure CLI 建立用來存取資源的服務主體](/azure/azure-resource-manager/resource-group-authenticate-service-principal-cli)

這些教學課程會提供您 `AppId` (用戶端識別碼)、`TenantId` 和 `ClientSecret` (驗證金鑰)，全部由管理程式庫用於驗證。 針對您想要執行的資源群組，您必須具備「擁有者」權限。

## <a name="programming-pattern"></a>程式設計模式

操控任何服務匯流排資源的模式，都會遵循共通的協定：

1. 使用 **Microsoft.IdentityModel.Clients.ActiveDirectory** 程式庫從 Azure AD 取得權杖：
   ```csharp
   var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

   var result = await context.AcquireTokenAsync("https://management.core.windows.net/", new ClientCredential(clientId, clientSecret));
   ```
2. 建立 `ServiceBusManagementClient` 物件：

   ```csharp
   var creds = new TokenCredentials(token);
   var sbClient = new ServiceBusManagementClient(creds)
   {
       SubscriptionId = SettingsCache["SubscriptionId"]
   };
   ```
3. 將 `CreateOrUpdate` 參數設定為您指定的值：

   ```csharp
   var queueParams = new QueueCreateOrUpdateParameters()
   {
       Location = SettingsCache["DataCenterLocation"],
       EnablePartitioning = true
   };
   ```
4. 執行呼叫：

   ```csharp
   await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, QueueName, queueParams);
   ```

## <a name="next-steps"></a>後續步驟

* [.NET 管理範例](https://github.com/Azure-Samples/service-bus-dotnet-management/)
* [Microsoft.Azure.Management.ServiceBus API 參考](/dotnet/api/Microsoft.Azure.Management.ServiceBus)
