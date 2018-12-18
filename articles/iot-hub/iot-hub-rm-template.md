---
title: 使用範本建立 Azure IoT 中樞 (.NET) | Microsoft Docs
description: 如何在 C# 程式中使用 Azure Resource Manager 範本建立 IoT 中樞。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: dobett
ms.openlocfilehash: 1a64749b7218fccfdad6b6eeebfac39a44aa0522
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38687766"
---
# <a name="create-an-iot-hub-using-azure-resource-manager-template-net"></a>使用 Azure Resource Manager 範本建立 IoT 中樞 (.NET)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

您可以使用 Azure 資源管理員，以程式設計方式建立和管理 Azure IoT 中樞。 本教學課程示範如何使用 Azure Resource Manager 範本從 C# 程式建立 IoT 中樞。

> [!NOTE]
> Azure 有兩種不同的部署模型可建立和處理資源：[Azure Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋使用 Azure Resource Manager 部署模型的部分。

若要完成此教學課程，您需要下列項目：

* Visual Studio 2015 或 Visual Studio 2017。
* 使用中的 Azure 帳戶。 <br/>如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。
* 可供您儲存 Azure Resource Manager 範本檔案的 [Azure 儲存體帳戶][lnk-storage-account]。
* [Azure PowerShell 1.0][lnk-powershell-install] 或更新版本。

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>準備 Visual Studio 專案

1. 在 Visual Studio 中，使用 [主控台應用程式 (.NET Framework)] 專案範本，建立 Visual C# Windows 傳統桌面專案。 將專案命名為 **CreateIoTHub**。

2. 在方案總管中，於專案上按一下滑鼠右鍵，然後按一下 [管理 NuGet 封裝] 。

3. 在 NuGet 套件管理員中，勾選 [包含發行前版本]，然後在 [瀏覽] 頁面上搜尋 **Microsoft.Azure.Management.ResourceManager**。 選取套件，按一下 [安裝]，在 [檢閱變更] 中按一下 [確定]，然後按一下 [我接受] 來接受授權。

4. 在 NuGet 套件管理員中，搜尋 **Microsoft.IdentityModel.Clients.ActiveDirectory**。  按一下 [安裝]，在 [檢閱變更] 中按一下 [確定]，然後按一下 [我接受] 來接受授權。

5. 在 Program.cs 中，以下列程式碼取代現有的 **using** 陳述式：

    ```csharp
    using System;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    ```

6. 在 Program.cs 中，以下列靜態變數取代預留位置值。 您先前已在本教學課程中記下 **ApplicationId**、**SubscriptionId**、**TenantId** 及 **Password**。 **儲存體帳戶名稱**是您儲存 Azure Resource Manager 範本檔案之「Azure 儲存體」帳戶的名稱。 **資源群組名稱**是您建立 IoT 中樞時所使用之資源群組的名稱。 名稱可以是預先存在或新的資源群組。 **部署名稱**是部署的名稱，例如 **Deployment_01**。

    ```csharp
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";
    static string storageAddress = "https://{Your storage account name}.blob.core.windows.net";
    static string rgName = "{Resource group name}";
    static string deploymentName = "{Deployment name}";
    ```

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="submit-a-template-to-create-an-iot-hub"></a>提交範本，以建立 IoT 中樞

使用 JSON 範本和參數檔案，在資源群組中建立 IoT 中樞。 您也可以使用 Azure Resource Manager 範本來對現有的 IoT 中樞進行變更。

1. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，按一下 [加入]，然後按一下 [新增項目]。 將名為 **template.json** 的 JSON 檔案新增到專案中。

2. 若要將標準 IoT 中樞新增到**美國東部**區域，請以下列資源定義取代 **template.json** 的內容。 如需目前支援「IoT 中樞」的區域清單，請參閱 [Azure 狀態][lnk-status]：

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": {
          "type": "string"
        }
      },
      "resources": [
      {
        "apiVersion": "2016-02-03",
        "type": "Microsoft.Devices/IotHubs",
        "name": "[parameters('hubName')]",
        "location": "East US",
        "sku": {
          "name": "S1",
          "tier": "Standard",
          "capacity": 1
        },
        "properties": {
          "location": "East US"
        }
      }
      ],
      "outputs": {
        "hubKeys": {
          "value": "[listKeys(resourceId('Microsoft.Devices/IotHubs', parameters('hubName')), '2016-02-03')]",
          "type": "object"
        }
      }
    }
    ```

3. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，按一下 [加入]，然後按一下 [新增項目]。 將名為 **parameters.json** 的 JSON 檔案新增到專案中。

4. 使用下列參數資訊來取代 **parameters.json** 的內容，此參數資訊會設定新 IoT 中樞的名稱，例如 **{your initials}mynewiothub**。 IoT 中樞名稱必須是全域唯一的，因此應該包含您的名稱或縮寫：

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": { "value": "mynewiothub" }
      }
    }
    ```
  [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

5. 在 [伺服器總管] 中，連接到您的 Azure 訂用帳戶，然後在您的「Azure 儲存體」帳戶中建立名為 **templates** 的容器。 在 [屬性] 面板中，將 **templates** 容器的 [公用讀取存取] 權限設定為 [Blob]。

6. 在 [伺服器總管] 中，於 [templates] 容器上按一下滑鼠右鍵，然後按一下 [檢視 Blob 容器]。 按一下 [上傳 Blob] 按鈕，選取 **parameters.json** 和 **templates.json** 這兩個檔案，然後按一下 [開啟]，將 JSON 檔案上傳至 **templates** 容器。 包含 JSON 資料的 blob 的 URL 如下：

    ```csharp
    https://{Your storage account name}.blob.core.windows.net/templates/parameters.json
    https://{Your storage account name}.blob.core.windows.net/templates/template.json
    ```
7. 將下列方法新增至 Program.cs：

    ```csharp
    static void CreateIoTHub(ResourceManagementClient client)
    {

    }
    ```

8. 將下列程式碼加入 **CreateIoTHub** 方法，以提交範本和參數檔案給 Azure Resource Manager：

    ```csharp
    var createResponse = client.Deployments.CreateOrUpdate(
        rgName,
        deploymentName,
        new Deployment()
        {
          Properties = new DeploymentProperties
          {
            Mode = DeploymentMode.Incremental,
            TemplateLink = new TemplateLink
            {
              Uri = storageAddress + "/templates/template.json"
            },
            ParametersLink = new ParametersLink
            {
              Uri = storageAddress + "/templates/parameters.json"
            }
          }
        });
    ```

9. 將下列程式碼加入 **CreateIoTHub** 方法，以顯示新的 IoT 中樞的狀態和金鑰：

    ```csharp
    string state = createResponse.Properties.ProvisioningState;
    Console.WriteLine("Deployment state: {0}", state);

    if (state != "Succeeded")
    {
      Console.WriteLine("Failed to create iothub");
    }
    Console.WriteLine(createResponse.Properties.Outputs);
    ```

## <a name="complete-and-run-the-application"></a>完成並執行應用程式

在建置和執行應用程式之前，您現在可以呼叫 **CreateIoTHub** 方法來完成應用程式。

1. 在 **Main** 方法的結尾加入下列程式碼：

    ```csharp
    CreateIoTHub(client);
    Console.ReadLine();
    ```

2. 按一下 [建置]，然後按一下 [建置方案]。 更正所有錯誤。

3. 按一下 [偵錯]，然後按一下 [開始偵錯] 以執行應用程式。 可能需要數分鐘的時間，部署才會開始執行。

4. 若要確認您的應用程式已新增新的 IoT 中樞，請前往 [Azure 入口網站][ lnk-azure-portal]並檢視您的資源清單。 或者，使用 **Get-AzureRmResource** PowerShell Cmdlet。

> [!NOTE]
> 此範例應用程式會加入您付費的「S1 標準 IoT 中樞」。 您可透過 [Azure 入口網站][lnk-azure-portal]刪除此 IoT 中樞，或在完成時，使用 **Remove-AzureRmResource** PowerShell Cmdlet。

## <a name="next-steps"></a>後續步驟
現在您已經使用 Azure Resource Manager 範本和 C# 程式部署 IoT 中樞，可以進一步探索：

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
[lnk-storage-account]:../storage/common/storage-create-storage-account.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
