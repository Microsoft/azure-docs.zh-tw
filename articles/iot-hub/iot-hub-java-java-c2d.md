---
title: 使用 Azure IoT 中樞傳送雲端到裝置訊息 (Java) | Microsoft Docs
description: 如何使用適用於 Java 的 Azure IoT SDK，將雲端到裝置訊息從 Azure IoT 中樞傳送至裝置。 您可以修改模擬裝置應用程式，以接收雲端到裝置訊息，也可以修改後端應用程式，以傳送雲端到裝置訊息。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 06/28/2017
ms.author: dobett
ms.openlocfilehash: e4d0df28449a2e50e72b192f0118a8eae3325d15
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220182"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-java"></a>使用 IoT 中樞傳送雲端到裝置訊息 (Java)
[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

Azure IoT 中樞是一項完全受控的服務，有助於讓數百萬個裝置和一個解決方案後端進行可靠且安全的雙向通訊。 [開始使用 IoT 中樞] 教學課程會示範如何建立 IoT 中樞、在其中佈建裝置識別，以及編寫模擬的裝置應用程式，以傳送裝置到雲端的訊息。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

本教學課程是以 [開始使用 IoT 中樞]為基礎。 這會說明如何：

* 從您的解決方案後端，透過 IoT 中樞將雲端到裝置訊息傳送給單一裝置。
* 接收裝置上的雲端到裝置訊息。
* 從您的解決方案後端，要求確認收到從 IoT 中樞傳送到裝置的訊息 (「意見反應」)。

您可以在 [IoT 中樞開發人員指南][IoT Hub developer guide - C2D]中，找到有關雲端到裝置訊息的詳細資訊。

在本教學課程結尾，您將執行兩個 Java 主控台應用程式：

* **simulated-device**：在 [開始使用 IoT 中樞]中建立的應用程式修改版本，可連接到您的「IoT 中樞」並接收雲端到裝置訊息。
* **send-c2d-messages**：會將雲端到裝置訊息透過「IoT 中樞」傳送到模擬的裝置應用程式，然後接收其傳遞通知。

> [!NOTE]
> 「IoT 中樞」透過 Azure IoT 裝置 SDK 為許多裝置平台和語言 (包括 C、Java 及 Javascript) 提供 SDK 支援。 如需有關如何將您的裝置與本教學課程中的程式碼連接 (通常是連接到「Azure IoT 中樞」) 的逐步指示，請參閱 [Azure IoT 開發人員中樞]。

若要完成此教學課程，您需要下列項目：

* [開始使用 IoT 中樞](quickstart-send-telemetry-java.md)或[處理 IoT 中樞裝置對雲端訊息](tutorial-routing.md)教學課程的完整運作版本。
* 最新的 [Java SE 開發套件 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [Maven 3](https://maven.apache.org/install.html)
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。)

## <a name="receive-messages-in-the-simulated-device-app"></a>在模擬的裝置應用程式中接收訊息

在本節中，您會修改在[開始使用 IoT 中樞]中建立的模擬裝置應用程式，以接收來自 IoT 中樞的雲端對裝置訊息。

1. 使用文字編輯器開啟 simulated-device\src\main\java\com\mycompany\app\App.java 檔案。

2. 在 **App** 類別內新增下列 **MessageCallback** 類別作為巢狀類別。 裝置從 IoT 中樞接收到訊息時會叫用 **execute** 方法。 在此範例中，裝置一律會通知 IoT 中樞其已完成訊息︰

    ```java
    private static class AppMessageCallback implements MessageCallback {
      public IotHubMessageResult execute(Message msg, Object context) {
        System.out.println("Received message from hub: "
          + new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET));
    
        return IotHubMessageResult.COMPLETE;
      }
    }
    ```
3. 修改 **main** 方法來建立 **AppMessageCallback** 執行個體，並在其開啟用戶端之前，先呼叫 **setMessageCallback** 方法，如以下所示︰

    ```java
    client = new DeviceClient(connString, protocol);
   
    MessageCallback callback = new AppMessageCallback();
    client.setMessageCallback(callback, null);
    client.open();
    ```

    > [!NOTE]
    > 如果您使用 HTTPS 而不是使用 MQTT 或 AMQP 作為傳輸，**DeviceClient** 執行個體將不會經常 (頻率低於每隔 25 分鐘) 檢查「IoT 中樞」是否有訊息。 如需有關 MQTT、AMQP 和 HTTPS 支援之間的差異以及「IoT 中樞」節流的詳細資訊，請參閱 [IoT 中樞開發人員指南][IoT Hub developer guide - C2D]。

4. 若要使用 Maven 建置 **simulated-device** 應用程式，請在命令提示字元中的 simulated-device 資料夾內執行下列命令：

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="send-a-cloud-to-device-message"></a>傳送雲端到裝置訊息

在本節中，您會建立 Java 主控台應用程式，以將雲端到裝置訊息傳送給模擬裝置應用程式。 您需要您在[開始使用 IoT 中樞]教學課程中所新增裝置的裝置識別碼。 您也需要中樞的 IoT 中樞連接字串 (可在 [Azure 入口網站]中找到)。

1. 在命令提示字元中使用下列命令，建立名為 **send-c2d-messages** 的 Maven 專案。 注意，此命令是單一且非常長的命令：

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=send-c2d-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示字元中，瀏覽到新的 send-c2d-messages 資料夾。

3. 使用文字編輯器，開啟 [send-c2d-messages] 資料夾中的 pom.xml 檔案，並將下列相依性新增到 [相依性] 節點。 新增相依性可讓您在應用程式中使用 **iothub-java-service-client** 套件與 IoT 中樞服務進行通訊：

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > 您可以使用 [Maven 搜尋][lnk-maven-service-search]來檢查最新版的 **iot-service-client**。

4. 儲存並關閉 pom.xml 檔案。

5. 使用文字編輯器開啟 send-c2d-messages\src\main\java\com\mycompany\app\App.java 檔案。

6. 在此檔案中新增下列 **import** 陳述式：

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. 將下列類別層級變數新增至 **App** 類別，其中以您稍早記下的值取代 **{yourhubconnectionstring}** 和 **{yourdeviceid}**：

    ```java
    private static final String connectionString = "{yourhubconnectionstring}";
    private static final String deviceId = "{yourdeviceid}";
    private static final IotHubServiceClientProtocol protocol = IotHubServiceClientProtocol.AMQPS;
    ```

8. 以下列程式碼取代 **main** 方法。 此程式碼會連線至 IoT 中樞，傳送訊息給您的裝置，然後等候裝置已接收並處理訊息的通知︰
   
    ```java
    public static void main(String[] args) throws IOException,
        URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(
        connectionString, protocol);
   
      if (serviceClient != null) {
        serviceClient.open();
        FeedbackReceiver feedbackReceiver = serviceClient
          .getFeedbackReceiver();
        if (feedbackReceiver != null) feedbackReceiver.open();
   
        Message messageToSend = new Message("Cloud to device message.");
        messageToSend.setDeliveryAcknowledgement(DeliveryAcknowledgement.Full);
   
        serviceClient.send(deviceId, messageToSend);
        System.out.println("Message sent to device");
   
        FeedbackBatch feedbackBatch = feedbackReceiver.receive(10000);
        if (feedbackBatch != null) {
          System.out.println("Message feedback received, feedback time: "
            + feedbackBatch.getEnqueuedTimeUtc().toString());
        }
   
        if (feedbackReceiver != null) feedbackReceiver.close();
        serviceClient.close();
      }
    }
    ```

    > [!NOTE]
    > 為了簡單起見，本教學課程不會實作任何重試原則。 在生產環境程式碼中，您應該如[暫時性錯誤處理](/azure/architecture/best-practices/transient-faults)文章所建議，實作重試原則 (例如指數型輪詢)。


9. 若要使用 Maven 建置 **simulated-device** 應用程式，請在命令提示字元中的 simulated-device 資料夾內執行下列命令：

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>執行應用程式

現在您已經準備好執行應用程式。

1. 在命令提示字元下，於 simulated-device 資料夾中執行下列命令，以開始將遙測傳送至 IoT 中樞，並接聽從中樞傳送的雲端到裝置訊息：

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    ```

    ![執行模擬裝置應用程式][img-simulated-device]

2. 在命令提示字元下，於 send-c2d-messages 資料夾中執行下列命令，以傳送雲端到裝置訊息並等候意見反應通知︰

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![執行命令以傳送雲端到裝置訊息][img-send-command]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何傳送和接收雲端到裝置的訊息。 

若要查看使用 IoT 中樞的完整端對端解決方案範例，請參閱 [Azure IoT 遠端監視解決方案加速器]。

若要深入了解如何使用 IoT 中樞開發解決方案，請參閱 [IoT 中樞開發人員指南]。

<!-- Images -->
[img-simulated-device]: media/iot-hub-java-java-c2d/receivec2d.png
[img-send-command]:  media/iot-hub-java-java-c2d/sendc2d.png
<!-- Links -->

[開始使用 IoT 中樞]: quickstart-send-telemetry-java.md
[IoT Hub developer guide - C2D]: iot-hub-devguide-messaging.md
[IoT 中樞開發人員指南]: iot-hub-devguide.md
[Azure IoT 開發人員中樞]: http://azure.microsoft.com/develop/iot
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdk-java
[Azure 入口網站]: https://portal.azure.com
[Azure IoT 遠端監視解決方案加速器]: https://azure.microsoft.com/documentation/suites/iot-suite/
[lnk-maven-service-search]: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22