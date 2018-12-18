---
title: Azure IoT 中樞調整 | Microsoft Docs
description: 如何調整 IoT 中樞以支援您預期的訊息輸送量和所需功能。 包含每個層級支援的輸送量和分區化選項的摘要。
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/02/2018
ms.author: kgremban
ms.openlocfilehash: 6ae0217ed4b8833eb42a4719a1f2525461f9dcdd
ms.sourcegitcommit: a1140e6b839ad79e454186ee95b01376233a1d1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43143643"
---
# <a name="choose-the-right-iot-hub-tier-for-your-solution"></a>選擇適合您解決方案的 IoT 中樞層

IoT 解決方案各不相同，因此 Azure IoT 中樞提供了幾個以定價和級別為基礎的選項。 本文旨在協助您評估 IoT 中樞需求。 如需有關 IoT 中樞層的定價資訊，請參閱 [IoT 中樞定價](https://azure.microsoft.com/pricing/details/iot-hub)。 

若要決定適合您解決方案的 IoT 中樞層，請問自己兩個問題：

**我打算使用哪些功能？**
Azure IoT 中樞提供基本和標準兩個層級，兩者所支援的功能數目不同。 如果您的 IoT 解決方案是以收集資料裝置再集中分析為基礎，則基本層可能最適合您。 如果您想要使用更進階的組態，以便從遠端控制 IoT 裝置或將您的某些工作負載分散到各個裝置本身，則請考慮使用標準層。 如需這兩個層級各自包含之功能的詳細明細，請繼續閱讀[基本和標準層](#basic-and-standard-tiers)。

**我打算每日移動多少資料？**
每個 IoT 中樞層均提供三種大小，並以在任何指定日所能處理的資料輸送量為基礎。 這三種大小會加上數字 1、2 和 3 以供識別。 例如，每單位的層級 1 IoT 中樞一天可以處理 40 萬個訊息，每單位的層級 3 則可處理 3 億個。 如需資料指引的詳細資訊，請繼續閱讀[訊息輸送量](#message-throughput)。

## <a name="basic-and-standard-tiers"></a>基本和標準層

標準層的 IoT 中樞可啟用所有功能，想要使用雙向通訊功能的 IoT 解決方案都必須使用此層級。 基本層則可啟用一小部分功能，可供只需要從裝置到雲端進行單向通訊的 IoT 解決方案使用。 這兩個層級均提供相同的安全性和驗證功能。

在建立 IoT 中樞後，您不需要中斷現有作業就可以從基本層升級到標準層。 如需詳細資訊，請參閱[如何升級 IoT 中樞](iot-hub-upgrade.md)。 請注意，基本層 IoT 中樞的分割區上限為 8，標準層的上限為 32。 大部分的 IoT 中樞只需要 4 個分割區。 分割區限制會在建立 IoT 中樞時選擇，就是裝置到雲端訊息數對同時閱讀這些訊息的人員數比例。 當您從基本層遷移到標準層時，這個值保持不變。 也請注意，每個 IoT 中樞只能選擇層次內的一個[版本](https://azure.microsoft.com/pricing/details/iot-hub/)類型。 例如，您建立的 IoT 中樞可以具有多個 S1 單位，但不能具有來自不同版本 (例如 S1 和 B3 或 S1 和 S2) 的混合單位。

| 功能 | 基本層 | 標準層 |
| ---------- | ---------- | ------------- |
| [裝置到雲端的遙測](iot-hub-devguide-messaging.md) | 是 | 是 |
| [每部裝置身分識別](iot-hub-devguide-identity-registry.md) | 是 | 是 |
| [訊息路由](iot-hub-devguide-messages-read-custom.md)和[事件格線整合](iot-hub-event-grid.md) | 是 | 是 |
| [HTTP、AMQP 和 MQTT 通訊協定](iot-hub-devguide-protocols.md) | 是 | 是 |
| [裝置佈建服務](../iot-dps/about-iot-dps.md) | 是 | 是 |
| [監視和診斷](iot-hub-monitor-resource-health.md) | 是 | 是 |
| [雲端到裝置傳訊](iot-hub-devguide-c2d-guidance.md) |   | 是 |
| [裝置對應項](iot-hub-devguide-device-twins.md)、[模組對應項](iot-hub-devguide-module-twins.md)和[裝置管理](iot-hub-device-management-overview.md) |   | 是 |
| [Azure IoT Edge](../iot-edge/about-iot-edge.md) |   | 是 |

IoT 中樞也會提供免費層供您測試和評估。 其具有標準層的所有功能，但允許的傳訊量有限。 您無法從免費層升級至基本或標準層。 

### <a name="iot-hub-rest-apis"></a>IoT 中樞 REST API

IoT 中樞的基本和標準層之間的支援功能差異，代表某些 API 呼叫無法與基本層中樞搭配運作。 下表顯示可用的 API： 

| API | 基本層 | 標準層 |
| --- | ---------- | ------------- |
| [刪除裝置](https://docs.microsoft.com/rest/api/iothub/service/deletedevice) | 是 | 是 |
| [取得裝置](https://docs.microsoft.com/rest/api/iothub/service/getdevice) | 是 | 是 |
| 刪除模組 | 是 | 是 |
| 取得模組 | 是 | 是 |
| [取得登錄統計資料](https://docs.microsoft.com/rest/api/iothub/service/getdeviceregistrystatistics) | 是 | 是 |
| [取得服務統計資料](https://docs.microsoft.com/rest/api/iothub/service/getservicestatistics) | 是 | 是 |
| [建立或更新裝置](https://docs.microsoft.com/rest/api/iothub/service/createorupdatedevice) | 是 | 是 |
| 放置模組 | 是 | 是 |
| [查詢 IoT 中樞](https://docs.microsoft.com/rest/api/iothub/service/queryiothub) | 是 | 是 |
| 查詢模組 | 是 | 是 |
| [建立檔案上傳 SAS URI](https://docs.microsoft.com/rest/api/iothub/device/createfileuploadsasuri) | 是 | 是 |
| [接收裝置繫結通知](https://docs.microsoft.com/rest/api/iothub/device/receivedeviceboundnotification) | 是 | 是 |
| [傳送裝置事件](https://docs.microsoft.com/rest/api/iothub/device/senddeviceevent) | 是 | 是 |
| 傳送模組事件 | 是 | 是 |
| [更新檔案上傳狀態](https://docs.microsoft.com/rest/api/iothub/device/updatefileuploadstatus) | 是 | 是 |
| [大量裝置作業](https://docs.microsoft.com/rest/api/iot-dps/deviceenrollment/bulkoperation) | 是，IoT Edge 功能除外 | 是 | 
| [清除命令佇列](https://docs.microsoft.com/rest/api/iothub/service/purgecommandqueue) |   | 是 |
| [取得裝置對應項](https://docs.microsoft.com/rest/api/iothub/service/gettwin) |   | 是 |
| 取得模組對應項 |   | 是 |
| [叫用裝置方法](https://docs.microsoft.com/rest/api/iothub/service/invokedevicemethod) |   | 是 |
| [更新裝置對應項](https://docs.microsoft.com/rest/api/iothub/service/updatetwin) |   | 是 | 
| 更新模組對應項 |   | 是 | 
| [放棄裝置繫結通知](https://docs.microsoft.com/rest/api/iothub/device/abandondeviceboundnotification) |   | 是 |
| [完成裝置繫結通知](https://docs.microsoft.com/rest/api/iothub/device/completedeviceboundnotification) |   | 是 |
| [取消作業](https://docs.microsoft.com/rest/api/iothub/service/canceljob) |   | 是 |
| [建立作業](https://docs.microsoft.com/rest/api/iothub/service/createjob) |   | 是 |
| [取得作業](https://docs.microsoft.com/rest/api/iothub/service/getjob) |   | 是 |
| [查詢作業](https://docs.microsoft.com/rest/api/iothub/service/queryjobs) |   | 是 |

## <a name="message-throughput"></a>訊息輸送量

若要評估每個單位的流量，最佳方式為調整 IoT 中樞解決方案的大小。 尤其要考慮以下類別的作業所需的尖峰輸送量：

* 裝置到雲端的訊息
* 雲端到裝置的訊息
* 身分識別登錄作業

流量的測量是以單位為基礎，而不是以中樞為基礎。 層級 1 或 2 的 IoT 中樞執行個體最多可以有 200 個相關聯的單位。 層級 3 的 IoT 中樞執行個體最多可以有 10 個單位。 在建立 IoT 中樞後，您不需要中斷現有作業，就可以變更單位數量或是在特定層的 1、2、3 大小之間遷移。 如需詳細資訊，請參閱[如何升級 IoT 中樞](iot-hub-upgrade.md)。

以每一層的流量功能為例，裝置到雲端訊息會遵循下列持續輸送量指引：

| 層 | 持續的輸送量 | 持續的傳送速率 |
| --- | --- | --- |
| B1, S1 |每個單位最多 1111 KB/分鐘<br/>(1.5 GB/天/單位) |每個單位平均 278 個訊息/分鐘<br/>(400,000 個訊息/天/單位) |
| B2, S2 |每個單位最多 16 MB/分鐘<br/>(22.8 GB/天/單位) |每個單位平均 4,167 個訊息/分鐘<br/>(600 萬個訊息/天/單位) |
| B3, S3 |每個單位最多 814 MB/分鐘<br/>(1144.4 GB/天/單位) |每個單位平均 208,333 個訊息/分鐘<br/>(3 億個訊息/天/單位) |

除了此輸送量資訊，請參閱 [IoT 中樞配額和節流][IoT Hub quotas and throttles]，並據以設計您的解決方案。

### <a name="identity-registry-operation-throughput"></a>身分識別登錄作業輸送量
大部分的 IoT 中樞識別登錄作業都與裝置佈建有關，應該都不是執行階段作業。

對於明顯暴增的效能數據，請參閱 [IoT 中樞配額和節流][IoT Hub quotas and throttles]。

## <a name="auto-scale"></a>自動調整規模
如果您即將達到 IoT 中樞上允許的訊息限制，可以使用這些[步驟來自動調整](https://azure.microsoft.com/resources/samples/iot-hub-dotnet-autoscale/)，以增加相同 IoT 中樞層的 IoT 中樞單位。

## <a name="sharding"></a>分區化
雖然單一 IoT 中樞可以擴充到數百萬個裝置，但有時候您的解決方案需要單一 IoT 中樞無法保證的特定效能特性。 在此情況下，您可以將裝置分割到多個 IoT 中樞。 多個 IoT 中心可減緩資料傳輸量暴增，並取得所需的輸送量或作業速率。

## <a name="next-steps"></a>後續步驟

* 如需 IoT 中樞功能和效能詳細資料的詳細資訊，請參閱 [IoT 中樞定價][link-pricing] 或 [IoT 中樞配額與節流][IoT Hub quotas and throttles]。
* 若要變更 IoT 中樞層，請遵循[升級 IoT 中樞](iot-hub-upgrade.md)內的步驟。

[lnk-pricing]: https://azure.microsoft.com/pricing/details/iot-hub
[IoT Hub quotas and throttles]: iot-hub-devguide-quotas-throttling.md

[lnk-devguide]: iot-hub-devguide.md
[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
