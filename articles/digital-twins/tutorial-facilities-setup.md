---
title: 部署 Azure Digital Twins | Microsoft Docs
description: 了解如何使用本教學課程中步驟，部署 Azure Digital Twins 執行個體及設定空間資源。
services: digital-twins
author: dsk-2015
ms.service: digital-twins
ms.topic: tutorial
ms.date: 10/15/2018
ms.author: dkshir
ms.openlocfilehash: 9ca8a9de9a286d4b14dae4a822f3e9baf4747c60
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49363328"
---
# <a name="tutorial-deploy-azure-digital-twins-and-configure-a-spatial-graph"></a>教學課程：部署 Azure Digital Twins 及設定空間圖形

Azure Digital Twins 服務可讓您在一致的空間系統中將人員、地點及裝置結合在一起。 這系列的教學課程會示範如何使用 Azure Digital Twins 來偵測溫度和空氣品質狀況最佳的會議室使用情况。 這些教學課程會逐步引導您使用 .NET 主控台應用程式來建置辦公大樓的案例，包含多個樓層以及各樓層的會議室。 會議室內含一些裝置，附加可偵測移動、周圍溫度和空氣品質的感應器。 您將了解如何使用 Digital Twins 服務，複寫建築物中的實體區域和實體作為數位物件。 您會使用另一個主控台應用程式來模擬裝置事件。 然後，您將了解如何以近乎即時的方式監視來自這些實體區域和實體的事件。 辦公室管理員可以使用此資訊，協助在此建築物內工作的員工預約狀況最佳的會議室。 辦公室設備管理員可以使用您的設定來找出會議室的使用量趨勢，以及監視運作狀況以便進行維護。

在本系列的第一個教學課程中，您將了解如何：

> [!div class="checklist"]
> * 部署 Digital Twins
> * 將權限授與應用程式
> * 修改 Digital Twins 範例應用程式
> * 佈建您的建築物


這些教學課程會使用並修改[尋找可用會議室快速入門](quickstart-view-occupancy-dotnet.md)所用的相同範例，以涵蓋更詳細且深入的概念。


## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有 Azure 帳戶，請建立一個[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- 這些教學課程中使用的 Digital Twins 範例都是以 C# 撰寫。 請務必在開發電腦上安裝 [.NET Core SDK 2.1.403 版或更新版本](https://www.microsoft.com/net/download)，以建置及執行範例。 在命令視窗中執行 `dotnet --version`，以檢查電腦上安裝的版本是否正確。

- [Visual Studio Code](https://code.visualstudio.com/) 以探索範例程式碼。 

<a id="deploy" />

## <a name="deploy-digital-twins"></a>部署 Digital Twins

使用這一節中的步驟，建立 Digital Twins 服務的新執行個體。 每個訂閱只能建立一個執行個體；如果您已經有一個執行中的執行個體，請跳到下一節。 

[!INCLUDE [create-digital-twins-portal](../../includes/create-digital-twins-portal.md)]


<a id="permissions" />

## <a name="grant-permissions-to-your-app"></a>將權限授與應用程式

Digital Twins 會使用 [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) 來控制服務的[讀取/寫入權限](../active-directory/develop/v1-permissions-and-consent.md)。 任何需要與 Digital Twins 執行個體連線的應用程式，都必須向 Azure Active Directory 註冊。 本節中的步驟示範如何註冊範例應用程式。

如果您已經有一項「應用程式註冊」，您可以將它重複使用於您的範例。 不過，請瀏覽本節內容，以確定您的應用程式註冊設定正確。

[!INCLUDE [digital-twins-permissions](../../includes/digital-twins-permissions.md)]


## <a name="configure-digital-twins-sample"></a>設定 Digital Twins 範例

本節將逐步引導您設定透過 [Digital Twins REST API](https://docs.westcentralus.azuresmartspaces.net/management/swagger/ui/index) 通訊的 Digital Twins 應用程式。 

### <a name="download-the-sample"></a>下載範例
如果您已經針對[尋找可用會議室快速入門](quickstart-view-occupancy-dotnet.md)下載範例，您可以略過這些步驟。

1. 下載 [Digital Twins .Net 範例](https://github.com/Azure-Samples/digital-twins-samples-csharp/archive/master.zip)。 
2. 將電腦上 ZIP 資料夾的內容解壓縮。 

### <a name="explore-the-sample"></a>探索範例
在擷取的範例資料夾中，開啟 Visual Studio Code 中的檔案 **_digital-twins-samples-csharp\digital-twins-samples.code-workspace_**。 此檔案包含兩個專案： 

1. 佈建範例 **_occupancy-quickstart_** 可讓您設定及佈建[空間智慧圖形](concepts-objectmodel-spatialgraph.md#graph)，這是實體空間及其內含資源的數位化影像。 它會使用[物件模型](concepts-objectmodel-spatialgraph.md#model)來定義智慧型建築物的物件。 如需 Digital Twin 和 REST API 的完整清單，請瀏覽[此 REST API 文件](https://docs.westcentralus.azuresmartspaces.net/management/swagger)，或針對[您的執行個體](#deploy)所建立的**管理 API** URL。

   若要探索範例以了解如何與 Digital Twins 執行個體通訊，您可以從 **_src\actions_** 資料夾著手。 此資料夾中的檔案會實作您將在這些教學課程中使用的命令：
    - provisionSample.cs 檔案示範如何佈建空間圖形，
    - *getSpaces.cs* 檔案可取得已佈建空間的相關資訊，
    - *getAvailableAndFreshSpaces.cs* 可取得自訂函式 (稱為使用者定義的函式) 的結果，以及
    - *createEndpoints.cs* 可建立端點來與其他服務互動。

1. 模擬範例 **_device-connectivity_** 可模擬感應器資料，並將資料傳送到針對您的 Digital Twins 執行個體所佈建的 IoT 中樞。 您會在佈建空間圖形之後，在[下一個教學課程中使用此範例](tutorial-facilities-udf.md#simulate)。 用來設定此範例的感應器和裝置識別碼，應該與您要用來佈建圖形的感應器和裝置識別碼相同。

### <a name="configure-the-provisioning-sample"></a>設定佈建範例
1. 開啟命令視窗，然後瀏覽至所下載的範例。 執行以下命令：

    ```cmd/sh
    cd occupancy-quickstart/src
    ```

1. 執行此命令，將相依性還原至範例專案：

    ```cmd/sh
    dotnet restore
    ```

1. 在 Visual Studio Code 中，開啟屬於 **occupancy-quickstart** 專案的 **_appSettings.json_** 檔案，並更新下列值：
    1. *ClientId*：輸入 AAD 應用程式註冊的 [應用程式識別碼]，其記載於[設定應用程式權限](#permissions)一節中。
    1. *租用戶*：輸入 [AAD 租用戶](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant)的**目錄識別碼**，同樣記載於[設定應用程式權限](#permissions)一節中。
    1. *BaseUrl*：輸入您的 Digital Twins 執行個體 URL。 若要取得此 URL，以您執行個體的值取代此 URL 中的預留位置：**https://yourDigitalTwinsName.yourLocation.azuresmartspaces.net/management/api/v1.0/**。 從[部署區段](#deploy)修改 [管理 API] URL，並將 **swagger/** 取代為 **api/v1.0/**，也可以取得此 URL。

1. 執行下列命令，以查看您可以使用此範例探索的 Digital Twins 功能清單。

    ```cmd/sh
    dotnet run
    ```

<a id="provision-spaces" />

## <a name="understand-provisioning-process"></a>了解佈建程序
本節說明範例如何佈建建築物的空間圖表。 

在 Visual Studio Code 中，瀏覽至 **_occupancy-quickstart\src\actions_** 資料夾並開啟 provisionSample.cs 檔案。 請注意下列函式：

```csharp
public static async Task<IEnumerable<ProvisionResults.Space>> ProvisionSample(HttpClient httpClient, ILogger logger)
{
    IEnumerable<SpaceDescription> spaceCreateDescriptions;
    using (var r = new StreamReader("actions/provisionSample.yaml"))
    {
        spaceCreateDescriptions = await GetProvisionSampleTopology(r);
    }

    var results = await CreateSpaces(httpClient, logger, spaceCreateDescriptions, Guid.Empty);

    Console.WriteLine($"Completed Provisioning: {JsonConvert.SerializeObject(results, Formatting.Indented)}");

    return results;
}

```

此函式會使用相同資料夾中的 provisionSample.yaml。 開啟此檔案，並記下辦公大樓的階層：Venue、Floor、Area和 Rooms。 上述任何實體空間均可包含「裝置」和「感應器」。 每個項目都有預先定義的 `type`，例如 Floor、Room。 

範例 *yaml* 檔案會使用 `Default` Digital Twins 物件模型來顯示空間圖形。 此模型會提供大部分類型的泛型名稱 (例如，「溫度」表示 SensorDataType、「地圖」表示 SpaceBlobType) 和空間類型 (例如，子類型為 FocusRoom、ConferenceRoom 等等的會議室)，這對建築物而言已足夠。 如果您必須針對不同類型的場地 (例如工廠) 建立空間圖形，您可能需要不同的物件模型。 您可以在命令列中針對佈建範例執行 `dotnet run GetOntologies` 命令，以找出可供使用的模型。 如需空間圖形和物件模型的詳細資訊，請參閱[了解 Digital Twins 物件模型和空間智慧圖形](concepts-objectmodel-spatialgraph.md)。 

### <a name="modify-sample-spatial-graph"></a>修改範例空間圖表
provisionSample.yaml 包含下列節點：

- **resources**：`resources` 節點會建立 IoT 中樞資源來與您設定中的裝置進行通訊。 位於您圖形根節點的 IoT 中樞，可以與您圖形中的所有裝置和感應器通訊。  

- **spaces**：在 Digital Twins 物件模型中，`spaces` 表示實際位置。 每個空間都有一個 `Type` (例如 Region、Venue 或 Customer)，以及易記的 `Name`。 空間可以屬於建立階層式結構的其他空間。 provisionSample.yaml 有虛構建築物的空間圖形。 請注意，`Venue` 內的 `Floor` 類型、樓層中的 `Area`，以及區域中 `Room` 節點的空間邏輯巢狀。 

- **devices**：空間可以包含 `devices`，這是可管理許多感應器的實體或虛擬實體。 比方說，裝置可能是使用者的電話、Raspberry Pi 感應器 Pod、閘道等等。在您範例的虛構建築物中，請注意名為「聚焦會議室」(Focus Room) 且包含 Raspberry Pi 3 A1 裝置的會議室。 每個裝置節點都是以唯一的 `hardwareId` 識別，這已在範例中硬式編碼。 若要針對實際生產環境設定這個範例，請以您設定中的值取代這些值。  

- **sensors**：裝置可以包含多個 `sensors`，其偵測並記錄實體變更，例如溫度、移動、電力等。每個感應器節點都是以 `hardwareId` 唯一識別 (已在此硬式編碼)。 對於實際應用程式，您設定中感應器的唯一識別碼取代這些值。 provisionSample.yaml 檔案有兩個感應器可記錄 Motion 和 CarbonDioxide。 在 CarbonDioxide 感應器的程式碼行下面新增下列幾行，以新增另一個感應器來記錄 Temperature。 請注意，在 provisionSample.yaml 中是以標為註解的程式碼行提供這些行；移除每一行前面的 '#' 字元，即可將它們取消註解。 

    ```yaml
            - dataType: Temperature
              hardwareId: SAMPLE_SENSOR_TEMPERATURE
    ```
    > [!NOTE]
    > 請確定 `dataType` 和 `hardwareId` 索引鍵的排列方式與此程式碼片段上方的陳述式相符。 此外確定您的編輯器不會以空格取代標籤。 

儲存並關閉 provisionSample.yaml 檔案。 在下一個教學課程中，您會在此檔案中新增更多資訊，然後佈建 Azure Digital Twins 範例建築物。


## <a name="clean-up-resources"></a>清除資源

如果您想要停止探索超過此時間點的 Azure Digital Twins，請放心地刪除在本教學課程中建立的資源：

1. 從 [Azure 入口網站](http://portal.azure.com)的左側功能表中，按一下 [所有資源]，選取您的 Digital Twins 資源群組，然後將它 [刪除]。
2. 如有需要，您可以繼續刪除您工作電腦上的範例應用程式。 


## <a name="next-steps"></a>後續步驟

請繼續進行本系列的下一個教學課程，以了解如何實作自訂邏輯來監視範例建築物的狀況。 
> [!div class="nextstepaction"]
> [教學課程：佈建建築物及監視運作狀況](tutorial-facilities-udf.md)

