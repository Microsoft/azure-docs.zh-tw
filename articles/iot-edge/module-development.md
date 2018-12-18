---
title: 開發適用於 Azure IoT Edge 的模組 | Microsoft Docs
description: 了解如何建立適用於 Azure IoT Edge 的自訂模組
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 10/05/2017
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: dbbd07e93602855afb0c9755e8872e0b46557611
ms.sourcegitcommit: 150a40d8ba2beaf9e22b6feff414f8298a8ef868
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37030014"
---
# <a name="understand-the-requirements-and-tools-for-developing-iot-edge-modules"></a>了解開發 IoT Edge 模組的需求和工具

本文說明撰寫要以 IoT Edge 模組執行的應用程式時可以使用哪些功能，以及如何充分利用它們。

## <a name="iot-edge-runtime-environment"></a>IoT Edge 執行階段環境
IoT Edge 執行階段提供基礎結構來整合多個 IoT Edge 模組的功能，並將其部署至 IoT Edge 裝置。 概括而言，任何程式都可封裝為 IoT Edge 模組。 不過，若要完全利用 IoT Edge 通訊和管理功能，在模組中執行的程式可以連線到本機 IoT Edge 中樞 (已在 IoT Edge 執行階段整合)。

## <a name="using-the-iot-edge-hub"></a>使用 IoT Edge 中樞
IoT Edge 中樞提供兩個主要功能：Proxy 到 IoT 中樞，以及本機通訊。

### <a name="iot-hub-primitives"></a>IoT 中樞基本項目
IoT 中樞會看見一個類似裝置的模組執行個體，這表示：

* 它具有一個模組對應項，其與[裝置對應項][ lnk-devicetwin]及該裝置的其他模組對應項截然不同；
* 它會傳送[裝置到雲端的訊息][lnk-iothub-messaging]；
* 它可以接收特別在其身分識別上設為目標的[直接方法][lnk-methods]。

模組目前無法接收雲端到裝置的訊息，也無法使用檔案上傳功能。

撰寫模組時，您可以只使用 [Azure IoT 裝置 SDK][lnk-devicesdk] 來連線到 IoT Edge 中樞，並使用上述功能，就像您在搭配裝置應用程式使用 IoT 中樞時所做的，唯一差異在於您必須從應用程式後端參考模組識別，而不是參考裝置識別。

請參閱[開發 IoT Edge 模組並部署至模擬裝置][lnk-tutorial2]，以取得傳送裝置到雲端訊息和使用模組對應項的模組應用程式範例。

### <a name="device-to-cloud-messages"></a>裝置到雲端的訊息
為了能夠對裝置到雲端的訊息進行複雜處理，IoT Edge 中樞在模組之間以及模組與 IoT 中樞之間提供宣告式路由傳送。
這樣可讓模組攔截並處理其他模組所傳送的訊息，並將它們傳播至複雜的管線。
[模組撰寫][lnk-module-comp]一文說明如何使用路由，將模組撰寫為複雜的管線。

IoT Edge 模組不同於一般的 IoT 中樞裝置應用程式，可接收其本機 IoT Edge 中樞正在進行 Proxy 處理的裝置到雲端訊息，以處理它們。

IoT Edge 中樞會根據[模組撰寫][lnk-module-comp]一文中所述的宣告式路由，將訊息傳播至模組。 開發 IoT Edge 模組時，您可以藉由設定訊息處理常式來接收這些訊息，如[開發 IoT Edge 模組並部署至模擬裝置][lnk-tutorial2]教學課程中所示。

為了簡化路由的建立，IoT Edge 新增了模組「輸入」和「輸出」端點的概念。 模組可以接收路由傳送給它的所有裝置到雲端訊息，而不需指定任何輸入，而且可以在未指定任何輸出的情況下傳送裝置到雲端訊息。
不過，使用明確的輸入和輸出，可讓路由傳送規則更容易了解。 如需路由傳送規則以及模組之輸入和輸出端點的詳細資訊，請參閱[模組撰寫][lnk-module-comp]。

最後，使用下列系統屬性來為 Edge 中樞所處理的裝置到雲端訊息加上戳記：

| 屬性 | 說明 |
| -------- | ----------- |
| $connectionDeviceId | 傳送訊息之用戶端的裝置識別碼 |
| $connectionModuleId | 傳送訊息之模組的模組識別碼 |
| $inputName | 收到這則訊息的輸入。 可以是空的。 |
| $outputName | 用來傳送訊息的輸出。 可以是空的。 |

### <a name="connecting-to-iot-edge-hub-from-a-module"></a>從模組連線到 IoT Edge 中樞
從模組連線到本機 IoT Edge 中樞包含兩個步驟：當您的模組啟動時，使用 IoT Edge 執行階段所提供的連接字串，並確定您的應用程式會接受該裝置上 IoT Edge 中樞所呈現的憑證。

要使用的連接字串會由 IoT Edge 執行階段插入環境變數 `EdgeHubConnectionString` 中。 這使得任何想使用它的程式都能使用。

類似情況是，IoT Edge 執行階段會在可於環境變數 `EdgeModuleCACertificateFile` 中取得其路徑的檔案中，插入用來驗證 IoT Edge 中樞連線的憑證。

[開發 IoT Edge 模組並部署至模擬裝置][lnk-tutorial2]教學課程示範如何確定憑證位於您模組應用程式的電腦存放區中。 顯然使用該憑證來信任連線的任何其他方法都適用。

## <a name="packaging-as-an-image"></a>封裝為映像
IoT Edge 模組會封裝為 Docker 映像。
您可以直接使用 Docker 工具鏈，或[開發 IoT Edge 模組並部署至模擬裝置][lnk-tutorial2]教學課程中所示的 Visual Studio Code。

## <a name="next-steps"></a>後續步驟

當您開發模組之後，請了解如何[大範圍部署和監視 IoT Edge 模組][lnk-howto-deploy]。

[lnk-devicesdk]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-devicetwin]: ../iot-hub/iot-hub-devguide-device-twins.md
[lnk-iothub-messaging]: ../iot-hub/iot-hub-devguide-messaging.md
[lnk-methods]: ../iot-hub/iot-hub-devguide-direct-methods.md
[lnk-tutorial2]: tutorial-csharp-module.md
[lnk-module-comp]: module-composition.md
[lnk-howto-deploy]: how-to-deploy-monitor.md
