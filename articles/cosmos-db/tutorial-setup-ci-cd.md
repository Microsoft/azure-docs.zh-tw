---
title: 使用 Azure Cosmos DB 模擬器建置工作來設定 CI/CD 管線
description: 如何使用 Cosmos DB 模擬器建置工作，在 Visual Studio Team Services (VSTS) 中設定建置和發行工作流程的教學課程
services: cosmos-db
keywords: Azure Cosmos DB 模擬器
author: deborahc
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: tutorial
ms.date: 8/28/2018
ms.author: dech
ms.openlocfilehash: 37bb43435c34f14145b3642aa12c5cb0f16d780c
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43783713"
---
# <a name="set-up-a-cicd-pipeline-with-the-azure-cosmos-db-emulator-build-task-in-visual-studio-team-services"></a>使用 Azure Cosmos DB 模擬器建置工作，在 Visual Studio Team Services 中設定 CI/CD 管線

Azure Cosmos DB 模擬器提供了一個模擬 Azure Cosmos DB 服務的本機環境，以供開發之用。 模擬器可讓您在本機開發及測試應用程式，不需建立 Azure 訂用帳戶，也不會產生任何費用。 

適用於 Visual Studio Team Services (VSTS) 的 Azure Cosmos DB 模擬器建置工作，可讓您執行與在 CI 環境中相同的工作。 您可以使用建置工作，在建置和發行工作流程當中，針對模擬器執行測試。 工作會啟動 Docker 容器，且模擬器已在執行中，並提供可由組建定義其餘部分使用的端點。 您可以視需要建立及啟動任意數量的模擬器執行個體，每個執行個體會在個別的容器中執行。 

本文示範如何在 VSTS 中，針對使用 Cosmos DB 模擬器建置工作執行測試的 ASP.NET 應用程式，設定 CI 管線。 

## <a name="install-the-emulator-build-task"></a>安裝模擬器建置工作

若要使用建置工作，首先我們需要將它安裝到 VSTS 組織。 在 [Marketplace](https://marketplace.visualstudio.com/items?itemName=azure-cosmosdb.emulator-public-preview) (英文) 中尋找 **Azure Cosmos DB 模擬器**擴充功能，然後按一下 [免費取得]。

![在 VSTS Marketplace 中尋找 Azure Cosmos DB 模擬器建置工作並安裝](./media/tutorial-setup-ci-cd/addExtension_1.png)

接下來，選擇要在其中安裝擴充功能的組織。 

> [!NOTE]
> 若要將擴充功能安裝到 VSTS 組織，您必須是帳戶擁有者或是專案集合系統管理員。 如果您沒有權限，但是您是帳戶成員，可以改為要求擴充功能。 [深入了解。](https://docs.microsoft.com/vsts/marketplace/faq-extensions?view=vsts#install-request-assign-and-access-extensions) 

![選擇要在其中安裝擴充功能的 VSTS 組織](./media/tutorial-setup-ci-cd/addExtension_2.png)

## <a name="create-a-build-definition"></a>建立組建定義

既然已安裝擴充功能，我們需要將它新增至[組建定義](https://docs.microsoft.com/vsts/pipelines/get-started-designer?view=vsts&tabs=new-nav) (英文)。 您可以修改現有的組建定義，或建立新的組建定義。 如果您已經有現有的組建定義，可以跳到[將模擬器建置工作新增至組建定義](#addEmulatorBuildTaskToBuildDefinition)。

若要建立新的組建定義，請巡覽至 VSTS 中的 [建置和發行] 索引標籤。 選取 [+新建]。

![建立新的組建定義](./media/tutorial-setup-ci-cd/CreateNewBuildDef_1.png) 選取所需的小組專案、存放庫和分支，以啟用組建。 

![選取組建定義的小組專案、存放庫和分支 ](./media/tutorial-setup-ci-cd/CreateNewBuildDef_2.png)

最後，選取所需的組建定義範本。 我們會在本教學課程中選取 **ASP.NET** 範本。 

![選取所需的組建定義範本 ](./media/tutorial-setup-ci-cd/CreateNewBuildDef_3.png)

現在我們有了組建定義，可加以設定以使用 Azure Cosmos DB 模擬器建置工作，看起來如下所示。 

![ASP.NET 組建定義範本](./media/tutorial-setup-ci-cd/CreateNewBuildDef_4.png)

## <a name="addEmulatorBuildTaskToBuildDefinition"></a>將工作新增至組建定義

若要新增模擬器建置工作，請在搜尋方塊中搜尋 **cosmos**，然後選取 [新增]。 建置工作會啟動容器，該容器具有已在執行中的 Cosmos DB 模擬器執行個體，因此工作需要放在預期模擬器執行的任何其他工作之前。

![將模擬器建置工作新增至組建定義](./media/tutorial-setup-ci-cd/addExtension_3.png) 在本教學課程中，我們會在階段 1 開頭新增工作，確定在執行測試之前模擬器可以使用。
已完成的組建定義現在看起來像這樣。 

![ASP.NET 組建定義範本](./media/tutorial-setup-ci-cd/CreateNewBuildDef_5.png)

## <a name="configure-tests-to-use-the-emulator"></a>設定測試使用模擬器
現在，我們會設定我們的測試使用模擬器。 模擬器建置工作會匯出環境變數 (CosmosDbEmulator.Endpoint)，建置管線中任何進一步的工作都可以針對該變數發出要求。 

在本教學課程中，我們會使用 [Visual Studio 測試工作](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/VsTestV2/README.md) (英文) 來執行單元測試 (該測試會透過 **.runsettings** 檔案來設定)。 若要深入了解單位測試設定，請造訪[文件](https://docs.microsoft.com/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file?view=vs-2017)。

以下是 **.runsettings** 檔案的範例，該檔案會定義要傳遞至應用程式單元測試的參數。 請注意，所使用的 `authKey` 變數是模擬器的[已知金鑰](https://docs.microsoft.com/azure/cosmos-db/local-emulator#authenticating-requests)。 這個 `authKey` 是模擬器建置工作所預期的金鑰，應該在您的 **.runsettings** 檔案中定義。

```csharp
<RunSettings>
    <TestRunParameters>
    <Parameter name="endpoint" value="https://localhost:8081" />
    <Parameter name="authKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==" />
    <Parameter name="database" value="ToDoListTest" />
    <Parameter name="collection" value="ItemsTest" />
  </TestRunParameters>
</RunSettings>
```
這些參數 `TestRunParameters` 會透過應用程式測試專案中的 `TestContext` 屬性進行參考。 以下是針對 Cosmos DB 執行測試的範例。 

```csharp
namespace todo.Tests
{
    [TestClass]
    public class TodoUnitTests
    {
        public TestContext TestContext { get; set; }

        [TestInitialize()]
        public void Initialize()
        {
            string endpoint = TestContext.Properties["endpoint"].ToString();
            string authKey = TestContext.Properties["authKey"].ToString();
            Console.WriteLine("Using endpoint: ", endpoint);
            DocumentDBRepository<Item>.Initialize(endpoint, authKey);
        }
        [TestMethod]
        public async Task TestCreateItemsAsync()
        {
            var item = new Item
            {
                Id = "1",
                Name = "testName",
                Description = "testDescription",
                Completed = false,
                Category = "testCategory"
            };

            // Create the item
            await DocumentDBRepository<Item>.CreateItemAsync(item);
            // Query for the item
            var returnedItem = await DocumentDBRepository<Item>.GetItemAsync(item.Id, item.Category);
            // Verify the item returned is correct.
            Assert.AreEqual(item.Id, returnedItem.Id);
            Assert.AreEqual(item.Category, returnedItem.Category);
        }

        [TestCleanup()]
        public void Cleanup()
        {
            DocumentDBRepository<Item>.Teardown();
        }
    }
}
```

巡覽至 Visual Studio 測試工作中的執行選項。 在 [設定檔] 選項中，請指定測試將使用 **.runsettings** 檔案進行設定。 在 [覆寫測試回合參數] 選項中，於 ` -endpoint $(CosmosDbEmulator.Endpoint)` 中新增。 如此一來，就會設定測試工作參考模擬器建置工作的端點，而不是在 **.runsettings** 檔案中定義的端點。  

![使用模擬器建置工作端點來覆寫端點變數](./media/tutorial-setup-ci-cd/addExtension_5.png)

## <a name="run-the-build"></a>執行組建
現在，請儲存組建並排入佇列中。 

![儲存並執行組建](./media/tutorial-setup-ci-cd/runBuild_1.png)

一旦開始建置，請觀察 Cosmos DB 模擬器工作是否開始使用安裝的模擬器下拉 Docker 映像。 

![儲存並執行組建](./media/tutorial-setup-ci-cd/runBuild_4.png)

在建置完成之後，請觀察您的測試是否通過，所有測試都是針對來自建置工作的 Cosmos DB 模擬器而執行！

![儲存並執行組建](./media/tutorial-setup-ci-cd/buildComplete_1.png)

## <a name="next-steps"></a>後續步驟

若要深入了解使用模擬器來進行本機開發和測試，請參閱[使用 Azure Cosmos DB 模擬器進行本機開發和測試](https://docs.microsoft.com/azure/cosmos-db/local-emulator)。

若要匯出模擬器 SSL 憑證，請參閱[匯出 Azure Cosmos DB 模擬器憑證，以便與 Java、Python 和 Node.js 搭配使用](https://docs.microsoft.com/azure/cosmos-db/local-emulator-export-ssl-certificates)
