---
title: 了解 Azure IoT SDK | Microsoft Docs
description: 開發人員指南 - 可用來建置裝置應用程式和後端應用程式的各種 Azure IoT 裝置和服務 SDK 的相關資訊和連結。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: dobett
ms.openlocfilehash: 710de8021abfa5b1fc17491af6b8b9f2bdd3a19f
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42919032"
---
# <a name="understand-and-use-azure-iot-hub-sdks"></a>了解和使用 Azure IoT 中樞 SDK

有兩種類別的軟體開發套件 (SDK) 可與「IoT 中樞」搭配使用：

* **裝置 SDK** 可讓您建立在 IoT 裝置上執行的應用程式。 這些應用程式會將遙測傳送至您的 IoT 中樞，並選擇性地接收來自 IoT 中樞的訊息、作業、方法或對應項更新。

* **服務 SDK** 可讓您管理 IoT 中樞，並選擇性地傳送訊息、排程作業、叫用直接方法，或將所需的屬性更新傳送到 IoT 裝置。

[在此][lnk-benefits-blog]了解使用 Azure IoT SDK 進行開發的優點。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="azure-iot-device-sdks"></a>Azure IoT 裝置 SDK

Microsoft Azure IoT 裝置 SDK 包含有助於建置裝置和應用程式的程式碼，並使裝置和應用程式連接到 Azure IoT 中心服務，且由其進行管理。

適用於 .NET 的 Azure IoT 中樞裝置 SDK： 
* 從 [Nuget][lnk-nuget-csharp-device] 安裝
* [原始程式碼][lnk-dotnet-sdk]
* [API 參考][lnk-dotnet-ref]

適用於 C 的 Azure IoT 中樞裝置 SDK：為了可攜性和廣泛平台相容性而以 ANSI C (C99) 撰寫
* 從 [apt-get、MBED、Arduino IDE 或 Nuget][lnk-c-package] 安裝
* [原始程式碼][lnk-c-sdk]
* [API 參考][lnk-c-ref]

適用於 Java 的 Azure IoT 中樞裝置 SDK： 
* 新增至 [Maven][lnk-maven-device] 專案
* [原始程式碼][lnk-java-sdk]
* [API 參考][lnk-java-ref]

適用於 Node.js 的 Azure IoT 中樞裝置 SDK： 
* 從 [npm][lnk-npm-device] 安裝
* [原始程式碼][lnk-node-sdk]
* [API 參考][lnk-node-ref]

適用於 Python 的 Azure IoT 中樞裝置 SDK： 
* 從 [pip][lnk-pip-device] 安裝
* [原始程式碼][lnk-python-sdk]

適用於 iOS 的 Azure IoT 中樞裝置 SDK： 
* 從 [CocoaPod][lnk-cocoa-device] 安裝
* [範例][lnk-ios-sample]

> [!NOTE]
> 請參閱 GitHub 儲存機制中的讀我檔案，以取得使用語言和平台特定套件管理員在開發電腦上安裝二進位檔和相依項目的相關資訊。
> 
> 

### <a name="os-platform-and-hardware-compatibility"></a>作業系統平台和硬體相容性

您可以在此[文件](iot-hub-device-sdk-platform-support.md)中找到支援 SDK 的平台。
如需 SDK 與特定硬體裝置之相容性的詳細資訊，請參閱 [Azure IoT 認證裝置目錄][lnk-certified]或個別的存放庫。

## <a name="azure-iot-service-sdks"></a>Azure IoT 服務 SDK

Microsoft Azure IoT 服務 SDK 包含可協助建置應用程式的程式碼，這些應用程式可直接與「IoT 中樞」互動來管理裝置和安全性。

適用於 .NET 的 Azure IoT 中樞服務 SDK：
* 從 [Nuget][lnk-nuget-csharp-service] 下載
* [原始程式碼][lnk-dotnet-sdk]
* [API 參考][lnk-dotnet-service-ref]

適用於 Java 的 Azure IoT 中樞服務 SDK： 
* 新增至 [Maven][lnk-maven-service] 專案
* [原始程式碼][lnk-java-sdk]
* [API 參考][lnk-java-service-ref]

適用於 Node.js 的 Azure IoT 中樞服務 SDK： 
* 從 [npm][lnk-npm-service] 下載
* [原始程式碼][lnk-node-sdk]
* [API 參考][lnk-node-service-ref]

適用於 Python 的 Azure IoT 中樞服務 SDK： 
* 從 [pip][lnk-pip-service] 下載
* [原始程式碼][lnk-python-sdk]

適用於 C 的 Azure IoT 中樞服務 SDK： 
* 從 [apt-get、MBED、Arduino IDE 或 Nuget][lnk-c-package] 下載
* [原始程式碼][lnk-c-sdk]

適用於 iOS 的 Azure IoT 中樞服務 SDK： 
* 從 [CocoaPod][lnk-cocoa-service] 安裝
* [範例][lnk-ios-sample]

> [!NOTE]
> 請參閱 GitHub 儲存機制中的讀我檔案，以取得使用語言和平台特定套件管理員在開發電腦上安裝二進位檔和相依項目的相關資訊。



## <a name="next-steps"></a>後續步驟

Azure IoT SDK 也提供一組工具以協助開發：
* [iothub-diagnostics](https://github.com/Azure/iothub-diagnostics)：跨平台命令列工具，可協助診斷與 IoT 中樞連線相關的問題。
* [device-explorer](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer)：Windows 桌面應用程式，可連線至 IoT 中樞。

此 IoT 中樞開發人員指南中的其他參考主題包括︰

* [IoT 中樞端點][lnk-devguide-endpoints]
* [裝置對應項、作業和訊息路由的 IoT 中樞查詢語言][lnk-devguide-query]
* [配額和節流][lnk-devguide-quotas]
* [IoT 中樞的 MQTT 支援][lnk-devguide-mqtt]
* [IoT 中樞 REST API 參考][lnk-rest-ref]
* [Auzre IoT SDK 平台支援](iot-hub-device-sdk-platform-support.md)

<!-- Links and images -->

[lnk-c-sdk]: https://github.com/Azure/azure-iot-sdk-c
[lnk-dotnet-sdk]: https://github.com/Azure/azure-iot-sdk-csharp
[lnk-java-sdk]: https://github.com/Azure/azure-iot-sdk-java
[lnk-node-sdk]: https://github.com/Azure/azure-iot-sdk-node
[lnk-python-sdk]: https://github.com/Azure/azure-iot-sdk-python
[lnk-certified]: https://catalog.azureiotsuite.com/

[lnk-dotnet-ref]: https://docs.microsoft.com/dotnet/api/microsoft.azure.devices?view=azure-dotnet
[lnk-dotnet-service-ref]: https://docs.microsoft.com/dotnet/api/microsoft.azure.devices
[lnk-c-ref]: https://azure.github.io/azure-iot-sdk-c/index.html
[lnk-java-ref]: https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device
[lnk-node-ref]: https://docs.microsoft.com/javascript/api/azure-iot-device/?view=azure-iot-typescript-latest
[lnk-rest-ref]: https://docs.microsoft.com/rest/api/iothub/
[lnk-java-service-ref]: https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.service
[lnk-node-service-ref]: https://docs.microsoft.com/javascript/api/azure-iothub/?view=azure-iot-typescript-latest

[lnk-maven-device]: https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-device-sdk
[lnk-maven-service]: https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk
[lnk-npm-device]: https://www.npmjs.com/package/azure-iot-device
[lnk-npm-service]: https://www.npmjs.com/package/azure-iothub
[lnk-nuget-csharp-device]: https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/
[lnk-nuget-csharp-service]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-c-package]: https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md
[lnk-pip-device]: https://pypi.python.org/pypi/azure-iothub-device-client/
[lnk-pip-service]: https://pypi.python.org/pypi/azure-iothub-service-client/


[lnk-devguide-endpoints]: iot-hub-devguide-endpoints.md
[lnk-devguide-quotas]: iot-hub-devguide-quotas-throttling.md
[lnk-devguide-query]: iot-hub-devguide-query-language.md
[lnk-devguide-mqtt]: iot-hub-mqtt-support.md
[lnk-benefits-blog]: https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/
[lnk-cocoa-device]: https://cocoapods.org/pods/AzureIoTHubClient
[lnk-ios-sample]: https://github.com/Azure-Samples/azure-iot-samples-ios
[lnk-cocoa-service]: https://cocoapods.org/pods/AzureIoTHubServiceClient
