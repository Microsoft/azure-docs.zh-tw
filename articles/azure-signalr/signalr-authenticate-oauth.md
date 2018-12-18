---
title: Azure SignalR 服務用戶端驗證教學課程 | Microsoft Docs
description: 在本教學課程中，您將會了解如何驗證 Azure SignalR 服務用戶端
services: signalr
documentationcenter: ''
author: sffamily
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: signalr
ms.workload: tbd
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 06/13/2018
ms.author: zhshang
ms.openlocfilehash: 8751e3485b97b67fd8dd4821480fecd7735c08cd
ms.sourcegitcommit: f58fc4748053a50c34a56314cf99ec56f33fd616
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/04/2018
ms.locfileid: "48268507"
---
# <a name="tutorial-azure-signalr-service-authentication"></a>教學課程：Azure SignalR 服務驗證

本教學課程會以快速入門中所介紹的聊天室應用程式為基礎。 如果您尚未完成[使用 SignalR 服務建立聊天室](signalr-quickstart-dotnet-core.md)，請先完成該項練習。 

在本教學課程中，您將了解如何實作自己的驗證，並讓它與 Microsoft Azure SignalR 服務進行整合。 

對於真實案例來說，快速入門的聊天室應用程式最初所使用的驗證服務過於簡單。 該應用程式可讓每個用戶端宣告其身分，而伺服器就只會接受該項宣告。 這個方法不太適用於真實應用程式，因為惡意使用者會模擬其他人來存取敏感性資料。 

[GitHub](https://github.com/) 會根據常用的業界標準通訊協定 [OAuth](https://oauth.net/) 來提供驗證 API。 這些 API 可讓第三方應用程式驗證 GitHub 帳戶。 在本教學課程中，您會使用這些 API 透過 GitHub 帳戶實作驗證，接著再允許用戶端登入聊天室應用程式。 在驗證 GitHub 帳戶後，系統便會將帳戶資訊新增為 Cookie 以供 Web 用戶端用來驗證。

若要深入了解透過 GitHub 所提供的 OAuth 驗證 API，請參閱[驗證的基本概念](https://developer.github.com/v3/guides/basics-of-authentication/)。

您可以使用任何程式碼編輯器來完成本快速入門中的步驟。 不過，於 Windows、macOS 和 Linux 平台上所提供的 [Visual Studio Code](https://code.visualstudio.com/) \(英文\) 是項不錯的選擇。

本教學課程的程式碼可於 [AzureSignalR-samples GitHub 存放庫](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/GitHubChat)下載。


![OAuth 已在 Azure 中完成裝載](media/signalr-authenticate-oauth/signalr-oauth-complete-azure.png)


在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用您的 GitHub 帳戶註冊新的 OAuth 應用程式
> * 新增驗證控制器以支援 GitHub 驗證
> * 將 ASP.NET Core Web 應用程式部署至 Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須具備下列先決條件：

* 已在 [GitHub](https://github.com/) 建立帳戶
* [Git](https://git-scm.com/)
* [.NET Core SDK](https://www.microsoft.com/net/download/windows) 
* [已設定 Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/quickstart)
* 下載或複製 [AzureSignalR-sample](https://github.com/aspnet/AzureSignalR-samples) GitHub 存放庫。


## <a name="create-an-oauth-app"></a>建立 OAuth 應用程式

1. 開啟網頁瀏覽器並瀏覽至 `https://github.com`，然後登入您的帳戶。

2. 在您的帳戶中，瀏覽至 [設定] > [開發人員設定]，然後按一下 [註冊新的應用程式]，或 [OAuth 應用程式] 底下的 [新增 OAuth 應用程式]。

3. 對新的 OAuth 應用程式使用下列設定，然後按一下 [註冊應用程式]：

    | 設定名稱 | 建議的值 | 說明 |
    | ------------ | --------------- | ----------- |
    | 應用程式名稱 | Azure SignalR Chat | GitHub 使用者應該要能夠辨識並信任他們用來進行驗證的應用程式。   |
    | 首頁 URL | *http://localhost:5000/home* | |
    | 應用程式說明 | 搭配使用 Azure SignalR 服務與 GitHub 驗證的聊天室範例 | 應用程式的實用說明，會協助您的應用程式使用者了解所使用驗證的內容。 |
    | 授權回呼 URL | *http://localhost:5000/signin-github* | 這項設定是 OAuth 應用程式最重要的設定。 GitHub 會在驗證成功後讓使用者返回此回呼 URL。 在本教學課程中，您必須針對 AspNet.Security.OAuth.GitHub 套件 (/signin-github) 使用預設的回呼 URL。  |

4. 新的 OAuth 應用程式註冊完成之後，請使用下列命令在祕密管理員中新增「用戶端識別碼」和「用戶端密碼」。 將 Your_GitHub_Client_Id 和 Your_GitHub_Client_Secret 取代為您的 OAuth 應用程式所具有的值。

        dotnet user-secrets set GitHubClientId Your_GitHub_Client_Id
        dotnet user-secrets set GitHubClientSecret Your_GitHub_Client_Secret


## <a name="implement-the-oauth-flow"></a>實作 OAuth 流程

### <a name="update-the-startup-class-to-support-github-authentication"></a>更新啟動類別以支援 GitHub 驗證

1. 新增最新 Microsoft.AspNetCore.Authentication.Cookies 和 AspNet.Security.OAuth.GitHub 套件的參考，並還原所有套件。

        dotnet add package Microsoft.AspNetCore.Authentication.Cookies -v 2.1.0-rc1-30656
        dotnet add package AspNet.Security.OAuth.GitHub -v 2.0.0-rc2-final
        dotnet restore

1. 開啟 Startup.cs，然後對下列命名空間新增 `using` 陳述式︰

    ```csharp
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Security.Claims;
    using Microsoft.AspNetCore.Authentication.Cookies;
    using Microsoft.AspNetCore.Authentication.OAuth;
    using Newtonsoft.Json.Linq;
    ```

2. 在 `Startup` 類別上方，針對保存 GitHub OAuth 應用程式祕密的祕密管理員金鑰新增常數。

    ```csharp
    private const string GitHubClientId = "GitHubClientId";
    private const string GitHubClientSecret = "GitHubClientSecret";
    ```

3. 對 `ConfigureServices` 方法新增下列程式碼，以支援使用 GitHub OAuth 應用程式進行驗證：

    ```csharp
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
        .AddCookie()
        .AddGitHub(options =>
        {
            options.ClientId = Configuration[GitHubClientId];
            options.ClientSecret = Configuration[GitHubClientSecret];
            options.Scope.Add("user:email");
            options.Events = new OAuthEvents
            {
                OnCreatingTicket = GetUserCompanyInfoAsync
            };
        });
    ```

4. 將 `GetUserCompanyInfoAsync` 協助程式方法新增至 `Startup` 類別。    

    ```csharp
    private static async Task GetUserCompanyInfoAsync(OAuthCreatingTicketContext context)
    {
        var request = new HttpRequestMessage(HttpMethod.Get, context.Options.UserInformationEndpoint);
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", context.AccessToken);

        var response = await context.Backchannel.SendAsync(request,
            HttpCompletionOption.ResponseHeadersRead, context.HttpContext.RequestAborted);

        var user = JObject.Parse(await response.Content.ReadAsStringAsync());
        if (user.ContainsKey("company"))
        {
            var company = user["company"].ToString();
            var companyIdentity = new ClaimsIdentity(new[]
            {
                new Claim("Company", company)
            });
            context.Principal.AddIdentity(companyIdentity);
        }
    }        
    ```

5. 使用下列程式碼更新啟動類別的 `Configure` 方法，然後儲存檔案。

        app.UseAuthentication();



### <a name="add-an-authentication-controller"></a>新增驗證控制器

在本節中，您會實作 `Login` API 以使用 GitHub OAuth 應用程式驗證用戶端。 經過驗證之後，API 會在 Web 用戶端回應中新增 Cookie，然後才將用戶端重新導向回到聊天應用程式。 該 Cookie 之後會用來識別用戶端。

1. 在 chattest\Controllers 目錄中新增控制器程式碼檔案。 將檔案命名為 AuthController.cs。

2. 對驗證控制器新增下列程式碼。 如果您的專案目錄不是 chattest，請務必更新命名空間：

    ```csharp
    using AspNet.Security.OAuth.GitHub;
    using Microsoft.AspNetCore.Authentication;
    using Microsoft.AspNetCore.Mvc;

    namespace chattest.Controllers
    {
        [Route("/")]
        public class AuthController : Controller
        {
            [HttpGet("login")]
            public IActionResult Login()
            {
                if (!User.Identity.IsAuthenticated)
                {
                    return Challenge(GitHubAuthenticationDefaults.AuthenticationScheme);
                }

                HttpContext.Response.Cookies.Append("githubchat_username", User.Identity.Name);
                HttpContext.SignInAsync(User);
                return Redirect("/");
            }
        }
    }    
    ```

3. 儲存您的變更。    

### <a name="update-the-hub-class"></a>更新中樞類別

依預設，當 Web 用戶端嘗試連線到 SignalR 服務時，系統會根據內部所提供的存取權杖來授與連線。 此存取權杖與已驗證的身分識別沒有關聯。 這項存取實際上是匿名存取。 

在本節中，您會開啟實際的驗證，方法是在中樞類別中新增 `Authorize` 屬性，並將中樞方法更新為從已驗證使用者的宣告讀取使用者名稱。

1. 開啟 Hub\Chat.cs，並新增這些命名空間的參考：

    ```csharp
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    ```

2. 更新中樞程式碼，如下所示。 這個程式碼會在 `Chat` 類別中新增 `Authorize` 屬性，並在中樞方法中使用使用者的已驗證身分識別。 此外，還會新增 `OnConnectedAsync` 方法，以在每次有新用戶端連線時，於聊天室中記錄系統訊息。

    ```csharp
    [Authorize]
    public class Chat : Hub
    {
        public override Task OnConnectedAsync()
        {
            return Clients.All.SendAsync("broadcastMessage", "_SYSTEM_", $"{Context.User.Identity.Name} JOINED");
        }

        // Uncomment this line to only allow user in Microsoft to send message
        //[Authorize(Policy = "Microsoft_Only")]
        public void BroadcastMessage(string message)
        {
            Clients.All.SendAsync("broadcastMessage", Context.User.Identity.Name, message);
        }

        public void Echo(string message)
        {
            var echoMessage = $"{message} (echo from server)";
            Clients.Client(Context.ConnectionId).SendAsync("echo", Context.User.Identity.Name, echoMessage);
        }
    }
    ```

3. 儲存您的變更。

### <a name="update-the-web-client-code"></a>更新 Web 用戶端程式碼

1. 開啟 wwwroot\index.html，並將會提示輸入使用者名稱的程式碼，取代為會使用驗證控制器所傳回 Cookie 的程式碼。

    從 index.html 移除下列程式碼：

    ```javascript
    // Get the user name and store it to prepend to messages.
    var username = generateRandomName();
    var promptMessage = 'Enter your name:';
    do {
        username = prompt(promptMessage, username);
        if (!username || username.startsWith('_') || username.indexOf('<') > -1 || username.indexOf('>') > -1) {
            username = '';
            promptMessage = 'Invalid input. Enter your name:';
        }
    } while(!username)
    ```

    新增下列程式碼來取代上面的程式碼，以使用 Cookie：

    ```javascript
    // Get the user name cookie.
    function getCookie(key) {
        var cookies = document.cookie.split(';').map(c => c.trim());
        for (var i = 0; i < cookies.length; i++) {
            if (cookies[i].startsWith(key + '=')) return unescape(cookies[i].slice(key.length + 1));
        }
        return '';
    }
    var username = getCookie('githubchat_username');    
    ```

2. 在為了使用 Cookie 所新增的程式碼正下方，針對 `appendMessage` 函式新增下列定義：

    ```javascript
    function appendMessage(encodedName, encodedMsg) {
        var messageEntry = createMessageEntry(encodedName, encodedMsg);
        var messageBox = document.getElementById('messages');
        messageBox.appendChild(messageEntry);
        messageBox.scrollTop = messageBox.scrollHeight;
    }
    ```

3. 使用下列程式碼將 `bindConnectionMessage` 和 `onConnected` 函式更新為使用 `appendMessage`。

    ```javascript
    function bindConnectionMessage(connection) {
        var messageCallback = function(name, message) {
            if (!message) return;
            // Html encode display name and message.
            var encodedName = name;
            var encodedMsg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
            appendMessage(encodedName, encodedMsg);
        };
        // Create a function that the hub can call to broadcast messages.
        connection.on('broadcastMessage', messageCallback);
        connection.on('echo', messageCallback);
        connection.onclose(onConnectionError);
    }

    function onConnected(connection) {
        console.log('connection started');
        document.getElementById('sendmessage').addEventListener('click', function (event) {
            // Call the broadcastMessage method on the hub.
            if (messageInput.value) {
                connection
                    .invoke('broadcastMessage', messageInput.value)
                    .catch(e => appendMessage('_BROADCAST_', e.message));
            }

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
        document.getElementById('message').addEventListener('keypress', function (event) {
            if (event.keyCode === 13) {
                event.preventDefault();
                document.getElementById('sendmessage').click();
                return false;
            }
        });
        document.getElementById('echo').addEventListener('click', function (event) {
            // Call the echo method on the hub.
            connection.send('echo', messageInput.value);

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
    }    
    ```    

4. 在 index.html 底部，如下所示地更新 `connection.start()` 的錯誤處理常式，以提示使用者登入。

    ```javascript
    connection.start()
        .then(function () {
            onConnected(connection);
        })
        .catch(function (error) {
            if (error) {
                if (error.message) {
                    console.error(error.message);
                }
                if (error.statusCode && error.statusCode === 401) {
                    appendMessage('_BROADCAST_', 'You\'re not logged in. Click <a href="/login">here</a> to login with GitHub.');
                }
            }
        });
    ```

5. 儲存您的變更。    



## <a name="build-and-run-the-app-locally"></a>於本機建置並執行應用程式

1. 儲存所有檔案的變更。 

2. 使用 .NET Core CLI 建置應用程式，在命令殼層中執行下列命令：

        dotnet build

3. 建置成功完成後，請執行下列命令以於本機執行 Web 應用程式：

        dotnet run

    根據預設，應用程式會裝載在本機的連接埠 5000 上：

        E:\Testing\chattest>dotnet run
        Hosting environment: Production
        Content root path: E:\Testing\chattest
        Now listening on: http://localhost:5000
        Application started. Press Ctrl+C to shut down.    

4. 啟動瀏覽器視窗並瀏覽至 `http://localhost:5000`。 按一下頂端的**這裡**連結以使用 GitHub 登入。 

    ![OAuth 已在 Azure 中完成裝載](media/signalr-authenticate-oauth/signalr-oauth-complete-azure.png)

    系統會提示您授權聊天應用程式存取您的 GitHub 帳戶。 按一下 [授權] 按鈕。 
    
    ![授權 OAuth 應用程式](media/signalr-authenticate-oauth/signalr-authorize-oauth-app.png)
    
    系統會將您重新導向回到聊天應用程式，並讓您使用您的 GitHub 帳戶名稱登入。 Web 應用程式會藉由使用您所新增的驗證來對您進行驗證，以決定您的帳戶名稱。

    ![已識別帳戶](media/signalr-authenticate-oauth/signalr-oauth-account-identified.png)

    聊天應用程式現在會使用 GitHub 執行驗證，並將驗證資訊儲存為 Cookie，所以您接著應該將其部署至 Azure，讓其他使用者可以使用他們的帳戶進行驗證，並從其他工作站進行通訊。 


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-the-app-to-azure"></a>將應用程式部署至 Azure

在本節中，您會從 Azure Cloud Shell 使用 Azure 命令列介面 (CLI) 來建立新的 [Azure Web 應用程式](https://docs.microsoft.com/azure/app-service/)，以在 Azure 中裝載您的 ASP.NET 應用程式。 您會將 Web 應用程式設定為使用本機 Git 部署。 您也會使用您的 SignalR 連接字串、GitHub OAuth 應用程式祕密和部署使用者來設定 Web 應用程式。

本節中的步驟會使用適用於 Azure CLI 的 signalr 擴充功能。 執行下列命令以安裝適用於 Azure CLI 的 signalr 擴充功能：

```azurecli-interactive
az extension add -n signalr
```

在建立下列資源時，請務必使用 SignalR 服務資源所在的相同資源群組。 當您之後想要移除所有資源時，這種方法會讓清除作業變得更容易。 所提供的範例假設您使用先前教學課程中所建議的群組名稱 SignalRTestResources。


### <a name="create-the-web-app-and-plan"></a>建立 Web 應用程式和方案

複製下列命令的文字，並更新參數。 將更新後的指令碼貼到 Azure Cloud Shell，然後按 **Enter** 鍵來建立新的 App Service 方案和 Web 應用程式。

```azurecli-interactive
#========================================================================
#=== Update these variable for your resource group name.              ===
#========================================================================
ResourceGroupName=SignalRTestResources

#========================================================================
#=== Update these variable for your web app.                          ===
#========================================================================
WebAppName=myWebAppName
WebAppPlan=myAppServicePlanName

# Create an App Service plan.
az appservice plan create --name $WebAppPlan --resource-group $ResourceGroupName \
    --sku FREE

# Create the new Web App
az webapp create --name $WebAppName --resource-group $ResourceGroupName \
    --plan $WebAppPlan


```


| 參數 | 說明 |
| -------------------- | --------------- |
| resourceGroupName | 這是先前的教學課程中所建議的資源群組名稱。 您最好將所有教學課程的資源群組在一起。 使用您在先前的教學課程中所使用的相同資源群組。 | 
| WebAppPlan | 輸入全新且唯一的 App Service 方案名稱。 | 
| WebAppName | 這會是新 Web 應用程式的名稱，並且是 URL 的一部分。 使用唯一名稱。 例如，signalrtestwebapp22665120。   | 



### <a name="add-app-settings-to-the-web-app"></a>在 Web 應用程式中新增應用程式設定

在本節中，您會針對下列元件新增應用程式設定：

* SignalR 服務資源連接字串
* GitHub OAuth 應用程式用戶端識別碼
* GitHub OAuth 應用程式用戶端密碼

複製下列命令的文字，並更新參數。 將更新後的指令碼貼到 Azure Cloud Shell，然後按 **Enter** 鍵來新增應用程式設定：

```azurecli-interactive
#========================================================================
#=== Update these variables for your GitHub OAuth App.                ===
#========================================================================
GitHubClientId=1234567890
GitHubClientSecret=1234567890

#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
SignalRServiceResource=mySignalRresourcename
WebAppName=myWebAppName

# Get the SignalR Service resource hostName
signalRhostname=$(az signalr show --name $SignalRServiceResource \
    --resource-group $ResourceGroupName --query hostName -o tsv)

# Get the SignalR primary key 
signalRprimarykey=$(az signalr key list --name $SignalRServiceResource \
    --resource-group $ResourceGroupName --query primaryKey -o tsv)

# Form the connection string to the service resource
connstring="Endpoint=https://$signalRhostname;AccessKey=$signalRprimarykey;"

#Add an app setting to the web app for the SignalR connection
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "Azure__SignalR__ConnectionString=$connstring" 

#Add the app settings to use with GitHub authentication
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientId=$GitHubClientId" 
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientSecret=$GitHubClientSecret" 

```

| 參數 | 說明 |
| -------------------- | --------------- |
| GitHubClientId | 對此變數指派您 GitHub OAuth 應用程式的祕密用戶端識別碼。 |
| GitHubClientSecret | 對此變數指派您 GitHub OAuth 應用程式的祕密密碼。 |
| resourceGroupName | 將此變數更新為您在上一節中所使用的相同資源群組名稱。 | 
| SignalRServiceResource | 使用您在快速入門中所建立的 SignalR 服務資源名稱來更新此變數。 例如，signalrtestsvc48778624。 | 
| WebAppName | 使用您在上一節中所建立的新 Web 應用程式名稱來更新此變數。 | 



### <a name="configure-the-web-app-for-local-git-deployment"></a>設定本機 Git 部署的 Web 應用程式

在 Azure Cloud Shell 中貼上下列指令碼。 此指令碼會建立新的部署使用者名稱和密碼，以供您在使用 Git 將程式碼部署至 Web 應用程式時使用。 此指令碼也會使用本機 Git 存放庫來設定部署的 Web 應用程式，並傳回 Git 部署 URL。

```azurecli-interactive
#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
WebAppName=myWebAppName

#========================================================================
#=== Update these variables for your deployment user.                 ===
#========================================================================
DeploymentUserName=myUserName
DeploymentUserPassword=myPassword

# Add the desired deployment user name and password
az webapp deployment user set --user-name $DeploymentUserName \
    --password $DeploymentUserPassword

# Configure Git deployment and note the deployment URL in the output
az webapp deployment source config-local-git --name $WebAppName \
    --resource-group $ResourceGroupName \
    --query [url] -o tsv

```

| 參數 | 說明 |
| -------------------- | --------------- |
| DeploymentUserName | 選擇新的部署使用者名稱。 |
| DeploymentUserPassword | 選擇新的部署使用者密碼。 |
| resourceGroupName | 使用您在上一節中所使用的相同資源群組名稱。 | 
| WebAppName | 這會是您先前所建立的新 Web 應用程式的名稱。 | 


記下從這個命令傳回的 Git 部署 URL。 您稍後會用到這個 URL。


### <a name="deploy-your-code-to-the-azure-web-app"></a>將您的程式碼部署至 Azure Web 應用程式

若要部署您的程式碼，請在 Git 殼層中執行下列命令。

1. 瀏覽至您專案目錄的根目錄。 如果您沒有使用 Git 存放庫所初始化的專案，請執行下列命令：

        git init

2. 針對您稍早所記下的 Git 部署 URL 新增遠端：

        git remote add Azure <your git deployment url>

3. 將所有檔案暫存在已初始化的存放庫中，然後新增認可。

        git add -A
        git commit -m "init commit"

4. 將您的程式碼部署至 Azure 中的 Web 應用程式。        

        git push Azure master

    系統會提示您進行驗證，以將程式碼部署至 Azure。 輸入您在上面針對部署使用者所建立的使用者名稱和密碼。

### <a name="update-the-github-oauth-app"></a>更新 GitHub OAuth 應用程式 

您需要執行的最後一件事是將 GitHub OAuth 應用程式的**首頁 URL** 和**授權回呼 URL** 更新為指向新裝載的應用程式。

1. 在瀏覽器中開啟 [http://github.com](http://github.com)，然後瀏覽至您帳戶的 [設定] > [開發人員設定] > [Oauth應用程式]。

2. 按一下您的驗證應用程式，然後更新**首頁 URL** 和**授權回呼 URL**，如下所示：

    | 設定 | 範例 |
    | ------- | ------- |
    | 首頁 URL | https://signalrtestwebapp22665120.azurewebsites.net/home |
    | 授權回呼 URL | https://signalrtestwebapp22665120.azurewebsites.net/signin-github |


3. 瀏覽至您的 Web 應用程式 URL，並測試應用程式。

    ![OAuth 已在 Azure 中完成裝載](media/signalr-authenticate-oauth/signalr-oauth-complete-azure.png)


## <a name="clean-up-resources"></a>清除資源

如果您準備繼續進行下一個教學課程，則可以保留在本快速入門中所建立的資源，並在下一個教學課程中重複使用它們。

否則，如果您已完成快速入門範例應用程式，便可以將在此快速入門中所建立的 Azure 資源刪除，以避免衍生費用。 

> [!IMPORTANT]
> 刪除資源群組是無法回復的動作，資源群組和其內的所有資源將會永久刪除。 請確定您不會不小心刪除錯誤的資源群組或資源。 如果您是在包含有需要保留之資源的現有資源群組內，建立用來裝載此範例的資源，則可以從每個資源各自的刀鋒視窗中個別刪除每個資源，而不必刪除正個資源群組。
> 
> 

登入 [Azure 入口網站](https://portal.azure.com)，然後按一下 [資源群組]。

在 [依名稱篩選...] 文字方塊中，輸入您的資源群組名稱。 本文的指示是使用名為 SignalRTestResources 的資源群組。 在結果清單中的目標資源群組上方，按一下 **...**，然後按一下 [刪除資源群組]。

   
![刪除](./media/signalr-authenticate-oauth/signalr-delete-resource-group.png)


系統將會要求您確認是否刪除資源。 輸入您的資源群組名稱來確認，然後按一下 [刪除]。
   
片刻過後，系統便會刪除該資源群組及其所有內含的資源。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已新增使用 OAuth 的驗證，以提供更好的方法來使用 Azure SignalR 服務進行驗證。 若要深入了解如何使用 Azure SignalR 伺服器，請繼續進行 SignalR 服務的 Azure CLI 範例。

> [!div class="nextstepaction"]
> [Azure SignalR CLI 範例](./signalr-cli-samples.md)
