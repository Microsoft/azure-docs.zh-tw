---
title: 使用 Azure IoT 中樞傳送雲端到裝置訊息 (.NET) | Microsoft Docs
description: 如何使用適用於 .NET 的 Azure IoT SDK，將雲端到裝置訊息從 Azure IoT 中樞傳送至裝置。 您可以修改裝置應用程式，以接收雲端到裝置訊息，也可以修改後端應用程式，以傳送雲端到裝置訊息。
author: fsautomata
manager: ''
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/24/2017
ms.author: elioda
ms.openlocfilehash: 2dca04cadd4c5b904c752ac8a01af8bcb7c6ae33
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220103"
---
# <a name="send-messages-from-the-cloud-to-your-device-with-iot-hub-net"></a>使用 IoT 中樞將訊息從雲端傳送至裝置 (.NET)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

## <a name="introduction"></a>簡介

Azure IoT 中樞是一項完全受控的服務，有助於讓數百萬個裝置和一個解決方案後端進行可靠且安全的雙向通訊。 [將遙測資料從裝置傳送到 IoT 中樞...](quickstart-send-telemetry-dotnet.md)說明了如何建立 IoT 中樞、在其中佈建裝置身分識別，以及編寫裝置應用程式，以傳送裝置到雲端的訊息。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

本教學課程是以快速入門[將遙測從裝置傳送到 IoT 中樞...](quickstart-send-telemetry-dotnet.md)為基礎而建置的。說明如何執行下列步驟：

* 從您的解決方案後端，透過 IoT 中樞將雲端到裝置訊息傳送給單一裝置。

* 接收裝置上的雲端到裝置訊息。

* 從您的解決方案後端，要求確認收到從 IoT 中樞傳送到裝置的訊息 (「意見反應」)。

您可以在[使用 IoT 中樞的 D2C 和 C2D 傳訊](iot-hub-devguide-messaging.md)中，找到有關雲端到裝置訊息的詳細資訊。

在本教學課程結尾處，您會執行兩個 .NET 主控台應用程式：

* **SimulatedDevice** 是在[將遙測資料從裝置傳送到 IoT 中樞...](quickstart-send-telemetry-dotnet.md)中建立的應用程式修改版本，可連線到您的 IoT 中樞，並接收雲端到裝置的訊息。

* **SendCloudToDevice**會透過 IoT 中樞，將雲端到裝置訊息傳送到裝置應用程式，然後接收其傳遞通知。

> [!NOTE]
> IoT 中樞會透過 [Azure IoT 裝置 SDK](iot-hub-devguide-sdks.md) 為許多裝置平台和語言 (包括 C、Java 及 Javascript) 提供 SDK 支援。 如需有關如何將您的裝置與本教學課程中的程式碼連接 (通常是連線至 Azure IoT 中樞) 的逐步指示，請參閱 [Azure IoT 中樞開發人員指南](iot-hub-devguide.md)。
> 

若要完成此教學課程，您需要下列項目：

* Visual Studio 2017

* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

## <a name="receive-messages-in-the-device-app"></a>在裝置應用程式中接收訊息

在本節中，您將修改在[將遙測資料從裝置傳送到 IoT 中樞...](quickstart-send-telemetry-dotnet.md)中建立的裝置應用程式，以接收來自 IoT 中樞的雲端到裝置訊息。

1. 在 Visual Studio 的 **SimulatedDevice** 專案中，將下列方法新增 [程式] 類別。

   ```csharp   
    private static async void ReceiveC2dAsync()
    {
        Console.WriteLine("\nReceiving cloud to device messages from service");
        while (true)
        {
            Message receivedMessage = await deviceClient.ReceiveAsync();
            if (receivedMessage == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received message: {0}", 
            Encoding.ASCII.GetString(receivedMessage.GetBytes()));
            Console.ResetColor();

            await deviceClient.CompleteAsync(receivedMessage);
        }
    }
   ```

   `ReceiveAsync` 方法會以非同步方式，在裝置收到訊息時，傳回收到的訊息。 它會在可指定的逾時期間過後傳回「null」  (在本例中，使用的是預設值 1 分鐘)。 當應用程式收到 *null* 時，應繼續等待新訊息。 這項要求是 `if (receivedMessage == null) continue` 行的的原因。
   
    對 `CompleteAsync()` 的呼叫會通知 IoT 中樞，說明已成功處理訊息。 可以安全地從裝置佇列中移除該訊息。 如果因故導致裝置應用程式無法完成訊息處理作業，「IoT 中樞」將會重新傳遞該訊息。 因此，裝置應用程式中的訊息處理邏輯必須是「等冪」，如此一來，多次接收相同的訊息才會產生相同的結果。 
    
    應用程式也可以暫時放棄訊息，這會使得「IoT 中樞」將訊息保留在佇列中以供未來取用。 或者，應用程式可以拒絕訊息，這會將訊息從佇列中永久移除。 如需有關雲端到裝置訊息生命週期的詳細資訊，請參閱[使用 IoT 中樞的 D2C 和 C2D 傳訊](iot-hub-devguide-messaging.md)。
   
   > [!NOTE]
   > 使用 HTTPS 而不使用 MQTT 或 AMQP 作為傳輸時，`ReceiveAsync` 方法會立即傳回。 使用 HTTPS 時，針對雲端到裝置訊息支援的模式是裝置以間歇方式連接而不常檢查訊息 (低於每 25 分鐘一次)。 發出更多 HTTPS 接收會導致「IoT 中樞」對要求進行節流。 如需有關 MQTT、AMQP 和 HTTPS 支援之間的差異，以及「IoT 中樞」節流的詳細資訊，請參閱[使用 IoT 中樞的 D2C 和 C2D 傳訊](iot-hub-devguide-messaging.md)。
   > 
   > 
2. 將下列方法新增到 **Main** 方法中緊接在 `Console.ReadLine()` 行前面：
   
   ``` csharp   
   ReceiveC2dAsync();
   ```

## <a name="send-a-cloud-to-device-message"></a>傳送雲端到裝置訊息
在本節中，您會撰寫 .NET 主控台應用程式，將雲端到裝置訊息傳送至裝置應用程式。

1. 在目前的 Visual Studio 方案中，使用 [主控台應用程式] 專案範本來建立「Visual C# 傳統型應用程式」專案。 將專案命名為 **SendCloudToDevice**。
   
   ![Visual Studio 中的新專案](./media/iot-hub-csharp-csharp-c2d/create-identity-csharp1.png)

2. 在 [方案總管] 中，以滑鼠右鍵按一下方案，然後按一下 [ **管理方案的 NuGet 封裝...**]。 
   
    此動作會開啟 [管理 NuGet 套件] 視窗。

3. 搜尋 **Microsoft.Azure.Devices**，按一下 [安裝]，然後接受使用規定。 
   
    這會下載和安裝 [Azure IoT 服務 SDK NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.Devices/)，並新增其參考。

4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：

   ``` csharp   
   using Microsoft.Azure.Devices;
   ```

5. 將下列欄位新增到 **Program** 類別。 使用來自[將遙測資料從裝置傳送到 IoT 中樞...](quickstart-send-telemetry-dotnet.md)的 IoT 中樞連接字串，取代預留位置值：

   ``` csharp
   static ServiceClient serviceClient;
   static string connectionString = "{iot hub connection string}";
   ```

6. 將下列方法加入至 **Program** 類別：
   
   ``` csharp
   private async static Task SendCloudToDeviceMessageAsync()
   {
        var commandMessage = new 
         Message(Encoding.ASCII.GetBytes("Cloud to device message."));
        await serviceClient.SendAsync("myFirstDevice", commandMessage);
   }
   ```

   這個方法會將新的雲端到裝置訊息傳送給識別碼為 `myFirstDevice`的裝置。 只有在您修改了在[將遙測資料從裝置傳送到 IoT 中樞...](quickstart-send-telemetry-dotnet.md)中使用的參數時，才要變更此參數。

7. 最後，將下列幾行新增至 **Main** 方法：

   ``` csharp
   Console.WriteLine("Send Cloud-to-Device message\n");
   serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
   
   Console.WriteLine("Press any key to send a C2D message.");
   Console.ReadLine();
   SendCloudToDeviceMessageAsync().Wait();
   Console.ReadLine();
   ```

8. 從 Visual Studio 中，在您的方案上按一下滑鼠右鍵，然後選取 [設定啟始專案...]。選取 [多個啟始專案]，然後同時針對 **ReadDeviceToCloudMessages**、**SimulatedDevice** 以及 **SendCloudToDevice** 選取 [啟動] 動作。

9. 按 **F5**。 三個應用程式應該全部都會啟動。 選取 [SendCloudToDevice] 視窗，然後按 **Enter**。 您應該會看到裝置應用程式正在接收訊息。
   
   ![正在接收訊息的應用程式](./media/iot-hub-csharp-csharp-c2d/sendc2d1.png)

## <a name="receive-delivery-feedback"></a>接收傳遞意見反應

您可以向「IoT 中樞」要求每個雲端到裝置訊息的傳遞 (或到期) 通知。 這個選項可讓解決方案後端輕鬆地通知重試或補償邏輯。 如需有關雲端到裝置意見反應的詳細資訊，請參閱[使用 IoT 中樞的 D2C 和 C2D 傳訊](iot-hub-devguide-messaging.md)。

在本節中，您會修改 **SendCloudToDevice** 應用程式以要求意見反應，然後從 IoT 中樞接收意見反應。

1. 在 Visual Studio 的 **SendCloudToDevice** 專案中，將下列方法加入至 **Program** 類別。

   ```csharp
   private async static void ReceiveFeedbackAsync()
   {
        var feedbackReceiver = serviceClient.GetFeedbackReceiver();
   
        Console.WriteLine("\nReceiving c2d feedback from service");
        while (true)
        {
            var feedbackBatch = await feedbackReceiver.ReceiveAsync();
            if (feedbackBatch == null) continue;
   
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received feedback: {0}", 
              string.Join(", ", feedbackBatch.Records.Select(f => f.StatusCode)));
            Console.ResetColor();
   
            await feedbackReceiver.CompleteAsync(feedbackBatch);
        }
    }
    ```

    請注意，此接收模式與用來從裝置應用程式接收雲端到裝置訊息的模式相同。

2. 緊接在 `serviceClient = ServiceClient.CreateFromConnectionString(connectionString)` 行後面，將下列方法新增到 **Main** 方法中：
   
   ``` csharp
   ReceiveFeedbackAsync();
   ```

3. 為了要求雲端到裝置訊息傳遞狀況的意見反應，您必須在 **SendCloudToDeviceMessageAsync** 方法中指定屬性。 將以下行加入 `var commandMessage = new Message(...);` 行之後：
   
   ``` csharp
   commandMessage.Ack = DeliveryAcknowledgement.Full;
   ```

4. 按 **F5**來執行應用程式。 您應該會看到三個應用程式全部都啟動。 選取 [SendCloudToDevice] 視窗，然後按 **Enter**。 您應該會看到裝置應用程式正在接收訊息，而幾秒之後，則會看到您的 **SendCloudToDevice** 應用程式正在接收意見反應訊息。
   
   ![正在接收訊息的應用程式](./media/iot-hub-csharp-csharp-c2d/sendc2d2.png)

> [!NOTE]
> 為了簡單起見，本教學課程不會實作任何重試原則。 在生產環境程式碼中，您應該如[暫時性錯誤處理](/azure/architecture/best-practices/transient-faults)一文所建議，實作重試原則 (例如指數型輪詢)。
> 

## <a name="next-steps"></a>後續步驟

在本操作說明中，您已了解如何傳送和接收雲端到裝置的訊息。 

若要查看使用 IoT 中樞的完整端對端解決方案範例，請參閱 [Azure IoT 遠端監視解決方案加速器](https://docs.microsoft.com/azure/iot-suite/)。

若要深入了解如何使用 IoT 中樞開發解決方案，請參閱 [IoT 中樞開發人員指南](iot-hub-devguide.md)。