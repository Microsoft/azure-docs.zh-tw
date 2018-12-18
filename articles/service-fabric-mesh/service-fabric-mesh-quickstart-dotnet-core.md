---
title: 快速入門 - 建立 Web 應用程式並部署至 Azure Service Fabric Mesh | Microsoft Docs
description: 本快速入門示範如何建立 ASP.NET Core 網站，並將它發行至 Azure Service Fabric Mesh。
services: service-fabric-mesh
documentationcenter: .net
author: tylermsft
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: quickstart
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/17/2018
ms.author: twhitney
ms.custom: mvc, devcenter
ms.openlocfilehash: 7e324b80968017e0160f41b88fa1824669050ac9
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407394"
---
# <a name="quickstart-create-and-deploy-a-web-app-to-azure-service-fabric-mesh"></a>快速入門：建立 Web 應用程式並部署至 Azure Service Fabric Mesh

Azure Service Fabric Mesh 是一個受到完整管理的服務，讓開發人員能夠部署微服務應用程式，而不需管理虛擬機器、儲存體或網路功能。

在本快速入門中，您將建立由 ASP.NET Core Web 應用程式所組成的新 Service Fabric Mesh 應用程式、在本機開發叢集上加以執行，然後加以發行以在 Azure 上執行。

您將需要 Azure 訂用帳戶。 如果您沒有，可輕易地建立一個免費的 Azure 訂用帳戶。開始前請先[建立免費帳戶](https://azure.microsoft.com/free/)。 您也必須[設定您的開發人員環境](service-fabric-mesh-howto-setup-developer-environment-sdk.md)。

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="create-a-service-fabric-mesh-project"></a>建立 Service Fabric Mesh 專案

開啟 Visual Studio，然後選取 [檔案] > [新增] > [專案]。

在 [新增專案] 對話方塊頂端的 [搜尋] 方塊中，輸入 `mesh`。 選取 **Service Fabric Mesh 應用程式**範本。 如果您未看到此範本，請確定已安裝 Mesh SDK 和 VS 工具預覽版，如[設定您的開發環境](service-fabric-mesh-howto-setup-developer-environment-sdk.md)中所說明。 

在 [名稱] 方塊中，輸入 **ServiceFabricMesh1**，然後在 [位置] 方塊中，設定專案檔儲存所在的資料夾路徑。

確定已勾選 [為方案建立目錄]，然後按一下 [確定] 以建立 Service Fabric Mesh 專案。

![Visual Studio 的新增 Service Fabric Mesh 專案對話方塊](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-new-project.png)

### <a name="create-a-service"></a>建立服務

按一下 [確定] 之後，[新增 Service Fabric 服務] 對話方塊隨即出現。 選取 [ASP.NET Core] 專案類型、確定將 [容器 OS] 設定為 [Windows]，然後按一下 [確定] 以建立 ASP.NET Core 專案。 

![Visual Studio 的新增 Service Fabric Mesh 專案對話方塊](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-new-service-fabric-service.png)

[新增 ASP.NET Core Web 應用程式] 對話方塊隨即出現。 選取 [Web 應用程式]，然後按一下 [確定]。

![Visual Studio 的新增 ASP.NET Core 應用程式](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-new-aspnetcore-app.png)

Visual Studio 會建立 Service Fabric Mesh 應用程式專案和 ASP.NET Core 專案。

## <a name="build-and-publish-to-your-local-cluster"></a>建置並發行至您的本機叢集

一旦您的專案載入之後，Docker 映像就會自動建置並發行至您的本機叢集。 此程序可能需要一些時間。 您可以視需要，在 [輸出] 視窗下拉式清單中選取 [Service Fabric 工具] 項目，藉以在 [輸出] 視窗中監視 Service Fabric 工具的進度。 您可以在部署 Dcker 映像時繼續工作。

建立專案之後，按一下 **F5**，對您的服務進行本機偵錯。 完成本機部署且 Visual Studio 正在執行您的專案時，瀏覽器視窗將會開啟並提供一個範例網頁。

當您完成瀏覽已部署的服務之後，在 Visual Studio 中按 **Shift+F5** 以停止對您的專案進行偵錯。

## <a name="publish-to-azure"></a>發佈至 Azure

若要將您的 Service Fabric Mesh 專案發行至 Azure，請在 Visual Studio 中以滑鼠右鍵按一下 [Service Fabric Mesh 專案]，然後選取 [發行]。

![Visual Studio 中以滑鼠右鍵按一下 Service Fabric Mesh 專案](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-right-click-publish.png)

您將會看見 [發行 Service Fabric 應用程式] 對話方塊。

![Visual Studio 的 Service Fabric Mesh 發行對話方塊](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-publish-dialog.png)

選取您的 Azure 帳戶和訂用帳戶。 選擇 [位置] 。 本文使用**美國東部**。

在 [資源群組] 下方，選取 [\<建立新的資源群組...>]。 [建立資源群組] 對話方塊隨即出現。 設定**資源群組名稱**和**位置**。  本快速入門使用**美國東部**的位置，並將群組命名為 **sfmeshTutorial1RG** (如果您的組織有多位人員使用同一個訂用帳戶，請選擇唯一的資源群組名稱)。  按一下 [建立] 以建立資源群組並返回發行對話方塊。

![Visual Studio 的 Service Fabric Mesh 新增資源群組對話方塊](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-publish-new-resource-group-dialog.png)

返回 [發佈 Service Fabric 應用程式] 對話方塊，在 [Azure Container Registry] 下方，選取 [\<建立新的容器登錄...>]。 在 [建立容器登錄] 對話方塊中，請針對**容器登錄名稱**使用唯一的名稱。 指定**位置** (本快速入門使用**美國東部**)。 從下拉式清單中選取您在上一個步驟中建立的**資源群組**，例如 **sfmeshTutorial1RG**。 將 **SKU** 設定為 [基本]，然後按一下 [建立] 以返回發行對話方塊。

![Visual Studio 的 Service Fabric Mesh 新增資源群組對話方塊](media/service-fabric-mesh-quickstart-dotnet-core/visual-studio-publish-new-container-registry-dialog.png)

在發行對話方塊中，按一下 [發行] 按鈕，將您的 Service Fabric Mesh 應用程式部署到 Azure。

當您首次發行到 Azure 時，會將 Dcker 映像推送到 Azure Container Registry (ACR)，所需時間取決於映像大小。 後續發行相同專案的速度將會加快。 您可以在 Visual Studio 的 [輸出] 視窗下拉式清單中選取 [Service Fabric 工具]，藉以監視部署的進度。 一旦部署完成之後，**Service Fabric 工具**輸出將會以 URL 形式來顯示應用程式的 IP 位址和連接埠。

```json
Packaging Application...
Building Images...
Web1 -> C:\Code\ServiceFabricMesh1\Web1\bin\Any CPU\Release\netcoreapp2.0\Web1.dll
Uploading the images to Azure Container Registry...
Deploying application to remote endpoint...
The application was deployed successfully and it can be accessed at http://...
```

開啟網頁瀏覽器並瀏覽至 URL，以查看在 Azure 中執行的網站：

![執行 Service Fabric Mesh Web 應用程式](media/service-fabric-mesh-tutorial-deploy-dotnetcore/deployed-web-project.png)

## <a name="clean-up-resources"></a>清除資源

不再需要時，請刪除您針對本快速入門建立的所有資源。 由於您已建立新的資源群組來裝載 ACR 和 Service Fabric Mesh 服務資源，所以可以安全地刪除此資源群組，這是刪除所有與其相關聯資源的簡單方法。

```azurecli
az group delete --resource-group sfmeshTutorial1RG
```

```powershell
Connect-AzureRmAccount
Remove-AzureRmResourceGroup -Name sfmeshTutorial1RG
```

或者，您可以[從 Azure 入口網站](https://portal.azure.com)來刪除資源群組。

## <a name="next-steps"></a>後續步驟

若要深入了解如何建立和部署 Service Fabric Mesh 應用程式，請繼續進行本教學課程。
> [!div class="nextstepaction"]
> [建立、偵錯並將多服務 Web 應用程式部署至 Service Fabric Mesh](service-fabric-mesh-tutorial-create-dotnetcore.md)