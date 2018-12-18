---
title: 包含檔案
description: 包含檔案
services: iot-hub
author: chrissie926
ms.service: iot-hub
ms.topic: include
ms.date: 04/26/2018
ms.author: menchi
ms.custom: include file
ms.openlocfilehash: 603a4892cc19991532c2858edd8d4ca90f850d1f
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43086444"
---
## <a name="create-a-module-identity"></a>建立模組身分識別

在本節中，您會建立 .NET 主控台應用程式，它會在 IoT 中樞的身分識別登錄中建立裝置身分識別和模組身分識別。 裝置或模組無法連線到 IoT 中樞，除非它在身分識別登錄中具有項目。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南的＜身分識別登錄＞一節](../articles/iot-hub/iot-hub-devguide-identity-registry.md)。 當您執行此主控台應用程式時，它會針對裝置和模組產生唯一的識別碼和金鑰。 當裝置和模組將裝置到雲端的訊息傳送給 IoT 中樞時，裝置和模組會使用這些值來識別自己。 識別碼會區分大小寫。


1. **建立 Visual Studio 專案** - 在 Visual Studio 中，使用 [主控台應用程式] \(.NET Framework\) 專案範本，將 Visual C# Windows 傳統桌面專案新增至新的解決方案。 確定 .NET Framework 為 4.6.1 或更新版本。 將專案命名為 **CreateIdentities**，將解決方案命名為 **IoTHubGetStarted**。

    ![建立 Visual Studio 解決方案](./media/iot-hub-get-started-create-module-identity-csharp/create-identities-csharp1.JPG)

2. **安裝 Azure IoT 中樞 .NET 服務 SDK V1.16.0-preview-001** - 模組身分識別和模組對應項均處於公開預覽狀態。 它只能在 IoT 中樞發行前版本服務 SDK 中取得。 在 Visual Studio 中，開啟[工具] > [Nuget 套件管理員] > [管理解決方案的 Nuget 套件]。 搜尋 Microsoft.Azure.Devices。 確定您已核取 [包含發行前版本] 核取方塊。 選取版本 1.16.0-preview-001 並安裝。 您現在可以存取所有的模組功能。 

    ![安裝 Azure IoT Hub .NET 服務 SDK V1.16.0-preview-001](./media/iot-hub-get-started-create-module-identity-csharp/install-sdk.png)

3. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：

   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Common.Exceptions;
   ```

4. 將下列欄位新增到 **Program** 類別。 將預留位置的值替換為您在上一節中為中樞所建立的 IoT 中樞連接字串。

   ```csharp
   const string connectionString = "<replace_with_iothub_connection_string>";
   const string deviceID = "myFirstDevice";
   const string moduleID = "myFirstModule";
   ```

5. 將以下程式碼新增至 **Main** 類別。
   
   ```csharp
   static void Main(string[] args)
   {
       AddDeviceAsync().Wait();
       AddModuleAsync().Wait();
   }
   ```

6. 將下列方法新增至 **Program** 類別：

    ```csharp
    private static async Task AddDeviceAsync()
    {
       RegistryManager registryManager = 
         RegistryManager.CreateFromConnectionString(connectionString);
       Device device;

       try
       {
           device = await registryManager.AddDeviceAsync(new Device(deviceID));
       }
       catch (DeviceAlreadyExistsException)
        {
            device = await registryManager.GetDeviceAsync(deviceID);
        }

        Console.WriteLine("Generated device key: {0}", 
          device.Authentication.SymmetricKey.PrimaryKey);
    }

    private static async Task AddModuleAsync()
    {
        RegistryManager registryManager = 
          RegistryManager.CreateFromConnectionString(connectionString);
        Module module;

        try
        {
            module = 
              await registryManager.AddModuleAsync(new Module(deviceID, moduleID));
        }
        catch (ModuleAlreadyExistsException)
        {
            module = await registryManager.GetModuleAsync(deviceID, moduleID);
        }

        Console.WriteLine("Generated module key: {0}", module.Authentication.SymmetricKey.PrimaryKey);
    }
    ```

    AddDeviceAsync() 方法會建立具有識別碼 **myFirstDevice**的裝置識別。 (如果該裝置識別碼已經存在身分識別登錄中，程式碼就只會擷取現有的裝置資訊)。接著，應用程式會顯示該身分識別的主要金鑰。 您會在模擬裝置應用程式中使用此金鑰來連線到您的 IoT 中樞。

    AddModuleAsync() 方法會在裝置 **myFirstDevice** 之下建立具有識別碼 **myFirstDevice** 的模組身分識別。 (如果該模組識別碼已經存在身分識別登錄中，程式碼就只會擷取現有的模組資訊)。接著，應用程式會顯示該身分識別的主要金鑰。 您會在模擬模組應用程式中使用此金鑰來連線到您的 IoT 中樞。

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

7. 執行此應用程式，並記下裝置金鑰和模組金鑰。

> [!NOTE]
> IoT 中樞身分識別登錄只會儲存裝置和模組身分識別，以啟用對 IoT 中樞的安全存取。 身分識別登錄會儲存裝置識別碼和金鑰，以作為安全性認證使用。 身分識別登錄也會儲存每個裝置的已啟用/已停用旗標，以便您用來停用該裝置的存取權。 如果您的應用程式需要儲存其他裝置特定的中繼資料，它應該使用應用程式專用的存放區。 模組身分識別沒有啟用/停用旗標。 如需詳細資訊，請參閱 [IoT 中樞開發人員指南](../articles/iot-hub/iot-hub-devguide-identity-registry.md)。
