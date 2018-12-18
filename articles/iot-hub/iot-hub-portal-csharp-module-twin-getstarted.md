---
title: 開始使用 Azure IoT 中樞模組身分識別和模組對應項 (入口網站和 .NET) | Microsoft Docs
description: 了解如何使用入口網站和 .NET，建立模組身分識別及更新模組對應項。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: dobett
ms.openlocfilehash: d85845aa9bbf9b9f311adfc2588a4a8d0c670826
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39619846"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-the-portal-and-net-device"></a>透過入口網站和 .NET 裝置開始使用 IoT 中樞模組身分識別和模組對應項

> [!NOTE]
> [模組身分識別與模組對應項](iot-hub-devguide-module-twins.md)類似於 Azure IoT 中樞裝置身分識別與裝置對應項，但提供更精細的細微性。 雖然 Azure IoT 中樞裝置身分識別與裝置對應項可讓後端應用程式設定裝置並提供裝置狀況的可見性，但模組身分識別和模組對應項可提供裝置個別元件的上述功能。 在具備多個元件的可用裝置 (例如以作業系統為基礎的裝置或韌體裝置) 上，它允許每個元件有獨立的組態和狀況。

在本教學課程中，您將了解： 
1. 了解如何在入口網站中建立模組身分識別。 
2. 如何從您的裝置使用 .NET 裝置 SDK 更新模組對應項。

> [!NOTE]
> 如需可用來建置兩個應用程式，以在裝置與您的解決方案後端執行之 Azure IoT SDK 的相關資訊，請參閱 [Azure IoT SDK][lnk-hub-sdks]。

若要完成此教學課程，您需要下列項目：

* Visual Studio 2015 或 Visual Studio 2017。
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

## <a name="create-a-device-identity-in-the-portal"></a>在入口網站中建立裝置身分識別

您現在有自己的 IoT 中樞。 開啟[入口網站](https://portal.azure.com)，然後瀏覽至 IoT 中樞。 在 IoT 裝置上按一下，然後按一下 [新增] 以建立裝置身分識別。 將它命名為 **MyFirstDevice**。 

![建立裝置身分識別][8]

儲存之後，在裝置身分識別清單中，您可以看到已成功建立 MyFirstDevice 身分識別。

![已建立裝置識別碼][11]

現在按一下資料列。 您會看見裝置詳細資料。

![裝置詳細資料][10]

## <a name="create-a-module-identity-in-the-portal"></a>在入口網站中建立模組身分識別

在一個裝置身分識別內，您可以建立最多 20 個模組身分識別。 按一下上方的 [新增模組身分識別] 按鈕，建立名稱為 **myFirstModule** 的第一個模組身分識別。 

![裝置詳細資料][9]

儲存並按一下剛建立的模組身分識別。 您可以看到模組身分識別詳細資料。 儲存連接字串 - 主索引鍵。 在下一節中，它會使用於在裝置上設定您的模組。

![裝置詳細資料][12]

<a id="D2C_csharp"></a>
## <a name="update-the-module-twin-using-net-device-sdk"></a>使用 .NET 裝置 SDK 更新模組對應項

您已成功在 IoT 中樞中建立模組識別身分。 讓我們嘗試從您的模擬裝置與雲端通訊。 一旦建立模組身分識別，模組對應項會隱含地建立於 IoT 中樞中。 在這一節中，您會在模擬裝置上建立 .NET 主控台應用程式，以便更新模組對應項報告的屬性。

1. **建立 Visual Studio 專案** - 在 Visual Studio 中，使用 [主控台應用程式] \(.NET Framework\) 專案範本，將 Visual C# Windows 傳統桌面專案新增至現有的解決方案。 確定 .NET Framework 為 4.6.1 或更新版本。 將專案命名為 **UpdateModuleTwinReportedProperties**。

    ![建立 Visual Studio 專案][13]

2. **安裝最新的 Azure IoT 中樞 .NET 裝置 SDK** - 模組身分識別和模組對應項均處於公開預覽狀態。 它只能在 IoT 中樞發行前版本裝置 SDK 中取得。 在 Visual Studio 中，開啟[工具] > [Nuget 套件管理員] > [管理解決方案的 Nuget 套件]。 搜尋 Microsoft.Azure.Devices.Client。 確定您已核取 [包含發行前版本] 核取方塊。 選取最新版本並安裝。 您現在可以存取所有的模組功能。 

    ![安裝 Azure IoT Hub .NET 服務 SDK V1.16.0-preview-005][14]

3. **取得您的模組連接字串** - 如果您登入 [Azure 入口網站][lnk-portal]，請立即取得。 瀏覽至您的 IoT 中樞並按一下 IoT 裝置。 尋找 myFirstDevice 並加以開啟，您會看到已成功建立 myFirstModule。 複製模組連接字串。 在下一個步驟中需要用到它。

    ![Azure 入口網站模組詳細資料][15]

4. **建立 UpdateModuleTwinReportedProperties 主控台應用程式** 在 `using`Program.cs**檔的頂端，新增下列** 陳述式：

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    ```

    將下列欄位新增到 **Program** 類別。 使用模組連接字串取代預留位置值。

    ```csharp
    private const string ModuleConnectionString = "<Your module connection string>";
    private static ModuleClient Client = null;
    ```

    將下列方法 **OnDesiredPropertyChanged** 新增至 **Program** 類別：

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
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
        Microsoft.Azure.Devices.Client.TransportType transport = Microsoft.Azure.Devices.Client.TransportType.Amqp;

        try
        {
            Client = ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
            Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
            Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();

            Console.WriteLine("Retrieving twin");
            var twinTask = Client.GetTwinAsync();
            twinTask.Wait();
            var twin = twinTask.Result;
            Console.WriteLine(JsonConvert.SerializeObject(twin));

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
        Console.ReadKey();
        Client.CloseAsync().Wait();
    }
    
    private static void ConnectionStatusChangeHandler(ConnectionStatus status, ConnectionStatusChangeReason reason)
    {
        Console.WriteLine($"Status {status} changed: {reason}");
    }
    ```

    此程式碼範例顯示如何擷取模組對應項，並使用 AMQP 通訊協定更新報告的屬性。 在公開預覽中，我們只支援 AMQP 進行模組對應項作業。

## <a name="run-the-apps"></a>執行應用程式

您現在可以開始執行應用程式。 在 Visual Studio 的 [方案總管] 中以滑鼠右鍵按一下您的方案，然後按一下 [設定啟始專案]。 選取 [多個啟始專案]，然後選取 [啟動] 作為主控台應用程式的動作。 然後按 F5 來啟動執行這兩個應用程式。 

## <a name="next-steps"></a>後續步驟

若要繼續開始使用 IoT 中樞並瀏覽其他 IoT 案例，請參閱︰

* [透過 .NET 備份和 .NET 裝置開始使用 IoT 中樞模組身分識別和模組對應項][lnk-csharp-csharp-getstarted]
* [開始使用 IoT Edge][lnk-iot-edge]


<!-- Images. -->
[8]:./media\iot-hub-portal-csharp-module-twin-getstarted/create-device-id.JPG
[9]:./media\iot-hub-portal-csharp-module-twin-getstarted/create-module-id.JPG
[10]:./media\iot-hub-portal-csharp-module-twin-getstarted/device-details.JPG
[11]:./media\iot-hub-portal-csharp-module-twin-getstarted/device-id-created.JPG
[12]:./media\iot-hub-portal-csharp-module-twin-getstarted/module-details.JPG
[13]: ./media\iot-hub-csharp-csharp-module-twin-getstarted/update-twins-csharp1.JPG
[14]: ./media\iot-hub-csharp-csharp-module-twin-getstarted/install-sdk.png
[15]: ./media\iot-hub-csharp-csharp-module-twin-getstarted/module-detail.JPG
<!-- Links -->
[lnk-hub-sdks]: iot-hub-devguide-sdks.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-portal]: https://portal.azure.com/

[lnk-csharp-csharp-getstarted]: iot-hub-csharp-csharp-module-twin-getstarted.md
[lnk-iot-edge]: ../iot-edge/tutorial-simulate-device-linux.md
