---
title: 開始使用 Azure IoT 中樞裝置管理 (.NET/.NET) | Microsoft Docs
description: 如何使用 Azure IoT 中樞裝置管理來起始遠端裝置重新開機。 您可以使用適用於 .NET 的 Azure IoT 裝置 SDK，實作模擬裝置應用程式 (包含直接方法)，也可以使用適用於 .NET 的 Azure IoT 服務 SDK，實作服務應用程式 (叫用直接方法)。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 09/15/2017
ms.author: dobett
ms.openlocfilehash: bca25f5e7a6fd7685d20236033a5a225c5183a2d
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225339"
---
# <a name="get-started-with-device-management-netnet"></a>開始使用裝置管理 (.NET/.NET)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

本教學課程說明如何：

* 使用 Azure 入口網站來建立 IoT 中樞，並且在 IoT 中樞建立裝置識別。

* 建立模擬裝置應用程式，其包含可將該裝置重新開機的直接方法。 直接方法是從雲端叫用。

* 建立 .NET 主控台應用程式，可透過您的 IoT 中樞在模擬的裝置應用程式中呼叫重新啟動直接方法。

在本教學課程結尾，您會有兩個 .NET 主控台應用程式：

* **SimulateManagedDevice** - 此應用程式會以先前建立的裝置身分識別連線到您的 IoT 中樞、接收重新啟動直接方法、模擬實體重新啟動，然後回報上次重新啟動的時間。

* **TriggerReboot**，它會在模擬裝置應用程式中呼叫直接方法、顯示回應，以及顯示更新的報告屬性。

若要完成此教學課程，您需要下列項目：

* Visual Studio 2017。

* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。)

[!INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[!INCLUDE [iot-hub-get-started-create-device-identity-portal](../../includes/iot-hub-get-started-create-device-identity-portal.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>使用直接方法在裝置上觸發遠端重新啟動

在本節中，您會建立 .NET 主控台應用程式 (使用 C#)，此應用程式會使用直接方法起始遠端重新開機。 應用程式使用裝置對應項查詢來探索該裝置的上次重新開機時間。

1. 在 Visual Studio 中，使用 [主控台應用程式 (.NET Framework)] 專案範本，將 Visual C# Windows 傳統桌面專案新增至新的解決方案。 確定 .NET Framework 為 4.5.1 或更新版本。 將專案命名為 **TriggerReboot**。

    ![新的 Visual C# Windows 傳統桌面專案](./media/iot-hub-csharp-csharp-device-management-get-started/createserviceapp.png)

2. 在 [方案總管] 中，以滑鼠右鍵按一下 **TriggerReboot** 專案，然後按一下 [管理 NuGet 套件]。

3. 在 [NuGet 套件管理員] 視窗中選取 [瀏覽]、搜尋 **Microsoft.Azure.Devices**、選取 [安裝] 以安裝 **Microsoft.Azure.Devices** 套件，並接受使用條款。 此程序會下載及安裝 [Azure IoT 服務 SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices/) (英文) NuGet 套件與其相依項目，並新增對它的參考。

    ![NuGet 封裝管理員視窗](./media/iot-hub-csharp-csharp-device-management-get-started/servicesdknuget.png)

4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：
   
   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Shared;
   ```
        
5. 將下列欄位新增到 **Program** 類別。 將預留位置的值替換為您在＜建立 IoT 中樞＞一節中為中樞建立的 IoT 中樞連接字串。 
   
   ```csharp
   static RegistryManager registryManager;
   static string connString = "{iot hub connection string}";
   static ServiceClient client;
   static string targetDevice = "myDeviceId";
   ```

6. 將下列方法新增至 **Program** 類別。  此程式碼會取得用於重新啟動裝置的裝置對應項，並輸出報告屬性。
   
   ```csharp
   public static async Task QueryTwinRebootReported()
   {
       Twin twin = await registryManager.GetTwinAsync(targetDevice);
       Console.WriteLine(twin.Properties.Reported.ToJson());
   }
   ```
        
7. 將下列方法新增至 **Program** 類別。  此程式碼會使用直接方法在裝置上起始重新啟動。

   ```csharp
   public static async Task StartReboot()
   {
       client = ServiceClient.CreateFromConnectionString(connString);
       CloudToDeviceMethod method = new CloudToDeviceMethod("reboot");
       method.ResponseTimeout = TimeSpan.FromSeconds(30);

       CloudToDeviceMethodResult result = await 
         client.InvokeDeviceMethodAsync(targetDevice, method);

       Console.WriteLine("Invoked firmware update on device.");
   }
   ```

7. 最後，將下列幾行新增至 **Main** 方法：
   
   ```csharp
   registryManager = RegistryManager.CreateFromConnectionString(connString);
   StartReboot().Wait();
   QueryTwinRebootReported().Wait();
   Console.WriteLine("Press ENTER to exit.");
   Console.ReadLine();
   ```

8. 建置方案。

> [!NOTE]
> 本教學課程只針對裝置的回報屬性執行單一查詢。 在生產環境程式碼中，建議您進行輪詢以偵測回報屬性中的變更。

## <a name="create-a-simulated-device-app"></a>建立模擬裝置應用程式

您會在本節中執行下列作業：

* 建立 .NET 主控台應用程式，以回應雲端所呼叫的直接方法。

* 觸發模擬裝置重新啟動。

* 使用回報的屬性來啟用裝置對應項查詢，以識別裝置及其上次重新啟動時間。

1. 在 Visual Studio 中，使用 [主控台應用程式] 專案範本，將 Visual C# Windows 傳統桌面專案新增至目前的方案。 將專案命名為 **SimulateManagedDevice**。
   
    ![新的 Visual C# Windows 傳統裝置應用程式](./media/iot-hub-csharp-csharp-device-management-get-started/createdeviceapp.png)
    
2. 在 [方案總管] 中，於 [SimulateManagedDevice] 專案上按一下滑鼠右鍵，然後按一下 [管理 NuGet 套件]。

3. 在 [NuGet 套件管理員] 視窗中，選取 [瀏覽]，並搜尋 **Microsoft.Azure.Devices.Client**。 選取 [安裝] 來安裝 **Microsoft.Azure.Devices.Client** 套件，並接受使用規定。 此程序會下載及安裝 [Azure IoT 裝置 SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) NuGet 套件與其相依性，並新增對它的參考。
   
    ![NuGet 套件管理員視窗用戶端應用程式](./media/iot-hub-csharp-csharp-device-management-get-started/clientsdknuget.png)
    
4. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：
   
    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    ```

5. 將下列欄位新增到 **Program** 類別。 將預留位置的值取代為您在上一節中所記下的裝置連接字串。

    ```csharp
    static string DeviceConnectionString = 
      "HostName=<yourIotHubName>.azure-devices.net;DeviceId=<yourIotDeviceName>;SharedAccessKey=<yourIotDeviceAccessKey>";
    static DeviceClient Client = null;
    ```
6. 新增下列項目以在裝置上實作直接方法：

   ```csharp
   static Task<MethodResponse> onReboot(MethodRequest methodRequest, object userContext)
   {
       // In a production device, you would trigger a reboot 
       //   scheduled to start after this method returns.
       // For this sample, we simulate the reboot by writing to the console
       //   and updating the reported properties.
       try
       {
           Console.WriteLine("Rebooting!");

           // Update device twin with reboot time. 
           TwinCollection reportedProperties, reboot, lastReboot;
           lastReboot = new TwinCollection();
           reboot = new TwinCollection();
           reportedProperties = new TwinCollection();
           lastReboot["lastReboot"] = DateTime.Now;
           reboot["reboot"] = lastReboot;
           reportedProperties["iothubDM"] = reboot;
           Client.UpdateReportedPropertiesAsync(reportedProperties).Wait();
       }
       catch (Exception ex)
       {
           Console.WriteLine();
           Console.WriteLine("Error in sample: {0}", ex.Message);
       }

       string result = "'Reboot started.'";
       return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
   }
   ```

7. 最後，在 **Main** 方法中新增下列程式碼以開啟 IoT 中樞的連線，並啟動方法接聽程式︰

   ```csharp
   try
   {
       Console.WriteLine("Connecting to hub");
       Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
         TransportType.Mqtt);

       // setup callback for "reboot" method
       Client.SetMethodHandlerAsync("reboot", onReboot, null).Wait();
       Console.WriteLine("Waiting for reboot method\n Press enter to exit.");
       Console.ReadLine();

       Console.WriteLine("Exiting...");

       // as a good practice, remove the "reboot" handler
       Client.SetMethodHandlerAsync("reboot", null, null).Wait();
       Client.CloseAsync().Wait();
   }
   catch (Exception ex)
   {
       Console.WriteLine();
       Console.WriteLine("Error in sample: {0}", ex.Message);
   }
   ```
        
8. 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下您的方案，然後按一下 [設定啟始專案...]選取 [單一啟始專案]，然後選取下拉式功能表中的 [SimulateManagedDevice] 專案。 建置方案。       

> [!NOTE]
> 為了簡單起見，本教學課程不會實作任何重試原則。 在生產環境程式碼中，您應該如[暫時性錯誤處理](/azure/architecture/best-practices/transient-faults)一文中所建議，實作重試原則 (例如指數型輪詢)。

## <a name="run-the-apps"></a>執行應用程式

您現在可以開始執行應用程式。

1. 執行 .NET 裝置應用程式 **SimulateManagedDevice**。 在 [SimulateManagedDevice] 專案上按一下滑鼠右鍵，選取 [偵錯]，然後選取 [啟動新的執行個體]。 它應該會開始接聽來自 IoT 中樞的方法呼叫。 

2. 既然已連接裝置並正在等候方法引動過程，現在請執行 .NET **TriggerReboot** 應用程式，以在模擬裝置應用程式中叫用重新啟動方法。 若要完成這項操作，請以滑鼠右鍵按一下 **TriggerReboot** 專案，選取 [偵錯]，然後選取 [啟動新的執行個體]。 您應該會看到 "Rebooting!" 顯示在 **SimulatedManagedDevice** 主控台中，以及看到回報的裝置屬性顯示在 **TriggerReboot** 主控台中，其中包括上次重新啟動時間。
   
    ![服務和裝置應用程式執行](./media/iot-hub-csharp-csharp-device-management-get-started/combinedrun.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
