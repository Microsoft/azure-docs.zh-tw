---
title: 開始使用適用於 .NET 的 Azure CDN 程式庫 | Microsoft Docs
description: 了解如何使用 Visual Studio，撰寫 .NET 應用程式來管理 Azure CDN。
services: cdn
documentationcenter: .net
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 63cf4101-92e7-49dd-a155-a90e54a792ca
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 5379586355ece98af6295236d6cbd09cb31c742b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38454459"
---
# <a name="get-started-with-azure-cdn-development"></a>開始使用 Azure CDN 開發
> [!div class="op_single_selector"]
> * [Node.js](cdn-app-dev-node.md)
> * [.NET](cdn-app-dev-net.md)
> 
> 

您可以使用 [適用於 .NET 的 Azure CDN 程式庫](https://msdn.microsoft.com/library/mt657769.aspx) ，自動建立和管理 CDN 設定檔與端點。  本教學課程會逐步建立簡單的 .NET 主控台應用程式，示範數個可用的作業。  本教學課程的目的不是詳細說明適用於 .NET 的 Azure CDN 程式庫的所有層面。

您需要 Visual Studio 2015，才能完成本教學課程。  [Visual Studio Community 2015](https://www.visualstudio.com/products/visual-studio-community-vs.aspx) 可供免費下載。

> [!TIP]
> 您可以在 MSDN 上下載 [本教學課程中完成的專案](https://code.msdn.microsoft.com/Azure-CDN-Management-1f2fba2c) 。
> 
> 

[!INCLUDE [cdn-app-dev-prep](../../includes/cdn-app-dev-prep.md)]

## <a name="create-your-project-and-add-nuget-packages"></a>建立專案並新增 Nuget 封裝
現在，我們已為 CDN 設定檔建立資源群組，並為 Azure AD 應用程式授與權限來管理該群組內的 CDN 設定檔和端點，我們可以開始建立應用程式。

從 Visual Studio 2015 中，依序按一下 [檔案]、[新增] 和 [專案...]，以開啟 [新增專案] 對話方塊。  展開 [Visual C#]，然後選取左側窗格中的 [Windows]。  按一下中央窗格的 [主控台應用程式]  。  為專案命名，然後按一下 [確定] 。  

![New Project](./media/cdn-app-dev-net/cdn-new-project.png)

我們的專案將使用 Nuget 封裝內含的一些 Azure 程式庫。  讓我們先將它們新增至專案。

1. 依序按一下 [工具] 功能表、[Nuget 封裝管理員] 及 [封裝管理員主控台]。
   
    ![管理 Nuget 封裝](./media/cdn-app-dev-net/cdn-manage-nuget.png)
2. 在 [封裝管理員主控台] 中，執行下列命令來安裝 **Active Directory Authentication Library (ADAL)**：
   
    `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory`
3. 執行下列命令來安裝 **Azure CDN 管理程式庫**：
   
    `Install-Package Microsoft.Azure.Management.Cdn`

## <a name="directives-constants-main-method-and-helper-methods"></a>指示詞、常數、主要方法和協助程式方法
讓我們開始撰寫程式的基本結構。

1. 回到 [Program.cs] 索引標籤，使用下列內容取代頂端的 `using` 指示詞：
   
    ```csharp
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Management.Cdn;
    using Microsoft.Azure.Management.Cdn.Models;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    ```
2. 我們必須定義一些我們的方法將用到的常數。  在 `Program` 類別中，但在 `Main` 方法之前，新增下列內容。  務必視需要使用您自己的值來取代預留位置，包括 **&lt;角括號&gt;**。
   
    ```csharp
    //Tenant app constants
    private const string clientID = "<YOUR CLIENT ID>";
    private const string clientSecret = "<YOUR CLIENT AUTHENTICATION KEY>"; //Only for service principals
    private const string authority = "https://login.microsoftonline.com/<YOUR TENANT ID>/<YOUR TENANT DOMAIN NAME>";
   
    //Application constants
    private const string subscriptionId = "<YOUR SUBSCRIPTION ID>";
    private const string profileName = "CdnConsoleApp";
    private const string endpointName = "<A UNIQUE NAME FOR YOUR CDN ENDPOINT>";
    private const string resourceGroupName = "CdnConsoleTutorial";
    private const string resourceLocation = "<YOUR PREFERRED AZURE LOCATION, SUCH AS Central US>";
    ```
3. 此外，也要在類別層級設定這兩個變數。  稍後將使用這些項目來判斷我們的設定檔和端點是否已經存在。
   
    ```csharp
    static bool profileAlreadyExists = false;
    static bool endpointAlreadyExists = false;
    ```
4. 取代 `Main` 方法，如下所示：
   
   ```csharp
   static void Main(string[] args)
   {
       //Get a token
       AuthenticationResult authResult = GetAccessToken();
   
       // Create CDN client
       CdnManagementClient cdn = new CdnManagementClient(new TokenCredentials(authResult.AccessToken))
           { SubscriptionId = subscriptionId };
   
       ListProfilesAndEndpoints(cdn);
   
       // Create CDN Profile
       CreateCdnProfile(cdn);
   
       // Create CDN Endpoint
       CreateCdnEndpoint(cdn);
   
       Console.WriteLine();
   
       // Purge CDN Endpoint
       PromptPurgeCdnEndpoint(cdn);
   
       // Delete CDN Endpoint
       PromptDeleteCdnEndpoint(cdn);
   
       // Delete CDN Profile
       PromptDeleteCdnProfile(cdn);
   
       Console.WriteLine("Press Enter to end program.");
       Console.ReadLine();
   }
   ```
5. 在我們的其他方法中有一些會透過「是/否」問題來提示使用者。  新增下列方法，以使其更加容易：
   
    ```csharp
    private static bool PromptUser(string Question)
    {
        Console.Write(Question + " (Y/N): ");
        var response = Console.ReadKey();
        Console.WriteLine();
        if (response.Key == ConsoleKey.Y)
        {
            return true;
        }
        else if (response.Key == ConsoleKey.N)
        {
            return false;
        }
        else
        {
            // They pressed something other than Y or N.  Let's ask them again.
            return PromptUser(Question);
        }
    }
    ```

現在已撰寫了程式的基本結構，我們應該建立 `Main` 方法所呼叫的方法。

## <a name="authentication"></a>驗證
在可以使用 Azure CDN 管理庫之前，需要先驗證服務主體，並取得驗證權杖。  這個方法會使用 ADAL 來擷取權杖。

```csharp
private static AuthenticationResult GetAccessToken()
{
    AuthenticationContext authContext = new AuthenticationContext(authority); 
    ClientCredential credential = new ClientCredential(clientID, clientSecret);
    AuthenticationResult authResult = 
        authContext.AcquireTokenAsync("https://management.core.windows.net/", credential).Result;

    return authResult;
}
```

如果您使用個別使用者驗證， `GetAccessToken` 方法看起來就會稍有不同。

> [!IMPORTANT]
> 如果您選擇使用個別使用者驗證，而不是服務主體，只需使用這個程式碼範例。
> 
> 

```csharp
private static AuthenticationResult GetAccessToken()
{
    AuthenticationContext authContext = new AuthenticationContext(authority);
    AuthenticationResult authResult = authContext.AcquireTokenAsync("https://management.core.windows.net/",
        clientID, new Uri("http://<redirect URI>"), new PlatformParameters(PromptBehavior.RefreshSession)).Result;

    return authResult;
}
```

務必使用您在 Azure AD 中註冊應用程式時所輸入的重新導向 URI 來取代 `<redirect URI>` 。

## <a name="list-cdn-profiles-and-endpoints"></a>列出 CDN 設定檔和端點
現在我們已經準備好執行 CDN 作業。  方法的首要任務就是列出資源群組中的所有設定檔和端點，而且如果找到與常數中指定的設定檔和端點名稱相符的項目，請記下該項目以供後續使用，如此一來我們就不用嘗試建立重複的項目。

```csharp
private static void ListProfilesAndEndpoints(CdnManagementClient cdn)
{
    // List all the CDN profiles in this resource group
    var profileList = cdn.Profiles.ListByResourceGroup(resourceGroupName);
    foreach (Profile p in profileList)
    {
        Console.WriteLine("CDN profile {0}", p.Name);
        if (p.Name.Equals(profileName, StringComparison.OrdinalIgnoreCase))
        {
            // Hey, that's the name of the CDN profile we want to create!
            profileAlreadyExists = true;
        }

        //List all the CDN endpoints on this CDN profile
        Console.WriteLine("Endpoints:");
        var endpointList = cdn.Endpoints.ListByProfile(p.Name, resourceGroupName);
        foreach (Endpoint e in endpointList)
        {
            Console.WriteLine("-{0} ({1})", e.Name, e.HostName);
            if (e.Name.Equals(endpointName, StringComparison.OrdinalIgnoreCase))
            {
                // The unique endpoint name already exists.
                endpointAlreadyExists = true;
            }
        }
        Console.WriteLine();
    }
}
```

## <a name="create-cdn-profiles-and-endpoints"></a>建立 CDN 設定檔和端點
接下來，我們將建立設定檔。

```csharp
private static void CreateCdnProfile(CdnManagementClient cdn)
{
    if (profileAlreadyExists)
    {
        Console.WriteLine("Profile {0} already exists.", profileName);
    }
    else
    {
        Console.WriteLine("Creating profile {0}.", profileName);
        ProfileCreateParameters profileParms =
            new ProfileCreateParameters() { Location = resourceLocation, Sku = new Sku(SkuName.StandardVerizon) };
        cdn.Profiles.Create(profileName, profileParms, resourceGroupName);
    }
}
```

建立設定檔之後，我們將建立端點。

```csharp
private static void CreateCdnEndpoint(CdnManagementClient cdn)
{
    if (endpointAlreadyExists)
    {
        Console.WriteLine("Profile {0} already exists.", profileName);
    }
    else
    {
        Console.WriteLine("Creating endpoint {0} on profile {1}.", endpointName, profileName);
        EndpointCreateParameters endpointParms =
            new EndpointCreateParameters()
            {
                Origins = new List<DeepCreatedOrigin>() { new DeepCreatedOrigin("Contoso", "www.contoso.com") },
                IsHttpAllowed = true,
                IsHttpsAllowed = true,
                Location = resourceLocation
            };
        cdn.Endpoints.Create(endpointName, endpointParms, profileName, resourceGroupName);
    }
}
```

> [!NOTE]
> 上述範例會為端點指派一個名為*Contoso* 的來源且主機名稱為 `www.contoso.com`。  您應該變更此項以指向您自己來源的主機名稱。
> 
> 

## <a name="purge-an-endpoint"></a>清除端點
假設端點已建立，我們可能想要在程式中執行的一個常見工作是清除此端點中的內容。

```csharp
private static void PromptPurgeCdnEndpoint(CdnManagementClient cdn)
{
    if (PromptUser(String.Format("Purge CDN endpoint {0}?", endpointName)))
    {
        Console.WriteLine("Purging endpoint. Please wait...");
        cdn.Endpoints.PurgeContent(endpointName, profileName, resourceGroupName, new List<string>() { "/*" });
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}
```

> [!NOTE]
> 在上述範例中，字串 `/*` 代表我想要清除端點路徑根目錄中的所有項目。  這相當於在 Azure 入口網站的 [清除] 對話方塊中勾選 [全部清除]。 在 `CreateCdnProfile` 方法中，我已經使用程式碼 `Sku = new Sku(SkuName.StandardVerizon)` 來建立設定檔做為**來自 Verizon 的 Azure CDN** 設定檔，因此這將會成功。  不過，**來自 Akamai 的 Azure CDN** 設定檔不支援 [全部清除]，因此，如果我在本教學課程中使用 Akamai 設定檔，就必須包含要清除的特定路徑。
> 
> 

## <a name="delete-cdn-profiles-and-endpoints"></a>刪除 CDN 設定檔和端點
最後一個方法會刪除我們的端點和設定檔。

```csharp
private static void PromptDeleteCdnEndpoint(CdnManagementClient cdn)
{
    if(PromptUser(String.Format("Delete CDN endpoint {0} on profile {1}?", endpointName, profileName)))
    {
        Console.WriteLine("Deleting endpoint. Please wait...");
        cdn.Endpoints.DeleteIfExists(endpointName, profileName, resourceGroupName);
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}

private static void PromptDeleteCdnProfile(CdnManagementClient cdn)
{
    if(PromptUser(String.Format("Delete CDN profile {0}?", profileName)))
    {
        Console.WriteLine("Deleting profile. Please wait...");
        cdn.Profiles.DeleteIfExists(profileName, resourceGroupName);
        Console.WriteLine("Done.");
        Console.WriteLine();
    }
}
```

## <a name="running-the-program"></a>執行程式
我們現在可以按一下 Visual Studio 中的 [開始]  按鈕來編譯及執行程式。

![程式正在執行中](./media/cdn-app-dev-net/cdn-program-running-1.png)

當程式達到上述提示時，您應該能夠回到您在 Azure 入口網站中的資源群組，並看見設定檔已建立。

![成功！](./media/cdn-app-dev-net/cdn-success.png)

接著，可確認提示來執行程式的其餘部分。

![正在完成程式](./media/cdn-app-dev-net/cdn-program-running-2.png)

## <a name="next-steps"></a>後續步驟
若要查看此逐步解說中已完成的專案，請 [下載範例](https://code.msdn.microsoft.com/Azure-CDN-Management-1f2fba2c)。

若要尋找適用於 .NET 的 Azure CDN 管理程式庫的其他相關文件，請檢視 [MSDN 上的參考](https://msdn.microsoft.com/library/mt657769.aspx)。

使用 [PowerShell](cdn-manage-powershell.md)管理 CDN 資源。

