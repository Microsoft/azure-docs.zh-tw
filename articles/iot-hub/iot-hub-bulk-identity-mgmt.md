---
title: Azure IoT 中樞裝置身分識別的匯入匯出 | Microsoft Docs
description: 如何使用 Azure IoT 服務 SDK 對身分識別登錄執行大量作業，以匯入和匯出裝置身分識別。 匯入作業可讓您建立、更新和刪除大量裝置身分識別。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/03/2017
ms.author: dobett
ms.openlocfilehash: aedf2d0012f5af8ea2eb8e944f06b20c7f1a6bb8
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42145516"
---
# <a name="manage-your-iot-hub-device-identities-in-bulk"></a>管理大量的 IoT 中樞裝置身分識別

每個 IoT 中樞都有一個身分識別登錄，您可用來在服務中建立各裝置的資源。 身分識別登錄也可讓您控制裝置面向端點的存取權。 本文說明如何在身分識別登錄中大量匯入和匯出裝置身分識別。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

匯入和匯出操作會在「作業」的內容中進行，其可讓您對 IoT 中樞執行大量服務操作。

**RegistryManager** 類別包含使用**作業**架構的 **ExportDevicesAsync** 和 **ImportDevicesAsync** 方法。 這些方法可讓您匯出、匯入和同步處理整個 IoT 中樞身分識別登錄。

本主題討論使用 **RegistryManager** 類別和**作業**的系統來執行將裝置大量匯入和匯出 IoT 中樞的身分識別登錄。 您也可以使用 Azure IoT 中樞裝置佈建服務，以無須人為介入的方式對一或多個 IoT 中樞進行 Just-In-Time 自動佈建。 若要深入了解，請參閱[佈建服務文件](/azure/iot-dps)。


## <a name="what-are-jobs"></a>什麼是作業？

身分識別登錄操作會使用**作業** 系統的前提是操作符合下列條件時：

* 相較於標準執行階段作業，執行時間可能很長。

* 會傳回大量資料給使用者。

與其讓單一 API 呼叫等候或封鎖操作的結果，操作會以非同步方式建立該 IoT 中樞的**作業**。 然後操作會立即傳回 **JobProperties** 物件。

下列 C# 程式碼片段示範如何建立匯出作業：

```csharp
// Call an export job on the IoT Hub to retrieve all devices
JobProperties exportJob = await 
  registryManager.ExportDevicesAsync(containerSasUri, false);
```

> [!NOTE]
> 若要在 C# 程式碼中使用 **RegistryManager** 類別，請將 **Microsoft.Azure.Devices** NuGet 套件新增至您的專案。 **RegistryManager** 類別位於 **Microsoft.Azure.Devices** 命名空間。

您可以使用 **RegistryManager** 類別來查詢**作業**的狀態 (使用所傳回的 **JobProperties** 中繼資料)。 若要建立 **RegistryManager** 類別的執行個體，請使用 **CreateFromConnectionString** 方法。

```csharp
RegistryManager registryManager =
  RegistryManager.CreateFromConnectionString("{your IoT Hub connection string}");
```

若要尋找 IoT 中樞的連接字串，請在 Azure 入口網站中：

- 瀏覽至您的 IoT 中樞。

- 選取 [共用存取原則]。

- 選取原則，並將您需要的權限列入考量。

- 從螢幕右側的面板複製連接字串。

下列 C# 程式碼片段示範如何每五秒輪詢一次以查看作業是否已執行完成：

```csharp
// Wait until job is finished
while(true)
{
  exportJob = await registryManager.GetJobAsync(exportJob.JobId);
  if (exportJob.Status == JobStatus.Completed || 
      exportJob.Status == JobStatus.Failed ||
      exportJob.Status == JobStatus.Cancelled)
  {
    // Job has finished executing
    break;
  }

  await Task.Delay(TimeSpan.FromSeconds(5));
}
```

## <a name="export-devices"></a>匯出裝置

使用 **ExportDevicesAsync** 方法將整個 IoT 中樞身分識別登錄匯出到使用[共用存取簽章](../storage/common/storage-security-guide.md#data-plane-security)的 [Azure 儲存體](../storage/index.yml) Blob 容器。

這個方法可讓您在所控制的 Blob 容器中建立可靠的裝置資訊備份。

**ExportDevicesAsync** 方法需要兩個參數：

* 包含 Blob 容器 URI 的「字串」 。 此 URI 必須包含可授與容器寫入權限的 SAS 權杖。 作業會在這個容器中建立用來儲存序列化匯出裝置資料的區塊 Blob。 SAS 權杖必須包含這些權限：

   ```csharp
   SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read 
     | SharedAccessBlobPermissions.Delete
   ```

* 指出是否要在匯出資料中排除驗證金鑰的 *布林值* 。 若為 **false**，驗證金鑰就會包含在匯出輸出中。 否則，會將金鑰匯出為 **null**。

下列 C# 程式碼片段示範如何啟動在匯出資料中包含裝置驗證金鑰的匯出作業，然後執行輪詢以完成作業：

```csharp
// Call an export job on the IoT Hub to retrieve all devices
JobProperties exportJob = 
  await registryManager.ExportDevicesAsync(containerSasUri, false);

// Wait until job is finished
while(true)
{
    exportJob = await registryManager.GetJobAsync(exportJob.JobId);
    if (exportJob.Status == JobStatus.Completed || 
        exportJob.Status == JobStatus.Failed ||
        exportJob.Status == JobStatus.Cancelled)
    {
    // Job has finished executing
    break;
    }

    await Task.Delay(TimeSpan.FromSeconds(5));
}
```

作業會在所提供的 Blob 容器中將其輸出儲存為名稱是 **devices.txt**的區塊 Blob。 輸出資料包含 JSON 序列化裝置資料，每行代表一個裝置。

下列範例顯示輸出資料：

```json
{"id":"Device1","eTag":"MA==","status":"enabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device2","eTag":"MA==","status":"enabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device3","eTag":"MA==","status":"disabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device4","eTag":"MA==","status":"disabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
{"id":"Device5","eTag":"MA==","status":"enabled","authentication":{"symmetricKey":{"primaryKey":"abc=","secondaryKey":"def="}}}
```

如果裝置有對應項資料，則對應項資料也會隨著裝置資料來匯出。 下列範例示範此格式。 「twinETag」行到結尾的所有資料都是對應項資料。

```json
{
   "id":"export-6d84f075-0",
   "eTag":"MQ==",
   "status":"enabled",
   "statusReason":"firstUpdate",
   "authentication":null,
   "twinETag":"AAAAAAAAAAI=",
   "tags":{
      "Location":"LivingRoom"
   },
   "properties":{
      "desired":{
         "Thermostat":{
            "Temperature":75.1,
            "Unit":"F"
         },
         "$metadata":{
            "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
            "$lastUpdatedVersion":2,
            "Thermostat":{
               "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
               "$lastUpdatedVersion":2,
               "Temperature":{
                  "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
                  "$lastUpdatedVersion":2
               },
               "Unit":{
                  "$lastUpdated":"2017-03-09T18:30:52.3167248Z",
                  "$lastUpdatedVersion":2
               }
            }
         },
         "$version":2
      },
      "reported":{
         "$metadata":{
            "$lastUpdated":"2017-03-09T18:30:51.1309437Z"
         },
         "$version":1
      }
   }
}
```

如果您需要存取程式碼中的這項資料，可以使用 **ExportImportDevice** 類別輕鬆地將此資料還原序列化。 下列 C# 程式碼片段示範如何讀取先前匯出至區塊 Blob 的裝置資訊：

```csharp
var exportedDevices = new List<ExportImportDevice>();

using (var streamReader = new StreamReader(await blob.OpenReadAsync(AccessCondition.GenerateIfExistsCondition(), null, null), Encoding.UTF8))
{
  while (streamReader.Peek() != -1)
  {
    string line = await streamReader.ReadLineAsync();
    var device = JsonConvert.DeserializeObject<ExportImportDevice>(line);
    exportedDevices.Add(device);
  }
}
```

## <a name="import-devices"></a>匯入裝置

**RegistryManager** 類別中的 **ImportDevicesAsync** 方法可讓您在 IoT 中樞身分識別登錄中執行大量匯入和同步處理操作。 與 **ExportDevicesAsync** 方法相同，**ImportDevicesAsync** 方法會使用**作業**架構。

請謹慎使用 **ImportDevicesAsync** 方法，因為除了在身分識別登錄中佈建新裝置外，此方法也會更新和刪除現有裝置。

> [!WARNING]
> 匯入操作是無法復原的。 請一律先使用 **ExportDevicesAsync** 方法將現有資料備份到另一個 Blob 容器，再對身分識別登錄進行大量變更。

**ImportDevicesAsync** 方法會採用兩個參數：

* 包含 [Azure 儲存體](../storage/index.yml) Blob 容器 URI 以作為作業之「輸入」的「字串」。 此 URI 必須包含可授與容器讀取權限的 SAS 權杖。 此容器必須包含名稱為 **devices.txt** 的 Blob，而此 Blob 包含要匯入到身分識別登錄的序列化裝置資料。 匯入資料必須包含 **ExportImportDevice** 作業建立 **devices.txt** Blob 時所使用之相同 JSON 格式的裝置資訊。 SAS 權杖必須包含這些權限：

   ```csharp
   SharedAccessBlobPermissions.Read
   ```

* 包含 [Azure 儲存體](https://azure.microsoft.com/documentation/services/storage/) Blob 容器 URI 以作為作業之「輸出」的「字串」。 作業會在此容器中建立區塊 Blob，以儲存來自已完成之匯入 **作業**的任何錯誤資訊。 SAS 權杖必須包含這些權限：

   ```csharp
   SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read 
     | SharedAccessBlobPermissions.Delete
   ```

> [!NOTE]
> 這兩個參數可以指向相同的 Blob 容器。 參數不同只會讓您更能掌控資料，因為輸出容器需要其他權限。

下列 C# 程式碼片段示範如何啟動匯入作業：

```csharp
JobProperties importJob = 
   await registryManager.ImportDevicesAsync(containerSasUri, containerSasUri);
```

這個方法也可用來匯入裝置對應項的資料。 資料輸入的格式與 **ExportDevicesAsync** 的區段中顯示的格式相同。 如此一來，您可以重新匯入已匯出的資料。 **$metadata** 是選擇性參數。

## <a name="import-behavior"></a>匯入行為

您可以使用 **ImportDevicesAsync** 方法，在您的身分識別登錄中執行下列大量作業：

* 大量註冊新裝置
* 大量刪除現有裝置
* 大量變更狀態 (啟用或停用裝置)
* 大量指派新的裝置驗證金鑰
* 大量自動重新產生裝置驗證金鑰
* 大量更新對應項資料

您可以在單一 **ImportDevicesAsync** 呼叫中執行上述作業的任意組合。 比方說，您可以同時間註冊新裝置並刪除或更新現有裝置。 搭配 **ExportDevicesAsync** 方法一起使用時，您可以將某個 IoT 中樞內的所有裝置移轉到另一個 IoT 中樞。

如果匯入檔案包含對應項中繼資料，則此中繼資料會覆寫現有的對應項中繼資料。 如果匯入檔案不包含對應項中繼資料，則只有 `lastUpdateTime` 中繼資料會使用目前的時間來更新。

在每個裝置的匯入序列化資料中使用選擇性的 **importMode** 屬性控制每個裝置的匯入程序。 **ImportMode** 屬性具有下列選項：

| importMode | 說明 |
| --- | --- |
| **createOrUpdate** |如果不存在具有指定 **識別碼**的裝置，則表示是新註冊的裝置。 <br/>如果裝置已存在，則會以所提供的輸入資料覆寫現有資訊，而不管 **ETag** 值為何。 <br> 使用者可以選擇性地指定對應項資料以及裝置資料。 對應項的 etag (若已指定) 會與裝置的 etag 分開處理。 如果現有對應項的 etag 有不相符之處，系統會在記錄檔中寫入錯誤。 |
| **create** |如果不存在具有指定 **識別碼**的裝置，則表示是新註冊的裝置。 <br/>如果裝置已存在，則會在記錄檔中寫入錯誤。 <br> 使用者可以選擇性地指定對應項資料以及裝置資料。 對應項的 etag (若已指定) 會與裝置的 etag 分開處理。 如果現有對應項的 etag 有不相符之處，系統會在記錄檔中寫入錯誤。 |
| **update** |如果已存在具有指定**識別碼**的裝置，則會以所提供的輸入資料覆寫現有資訊，而不管 **ETag** 值為何。 <br/>如果裝置不存在，則會在記錄檔中寫入錯誤。 |
| **pdateIfMatchETagu** |如果已存在具有指定**識別碼**的裝置，則當 **ETag** 相符時，才會以所提供的輸入資料覆寫現有資訊。 <br/>如果裝置不存在，則會在記錄檔中寫入錯誤。 <br/>如果 **ETag** 不相符，則會在記錄檔中寫入錯誤。 |
| **createOrUpdateIfMatchETag** |如果不存在具有指定 **識別碼**的裝置，則表示是新註冊的裝置。 <br/>如果裝置已存在，則當 **ETag** 相符時，才會以所提供的輸入資料覆寫現有資訊。 <br/>如果 **ETag** 不相符，則會在記錄檔中寫入錯誤。 <br> 使用者可以選擇性地指定對應項資料以及裝置資料。 對應項的 etag (若已指定) 會與裝置的 etag 分開處理。 如果現有對應項的 etag 有不相符之處，系統會在記錄檔中寫入錯誤。 |
| **delete** |如果已存在具有指定**識別碼**的裝置，則會遭到刪除，而不管 **ETag** 值為何。 <br/>如果裝置不存在，則會在記錄檔中寫入錯誤。 |
| **deleteIfMatchETag** |如果已存在具有指定**識別碼**的裝置，則只會在 **ETag** 相符時予以刪除。 如果裝置不存在，則會在記錄檔中寫入錯誤。 <br/>如果 ETag 不相符，則會在記錄檔中寫入錯誤。 |

> [!NOTE]
> 如果序列化資料未明確定義裝置的 **importMode** 旗標，則會在匯入作業期間預設為 **createOrUpdate**。

## <a name="import-devices-example--bulk-device-provisioning"></a>匯入裝置範例 - 大量裝置佈建

下列 C# 程式碼範例說明如何產生多個裝置身分識別，以便：

* 包含驗證金鑰。
* 將該裝置資訊寫入至區塊 Blob。
* 將裝置匯入至身分識別登錄。

```csharp
// Provision 1,000 more devices
var serializedDevices = new List<string>();

for (var i = 0; i < 1000; i++)
{
  // Create a new ExportImportDevice
  // CryptoKeyGenerator is in the Microsoft.Azure.Devices.Common namespace
  var deviceToAdd = new ExportImportDevice()
  {
    Id = Guid.NewGuid().ToString(),
    Status = DeviceStatus.Enabled,
    Authentication = new AuthenticationMechanism()
    {
      SymmetricKey = new SymmetricKey()
      {
        PrimaryKey = CryptoKeyGenerator.GenerateKey(32),
        SecondaryKey = CryptoKeyGenerator.GenerateKey(32)
      }
    },
    ImportMode = ImportMode.Create
  };

  // Add device to the list
  serializedDevices.Add(JsonConvert.SerializeObject(deviceToAdd));
}

// Write the list to the blob
var sb = new StringBuilder();
serializedDevices.ForEach(serializedDevice => sb.AppendLine(serializedDevice));
await blob.DeleteIfExistsAsync();

using (CloudBlobStream stream = await blob.OpenWriteAsync())
{
  byte[] bytes = Encoding.UTF8.GetBytes(sb.ToString());
  for (var i = 0; i < bytes.Length; i += 500)
  {
    int length = Math.Min(bytes.Length - i, 500);
    await stream.WriteAsync(bytes, i, length);
  }
}

// Call import using the blob to add new devices
// Log information related to the job is written to the same container
// This normally takes 1 minute per 100 devices
JobProperties importJob =
   await registryManager.ImportDevicesAsync(containerSasUri, containerSasUri);

// Wait until job is finished
while(true)
{
  importJob = await registryManager.GetJobAsync(importJob.JobId);
  if (importJob.Status == JobStatus.Completed || 
      importJob.Status == JobStatus.Failed ||
      importJob.Status == JobStatus.Cancelled)
  {
    // Job has finished executing
    break;
  }

  await Task.Delay(TimeSpan.FromSeconds(5));
}
```

## <a name="import-devices-example--bulk-deletion"></a>匯入裝置範例 - 大量刪除

下列程式碼範例示範如何刪除使用前一個程式碼範例所新增的裝置：

```csharp
// Step 1: Update each device's ImportMode to be Delete
sb = new StringBuilder();
serializedDevices.ForEach(serializedDevice =>
{
  // Deserialize back to an ExportImportDevice
  var device = JsonConvert.DeserializeObject<ExportImportDevice>(serializedDevice);

  // Update property
  device.ImportMode = ImportMode.Delete;

  // Re-serialize
  sb.AppendLine(JsonConvert.SerializeObject(device));
});

// Step 2: Write the new import data back to the block blob
await blob.DeleteIfExistsAsync();
using (CloudBlobStream stream = await blob.OpenWriteAsync())
{
  byte[] bytes = Encoding.UTF8.GetBytes(sb.ToString());
  for (var i = 0; i < bytes.Length; i += 500)
  {
    int length = Math.Min(bytes.Length - i, 500);
    await stream.WriteAsync(bytes, i, length);
  }
}

// Step 3: Call import using the same blob to delete all devices
importJob = await registryManager.ImportDevicesAsync(containerSasUri, containerSasUri);

// Wait until job is finished
while(true)
{
  importJob = await registryManager.GetJobAsync(importJob.JobId);
  if (importJob.Status == JobStatus.Completed || 
      importJob.Status == JobStatus.Failed ||
      importJob.Status == JobStatus.Cancelled)
  {
    // Job has finished executing
    break;
  }

  await Task.Delay(TimeSpan.FromSeconds(5));
}
```

## <a name="get-the-container-sas-uri"></a>取得容器 SAS URI

下列程式碼範例示範如何產生具有 Blob 容器之讀取、寫入和刪除權限的 [SAS URI](../storage/blobs/storage-dotnet-shared-access-signature-part-2.md)：

```csharp
static string GetContainerSasUri(CloudBlobContainer container)
{
  // Set the expiry time and permissions for the container.
  // In this case no start time is specified, so the
  // shared access signature becomes valid immediately.
  var sasConstraints = new SharedAccessBlobPolicy();
  sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
  sasConstraints.Permissions = 
    SharedAccessBlobPermissions.Write | 
    SharedAccessBlobPermissions.Read | 
    SharedAccessBlobPermissions.Delete;

  // Generate the shared access signature on the container,
  // setting the constraints directly on the signature.
  string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

  // Return the URI string for the container,
  // including the SAS token.
  return container.Uri + sasContainerToken;
}
```

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何對 IoT 中樞內的身分識別登錄執行大量操作。 遵循下列連結以深入了解如何管理 Azure IoT 中樞：

* [IoT 中樞計量](iot-hub-metrics.md)
* [作業監視](iot-hub-operations-monitoring.md)

若要進一步探索 IoT 中樞的功能，請參閱︰

* [IoT 中樞開發人員指南](iot-hub-devguide.md)
* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置](../iot-edge/tutorial-simulate-device-linux.md)

若要探索使用 IoT 中樞裝置佈建服務進行 Just-In-Time 自動佈建，請參閱： 

* [Azure IoT 中樞裝置佈建服務](/azure/iot-dps)
