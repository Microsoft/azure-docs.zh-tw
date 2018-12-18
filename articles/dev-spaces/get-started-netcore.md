---
title: 使用 .NET Core 和 VS Code 在雲端建立 Kubernetes 開發人員空間 | Microsoft Docs
titleSuffix: Azure Dev Spaces
services: azure-dev-spaces
ms.service: azure-dev-spaces
ms.component: azds-kubernetes
author: ghogen
ms.author: ghogen
ms.date: 09/26/2018
ms.topic: tutorial
description: 在 Azure 上使用容器和微服務快速進行 Kubernetes 開發
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, 容器
manager: douge
ms.openlocfilehash: f740d1b95b2150d8d814531c88f5578b543b2922
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47585657"
---
# <a name="get-started-on-azure-dev-spaces-with-net-core"></a>在使用 .NET Core 的 Azure 開發人員空間上開始使用

在本指南中，您將了解如何：

- 在 Azure 中建立最適合用於開發的 Kubernetes 型環境 - _開發人員空間_。
- 使用 VS Code 和命令列在容器中反覆開發程式碼。
- 在小組環境中有效率地開發及測試您的程式碼。

> [!Note]
> **如果作業出現停滯的情況**，請參閱[疑難排解](troubleshooting.md)一節，或在此頁面上張貼留言。

您現在可以在 Azure 中建立以 Kubernetes 為基礎的開發人員空間。

## <a name="install-the-azure-cli"></a>安裝 Azure CLI
Azure 開發人員空間需要基本的本機電腦設定。 大部分開發人員空間的組態都會儲存在雲端，而且可與其他使用者共用。 本機電腦可以執行 Windows、Mac 或 Linux。 若為 Linux，支援的散發套件如下：Ubuntu (18.04、16.04 和 14.04)、Debian 8 和 9、RHEL 7、Fedora 26+、CentOS 7、openSUSE 42.2 和 SLES 12。

從下載和執行 [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) 著手。 

> [!IMPORTANT]
> 如果您已安裝 Azure CLI，請確定您使用的是 2.0.43 版或更新版本。

### <a name="sign-in-to-azure-cli"></a>登入 Azure CLI
登入 Azure。 在終端機視窗中輸入下列命令：

```cmd
az login
```

> [!Note]
> 如果您沒有 Azure 訂用帳戶，您可以建立[免費帳戶](https://azure.microsoft.com/free)。

#### <a name="if-you-have-multiple-azure-subscriptions"></a>如果您有多個 Azure 訂用帳戶...
您可以執行下列命令以檢視訂用帳戶： 

```cmd
az account list
```
在 JSON 輸出中找出含有 `isDefault: true` 的訂用帳戶。
如果這不是您要使用的訂用帳戶，您可以變更預設訂用帳戶：

```cmd
az account set --subscription <subscription ID>
```

## <a name="create-a-kubernetes-cluster-enabled-for-azure-dev-spaces"></a>建立已針對 Azure Dev Spaces 啟用的 Kubernetes 叢集

在命令提示字元中，建立資源群組。 使用目前支援的其中一個區域 (EastUS、CentralUS、WestUS2、WestEurope、CanadaCentral 或 CanadaEast)。

```cmd
az group create --name MyResourceGroup --location <region>
```

使用下列命令來建立 Kubernetes 叢集：

```cmd
az aks create -g MyResourceGroup -n MyAKS --location <region> --kubernetes-version 1.11.2 --enable-addons http_application_routing
```

建立叢集需要幾分鐘的時間。

### <a name="configure-your-aks-cluster-to-use-azure-dev-spaces"></a>設定您的 AKS 叢集以使用 Azure Dev Spaces

使用其中包含您 AKS 叢集和 AKS 叢集名稱的資源群組，輸入下列 Azure CLI 命令。 命令會使用 Azure Dev Spaces 的支援來設定您的叢集。

   ```cmd
   az aks use-dev-spaces -g MyResourceGroup -n MyAKS
   ```

## <a name="get-kubernetes-debugging-for-vs-code"></a>取得適用於 VS Code 的 Kubernetes 偵錯功能
使用 VS Code 的 .NET Core 和 Node.js 開發人員有豐富的功能可供使用，例如 Kubernetes 偵錯。

1. 如果您沒有 [VS Code](https://code.visualstudio.com/Download)，請加以安裝。
1. 下載及更新 [VS Azure Dev Spaces 擴充功能](https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds)。 在擴充功能的 Marketplace 頁面上按一下 [安裝]，然後在 VS Code 再按一次。 

## <a name="create-a-web-app-running-in-a-container"></a>建立在容器中執行的 Web 應用程式

在本節中，您會建立 ASP.NET Core Web 應用程式，並使其在 Kubernetes 的容器中執行。

### <a name="create-an-aspnet-core-web-app"></a>建立 ASP.NET Core Web 應用程式
如果您已安裝 [.NET Core](https://www.microsoft.com/net)，您可以在名為 `webfrontend` 的資料夾中快速建立 ASP.NET Core Web 應用程式。
    
```cmd
dotnet new mvc --name webfrontend
```

或者，瀏覽至 https://github.com/Azure/dev-spaces 以**從 GitHub 下載範例程式碼**，然後選取 [複製或下載]，將 GitHub 存放庫下載到您的本機環境。 本指南的程式碼位於 `samples/dotnetcore/getting-started/webfrontend`。

## <a name="preparing-code-for-docker-and-kubernetes-development"></a>準備程式碼以進行 Docker 和 Kubernetes 開發
目前為止，您具有可以在本機執行的基本 Web 應用程式。 您現在會藉由建立資產 (定義應用程式的容器) 並將其部署到 Kubernetes 的方式，將應用程式容器化。 使用 Azure Dev Spaces 可以輕鬆完成此工作： 

1. 啟動 VS Code 並開啟 `webfrontend` 資料夾。 (您可以略過要新增偵錯資產或還原專案的任何預設提示。)
1. 在 VS Code 中開啟整合式終端機 (使用 [檢視 > 整合式終端機] 功能表)。
1. 執行這個命令 (請確定您目前的資料夾是 **webfrontend**)：

    ```cmd
    azds prep --public
    ```

Azure CLI 的 `azds prep` 命令會產生具有預設設定的 Docker 和 Kubernetes 資產：
* `./Dockerfile` 描述了應用程式的容器映像，以及如何建置原始程式碼和在容器內執行。
* `./charts/webfrontend` 底下的 [Helm 圖表](https://docs.helm.sh)會說明如何將容器部署至 Kubernetes。

現在不需要了解這些檔案的完整內容。 但是依然很值得指出，**從開發環境到生產環境都可以使用相同的 Kubernetes 和 Docker 組態即程式碼資產，而為不同的環境之間提供更佳的一致性。**
 
`prep` 命令也會產生名為 `./azds.yaml` 的檔案，該檔案是 Azure Dev Spaces 的組態檔。 它可以透過在 Azure 中實現反覆式開發法體驗的其他組態，來補足 Docker 和 Kubernetes 成品。

## <a name="build-and-run-code-in-kubernetes"></a>在 Kubernetes 中建置及執行程式碼
讓我們來執行程式碼！ 在終端機視窗中，從**根程式碼資料夾**中執行 webfrontend 這個命令：

```cmd
azds up
```

請留意命令的輸出，您會在進行時注意到幾件事：
- 原始程式碼會同步至 Azure 中的開發人員空間。
- 容器映像內建在 Azure 中，如同程式碼資料夾中的 Docker 資產所指定。
- 系統會建立 Kubernetes 物件，它會利用程式碼資料夾中 Helm 圖表所指定的容器映像。
- 容器端點的相關資訊隨即顯示。 在我們的案例中，我們預期是公用 HTTP URL。
- 假設上述階段成功完成，您應該會在容器啟動時開始看到 `stdout` (和 `stderr`) 輸出。

> [!Note]
> 這些步驟在 `up` 命令第一次執行時會花費較長的時間，但是後續執行應該會更快。

### <a name="test-the-web-app"></a>測試 Web 應用程式
請掃描主控台輸出以取得 `up` 命令所建立的公用 URL 相關資訊。 它會在表單中： 

```
(pending registration) Service 'webfrontend' port 'http' will be available at <url>
Service 'webfrontend' port 80 (TCP) is available at http://localhost:<port>
```

在瀏覽器視窗中開啟此 URL，您應該會看到 Web 應用程式載入。 當容器執行時，`stdout` 和 `stderr` 輸出會串流至終端機視窗。

> [!Note]
> 第一次執行時，可能需要數分鐘的時間才能備妥公用 DNS。 如果無法解析公用 URL，您可以使用主控台輸出中顯示的替代 http://localhost:<portnumber> URL。 如果您使用 localhost URL，容器可能看起來像在本機執行，但實際是在 AKS 中執行。 為了方便您操作以及與本機電腦上的服務互動，Azure 開發人員空間會建立暫存的 SSH 通道，連到在 Azure 中執行的容器。 當 DNS 記錄備妥時，您可以返回且稍後嘗試公用 URL。

### <a name="update-a-content-file"></a>更新內容檔案
Azure 開發人員空間不只讓程式碼中在 Kubernetes 中執行 - 還可讓您快速地反覆查看您的程式碼變更是否在雲端 Kubernetes 環境中生效。

1. 找出檔案 `./Views/Home/Index.cshtml` 並進行 HTML 編輯。 例如，將第 70 行 `<h2>Application uses</h2>` 變更如下：`<h2>Hello k8s in Azure!</h2>`
1. 儲存檔案。 稍後，您會在終端機視窗中看到一則訊息，指出執行中容器中的檔案已更新。
1. 移至您的瀏覽器並重新整理頁面。 您應該會看到網頁顯示更新後的 HTML。

發生什麼情形？ 編輯內容檔案 (例如 HTML 和 CSS) 時，不需要在 .NET Core Web 應用程式中重新編譯，所以作用中 `azds up` 命令會自動將任何修改過的內容檔案，直接同步處理到 Azure 中的執行中容器，您即可立即查看內容編輯。

### <a name="update-a-code-file"></a>更新程式碼檔案
更新程式碼檔案需要更多的工作，因為.NET Core 應用程式需要重建及產生更新後的應用程式二進位檔。

1. 在終端機視窗中，按 `Ctrl+C` (以停止 `azds up`)。
1. 開啟名為 `Controllers/HomeController.cs` 的程式碼檔案，然後編輯 [關於] 頁面將顯示的訊息：`ViewData["Message"] = "Your application description page.";`
1. 儲存檔案。
1. 在終端機視窗中執行 `azds up`。 

此命令會重建容器映像，並重新部署 Helm 圖表。 若要查看程式碼變更是否在執行中應用程式中生效，請移至 Web 應用程式中的 [關於] 功能表。


但是還有「更加快速的方法」可開發程式碼，您將在下一節中加以探索。 

## <a name="debug-a-container-in-kubernetes"></a>在 Kubernetes 中進行容器偵錯

在本節中，您將使用 VS Code 直接對我們在 Azure 中執行的容器進行偵錯。 您也將了解如何取得更快速的編輯-執行-測試迴圈。

![](media/common/edit-refresh-see.png)

> [!Note]
> **如果作業出現停滯的情況**，請參閱[疑難排解](troubleshooting.md)一節，或在此頁面上張貼留言。

### <a name="initialize-debug-assets-with-the-vs-code-extension"></a>使用 VS Code 擴充功能初始化偵錯資產
首先您必須設定您的程式碼專案，讓 VS Code 與我們在 Azure 中的開發人員空間通訊。 適用於 Azure Dev Spaces 的 VS Code 擴充功能提供協助程式命令，可以設定偵錯組態。 

開啟 [命令選擇區] (使用 [檢視 | 命令選擇區] 功能表)，然後使用自動完成功能以輸入及選取以下命令：`Azure Dev Spaces: Prepare configuration files for Azure Dev Spaces`。 

這樣會為 `.vscode` 資料夾底下的 Azure Dev Spaces 新增偵錯組態。 此命令不應與 `azds prep` 命令混淆；後者會設定要部署的專案。

![](media/common/command-palette.png)


### <a name="select-the-azds-debug-configuration"></a>選取 AZDS 偵錯組態
1. 若要開啟 [偵錯] 檢視，請按一下 VS Code 側邊的 [活動列] 中的 [偵錯] 圖示。
1. 選取 [.NET Core Launch (AZDS)] 作為作用中偵錯組態。

![](media/get-started-netcore/debug-configuration.png)

> [!Note]
> 如果您未在 [命令選擇區] 中看到任何 Azure 開發人員空間命令，請確定您已安裝適用於 Azure 開發人員空間的 VS Code 擴充功能。 請確定您在 VS Code 中開啟的工作區是包含 azds.yaml 的資料夾。


### <a name="debug-the-container-in-kubernetes"></a>在 Kubernetes 中進行容器偵錯
按 **F5** 可在 Kubernetes 中進行程式碼偵錯。

與 `up` 命令相同，程式碼也會同步到開發人員空間，且容器會建置並部署到 Kubernetes。 此時，偵錯工具當然會連結至遠端容器。

> [!Tip]
> VS Code 狀態列會顯示可點按的 URL。

![](media/common/vscode-status-bar-url.png)

在伺服器端程式碼檔案中設定中斷點，例如在 `Controllers/HomeController.cs` 來源檔案的 `Index()` 函式內。 重新整理瀏覽器頁面會導致叫用中斷點。

就如同已在本機執行程式碼一樣，您擁有偵錯資訊的完整存取權，例如呼叫堆疊、區域變數、例外狀況資訊等等。

### <a name="edit-code-and-refresh"></a>編輯程式碼並重新整理
使用有效的偵錯工具，進行程式碼編輯。 例如，在 `Controllers/HomeController.cs` 中修改 [關於] 頁面的訊息。 

```csharp
public IActionResult About()
{
    ViewData["Message"] = "My custom message in the About page.";
    return View();
}
```

儲存檔案，然後在 [偵錯動作] 窗格中，按一下 [重新整理] 按鈕。 

![](media/get-started-netcore/debug-action-refresh.png)

Azure 開發人員空間會以累加方式重新編譯現有容器中的程式碼，以提供更快的編輯/偵錯迴圈，而不是在每次進行程式碼編輯時重新建置及重新部署新的容器映像 (這通常要花費相當長的時間)。

請在瀏覽器中重新整理 Web 應用程式，並移至 [關於] 頁面。 您應會看到自訂訊息出現在 UI 中。

**您現在有辦法在 Kubernetes 中快速逐一查看程式碼及直接進行偵錯！** 接下來，您將了解如何建立和呼叫第二個容器。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解小組開發](team-development-netcore.md)
