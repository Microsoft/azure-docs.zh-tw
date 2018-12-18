---
title: 包含檔案
description: 包含檔案
services: iot-suite
author: dominicbetts
ms.service: iot-suite
ms.topic: include
ms.date: 04/25/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: f27dbf20eab5866ac3061d888c17e970a1d197f7
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34666750"
---
# <a name="azure-and-the-internet-of-things"></a>Azure 和物聯網

歡迎使用 Microsoft Azure 與物聯網 (IoT)。 本文將說明雲端 IoT 解決方案的共同特性。 IoT 解決方案需要裝置 (可能數以百萬計) 與解決方案後端之間有安全、雙向的通訊。 例如，解決方案可能會使用自動化的預測性分析，以從裝置到雲端的事件串流中發掘出有用見解。

## <a name="iot-solution-architecture"></a>IoT 解決方案架構

下圖顯示典型 IoT 解決方案架構的重要元素。 此圖表與不限定於特定實作詳細資料 (如所使用的 Azure 服務和裝置作業系統)。 在此架構中，IoT 裝置會收集其傳送到雲端閘道的資料。 雲端閘道可讓資料供其他後端服務處理。 這些後端服務可以將資料提供給：

* 其他企業營運應用程式。
* 操作人員 (透過儀表板或其他簡報裝置)。

![IoT 解決方案架構][img-solution-architecture]

> [!NOTE]
> 如需 IoT 架構的深入討論，請參閱 [Microsoft Azure IoT 參考架構][lnk-refarch]。

### <a name="device-connectivity"></a>裝置連線能力

在 IoT 解決方案架構中，裝置通常會將遙測傳送至雲端進行儲存和處理。 例如，在預測性維護案例中，解決方案後端可能會使用感應器資料流，來判斷特定幫浦何時需要維護。 裝置也可以透過讀取來自雲端端點的訊息，以接收和回應雲端到裝置訊息。 在某些例子中，解決方案後端可能會傳送訊息給幫浦站的其他幫浦，以在維護應開始之前先重新路由流量。 此程序可確保維護工程師一到場時即可開始工作。

安全、可靠的裝置連線通常是 IoT 方案中的最大挑戰。 這是因為相較於其他用戶端 (例如瀏覽器和行動應用程式)，IoT 裝置有不同的特性。 具體來說，IoT 裝置有以下特性：

* 通常是無人操作的嵌入式系統 (不同於手機)。
* 可以部署於實體存取成本昂貴的遠端位置。
* 可能只能透過解決方案後端來存取。 沒有其他方式可與裝置互動。
* 能力和/或處理資源可能都有限。
* 網路連線能力可能不穩定、緩慢或昂貴。
* 可能需要使用專屬、自訂或業界特定的應用程式通訊協定。
* 可以使用一組大型常見的硬體和軟體平台來建立。

除了上述限制，IoT 解決方案還必須是可擴充、安全和可靠的。

根據不同通訊協定和網路可用性，裝置可以與雲端直接通訊或透過中繼閘道通訊。 IoT 架構通常會是這兩種通訊模式的混合。

### <a name="data-processing-and-analytics"></a>資料處理和分析

在最新的 IoT 解決方案中，資料處理可以在雲端或裝置端進行。 裝置端處理指的是「邊緣運算」。 選擇資料處理位置的決定因素如下：

* 網路條件。 如果裝置與雲端之間的頻寬有限，就需要進行更多邊緣處理。
* 回應時間。 如果要在裝置上執行幾乎需要即時處理的動作，可能比較適合在裝置本身中處理回應。 例如，機器人手臂需要在發生緊急狀況時停止。
* 法規環境。 某些資料無法傳送至雲端。

一般來說，邊緣和雲端中的資料處理皆包含下列功能：

* 接收大規模來自裝置的遙測資料，並判斷如何處理與儲存該資料。
* 分析遙測資料並提供見解 (即時或事後)。
* 將命令從雲端或閘道裝置傳送到特定裝置。

此外，IoT 雲端後端應提供：

* 裝置註冊功能，讓您可以：
    * 佈建裝置。
    * 控制哪些裝置可連接到您的基礎結構。
* 可讓您控制裝置狀態並監視其活動的裝置管理。

例如，在預測性維護案例中，雲端後端會儲存歷史遙測資料。 解決方案會使用此資料來識別特定幫浦上的潛在異常行為，以免真的發生問題。 使用資料分析，雲端後端就可以識別預防性解決方案要將命令傳回裝置，並採取矯正措施。 此程序會在裝置與雲端間產生自動回應迴圈，大幅提升解決方案的效率。

### <a name="presentation-and-business-connectivity"></a>簡報及商務連線

簡報及商務連線層可讓終端使用者與 IoT 解決方案及裝置互動。 其讓使用者能夠檢視和分析從他們的裝置所收集的資料。 這些檢視可以採用儀表板或 BI 報表的格式，以顯示歷程記錄資料及/或接近即時的資料。 例如，操作員可確認特定幫浦站的狀態，並查看系統產生的任何警示。 此層也可整合 IoT 解決方案與現有企業營運應用程式，以連結企業商務程序或工作流程。 比方說，預測性維護解決方案加速器能整合排程系統，便可在解決方案識別出需要維護的幫浦時，預約工程師到幫浦站檢查。

[img-solution-architecture]: ./media/iot-azure-and-iot/iot-reference-architecture.png
[img-dashboard]: ./media/iot-azure-and-iot/iot-suite.png

[lnk-iot-hub]: ../articles/iot-hub/iot-hub-what-is-iot-hub.md
[lnk-iot-suite]: ../articles/iot-accelerators/iot-accelerators-what-are-solution-accelerators.md
[lnk-machinelearning]: http://azure.microsoft.com/documentation/services/machine-learning/
[Azure IoT solution accelerators]: http://azure.microsoft.com/solutions/iot
[lnk-protocol-gateway]:  ../articles/iot-hub/iot-hub-protocol-gateway.md
[lnk-refarch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf
