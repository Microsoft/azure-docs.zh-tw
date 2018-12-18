---
title: Azure IoT Edge C 教學課程 | Microsoft Docs
description: 本教學課程說明如何使用 C 程式碼建立 IoT Edge 模組，並將其部署到邊緣裝置
services: iot-edge
author: shizn
manager: timlt
ms.author: xshi
ms.date: 09/21/2018
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: fff53e7412a17d1b2c1c444e189151651d8d3235
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166928"
---
# <a name="tutorial-develop-a-c-iot-edge-module-and-deploy-to-your-simulated-device"></a>教學課程：開發 C IoT Edge 模組並部署至模擬裝置

您可以使用 IoT Edge 模組來部署程式碼，將您的商務邏輯直接實作到您的 IoT Edge 裝置。 本教學課程會逐步引導您建立並部署能篩選感應器資料的 IoT Edge 模組。 在本教學課程中，您了解如何：    

> [!div class="checklist"]
> * 使用 Visual Studio Code 以 C 語言建立 IoT Edge 模組
> * 使用 Visual Studio Code 和 Docker 建立 Docker 映像，並將其發佈至容器登錄 
> * 將模組部署到您的 IoT Edge 裝置
> * 檢視產生的資料


您於此教學課程中建立的 IoT Edge 模組，能夠篩選由您的裝置所產生的溫度資料。 它只有在溫度超過指定的閾值時，才會將訊息往上游傳送。 這類於邊緣所進行的分析，對於減少針對雲端所傳輸及儲存的資料量相當有幫助。 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="prerequisites"></a>必要條件

Azure IoT Edge 裝置：

* 您可以遵循 [Linux](quickstart-linux.md) 或 [Windows 裝置](quickstart.md)快速入門中的步驟，使用您的開發電腦或虛擬機器作為邊緣裝置。
* 適用於 Azure IoT Edge 的 C 模組不支援 Windows 容器。 如果您的 IoT Edge 裝置是 Windows 機器，將它設定為[使用 Linux 容器](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers)

雲端資源：

* Azure 中的免費或標準層 [IoT 中樞](../iot-hub/iot-hub-create-through-portal.md)。 

開發資源：

* [Visual Studio Code](https://code.visualstudio.com/)。 
* 適用於 Visual Studio Code 的 [C/C++ 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)。
* 適用於 Visual Studio Code 的 [Azure IoT Edge 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge)。
* [Docker CE](https://docs.docker.com/install/)。 

>[!Note]
>適用於 Azure IoT Edge 的 C 模組不支援 Windows 容器。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="create-a-container-registry"></a>建立容器登錄庫
在本教學課程中，您會使用適用於 VS Code 的 Azure IoT Edge 擴充功能來建置模組，並從檔案建立**容器映像**。 接著，您會將此映像推送至儲存並管理映像的**登錄**。 最後，您會從登錄部署該映像，以在 IoT Edge 裝置上執行。  

您可以此教學課程中使用任何與 Docker 相容的登錄。 於雲端中提供使用的兩個熱門 Docker 登錄服務為 [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) 和 [Docker Hub](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags) \(英文\)。 本教學課程使用的是 Azure Container Registry。 

下列 Azure CLI 命令會在名為 **IoTEdgeResources** 的資源群組中建立登錄。 將 **{acr_name}** 更換為登錄的唯一名稱。 

   ```azurecli-interactive
   az acr create --resource-group IoTEdgeResources --name {acr_name} --sku Basic --admin-enabled true
   ```

擷取登錄的認證。 

   ```azurecli-interactive
   az acr credential show --name {acr_name}
   ```

複製 [使用者名稱] 的值和其中一個密碼。 稍後您在本教學課程中，將 Docker 映像發佈到您的登錄，或將登錄認證新增至 Edge 執行階段時，將會用到這些值。 

## <a name="create-an-iot-edge-module-project"></a>建立 IoT Edge 模組專案
下列步驟說明如何使用 Visual Studio Code 和 Azure IoT Edge 擴充功能，建立以 .NET Core 2.0 為基礎的 IoT Edge 模組專案。

### <a name="create-a-new-solution"></a>建立新解決方案

建立您可以使用自己的程式碼自訂的 C 解決方案範本。 

1. 選取 [檢視] > [命令選擇區]，以開啟 VS Code 命令選擇區。 

2. 在命令選擇區中，輸入並執行命令 **Azure: Sign in**，然後依照指示登入您的 Azure 帳戶。 如果您已登入，則可以略過此步驟。

3. 在命令選擇區中，輸入並執行命令 **Azure IoT Edge: New IoT Edge solution**。 在命令選擇區中提供下列資訊，以建立解決方案： 

   1. 選取要用來建立解決方案的資料夾。 
   2. 為解決方案提供名稱，或是接受預設值 **EdgeSolution**。
   3. 選擇 [C 模組] 作為模組範本。 
   4. 將模組命名為 **CModule**。 
   5. 將您在上一節所建立的 Azure Container Registry 指定為第一個模組的映像存放庫。 將 **localhost:5000** 取代為 **\<登錄名稱\>.azurecr.io**。 僅取代字串的 localhost 部分即可，不要刪除您的模組名稱。 

   ![提供 Docker 映像存放庫](./media/tutorial-c-module/repository.png)

VS Code 視窗會載入您的 IoT Edge 方案工作區。 解決方案工作區包含五個最上層元件。 您在本教學課程中將不會編輯 **\.vscode** 資料夾或 **\.gitignore** 檔案。 **modules** 資料夾包含您的模組所需的 C 程式碼，以及用來將模組建置為容器映像的 Dockerfile。 **\.env** 檔案會儲存您的容器登錄認證。 **Deployment.template.json** 檔案包含 IoT Edge 執行階段用來在裝置上部署模組的資訊。 

如果您在建立解決方案時未指定容器登錄，但接受了預設的 localhost:5000 值，則不會有 \.env 檔案。 

   ![C 解決方案工作區](./media/tutorial-c-module/workspace.png)

### <a name="add-your-registry-credentials"></a>新增登錄認證

環境檔案會儲存容器登錄的認證，並與 IoT Edge 執行階段共用這些認證。 執行階段需要有這些認證才能將私人映像提取到 IoT Edge 裝置。 

1. 在 VS Code 總管中，開啟 .env 檔案。 
2. 使用從 Azure Container Registry 複製過來的 [使用者名稱] 和 [密碼] 值來更新欄位。 
3. 儲存這個檔案。 

### <a name="update-the-module-with-custom-code"></a>使用自訂程式碼來更新模組

將程式碼新增至您的 C 模組，使其能夠從感應器讀取資料、檢查回報的機器溫度是否超過安全閾值，並將該資訊傳至 IoT 中樞。 

5. 在此案例中，來自感應器的資料會以 JSON 格式傳入。 若要篩選 JSON 格式的訊息，請匯入適用於 C 的 JSON 程式庫。本教學課程使用 Parson。

   1. 下載 [Parson Github 存放庫](https://github.com/kgabis/parson)。 將 **parson.c** 和 **parson.h** 檔案複製到 **CModule** 資料夾中。

   2. 開啟 [模組] > [CModule] > [CMakeLists.txt]。 在檔案頂端匯入 Parson 檔案，作為名為 **my_parson** 的程式庫。

      ```
      add_library(my_parson
          parson.c
          parson.h
      )
      ```

   3. 在 CMakeLists.txt 的 **target_link_libraries** 函式中，將 **my_parson** 新增至程式庫清單。

   4. 儲存 **CMakeLists.txt** 檔案。

   5. 開啟 [模組] > [CModule] > [main.c]。 在 Include 陳述式清單底部新增包含 `parson.h` 的新陳述式，以支援 JSON：

      ```c
      #include "parson.h"
      ```

6. 在 **main.c** 檔案中的 Include 區段後面，新增名為 `temperatureThreshold` 的全域變數。 此變數會設定在將資料傳送至 IoT 中樞之前，測量的溫度必須超過的值。 

    ```c
    static double temperatureThreshold = 25;
    ```

7. 將整個 `CreateMessageInstance` 函式取代為下列程式碼。 此函式會配置回呼的內容。 

    ```c
    static MESSAGE_INSTANCE* CreateMessageInstance(IOTHUB_MESSAGE_HANDLE message)
    {
        MESSAGE_INSTANCE* messageInstance = (MESSAGE_INSTANCE*)malloc(sizeof(MESSAGE_INSTANCE));
        if (NULL == messageInstance)
        {
            printf("Failed allocating 'MESSAGE_INSTANCE' for pipelined message\r\n");
        }
        else
        {
            memset(messageInstance, 0, sizeof(*messageInstance));

            if ((messageInstance->messageHandle = IoTHubMessage_Clone(message)) == NULL)
            {
                free(messageInstance);
                messageInstance = NULL;
            }
            else
            {
                messageInstance->messageTrackingId = messagesReceivedByInput1Queue;
                MAP_HANDLE propMap = IoTHubMessage_Properties(messageInstance->messageHandle);
                if (Map_AddOrUpdate(propMap, "MessageType", "Alert") != MAP_OK)
                {
                    printf("ERROR: Map_AddOrUpdate Failed!\r\n");
                }
            }
        }

        return messageInstance;
    }
    ```

8. 將整個 `InputQueue1Callback` 函式取代為下列程式碼。 此函式會實作實際的傳訊篩選條件。 

    ```c
    static IOTHUBMESSAGE_DISPOSITION_RESULT InputQueue1Callback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
    {
        IOTHUBMESSAGE_DISPOSITION_RESULT result;
        IOTHUB_CLIENT_RESULT clientResult;
        IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle = (IOTHUB_MODULE_CLIENT_LL_HANDLE)userContextCallback;

        unsigned const char* messageBody;
        size_t contentSize;

        if (IoTHubMessage_GetByteArray(message, &messageBody, &contentSize) != IOTHUB_MESSAGE_OK)
        {
            messageBody = "<null>";
        }

        printf("Received Message [%zu]\r\n Data: [%s]\r\n", 
                messagesReceivedByInput1Queue, messageBody);

        JSON_Value *root_value = json_parse_string(messageBody);
        JSON_Object *root_object = json_value_get_object(root_value);
        double temperature;
        if (json_object_dotget_value(root_object, "machine.temperature") != NULL && (temperature = json_object_dotget_number(root_object, "machine.temperature")) > temperatureThreshold)
        {
            printf("Machine temperature %f exceeds threshold %f\r\n", temperature, temperatureThreshold);
            // This message should be sent to next stop in the pipeline, namely "output1".  What happens at "outpu1" is determined
            // by the configuration of the Edge routing table setup.
            MESSAGE_INSTANCE *messageInstance = CreateMessageInstance(message);
            if (NULL == messageInstance)
            {
                result = IOTHUBMESSAGE_ABANDONED;
            }
            else
            {
                printf("Sending message (%zu) to the next stage in pipeline\n", messagesReceivedByInput1Queue);

                clientResult = IoTHubModuleClient_LL_SendEventToOutputAsync(iotHubModuleClientHandle, messageInstance->messageHandle, "output1", SendConfirmationCallback, (void *)messageInstance);
                if (clientResult != IOTHUB_CLIENT_OK)
                {
                    IoTHubMessage_Destroy(messageInstance->messageHandle);
                    free(messageInstance);
                    printf("IoTHubModuleClient_LL_SendEventToOutputAsync failed on sending msg#=%zu, err=%d\n", messagesReceivedByInput1Queue, clientResult);
                    result = IOTHUBMESSAGE_ABANDONED;
                }
                else
                {
                    result = IOTHUBMESSAGE_ACCEPTED;
                }
            }
        }
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

9. 新增 `moduleTwinCallback` 函式。 此方法會從模組對應項接收所需的屬性，並會更新 **temperatureThreshold** 變數以符合該屬性。 所有模組都具有自己的模組對應項，這可讓您直接從雲端設定於模組內執行的程式碼。

    ```c
    static void moduleTwinCallback(DEVICE_TWIN_UPDATE_STATE update_state, const unsigned char* payLoad, size_t size, void* userContextCallback)
    {
        printf("\r\nTwin callback called with (state=%s, size=%zu):\r\n%s\r\n", 
            ENUM_TO_STRING(DEVICE_TWIN_UPDATE_STATE, update_state), size, payLoad);
        JSON_Value *root_value = json_parse_string(payLoad);
        JSON_Object *root_object = json_value_get_object(root_value);
        if (json_object_dotget_value(root_object, "desired.TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_dotget_number(root_object, "desired.TemperatureThreshold");
        }
        if (json_object_get_value(root_object, "TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_get_number(root_object, "TemperatureThreshold");
        }
    }
    ```

10. 將 `SetupCallbacksForModule` 函式取代為下列程式碼。 

    ```c
    static int SetupCallbacksForModule(IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle)
    {
        int ret;

        if (IoTHubModuleClient_LL_SetInputMessageCallback(iotHubModuleClientHandle, "input1", InputQueue1Callback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
        {
            printf("ERROR: IoTHubModuleClient_LL_SetInputMessageCallback(\"input1\")..........FAILED!\r\n");
            ret = __FAILURE__;
        }
        else if (IoTHubModuleClient_LL_SetModuleTwinCallback(iotHubModuleClientHandle, moduleTwinCallback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
        {
            printf("ERROR: IoTHubModuleClient_LL_SetModuleTwinCallback(default)..........FAILED!\r\n");
            ret = __FAILURE__;
        }
        else
        {
            ret = 0;
        }

        return ret;
    }
    ```

11. 儲存 **main.c** 檔案。

## <a name="build-your-iot-edge-solution"></a>建置 IoT Edge 解決方案

在上一節中，您已建立 IoT Edge 解決方案，並在 CModule 中新增了程式碼，以篩選出回報的機器溫度在可接受限制內的訊息。 現在，您需要建置容器映像形式的解決方案，並將它推送到容器登錄。 

1. 選取 [檢視] > [整合式終端機]，以開啟 VS Code 整合式終端機。 

1. 透過在 Visual Studio Code 整合式終端機中輸入下列命令，來登入 Docker。 您必須使用 Azure Container Registry 認證登入，才可將您的模組映像推送至登錄。 
     
   ```csh/sh
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```
   使用您在第一節中從 Azure Container Registry 複製而來的使用者名稱、密碼及登入伺服器。 或者，在 Azure 入口網站中從登錄的 [存取金鑰] 區段再擷取一次這些資料。

2. 在 VS Code 總管中，於 IoT Edge 解決方案工作區中開啟 **deployment.template.json** 檔案。 此檔案會指示 `$edgeAgent` 部署兩個模組：**tempSensor** 和 **CModule**。 `CModule.image` 值會設定為映像的 Linux amd64 版本。 若要深入了解部署資訊清單，請參閱[了解如何使用、設定以及重複使用 IoT Edge 模組](module-composition.md)。

4. 在部署資訊清單中新增 CModule 模組對應項。 在 `moduleContent` 區段底部，於 `$edgeHub` 模組對應項後面插入下列 JSON 內容： 

    ```json
        "CModule": {
            "properties.desired":{
                "TemperatureThreshold":25
            }
        }
    ```

   ![將 CModule 對應項新增至部署範本](./media/tutorial-c-module/module-twin.png)

4. 儲存 **deployment.template.json** 檔案。
5. 在 VS Code 總管中，以滑鼠右鍵按一下 **deployment.template.json** 檔案，然後選取 [建置並推送 IoT Edge 解決方案]。 

當您指示 Visual Studio Code 建置解決方案時，它會先在新的 **config** 資料夾中產生 `deployment.json` 檔案。 deployment.json 檔案的資訊收集自您已更新的範本檔案、您用來儲存容器登錄認證的 .env 檔案，以及 CModule 資料夾中的 module.json 檔案。 

然後，Visual Studio Code 會在整合式終端機中執行兩個命令：`docker build` 和 `docker push`。 這兩個命令會建置程式碼、將 `CModule.dll` 容器化，再將程式碼推送至您在初始化解決方案時所指定的容器登錄。 

您可以在 VS Code 整合式終端機中檢視完整容器映像位址。 系統會根據 `module.json` 檔案中的資訊建置映像位址，其格式如下：**\<存放庫\>:\<版本\>-\<平台\>**。 在本教學課程中，它應會顯示為 **myregistry.azurecr.io/cmodule:0.0.1-amd64**。

## <a name="deploy-and-run-the-solution"></a>部署並執行解決方案

在您用來設定 IoT Edge 裝置的快速入門文章中，您使用 Azure 入口網站部署了模組。 您可以使用 Visual Studio Code 的 Azure IoT 工具組擴充功能來部署模組。 您已備妥您的案例所需的部署資訊清單，即 **deployment.json** 檔案。 現在您只需選取要接收部署的裝置即可。

1. 在 VS Code 命令選擇區中，執行 [Azure IoT 中樞：選取 IoT 中樞]。 

2. 選擇您要設定的 IoT Edge 裝置所屬的訂用帳戶和 IoT 中樞。 

3. 在 VS Code 總管中，展開 [Azure IoT 中樞裝置] 區段。 

4. 以滑鼠右鍵按一下 IoT Edge 裝置的名稱，然後選取 [建立單一裝置的部署]。 

   ![建立單一裝置的部署](./media/tutorial-c-module/create-deployment.png)

5. 選取 **config** 資料夾中的 **deployment.json** 檔案，然後按一下 [選取 Edge 部署資訊清單]。 請勿使用 deployment.template.json 檔案。 

6. 按一下 [重新整理] 按鈕。 您應該會看到新的 **CModule** 正在與 **TempSensor** 模組以及 **$edgeAgent** 和 **$edgeHub** 一起執行。 

## <a name="view-generated-data"></a>檢視產生的資料

在您將部署資訊清單套用至 IoT Edge 裝置後，裝置的 IoT Edge 執行階段即會收集新的部署資訊，並開始在裝置上執行。 裝置上任何執行中、但未包含在部署資訊清單中的模組都會停止。 裝置中遺漏的任何模組都會啟動。 

您可以使用 Visual Studio Code 總管的 [Azure IoT 中樞裝置] 區段，檢視 IoT Edge 裝置的狀態。 請展開裝置的詳細資料，以查看已部署且執行中的模組清單。 

在 IoT Edge 裝置本身之中，您可以使用 `iotedge list` 命令查看部署模組的狀態。 您應該會看到四個模組：兩個 IoT Edge 執行階段模組、tempSensor 和您在本教學課程中建立的自訂模組。 啟動所有模組可能需要幾分鐘的時間，因此若您一開始未看到所有模組，請重新執行命令。 

若要檢視任何模組所產生的訊息，請使用 `iotedge logs <module name>` 命令。 

您可以在訊息送達 IoT 中樞時使用 Visual Studio Code 加以檢視。 

1. 若要監視送達 IoT 中樞的資料，請按一下 [...]，然後選取 [開始監視 D2C 訊息]。
2. 若要監視特定裝置的 D2C 訊息，請以滑鼠右鍵按一下清單中的裝置，然後選取 [開始監視 D2C 訊息]。
3. 若要停止監視資料，請在命令選擇區中執行命令 **Azure IoT Hub: Stop monitoring D2C message**。 
4. 若要檢視或更新模組對應項，請以滑鼠右鍵按一下清單中的模組，然後選取 [編輯模組對應項]。 若要更新模組對應項，請儲存對應項 JSON 檔案，並以滑鼠右鍵按一下編輯器區域，然後選取 [更新模組對應項]。
5. 若要檢視 Docker 記錄，您可以安裝適用於 VS Code 的 [Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)，並在 Docker 總管中於本機尋找執行中的模組。 在快顯功能表中，按一下 [顯示記錄] 以在整合式終端機中進行檢視。
 
## <a name="clean-up-resources"></a>清除資源 

如果您打算繼續閱讀下一篇建議的文章，則可以保留您所建立的資源和組態，並加以重複使用。 您可以也繼續使用相同的 IoT Edge 裝置作為測試裝置。 

否則，可以刪除您在本文中建立的本機組態和 Azure 資源，以避免產生費用。 

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

[!INCLUDE [iot-edge-clean-up-local-resources](../../includes/iot-edge-clean-up-local-resources.md)]


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立包含程式碼的 IoT Edge 模組，可篩選您 IoT Edge 裝置所產生的原始資料。 當您準備好要建置自己的模組時，可以進一步了解如何[使用適用於 Visual Studio Code 的 Azure IoT Edge 開發 C 模組](how-to-develop-c-module.md)。 您可以繼續進行後續教學課程，以了解 Azure IoT Edge 有什麼其他方法可協助您將此資料轉換成 Edge 上的商業見解。

> [!div class="nextstepaction"]
> [使用 SQL Server 資料庫在Edge 上儲存資料](tutorial-store-data-sql-server.md)

<!-- Links -->
[lnk-tutorial1-win]: quickstart.md
[lnk-tutorial1-lin]: quickstart-linux.md
