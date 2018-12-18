---
title: 從 Azure IoT 中樞同步處理裝置狀態 | Microsoft Docs
description: 使用裝置對應項同步處理您的裝置與 IoT 中樞之間的狀態
services: iot-hub
documentationcenter: ''
author: dominicbetts
manager: timlt
ms.assetid: ''
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/14/2018
ms.author: dobett
ms.custom: mvc
ms.openlocfilehash: 3d0f24331243c22fa356de7778a89185df2cde4e
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40003205"
---
<!-- **TODO** Update publish config with repo paths before publishing! -->

# <a name="tutorial-configure-your-devices-from-a-back-end-service"></a>教學課程：從後端服務設定您的裝置

除了從您的裝置接收遙測資料，您可能也需要從後端服務設定您的裝置。 當您將所需的組態傳送至裝置時，您也可以從這些裝置接收狀態和相容性更新。 例如，您可以設定裝置的目標操作溫度範圍，或從裝置收集韌體版本資訊。

若要同步處理裝置與 IoT 中樞之間的狀態資訊，請使用_裝置對應項_。 [裝置對應項](iot-hub-devguide-device-twins.md)是與特定裝置相關聯的 JSON 文件，IoT 中樞會在雲端中將其儲存於可供您[查詢](iot-hub-devguide-query-language.md)之處。 裝置對應項包含_所需屬性_、_報告屬性_和_標記_。 所需屬性由後端應用程式所設定，供裝置讀取。 報告屬性是由裝置所設定，供後端應用程式讀取。 標記由後端應用程式所設定，且一律不會傳送至裝置。 您可以使用標記來組織裝置。 本教學課程說明如何使用所需屬性和報告屬性來同步處理狀態資訊：

![對應項摘要](media/tutorial-device-twins/DeviceTwins.png)

在本教學課程中，您會執行下列工作：

> [!div class="checklist"]
> * 建立 IoT 中樞，並將測試裝置新增至身分識別登錄。
> * 使用所需屬性將狀態資訊傳送至您的模擬裝置。
> * 使用報告屬性從您的模擬裝置接收狀態資訊。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

您在此快速入門中執行的兩個範例應用程式是使用 Node.js 所撰寫的。 您的開發電腦上需要 Node.js 4.x.x 版或更高版本。

您可以從 [nodejs.org](https://nodejs.org) 下載適用於多種平台的 Node.js。

您可以使用下列命令，以確認開發電腦上目前的 Node.js 版本：

```cmd/sh
node --version
```

從 https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip 下載範例 Node.js 專案並將 ZIP 封存檔解壓縮。

## <a name="set-up-azure-resources"></a>設定 Azure 資源

若要完成本教學課程，您的 Azure 訂用帳戶必須包含將裝置新增至身分識別登錄的 IoT 中樞。 裝置身分識別登錄中的項目可讓您在此教學課程中執行的模擬裝置連線至中樞。

如果您尚未在訂用帳戶中設定 IoT 中樞，您可以使用下列 CLI 指令碼設定一個。 此指令碼會使用 **tutorial-iot-hub** 作為 IoT 中樞的名稱，但您在執行指令碼時，應將此名稱取代為您自己的唯一名稱。 此指令碼會在**美國中部**區域建立資源群組和中樞，但您可以變更為您附近的區域。 此指令碼會擷取您的 IoT 中樞服務連接字串，此即為您在後端範例中用來連線至 IoT 中樞的連接字串：

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed:
az extension add --name azure-cli-iot-ext

# Create a resource group:
az group create --name tutorial-iot-hub-rg --location $location

# Create your free-tier IoT Hub. You can only have one free IoT Hub per subscription:
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --sku F1

# Make a note of the service connection string, you need it later:
az iot hub show-connection-string --hub-name $hubname -o table

```

本教學課程使用名為 **MyTwinDevice** 的模擬裝置。 下列指令碼會將此裝置新增至您的身分識別登錄，並擷取其連接字串：

```azurecli-interactive
# Set the name of your IoT hub:
hubname=tutorial-iot-hub

# Create the device in the identity registry:
az iot hub device-identity create --device-id MyTwinDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Retrieve the device connection string, you need this later:
az iot hub device-identity show-connection-string --device-id MyTwinDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

## <a name="send-state-information"></a>傳送狀態資訊

您可以使用所需屬性將狀態資訊從後端應用程式傳送至裝置。 在本節中，您將了解如何：

* 在裝置上接收和處理所需屬性。
* 從後端應用程式傳送所需屬性。

若要檢視會接收所需屬性的模擬裝置範例程式碼，請在您所下載的範例 Node.js 專案中，瀏覽至 **iot-hub/Tutorials/DeviceTwins** 資料夾。 接著，在文字編輯器中開啟 SimulatedDevice.js 檔案。

下列各節說明的程式碼，是為了回應從後端應用程式傳送的所需屬性變更，而在模擬裝置上執行的程式碼：

### <a name="retrieve-the-device-twin-object"></a>擷取裝置對應項物件

下列程式碼會使用裝置連接字串連線至您的 IoT 中樞：

[!code-javascript[Create IoT Hub client](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=createhubclient&highlight=2 "Create IoT Hub client")]

下列程式碼會從用戶端物件中取得對應項：

[!code-javascript[Get twin](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=gettwin&highlight=2 "Get twin")]

### <a name="sample-desired-properties"></a>範例所需屬性

您也可以用任何方便應用程式運作的方式，來建構您的所需屬性。 此範例會使用一個名為 **fanOn** 的最上層屬性，並將其餘屬性分組到個別的**元件**中。 下列 JSON 程式碼片段說明本教學課程使用的所需屬性結構：

[!code[Sample desired properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/desired.json "Sample desired properties")]

### <a name="create-handlers"></a>建立處理常式

您可以建立所需屬性更新的處理常式，以回應 JSON 階層中不同層級的更新。 例如，此處理常式會檢視從後端應用程式傳送至裝置的所有所需屬性變更。 **差異**變數包含從解決方案後端傳送的所需屬性：

[!code-javascript[Handle all properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=allproperties&highlight=2 "Handle all properties")]

下列處理常式只會回應對 **fanOn** 所需屬性所做的變更：

[!code-javascript[Handle fan property](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=fanproperty&highlight=2 "Handle fan property")]

### <a name="handlers-for-multiple-properties"></a>多個屬性的處理常式

在先前顯示的範例所需屬性 JSON 中，**元件**下的**氣候**節點包含兩個屬性：**minTemperature** 和 **maxTemperature**。

裝置的本機**對應項**物件會儲存一組完整的所需屬性和報告屬性。 從後端傳送的**差異**可能僅更新了所需屬性的子集。 在下列程式碼片段中，如果模擬裝置接收到僅對 **minTemperature** 和 **maxTemperature** 其中之一所做的更新，它將會以本機對應項中的值作為另一個值，來設定裝置：

[!code-javascript[Handle climate component](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=climatecomponent&highlight=2 "Handle climate component")]

本機**對應項**物件會儲存一組完整的所需屬性和報告屬性。 從後端傳送的**差異**可能僅更新了所需屬性的子集。

### <a name="handle-insert-update-and-delete-operations"></a>處理插入、更新和刪除作業

從後端傳送的所需屬性不會指出正在對特定的所需屬性執行何種作業。 您的程式碼必須從目前儲存於本機的所需屬性集，以及從中樞傳送的變更，來推斷作業類型。

下列程式碼片段說明模擬裝置如何對所需屬性中所列的**元件**處理插入、更新和刪除作業。 您可以了解如何使用 **null** 值指出應將元件刪除：

[!code-javascript[Handle components](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=components&highlight=2,6,13 "Handle components")]

### <a name="send-desired-properties-to-a-device-from-the-back-end"></a>將所需屬性從後端傳送至裝置

您已了解裝置如何實作用來接收所需屬性更新的處理常式。 本節說明如何將所需屬性變更從後端應用程式傳送至裝置。

若要檢視會接收所需屬性的模擬裝置範例程式碼，請在您所下載的範例 Node.js 專案中，瀏覽至 **iot-hub/Tutorials/DeviceTwins** 資料夾。 接著，在文字編輯器中開啟 ServiceClient.js 檔案。

下列程式碼片段說明如何連線至裝置身分識別登錄，並存取特定裝置的對應項：

[!code-javascript[Create registry and get twin](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=getregistrytwin&highlight=2,6 "Create registry and get twin")]

下列程式碼片段顯示後端應用程式傳送至裝置的不同所需屬性*修補程式*：

[!code-javascript[Patches sent to device](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=patches&highlight=2,12,26,41,56 "Patches sent to device")]

下列程式碼片段說明後端應用程式如何將所需屬性更新傳送至裝置：

[!code-javascript[Send desired properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=senddesiredproperties&highlight=2 "Send desired properties")]

### <a name="run-the-applications"></a>執行應用程式

在本節中，您會執行兩個範例應用程式，以觀察後端應用程式將所需屬性更新傳送至模擬裝置應用程式的過程。

若要執行模擬裝置和後端應用程式，您必須要有裝置和服務的連接字串。 您在本教學課程一開始建立資源時已記下連接字串。

若要執行模擬裝置應用程式，請開啟殼層或命令提示字元視窗，並在您所下載的 Node.js 專案中瀏覽至 **iot-hub/Tutorials/DeviceTwins** 資料夾。 然後，執行下列命令：

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

若要執行後端應用程式，請開啟另一個殼層或命令提示字元視窗。 然後，在您所下載的 Node.js 專案中瀏覽至 **iot-hub/Tutorials/DeviceTwins** 資料夾。 然後，執行下列命令：

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

下列螢幕擷取畫面顯示模擬裝置應用程式的輸出，並醒目提示它如何處理 **maxTemperature** 所需屬性的更新。 您可以看到最上層處理常式和氣候元件處理常式的執行情形：

![模擬裝置](./media/tutorial-device-twins/SimulatedDevice1.png)

下列螢幕擷取畫面顯示後端應用程式的輸出，並醒目提示它如何傳送 **maxTemperature** 所需屬性的更新：

![後端應用程式](./media/tutorial-device-twins/BackEnd1.png)

## <a name="receive-state-information"></a>接收狀態資訊

後端應用程式會以報告屬性的形式接收來自裝置的狀態資訊。 裝置會設定報告屬性，並將其傳送至您的中樞。 後端應用程式可從儲存於中樞的裝置對應項讀取報告屬性的目前值。

### <a name="send-reported-properties-from-a-device"></a>從裝置傳送報告屬性

您可以用修補程式的形式傳送報告屬性值的更新。 下列程式碼片段顯示模擬裝置所傳送之修補程式的範本。 模擬裝置會先更新修補程式中的欄位，再將其傳送至中樞：

[!code-javascript[Reported properties patches](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=reportedpatch&highlight=2 "Reported properties patches")]

模擬裝置會使用下列函式，將包含報告屬性的修補程式傳送至中樞：

[!code-javascript[Send reported properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=sendreportedproperties&highlight=2 "Send reported properties")]

### <a name="process-reported-properties"></a>處理報告屬性

後端應用程式會透過裝置對應項，存取裝置目前的報告屬性值。 下列程式碼片段說明後端應用程式如何讀取模擬裝置的報告屬性值：

[!code-javascript[Display reported properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=displayreportedproperties&highlight=2 "Display reported properties")]

### <a name="run-the-applications"></a>執行應用程式

在本節中，您會執行兩個範例應用程式，以觀察模擬裝置應用程式將報告屬性更新傳送至後端應用程式的過程。

您會執行先前為了觀察所需屬性如何傳送至裝置而執行的兩個相同的範例應用程式。

若要執行模擬裝置和後端應用程式，您必須要有裝置和服務的連接字串。 您在本教學課程一開始建立資源時已記下連接字串。

若要執行模擬裝置應用程式，請開啟殼層或命令提示字元視窗，並在您所下載的 Node.js 專案中瀏覽至 **iot-hub/Tutorials/DeviceTwins** 資料夾。 然後，執行下列命令：

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

若要執行後端應用程式，請開啟另一個殼層或命令提示字元視窗。 然後，在您所下載的 Node.js 專案中瀏覽至 **iot-hub/Tutorials/DeviceTwins** 資料夾。 然後，執行下列命令：

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

下列螢幕擷取畫面顯示模擬裝置應用程式的輸出，並醒目提示它如何將報告屬性更新傳送至您的中樞：

![模擬裝置](./media/tutorial-device-twins/SimulatedDevice2.png)

下列螢幕擷取畫面顯示後端應用程式的輸出，並醒目提示它如何裝置擷取及處理報告屬性更新：

![後端應用程式](./media/tutorial-device-twins/BackEnd2.png)

## <a name="clean-up-resources"></a>清除資源

如果您打算繼續完成下一個教學課程，請保留資源群組和 IoT 中樞，以供稍後重複使用。

如果您不再需要 IoT 中樞，請在入口網站中刪除它和資源群組。 若要這麼做，請選取包含 IoT 中樞的 **tutorial-iot-hub-rg** 資源群組，然後按一下 [刪除]。

或者，請使用 CLI：

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何同步處理您的裝置與 IoT 中樞之間的狀態資訊。 繼續進行下一個教學課程，以了解如何使用裝置對應項實作韌體更新程序。

> [!div class="nextstepaction"]
[實作裝置韌體更新程序](tutorial-firmware-update.md)
