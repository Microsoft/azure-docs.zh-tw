---
title: 匯出 Azure IoT Central 中的資料 | Microsoft Docs
description: 如何從 Azure IoT Central 應用程式匯出資料
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: iot-central
manager: peterpr
ms.openlocfilehash: 86128abd82ee41459a84fc7d9169042179807793
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034904"
---
# <a name="export-your-data-in-azure-iot-central"></a>匯出 Azure IoT Central 中的資料

*此主題適用於系統管理員。*

此文章說明如何使用 Azure IoT Central 中的連續資料匯出功能定期將資料匯出到您的 Azure Blob 儲存體帳戶。 您能以 [Apache AVRO](https://avro.apache.org/docs/current/index.html) 格式的檔案匯出**量測**、**裝置**和**裝置範本**。 所匯出的資料可用來進行冷路徑分析 (例如，在 Azure Machine Learning 中為模型定型) 或在 Microsoft Power BI 中進行長期趨勢分析。

> [!Note]
> 當您開啟「連續資料匯出」時，只會取得從該時刻的資料。 目前，無法擷取「連續資料匯出」時的資料。 若要保留更多歷史資料，請儘早開啟「連續資料匯出」。

## <a name="prerequisites"></a>必要條件

- 延伸麼 30 天 IoT Central 應用程式，或付費應用程式。
- 具有 Azure 訂用帳戶的 Azure 帳戶。
- 同一個 Azure 帳戶也是 IoT Central 應用程式中的系統管理員。
- 同一個 Azure 帳戶有權限可以建立儲存體帳戶，或存取相同 Azure 訂用帳戶中的現有儲存體帳戶。

## <a name="types-of-data-to-export"></a>要匯出的資料類型

### <a name="measurements"></a>量測

裝置所傳送的量測每隔一分鐘就會匯出至您的儲存體帳戶。 該資料有 IoT Central 在該時段內從所有裝置接收的所有新訊息。 所匯出 AVRO 檔案的格式，會和 [IoT 中樞訊息路由](https://docs.microsoft.com/azure/iot-hub/iot-hub-csharp-csharp-process-d2c)匯出至 Blob 儲存體的訊息檔案格式相同。

> [!NOTE]
> 傳送量測的裝置會以裝置識別碼來表示 (請參閱以下各節)。 若要取得裝置名稱，請匯出裝置快照集。 請使用與裝置記錄之 **deviceId** 相符的 **connectionDeviceId** 來相互關聯每個訊息記錄。

下列範例顯示已解碼之 AVRO 檔案中的範例：

```json
{
    "EnqueuedTimeUtc": "2018-06-11T00:00:08.2250000Z",
    "Properties": {},
    "SystemProperties": {
        "connectionDeviceId": "<connectionDeviceId>",
        "connectionAuthMethod": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "connectionDeviceGenerationId": "<generationId>",
        "enqueuedTime": "2018-06-11T00:00:08.2250000Z"
    },
    "Body": "{\"humidity\":80.59100954598546,\"magnetometerX\":0.29451796907056726,\"magnetometerY\":0.5550332126050068,\"magnetometerZ\":-0.04116681874733441,\"connectivity\":\"connected\",\"opened\":\"triggered\"}"
}
```

### <a name="devices"></a>裝置

當第一次開啟「連續資料匯出」時，會匯出包含所有裝置的單一快照集。 每個裝置包含：
- IoT Central 中裝置的 `id`
- 裝置的 `name`
- 來自[裝置佈建服務](https://aka.ms/iotcentraldocsdps) 的 `deviceId`
- 裝置範本資訊
- 屬性值
- 設定值

系統每隔一分鐘就會寫入新的快照集。 快照集包括：

- 自上一個快照集以來所新增的裝置。
- 自上一個快照集以來屬性與設定值已變更的裝置。

> [!NOTE]
> 自上一個快照集以來所刪除的裝置則不會匯出。 目前，快照集沒有已刪除之裝置的指標。
>
> 每個裝置所屬的裝置範本會以裝置範本識別碼來表示。 若要取得裝置範本的名稱，請匯出裝置範本快照集。

已解碼 AVRO 檔案中的記錄看起來可能像這樣：

```json
{
    "id": "<id>",
    "name": "Refrigerator 2",
    "simulated": true,
    "deviceId": "<deviceId>",
    "deviceTemplate": {
        "id": "<template id>",
        "version": "1.0.0"
    },
    "properties": {
        "cloud": {
            "location": "New York",
            "maintCon": true,
            "tempThresh": 20
        },
        "device": {
            "lastReboot": "2018-02-09T22:22:47.156Z"
        }
    },
    "settings": {
        "device": {
            "fanSpeed": 0
        }
    }
}
```

### <a name="device-templates"></a>裝置範本

當第一次開啟「連續資料匯出」時，會匯出包含所有裝置範本的單一快照集。 每個裝置範本包含：
- 裝置範本的 `id`
- 裝置範本的 `name`
- 裝置範本的 `version`
- 量測資料類型和最小值/最大值。
- 屬性資料類型和預設值。
- 設定資料類型和預設值。

系統每隔一分鐘就會寫入新的快照集。 快照集包括：

- 自上一個快照集以來所新增的新裝置範本。
- 自上一個快照集以來量測、屬性與設定定義已變更的裝置範本。

> [!NOTE]
> 自上一個快照集以來所刪除的裝置範本則不會匯出。 目前，快照集沒有已刪除之裝置範本的指標。

已解碼 AVRO 檔案中的記錄看起來可能像這樣：

```json
{
    "id": "<id>",
    "name": "Refrigerated Vending Machine",
    "version": "1.0.0",
    "measurements": {
        "telemetry": {
            "humidity": {
                "dataType": "double",
                "name": "Humidity"
            },
            "magnetometerX": {
                "dataType": "double",
                "name": "Magnetometer X"
            },
            "magnetometerY": {
                "dataType": "double",
                "name": "Magnetometer Y"
            },
            "magnetometerZ": {
                "dataType": "double",
                "name": "Magnetometer Z"
            }
        },
        "states": {
            "connectivity": {
                "dataType": "enum",
                "name": "Connectivity"
            }
        },
        "events": {
            "opened": {
                "name": "Door Opened",
                "category": "informational"
            }
        }
    },
    "settings": {
        "device": {
            "fanSpeed": {
                "dataType": "double",
                "name": "Fan Speed",
                "initialValue": 0
            }
        }
    },
    "properties": {
        "cloud": {
            "location": {
                "dataType": "string",
                "name": "Location",
                "initialValue": "Seattle"
            },
            "maintCon": {
                "dataType": "boolean",
                "name": "Maintenance Contract",
                "initialValue": true
            },
            "tempThresh": {
                "dataType": "double",
                "name": "Temperature Alert Threshold",
                "initialValue": 30
            }
        },
        "device": {
            "lastReboot": {
                "dataType": "dateTime",
                "name": "Last Reboot"
            }
        }
    }
}
```

## <a name="set-up-continuous-data-export"></a>設定連續資料匯出

1. 如果您沒有 Azure 儲存體帳戶，請在 Azure 入口網站中[建立新儲存體帳戶](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM)。 **在 IoT Central 應用程式所在的 Azure 訂用帳戶中**建立儲存體帳戶。
    - 針對帳戶類型，請選擇 [一般用途] 或 [Blob 儲存體]。
    - 選取 IoT Central 應用程式所在的訂用帳戶。 如果您沒有看到訂用帳戶，您可能需要登入不同 Azure 帳戶，或要求訂用帳戶的存取權。
    - 選擇現有的資源群組或建立新群組。 了解[如何建立新的儲存體帳戶](https://aka.ms/blobdocscreatestorageaccount)。

2. 在儲存體帳戶中建立容器以供存放匯出的 IoT Central 資料。 移至您的儲存體帳戶。 在 [Blob 服務] 下，選取 [瀏覽 Blob]。 選取 [容器] 以建立新的容器。

   ![建立容器](media/howto-export-data/createcontainer.png)

3. 使用相同的 Azure 帳戶登入 IoT Central 應用程式。

4. 在 [系統管理] 下，選取 [資料匯出]。

5. 在 [儲存體帳戶] 下拉式清單方塊中，選取您的儲存體帳戶。 在 [容器] 下拉式清單方塊中，選取您的容器。 在 [要匯出的資料] 下，透過將類型設定為 [開啟] 以指定每種要匯出的資料類型。

6. 若要開啟連續資料匯出，請將 [資料匯出] 設定為 [開啟]。 選取 [ **儲存**]。

  ![設定連續資料匯出](media/howto-export-data/continuousdataexport.PNG)

7. 在幾分鐘之後，您的資料就會出現在您的儲存體帳戶中。 瀏覽您的儲存體帳戶。 選取 [瀏覽 Blob] > 您的容器。 您會看到用於匯出資料的三個資料夾。 具有匯出資料之 AVRO 檔案的預設路徑是：
    - 訊息：{container}/measurements/{hubname}/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}.avro
    - 裝置：{container}/devices/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}.avro
    - 裝置範本：{container}/deviceTemplates/{YYYY}/{MM}/{dd}/{hh}/{mm}/{filename}.avro

## <a name="read-exported-avro-files"></a>讀取匯出的 AVRO 檔案

AVRO 是二進位格式，因此無法以原始狀態讀取檔案。 該檔案可解碼為 JSON 格式。 下列範例說明如何剖析量測、裝置和裝置範本 AVRO 檔案。 這些範例對應到上一節中所述的範例。

### <a name="read-avro-files-by-using-python"></a>使用 Python 讀取 AVRO 檔案

#### <a name="install-pandas-and-the-pandavro-package"></a>安裝 pandas 與 pandavro 套件

```python
pip install pandas
pip install pandavro
```

#### <a name="parse-a-measurements-avro-file"></a>剖析量測 AVRO 檔案

```python
import json
import pandavro as pdx
import pandas as pd

def parse(filePath):
    # Pandavro loads the AVRO file into a pandas DataFrame
    # where each record is a single row.
    measurements = pdx.from_avro(filePath)

    # This example creates a new DataFrame and loads a series
    # for each column that's mapped into a column in our new DataFrame.
    transformed = pd.DataFrame()

    # The SystemProperties column contains a dictionary
    # with the device ID located under the connectionDeviceId key.
    transformed["device_id"] = measurements["SystemProperties"].apply(lambda x: x["connectionDeviceId"])

    # The Body column is a series of UTF-8 bytes that is stringified
    # and parsed as JSON. This example pulls the humidity property
    # from each column to get the humidity field.
    transformed["humidity"] = measurements["Body"].apply(lambda x: json.loads(bytes(x).decode('utf-8'))["humidity"])

    # Finally, print the new DataFrame with our device IDs and humidities.
    print(transformed)

```

#### <a name="parse-a-devices-avro-file"></a>剖析裝置 AVRO 檔案

```python
import json
import pandavro as pdx
import pandas as pd

def parse(filePath):
    # Pandavro loads the AVRO file into a pandas DataFrame
    # where each record is a single row.
    devices = pdx.from_avro(filePath)

    # This example creates a new DataFrame and loads a series
    # for each column that's mapped into a column in our new DataFrame.
    transformed = pd.DataFrame()

    # The device ID is available in the id column.
    transformed["device_id"] = devices["deviceId"]

    # The template ID and version are present in a dictionary under
    # the deviceTemplate column.
    transformed["template_id"] = devices["deviceTemplate"].apply(lambda x: x["id"])
    transformed["template_version"] = devices["deviceTemplate"].apply(lambda x: x["version"])

    # The fanSpeed setting value is located in a nested dictionary
    # under the settings column.
    transformed["fan_speed"] = devices["settings"].apply(lambda x: x["device"]["fanSpeed"])

    # Finally, print the new DataFrame with our device and template
    # information, along with the value of the fan speed.
    print(transformed)

```

#### <a name="parse-a-device-templates-avro-file"></a>剖析裝置範本 AVRO 檔案

```python
import json
import pandavro as pdx
import pandas as pd

def parse(filePath):
    # Pandavro loads the AVRO file into a pandas DataFrame
    # where each record is a single row.
    templates = pdx.from_avro(filePath)

    # This example creates a new DataFrame and loads a series
    # for each column that's mapped into a column in our new DataFrame.
    transformed = pd.DataFrame()

    # The template and version are available in the id and version columns.
    transformed["template_id"] = templates["id"]
    transformed["template_version"] = templates["version"]

    # The fanSpeed setting value is located in a nested dictionary
    # under the settings column.
    transformed["fan_speed"] = templates["settings"].apply(lambda x: x["device"]["fanSpeed"])

    # Finally, print the new DataFrame with our device and template
    # information, along with the value of the fan speed.
    print(transformed)
```

### <a name="read-avro-files-by-using-c"></a>使用 C# 讀取 AVRO 檔案

#### <a name="install-the-microsofthadoopavro-package"></a>安裝 Microsoft.Hadoop.Avro 套件

```csharp
Install-Package Microsoft.Hadoop.Avro -Version 1.5.6
```

#### <a name="parse-a-measurements-avro-file"></a>剖析量測 AVRO 檔案

```csharp
using Microsoft.Hadoop.Avro;
using Microsoft.Hadoop.Avro.Container;
using Newtonsoft.Json;

public static async Task Run(string filePath)
{
    using (var fileStream = File.OpenRead(filePath))
    {
        using (var reader = AvroContainer.CreateGenericReader(fileStream))
        {
            // For one AVRO container, where a container can contain multiple blocks,
            // loop through each block in the container.
            while (reader.MoveNext())
            {
                // Loop through the AVRO records in the block and extract the fields.
                foreach (AvroRecord record in reader.Current.Objects)
                {
                    var systemProperties = record.GetField<IDictionary<string, object>>("SystemProperties");
                    var deviceId = systemProperties["connectionDeviceId"] as string;
                    Console.WriteLine("Device ID: {0}", deviceId);

                    using (var stream = new MemoryStream(record.GetField<byte[]>("Body")))
                    {
                        using (var streamReader = new StreamReader(stream, Encoding.UTF8))
                        {
                            var body = JsonSerializer.Create().Deserialize(streamReader, typeof(IDictionary<string, dynamic>)) as IDictionary<string, dynamic>;
                            var humidity = body["humidity"];
                            Console.WriteLine("Humidity: {0}", humidity);
                        }
                    }
                }
            }
        }
    }
}
```

#### <a name="parse-a-devices-avro-file"></a>剖析裝置 AVRO 檔案

```csharp
using Microsoft.Hadoop.Avro;
using Microsoft.Hadoop.Avro.Container;

public static async Task Run(string filePath)
{
    using (var fileStream = File.OpenRead(filePath))
    {
        using (var reader = AvroContainer.CreateGenericReader(fileStream))
        {
            // For one AVRO container, where a container can contain multiple blocks,
            // loop through each block in the container.
            while (reader.MoveNext())
            {
                // Loop through the AVRO records in the block and extract the fields.
                foreach (AvroRecord record in reader.Current.Objects)
                {
                    // Get the field value directly. You can also yield return
                    // records and make the function IEnumerable<AvroRecord>.
                    var deviceId = record.GetField<string>("deviceId");

                    // The device template information is stored in a sub-record
                    // under the deviceTemplate field.
                    var deviceTemplateRecord = record.GetField<AvroRecord>("deviceTemplate");
                    var templateId = deviceTemplateRecord.GetField<string>("id");
                    var templateVersion = deviceTemplateRecord.GetField<string>("version");

                    // The settings and properties are nested two levels deep.
                    // The first level indicates settings or properties.
                    // The second level indicates the type of setting or property.
                    var settingsRecord = record.GetField<AvroRecord>("settings");
                    var deviceSettingsRecord = settingsRecord.GetField<IDictionary<string, dynamic>>("device");
                    var fanSpeed = deviceSettingsRecord["fanSpeed"];
                    
                    Console.WriteLine(
                        "Device ID: {0}, Template ID: {1}, Template Version: {2}, Fan Speed: {3}",
                        deviceId,
                        templateId,
                        templateVersion,
                        fanSpeed
                    );
                }
            }
        }
    }
}

```

#### <a name="parse-a-device-templates-avro-file"></a>剖析裝置範本 AVRO 檔案

```csharp
using Microsoft.Hadoop.Avro;
using Microsoft.Hadoop.Avro.Container;

public static async Task Run(string filePath)
{
    using (var fileStream = File.OpenRead(filePath))
    {
        using (var reader = AvroContainer.CreateGenericReader(fileStream))
        {
            // For one AVRO container, where a container can contain multiple blocks,
            // loop through each block in the container.
            while (reader.MoveNext())
            {
                // Loop through the AVRO records in the block and extract the fields.
                foreach (AvroRecord record in reader.Current.Objects)
                {
                    // Get the field value directly. You can also yield return
                    // records and make the function IEnumerable<AvroRecord>.
                    var id = record.GetField<string>("id");
                    var version = record.GetField<string>("version");

                    // The settings and properties are nested two levels deep.
                    // The first level indicates settings or properties.
                    // The second level indicates the type of setting or property.
                    var settingsRecord = record.GetField<AvroRecord>("settings");
                    var deviceSettingsRecord = settingsRecord.GetField<IDictionary<string, dynamic>>("device");
                    var fanSpeed = deviceSettingsRecord["fanSpeed"];
                    
                    Console.WriteLine(
                        "ID: {1}, Version: {2}, Fan Speed: {3}",
                        id,
                        version,
                        fanSpeed
                    );
                }
            }
        }
    }
}
```

### <a name="read-avro-files-by-using-javascript"></a>使用 Javascript 讀取 AVRO 檔案

#### <a name="install-the-avsc-package"></a>安裝 avsc 套件

```javascript
npm install avsc
```

#### <a name="parse-a-measurements-avro-file"></a>剖析量測 AVRO 檔案

```javascript
const avro = require('avsc');

// Read the AVRO file. Parse the device ID and humidity from each record.
async function parse(filePath) {
    const records = await load(filePath);
    for (const record of records) {
        // Fetch the device ID from the system properties.
        const deviceId = record.SystemProperties.connectionDeviceId;

        // Convert the body from a buffer to a string and parse it.
        const body = JSON.parse(record.Body.toString());

        // Get the humidty property from the body.
        const humidity = body.humidity;

        // Log the retrieved device ID and humidity.
        console.log(`Device ID: ${deviceId}`);
        console.log(`Humidity: ${humidity}`);
    }
}

function load(filePath) {
    return new Promise((resolve, reject) => {
        // The file decoder emits each record as a data event on a stream.
        // Collect the records into an array and return them at the end.
        const records = [];
        avro.createFileDecoder(filePath)
            .on('data', record => { records.push(record); })
            .on('end', () => resolve(records))
            .on('error', reject);
    });
}
```

#### <a name="parse-a-devices-avro-file"></a>剖析裝置 AVRO 檔案

```javascript
const avro = require('avsc');

// Read the AVRO file. Parse the device and template identification
// information and the fanSpeed setting for each device record.
async function parse(filePath) {
    const records = await load(filePath);
    for (const record of records) {
        // Fetch the device ID from the deviceId property.
        const deviceId = record.deviceId;

        // Fetch the template ID and version from the deviceTemplate property.
        const deviceTemplateId = record.deviceTemplate.id;
        const deviceTemplateVersion = record.deviceTemplate.version;

        // Get the fanSpeed from the nested device settings property.
        const fanSpeed = record.settings.device.fanSpeed;

        // Log the retrieved device ID and humidity.
        console.log(`deviceID: ${deviceId}, Template ID: ${deviceTemplateId}, Template Version: ${deviceTemplateVersion}, Fan Speed: ${fanSpeed}`);
    }
}

function load(filePath) {
    return new Promise((resolve, reject) => {
        // The file decoder emits each record as a data event on a stream.
        // Collect the records into an array and return them at the end.
        const records = [];
        avro.createFileDecoder(filePath)
            .on('data', record => { records.push(record); })
            .on('end', () => resolve(records))
            .on('error', reject);
    });
}
```

#### <a name="parse-a-device-templates-avro-file"></a>剖析裝置範本 AVRO 檔案

```javascript
const avro = require('avsc');

// Read the AVRO file. Parse the device and template identification
// information and the fanSpeed setting for each device record.
async function parse(filePath) {
    const records = await load(filePath);
    for (const record of records) {
        // Fetch the template ID and version from the id and verison properties.
        const templateId = record.id;
        const templateVersion = record.version;

        // Get the fanSpeed from the nested device settings property.
        const fanSpeed = record.settings.device.fanSpeed;

        // Log the retrieved device id and humidity.
        console.log(`Template ID: ${templateId}, Template Version: ${templateVersion}, Fan Speed: ${fanSpeed}`);
    }
}

function load(filePath) {
    return new Promise((resolve, reject) => {
        // The file decoder emits each record as a data event on a stream.
        // Collect the records into an array and return them at the end.
        const records = [];
        avro.createFileDecoder(filePath)
            .on('data', record => { records.push(record); })
            .on('end', () => resolve(records))
            .on('error', reject);
    });
}
```

## <a name="next-steps"></a>後續步驟

現在您已經知道如何匯出資料，請繼續下一個步驟：

> [!div class="nextstepaction"]
> [如何在 Power BI 中將資料視覺化](howto-connect-powerbi.md)
