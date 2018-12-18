---
title: 使用資源提供者 REST API 建立 Azure IoT 中樞 | Microsoft Docs
description: 如何使用資源提供者 REST API 建立 IoT 中樞。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: dobett
ms.openlocfilehash: 06f05da839ebca2ae53e255acce7f07d1989673c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38539766"
---
# <a name="create-an-iot-hub-using-the-resource-provider-rest-api-net"></a>使用資源提供者 REST API 建立 IoT 中樞 (.NET)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

您可透過程式設計方式，使用 [IoT 中樞資源提供者 REST API][lnk-rest-api] 建立和管理 Azure IoT 中樞。 本教學課程說明如何使用「IoT 中樞資源提供者 REST API」從 C# 程式建立 IoT 中樞。

> [!NOTE]
> Azure 有兩種不同的部署模型可建立和處理資源：[Azure Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋使用 Azure Resource Manager 部署模型的部分。

若要完成此教學課程，您需要下列項目：

* Visual Studio 2015 或 Visual Studio 2017。
* 使用中的 Azure 帳戶。 <br/>如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。
* [Azure PowerShell 1.0][lnk-powershell-install] 或更新版本。

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>準備 Visual Studio 專案

1. 在 Visual Studio 中，使用 [主控台應用程式 (.NET Framework)] 專案範本，建立 Visual C# Windows 傳統桌面專案。 將專案命名為 **CreateIoTHubREST**。

2. 在方案總管中，於專案上按一下滑鼠右鍵，然後按一下 [管理 NuGet 封裝] 。

3. 在 NuGet 套件管理員中，勾選 [包含發行前版本]，然後在 [瀏覽] 頁面上搜尋 **Microsoft.Azure.Management.ResourceManager**。 選取套件，按一下 [安裝]，在 [檢閱變更] 中按一下 [確定]，然後按一下 [我接受] 來接受授權。

4. 在 NuGet 套件管理員中，搜尋 **Microsoft.IdentityModel.Clients.ActiveDirectory**。  按一下 [安裝]，在 [檢閱變更] 中按一下 [確定]，然後按一下 [我接受] 來接受授權。

5. 在 Program.cs 中，以下列程式碼取代現有的 **using** 陳述式：

    ```csharp
    using System;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    using Microsoft.Rest;
    using System.Linq;
    using System.Threading;
    ```

6. 在 Program.cs 中，以下列靜態變數取代預留位置值。 您先前已在本教學課程中記下 **ApplicationId**、**SubscriptionId**、**TenantId** 及 **Password**。 **資源群組名稱**是您建立 IoT 中樞時所使用之資源群組的名稱。 您可以使用預先存在或新的資源群組。 **IoT 中樞名稱**是您建立的 IoT 中樞名稱，例如 **MyIoTHub**。 您 IoT 中樞的名稱必須是全域唯一的。 **部署名稱**是部署的名稱，例如 **Deployment_01**。

    ```csharp
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";

    static string rgName = "{Resource group name}";
    static string iotHubName = "{IoT Hub name including your initials}";
    ```
[!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="use-the-resource-provider-rest-api-to-create-an-iot-hub"></a>使用資源提供者 REST API 建立 IoT 中樞

使用 [IoT 中樞資源提供者 REST API][lnk-rest-api] 在資源群組中建立 IoT 中樞。 您也可以使用資源提供者 REST API 變更現有的 IoT 中樞。

1. 將下列方法新增至 Program.cs：

    ```csharp
    static void CreateIoTHub(string token)
    {

    }
    ```

2. 將下列程式碼加入 **CreateIoTHub** 方法。 此程式碼會建立標頭中含有驗證權杖的 **HttpClient** 物件：

    ```csharp
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    ```

3. 將下列程式碼加入 **CreateIoTHub** 方法。 此程式碼說明 IoT 中樞建立並產生 JSON 表示法。 如需目前支援「IoT 中樞」的位置清單，請參閱 [Azure 狀態][lnk-status]：

    ```csharp
    var description = new
    {
      name = iotHubName,
      location = "East US",
      sku = new
      {
        name = "S1",
        tier = "Standard",
        capacity = 1
      }
    };

    var json = JsonConvert.SerializeObject(description, Formatting.Indented);
    ```

4. 將下列程式碼加入 **CreateIoTHub** 方法。 這段程式碼會將 REST 要求提交至 Azure。 程式碼接著會檢查回應，並擷取可用來監視部署工作狀態的 URL：

    ```csharp
    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
    var requestUri = string.Format("https://management.azure.com/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var result = client.PutAsync(requestUri, content).Result;

    if (!result.IsSuccessStatusCode)
    {
      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
      return;
    }

    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    ```

5. 將下列程式碼加入 **CreateIoTHub** 方法的結尾。 此程式碼會使用上一個步驟中擷取的 **asyncStatusUri** 位址來等待部署完成：

    ```csharp
    string body;
    do
    {
      Thread.Sleep(10000);
      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
      body = deploymentstatus.Content.ReadAsStringAsync().Result;
    } while (body == "{\"status\":\"Running\"}");
    ```

6. 將下列程式碼加入 **CreateIoTHub** 方法的結尾。 此程式碼會擷取您建立的 IoT 中樞索引鍵，並列印到主控台︰

    ```csharp
    var listKeysUri = string.Format("https://management.azure.com/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var keysresults = client.PostAsync(listKeysUri, null).Result;

    Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);
    ```

## <a name="complete-and-run-the-application"></a>完成並執行應用程式

在建置和執行應用程式之前，您現在可以呼叫 **CreateIoTHub** 方法來完成應用程式。

1. 在 **Main** 方法的結尾加入下列程式碼：

    ```csharp
    CreateIoTHub(token.AccessToken);
    Console.ReadLine();
    ```

2. 按一下 [建置]，然後按一下 [建置方案]。 更正所有錯誤。

3. 按一下 [偵錯]，然後按一下 [開始偵錯] 以執行應用程式。 可能需要數分鐘的時間，部署才會開始執行。

4. 若要確認您的應用程式已新增新的 IoT 中樞，請前往 [Azure 入口網站][ lnk-azure-portal]並檢視您的資源清單。 或者，使用 **Get-AzureRmResource** PowerShell Cmdlet。

> [!NOTE]
> 此範例應用程式會加入您付費的「S1 標準 IoT 中樞」。 完成後，您可透過 [Azure 入口網站][lnk-azure-portal]刪除此 IoT 中樞，或在完成後使用 **Remove-AzureRmResource** PowerShell Cmdlet 加以刪除。

## <a name="next-steps"></a>後續步驟
現在您已經使用資源提供者 REST API 部署 IoT 中樞，您可以進一步探索：

* 閱讀 [IoT 中樞資源提供者 REST API][lnk-rest-api] 功能的相關資訊。
* 如需 Azure Resource Manager 功能的詳細資訊，請參閱 [Azure Resource Manager 概觀][lnk-azure-rm-overview]。

若要深入了解如何開發 IoT 中樞，請參閱以下文章︰

* [C SDK 簡介][lnk-c-sdk]
* [Azure IoT SDK][lnk-sdks]

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: https://docs.microsoft.com/powershell/azure/install-azurerm-ps
[lnk-rest-api]: https://docs.microsoft.com/rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/resource-group-overview.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
