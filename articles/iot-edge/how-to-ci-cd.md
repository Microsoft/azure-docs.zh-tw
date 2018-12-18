---
title: Azure IoT Edge 持續整合與持續部署 | Microsoft Docs
description: Azure IoT Edge 的持續整合與持續部署概觀
author: shizn
manager: ''
ms.author: xshi
ms.date: 06/27/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 5099ca70503ba2ed4ae8f4969a9199816c4986fb
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302566"
---
# <a name="continuous-integration-and-continuous-deployment-to-azure-iot-edge"></a>Azure IoT Edge 的持續整合與持續部署

本文示範如何使用 Azure DevOps Services 和 Microsoft Team Foundation Server (TFS) 的持續整合與持續部署功能，快速且有效率地建置、測試應用程式，並且將其部署到您的 Azure IoT Edge。 

在本文中，您將了解如何：
* 建立及簽入包含單元測試的範例 IoT Edge 解決方案。
* 為您的 Azure DevOps 安裝 Azure IoT Edge 擴充功能。
* 設定持續整合 (CI)，以建置解決方案並執行單元測試。
* 設定持續部署 (CD)，以部署解決方案並檢視回應。

完成本文中的步驟需要 30 分鐘的時間。

![CI 和 CD](./media/how-to-ci-cd/cd.png)

## <a name="create-a-sample-azure-iot-edge-solution-using-visual-studio-code"></a>使用 Visual Studio Code 建立範例 Azure IoT Edge 解決方案

在本節中，您會建立包含單元測試的範例 IoT Edge 解決方案，您可以在建置程序中執行該單元測試。 遵循本節中的指引之前，請完成[使用 Visual Studio Code 中的多個模組來開發 IoT Edge 解決方案](tutorial-multiple-modules-in-vscode.md)中的步驟。

1. 在 VS Code 命令選擇區中，輸入並執行命令**Edge: New IoT Edge solution** \(Edge：新增 IoT Edge 方案\)。 然後選取您的工作區資料夾、提供解決方案名稱 (預設名稱是 **EdgeSolution**)，然後建立 C# 模組 (**FilterModule**) 作為此解決方案中的第一個使用者模組。 您也需要為第一個模組指定 Docker 映像存放庫。 預設的映像存放庫會以本機 Docker 登錄 (`localhost:5000/filtermodule`) 為基礎。 您必須將它變更為 Azure Container Registry(`<your container registry address>/filtermodule`) 或 Docker Hub 以進一步進行持續整合。

    ![設定 ACR](./media/how-to-ci-cd/acr.png)

2. VS Code 視窗將載入您的 IoT Edge 方案工作區。 您可以選擇性地輸入及執行 **Edge：新增 IoT Edge 模組**以新增更多模組。 根資料夾中有 `modules` 資料夾、`.vscode` 資料夾，以及部署資訊清單範本檔案。 所有使用者模組代碼都將是 `modules` 資料夾下的子資料夾。 `deployment.template.json` 是部署資訊清單範本。 此檔案中的部分參數將會從 `module.json` (存在於每個模組資料夾中) 進行剖析。

3. 現在您的範例 IoT Edge 解決方案已準備就緒。 預設 C# 模組會作為管道訊息模組。 在 `deployment.template.json` 中，您將看到此解決方案包含兩個模組。 訊息將會從 `tempSensor` 模組產生，並將透過 `FilterModule` 直接進行管道傳送，然後傳送至您的 IoT 中樞。 使用以下內容取代整個 **Program.cs** 檔案。 如需此程式碼片段的詳細資訊，請參閱[建立 IoT Edge C# 模組專案](https://docs.microsoft.com/azure/iot-edge/tutorial-csharp-module#create-an-iot-edge-module-project)。

    ```csharp
    namespace FilterModule
    {
        using System;
        using System.IO;
        using System.Runtime.InteropServices;
        using System.Runtime.Loader;
        using System.Security.Cryptography.X509Certificates;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using Microsoft.Azure.Devices.Client;
        using Microsoft.Azure.Devices.Client.Transport.Mqtt;
        using System.Collections.Generic;     // for KeyValuePair<>
        using Microsoft.Azure.Devices.Shared; // for TwinCollection
        using Newtonsoft.Json;                // for JsonConvert

        public class MessageBody
        {
            public Machine machine { get; set; }
            public Ambient ambient { get; set; }
            public string timeCreated { get; set; }
        }
        public class Machine
        {
            public double temperature { get; set; }
            public double pressure { get; set; }
        }
        public class Ambient
        {
            public double temperature { get; set; }
            public int humidity { get; set; }
        }

        public class Program
        {
            static int counter;
            static int temperatureThreshold { get; set; } = 25;

            static void Main(string[] args)
            {
                Init().Wait();

                // Wait until the app unloads or is cancelled
                var cts = new CancellationTokenSource();
                AssemblyLoadContext.Default.Unloading += (ctx) => cts.Cancel();
                Console.CancelKeyPress += (sender, cpe) => cts.Cancel();
                WhenCancelled(cts.Token).Wait();
            }

            /// <summary>
            /// Handles cleanup operations when app is cancelled or unloads
            /// </summary>
            public static Task WhenCancelled(CancellationToken cancellationToken)
            {
                var tcs = new TaskCompletionSource<bool>();
                cancellationToken.Register(s => ((TaskCompletionSource<bool>)s).SetResult(true), tcs);
                return tcs.Task;
            }

            /// <summary>
            /// Initializes the ModuleClient and sets up the callback to receive
            /// messages containing temperature information
            /// </summary>
            static async Task Init()
            {
                MqttTransportSettings mqttSetting = new MqttTransportSettings(TransportType.Mqtt_Tcp_Only);
                ITransportSettings[] settings = { mqttSetting };

                // Open a connection to the Edge runtime
                ModuleClient ioTHubModuleClient = await ModuleClient.CreateFromEnvironmentAsync(settings);
                await ioTHubModuleClient.OpenAsync();
                Console.WriteLine("IoT Hub module client initialized.");

                // Register callback to be called when a message is received by the module
                await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FilterMessage, ioTHubModuleClient);
            }

            /// <summary>
            /// This method is called whenever the module is sent a message from the EdgeHub. 
            /// It just pipe the messages without any change.
            /// It prints all the incoming messages.
            /// </summary>
            static async Task<MessageResponse> FilterMessage(Message message, object userContext)
            {
                int counterValue = Interlocked.Increment(ref counter);

                var moduleClient = userContext as ModuleClient;
                if (moduleClient == null)
                {
                    throw new InvalidOperationException("UserContext doesn't contain " + "expected values");
                }

                byte[] messageBytes = message.GetBytes();
                string messageString = Encoding.UTF8.GetString(messageBytes);
                Console.WriteLine($"Received message: {counterValue}, Body: [{messageString}]");

                var filteredMessage = filter(message);

                if (filteredMessage != null && !string.IsNullOrEmpty(messageString))
                {
                    var pipeMessage = new Message(messageBytes);
                    foreach (var prop in message.Properties)
                    {
                        pipeMessage.Properties.Add(prop.Key, prop.Value);
                    }
                    await moduleClient.SendEventAsync("output1", pipeMessage);
                    Console.WriteLine("Received message sent");
                }
                return MessageResponse.Completed;
            }

            public static Message filter(Message message)
            {
                var counterValue = Interlocked.Increment(ref counter);

                var messageBytes = message.GetBytes();
                var messageString = Encoding.UTF8.GetString(messageBytes);
                Console.WriteLine($"Received message {counterValue}: [{messageString}]");

                // Get message body
                var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

                if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
                {
                    Console.WriteLine($"Machine temperature {messageBody.machine.temperature} " +
                        $"exceeds threshold {temperatureThreshold}");
                    var filteredMessage = new Message(messageBytes);
                    foreach (KeyValuePair<string, string> prop in message.Properties)
                    {
                        filteredMessage.Properties.Add(prop.Key, prop.Value);
                    }

                    filteredMessage.Properties.Add("MessageType", "Alert");
                    return filteredMessage;
                }
                return null;
            }
        }
    }
    ```

4. 建立 .Net Core 單元測試專案。 在 VS Code 檔案總管中，在您的工作區中建立新資料夾 **tests\FilterModuleTest**。 然後在 VS Code 整合式終端機 (**Ctrl + `**) 中，執行下列命令以建立 xunit 測試專案，以及將參考新增至 **FilterModule** 專案。

    ```cmd
    cd tests\FilterModuleTest
    dotnet new xunit
    dotnet add reference ../../modules/FilterModule/FilterModule.csproj
    ```

    ![資料夾結構](./media/how-to-ci-cd/add-test-project.png)

5. 在 **FilterModuleTest** 資料夾中，將 **UnitTest1.cs** 的檔案名稱更新為 **FilterModuleTest.cs**。 選取並開啟 **FilterModuleTest.cs**，使用以下程式碼片段取代整個程式碼，其中包含針對 FilterModule 專案的單元測試。

    ```csharp
    using Xunit;
    using FilterModule;
    using Newtonsoft.Json;
    using System;
    using System.IO;
    using System.Runtime.InteropServices;
    using System.Runtime.Loader;
    using System.Security.Cryptography.X509Certificates;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Client.Transport.Mqtt;

    namespace FilterModuleTest
    {
        public class FilterModuleTest
        {
            [Fact]
            public void filterLessThanThresholdTest()
            {
                var source = createMessage(25 - 1);
                var result = Program.filter(source);
                Assert.True(result == null);
            }

            [Fact]
            public void filterMoreThanThresholdAlertPropertyTest()
            {
                var source = createMessage(25 + 1);
                var result = Program.filter(source);
                Assert.True(result.Properties["MessageType"] == "Alert");
            }

            [Fact]
            public void filterMoreThanThresholdCopyPropertyTest()
            {
                var source = createMessage(25 + 1);
                source.Properties.Add("customTestKey", "customTestValue");
                var result = Program.filter(source);
                Assert.True(result.Properties["customTestKey"] == "customTestValue");
            }

            private Message createMessage(int temperature)
            {
                var messageBody = createMessageBody(temperature);
                var messageString = JsonConvert.SerializeObject(messageBody);
                var messageBytes = Encoding.UTF8.GetBytes(messageString);
                return new Message(messageBytes);
            }

            private MessageBody createMessageBody(int temperature)
            {
                var messageBody = new MessageBody
                {
                    machine = new Machine
                    {
                        temperature = temperature,
                        pressure = 0
                    },
                    ambient = new Ambient
                    {
                        temperature = 0,
                        humidity = 0
                    },
                    timeCreated = string.Format("{0:O}", DateTime.Now)
                };

                return messageBody;
            }
        }
    }
    ```

6. 在整合式終端機中，您可以輸入下列命令以在本機執行單元測試。 
    ```cmd
    dotnet test
    ```

    ![單元測試](./media/how-to-ci-cd/unit-test.png)

7. 儲存這些專案，然後將其簽入您的 Azure DevOps 或 TFS 存放庫中。
    

> [!NOTE]
> 如需使用 Azure Repos 的詳細資訊，請參閱[與 Visual Studio 和 Azure Repos 共用程式碼](https://docs.microsoft.com/azure/devops/repos/git/share-your-code-in-git-vs?view=vsts)。


## <a name="configure-continuous-integration"></a>設定持續整合
在本節中，您會建立一個組建管線，並將其設定為在您將任何變更簽入範例 IoT Edge 解決方案時自動執行，而且它也會自動執行其中包含的單元測試。

1. 登入您的 Azure DevOps 組織 (**https://**_your-account_**.visualstudio.com**)，然後開啟您簽入範例應用程式所在的專案。

    ![簽入程式碼](./media/how-to-ci-cd/init-project.png)

1. 請造訪 Azure DevOps Marketplace 上的 [Azure IoT Edge For Azure DevOps](https://marketplace.visualstudio.com/items?itemName=vsc-iot.iot-edge-build-deploy)。 按一下 [免費取得]，並依照精靈指示將此擴充功能安裝至您的 Azure DevOps 組織或下載到您的 TFS。

    ![安裝擴充功能](./media/how-to-ci-cd/install-extension.png)

1. 在您的 Azure DevOps 中開啟 [組建與發行] 中樞，然後在 [建置] 索引標籤上選擇 [+ 新增管線]。 或者，如果您已經有組建管線，請選擇 [+ 新增] 按鈕。 

    ![新增組建](./media/how-to-ci-cd/add-new-build.png)

1. 如果出現提示，請選取 [Azure DevOps Git] 來源類型，然後選取您的程式碼所在的專案、存放庫及分支。 選擇 [繼續]。

    ![選取 Azure DevOps git](./media/how-to-ci-cd/select-vsts-git.png)

1. 在 [選取範本] 視窗中，選擇 [以空白的處理序開始]。

    ![以空白開始](./media/how-to-ci-cd/start-with-empty.png)

1. 按一下 [階段 1] 右邊的 **+**，以將工作新增至階段。 然後搜尋並選取 [.Net Core]，然後按一下 [新增] 以將此工作新增至階段。

    ![Dotnet 測試](./media/how-to-ci-cd/add-dot-net-core.png)

1. 將**顯示名稱**更新為 **dotnet 測試**，然後在 [命令] 下拉式清單中選取 [測試]。 將以下路徑新增至 [專案路徑]。

    ```
    tests/FilterModuleTest/*.csproj
    ```

    ![設定 Dotnet 測試](./media/how-to-ci-cd/dotnet-test.png)

1. 按一下 [階段 1] 右邊的 **+**，以將工作新增至階段。 然後搜尋並選取 [Azure IoT Edge]，按兩下 [新增] 按鈕以將這些工作新增至階段。

    ![IoT Edge](./media/how-to-ci-cd/add-azure-iot-edge.png)

1. 在第一個 Azure IoT Edge 工作中，將**顯示名稱**更新為**模組建置及推送**，然後在 [動作] 下拉式清單中選取 [建置及推送]。 在 [Module.json 檔案] 文字方塊中，新增以下路徑。 然後選擇 [容器登錄類型]，確定您設定及選取與程式碼中相同的登錄。 這項工作會建置及推送解決方案中的所有模組，並且發佈至您指定的容器登錄。 如果您的模組會推送至不同的登錄，您可以有多個**模組建置及推送**工作。

    ```
    **/module.json
    ```

    ![模組建置及推送](./media/how-to-ci-cd/module-build-push.png)

1. 在第二個 Azure IoT Edge 工作中，將**顯示名稱**更新為**部署到 IoT Edge 裝置**，然後在 [動作] 下拉式清單中選取 [部署到 IoT Edge裝置]。 選取您的 Azure 訂用帳戶並且輸入您的 IoT 中樞名稱。 您可以指定 IoT Edge 部署識別碼和部署優先順序。 您也可以選擇要部署到單一或多個裝置。 如果您要部署到多個裝置，您必須指定裝置目標條件。 例如，如果您想要使用裝置標記作為條件，您必須在部署之前更新您的對應裝置標記。 

    ![部署到 Edge](./media/how-to-ci-cd/deploy-to-edge.png)

1. 按一下 [程序]，並確定您的**代理程式佇列**是**裝載的 Linux 預覽**。

    ![設定](./media/how-to-ci-cd/configure-env.png)

1. 開啟 [觸發程序] 索引標籤，然後開啟 [持續整合] 觸發程序。 請確定已納入包含程式碼的分支。

    ![觸發程序](./media/how-to-ci-cd/configure-trigger.png)

1. 儲存新的組建管線，並將新組建排入佇列。 按一下 [儲存並佇列] 按鈕。

1. 在顯示的訊息列中選擇組建的連結。 或者移至組建管線，以查看最新排入佇列的組建作業。

    ![建置](./media/how-to-ci-cd/build-def.png)

1. 組建完成之後，您會在即時記錄檔中看到每個工作的摘要和結果。 
    
    ![完成](./media/how-to-ci-cd/complete.png)

1. 您可以移回至 VS Code，並檢查 IoT 中樞裝置總管。 具有模組的 Edge 裝置應該會開始執行 (請確定您已將登錄認證新增至 Edge 執行階段)。

    ![Edge 執行中](./media/how-to-ci-cd/edge-running.png)

## <a name="continuous-deployment-to-iot-edge-devices"></a>IoT Edge 裝置的持續部署

若要啟用持續部署，基本上您必須設定 CI 作業與適當的 IoT Edge 裝置，為您專案中的分支啟用**觸發程序**。 在傳統 DevOps 練習中，專案會包含兩個主要分支。 主分支應該是程式碼的穩定版本，而開發分支包含最新的程式碼變更。 小組中的每個開發人員應該在開始更新程式碼時，將開發分支分支到他們自己的功能分支，這表示開發分支上的所有認可都會在功能分支上發生。 而且每個推送認可應該透過 CI 系統進行測試。 在本機完整測試程式碼之後，應該透過提取要求將功能分支合併到開發分支。 透過 CI 系統對開發人員分支上的程式碼進行測試之後，可以透過提取要求將其合併到主分支。

因此，在部署到 IoT Edge 裝置時，會有三種主要環境。
- 在功能分支上，您可以在開發機器上使用模擬的 IoT Edge 裝置，或部署到實體 IoT Edge 裝置。
- 在開發分支上，您應該部署到實體 IoT Edge 裝置。
- 在主分支上，目標 IoT Edge 裝置應該是生產裝置。

## <a name="next-steps"></a>後續步驟

* 在[了解單一裝置或大規模的 IoT Edge 部署](module-deployment-monitoring.md)中，了解 IoT Edge 部署。
* 逐步解說[大規模部署和監視 IoT Edge 模組](how-to-deploy-monitor.md)中，用來建立、更新或刪除部署的步驟。
