---
title: 將裝置中的檔案上傳至使用 .NET 的 Azure IoT 中樞 | Microsoft Docs
description: 如何使用適用於 .NET 的 Azure IoT 裝置 SDK 將檔案從裝置上傳至雲端。 上傳的檔案會儲存在 Azure 儲存體 blob 容器中。
author: fsautomata
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 07/04/2017
ms.author: elioda
ms.openlocfilehash: c881ead472d07200bdf4284f30bcf097f0efcba4
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223793"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-using-net"></a>以使用 .NET 的 IoT 中樞將檔案從裝置上傳至雲端

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

本教學課程是以 [使用 IoT 中樞傳送雲端到裝置訊息](iot-hub-csharp-csharp-c2d.md) 教學課程中的程式碼來建置，示範如何使用 IoT 中樞的檔案上傳功能。 這會說明如何：

- 安全地將 Azure Blob URI 提供給裝置，以便上傳檔案。

- 您可以使用 IoT 中樞檔案上傳通知來觸發在您的應用程式後端中處理此檔案。

[將遙測資料從裝置傳送到 IoT 中樞](quickstart-send-telemetry-dotnet.md)和[使用 IoT 中樞傳送雲端到裝置的訊息](iot-hub-csharp-csharp-c2d.md)兩篇文章，會示範 IoT 中樞基本的裝置到雲端和雲端到裝置傳訊功能。 [使用 IoT 中樞設定訊息路由](tutorial-routing.md)教學課程說明了能在 Azure Blob 儲存體中，可靠地儲存裝置到雲端訊息的方法。 不過，在某些情況下，您無法輕易地將裝置傳送的資料對應到 IoT 中樞接受且相對較小的裝置到雲端訊息。 例如︰

* 包含映像的大型檔案
* 影片
* 取樣高頻率的震動資料
* 某種經前置處理過的資料

這些檔案通常會使用工具 (例如 [Azure Data Factory](../data-factory/introduction.md) 或 [Hadoop](../hdinsight/index.yml) 堆疊) 在雲端中進行批次處理。 當您需要從裝置上傳檔案時，您仍然可以使用安全可靠的 IoT 中樞。

在本教學課程結尾處，您會執行兩個 .NET 主控台應用程式：

* **SimulatedDevice**，此為[使用 IoT 中樞傳送雲端到裝置訊息](iot-hub-csharp-csharp-c2d.md)教學課程中建立的應用程式修改版本。 此應用程式可以使用 IoT 中樞提供的 SAS URI，將檔案上傳到儲存體。

* **ReadFileUploadNotification**，它會接收來自 IoT 中樞的檔案上傳通知。

> [!NOTE]
> IoT 中樞透過 Azure IoT 裝置 SDK 來支援許多裝置平台和語言 (包括 C、Java 及 Javascript)。 如需如何將您的裝置連接到 Azure IoT 中樞的逐步指示，請參閱 [Azure IoT 開發人員中心](http://azure.microsoft.com/develop/iot)。

若要完成此教學課程，您需要下列項目：

* Visual Studio 2017
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>從裝置應用程式上傳檔案

在本節中，您將修改在[使用 IoT 中樞傳送雲端到裝置訊息](iot-hub-csharp-csharp-c2d.md)中建立的裝置應用程式，以接收來自 IoT 中樞的雲端到裝置訊息。

1. 在 Visual Studio 中，以滑鼠右鍵按一下 **SimulatedDevice** 專案，按一下 [新增]，然後按一下 [現有項目]。 瀏覽至影像檔並將它包含在您的專案中。 本教學課程假設影像名稱為 `image.jpg`。

1. 以滑鼠右鍵按一下影像，然後按一下 [內容] 。 確定 [複製到輸出目錄] 是設為 [一律複製]。

    ![顯示要在其中更新 [複製到輸出目錄] 映像屬性的位置](./media/iot-hub-csharp-csharp-file-upload/image-properties.png)

1. 在 **Program.cs** 檔的頂端，新增下列陳述式：

    ```csharp
    using System.IO;
    ```

1. 將下列方法加入至 **Program** 類別：

    ```csharp
    private static async void SendToBlobAsync()
    {
        string fileName = "image.jpg";
        Console.WriteLine("Uploading file: {0}", fileName);
        var watch = System.Diagnostics.Stopwatch.StartNew();

        using (var sourceData = new FileStream(@"image.jpg", FileMode.Open))
        {
            await deviceClient.UploadToBlobAsync(fileName, sourceData);
        }

        watch.Stop();
        Console.WriteLine("Time to upload file: {0}ms\n", watch.ElapsedMilliseconds);
    }
    ```

    `UploadToBlobAsync` 方法會取得要上傳之檔案的檔案名稱與資料流來源，然後處理上傳至儲存體的工作。 主控台應用程式會顯示上傳檔案所花費的時間。

1. 將下列方法新增到 **Main** 方法中緊接在 `Console.ReadLine()` 行前面：

    ```csharp
    SendToBlobAsync();
    ```

> [!NOTE]
> 為了簡單起見，本教學課程不會實作任何重試原則。 在生產環境程式碼中，您應該如[暫時性錯誤處理](/azure/architecture/best-practices/transient-faults)文章所建議，實作重試原則 (例如指數型輪詢)。

## <a name="receive-a-file-upload-notification"></a>接收檔案上傳通知

在本節中，您將撰寫 .NET 主控台應用程式，以接收來自 IoT 中樞的檔案上傳通知訊息。

1. 在目前的 Visual Studio 方案中，使用 [主控台應用程式] 專案範本來建立 Visual C# Windows 專案。 將專案命名為 **ReadFileUploadNotification**。

    ![Visual Studio 中的新專案](./media/iot-hub-csharp-csharp-file-upload/file-upload-project-csharp1.png)

2. 在 [方案總管] 中，以滑鼠右鍵按一下 **ReadFileUploadNotification** 專案，然後按一下 [管理 NuGet 套件...]。

3. 在 [NuGet 套件管理員] 視窗中，搜尋 **Microsoft.Azure.Devices**，按一下 [安裝] 並接受使用規定。

    此動作會下載和安裝 [Azure IoT 服務 SDK NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.Devices/)，並在 **ReadFileUploadNotification** 專案中新增此套件的參考。

4. 在 **Program.cs** 檔的頂端，新增下列陳述式：

    ```csharp
    using Microsoft.Azure.Devices;
    ```

5. 將下列欄位新增到 **Program** 類別。 使用來自[將遙測資料從裝置傳送到 IoT 中樞](quickstart-send-telemetry-dotnet.md) 的 IoT 中樞連接字串，取代預留位置值：

    ```csharp
    static ServiceClient serviceClient;
    static string connectionString = "{iot hub connection string}";
    ```

6. 將下列方法加入至 **Program** 類別：

    ```csharp
    private async static void ReceiveFileUploadNotificationAsync()
    {
        var notificationReceiver = serviceClient.GetFileNotificationReceiver();

        Console.WriteLine("\nReceiving file upload notification from service");
        while (true)
        {
            var fileUploadNotification = await notificationReceiver.ReceiveAsync();
            if (fileUploadNotification == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received file upload notification: {0}", 
              string.Join(", ", fileUploadNotification.BlobName));
            Console.ResetColor();

            await notificationReceiver.CompleteAsync(fileUploadNotification);
        }
    }   
    ```

    請注意，此接收模式與用來從裝置應用程式接收雲端到裝置訊息的模式相同。

7. 最後，將下列幾行新增至 **Main** 方法：

    ```csharp
    Console.WriteLine("Receive file upload notifications\n");
    serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
    ReceiveFileUploadNotificationAsync();
    Console.WriteLine("Press Enter to exit\n");
    Console.ReadLine();
    ```

## <a name="run-the-applications"></a>執行應用程式

現在您已經準備好執行應用程式。

1. 在 Visual Studio 中，在您的方案上按一下滑鼠右鍵，然後選取 [設定啟始專案]。 選取 [多個啟始專案]，然後針對 **ReadFileUploadNotification** 和 **SimulatedDevice** 選取 [啟動]。

2. 按 **F5**。 這兩個應用程式應該都會啟動。 您應該會在其中一個主控台應用程式中看到上傳已完成，以及另一個主控台應用程式所收到的上傳通知訊息。 您可以使用 [Azure 入口網站](https://portal.azure.com/)或 Visual Studio 伺服器總管，在您的 Azure 儲存體帳戶中檢查上傳的檔案是否存在。

    ![顯示輸出畫面的螢幕擷取畫面](./media/iot-hub-csharp-csharp-file-upload/run-apps1.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已學到如何使用 IoT 中樞的檔案上傳功能來簡化從裝置上傳檔案。 您可以利用下列文章繼續探索 IoT 中樞功能和案例：

* [以程式設計方式建立 IoT 中樞](iot-hub-rm-template-powershell.md)
* [C SDK 簡介](iot-hub-device-sdk-c-intro.md)
* [Azure IoT SDK](iot-hub-devguide-sdks.md)

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置](../iot-edge/tutorial-simulate-device-linux.md)

