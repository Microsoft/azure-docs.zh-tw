---
title: 使用 Visual Studio 開發和部署 WebJob - Azure
description: 了解如何使用 Visual Studio 開發 Azure WebJob，並將其部署至 Azure App Service。
services: app-service
documentationcenter: ''
author: ggailey777
manager: erikre
editor: jimbe
ms.assetid: a3a9d320-1201-4ac8-9398-b4c9535ba755
ms.service: app-service
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: vs-azure
ms.workload: azure-vs
ms.date: 09/12/2017
ms.author: glenga;david.ebbo;suwatch;pbatum;naren.soni
ms.openlocfilehash: 64fdb6dceb1ca10e68411f95c310fdd9a2e25202
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42444126"
---
# <a name="develop-and-deploy-webjobs-using-visual-studio---azure-app-service"></a>使用 Visual Studio 開發和部署 WebJob - Azure App Service

## <a name="overview"></a>概觀

本主題說明如何使用 Visual Studio 將主控台應用程式專案部署為 [App Service](app-service-web-overview.md)中之 Web 應用程式的 [Azure WebJob](http://go.microsoft.com/fwlink/?LinkId=390226)。 如需如何使用 [Azure 入口網站](https://portal.azure.com)部署 WebJob 的相關資訊，請參閱[使用 WebJob 執行背景工作](web-sites-create-web-jobs.md)。

當 Visual Studio 部署具有 WebJobs 功能的主控台應用程式專案時，它會執行兩個工作：

* 將執行階段檔案複製到 Web 應用程式中的適當資料夾 (若是連續 WebJobs，則是 *App_Data/jobs/continuous*，若是排程和隨選 WebJobs，則是 *App_Data/jobs/triggered*)。
* 為已排定在特定時間執行的 WebJobs 設定 [Azure 排程器](https://docs.microsoft.com/azure/scheduler/)工作。 (無需為連續 WebJobs 執行此動作。)

具有 WebJobs 功能的專案會新增下列項目：

* [Microsoft.Web.WebJobs.Publish](http://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) NuGet 封裝。
* 包含部署和排程器設定的 [webjob-publish-settings.json](#publishsettings) 檔案。 

![Diagram showing what is added to a Console App to enable deployment as a WebJob](./media/websites-dotnet-deploy-webjobs/convert.png)

您可以將這些項目新增至現有的主控台應用程式專案，或使用範本建立具有 WebJobs 功能的新主控台應用程式專案。 

以 WebJob 的方式自我部署專案，或將專案連結到 Web 專案，因此每當您要部署 Web 專案時，專案便會自動部署。 若要連結專案，Visual Studio 會在 Web 專案的 [webjobs-list.json](#webjobslist) 檔案中加上具有 WebJobs 功能的專案名稱。

![Diagram showing WebJob project linking to web project](./media/websites-dotnet-deploy-webjobs/link.png)

## <a name="prerequisites"></a>必要條件

如果您是使用 Visual Studio 2015，請安裝 [Azure SDK for .NET (Visual Studio 2015)](https://azure.microsoft.com/downloads/)。

如果您是使用 Visual Studio 2017，請安裝 [Azure 開發工作負載](https://docs.microsoft.com/visualstudio/install/install-visual-studio#step-4---select-workloads)。

## <a id="convert"></a>啟用現有主控台應用程式專案的 WebJobs 部署

您有兩個選擇：

* [透過 Web 專案啟用自動部署](#convertlink)。

  設定現有主控台應用程式專案，以便在您部署 Web 專案時，它會以 WebJob 的方式自動部署。 當您要在與執行相關 Web 應用程式相同的 Web 應用程式中執行 WebJob 時，請使用此選項。

* [不透過 Web 專案啟用部署](#convertnolink)。

  設定現有主控台應用程式專案以 WebJob 的方式自我部署，且不具 Web 專案的連結。 當您要在 Web 應用程式中讓應用程式自行執行 WebJob，且 Web 應用程式中沒有正在執行的 Web 應用程式時，請使用此選項。 為了調整不受 Web 應用程式資源影響的 WebJob 資源，您可能會想執行此動作。

### <a id="convertlink"></a> 透過 Web 專案啟用自動 WebJobs 部署

1. 以滑鼠右鍵按一下 [方案總管] 中的 Web 專案，然後依序按一下 [新增] > [Existing Project as Azure WebJob]。
   
    ![Existing Project as Azure WebJob](./media/websites-dotnet-deploy-webjobs/eawj.png)
   
    [[Add Azure WebJob]](#configure) 對話方塊隨即出現。
2. 在 [專案名稱]  下拉式清單中，選取要新增為 WebJob 的主控台應用程式專案。
   
    ![Selecting project in Add Azure WebJob dialog](./media/websites-dotnet-deploy-webjobs/aaw1.png)
3. 完成 [[Add Azure WebJob]](#configure) 對話方塊，然後按一下 [確定]。 

### <a id="convertnolink"></a> 不透過 Web 專案啟用 WebJobs 部署
1. 以滑鼠右鍵按一下 [方案總管] 中的主控台應用程式專案，然後按一下 [發行為 Azure WebJob]。 
   
    ![發行為 Azure WebJob](./media/websites-dotnet-deploy-webjobs/paw.png)
   
    [[加入 Azure WebJob]](#configure) 對話方塊隨即出現，而且 [專案名稱] 方塊中已選取此專案。
2. 完成 [[加入 Azure WebJob]](#configure) 對話方塊，然後按一下 [確定]。
   
   此時會出現 [發行 Web] 精靈。  如果您不打算立即發行，請關閉精靈。 您所輸入的設定會被儲存下來，以供[部署專案](#deploy)時使用。

## <a id="create"></a>建立具有 WebJobs 功能的新專案
若要建立具有 WebJobs 功能的新專案，您可以使用主控台應用程式專案範本，並如 [上一節](#convert)中所述來啟用 WebJobs 部署。 另一種方式是，您可以使用 WebJobs 新專案範本：

* [在獨立的 WebJob 中使用 WebJobs 新專案範本](#createnolink)
  
    建立專案，並將專案設定為以 WebJob 的方式自我部署，且不具 Web 專案的連結。 當您要在 Web 應用程式中讓應用程式自行執行 WebJob，且 Web 應用程式中沒有正在執行的 Web 應用程式時，請使用此選項。 為了調整不受 Web 應用程式資源影響的 WebJob 資源，您可能會想執行此動作。
* [在連結至 Web 專案的 WebJob 中使用 WebJobs 新專案範本](#createlink)
  
    建立專案，並設定在針對位於相同解決方案中的 Web 專案進行部署時，會自動以 WebJob 的方式部署此專案。 當您要在與執行相關 Web 應用程式相同的 Web 應用程式中執行 WebJob 時，請使用此選項。

> [!NOTE]
> WebJobs 新專案範本自動安裝 NuGet 套件，並包括適用於 *WebJobs SDK* 的 [Program.cs](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs)程式碼。 如果您不想使用 WebJobs SDK，請移除或變更 *Program.cs* 中的 `host.RunAndBlock` 陳述式。
> 
> 

### <a id="createnolink"></a> 在獨立的 WebJob 中使用 WebJobs 新專案範本
1. 依序按一下 [檔案] > [新增專案]，然後在 [新增專案] 對話方塊中，依序按一下 [雲端] > [Azure WebJob (.NET Framework)]。
   
    ![顯示 WebJob 範本的 [新增專案] 對話方塊](./media/websites-dotnet-deploy-webjobs/np.png)
2. 請依照稍早所示的指示，[將主控台應用程式專案設定成獨立的 WebJobs 專案](#convertnolink)。

### <a id="createlink"></a> 在連結至 Web 專案的 WebJob 中使用 WebJobs 新專案範本
1. 以滑鼠右鍵按一下 [方案總管] 中的 Web 專案，然後依序按一下 [新增] > [New Azure WebJob Project]。
   
    ![New Azure WebJob Project menu entry](./media/websites-dotnet-deploy-webjobs/nawj.png)
   
    [[Add Azure WebJob]](#configure) 對話方塊隨即出現。
2. 完成 [[Add Azure WebJob]](#configure) 對話方塊，然後按一下 [確定]。

## <a id="configure"></a>[新增 Azure WebJob] 對話方塊
[加入 Azure WebJob] 對話方塊可讓您輸入 WebJob 名稱和 WebJob 的執行模式設定。 

![[加入 Azure WebJob] 對話方塊](./media/websites-dotnet-deploy-webjobs/aaw2.png)

此對話方塊中的欄位會對應至 Azure 入口網站中 [新增 WebJob] 對話方塊上的欄位。 如需詳細資訊，請參閱[使用 WebJobs 執行背景工作](web-sites-create-web-jobs.md)。

> [!NOTE]
> * 如需命令列部署的詳細資訊，請參閱[啟用 Azure WebJobs 的命令列或連續傳遞](https://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs/)。
> * 如果您部署了 WebJob，但之後決定變更 WebJob 的類型並重新部署，就必須刪除 webjobs-publish-settings.json 檔案。 這樣會讓 Visual Studio 再次顯示發佈選項，您才能夠變更 WebJob 的類型。
> * 如果部署 WebJob，並在稍後將執行模式從連續變更為非連續 (或相反情形)，則在您重新部署時，Visual Studio 會在 Azure 中建立新的 WebJob。 如果您變更其他排程設定但保持執行模式不變，或在排程和隨選模式之間切換，則 Visual Studio 會更新現有的工作，而非建立新的工作。
> 
> 

## <a id="publishsettings"></a>webjob-publish-settings.json
設定 WebJobs 部署的主控台應用程式時，Visual Studio 會安裝 [Microsoft.Web.WebJobs.Publish](http://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish/) NuGet 封裝，並將排程資訊儲存在 WebJobs 專案中專案 *Properties* 資料夾的 *webjob-publish-settings.json* 檔案。 以下是該檔案的範例：

        {
          "$schema": "http://schemastore.org/schemas/json/webjob-publish-settings.json",
          "webJobName": "WebJob1",
          "startTime": "null",
          "endTime": "null",
          "jobRecurrenceFrequency": "null",
          "interval": null,
          "runMode": "Continuous"
        }

您可以編輯此檔案目錄，而 Visual Studio 會提供 IntelliSense。 檔案結構描述會儲存在 [http://schemastore.org](http://schemastore.org/schemas/json/webjob-publish-settings.json)，您可以在該處進行檢視。  

## <a id="webjobslist"></a>webjobs-list.json
當您將具有 WebJobs 功能的專案連結到 Web 專案時，Visual Studio 會將 WebJobs 專案的名稱儲存在 Web 專案中 *Properties* 資料夾的 *webjobs-list.json* 檔案。 此清單可能包含多個 WebJobs 專案，如下列範例所示：

        {
          "$schema": "http://schemastore.org/schemas/json/webjobs-list.json",
          "WebJobs": [
            {
              "filePath": "../ConsoleApplication1/ConsoleApplication1.csproj"
            },
            {
              "filePath": "../WebJob1/WebJob1.csproj"
            }
          ]
        }

您可以編輯此檔案目錄，而 Visual Studio 會提供 IntelliSense。 檔案結構描述會儲存在 [http://schemastore.org](http://schemastore.org/schemas/json/webjobs-list.json)，您可以在該處進行檢視。

## <a id="deploy"></a>部署 WebJobs 專案
已連結到 Web 專案的 WebJobs 專案會透過 Web 專案自動部署。 如需 Web 專案部署的相關資訊，請參閱左邊功能窗格的**使用說明指南** > **部署應用程式**。

若要自我部署 WebJobs 專案，請以滑鼠右鍵按一下 [方案總管] 中的專案，然後按一下 [發行為 Azure WebJob]。 

![發行為 Azure WebJob](./media/websites-dotnet-deploy-webjobs/paw.png)

若是獨立的 WebJob，則 Web 專案所使用的相同 [發行 Web]  精靈隨即出現，但其中幾個設定可以變更。
