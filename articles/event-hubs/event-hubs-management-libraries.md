---
title: Azure 事件中樞管理程式庫 | Microsoft Docs
description: 從 .NET 管理事件中樞命名空間和實體
services: event-hubs
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.devlang: dotnet
ms.topic: article
ms.date: 08/13/2018
ms.author: shvija
ms.openlocfilehash: 79cddcac4d469753bc39107e6db2d8ce901111d1
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746411"
---
# <a name="event-hubs-management-libraries"></a>事件中樞管理程式庫

您可以使用 Azure 事件中樞管理程式庫來動態佈建事件中樞命名空間和實體。 這個動態本質適合複雜的部署和傳訊案例，可讓您以程式設計方式決定要佈建的實體。 這些程式庫目前適用於 .NET。

## <a name="supported-functionality"></a>支援的功能

* 建立、更新、刪除命名空間
* 建立、更新、刪除事件中樞
* 建立、更新、刪除取用者群組

## <a name="prerequisites"></a>必要條件

若要開始使用事件中樞管理程式庫，您必須使用 Azure Active Directory (AAD) 來驗證。 AAD 會要求您以提供 Azure 資源存取權的服務主體來進行驗證。 如需建立服務主體的詳細資訊，請參閱以下其中一篇文章：  

* [使用 Azure 入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)
* [使用 Azure PowerShell 建立用來存取資源的服務主體](../azure-resource-manager/resource-group-authenticate-service-principal.md)
* [使用 Azure CLI 建立用來存取資源的服務主體](../azure-resource-manager/resource-group-authenticate-service-principal-cli.md)

這些教學課程會提供您 `AppId` (用戶端識別碼)、`TenantId` 和 `ClientSecret` (驗證金鑰)，全部由管理程式庫用於驗證。 針對您想要執行的資源群組，您必須具備「擁有者」權限。

## <a name="programming-pattern"></a>程式設計模式

操控任何事件中樞資源的模式，都會遵循共通的協定：

1. 使用程式庫中的 `Microsoft.IdentityModel.Clients.ActiveDirectory` 來從 AAD 取得權杖。
    ```csharp
    var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

    var result = await context.AcquireTokenAsync(
        "https://management.core.windows.net/",
        new ClientCredential(clientId, clientSecret)
    );
    ```

1. 建立 `EventHubManagementClient` 物件。
    ```csharp
    var creds = new TokenCredentials(token);
    var ehClient = new EventHubManagementClient(creds)
    {
        SubscriptionId = SettingsCache["SubscriptionId"]
    };
    ```

1. 將 `CreateOrUpdate` 參數設定為您指定的值。
    ```csharp
    var ehParams = new EventHubCreateOrUpdateParameters()
    {
        Location = SettingsCache["DataCenterLocation"]
    };
    ```

1. 執行呼叫。
    ```csharp
    await ehClient.EventHubs.CreateOrUpdateAsync(resourceGroupName, namespaceName, EventHubName, ehParams);
    ```

## <a name="next-steps"></a>後續步驟
* [.NET 管理範例](https://github.com/Azure-Samples/event-hubs-dotnet-management/)
* [Microsoft.Azure.Management.EventHub 參考](/dotnet/api/Microsoft.Azure.Management.EventHub) 
