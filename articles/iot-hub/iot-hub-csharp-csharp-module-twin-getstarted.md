---
title: 開始使用 Azure IoT 中樞模組身分識別和模組對應項 (.NET) | Microsoft Docs
description: 了解如何使用適用於 .NET 的 SDK，建立模組身分識別及更新模組對應項。
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: menchi
ms.openlocfilehash: 7ff867bc29e81e47a4bf66173ce3056792f2f445
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43091627"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-net-back-end-and-net-device"></a>透過 .NET 後端和 .NET 裝置開始使用 IoT 中樞模組身分識別和模組對應項

> [!NOTE]
> [模組身分識別與模組對應項](iot-hub-devguide-module-twins.md)類似於 Azure IoT 中樞裝置身分識別與裝置對應項，但提供更精細的細微性。 雖然 Azure IoT 中樞裝置身分識別與裝置對應項可讓後端應用程式設定裝置並提供裝置狀況的可見性，但模組身分識別和模組對應項可提供裝置個別元件的上述功能。 在具備多個元件的可用裝置 (例如以作業系統為基礎的裝置或韌體裝置) 上，它允許每個元件有獨立的組態和狀況。

在本教學課程結尾，您會有兩個 .NET 主控台應用程式：

* **CreateIdentities**，這會建立裝置身分識別、模組身分識別和相關聯的安全性金鑰，以連線到您的裝置和模組用戶端。

* **UpdateModuleTwinReportedProperties**，這會將已更新的模組對應項報告的屬性傳送到您的 IoT 中樞。

> [!NOTE]
> 如需可用來建置兩個應用程式，以在裝置上與解決方案後端執行的 Azure IoT SDK 詳細資訊，請參閱 [Azure IoT SDK](iot-hub-devguide-sdks.md)。

若要完成此教學課程，您需要下列項目：

* Visual Studio 2017。

* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

您現在已經建立 IoT 中樞，因此您已具有完成本教學課程的其餘部分所需的主機名稱和 IoT 中樞連接字串。

[!INCLUDE [iot-hub-get-started-create-module-identity-csharp](../../includes/iot-hub-get-started-create-module-identity-csharp.md)]


## <a name="update-the-module-twin-using-net-device-sdk"></a>使用 .NET 裝置 SDK 更新模組對應項

在這一節中，您會在模擬裝置上建立 .NET 主控台應用程式，以便更新模組對應項報告的屬性。

1. **建立 Visual Studio 專案：** 在 Visual Studio 中，使用 [主控台應用程式 (.NET Framework)] 專案範本，將 Visual C# Windows 傳統桌面專案新增至現有的解決方案中。 確定 .NET Framework 為 4.6.1 或更新版本。 將專案命名為 **UpdateModuleTwinReportedProperties**。

    ![建立 Visual Studio 專案](./media/iot-hub-csharp-csharp-module-twin-getstarted/update-twins-csharp1.JPG)

2. **安裝最新的 Azure IoT 中樞 .NET 裝置 SDK**：模組身分識別和模組對應項均處於公開預覽狀態。 它只能在 IoT 中樞發行前版本裝置 SDK 中取得。 在 Visual Studio 中，開啟[工具] > [Nuget 套件管理員] > [管理解決方案的 Nuget 套件]。 搜尋 Microsoft.Azure.Devices.Client。 確定您已核取 [包含發行前版本] 核取方塊。 選取最新版本並安裝。 您現在可以存取所有的模組功能。 

    ![安裝 Azure IoT Hub .NET 服務 SDK V1.16.0-preview-005](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-sdk.png)

3. **取得您的模組連接字串** - 如果您登入 [Azure 入口網站](https://portal.azure.com/)，請立即取得。 瀏覽至您的 IoT 中樞並按一下 IoT 裝置。 尋找 myFirstDevice 並加以開啟，您會看到已成功建立 myFirstModule。 複製模組連接字串。 在下一個步驟中需要用到它。

    ![Azure 入口網站模組詳細資料](./media/iot-hub-csharp-csharp-module-twin-getstarted/module-detail.JPG)

4. **建立 UpdateModuleTwinReportedProperties 主控台應用程式**

    在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    ```

    將下列欄位新增到 **Program** 類別。 使用模組連接字串取代預留位置值。

    ```csharp
    private const string ModuleConnectionString = 
      "<Your module connection string>";
    private static ModuleClient Client = null;
    static void ConnectionStatusChangeHandler(ConnectionStatus status, 
      ConnectionStatusChangeReason reason)
    {
        Console.WriteLine("Connection Status Changed to {0}; the reason is {1}", 
          status, reason);
    }
    ```

    將下列方法 **OnDesiredPropertyChanged** 新增至 **Program** 類別：

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
        {
            Console.WriteLine("desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
            Console.WriteLine("Sending current time as reported property");
            TwinCollection reportedProperties = new TwinCollection
            {
                ["DateTimeLastDesiredPropertyChangeReceived"] = DateTime.Now
            };

            await Client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        }
    ```

    最後，將下列幾行新增至 **Main** 方法：

    ```csharp
    static void Main(string[] args)
    {
        Microsoft.Azure.Devices.Client.TransportType transport = 
          Microsoft.Azure.Devices.Client.TransportType.Amqp;

        try
        {
            Client = 
              ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
            Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
            Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();

            Console.WriteLine("Retrieving twin");
            var twinTask = Client.GetTwinAsync();
            twinTask.Wait();
            var twin = twinTask.Result;
            Console.WriteLine(JsonConvert.SerializeObject(twin.Properties)); 

            Console.WriteLine("Sending app start time as reported property");
            TwinCollection reportedProperties = new TwinCollection();
            reportedProperties["DateTimeLastAppLaunch"] = DateTime.Now;

            Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (AggregateException ex)
        {
            Console.WriteLine("Error in sample: {0}", ex);
        }

        Console.WriteLine("Waiting for Events.  Press enter to exit...");
        Console.ReadLine();
        Client.CloseAsync().Wait();
    }
    ```

    此程式碼範例顯示如何擷取模組對應項，並使用 AMQP 通訊協定更新報告的屬性。 在公開預覽中，我們只支援 AMQP 進行模組對應項作業。

5. 除了上述 **Main** 方法以外，您可以新增下列程式碼區塊，從您的模組將事件傳送到 IoT 中樞：

    ```csharp
    Byte[] bytes = new Byte[2];
    bytes[0] = 0;
    bytes[1] = 1;
    var sendEventsTask = Client.SendEventAsync(new Message(bytes));
    sendEventsTask.Wait();
    Console.WriteLine("Event sent to IoT Hub.");
    ```

## <a name="run-the-apps"></a>執行應用程式

您現在可以開始執行應用程式。 在 Visual Studio 的 [方案總管] 中以滑鼠右鍵按一下您的方案，然後按一下 [設定啟始專案]。 選取 [多個啟始專案]，然後選取 [啟動] 作為主控台應用程式的動作。 然後按 F5 啟動應用程式。 

## <a name="next-steps"></a>後續步驟

若要繼續開始使用 IoT 中樞並瀏覽其他 IoT 案例，請參閱︰

* [開始使用裝置管理](iot-hub-node-node-device-management-get-started.md)
* [開始使用 IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)