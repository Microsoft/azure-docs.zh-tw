---
title: 開始使用 Azure IoT 中樞裝置對應項 (Java) | Microsoft Docs
description: 如何使用 Azure IoT 中樞裝置對應項來新增標籤，然後使用 IoT 中樞查詢。 您可以使用適用於 Java 的 Azure IoT 裝置 SDK，實作裝置應用程式，也可以使用適用於 Java 的 Azure IoT 服務 SDK，實作服務應用程式，以新增標籤和執行 IoT 中樞查詢。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 07/04/2017
ms.author: dobett
ms.openlocfilehash: b8884cafbf250b9d7a88219b5647addafee9904a
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39186895"
---
# <a name="get-started-with-device-twins-java"></a>開始使用裝置對應項 (Java)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

在本教學課程中，您將建立兩個 Java 主控台應用程式：

* **add-tags-query**，可新增標籤和查詢裝置對應項的 Java 後端應用程式。
* **simulated-device**，Java 裝置應用程式，可連線到 IoT 中樞，並使用回報屬性報告其連線狀況。

> [!NOTE]
> [Azure IoT SDK](iot-hub-devguide-sdks.md) 一文提供可用來建置裝置和後端應用程式之 Azure IoT SDK 的相關資訊。

若要完成本教學課程，您需要：

* 最新的 [Java SE 開發套件 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [Maven 3](https://maven.apache.org/install.html)
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[!INCLUDE [iot-hub-get-started-create-device-identity-portal](../../includes/iot-hub-get-started-create-device-identity-portal.md)]

## <a name="create-the-service-app"></a>建立服務應用程式

在本節中，您要建立一個 Java 應用程式，將位置中繼資料當成標籤新增至 IoT 中樞中與 **myDeviceId** 建立關聯的裝置對應項。 應用程式會先查詢 IoT 中樞是否有位於美國的裝置，再查詢是否有回報行動電話通訊網路連線的裝置。

1. 在您的開發電腦上建立名為 `iot-java-twin-getstarted` 的空資料夾。

1. 在 `iot-java-twin-getstarted` 資料夾中，在命令提示字元中使用下列命令，建立名為 **add-tags-query** 的 Maven 專案。 注意，這是一個單一且非常長的命令：

    `mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=add-tags-query -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`

1. 在命令提示字元中，巡覽至 `add-tags-query` 資料夾。

1. 使用文字編輯器，開啟 `add-tags-query` 資料夾中的 `pom.xml` 檔案，並在 [相依性] 節點中新增下列相依性。 這個相依性可讓您在應用程式中使用 **iot-service-client** 套件與 IoT 中樞通訊：

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
      <type>jar</type>
    </dependency>
    ```

    > [!NOTE]
    > 您可以使用 [Maven 搜尋](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22)來檢查最新版的 **iot-service-client**。

1. 將下列 [建置] 節點新增至 [相依性] 節點之後。 此設定會指示 Maven 使用 Java 1.8 來建置應用程式：

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

1. 儲存並關閉 `pom.xml` 檔案。

1. 使用文字編輯器開啟 `add-tags-query\src\main\java\com\mycompany\app\App.java` 檔案。

1. 在此檔案中新增下列 **import** 陳述式：

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.*;
    import com.microsoft.azure.sdk.iot.service.exceptions.IotHubException;

    import java.io.IOException;
    import java.util.HashSet;
    import java.util.Set;
    ```

1. 將下列類別層級變數新增到 **App** 類別中。 以您在＜建立 IoT 中樞＞一節中所記下的 IoT 中樞連接字串取代 `{youriothubconnectionstring}`：

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    public static final String region = "US";
    public static final String plant = "Redmond43";
    ```

1. 更新 **Main** 方法簽章，以包含下列 `throws` 子句：

    ```java
    public static void main( String[] args ) throws IOException
    ```

1. 將下列程式碼新增至 **Main** 方法，建立 **DeviceTwin** 和 **DeviceTwinDevice** 物件。 **DeviceTwin** 物件會處理與 IoT 中樞的通訊。 **DeviceTwinDevice** 物件會以其屬性和標籤代表裝置對應項：

    ```java
    // Get the DeviceTwin and DeviceTwinDevice objects
    DeviceTwin twinClient = DeviceTwin.createFromConnectionString(iotHubConnectionString);
    DeviceTwinDevice device = new DeviceTwinDevice(deviceId);
    ```

1. 將以下 `try/catch` 區塊新增至 **Main** 方法：

    ```java
    try {
      // Code goes here
    } catch (IotHubException e) {
      System.out.println(e.getMessage());
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
    ```

1. 若要更新裝置對應項的**區域**和**工廠**裝置對應項標籤，請在 `try` 區塊中新增下列程式碼：

    ```java
    // Get the device twin from IoT Hub
    System.out.println("Device twin before update:");
    twinClient.getTwin(device);
    System.out.println(device);

    // Update device twin tags if they are different
    // from the existing values
    String currentTags = device.tagsToString();
    if ((!currentTags.contains("region=" + region) && !currentTags.contains("plant=" + plant))) {
      // Create the tags and attach them to the DeviceTwinDevice object
      Set<Pair> tags = new HashSet<Pair>();
      tags.add(new Pair("region", region));
      tags.add(new Pair("plant", plant));
      device.setTags(tags);

      // Update the device twin in IoT Hub
      System.out.println("Updating device twin");
      twinClient.updateTwin(device);
    }

    // Retrieve the device twin with the tag values from IoT Hub
    System.out.println("Device twin after update:");
    twinClient.getTwin(device);
    System.out.println(device);
    ```

1. 若要在 IoT 中樞中查詢裝置對應項，請將下列程式碼新增至 `try` 區塊，此區塊在上個步驟所新增的程式碼後面。 程式碼執行兩個查詢。 每個查詢都會傳回上限為 100 的裝置：

    ```java
    // Query the device twins in IoT Hub
    System.out.println("Devices in Redmond:");

    // Construct the query
    SqlQuery sqlQuery = SqlQuery.createSqlQuery("*", SqlQuery.FromType.DEVICES, "tags.plant='Redmond43'", null);

    // Run the query, returning a maximum of 100 devices
    Query twinQuery = twinClient.queryTwin(sqlQuery.getQuery(), 100);
    while (twinClient.hasNextDeviceTwin(twinQuery)) {
      DeviceTwinDevice d = twinClient.getNextDeviceTwin(twinQuery);
      System.out.println(d.getDeviceId());
    }

    System.out.println("Devices in Redmond using a cellular network:");

    // Construct the query
    sqlQuery = SqlQuery.createSqlQuery("*", SqlQuery.FromType.DEVICES, "tags.plant='Redmond43' AND properties.reported.connectivityType = 'cellular'", null);

    // Run the query, returning a maximum of 100 devices
    twinQuery = twinClient.queryTwin(sqlQuery.getQuery(), 3);
    while (twinClient.hasNextDeviceTwin(twinQuery)) {
      DeviceTwinDevice d = twinClient.getNextDeviceTwin(twinQuery);
      System.out.println(d.getDeviceId());
    }
    ```

1. 儲存並關閉 `add-tags-query\src\main\java\com\mycompany\app\App.java` 檔案。

1. 建置 **add-tags-query** 應用程式，並更正所有錯誤。 在命令提示字元中，巡覽至 `add-tags-query` 資料夾，並執行下列命令：

    `mvn clean package -DskipTests`

## <a name="create-a-device-app"></a>建立裝置應用程式

在本節中，您要建立 Java 主控台應用程式，以設定傳送到 IoT 中樞的回報屬性值。

1. 在 `iot-java-twin-getstarted` 資料夾中，在命令提示字元中使用下列命令，建立名為 **simulated-device** 的 Maven 專案。 注意，這是一個單一且非常長的命令：

    `mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`

1. 在命令提示字元中，巡覽至 `simulated-device` 資料夾。

1. 使用文字編輯器，開啟 `simulated-device` 資料夾中的 `pom.xml` 檔案，並在 [相依性] 節點中新增下列相依性。 這個相依性可讓您在應用程式中使用 **iot-device-client** 套件與 IoT 中樞通訊：

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-device-client</artifactId>
      <version>1.3.32</version>
    </dependency>
    ```

    > [!NOTE]
    > 您可以使用 [Maven 搜尋](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22)來檢查最新版的 **iot-device-client**。

1. 將下列 [建置] 節點新增至 [相依性] 節點之後。 此設定會指示 Maven 使用 Java 1.8 來建置應用程式：

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

1. 儲存並關閉 `pom.xml` 檔案。

1. 使用文字編輯器開啟 `simulated-device\src\main\java\com\mycompany\app\App.java` 檔案。

1. 在此檔案中新增下列 **import** 陳述式：

    ```java
    import com.microsoft.azure.sdk.iot.device.*;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.*;

    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.Scanner;
    ```

1. 將下列類別層級變數新增到 **App** 類別中。 以您的 IoT 中樞名稱取代 `{youriothubname}`，並以您在＜建立裝置身分識別＞一節中產生的裝置金鑰值取代 `{yourdevicekey}`：

    ```java
    private static String connString = "HostName={youriothubname}.azure-devices.net;DeviceId=myDeviceID;SharedAccessKey={yourdevicekey}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static String deviceId = "myDeviceId";
    ```

    此範例應用程式在具現化 **DeviceClient** 物件時使用 **protocol** 變數。 

1. 將以下程式碼新增至 **Main** 方法，以：
    * 建立與 IoT 中樞通訊的裝置用戶端。
    * 建立**裝置**物件以儲存裝置對應項屬性。

    ```java
    DeviceClient client = new DeviceClient(connString, protocol);

    // Create a Device object to store the device twin properties
    Device dataCollector = new Device() {
      // Print details when a property value changes
      @Override
      public void PropertyCall(String propertyKey, Object propertyValue, Object context) {
        System.out.println(propertyKey + " changed to " + propertyValue);
      }
    };
    ```

1. 將下列程式碼新增至 **Main** 方法，建立 **connectivityType** 回報屬性，並將它傳送到 IoT 中樞：

    ```java
    try {
      // Open the DeviceClient and start the device twin services.
      client.open();
      client.startDeviceTwin(new DeviceTwinStatusCallBack(), null, dataCollector, null);

      // Create a reported property and send it to your IoT hub.
      dataCollector.setReportedProp(new Property("connectivityType", "cellular"));
      client.sendReportedProperties(dataCollector.getReportedProp());
    }
    catch (Exception e) {
      System.out.println("On exception, shutting down \n" + " Cause: " + e.getCause() + " \n" + e.getMessage());
      dataCollector.clean();
      client.close();
      System.out.println("Shutting down...");
    }
    ```

1. 在 **Main** 方法的結尾新增下列程式碼。 等候 **Enter** 索引鍵給 IoT 中樞時間回報裝置對應項作業的狀態：

    ```java
    System.out.println("Press any key to exit...");

    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();

    dataCollector.clean();
    client.close();
    ```

1. 儲存並關閉 `simulated-device\src\main\java\com\mycompany\app\App.java` 檔案。

1. 建置 **simulated-device** 應用程式，並更正所有錯誤。 在命令提示字元中，巡覽至 `simulated-device` 資料夾，並執行下列命令：

    `mvn clean package -DskipTests`

## <a name="run-the-apps"></a>執行應用程式

您現在已經準備好執行主控台應用程式。

1. 在 `add-tags-query` 資料夾的命令提示字元中執行下列命令，以執行 **add-tags-query** 服務應用程式：

    `mvn exec:java -Dexec.mainClass="com.mycompany.app.App"`

    ![更新標籤值並執行裝置查詢的 Java IoT 中樞服務應用程式](media/iot-hub-java-java-twin-getstarted/service-app-1.png)

    您可以看到新增至裝置對應項的**工廠**和**區域**標籤。 第一個查詢會傳回您的裝置，但第二個則不然。

1. 在 `simulated-device` 資料夾的命令提示字元中，執行下列命令將 **connectivityType** 回報屬性新增至裝置對應項：

    `mvn exec:java -Dexec.mainClass="com.mycompany.app.App"`

    ![裝置用戶端會新增 **connectivityType** 回報屬性](media/iot-hub-java-java-twin-getstarted/device-app-1.png)

1. 在 `add-tags-query` 資料夾的命令提示字元中執行下列命令，以再次執行 **add-tags-query** 服務應用程式：

    `mvn exec:java -Dexec.mainClass="com.mycompany.app.App"`

    ![更新標籤值並執行裝置查詢的 Java IoT 中樞服務應用程式](media/iot-hub-java-java-twin-getstarted/service-app-2.png)

    現在您的裝置已將 **connectivityType** 屬性傳送到 IoT 中樞，第二個查詢會傳回您的裝置。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您在 Azure 入口網站中設定了新的 IoT 中樞，然後在 IoT 中樞的身分識別登錄中建立了裝置身分識別。 您已從後端應用程式將裝置中繼資料新增為標籤，並撰寫裝置應用程式來報告裝置對應項中的裝置連線資訊。 您也了解如何使用類似 SQL 的 IoT 中樞查詢語言來查詢裝置對應項資訊。

使用下列資源來了解如何：

* 利用[開始使用 IoT 中樞](quickstart-send-telemetry-java.md)教學課程，傳送裝置的遙測資料。
* 以互動方式控制裝置 (例如，從使用者控制的應用程式開啟風扇)，請參閱[使用直接方法](quickstart-control-device-java.md)教學課程。

<!-- Images. -->
[7]: ./media/iot-hub-java-java-twin-getstarted/invoke-method.png
[8]: ./media/iot-hub-java-java-twin-getstarted/device-listen.png
[9]: ./media/iot-hub-java-java-twin-getstarted/device-respond.png

<!-- Links -->
[lnk-hub-sdks]: iot-hub-devguide-sdks.md
