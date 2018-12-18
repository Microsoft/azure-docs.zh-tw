---
title: 開始使用 Azure IoT 中樞裝置對應項 (.NET/.NET) | Microsoft Docs
description: 如何使用 Azure IoT 中樞裝置對應項來新增標籤，然後使用 IoT 中樞查詢。 您可以使用適用於 .NET 的 Azure IoT 裝置 SDK，實作模擬的裝置應用程式，也可以使用適用於 .NET 的 Azure IoT 服務 SDK，實作服務應用程式，以新增標籤和執行 IoT 中樞查詢。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 05/15/2017
ms.author: dobett
ms.openlocfilehash: defb9f72148e319b36954a8d448d5f441621fc43
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49321575"
---
# <a name="get-started-with-device-twins-netnet"></a>開始使用裝置對應項 (.NET/.NET)
[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

在本教學課程結尾，您將會有下列 .NET 主控台應用程式：

* **CreateDeviceIdentity**，這是 .NET 應用程式，可建立裝置識別與相關聯的安全性金鑰，來連線到您的模擬裝置應用程式。

* **AddTagsAndQuery**，這是 .NET 後端應用程式，可新增標籤和查詢裝置對應項。

* **ReportConnectivity**，這是會模擬裝置的 .NET 裝置應用程式，此裝置會以稍早建立的裝置識別連線到您的 IoT 中樞，並報告其連線狀況。

> [!NOTE]
> [Azure IoT SDK](iot-hub-devguide-sdks.md) 一文提供可用來建置裝置和後端應用程式之 Azure IoT SDK 的相關資訊。
> 

若要完成此教學課程，您需要下列項目：

* Visual Studio 2017。
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[!INCLUDE [iot-hub-get-started-create-device-identity-portal](../../includes/iot-hub-get-started-create-device-identity-portal.md)]

## <a name="create-the-service-app"></a>建立服務應用程式

在本節中，您將 (使用 C#) 建立一個 .NET 主控台應用程式，此應用程式會將位置中繼資料新增至與 **myDeviceId** 相關聯的裝置對應項。 接著，它會選取位於美國的裝置來查詢儲存在 IoT 中樞的裝置對應項，再查詢會報告行動電話連線的對應項。

1. 在 Visual Studio 中，使用 [主控台應用程式] 專案範本，將 Visual C# Windows 傳統桌面專案新增至目前的方案。 將專案命名為 **AddTagsAndQuery**。
   
    ![新的 Visual C# Windows 傳統桌面專案](./media/iot-hub-csharp-csharp-twin-getstarted/createnetapp.png)

2. 在 [方案總管] 中，以滑鼠右鍵按一下 **AddTagsAndQuery** 專案，然後按一下 [管理 NuGet 套件...]。

3. 在 [NuGet 套件管理員] 視窗中，選取 [瀏覽]，並搜尋 **Microsoft.Azure.Devices**。 選取 [安裝] 來安裝 **Microsoft.Azure.Devices** 套件，並接受使用規定。 此程序會下載及安裝 [Azure IoT 服務 SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices/) (英文) NuGet 套件與其相依項目，並新增對它的參考。
   
    ![NuGet 封裝管理員視窗](./media/iot-hub-csharp-csharp-twin-getstarted/servicesdknuget.png)

4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：

    ```csharp  
    using Microsoft.Azure.Devices;
    ```

5. 將下列欄位新增到 **Program** 類別。 將預留位置的值替換為您在上一節中為中樞所建立的 IoT 中樞連接字串。

    ```csharp  
    static RegistryManager registryManager;
    static string connectionString = "{iot hub connection string}";
    ```

6. 將下列方法加入至 **Program** 類別：

    ```csharp  
    public static async Task AddTagsAndQuery()
    {
        var twin = await registryManager.GetTwinAsync("myDeviceId");
        var patch =
            @"{
                tags: {
                    location: {
                        region: 'US',
                        plant: 'Redmond43'
                    }
                }
            }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);
   
        var query = registryManager.CreateQuery(
          "SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
        var twinsInRedmond43 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43: {0}", 
          string.Join(", ", twinsInRedmond43.Select(t => t.DeviceId)));
   
        query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
        var twinsInRedmond43UsingCellular = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43 using cellular network: {0}", 
          string.Join(", ", twinsInRedmond43UsingCellular.Select(t => t.DeviceId)));
    }
    ```
   
    **RegistryManager** 類別會公開從服務來與裝置對應項進行互動時所需的所有方法。 先前的程式碼會先初始化 **registryManager** 物件，然後擷取 **myDeviceId** 的裝置對應項，最後會以所需的位置資訊來更新其標籤。
   
    在更新後，它會執行兩個查詢：第一個只選取位於 **Redmond43** 工廠的裝置對應項，第二個會修改查詢，只選取也透過行動電話網路來連接的裝置。
   
    請注意，先前的程式碼在建立 **查詢** 物件時，指定傳回的最大文件數。 **query** 物件包含 **HasMoreResults** 布林值屬性，可用來多次叫用 **GetNextAsTwinAsync** 方法以擷取所有結果。 有一個稱為 **GetNextAsJson** 的方法適用於不是裝置對應項的結果，例如彙總查詢的結果。

7. 最後，將下列幾行新增至 **Main** 方法：

    ```csharp  
    registryManager = RegistryManager.CreateFromConnectionString(connectionString);
    AddTagsAndQuery().Wait();
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

8. 在 [方案總管] 中，開啟 [設定起始專案...]，並確定 **AddTagsAndQuery** 專案的 [動作] 是 [啟動]。 建置方案。

9. 以滑鼠右鍵按一下 **AddTagsAndQuery** 專案，並選取 [偵錯]，後面接著 [開始新執行個體] 來執行此應用程式。 如果是查詢所有位於 **Redmond43** 中的裝置，您在結果中會看到一個裝置，而如果查詢將結果限於使用行動電話網路的裝置，則您不會看到任何裝置。
   
    ![視窗中的查詢結果](./media/iot-hub-csharp-csharp-twin-getstarted/addtagapp.png)

在下一節，您將建立一個裝置應用程式，以報告連線資訊並變更上一節的查詢結果。

## <a name="create-the-device-app"></a>建立裝置應用程式

在本節中，您將建立一個 .NET 主控台應用程式，此應用程式會以 **myDeviceId** 來連線到您的中樞，然後更新其回報屬性，以包含資訊來指出目前使用行動電話通訊網路來連線。

1. 在 Visual Studio 中，使用 [主控台應用程式] 專案範本，將 Visual C# Windows 傳統桌面專案新增至目前的方案。 將專案命名為 **ReportConnectivity**。
   
    ![新的 Visual C# Windows 傳統裝置應用程式](./media/iot-hub-csharp-csharp-twin-getstarted/createdeviceapp.png)
    
2. 在方案總管中，以滑鼠右鍵按一下 **ReportConnectivity** 專案，然後按一下 [管理 NuGet 套件...]。

3. 在 [NuGet 套件管理員] 視窗中，選取 [瀏覽]，並搜尋 **Microsoft.Azure.Devices.Client**。 選取 [安裝] 來安裝 **Microsoft.Azure.Devices.Client** 套件，並接受使用規定。 此程序會下載及安裝 [Azure IoT 裝置 SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) NuGet 套件與其相依性，並新增對它的參考。
   
    ![NuGet 套件管理員視窗用戶端應用程式](./media/iot-hub-csharp-csharp-twin-getstarted/clientsdknuget.png)

4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：

    ```csharp  
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

5. 將下列欄位新增到 **Program** 類別。 將預留位置的值取代為您在上一節中所記下的裝置連接字串。

    ```csharp  
    static string DeviceConnectionString = "HostName=<yourIotHubName>.azure-devices.net;
      DeviceId=<yourIotDeviceName>;SharedAccessKey=<yourIotDeviceAccessKey>";
    static DeviceClient Client = null;
    ```

6. 將下列方法加入至 **Program** 類別：

    ```csharp
    public static async void InitClient()
    {
        try
        {
            Console.WriteLine("Connecting to hub");
            Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
              TransportType.Mqtt);
            Console.WriteLine("Retrieving twin");
            await Client.GetTwinAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

    **Client** 物件會公開從裝置來與裝置對應項進行互動時所需的所有方法。 以上所示的程式碼會初始化**用戶端**物件，然後擷取 **myDeviceId** 的裝置對應項。

7. 將下列方法加入至 **Program** 類別：

    ```csharp  
    public static async void ReportConnectivity()
    {
        try
        {
            Console.WriteLine("Sending connectivity data as reported property");
            
            TwinCollection reportedProperties, connectivity;
            reportedProperties = new TwinCollection();
            connectivity = new TwinCollection();
            connectivity["type"] = "cellular";
            reportedProperties["connectivity"] = connectivity;
            await Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

   上述程式碼會將 **myDeviceId** 的回報屬性更新為連線資訊。

8. 最後，將下列幾行新增至 **Main** 方法：

    ```csharp
    try
    {
        InitClient();
        ReportConnectivity();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

9. 在方案總管中，開啟 [設定起始專案...]，並確定 **ReportConnectivity** 專案的 [動作] 是 [啟動]。 建置方案。

10. 以滑鼠右鍵按一下 **ReportConnectivity** 專案，並選取 [偵錯]，後面接著 [開始新執行個體] 來執行此應用程式。 您應該會看到它取得對應項資訊，然後傳送連線作為「回報屬性」。
   
    ![執行裝置應用程式以回報連線](./media/iot-hub-csharp-csharp-twin-getstarted/rundeviceapp.png)
       
11. 現在，裝置已回報其連線資訊，它應該會出現在這兩個查詢中。 執行 .NET **AddTagsAndQuery** 應用程式，以再次執行查詢。 這次，**myDeviceId** 應該會出現在這兩個查詢結果中。
   
    ![已成功回報裝置連線](./media/iot-hub-csharp-csharp-twin-getstarted/tagappsuccess.png)

## <a name="next-steps"></a>後續步驟

在此教學課程中，您在 Azure 入口網站中設定了新的 IoT 中樞，然後在 IoT 中樞的身分識別登錄中建立了裝置身分識別。 您已從後端應用程式將裝置中繼資料新增為標籤，並撰寫模擬裝置應用程式來報告裝置對應項中的裝置連線資訊。 您也了解如何使 類似 SQL 的 IoT 中樞查詢語言來查詢此資訊。

使用下列資源來了解如何：

* 使用[將遙測資料從裝置傳送到 IoT 中樞](quickstart-send-telemetry-dotnet.md)教學課程，從裝置傳送遙測，

* 使用裝置對應項所需的屬性來設定裝置，請參閱[使用所需的屬性來設定裝置](tutorial-device-twins.md)教學課程，

* 以互動方式控制裝置 (例如，從使用者控制的應用程式開啟風扇)，請參閱[使用直接方法](quickstart-control-device-dotnet.md)教學課程。
