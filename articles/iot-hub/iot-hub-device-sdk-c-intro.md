---
title: 適用於 C 的 Azure IoT 裝置 SDK | Microsoft Docs
description: 開始使用適用於 C 的 Azure IoT 裝置 SDK，以及了解如何建立與 IoT 中樞通訊的裝置應用程式。
author: yzhong94
manager: arjmands
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: conceptual
ms.date: 08/25/2017
ms.author: yizhon
ms.openlocfilehash: db9c22acfba0f6f1781348b36a1d253a515cc063
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977261"
---
# <a name="azure-iot-device-sdk-for-c"></a>適用於 C 的 Azure IoT 裝置 SDK

**Azure IoT 裝置 SDK** 是一組程式庫，其設計目的是要簡化從 **Azure IoT 中樞**服務傳送訊息和接收訊息的程序。 有各種不同的 SDK，每個 SDK 都以特定的平台為目標，而本文將說明的是「Azure IoT 裝置 SDK (適用於 C)」 。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Azure IoT 裝置 SDK (適用於 C) 是以 ANSI C (C99) 撰寫，以獲得最大可攜性。 此功能讓程式庫很適合在多個平台和裝置上運作，尤其是在以將磁碟和記憶體使用量降至最低做為優先考量的情況下。

有許多平台已進行過 SDK 測試 (如需詳細資料，請參閱 [Azure IoT 認證裝置目錄](https://catalog.azureiotsuite.com/))。 雖然本文包含在 Windows 平台上執行之範例程式碼的逐步解說，但本文所述的程式碼在各種支援的平台上都完全相同。

下列影片將介紹 Azure IoT SDK for C 的概觀：

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-IoT-C-SDK-insights/Player]

本文介紹 Azure IoT 裝置 SDK (適用於 C) 的架構。文中會示範如何初始化裝置程式庫，將資料傳送到 IoT 中樞，以及從 IoT 中樞接收訊息。 本文中的資訊應足以讓您開始使用 SDK，但也提供了可取得程式庫其他相關資訊的指標。

## <a name="sdk-architecture"></a>SDK 架構

您可以尋找[**適用於 C 的 Azure IoT 裝置 SDK**](https://github.com/Azure/azure-iot-sdk-c) GitHub 儲存機制，然後在 [C API 參考資料](https://azure.github.io/azure-iot-sdk-c/index.html)中檢視 API 的詳細資料。

在此儲存機制的 **master** 分支中可找到最新版的程式庫：

  ![](media/iot-hub-device-sdk-c-intro/01-MasterBranch.PNG)

* SDK 的核心實作位於 **iothub\_client** 資料夾中，此資料夾包含 SDK 中最低 API 層級的實作：**IoTHubClient** 程式庫。 **IoTHubClient** 程式庫包含了將訊息傳送至 IoT 中樞，以及從 IoT 中樞接收訊息之原始傳訊的 API 實作。 使用此程式庫時，您要負責實作訊息序列化，但與 IoT 中樞通訊的其他細節則是由系統為您處理。
* **serializer** 資料夾包含 helper 函式和示範如何在資料傳送至 Azure IoT 中樞之前，使用用戶端程式庫將資料序列化的範例。 使用序列化程式並非必要，只是為了便利性而提供。 若要使用 **serializer** 程式庫，您需要定義一個模型，以指定要傳送至 IoT 中樞的資料以及您期望從 IoT 中樞收到的訊息。 定義此模型後，SDK 會提供您一個 API 介面，可讓您輕鬆地處理裝置到雲端的訊息和雲端到裝置的訊息，而不需操心序列化細節。 此程式庫需倚賴其他使用 MQTT 和 AMQP 等通訊協定來實作傳輸的開放原始碼程式庫。
* **IoTHubClient** 程式庫相依於其他開放原始碼程式庫：
  * [Azure C 共用公用程式](https://github.com/Azure/azure-c-shared-utility)程式庫，此程式庫提供數個 Azure 相關 C SDK 所需之基本工作 (例如字串、清單操作和 IO) 的常見功能。
  * [Azure uAMQP](https://github.com/Azure/azure-uamqp-c) 程式庫，這是針對資源條件約束裝置最佳化的 AMQP 用戶端端實作。
  * [Azure uMQTT](https://github.com/Azure/azure-umqtt-c) 程式庫，這是實作 MQTT 通訊協定並已針對資源條件約束裝置最佳化的一般用途程式庫。

查看程式碼範例可以較容易了解如何使用這些程式庫。 下列各節將為您逐步解說 SDK 中包含的幾個範例應用程式。 此逐步解說應可讓您輕鬆了解 SDK 架構層的各種功能以及 API 運作方式的簡介。

## <a name="before-you-run-the-samples"></a>在您執行範例之前

您必須先在您的 Azure 訂用帳戶上[建立一個 IoT 中樞服務執行個體](iot-hub-create-through-portal.md)，才能在適用於 C 的 Azure IoT 裝置 SDK 中執行範例。 然後完成下列工作：

* 準備您的開發環境
* 取得裝置認證。

### <a name="prepare-your-development-environment"></a>準備您的開發環境

套件是針對常見平台 (例如適用於 Windows 的 NuGet 或適用於 Debian 和 Ubuntu 的 apt_get) 所提供，且範例會在這些套件可用時加以使用。 在某些情況下，您需要為您的裝置或在裝置編譯 SDK。 如果您需要編譯 SDK，請參閱 GitHub 儲存機制中的[準備您的開發環境](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md)。

若要取得應用程式的程式碼範例，請從 GitHub 下載 SDK 的複本。 從 [GitHub 儲存機制](https://github.com/Azure/azure-iot-sdk-c)的 **master** 分支取得一份原始檔複本。


### <a name="obtain-the-device-credentials"></a>取得裝置認證

現在您已擁有原始程式碼範例，下一件事就是取得一組裝置認證。 若要讓裝置能夠存取 IoT 中樞，您必須先將該裝置新增至 IoT 中樞身分識別登錄。 當您新增裝置時，您會取得一組所需的裝置認證，以便裝置能夠連線到 IoT 中樞。 下一節所討論的應用程式範例預期這些認證的形式為**裝置連接字串**。

有幾個開放原始碼工具可協助您管理 IoT 中樞。

* 稱為[裝置總管](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer)的 Windows 應用程式。
* 名為 [Azure IoT Toolkit](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) 的跨平台 Visual Studio Code 延伸模組。
* 名為 [Azure CLI 的 IoT 擴充功能](https://github.com/Azure/azure-iot-cli-extension)的跨平台 Python CLI。

本教學課程使用圖形化「裝置總管」工具。 若您在 VS Code 中開發，您可以使用*適用於 VS Code 的 Azure IoT Toolkit 延伸模組*。 如果您偏好使用 CLI 工具，也可以使用 Azure CLI 2.0 的 IoT 擴充功能工具。

裝置總管工具會使用 Azure IoT 服務程式庫在 IoT 中樞上執行各種功能，包括新增裝置。 如果您使用裝置總管工具來新增裝置，您會得到裝置的連接字串。 您需要此連接字串才能執行應用程式範例。

如果您不熟悉此裝置總管工具，下列程序說明如何使用它來新增裝置和取得裝置連接字串。

若要安裝裝置總管工具，請參閱[如何對 IoT 中樞裝置使用裝置總管](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer)。

當您執行程式時，您會看到此介面：

  ![](media/iot-hub-device-sdk-c-intro/03-DeviceExplorer.PNG)

請在第一個欄位中輸入您的 [IoT 中樞連接字串]，然後按一下 [更新]。 此步驟可設定此工具，以便與 IoT 中樞通訊。 您可以在 [IoT 中樞服務] > [設定] > **[共用存取權責]** > **iothubowner** 下找到 [連接字串]。

設定 IoT 中樞連接字串後，請按一下 [管理] 索引標籤：

  ![](media/iot-hub-device-sdk-c-intro/04-ManagementTab.PNG)

此索引標籤可讓您管理在 IoT 中樞註冊的裝置。

按一下 [建立] 按鈕即可建立裝置。 將會顯示一個已預先填入一組金鑰 (主要和次要) 的對話方塊。 輸入 [裝置識別碼]，然後按一下 [建立]。

  ![](media/iot-hub-device-sdk-c-intro/05-CreateDevice.PNG)

建立裝置後，就會以所有註冊的裝置 (包括您剛才建立的裝置) 更新 [裝置] 清單。 如果您在新裝置上按一下滑鼠右鍵，您會看到此功能表︰

  ![](media/iot-hub-device-sdk-c-intro/06-RightClickDevice.PNG)

如果您選擇 [複製所選裝置的連接字串]，裝置連接字串就會複製到剪貼簿。 請保留一份裝置連接字串。 在執行後續各節中所述的範例應用程式時，您會需要它。

完成上述步驟後，您就可以開始執行一些程式碼。 大部分範例在主要原始程式檔頂端都有一個常數，此常數可讓您輸入連接字串。 例如，來自 **iothub\_client\_sample\_mqtt** 應用程式的對應行會如以下所示。

```c
static const char* connectionString = "[device connection string]";
```

## <a name="use-the-iothubclient-library"></a>使用 IoTHubClient 程式庫

在 [azure-iot-sdk-c](https://github.com/azure/azure-iot-sdk-c) 儲存機制的 **iothub\_client** 資料夾中，有一個 [samples] 資料夾，當中包含名為 **iothub\_client\_sample\_mqtt** 的應用程式。

Windows 版本的 **iothub\_client\_sample\_mqtt** 應用程式包含下列 Visual Studio 方案：

  ![](media/iot-hub-device-sdk-c-intro/12-iothub-client-sample-mqtt.PNG)

> [!NOTE]
> 如果您在 Visual Studio 2017 開啟這個專案，請接受提示以將專案的目標重定為最新版本。

此解決方案內含單一專案： 此解決方案中安裝了四個 NuGet 套件：

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.umqtt

使用 SDK 時，您永遠需要 **Microsoft.Azure.C.SharedUtility** 封裝。 此範例使用 MQTT 通訊協定，因此您必須納入 **Microsoft.Azure.umqtt** 和 **Microsoft.Azure.IoTHub.MqttTransport** 套件 (有 AMQP 和 HTTPS 的對等套件)。 由於此範例使用 **IoTHubClient** 程式庫，因此您也必須在方案中納入 **Microsoft.Azure.IoTHub.IoTHubClient** 套件。

您可以在 **iothub\_client\_sample\_mqtt.c** 原始程式檔中找到範例應用程式的實作。

下列步驟使用此應用程式範例來逐步解說使用 **IoTHubClient** 程式庫時所需的項目。

### <a name="initialize-the-library"></a>初始化程式庫

> [!NOTE]
> 開始使用程式庫之前，您可能需要執行某些平台特定的初始化。 例如，如果您打算在 Linux 上使用 AMQP，您就必須初始化 OpenSSL 程式庫。 [GitHub 儲存機制](https://github.com/Azure/azure-iot-sdk-c)中的範例會在用戶端啟動時呼叫 **platform\_init** 公用程式函式，並在結束之前呼叫 **platform\_deinit** 函式。 這些函式在 platform.h 標頭檔中宣告。 請在[儲存機制](https://github.com/Azure/azure-iot-sdk-c)中為目標平台檢查這些函式的定義，以判斷是否需要在您的用戶端加入任何平台特定的初始化程式碼。

若要開始使用程式庫，請先配置 IoT 中樞用戶端控制代碼︰

```c
if ((iotHubClientHandle = IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)) == NULL)
{
    (void)printf("ERROR: iotHubClientHandle is NULL!\r\n");
}
else
{
    ...
```

您要將從裝置總管工具取得的裝置連接字串複本傳遞給此函式。 您也可以指定要使用的通訊協定。 此範例使用 MQTT，但 AMQP 與 HTTPS 也是選項。

當您具有有效的 **IOTHUB\_CLIENT\_HANDLE** 時，您可以開始呼叫 API 以對 IoT 中樞傳送和接收訊息。

### <a name="send-messages"></a>傳送訊息

應用程式範例會設定迴圈以將訊息傳送至 IoT 中樞。 下列程式碼片段︰

- 建立訊息。
- 將屬性新增至訊息。
- 傳送訊息。

首先，建立一則訊息：

```c
size_t iterator = 0;
do
{
    if (iterator < MESSAGE_COUNT)
    {
        sprintf_s(msgText, sizeof(msgText), "{\"deviceId\":\"myFirstDevice\",\"windSpeed\":%.2f}", avgWindSpeed + (rand() % 4 + 2));
        if ((messages[iterator].messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText))) == NULL)
        {
            (void)printf("ERROR: iotHubMessageHandle is NULL!\r\n");
        }
        else
        {
            messages[iterator].messageTrackingId = iterator;
            MAP_HANDLE propMap = IoTHubMessage_Properties(messages[iterator].messageHandle);
            (void)sprintf_s(propText, sizeof(propText), "PropMsg_%zu", iterator);
            if (Map_AddOrUpdate(propMap, "PropName", propText) != MAP_OK)
            {
                (void)printf("ERROR: Map_AddOrUpdate Failed!\r\n");
            }

            if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messages[iterator].messageHandle, SendConfirmationCallback, &messages[iterator]) != IOTHUB_CLIENT_OK)
            {
                (void)printf("ERROR: IoTHubClient_LL_SendEventAsync..........FAILED!\r\n");
            }
            else
            {
                (void)printf("IoTHubClient_LL_SendEventAsync accepted message [%d] for transmission to IoT Hub.\r\n", (int)iterator);
            }
        }
    }
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(1);

    iterator++;
} while (g_continueRunning);
```

每次傳送訊息時，您會指定傳送資料時所叫用的回呼函式的參考。 在此範例中，回呼函式稱為 **SendConfirmationCallback**。 下列程式碼片段會顯示這個回呼函式︰

```c
static void SendConfirmationCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    EVENT_INSTANCE* eventInstance = (EVENT_INSTANCE*)userContextCallback;
    (void)printf("Confirmation[%d] received for message tracking id = %zu with result = %s\r\n", callbackCounter, eventInstance->messageTrackingId, ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
    /* Some device specific action code goes here... */
    callbackCounter++;
    IoTHubMessage_Destroy(eventInstance->messageHandle);
}
```

請注意在您完成訊息時對 **IoTHubMessage\_Destroy** 函式進行的呼叫。 此函式會釋放在建立訊息時配置的資源。

### <a name="receive-messages"></a>接收訊息

接收訊息是非同步作業。 首先，您會登錄在裝置接收訊息時所要叫用的回呼：

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext) != IOTHUB_CLIENT_OK)
{
    (void)printf("ERROR: IoTHubClient_LL_SetMessageCallback..........FAILED!\r\n");
}
else
{
    (void)printf("IoTHubClient_LL_SetMessageCallback...successful.\r\n");
...
```

最後一個參數是您所要項目的無效指標。 在範例中，這是一個整數的指標，但可能是更複雜的資料結構的指標。 此參數可讓回呼函式與此函式的呼叫者以共用狀態運作。

當裝置接收訊息時，會叫用登錄的回呼函式。 此回呼函式會擷取︰

* 訊息識別碼和訊息中的相互關聯識別碼。
* 訊息內容。
* 訊息中的任何自訂屬性。

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    int* counter = (int*)userContextCallback;
    const char* buffer;
    size_t size;
    MAP_HANDLE mapProperties;
    const char* messageId;
    const char* correlationId;

    // Message properties
    if ((messageId = IoTHubMessage_GetMessageId(message)) == NULL)
    {
        messageId = "<null>";
    }

    if ((correlationId = IoTHubMessage_GetCorrelationId(message)) == NULL)
    {
        correlationId = "<null>";
    }

    // Message content
    if (IoTHubMessage_GetByteArray(message, (const unsigned char**)&buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        (void)printf("unable to retrieve the message data\r\n");
    }
    else
    {
        (void)printf("Received Message [%d]\r\n Message ID: %s\r\n Correlation ID: %s\r\n Data: <<<%.*s>>> & Size=%d\r\n", *counter, messageId, correlationId, (int)size, buffer, (int)size);
        // If we receive the work 'quit' then we stop running
        if (size == (strlen("quit") * sizeof(char)) && memcmp(buffer, "quit", size) == 0)
        {
            g_continueRunning = false;
        }
    }

    // Retrieve properties from the message
    mapProperties = IoTHubMessage_Properties(message);
    if (mapProperties != NULL)
    {
        const char*const* keys;
        const char*const* values;
        size_t propertyCount = 0;
        if (Map_GetInternals(mapProperties, &keys, &values, &propertyCount) == MAP_OK)
        {
            if (propertyCount > 0)
            {
                size_t index;

                printf(" Message Properties:\r\n");
                for (index = 0; index < propertyCount; index++)
                {
                    (void)printf("\tKey: %s Value: %s\r\n", keys[index], values[index]);
                }
                (void)printf("\r\n");
            }
        }
    }

    /* Some device specific action code goes here... */
    (*counter)++;
    return IOTHUBMESSAGE_ACCEPTED;
}
```

使用 **IoTHubMessage\_GetByteArray** 函式來擷取訊息 (在此範例中是一個字串)。

### <a name="uninitialize-the-library"></a>解除初始化程式庫

當您完成事件傳送和訊息接收時，您可以將 IoT 程式庫解除初始化。 若要這麼做，請發出下列函式呼叫：

```
IoTHubClient_LL_Destroy(iotHubClientHandle);
```

此呼叫會釋出 **IoTHubClient\_CreateFromConnectionString** 函式先前所配置的資源。

如您所見，使用 **IoTHubClient** 程式庫可以輕鬆傳送和接收訊息。 此程式庫會處理與 IoT 中樞進行的通訊細節，包括要使用哪個通訊協定 (從開發人員的觀點來看，這是一個簡單的設定選項)。

**IoTHubClient** 程式庫也可讓您精確地控制如何將裝置傳送到 IoT 中樞的資料序列化。 在某些情況下，此控制層級是一項優點，但在其他情況下，這是您不想要參與的實作細節。 如果是後者，您可以考慮使用下一節中說明的**序列化程式**程式庫。

## <a name="use-the-serializer-library"></a>使用序列化程式程式庫

在概念上，「序列化程式」程式庫位於 SDK 中的 **IoTHubClient** 程式庫之上。 它會使用 **IoTHubClient** 程式庫與 IoT 中樞進行基礎通訊，但是會新增模型化功能，以減輕開發人員處理訊息序列化的負擔。 此程式庫的運作方式最好是由範例示範。

在 [azure-iot-sdk-c 儲存機制](https://github.com/Azure/azure-iot-sdk-c)的 [serializer] 資料夾中，有一個 [samples] 資料夾，當中包含名為 **simplesample\_mqtt** 的應用程式。 Windows 版本的這個範例包含下列 Visual Studio 解決方案：

  ![](media/iot-hub-device-sdk-c-intro/14-simplesample_mqtt.PNG)

> [!NOTE]
> 如果您在 Visual Studio 2017 開啟這個專案，請接受提示以將專案的目標重定為最新版本。

如同先前的範例，此範例也包含數個 NuGet 套件：

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.IoTHub.Serializer
* Microsoft.Azure.umqtt

您已在先前的範例中看過上述大多數的套件，但 **Microsoft.Azure.IoTHub.Serializer** 是新的。 當您使用 **serializer** 程式庫時必須使用此套件。

您可以在 **simplesample\_mqtt.c** 檔案中找到範例應用程式的實作。

下列各節將逐步解說此範例的重要部分。

### <a name="initialize-the-library"></a>初始化程式庫

若要開始使用 **serializer** 程式庫，請呼叫初始化 API︰

```c
if (serializer_init(NULL) != SERIALIZER_OK)
{
    (void)printf("Failed on serializer_init\r\n");
}
else
{
    IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle = IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol);
    srand((unsigned int)time(NULL));
    int avgWindSpeed = 10;

    if (iotHubClientHandle == NULL)
    {
        (void)printf("Failed on IoTHubClient_LL_Create\r\n");
    }
    else
    {
        ContosoAnemometer* myWeather = CREATE_MODEL_INSTANCE(WeatherStation, ContosoAnemometer);
        if (myWeather == NULL)
        {
            (void)printf("Failed on CREATE_MODEL_INSTANCE\r\n");
        }
        else
        {
...
```

對 **serializer\_init** 函式的呼叫是單次呼叫，此呼叫可將基礎程式庫初始化。 接著，您需呼叫 **IoTHubClient\_LL\_CreateFromConnectionString** 函式，這與 **IoTHubClient** 範例中的 API 相同。 此呼叫會設定您的裝置連接字串 (此呼叫也可用來選擇您要使用的通訊協定)。 此範例使用 MQTT 作為傳輸，但也可以使用 AMQP 或 HTTPS。

最後，呼叫 **CREATE\_MODEL\_INSTANCE** 函式。 **WeatherStation** 是模型的命名空間，而 **ContosoAnemometer** 是模型的名稱。 建立模型執行個體後，您便可以使用它來開始傳送和接收訊息。 不過，請務必了解模型是什麼。

### <a name="define-the-model"></a>定義模型

**serializer** 程式庫中的模型會定義您的裝置可傳送至 IoT 中樞的訊息以及其可接收的訊息 (在模組化語言中稱為*動作*)。 您可使用如 **simplesample\_mqtt** 應用程式範例中的一組 C 巨集來定義模型︰

```c
BEGIN_NAMESPACE(WeatherStation);

DECLARE_MODEL(ContosoAnemometer,
WITH_DATA(ascii_char_ptr, DeviceId),
WITH_DATA(int, WindSpeed),
WITH_ACTION(TurnFanOn),
WITH_ACTION(TurnFanOff),
WITH_ACTION(SetAirResistance, int, Position)
);

END_NAMESPACE(WeatherStation);
```

**BEGIN\_NAMESPACE** 和 **END\_NAMESPACE** 這兩個巨集都會以模型的命名空間作為引數。 介於這兩個巨集之間的項目應該就是您的一或多個模型的定義和模型所使用的資料結構。

在此範例中，有一個名為 **ContosoAnemometer**的模型。 此模型會定義您裝置可傳送到 IoT 中樞的兩個資料片段︰**DeviceId** 和 **WindSpeed**。 它也定義您裝置可接收的三個動作 (訊息)：**TurnFanOn**、**TurnFanOff** 及 **SetAirResistance**。 每個資料元素都有類型，而每個動作都有名稱 (以及一組選擇性的參數)。

模型中定義的資料和動作可定義 API 介面，此介面可供您用來將訊息傳送到 IoT 中樞，以及回應傳送至裝置的訊息。 透過範例最能了解如何使用此模型。

### <a name="send-messages"></a>傳送訊息

此模型會定義您可以傳送到 IoT 中樞的資料。 在此範例中，這是指使用 **WITH_DATA** 巨集來定義的兩個資料項目之一。 要將 **DeviceId** 和 **WindSpeed** 值傳送至 IoT 中樞需要執行幾個步驟。 第一個步驟是設定您要傳送的資料：

```c
myWeather->DeviceId = "myFirstDevice";
myWeather->WindSpeed = avgWindSpeed + (rand() % 4 + 2);
```

您稍早定義的模型可讓您藉由設定 **struct** 的成員來設定值。 接下來，將您需要傳送的訊息序列化︰

```c
unsigned char* destination;
size_t destinationSize;
if (SERIALIZE(&destination, &destinationSize, myWeather->DeviceId, myWeather->WindSpeed) != CODEFIRST_OK)
{
    (void)printf("Failed to serialize\r\n");
}
else
{
    sendMessage(iotHubClientHandle, destination, destinationSize);
    free(destination);
}
```

此程式碼會將裝置到雲端序列化至緩衝區 (由 **destination** 參考)。 此程式碼接著會叫用 **sendMessage** 函式以將訊息傳送至 IoT 中樞︰

```c
static void sendMessage(IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
{
    static unsigned int messageTrackingId;
    IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
    if (messageHandle == NULL)
    {
        printf("unable to create a new IoTHubMessage\r\n");
    }
    else
    {
        if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)(uintptr_t)messageTrackingId) != IOTHUB_CLIENT_OK)
        {
            printf("failed to hand over the message to IoTHubClient");
        }
        else
        {
            printf("IoTHubClient accepted the message for delivery\r\n");
        }
        IoTHubMessage_Destroy(messageHandle);
    }
    messageTrackingId++;
}
```


**IoTHubClient\_LL\_SendEventAsync** 的倒數第二個參數是對成功傳送資料時所呼叫之回呼函式的參考。 以下是範例中的回呼函式︰

```c
void sendCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    unsigned int messageTrackingId = (unsigned int)(uintptr_t)userContextCallback;

    (void)printf("Message Id: %u Received.\r\n", messageTrackingId);

    (void)printf("Result Call Back Called! Result is: %s \r\n", ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
}
```

第二個參數是使用者內容的指標；即傳遞給 **IoTHubClient\_LL\_SendEventAsync** 的同一指標。 在此案例中，此內容是一個簡易計數器，但它可以是您想要的任何東西。

傳送裝置到雲端的訊息就是這麼簡單。 最後只剩下說明如何接收訊息。

### <a name="receive-messages"></a>接收訊息

接收訊息的方式類似於在 **IoTHubClient** 程式庫中使用訊息的方式。 首先，您需登錄訊息回呼函式：

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, IoTHubMessage, myWeather) != IOTHUB_CLIENT_OK)
{
    printf("unable to IoTHubClient_SetMessageCallback\r\n");
}
else
{
...
```

然後，撰寫在接收訊息時所叫用的回呼函式：

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT IoTHubMessage(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    IOTHUBMESSAGE_DISPOSITION_RESULT result;
    const unsigned char* buffer;
    size_t size;
    if (IoTHubMessage_GetByteArray(message, &buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        printf("unable to IoTHubMessage_GetByteArray\r\n");
        result = IOTHUBMESSAGE_ABANDONED;
    }
    else
    {
        /*buffer is not zero terminated*/
        char* temp = malloc(size + 1);
        if (temp == NULL)
        {
            printf("failed to malloc\r\n");
            result = IOTHUBMESSAGE_ABANDONED;
        }
        else
        {
            (void)memcpy(temp, buffer, size);
            temp[size] = '\0';
            EXECUTE_COMMAND_RESULT executeCommandResult = EXECUTE_COMMAND(userContextCallback, temp);
            result =
                (executeCommandResult == EXECUTE_COMMAND_ERROR) ? IOTHUBMESSAGE_ABANDONED :
                (executeCommandResult == EXECUTE_COMMAND_SUCCESS) ? IOTHUBMESSAGE_ACCEPTED :
                IOTHUBMESSAGE_REJECTED;
            free(temp);
        }
    }
    return result;
}
```

此程式碼會重複使用 - 對任何解決方案而言都一樣。 此函式會接收訊息，然後透過呼叫 **EXECUTE\_COMMAND** 來負責將它路由傳送至適當的函式。 此時所呼叫的函式取決於模型中的動作定義。

當您在模型中定義動作時，您必須實作在裝置接收對應的訊息時所呼叫的函式。 例如，如果您的模型定義這項動作：

```c
WITH_ACTION(SetAirResistance, int, Position)
```

使用此簽章定義函式：

```c
EXECUTE_COMMAND_RESULT SetAirResistance(ContosoAnemometer* device, int Position)
{
    (void)device;
    (void)printf("Setting Air Resistance Position to %d.\r\n", Position);
    return EXECUTE_COMMAND_SUCCESS;
}
```

注意函式的名稱如何與模型中的動作名稱相符，以及函式的參數如何與為此動作指定的參數相符。 第一個參數是必要參數，含有模型執行個體的指標。

當裝置收到符合此簽章的訊息時，就會呼叫對應的函式。 因此，除了必須包含 **IoTHubMessage**中重複使用的程式碼之外，接收訊息所涉及的只有為模型中定義的每個動作定義一個簡單的函式。

### <a name="uninitialize-the-library"></a>解除初始化程式庫

當您完成資料傳送和訊息接收時，您可以解除初始化 IoT 程式庫：

```c
...
        DESTROY_MODEL_INSTANCE(myWeather);
    }
    IoTHubClient_LL_Destroy(iotHubClientHandle);
}
serializer_deinit();
```

這三個函式每一個都與先前所述的三個初始化函式密切合作。 呼叫這些 API 可確保您釋放先前配置的資源。

## <a name="next-steps"></a>後續步驟

本文涵蓋使用「適用於 Azure IoT 裝置 SDK」中程式庫的基本概念。這提供您足夠的資訊來了解 SDK 中包含什麼、其架構，以及如何開始使用 Windows 範例。 下一篇文章藉由說明 [IoTHubClient 程式庫的相關資訊](iot-hub-device-sdk-c-iothubclient.md)來繼續說明 SDK。

若要深入了解如何開發 IoT 中樞，請參閱 [Azure IoT SDK][lnk-sdks]。

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置][lnk-iotedge]

[lnk-file upload]: iot-hub-csharp-csharp-file-upload.md
[lnk-create-hub]: iot-hub-rm-template-powershell.md
[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
