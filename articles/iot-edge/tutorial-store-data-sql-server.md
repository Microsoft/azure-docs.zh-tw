---
title: 使用 Azure IoT Edge SQL 模組儲存資料 | Microsoft Docs
description: 了解如何使用 SQL Server 模組將資料儲存在 IoT Edge 裝置的本機上
services: iot-edge
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 09/21/2018
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 88537c8579a3acb2fd89a43bfcfa50be89cac5cc
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49114085"
---
# <a name="tutorial-store-data-at-the-edge-with-sql-server-databases"></a>教學課程：使用 SQL Server 資料庫在 Edge 上儲存資料

使用 Azure IoT Edge 和 SQL Server 儲存和查詢 Edge 上的資料。 Azure IoT Edge 內建有基本儲存體功能，可在裝置離線時快取訊息，然後在連線重新建立時轉送訊息。 不過，您可能想要有更進階的儲存體功能，例如能夠在本機查詢資料。 藉由納入本機資料庫，您的 IoT Edge 裝置將可執行更複雜的計算，而無須保有 IoT 中樞的連線。 例如，現場技術人員可以在本機電腦將過去幾天感應器資料視覺化，即使該資料僅逐月上傳至雲端以協助改善機器學習模型。

本文提供如何將 SQL Server 資料庫部署到 IoT Edge 裝置的指示。 在 IoT Edge 裝置上執行的 Azure Functions 會將傳入的資料結構化，然後傳送到資料庫。 本文中的步驟也可套用到會在 MySQL 或 PostgreSQL 等容器中工作的其他資料庫。

在本教學課程中，您了解如何： 

> [!div class="checklist"]
> * 使用 Visual Studio Code 來建立 Azure Function
> * 將 SQL 資料庫部署至您的 IoT Edge 裝置
> * 使用 Visual Studio Code 建置模組，並將其部署至您的 IoT Edge 裝置
> * 檢視產生的資料

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

Azure IoT Edge 裝置：

* 您可以遵循 [Linux](quickstart-linux.md) 或 [Windows 裝置](quickstart.md)快速入門中的步驟，使用您的開發電腦或虛擬機器作為邊緣裝置。

雲端資源：

* Azure 中的免費或標準層 [IoT 中樞](../iot-hub/iot-hub-create-through-portal.md)。 

開發資源：

* [Visual Studio Code](https://code.visualstudio.com/)。 
* [C# for Visual Studio Code (由 OmniSharp 提供技術支援) 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) (適用於 Visual Studio Code)。 
* [適用於 Visual Studio Code 的 Azure IoT Edge 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge)。 
* [.NET Core 2.1 SDK](https://www.microsoft.com/net/download)。 
* [Docker CE](https://docs.docker.com/install/)。 

## <a name="create-a-container-registry"></a>建立容器登錄庫
在本教學課程中，您會使用適用於 VS Code 的 Azure IoT Edge 擴充功能來建置模組，並從檔案建立**容器映像**。 接著，您會將此映像推送至儲存並管理映像的**登錄**。 最後，您會從登錄部署該映像，以在 IoT Edge 裝置上執行。  

您可以此教學課程中使用任何與 Docker 相容的登錄。 於雲端中提供使用的兩個熱門 Docker 登錄服務為 [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) 和 [Docker Hub](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags) \(英文\)。 本教學課程使用的是 Azure Container Registry。 

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [建立資源] > [容器] > [容器登錄]。

    ![建立容器登錄](./media/tutorial-deploy-function/create-container-registry.png)

2. 為您的登錄指定名稱，然後選擇訂用帳戶。
3. 針對資源群組，建議您使用包含 IoT 中樞的相同資源群組名稱。 您可以將所有資源放在相同群組中，而一併加以管理。 例如，刪除用於測試的資源群組時，將會刪除該群組中包含的所有測試資源。 
4. 將 SKU 設定為 [基本]，並將 [管理使用者] 切換為 [啟用]。 
5. 按一下頁面底部的 [新增] 。
6. 建立容器登錄之後，請瀏覽至它並選取 [存取金鑰]。 
7. 複製 [登入伺服器]、[使用者名稱] 及 [密碼] 的值。 您在本教學課程後續的內容中將會用到這些值。 

## <a name="create-a-function-project"></a>建立函式專案

若要將資料傳送到資料庫中，您需要可將資料適當結構化，並將其儲存在資料表中的模組。 

下列步驟會示範如何使用 Visual Studio Code 和 Azure IoT Edge 擴充功能來建立 IoT Edge 函式。

1. 開啟 Visual Studio Code。
2. 選取 [檢視] > [終端機]，以開啟 VS Code 整合式終端機。
3. 選取 [檢視] > [命令選擇區]，以開啟 VS Code 命令選擇區。
4. 在命令選擇區中，輸入並執行命令 **Azure: Sign in**，然後依照指示登入您的 Azure 帳戶。 如果您已登入，則可以略過此步驟。
3. 在命令選擇區中，輸入並執行命令 **Azure IoT Edge: New IoT Edge solution**。 在命令選擇區中提供下列資訊，以建立解決方案： 
   1. 選取要用來建立解決方案的資料夾。 
   2. 為解決方案提供名稱，或是接受預設值 **EdgeSolution**。
   3. 選擇 [Azure Functions - C#] 作為模組範本。 
   4. 將模組命名為 **sqlFunction**。 
   5. 將您在上一節所建立的 Azure Container Registry 指定為第一個模組的映像存放庫。 將 **localhost:5000** 取代為您所複製的登入伺服器值。 最終字串看起來會類似於：**\<登錄名稱\>.azurecr.io/sqlFunction**。

4. VS Code 視窗會載入您的 IoT Edge 方案工作區。 裡面會有 **modules** 資料夾、**.vscode** 資料夾，以及部署資訊清單範本檔案。 開啟 [模組] > [sqlFunction] > [EdgeHubTrigger-Csharp] > [run.csx]。

5. 使用下列程式碼取代檔案的內容：

   ```csharp
   #r "Microsoft.Azure.Devices.Client"
   #r "Newtonsoft.Json"
   #r "System.Data.SqlClient"

   using System.IO;
   using Microsoft.Azure.Devices.Client;
   using Newtonsoft.Json;
   using Sql = System.Data.SqlClient;
   using System.Threading.Tasks;

   // Filter messages based on the temperature value in the body of the message and the temperature threshold value.
   public static async Task Run(Message messageReceived, IAsyncCollector<Message> output, TraceWriter log)
   {
       const int temperatureThreshold = 25;
       byte[] messageBytes = messageReceived.GetBytes();
       var messageString = System.Text.Encoding.UTF8.GetString(messageBytes);

       if (!string.IsNullOrEmpty(messageString))
       {
           // Get the body of the message and deserialize it
           var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

           //Store the data in SQL db
           const string str = "<sql connection string>";
           using (Sql.SqlConnection conn = new Sql.SqlConnection(str))
           {
           conn.Open();
           var insertMachineTemperature = "INSERT INTO MeasurementsDB.dbo.TemperatureMeasurements VALUES (CONVERT(DATETIME2,'" + messageBody.timeCreated + "', 127), 'machine', " + messageBody.machine.temperature + ");";
           var insertAmbientTemperature = "INSERT INTO MeasurementsDB.dbo.TemperatureMeasurements VALUES (CONVERT(DATETIME2,'" + messageBody.timeCreated + "', 127), 'ambient', " + messageBody.ambient.temperature + ");"; 
               using (Sql.SqlCommand cmd = new Sql.SqlCommand(insertMachineTemperature + "\n" + insertAmbientTemperature, conn))
               {
               //Execute the command and log the # rows affected.
               var rows = await cmd.ExecuteNonQueryAsync();
               log.Info($"{rows} rows were updated");
               }
           }

           if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
           {
               // Send the message to the output as the temperature value is greater than the threshold
               var filteredMessage = new Message(messageBytes);
               // Copy the properties of the original message into the new Message object
               foreach (KeyValuePair<string, string> prop in messageReceived.Properties)
               {
                   filteredMessage.Properties.Add(prop.Key, prop.Value);
               }
               // Add a new property to the message to indicate it is an alert
               filteredMessage.Properties.Add("MessageType", "Alert");
               // Send the message        
               await output.AddAsync(filteredMessage);
               log.Info("Received and transferred a message with temperature above the threshold");
           }
       }
   }

   //Define the expected schema for the body of incoming messages
   class MessageBody
   {
       public Machine machine {get;set;}
       public Ambient ambient {get; set;}
       public string timeCreated {get; set;}
   }
   class Machine
   {
      public double temperature {get; set;}
      public double pressure {get; set;}         
   }
   class Ambient
   {
      public double temperature {get; set;}
      public int humidity {get; set;}         
   }
   ```

6. 在第 24 行中，將字串 **\<sql 連接字串\>** 取代為下列字串。 **資料來源**屬性會參考 SQL Server 容器名稱，這是您將在下一節以名稱 **SQL** 建立的名稱。 

   ```C#
   Data Source=tcp:sql,1433;Initial Catalog=MeasurementsDB;User Id=SA;Password=Strong!Passw0rd;TrustServerCertificate=False;Connection Timeout=30;
   ```

7. 儲存 **run.csx** 檔案。 

## <a name="add-a-sql-server-container"></a>新增 SQL Server 容器

[部署資訊清單](module-composition.md)會宣告 IoT Edge 執行階段會在 IoT Edge 裝置上安裝哪些模組。 您在上一節中新增了用來建立自訂函式模組的程式碼，但 SQL Server 模組已內建。 您只需要指示 IoT Edge 執行階段納入該模組，然後在裝置上加以設定即可。 

1. 在 Visual Studio Code 總管中，開啟 **deployment.template.json** 檔案。 
2. 尋找 **moduleContent.$edgeAgent.properties.desired.modules** 區段。 此時應該會列出兩個模組：會產生模擬資料的 **tempSensor**，和您的 **sqlFunction** 模組。
3. 如果您使用 Windows 容器，請修改 **sqlFunction.settings.image** 區段。
    ```json
    "image": "${MODULES.sqlFunction.windows-amd64}"
    ```
4. 新增下列程式碼以宣告第三個模組。 在 sqlFunction 區段之後加上逗號並插入：

   ```json
   "sql": {
       "version": "1.0",
       "type": "docker",
       "status": "running",
       "restartPolicy": "always",
       "settings": {
           "image": "",
           "createOptions": ""
       }
   }
   ```

   若您對新增 JSON 元素有任何不解之處，以下提供了範例。 ![新增 SQL Server 容器](./media/tutorial-store-data-sql-server/view_json_sql.png)

5. 請根據您 IoT Edge 裝置上的 Docker 容器類型，使用下列程式碼更新 **sql.settings** 參數：

   * Windows 容器：

      ```json
      "image": "microsoft/mssql-server-windows-developer",
      "createOptions": "{\"Env\": [\"ACCEPT_EULA=Y\",\"SA_PASSWORD=Strong!Passw0rd\"],\"HostConfig\": {\"Mounts\": [{\"Target\": \"C:\\\\mssql\",\"Source\": \"sqlVolume\",\"Type\": \"volume\"}],\"PortBindings\": {\"1433/tcp\": [{\"HostPort\": \"1401\"}]}}}"
      ```

   * Linux 容器：

      ```json
      "image": "mcr.microsoft.com/mssql/server:latest",
      "createOptions": "{\"Env\": [\"ACCEPT_EULA=Y\",\"MSSQL_SA_PASSWORD=Strong!Passw0rd\"],\"HostConfig\": {\"Mounts\": [{\"Target\": \"/var/opt/mssql\",\"Source\": \"sqlVolume\",\"Type\": \"volume\"}],\"PortBindings\": {\"1433/tcp\": [{\"HostPort\": \"1401\"}]}}}"
      ```

   >[!Tip]
   >每當您在生產環境中建立了 SQL Server 容器時，就應該[變更預設的系統管理員密碼](https://docs.microsoft.com/sql/linux/quickstart-install-connect-docker#change-the-sa-password)。

6. 儲存 **deployment.template.json** 檔案。

## <a name="build-your-iot-edge-solution"></a>建置 IoT Edge 解決方案

在前幾節中，您以一個模組建立了解決方案，然後將另一個模組新增至部署資訊清單範本。 現在，您必須建置解決方案、建立模組的容器映像，並將其推送至容器登錄。 

1. 在 .env 檔案中，為 IoT Edge 執行階段提供您的登錄認證，使其能夠存取您的模組映像。 找出 **CONTAINER_REGISTRY_USERNAME** 和 **CONTAINER_REGISTRY_PASSWORD** 區段，在等號後面插入您的認證： 

   ```env
   CONTAINER_REGISTRY_USERNAME_yourContainerReg=<username>
   CONTAINER_REGISTRY_PASSWORD_yourContainerReg=<password>
   ```

2. 儲存 .env 檔案。
3. 在 Visual Studio Code 中登入您的容器登錄，以便將您的映像推送到登錄。 請使用您新增至 .env 檔案中的相同認證。 在整合式終端機中輸入下列命令：

    ```csh/sh
    docker login -u <ACR username> <ACR login server>
    ```
    系統會提示您輸入密碼。 將您的密碼貼到提示中，然後按 **Enter** 鍵。

    ```csh/sh
    Password: <paste in the ACR password and press enter>
    Login Succeeded
    ```

4. 在 VS Code 總管中，以滑鼠右鍵按一下 **deployment.template.json** 檔案，然後選取 [建置並推送 IoT Edge 解決方案]。 

## <a name="deploy-the-solution-to-a-device"></a>將解決方案部署至裝置

您可以透過 IoT 中樞在裝置上設定模組，但您也可以透過 Visual Studio Code 存取 IoT 中樞和裝置。 在本節中，您會設定對 IoT 中樞的存取，然後使用 VS Code 將解決方案部署至您的 IoT Edge 裝置。 

1. 在 VS Code 命令選擇區中，選取 [Azure IoT 中樞: 選取 IoT 中樞]。
2. 依照提示來登入您的 Azure 帳戶。 
3. 在命令選擇區中選取您的 Azure 訂用帳戶，然後選取 IoT 中樞。 
4. 在 VS Code 總管中，展開 [Azure IoT 中樞裝置] 區段。 
5. 以滑鼠右鍵按一下部署的目標裝置，然後選取 [建立單一裝置的部署]。 
6. 在檔案總管中，瀏覽至解決方案內的 **config** 資料夾，然後選擇 **deployment.json**。 按一下 [選取 Edge 部署資訊清單]。 

如果部署成功，確認訊息會列印在 VS Code 輸出中。 您也可以查看並確認所有模組均已在裝置上啟動並執行。 

在 IoT Edge 裝置上執行下列命令，以查看模組的狀態。 可能需要數分鐘的時間。

   ```PowerShell
   iotedge list
   ```

## <a name="create-the-sql-database"></a>建立 SQL 資料庫

當您將部署資訊清單套用至裝置時，您會有三個執行中的模組。 tempSensor 模組會產生模擬的環境資料。 sqlFunction 模組會取用資料，並針對資料庫加以格式化。 

本節會引導您設定 SQL 資料庫，以便儲存溫度資料。 

1. 在命令列工具中連線至您的資料庫。 
   * Windows 容器：
   
      ```cmd
      docker exec -it sql cmd
      ```
    
   * Linux 容器： 

      ```bash
      docker exec -it sql bash
      ```

2. 開啟 SQL 命令工具。
   * Windows 容器：

      ```cmd
      sqlcmd -S localhost -U SA -P "Strong!Passw0rd"
      ```

   * Linux 容器： 

      ```bash
      /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Strong!Passw0rd'
      ```

3. 建立您的資料庫： 

   * Windows 容器
      ```sql
      CREATE DATABASE MeasurementsDB
      ON
      (NAME = MeasurementsDB, FILENAME = 'C:\mssql\measurementsdb.mdf')
      GO
      ```

   * Linux 容器
      ```sql
      CREATE DATABASE MeasurementsDB
      ON
      (NAME = MeasurementsDB, FILENAME = '/var/opt/mssql/measurementsdb.mdf')
      GO
      ```

4. 定義您的資料表。

   ```sql
   CREATE TABLE MeasurementsDB.dbo.TemperatureMeasurements (measurementTime DATETIME2, location NVARCHAR(50), temperature FLOAT)
   GO
   ```

您可以自訂 SQL Server Docker 檔案，以便自動設定您要部署在多個 IoT Edge 裝置上的 SQL Server。 如需詳細資訊，請參閱 [Microsoft SQL Server 容器示範專案](https://github.com/twright-msft/mssql-node-docker-demo-app)。 

## <a name="view-the-local-data"></a>檢視本機資料

在您的資料表建立後，sqlFunction 模組即會開始將資料儲存在 IoT Edge 裝置上的本機 SQL Server 2017 資料庫中。 

在 SQL 命令工具中執行下列命令，以檢視您格式化的資料表資料： 

   ```sql
   SELECT * FROM MeasurementsDB.dbo.TemperatureMeasurements
   GO
   ```

   ![檢視本機資料](./media/tutorial-store-data-sql-server/view-data.png)



## <a name="clean-up-resources"></a>清除資源

如果您打算繼續閱讀下一篇建議的文章，則可以保留您所建立的資源和組態，並加以重複使用。 您可以也繼續使用相同的 IoT Edge 裝置作為測試裝置。 

否則，可以刪除您在本文中建立的本機組態和 Azure 資源，以避免產生費用。 

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

[!INCLUDE [iot-edge-clean-up-local-resources](../../includes/iot-edge-clean-up-local-resources.md)]



## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立包含程式碼的 Azure Functions 模組，用以篩選您的 IoT Edge 裝置所產生的原始資料。 當您準備好要建置自己的模組時，您可以進一步了解如何[使用適用於 Visual Studio Code 的 Azure IoT Edge 開發 Azure Functions](how-to-develop-csharp-function.md)。 

請繼續進行後續教學課程，以了解 Azure IoT Edge 有什麼其他方法可協助您將此資料轉換成具有優勢的商業見解。

> [!div class="nextstepaction"]
> [使用 Azure 串流分析中的浮動視窗尋找平均值](tutorial-deploy-stream-analytics.md)
