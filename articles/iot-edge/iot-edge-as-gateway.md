---
title: 使用 Azure IoT Edge 裝置作為閘道 | Microsoft Docs
description: 使用 Azure IoT Edge 來建立透明、不透明或 Proxy 閘道裝置，將資料從多個下游裝置傳送到雲端或在本機處理。
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 09/21/2017
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: e1825bcdd8dbb06ef027919416b2d0532a7df9c2
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47422825"
---
# <a name="how-an-iot-edge-device-can-be-used-as-a-gateway"></a>如何使用 IoT Edge 裝置作為閘道

IoT 解決方案中的閘道用途為解決方案所特有，結合了裝置連線能力與邊緣分析。 不論 Azure IoT Edge 是與連線能力、身分識別或邊緣分析相關，都可用來滿足 IoT 閘道的所有需求。 本文中的閘道模式只是指下游裝置連線能力和裝置身分識別的特性，而非在閘道上處理裝置資料的方式。

## <a name="patterns"></a>模式
使用 IoT Edge 裝置作為閘道的模式有三種：透明、通訊協定轉譯及身分識別轉譯：
* **透明** – 理論上可連線到 IoT 中樞的裝置可以改為連線到閘道裝置。 這表示下游裝置有自己的 IoT 中樞身分識別，而且使用任何 MQTT、AMQP 或 HTTP 通訊協定。 閘道只會傳遞裝置與 IoT 中樞之間的通訊。 裝置並未察覺它們是透過閘道來與雲端通訊，而與 IoT 中樞中的裝置互動的使用者則未察覺中繼閘道裝置。 因此，閘道是透明的。 請參閱[建立透明閘道][ lnk-iot-edge-as-transparent-gateway]作法，以了解使用 IoT Edge 裝置作為透明閘道的詳情。
* **通訊協定轉譯** – 又稱為不透明閘道模式；不支援 MQTT、AMQP 或 HTTP 的裝置會使用閘道裝置將資料傳送到 IoT 中樞。 閘道很聰明，足以了解下游裝置所使用的通訊協定；不過，它是在 IoT 中樞唯一具有身分識別的裝置。 所有資訊看似來自一部裝置 (也就是閘道)。 這意味著，如果雲端應用程式想要推論每個裝置的資料，下游裝置必須在其訊息中內嵌其他身分識別資訊。 此外，IoT 中樞基本元件如同對應項，而方法僅適用於閘道裝置，而不適用於下游裝置。
* **身分識別轉譯** - 無法連線到 IoT 中樞的裝置會連線到閘道裝置，以代表下游裝置提供 IoT 中樞身分識別和通訊協定轉譯。 閘道很聰明，足以了解下游裝置所使用的通訊協定，可提供這些裝置的身分識別，以及轉譯 IoT 中樞基本元件。 下游裝置會出現於 IoT 中樞，成為具有對應項和方法的第一級裝置。 使用者可與 IoT 中樞中的裝置互動，且並未察覺中繼閘道裝置。

![閘道模式的圖表][1]

## <a name="use-cases"></a>使用案例
所有閘道模式均可提供下列優點：
* **邊緣分析** – 使用 AI 服務在本機處理來自下游裝置的資料，而不需將完整精確度遙測資料傳送至雲端。 在本機尋找及回應深入解析，而且只將部分資料傳送到 IoT 中樞。 
* **下游裝置隔離** – 閘道裝置可以防護所有下游裝置，以免暴露於網際網路中。 它可位於沒有連線能力的 OT 網路與可供存取 Web 的 IT 網路之間。 
* **連線多工作業** - 所有透過 IoT Edge 裝置連線至 IoT 中樞的裝置將會使用相同基礎連線。
* **流量平滑化** - 如果 IoT 中樞發生節流，IoT Edge 裝置會自動實作指數倒回 (exponential backoff)，同時在本機保存訊息。 這將使您的方案恢復到流量尖峰。
* **有限的離線支援** - 閘道裝置會在本機儲存無法傳遞到 IoT 中樞的訊息和對應項更新。

進行通訊協定轉譯的閘道也可以對資源受限的現有裝置和新裝置，執行邊緣分析、裝置隔離、流量平滑化和離線支援。 許多現有裝置會產生可增強商業深入解析的資料，不過其設計並未考量雲端連線能力。 不透明閘道允許此資料在端對端 IoT 解決方案中解除鎖定並使用。

進行身分識別轉譯的閘道可提供通訊協定轉譯的優點，此外還可從雲端進行下游裝置的完整管理。 不論 IoT 解決方案中的所有裝置使用的通訊協定為何，全都會出現在 IoT 中樞中。

## <a name="cheat-sheet"></a>功能提要
以下的速查表提供使用透明、不透明 (通訊協定) 和 Proxy 閘道時的 IoT 中樞基本項目比較。

| &nbsp; | 透明閘道 | 通訊協定翻譯 | 身分識別協定翻譯 |
|--------|-------------|--------|--------|
| 儲存在 IoT 中樞身分識別登錄中的身分識別 | 所有已連線裝置的身分識別 | 僅限閘道裝置的身分識別 | 所有已連線裝置的身分識別 |
| 裝置對應項 | 每個已連線的裝置都有自己的裝置對應項 | 只有閘道有裝置和模組對應項 | 每個已連線的裝置都有自己的裝置對應項 |
| 直接方法及雲端到裝置訊息 | 雲端可以個別處理每個連線的裝置 | 雲端只可以處理閘道裝置 | 雲端可以個別處理每個連線的裝置 |
| [IoT 中樞節流與配額][lnk-iothub-throttles-quotas] | 套用至每個裝置 | 套用至閘道裝置 | 套用至每個裝置 |

使用不透明的閘道 (通訊協定轉譯) 模式時，所有透過該閘道連線的裝置會共用相同的雲端到裝置佇列，其中可以包含最多 50 則訊息。 因此，不透明閘道模式應僅在少數裝置透過每個在地閘道 (field gateway) 連線，及其雲端到裝置流量較低時使用。

## <a name="next-steps"></a>後續步驟
使用 IoT Edge 裝置作為[透明閘道][lnk-iot-edge-as-transparent-gateway] 

[lnk-iot-edge-as-transparent-gateway]: ./how-to-create-transparent-gateway-linux.md
[lnk-iothub-throttles-quotas]: ../iot-hub/iot-hub-devguide-quotas-throttling.md

[1]: ./media/iot-edge-as-gateway/edge-as-gateway.png
