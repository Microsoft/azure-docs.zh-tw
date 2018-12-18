---
title: Azure IoT Edge 離線功能 |Microsoft Docs
description: 了解 IoT Edge 裝置及模組針對延長之時間期間離線執行的方式，以及 IoT Edge 讓一般 IoT 裝置也能離線執行的方式。
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 09/20/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: f4afad753da4a314ade3fb7433c6be3e489e05b0
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033680"
---
# <a name="understand-extended-offline-capabilities-for-iot-edge-devices-modules-and-child-devices-preview"></a>了解 IoT Edge 裝置、模組及子裝置的延伸離線功能 (預覽)

Azure IoT Edge 支援 IoT Edge 裝置上的延伸離線作業，也可以讓非 Edge 子裝置離線執行。 只要 IoT Edge 裝置已有機會連線到 IoT 中樞，該裝置及任何子裝置便能在間歇性或無網際網路連線的情況下繼續運作。 

>[!NOTE]
>IoT Edge 的離線支援目前處於[公開預覽](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)階段。

## <a name="how-it-works"></a>運作方式

當 IoT Edge 裝置進入離線模式時，Edge 中樞便會擔任三個角色。 首先，它會儲存任何可能前往上游的訊息，並會持續儲存到裝置重新連線為止。 其次，它會代替 IoT 中樞驗證模組及子裝置，使其能繼續執行。 第三，它可在子裝置之間進行平常需要透過 IoT 中樞進行的通訊。 

下列範例示範 IoT Edge 案例於離線模式執行的方式：

1. **設定 IoT Edge 裝置。**

   IoT Edge 裝置會自動啟用離線功能。 若要將該功能延伸至其他 IoT 裝置，您需要宣告 IoT 中樞內裝置間的父子關聯。 

2. **與 IoT 中樞進行同步處理。**

   在安裝完 IoT Edge 執行階段之後，IoT Edge 裝置至少需要處於線上狀態一次，以和 IoT 中樞進行同步處理。 在此同步中，IoT Edge 裝置會取得指派給它的任何子裝置詳細資料。 IoT Edge 裝置也會安全地更新其本機快取，允取離線作業，並會擷取遙測訊息本機存放區的設定。 

3. **離線。**

   從 IoT 中樞中斷連線時，IoT Edge 裝置、其部署的模組，以及任何子 IoT 裝置都能無限期的繼續執行。 模組及子裝置可在離線狀態時，透過向 Edge 中樞進行驗證來啟動或重新啟動。 至 IoT 中樞的遙測繫結上游會儲存在本機。 模組或子 IoT 裝置之間的通訊則會透過直接方法或訊息維持。 

4. **重新連線並與 IoT 中樞再次進行同步處理。**

   一旦與 IoT 中樞的連線還原，IoT Edge 裝置便會再次進行同步處理。 本機儲存訊息會依照儲存時的相同順序進行傳遞。 模組和裝置所需屬性及回報屬性間的任何差異都會進行協調。 IoT Edge 裝置會更新任何對其受指派子 IoT 裝置集合進行的變更。

## <a name="restrictions-and-limits"></a>限制

本文描述的延伸離線功能適用於 [IoT Edge 版本 1.0.2 或更高版本](https://github.com/Azure/azure-iotedge/releases)。 先前版本具備一部分的離線功能。 不具備延伸離線功能的現有 IoT Edge 裝置無法透過變更執行階段版本來進行升級，而必須使用新的 IoT Edge 裝置識別重新設定，才能獲得這些功能。 

延伸離線支援適用於所有可使用 IoT 中樞的區域 (美國東部與西歐除外)。 

只有非 Edge IoT 裝置才能作為子裝置新增。 

IoT Edge 裝置及其受指派的子裝置可在一開始的首次同步處理之後，於離線狀態下無限期運作。但是，儲存訊息則取決於存留時間 (TTL) 設定及可用於儲存訊息的磁碟空間。 

## <a name="set-up-an-edge-device"></a>設定 Edge 裝置

針對任何您想要在延伸離線期間執行的 IoT Edge 裝置，請設定 IoT Edge 執行階段，以透過 MQTT 進行通訊。 

針對要將其延伸離線功能延伸至子 IoT 裝置的 IoT Edge 裝置，您需要在 Azure 入口網站中宣告父子關聯。

### <a name="set-the-upstream-protocol-to-mqtt"></a>設定前往 MQTT 的上游通訊協定

設定 Edge 中樞和 Edge 代理程式，以作為上游通訊協定與 MQTT 進行通訊。 此通訊協定是使用部署資訊清單中的環境變數宣告。 

在 Azure 入口網站中，您可以透過在設定部署模組時，選取 [設定進階 Edge 執行階段設定] 來存取 Edge 中樞和 Edge 代理程式模組定義。 針對這兩個模組，建立稱為 **UpstreamProtocol** 的環境變數，然後將其值設為 **MQTT**。 

在部署範本 JSON 中，環境變數會以下列範例所示的方式宣告： 

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
        "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}]}}}"
    },
    "env": {
        "UpstreamProtocol": {
            "value": "MQTT"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

### <a name="assign-child-devices"></a>指派子裝置

子裝置可為任何註冊到相同 IoT 中樞的非 Edge 裝置。 您可以在建立新裝置時，或是從父 IoT Edge 裝置或子 IoT 裝置的裝置詳細資料頁面管理父子關聯。 

   ![從 IoT Edge 裝置詳細資料頁面管理子裝置](./media/offline-capabilities/manage-child-devices.png)

父裝置可擁有多個子裝置，但子裝置只能擁有一個父裝置。

## <a name="optional-offline-settings"></a>選擇性離線設定

若您預期裝置將在長時間內處於離線狀態，並且想要在這之後收集所有期間產生的訊息，請設定 Edge 中樞，使其能夠儲存所有訊息。 您有兩個變更選項，可讓 Edge 中樞儲存長期訊息。 首先請增加存留時間設定，然後為訊息儲存體新增額外的磁碟空間。 

### <a name="time-to-live"></a>存留時間

存留時間設定是在過期前，訊息所能等待傳遞的時間長度。 預設為 7200 秒 (兩小時)。 

此設定是 Edge 中樞的所需設定，會儲存在模組對應項中。 您可以在 Azure 入口網站的 [設定進階 Edge 執行階段設定] 區段內，或是直接於部署資訊清單中設定它。 

```json
"$edgeHub": {
    "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {},
        "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
        }
    }
}
```

### <a name="additional-offline-storage"></a>其他離線儲存體

根據預設，訊息會儲存在 Edge 中樞的容器檔案系統。 若該儲存體數量不足以應付您的離線需求，您可以在 IoT Edge 裝置上專用本機存放區。 您需要為 Edge 中樞建立環境變數，指向容器中的儲存體資料夾。 然後，使用建立選項，將該儲存體資料夾與主機電腦上的資料夾繫結。 

您可以在 Azure 入口網站內的 [設定進階 Edge 執行階段設定] 區段設定環境變數及建立 Edge 中樞模組的選項。 或者，您也可以在部署資訊清單中直接設定。 

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
        "createOptions": "{\"HostConfig\":{\"Binds\":[\"C:\\\\HostStoragePath:C:\\\\ModuleStoragePath\"],\"PortBindings\":{\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}]}}}"
    },
    "env": {
        "storageFolder": {
            "value": "C:\\\\ModuleStoragePath"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

## <a name="next-steps"></a>後續步驟

為 [Linux](how-to-create-transparent-gateway-linux.md) 或 [Windows](how-to-create-transparent-gateway-windows.md) 裝置啟用您透明閘道案例的延伸離線作業。 