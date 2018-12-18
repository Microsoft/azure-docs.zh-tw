---
title: 在 Azure 中的 Service Fabric 上監視和診斷 ASP.NET Core 服務 | Microsoft Docs
description: 在本教學課程中，您會了解如何設定 Azure Service Fabric ASP.NET Core 應用程式的監視和診斷。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/14/2017
ms.author: dekapur
ms.custom: mvc
ms.openlocfilehash: b592ac10d483bbf0ca058e87f819b8f67847c54e
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44296103"
---
# <a name="tutorial-monitor-and-diagnose-an-aspnet-core-application-on-service-fabric-using-application-insights"></a>教學課程：使用 Application Insights 監視和診斷 Service Fabric 上的 ASP.NET Core 應用程式

本教學課程是一個系列的第五部分。 其中會逐步設定使用 Application Insights 來監視和診斷 Service Fabric 叢集上執行的 ASP.NET Core 應用程式。 我們將從教學課程第一部分[建置 .NET Service Fabric 應用程式](service-fabric-tutorial-create-dotnet-app.md)中所開發的應用程式收集遙測資料。

在教學課程系列的第四部分，您將了解如何：
> [!div class="checklist"]
> * 設定應用程式的 Application Insights
> * 收集回應遙測資料以追蹤服務之間以 HTTP 為基礎的通訊
> * 使用 Application Insights 中的應用程式對應功能
> * 使用 Application Insights API 新增自訂事件

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [建置 .NET Service Fabric 應用程式](service-fabric-tutorial-create-dotnet-app.md)
> * [將應用程式部署到遠端叢集](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [將 HTTPS 端點新增至 ASP.NET Core 前端服務](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
> * [使用 Azure Pipelines 設定 CI/CD](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * 設定應用程式的監視和診斷

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [安裝 Visual Studio 2017](https://www.visualstudio.com/) 並安裝 **Azure 開發**以及 **ASP.NET 和 Web 開發**工作負載。
* [安裝 Service Fabric SDK](service-fabric-get-started.md)

## <a name="download-the-voting-sample-application"></a>下載投票應用程式範例

如果您未在[本教學課程系列的第一部分](service-fabric-tutorial-create-dotnet-app.md)中建置投票應用程式範例，可以下載它。 在命令視窗或終端機中，執行下列命令，將範例應用程式存放庫複製到本機電腦。

```git
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart
```

## <a name="set-up-an-application-insights-resource"></a>設定 Application Insights 資源

Application Insights 是 Azure 的應用程式效能管理平台，也是 Service Fabric 建議的應用程式監視和診斷平台。 若要建立 Application Insights 資源，請瀏覽至 [Azure 入口網站](https://portal.azure.com)。 在左導覽功能表上按一下 [建立資源]，以開啟 Azure Marketplace。 按一下 [監視 + 管理]，然後按一下 [Application Insights]。

![建立新的 AI 資源](./media/service-fabric-tutorial-monitoring-aspnet/new-ai-resource.png)

現在，針對要建立之資源的屬性，您需要填寫必要資訊。 輸入適當的 [名稱]、[資源群組] 和 [訂用帳戶]。 設定未來部署 Service Fabric 叢集的目標 [位置]。 在本教學課程中，我們會將應用程式部署到本機叢集，因此 [位置] 欄位無關緊要。 [應用程式類型] 應該保留為「ASP.NET Web 應用程式」。

![AI 資源屬性](./media/service-fabric-tutorial-monitoring-aspnet/new-ai-resource-attrib.png)

填寫必要資訊之後，請按一下 [建立] 來佈建資源 - 需要大約一分鐘。
<!-- When completed, navigate to the newly deployed resource, and find the "Instrumentation Key" (visible in the "Essentials" drop down section). Copy it to clipboard, since we will need it in the next step. -->

## <a name="add-application-insights-to-the-applications-services"></a>將 Application Insights 新增至應用程式的服務

使用較高的權限啟動 Visual Studio 2017。 作法是以滑鼠右鍵按一下 [開始] 功能表中的 Visual Studio 圖示，然後選擇 [以系統管理員身分執行]。 按一下 [檔案] > **[開啟]** > **[專案/方案]**，並瀏覽至投票應用程式 (在教學課程的第一部分中建立，或透過 git 複製)。 開啟 *Voting.sln*，如果系統提示您還原應用程式的 NuGet 套件，請按一下 [是]。

請遵循下列步驟來設定 VotingWeb 和 VotingData 服務的 Application Insights：

1. 以滑鼠右鍵按一下服務名稱，然後按一下 [設定 Application Insights...]。

    ![設定 AI](./media/service-fabric-tutorial-monitoring-aspnet/configure-ai.png)

2. 按一下 [開始免費試用]。
3. 登入您的帳戶 (這也用來設定您的 Azure 訂用帳戶)，並選取您在其中建立 Application Insights 資源的訂用帳戶。 從 [資源] 下拉式清單中，在 [現有的 Application Insights 資源] 下尋找資源。 按一下 [註冊] 將 Application Insights 新增至您的服務。

    ![註冊 AI](./media/service-fabric-tutorial-monitoring-aspnet/register-ai.png)

4. 當快顯對話方塊完成動作後，按一下 [完成]。

務必對應用程式中的**兩個**服務都執行上述步驟，以完成設定應用程式的 Application Insights。 兩個服務都使用相同的 Application Insights 資源，以便查看服務之間的傳入和傳出要求和通訊。

## <a name="add-the-microsoftapplicationinsightsservicefabricnative-nuget-to-the-services"></a>將 Microsoft.ApplicationInsights.ServiceFabric.Native NuGet 新增至服務

Application Insights 有兩個 Service Fabric 特定的 NuGet，可依情節來使用。 其中一個用於 Service Fabric 的原生服務，另一個用於容器和客體可執行檔。 在此案例中，我們將使用 Microsoft.ApplicationInsights.ServiceFabric.Native NuGet 以深入了解它所提供的服務內容。 若要深入了解 Application Insights SDK 及 Service Fabric 特定的 NuGet，請參閱 [Microsoft Application Insights for Service Fabric](https://github.com/Microsoft/ApplicationInsights-ServiceFabric/blob/master/README.md)。

設定 NuGet 的步驟如下：

1. 以滑鼠右鍵按一下 [方案總管] 頂端的 [方案 'Voting']，然後按一下 [管理方案的 NuGet 套件...]。
2. 在 [NuGet - 方案] 視窗的頂端導覽功能表上，按一下 [瀏覽]，並勾選搜尋列旁邊的 [包含發行前版本] 方塊。
3. 搜尋 `Microsoft.ApplicationInsights.ServiceFabric.Native`，然後按一下適當的 NuGet 套件。

>[!NOTE]
>如果在安裝 Application Insights 套件之前未預先安裝 Microsoft.ServiceFabric.Diagnostics.Internal 套件，您可能需要以類似方式安裝該套件

4. 在右側，按一下應用程式中兩個服務 (**VotingWeb** 和 **VotingData**) 旁邊的兩個核取方塊，然後按一下 [安裝]。
    ![AI sdk Nuget](./media/service-fabric-tutorial-monitoring-aspnet/ai-sdk-nuget-new.png)
5. 按一下 [檢閱變更] 快顯對話方塊上的 [確定]，並選擇 [接受授權]。 這會完成將 NuGet 新增至服務。
6. 您現在需要在兩個服務中設定遙測初始設定式。 請開啟 *VotingWeb.cs* 和 *VotingData.cs*。 對這兩者執行下列兩個步驟：
    1. 在每個 *\<ServiceName>.cs* 的頂端，新增下列兩個 *using* 陳述式：

    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.ServiceFabric;
    ```

    2. 在 *CreateServiceInstanceListeners()* 或 *CreateServiceReplicaListeners()* 的巢狀 *return* 陳述式中，在 *ConfigureServices* > *services* 之下宣告的兩個 Singleton 服務之間，新增：`.AddSingleton<ITelemetryInitializer>((serviceProvider) => FabricTelemetryInitializerExtension.CreateFabricTelemetryInitializer(serviceContext))`
    這會將「服務內容」新增至您的遙測資料，讓您更充分了解 Application Insights 中的遙測資料來源。 *VotingWeb.cs*中的巢狀 *return* 陳述式看起來應該像這樣：

    ```csharp
    return new WebHostBuilder()
        .UseKestrel()
        .ConfigureServices(
            services => services
                .AddSingleton<HttpClient>(new HttpClient())
                .AddSingleton<FabricClient>(new FabricClient())
                .AddSingleton<StatelessServiceContext>(serviceContext)
                .AddSingleton<ITelemetryInitializer>((serviceProvider) => FabricTelemetryInitializerExtension.CreateFabricTelemetryInitializer(serviceContext)))
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseStartup<Startup>()
        .UseApplicationInsights()
        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
        .UseUrls(url)
        .Build();
    ```

    同樣地，在 *VotingData.cs* 中，您應該會有：

    ```csharp
    return new WebHostBuilder()
        .UseKestrel()
        .ConfigureServices(
            services => services
                .AddSingleton<StatefulServiceContext>(serviceContext)
                .AddSingleton<IReliableStateManager>(this.StateManager)
                .AddSingleton<ITelemetryInitializer>((serviceProvider) => FabricTelemetryInitializerExtension.CreateFabricTelemetryInitializer(serviceContext)))
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseStartup<Startup>()
        .UseApplicationInsights()
        .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
        .UseUrls(url)
        .Build();
    ```

再次確認已在這兩個檔案中呼叫 `UseApplicationInsights()` 方法，如前所述。

>[!NOTE]
>此範例應用程式會使用 HTTP 進行服務的通訊。 如果您使用服務遠端處理第 2 版開發應用程式，則您也必須依照前述作法，在相同的位置加入以下幾行程式碼

```csharp
ConfigureServices(services => services
    ...
    .AddSingleton<ITelemetryModule>(new ServiceRemotingDependencyTrackingTelemetryModule())
    .AddSingleton<ITelemetryModule>(new ServiceRemotingRequestTrackingTelemetryModule())
)
```

此時，您已經準備好要部署應用程式。 按一下頂端的 [開始] \(或 **F5**)，Visual Studio 會建置並封裝應用程式、設定本機叢集，然後將應用程式部署到此叢集。

完成應用程式部署後，請移至 [localhost:8080](localhost:8080)，您應該可以看到「投票範例」單一頁面應用程式。 票選您喜歡的幾個不同項目，以建立一些樣本資料和遙測資料 - 我去吃甜點了！

![AI 範例投票](./media/service-fabric-tutorial-monitoring-aspnet/vote-sample.png)

投下幾票之後，也請隨意「移除」一些投票選項。

## <a name="view-telemetry-and-the-app-map-in-application-insights"></a>在 Application Insights 中檢視遙測資料和應用程式對應

在 Azure 入口網站中瀏覽至您的 Application Insights 資源。

按一下 [概觀]，以回到資源的登陸頁面。 然後按一下頂端的 [搜尋]，可看到追蹤開始湧入。 經過幾分鐘，追蹤就會出現在 Application Insights 中。 如果沒有看到任何追蹤，請稍候片刻，然後按一下頂端的 [重新整理] 按鈕。
![AI 查看追蹤](./media/service-fabric-tutorial-monitoring-aspnet/ai-search.png)

在 [搜尋] 視窗中向下捲動，就會看到 Application Insights 傳入的所有立即可用的遙測資料。 針對您在投票應用程式中執行的每個動作，應該會有一個從 *VotingWeb* 傳出的 PUT 要求 (PUT Votes/Put [name])、一個從 *VotingData* 傳入的 PUT 要求 (PUT VoteData/Put [name])，後面接著一對 GET 要求來重新整理所顯示的資料。 因為這些是 HTTP 要求，所以 localhost 上也會有 HTTP 的相依性追蹤。 以下是如何新增一張選票的範例畫面：![AI 範例要求追蹤](./media/service-fabric-tutorial-monitoring-aspnet/sample-request.png)

您可以按一下其中一個追蹤，以檢視更多詳細資料。 關於 Application Insights 提供的要求，有一些很有用的資訊，包括「回應時間」和「要求 URL」。 此外，因為您已新增 Service Fabric 特定的 NuGet，在下方的「自訂資料」區段中，您也會看到應用程式在 Service Fabric 叢集環境中的相關資料。 這包括服務內容，可讓您查看要求來源的 *PartitionID* 和 *ReplicaId*，在診斷應用程式的錯誤時更準確查明問題所在。

![AI 追蹤詳細資料](./media/service-fabric-tutorial-monitoring-aspnet/trace-details.png)

此外，您可以在 [概觀] 頁面的左側功能表上按一下 [應用程式對應]，或按一下 [應用程式對應] 圖示，以導向至會顯示兩個連線服務的 [應用程式對應]。

![AI 追蹤詳細資料](./media/service-fabric-tutorial-monitoring-aspnet/app-map-new.png)

「應用程式對應」可協助您更充分了解應用程式拓撲，特別是當您開始新增多個一起運作的不同服務時。 另外也提供要求成功率的基本資料，還可協助您診斷失敗的要求，以查明可能出錯的地方。 若要深入了解使用應用程式對應，請參閱 [Application Insights 中的應用程式對應](../application-insights/app-insights-app-map.md)。

## <a name="add-custom-instrumentation-to-your-application"></a>將自訂檢測新增至應用程式

雖然 Application Insights 提供許多立即可用的遙測資料，但建議您新增進一步的自訂檢測。 這可能是基於業務需求，或在應用程式出錯時為了改善診斷。 Application Insights 有一個 API 可內嵌自訂事件和計量，您可以在[這裡](../application-insights/app-insights-api-custom-events-metrics.md)深入了解此 API。

讓我們將一些自訂事件新增至 *VoteDataController.cs* (在 *VotingData* > *Controllers* 下)，以追蹤基礎 *votesDictionary* 中何時新增和刪除選票。

1. 在其他 using 陳述式的結尾新增 `using Microsoft.ApplicationInsights;`。
2. 在類別開頭，在建立 *IReliableStateManager* 下方宣告新的 *TelemetryClient*：`private TelemetryClient telemetry = new TelemetryClient();`。
3. 在 *Put()* 函式中，新增事件來確認已新增選票。 在交易完成之後，緊鄰在傳回的 *OkResult* 陳述式前面新增 `telemetry.TrackEvent($"Added a vote for {name}");`。
4. 在 *delete （)* 中，有一個 "if/else" 是以 *votesDictionary* 包含給定投票選項的選票為條件。
    1. 在 *if* 陳述式中，在 *await tx.CommitAsync()* 後面新增事件來確認刪除選票：`telemetry.TrackEvent($"Deleted votes for {name}");`
    2. 在 *else* 陳述式中，在 return 陳述式前面新增事件來指出刪除未發生：`telemetry.TrackEvent($"Unable to delete votes for {name}, voting option not found");`

以下示範 *Put()* 和 *Delete()* 函式在新增事件之後可能的樣子：

```csharp
// PUT api/VoteData/name
[HttpPut("{name}")]
public async Task<IActionResult> Put(string name)
{
    var votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

    using (ITransaction tx = this.stateManager.CreateTransaction())
    {
        await votesDictionary.AddOrUpdateAsync(tx, name, 1, (key, oldvalue) => oldvalue + 1);
        await tx.CommitAsync();
    }

    telemetry.TrackEvent($"Added a vote for {name}");
    return new OkResult();
}

// DELETE api/VoteData/name
[HttpDelete("{name}")]
public async Task<IActionResult> Delete(string name)
{
    var votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

    using (ITransaction tx = this.stateManager.CreateTransaction())
    {
        if (await votesDictionary.ContainsKeyAsync(tx, name))
        {
            await votesDictionary.TryRemoveAsync(tx, name);
            await tx.CommitAsync();
            telemetry.TrackEvent($"Deleted votes for {name}");
            return new OkResult();
        }
        else
        {
            telemetry.TrackEvent($"Unable to delete votes for {name}, voting option not found");
            return new NotFoundResult();
        }
    }
}
```

完成這些變更之後，請**啟動**應用程式，以建置和部署最新版本。 應用程式部署完成之後，請移至 [localhost:8080](localhost:8080)，並新增和刪除一些投票選項。 然後，回到 Application Insights 資源，以查看最後一次執行的追蹤 (同樣地，追蹤需要 1-2 分鐘，才會出現在 Application Insights 中)。 針對您新增和刪除的所有選票，您現在應該會看到「自訂事件」\*及所有回應遙測資料。

![自訂事件](./media/service-fabric-tutorial-monitoring-aspnet/custom-events.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：
> [!div class="checklist"]
> * 設定應用程式的 Application Insights
> * 收集回應遙測資料以追蹤服務之間以 HTTP 為基礎的通訊
> * 使用 Application Insights 中的應用程式對應功能
> * 使用 Application Insights API 新增自訂事件

現在您已完成設定 ASP.NET 應用程式的監視和診斷，接著請嘗試：

* [在 Service Fabric 中進一步探索監視和診斷](service-fabric-diagnostics-overview.md)
* [使用 Application Insights 分析 Service Fabric 事件](service-fabric-diagnostics-event-analysis-appinsights.md)
* 若要深入了解 Application Insights，請參閱 [Application Insights 文件](https://docs.microsoft.com/azure/application-insights/)
