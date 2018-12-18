---
title: 透過 Azure IoT 中樞更新裝置韌體 | Microsoft Docs
description: 使用作業和裝置對應項實作裝置韌體更新程序
services: iot-hub
author: dominicbetts
manager: timlt
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/21/2018
ms.author: dobett
ms.custom: mvc
ms.openlocfilehash: bc1887ef3cdbc56732317aea15be7a618c35847e
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40003571"
---
# <a name="tutorial-implement-a-device-firmware-update-process"></a>教學課程：實作裝置韌體更新程序

您可能需要對連線至 IoT 中樞的裝置更新軔體。 例如，您可以將新功能新增至韌體，或套用安全性修補程式。 在許多 IoT 案例中，實際瀏覽並手動將韌體更新套用至裝置，並非切實可行的做法。 本教學課程將說明透過連線至中樞的後端應用程式，從遠端啟動並監視韌體更新程序。

若要建立及監視韌體更新程序，請在本教學課程中，以後端應用程式在 IoT 中樞內建立_組態_。 IoT 中樞[自動裝置管理](iot-hub-auto-device-config.md)會使用此組態，在您的所有 Chiller 裝置上更新一組_裝置對應項的所需屬性_。 所需屬性會指定必要韌體更新的詳細資料。 Chiller 裝置在執行韌體更新程序時，會使用_裝置對應項的報告屬性_將其狀態報告給後端應用程式。 後端應用程式可使用此組態來監視從裝置傳送的報告屬性，並追蹤韌體更新程序的完成進度：

![韌體更新程序](media/tutorial-firmware-update/Process.png)

在本教學課程中，您會完成下列工作：

> [!div class="checklist"]
> * 建立 IoT 中樞，並將測試裝置新增至裝置身分識別登錄。
> * 建立用以定義軔體更新的組態。
> * 模擬裝置上的軔體更新程序。
> * 在韌體更新進行時接收來自裝置的狀態更新。

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

若要完成本教學課程，您的 Azure 訂用帳戶必須具有已將裝置新增至身分識別登錄的 IoT 中樞。 裝置身分識別登錄中的項目可讓您在此教學課程中執行的模擬裝置連線至中樞。

如果您尚未在訂用帳戶中設定 IoT 中樞，您可以使用下列 CLI 指令碼設定一個。 此指令碼會使用 **tutorial-iot-hub** 作為 IoT 中樞的名稱，但您在執行指令碼時，應將此名稱取代為您自己的唯一名稱。 此指令碼會在**美國中部**區域建立資源群組和中樞，但您可以變更為您附近的區域。 此指令碼會擷取您的 IoT 中樞服務連接字串，此即為您在後端範例應用程式中用來連線至 IoT 中樞的連接字串：

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed
az extension add --name azure-cli-iot-ext

# Create a resource group
az group create --name tutorial-iot-hub-rg --location $location

# Create your free-tier IoT Hub. You can only have one free IoT Hub per subscription
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --sku F1

# Make a note of the service connection string, you need it later
az iot hub show-connection-string --hub-name $hub-name -o table

```

本教學課程使用名為 **MyFirmwareUpdateDevice** 的模擬裝置。 下列指令碼會將此裝置新增至您的裝置身分識別登錄、設定標記值，並擷取其連接字串：

```azurecli-interactive
# Set the name of your IoT hub
hubname=tutorial-iot-hub

# Create the device in the identity registry
az iot hub device-identity create --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Add a device type tag
az iot hub device-twin update --device-id MyFirmwareUpdateDevice --hub-name $hubname --set tags='{"devicetype":"chiller"}'

# Retrieve the device connection string, you need this later
az iot hub device-identity show-connection-string --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

如果您在 Windows 命令提示字元或 Powershell 命令提示字元中執行這些命令，請參閱 [azure-iot-cli-extension tips](https://github.com/Azure/azure-iot-cli-extension/wiki/Tips
) 頁面，以取得如何為 JSON 字串加上引號的相關資訊。

## <a name="start-the-firmware-update"></a>開始進行韌體更新

您可以在後端應用程式中建立[自動裝置管理組態](iot-hub-auto-device-config.md#create-a-configuration)，以開始在 **devicetype** 標記為 Chiller 的所有裝置上進行韌體更新程序。 在本節中，您將了解如何：

* 從後端應用程式建立組態。
* 監視作業的完成進度。

### <a name="use-desired-properties-to-start-the-firmware-upgrade-from-the-back-end-application"></a>使用所需屬性，從後端應用程式開始進行韌體升級

若要檢視會建立組態的後端應用程式程式碼，請在您所下載的範例 Node.js 專案中瀏覽至 **iot-hub/Tutorials/FirmwareUpdate** 資料夾。 接著，在文字編輯器中開啟 ServiceClient.js 檔案。

後端應用程式會建立下列組態：

[!code-javascript[Automatic device management configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=configuration "Automatic device management configuration")]

此組態包含下列區段︰

* `content` 會指定傳送至所選裝置的韌體所需屬性。
* `metrics` 會指定要執行以報告韌體更新狀態的查詢。
* `targetCondition` 會選取要接收韌體更新的裝置。
* `priorty` 會設定此組態相對於其他組態的優先權。

後端應用程式會使用下列程式碼來建立用以設定所需屬性的組態：

[!code-javascript[Create configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=createConfiguration "Create configuration")]

此應用程式建立組態後，即會監視韌體更新：

[!code-javascript[Monitor firmware update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=monitorConfiguration "Monitor firmware update")]

組態會報告兩種類型的計量：

* 系統計量，會報告有多少裝置設為目標，以及有多少裝置已套用更新。
* 您新增至組態的查詢所產生的自訂計量。 在本教學課程中，查詢會報告更新程序的各個階段有多少個裝置。

### <a name="respond-to-the-firmware-upgrade-request-on-the-device"></a>回應裝置的韌體升級要求

若要檢視負責對從後端應用程式傳送的韌體所需屬性進行處理的模擬裝置程式碼，請在您所下載的範例 Node.js 專案中瀏覽至 **iot-hub/Tutorials/FirmwareUpdate** 資料夾。 接著，在文字編輯器中開啟 SimulatedDevice.js 檔案。

模擬裝置應用程式會在裝置對應項中建立 **properties.desired.firmware** 所需屬性的更新所適用的處理常式。 在此處理常式中，它會對所需屬性執行一些基本檢查，然後再啟動更新程序：

[!code-javascript[Handle desired property update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=initiateUpdate "Handle desired property update")]

## <a name="update-the-firmware"></a>更新韌體

**InitiateFirmwareUpdateFlow** 函式會執行更新。 此函式會使用 **waterfall** 函式循序執行更新程序的各個階段。 在此範例中，韌體更新分成四個階段。 第一個階段會下載映像、第二個階段會使用總和檢查碼驗證映像、第三個階段會套用映像，最後一個階段會將裝置重新開機：

[!code-javascript[Firmware update flow](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=firmwareupdateflow "Firmware update flow")]

在進行更新的過程中，模擬裝置會使用報告的屬性來報告進度：

[!code-javascript[Reported properties](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=reportedProperties "Reported properties")]

下列程式碼片段說明下載階段的實作方式。 在下載階段中，模擬裝置會使用報告的屬性將狀態資訊傳送至後端應用程式：

[!code-javascript[Download image phase](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=downloadimagephase "Download image phase")]

## <a name="run-the-sample"></a>執行範例

在本節中，您會執行兩個範例應用程式，以觀察後端應用程式如何建立組態以管理模擬裝置的韌體更新程序。

若要執行模擬裝置和後端應用程式，您必須要有裝置和服務的連接字串。 您在本教學課程一開始建立資源時已記下連接字串。

若要執行模擬裝置應用程式，請開啟殼層或命令提示字元視窗，並在您所下載的 Node.js 專案中瀏覽至 **iot-hub/Tutorials/FirmwareUpdate** 資料夾。 然後，執行下列命令：

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

若要執行後端應用程式，請開啟另一個殼層或命令提示字元視窗。 然後，在您所下載的 Node.js 專案中瀏覽至 **iot-hub/Tutorials/FirmwareUpdate** 資料夾。 然後，執行下列命令：

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

下列螢幕擷取畫面顯示模擬裝置應用程式的輸出，並顯示它如何回應來自後端應用程式的韌體所需屬性更新：

![模擬裝置](./media/tutorial-firmware-update/SimulatedDevice.png)

下列螢幕擷取畫面顯示後端應用程式的輸出，並醒目提示它如何建立組態以更新韌體所需屬性：

![後端應用程式](./media/tutorial-firmware-update/BackEnd1.png)

下列螢幕擷取畫面顯示後端應用程式的輸出，並醒目提示它如何監視來自模擬裝置的韌體更新計量：

![後端應用程式](./media/tutorial-firmware-update/BackEnd2.png)

由於 IoT 中樞裝置身分識別登錄中的延遲，您可能不會看到每個傳送至後端應用程式的狀態更新。 您也可以在入口網站中，經由 IoT 中樞的 [自動裝置管理] -> [IoT 裝置組態] 區段來檢視計量：

![在入口網站中檢視組態](./media/tutorial-firmware-update/portalview.png)

## <a name="clean-up-resources"></a>清除資源

如果您打算繼續完成下一個教學課程，請保留資源群組和 IoT 中樞，以供稍後重複使用。

如果您不再需要 IoT 中樞，請在入口網站中刪除它和資源群組。 若要這麼做，請選取包含 IoT 中樞的 **tutorial-iot-hub-rg** 資源群組，然後按一下 [刪除]。

或者，請使用 CLI：

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何實作連線裝置的韌體更新程序。 請繼續進行下一個教學課程，以了解如何使用 Azure IoT 中樞入口網站工具和 Azure CLI 命令來測試裝置連線能力。

> [!div class="nextstepaction"]
[使用模擬裝置來測試與您 IoT 中樞的連線能力](tutorial-connectivity.md)
