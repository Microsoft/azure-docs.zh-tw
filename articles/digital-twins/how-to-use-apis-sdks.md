---
title: 使用 Azure Digital Twins API 和 SDK
titleSuffix: Azure Digital Twins
description: 瞭解如何使用 Azure 數位 Twins Api （包括 via SDK）。
author: baanders
ms.author: baanders
ms.date: 06/04/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 4a2667e4876682a6b0aa6d6b7a8cf67eaee376cc
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98683662"
---
# <a name="use-the-azure-digital-twins-apis-and-sdks"></a>使用 Azure Digital Twins API 和 SDK

Azure 數位 Twins 隨附 **控制平面 api** 和 **資料平面 api** ，可用於管理您的實例和其元素。 
* 控制平面 Api 是 [Azure Resource Manager (ARM) ](../azure-resource-manager/management/overview.md) api，並涵蓋資源管理作業，例如建立和刪除您的實例。 
* 資料平面 Api 是 Azure 數位 Twins Api，用於管理模型、Twins 和圖形等資料管理作業。

本文概述可用的 Api，以及與其互動的方法。 您可以透過 [Postman](how-to-use-postman.md)) 或 SDK 之類的工具，直接使用 REST api 與其相關聯的 swagger (。

## <a name="overview-control-plane-apis"></a>總覽：控制平面 Api

控制平面 Api 是用來管理整個 Azure 數位 Twins 實例的 [ARM](../azure-resource-manager/management/overview.md) api，因此涵蓋建立或刪除整個實例等作業。 您也會使用這些來建立和刪除端點。

最新的控制平面 API 版本為 _**2020-12-01**_。

若要使用控制平面 Api：
* 您可以參考 [控制平面 Swagger](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/resource-manager/Microsoft.DigitalTwins/stable)存放庫中最新的 Swagger 資料夾，直接呼叫 api。 此資料夾也包含顯示使用方式的範例資料夾。
* 您目前可以在中存取控制項 Api 的 Sdk .。。
  - [**.Net (c # )**](https://www.nuget.org/packages/Microsoft.Azure.Management.DigitalTwins/) ([參考 [自動產生]](/dotnet/api/overview/azure/digitaltwins/management?view=azure-dotnet&preserve-view=true))  ([來源](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Microsoft.Azure.Management.DigitalTwins)) 
  - [**JAVA**](https://search.maven.org/search?q=a:azure-mgmt-digitaltwins) ([參考 [自動產生]](/java/api/overview/azure/digitaltwins?view=azure-java-stable&preserve-view=true))  ([來源](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins)) 
  - [**JavaScript**](https://www.npmjs.com/package/@azure/arm-digitaltwins) ([來源](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/arm-digitaltwins)) 
  - [**Python**](https://pypi.org/project/azure-mgmt-digitaltwins/) ([來源](https://github.com/Azure/azure-sdk-for-python/tree/release/v3/sdk/digitaltwins/azure-mgmt-digitaltwins)) 
  - [**前往**](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/digitaltwins/mgmt/2020-10-31/digitaltwins) ([來源](https://github.com/Azure/azure-sdk-for-go/tree/master/services/digitaltwins/mgmt/2020-10-31/digitaltwins)) 

您也可以透過 [Azure 入口網站](https://portal.azure.com) 和 [CLI](how-to-use-cli.md)與 Azure 數位 Twins 互動，來練習控制項平面 api。

## <a name="overview-data-plane-apis"></a>總覽：資料平面 Api

資料平面 Api 是用來管理 Azure 數位 Twins 實例內元素的 Azure 數位 Twins Api。 這些作業包括建立路由、上傳模型、建立關聯性，以及管理 twins 等作業。 它們可以廣泛分成下列類別：
* **DigitalTwinModels** -DigitalTwinModels 類別包含 api 來管理 Azure 數位 Twins 實例中的 [模型](concepts-models.md) 。 管理活動包括上傳、驗證、抓取及刪除以 DTDL 撰寫的模型。
* **DigitalTwins** -DigitalTwins 類別包含的 api 可讓開發人員在 Azure 數位 twins 實例中建立、修改和刪除 [數位 twins](concepts-twins-graph.md) 及其關聯性。
* **查詢** -查詢類別可讓開發人員在關聯性 [的對應項圖形中尋找數位 twins 的集合](how-to-query-graph.md) 。
* **事件路由** -事件路由類別包含可透過系統和下游服務 [路由資料](concepts-route-events.md)的 api。

最新的資料平面 API 版本為 _**2020-10-31**_。

若要使用資料平面 Api：
* 您可以直接呼叫 Api，方法是 .。。
   - 參考 [資料平面 swagger](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/data-plane/Microsoft.DigitalTwins)存放庫中最新的 Swagger 資料夾。 此資料夾也包含顯示使用方式的範例資料夾。 
   - 查看 [API 參考檔](/rest/api/azure-digitaltwins/)。
* 您可以使用 **.net (c # ) SDK**。 若要使用 .NET SDK .。。
   - 您可以從 NuGet： [DigitalTwins](https://www.nuget.org/packages/Azure.DigitalTwins.Core)來查看和新增套件。 
   - 您可以查看 [SDK 參考檔](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true)。
   - 您可以在 GitHub： [適用于 .net 的 Azure IoT 數位 Twins 用戶端程式庫](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core)中找到 SDK 來源，包括範例的資料夾。 
   - 您可以繼續閱讀本文的 [*.net (c # ) SDK (資料平面)*](#net-c-sdk-data-plane) 一節，以查看詳細的資訊和使用範例。
* 您可以使用 **JAVA SDK**。 使用 JAVA SDK .。。
   - 您可以從 Maven 查看並安裝套件： [`com.azure:azure-digitaltwins-core`](https://search.maven.org/artifact/com.azure/azure-digitaltwins-core/1.0.0/jar)
   - 您可以查看 [SDK 參考檔](/java/api/overview/azure/digitaltwins/client?preserve-view=true&view=azure-java-stable)
   - 您可以在 GitHub 中找到 SDK 來源： [適用于 JAVA 的 Azure IoT 數位 Twins 用戶端程式庫](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/digitaltwins/azure-digitaltwins-core)
* 您可以使用 **JAVASCRIPT SDK**。 使用 JavaScript SDK .。。
   - 您可以從 npm 查看和安裝套件： [適用于 JavaScript 的 Azure Azure 數位 Twins Core 用戶端程式庫](https://www.npmjs.com/package/@azure/digital-twins-core)。
   - 您可以查看 [SDK 參考檔](/javascript/api/@azure/digital-twins-core/?branch=master&view=azure-node-latest&preserve-view=true)。
   - 您可以在 GitHub 中找到 SDK 來源： [適用于 JavaScript 的 Azure Azure 數位 Twins Core 用戶端程式庫](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/digitaltwins/digital-twins-core)
* 您可以使用 **PYTHON SDK**。 使用 Python SDK .。。
   - 您可以從 PyPi： [適用于 Python 的 Azure Azure 數位 Twins Core 用戶端程式庫](https://pypi.org/project/azure-digitaltwins-core/)，查看並安裝套件。
   - 您可以查看 [SDK 參考檔](/python/api/azure-digitaltwins-core/azure.digitaltwins.core?view=azure-python&preserve-view=true)。
   - 您可以在 GitHub 中找到 SDK 來源： [適用于 Python 的 Azure Azure 數位 Twins Core 用戶端程式庫](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/digitaltwins/azure-digitaltwins-core)
* 您可以使用 AutoRest 來產生另一種語言的 SDK。 遵循 how [*to：使用 AutoRest 建立 Azure 數位 Twins 的自訂 sdk*](how-to-create-custom-sdks.md)中的指示。

您也可以透過 [CLI](how-to-use-cli.md)與 Azure 數位 Twins 互動，來練習日期平面 api。

## <a name="net-c-sdk-data-plane"></a>.NET (c # ) SDK (資料平面) 

Azure 數位 Twins .NET (c # ) SDK 是 Azure SDK for .NET 的一部分。 它是開放原始碼，並以 Azure 數位 Twins 資料平面 Api 為基礎。

> [!NOTE]
> 如需 SDK 設計的詳細資訊，請參閱 [Azure sdk 的一般設計原則](https://azure.github.io/azure-sdk/general_introduction.html) 和特定的 [.net 設計指導方針](https://azure.github.io/azure-sdk/dotnet_introduction.html)。

若要使用 SDK，請在您的專案中包含 NuGet 套件 **DigitalTwins** 。 您也需要最新版的 **Azure 身分識別** 套件。 在 Visual Studio 中，您可以使用 NuGet 封裝管理員來新增這些套件 (*> nuget 封裝管理員 > 管理方案) 的 Nuget 套件* 。 或者，您可以使用 .NET 命令列工具搭配下列 NuGet 套件連結中所找到的命令，將這些專案新增至您的專案：
* [**Azure.DigitalTwins.Core**](https://www.nuget.org/packages/Azure.DigitalTwins.Core)。 這是適用於[適用於 .NET 的 Azure Digital Twins SDK](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) 的套件。 
* [**Azure.Identity**](https://www.nuget.org/packages/Azure.Identity). 此程式庫會提供協助驗證 Azure 的工具。

如需在實務中使用 Api 的詳細逐步解說，請參閱 [*教學課程：撰寫用戶端應用程式程式碼*](tutorial-code.md)。 

### <a name="net-sdk-usage-examples"></a>.NET SDK 使用範例

以下是說明如何使用 .NET SDK 的一些程式碼範例。

對服務進行驗證：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="DefaultAzureCredential_basic":::

[!INCLUDE [Azure Digital Twins: local credentials note](../../includes/digital-twins-local-credentials-note.md)] 

上傳模型：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="CreateModel":::

列出模型：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/model_operations.cs" id="GetModels":::

建立 twins：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="CreateTwin_withHelper":::

查詢 twins 和迴圈結果：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/queries.cs" id="FullQuerySample":::

如需此範例應用程式程式碼的逐步解說，請參閱 [*教學課程：撰寫用戶端應用程式程式*](tutorial-code.md) 代碼。 

您也可以在 GitHub 存放庫中找到 [.net (c # ) SDK](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core/samples)的其他範例。

#### <a name="serialization-helpers"></a>序列化協助程式

序列化協助程式是 SDK 內提供的 helper 函式，可用來快速建立或還原序列化對應項資料，以存取基本資訊。 由於核心 SDK 方法預設會將對應項資料傳回為 JSON，因此使用這些協助程式類別可進一步中斷對應項資料可能會很有説明。

可用的 helper 類別包括：
* `BasicDigitalTwin`：表示數位對應項的核心資料
* `BasicRelationship`：代表關聯性的核心資料
* `UpdateOperationUtility`：表示 update 呼叫中使用的 JSON 修補程式資訊
* `WriteableProperty`：表示屬性中繼資料

##### <a name="deserialize-a-digital-twin"></a>還原序列化數位對應項

您一律可以使用您選擇的 JSON 程式庫（例如或）來還原序列化對應項資料 `System.Test.Json` `Newtonsoft.Json` 。 針對對應項的基本存取權，helper 類別讓這項工作變得更方便。

`BasicDigitalTwin`Helper 類別也可讓您透過來存取對應項上定義的屬性 `Dictionary<string, object>` 。 若要列出對應項的屬性，您可以使用：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="GetTwin":::

##### <a name="create-a-digital-twin"></a>建立數位對應項

`BasicDigitalTwin`您可以使用類別來準備資料，以建立對應項實例：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_sample.cs" id="CreateTwin_withHelper":::

上述程式碼相當於下列「手動」變異：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="CreateTwin_noHelper":::

##### <a name="deserialize-a-relationship"></a>將關聯性還原序列化

您一律可以將關聯性資料還原序列化為您選擇的類型。 如需關聯性的基本存取權，請使用類型 `BasicRelationship` 。

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="GetRelationshipsCall":::

`BasicRelationship`Helper 類別也可讓您透過來存取關聯性上定義的屬性 `IDictionary<string, object>` 。 若要列出屬性，您可以使用：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_other.cs" id="ListRelationshipProperties":::

##### <a name="create-a-relationship"></a>建立關聯性

`BasicRelationship`您也可以使用類別來準備資料，以便在對應項實例上建立關聯性：

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_other.cs" id="CreateRelationship_short":::

##### <a name="create-a-patch-for-twin-update"></a>建立對應項更新的修補程式

更新 twins 和關聯性的呼叫會使用 [JSON 修補程式](http://jsonpatch.com/) 結構。 若要建立 JSON 修補程式作業的清單，您可以使用， `JsonPatchDocument` 如下所示。

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/twin_operations_other.cs" id="UpdateTwin":::

## <a name="general-apisdk-usage-notes"></a>一般 API/SDK 使用注意事項

> [!NOTE]
> 請注意，Azure 數位 Twins 目前不支援 **跨原始來源資源分享 (CORS)**。 如需有關影響和解決策略的詳細資訊，請參閱「 [*跨原始資源分享 (CORS)*](concepts-security.md#cross-origin-resource-sharing-cors) *概念： Azure 數位 Twins 解決方案的安全性*」一節。

下列清單提供使用 Api 和 Sdk 的其他詳細資料和一般指導方針。

* 您可以使用 HTTP REST 測試控管（例如 Postman）來對 Azure 數位 Twins Api 進行直接呼叫。 如需此程式的詳細資訊，請參閱 how [*to：利用 Postman 提出要求*](how-to-use-postman.md)。
* 若要使用 SDK，請將類別具現化 `DigitalTwinsClient` 。 此函式需要可以使用套件中各種驗證方法取得的認證 `Azure.Identity` 。 如需詳細資訊 `Azure.Identity` ，請參閱其 [命名空間檔](/dotnet/api/azure.identity?preserve-view=true&view=azure-dotnet)。 
* 您可能會在 `InteractiveBrowserCredential` 開始使用時發現有用的功能，但還有其他幾個選項，包括 [受控識別](/dotnet/api/azure.identity.interactivebrowsercredential?preserve-view=true&view=azure-dotnet)的認證，您可能會使用這些選項來驗證 [使用 MSI](../app-service/overview-managed-identity.md?tabs=dotnet) 針對 azure 數位 Twins 所設定的 azure 函式。 如需詳細資訊 `InteractiveBrowserCredential` ，請參閱其 [類別檔](/dotnet/api/azure.identity.interactivebrowsercredential?preserve-view=true&view=azure-dotnet)。
* 所有服務 API 呼叫都會公開為類別上的成員函式 `DigitalTwinsClient` 。
* 所有服務函數都存在於同步和非同步版本中。
* 所有服務函式都會針對400或更新版本的任何傳回狀態擲回例外狀況。 請務必將呼叫包裝到 `try` 區段中，並至少攔截 `RequestFailedExceptions` 。 如需這種例外狀況類型的詳細資訊，請參閱 [這裡](/dotnet/api/azure.requestfailedexception?preserve-view=true&view=azure-dotnet)。
* 大部分的服務方法 `Response<T>` `Task<Response<T>>` 會針對非同步呼叫傳回或 () ，其中 `T` 是服務呼叫的 return 物件類別。 [`Response`](/dotnet/api/azure.response-1?preserve-view=true&view=azure-dotnet)類別會封裝服務傳回，並在其欄位中提供傳回值 `Value` 。  
* 具有分頁結果的服務方法會傳回 `Pageable<T>` 或 `AsyncPageable<T>` 作為結果。 如需類別的詳細資訊 `Pageable<T>` ，請參閱 [這裡](/dotnet/api/azure.pageable-1?preserve-view=true&view=azure-dotnet); 如需詳細資訊 `AsyncPageable<T>` ，請參閱 [這裡](/dotnet/api/azure.asyncpageable-1?preserve-view=true&view=azure-dotnet)。
* 您可以使用迴圈來反復查看分頁 `await foreach` 的結果。 如需此程式的詳細資訊，請參閱 [這裡](/archive/msdn-magazine/2019/november/csharp-iterating-with-async-enumerables-in-csharp-8)。
* 基礎 SDK 為 `Azure.Core` 。 請參閱 [Azure 命名空間檔](/dotnet/api/azure?preserve-view=true&view=azure-dotnet) 以取得 SDK 基礎結構和類型的參考。

服務方法會盡可能傳回強型別的物件。 不過，因為 Azure 數位 Twins 是以使用者在執行時間中自訂的模型為基礎， (透過上傳至服務) 的 DTDL 模型來執行，所以許多服務 Api 會以 JSON 格式採用和傳回對應項資料。

## <a name="monitor-api-metrics"></a>監視 API 計量

您可以在 [Azure 入口網站](https://portal.azure.com/)中查看 API 計量，例如要求、延遲和失敗率。 

從入口網站首頁，搜尋您的 Azure 數位 Twins 實例以提取其詳細資料。 從 Azure 數位 Twins 實例的功能表中選取 [ **計量** ] 選項，以顯示 [ *計量* ] 頁面。

:::image type="content" source="media/troubleshoot-metrics/azure-digital-twins-metrics.png" alt-text="顯示 Azure 數位 Twins 計量頁面的螢幕擷取畫面":::

您可以從這裡查看實例的計量，並建立自訂的視圖。

## <a name="next-steps"></a>下一步

瞭解如何使用 Postman 對 Api 進行直接要求：
* [*How to：使用 Postman 提出要求*](how-to-use-postman.md)

或者，在本教學課程中建立用戶端應用程式，練習使用 .NET SDK：
* [*教學課程：撰寫用戶端應用程式的程式碼*](tutorial-code.md)