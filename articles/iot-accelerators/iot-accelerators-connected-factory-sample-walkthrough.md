---
title: 連線的處理站解決方案的逐步解說 - Azure | Microsoft Docs
description: 說明 Azure IoT 解決方案加速器連線處理站及其架構。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: dobett
ms.openlocfilehash: ae5218bae12b9489d67b0264f0e5fdb6d833cb9e
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39187762"
---
# <a name="connected-factory-solution-accelerator-walkthrough"></a>連線的處理站解決方案加速器的逐步解說

連線的處理站[解決方案加速器][lnk-preconfigured-solutions]是端對端企業解決方案的實作，能夠︰

* 連線至模擬處理站生產線中執行 OPC UA 的模擬工業裝置，和實際的 OPC UA 伺服器裝置。 如需 OPC UA 的詳細資訊，請參閱[連線的處理站常見問題集](iot-accelerators-faq-cf.md)。
* 顯示作業 KPI 和這些裝置與生產線的 OEE。
* 示範如何將以雲端為基礎的應用程式用來與 OPC UA 伺服器系統進行互動。
* 可讓您連線到自己的 OPC UA 伺服器裝置。
* 可讓您瀏覽並修改 OPC UA 伺服器資料。
* 與 Azure Time Series Insights (TSI) 服務進行整合，可提供 OPC UA 伺服器資料的自訂檢視。

您可以將此解決方案做為自己實作的起點，並加以[自訂][lnk-customize]以符合自己特有的商務需求。

本文將逐步說明連線的處理站解決方案的一些重要元素，讓您了解其運作方式。 本文也描述資料如何流經此解決方案。 這項知識能協助您︰

* 在解決方案中進行疑難排解。
* 規劃如何自訂解決方案以滿足您的特定需求。
* 設計使用 Azure 服務的 IoT 解決方案。

如需詳細資訊，請參閱[連線的處理站常見問題集](iot-accelerators-faq-cf.md)。

## <a name="logical-architecture"></a>邏輯架構

下圖概述解決方案加速器的邏輯元件：

![連線處理站的邏輯架構][connected-factory-logical]

## <a name="communication-patterns"></a>通訊模式

此解決方案會使用 [OPC UA Pub/Sub 規格](https://opcfoundation.org/news/opc-foundation-news/opc-foundation-announces-support-of-publish-subscribe-for-opc-ua/)，以 JSON 格式將 OPC UA 遙測資料傳送到 IoT 中樞。 此解決方案會就此目的使用 [OPC 發行者](https://github.com/Azure/iot-edge-opc-publisher) IoT Edge 模組。

此解決方案也有整合至 Web 應用程式的 OPC UA 用戶端，該應用程式可以建立與內部部署 OPC UA 伺服器的連線。 用戶端會使用 [reverse-proxy](https://wikipedia.org/wiki/Reverse_proxy) 並接受 IoT 中樞的協助進行連線，而不需要內部部署防火牆中的開啟連接埠。 此通訊模式稱為[服務輔助通訊](https://blogs.msdn.microsoft.com/clemensv/2014/02/09/service-assisted-communication-for-connected-devices/)。 此解決方案會就此目的使用 [OPC Proxy](https://github.com/Azure/iot-edge-opc-proxy/) IoT Edge 模組。


## <a name="simulation"></a>模擬

模擬工作站與模擬製造執行系統 (MES) 會構成處理站生產線。 模擬裝置和 OPC 發行者模組是根據 OPC Foundation 所發佈的 [OPC UA .NET 標準][lnk-OPC-UA-NET-Standard]。

OPC Proxy 和 OPC 發行者會根據 [Azure IoT Edge][lnk-Azure-IoT-Gateway] 實作為模組。 每個模擬產品線都已連結指定的閘道。

所有模擬元件都是在裝載於 Azure Linux VM 的 Docker 容器中執行。 依預設，會將模擬設定為執行八個模擬生產線。

## <a name="simulated-production-line"></a>模擬生產線

生產線可製造零件。 它是由不同的工作站所組成︰組裝工作站、測試工作站和包裝工作站。

模擬所執行及更新的資料是透過 OPC UA 節點公開。 所有模擬生產線工作站都是透過 OPC UA 由 MES 進行協調。

## <a name="simulated-manufacturing-execution-system"></a>模擬製造執行系統

MES 會透過 OPC UA 來監視生產線中的每個工作站，從而偵測工作站的狀態變更。 它會呼叫 OPC UA 方法來控制工作站，並將產品從一個工作站傳遞到下一個工作站，直到完成為止。

## <a name="gateway-opc-publisher-module"></a>閘道 OPC 發行者模組

OPC 發行者模組會連線至工作站 OPC UA 伺服器，並訂閱所要發佈的 OPC 節點。 此模組會將節點資料轉換成 JSON 格式、將它加密，並傳送至 IoT 中樞作為 OPC UA Pub/Sub 訊息。

OPC 發行者模組只需要輸出的 https 連接埠 (443)，並且可以使用現有的企業基礎結構。

## <a name="gateway-opc-proxy-module"></a>閘道 OPC Proxy 模組

閘道 OPC UA Proxy 模組可建立二進位 OPC UA 命令與控制訊息的通道，且只需要輸出的 https 連接埠 (443)。 它可以使用現有的企業基礎結構，包括 Web Proxy。

它會使用 IoT 中樞裝置方法，將應用程式層中的已封包 TCP/IP 資料進行傳輸，因此可使用 SSL/TLS 來確保端點信任、資料加密及完整性。

透過 Proxy 本身轉送的 OPC UA 二進位通訊協定使用的是 UA 驗證及加密。

## <a name="azure-time-series-insights"></a>Azure Time Series Insights

閘道 OPC 發行者模組會訂閱 OPC UA 伺服器節點，以偵測資料值中的變更。 如果在其中一個節點中偵測到資料變更，此模組就會將訊息傳送至 Azure IoT 中樞。

IoT 中樞會向 Azure TSI 提供事件來源。 根據附加至訊息的時間戳記，TSI 會將資料儲存 30 天。 這項資料包括︰

* OPC UA ApplicationUri
* OPC UA NodeId
* 節點的值
* 來源時間戳記
* OPC UA DisplayName

目前，TSI 不允許客戶自訂想要保留資料的時間。

TSI 會使用 **SearchSpan** (**Time.From**、**Time.To**) 針對節點資料進行查詢，並依 **OPC UA ApplicationUri** 或 **OPC UA NodeId** 或 **OPC UA DisplayName** 進行彙總。

若要擷取 OEE 和 KPI 量測計的資料，以及時間序列圖表，則會依事件、總和、平均值、最小值和最大值的計數將資料進行彙總。

會使用不同的程序來建立時間序列。 OEE 和 KPI 是由工作站基底資料所計算出來，並針對應用程式中的拓撲 (生產線、處理站、企業) 進行反昇。

此外，每當顯示的時間範圍就緒時，就會在應用程式中計算 OEE 和 KPI 拓撲的時間序列。 例如，每個小時都會更新 [日] 檢視。

節點資料的時間序列檢視會直接來自於使用時間範圍彙總的 TSI。

## <a name="iot-hub"></a>IoT 中樞
[IoT 中樞][lnk-IoT Hub]會接收從 OPC 發行者模組傳送至雲端的資料，並提供給 Azure TSI 服務。 

解決方案中的 IoT 中樞也可︰
- 會將儲存所有 OPC 發行者模組和所有 OPC Proxy 模組的身分識別登錄加以維護。
- 用來作為 OPC Proxy 模組雙向通訊的傳輸通道。

## <a name="azure-storage"></a>Azure 儲存體
解決方案會使用 Azure Blob 儲存體作為 VM 的磁碟儲存體，及用來儲存部署資料。

## <a name="web-app"></a>Web 應用程式
部署作為解決方案加速器一部分的 Web 應用程式，包含整合式 OPC UA 用戶端、警示處理和遙測視覺效果。

## <a name="telemetry-data-flow"></a>遙測資料流程

![遙測資料流程](./media/iot-accelerators-connected-factory-sample-walkthrough/telemetry_dataflow.png)

### <a name="flow-steps"></a>流程步驟

1. OPC 發行者讀取來自本機憑證存放區的必要 OPC UA X 509 憑證和 IoT 中樞安全性認證。
    - 如有必要，OPC 發行者會建立憑證存放區，並將任何遺失的憑證或認證儲存在其中。

2. OPC 發行者向 IoT 中樞註冊其本身。
    - 使用設定的通訊協定。 可以使用任何 IoT 中樞用戶端 SDK 支援的通訊協定。 預設為 MQTT。
    - 通訊協定通訊受到 TLS 保護。

3. OPC 發行者會讀取組態檔。

4. OPC 發行者會以每個設定的 OPC UA 伺服器建立 OPC 工作階段。
    - 使用 TCP 連線。
    - OPC 發行者和 OPC UA 伺服器使用 X509 憑證相互驗證。
    - 所有進一步的 OPC UA 流量是由已設定的 OPC UA 加密機制加密。
    - OPC 發行者會在每個設定的發佈間隔之 OPC 工作階段中，建立 OPC 訂用帳戶。
    - 建立 OPC 節點的 OPC 監視項目，以在 OPC 訂用帳戶中發佈。

5. 如果受監視的 OPC 節點值變更，OPC UA 伺服器會將更新傳送給 OPC 發行者。

6. OPC 發行者會轉碼新值。
    - 如果已啟用批次處理，則會批次處理多項變更。
    - 建立 IoT 中樞訊息。

7. OPC 發行者會將訊息傳送到 IoT 中樞。
    - 使用設定的通訊協定。
    - 通訊受到 TLS 保護。

8. Time Series Insights (TSI) 會讀取 IoT 中樞的訊息。
    - 使用 AMQP over TCP/TLS。
    - 這個步驟是在資料中心內部。

9. TSI 中的待用資料。

10. Azure AppService 中連線的處理站 WebApp 從 TSI 查詢所需的資料。
    - 使用 TCP/TLS 安全通訊。
    - 這個步驟是在資料中心內部。

11. 網頁瀏覽器連線至連線的處理站 WebApp。
    - 轉譯連線的處理站儀表板。
    - 透過 HTTPS 連線。
    - 存取連線的處理站應用程式時必須透過 Azure Active Directory 來驗證使用者。
    - 對於連線的處理站應用程式的任何 WebApi 呼叫，都受到防偽權杖的保護。

12. 在資料更新時，連線的處理站 WebApp 會將更新資料傳送至網頁瀏覽器。
    - 使用 SignalR 通訊協定。
    - 受到 TCP/TLS 保護。

## <a name="browsing-data-flow"></a>瀏覽資料流程

![瀏覽資料流程](./media/iot-accelerators-connected-factory-sample-walkthrough/browsing_dataflow.png)

### <a name="flow-steps"></a>流程步驟

1. OPC Proxy (伺服器元件) 會啟動。
    - 從本機存放區讀取共用存取金鑰。
    - 如有必要，將遺失的存取金鑰儲存在存放區。

2. OPC Proxy (伺服器元件) 會向 IoT 中樞註冊其本身。
    - 從 IoT 中樞讀取所有已知裝置。
    - 使用 MQTT over TLS over Socket 或 Secure Websocket。

3. 網頁瀏覽器會連線至連線的處理站 WebApp，並且轉譯連線的處理站儀表板。
    - 使用 HTTPS。
    - 使用者會選取要連線的 OPC UA 伺服器。

4. 連線的處理站 WebApp 會對選取的 OPC UA 伺服器建立 OPC UA 工作階段。
    - 使用 OPC UA 堆疊。

5. OPC Proxy 傳輸會收到來自 OPC UA 堆疊的要求，以建立與 OPC UA 伺服器的 TCP 通訊端連線。
    - 它只會擷取 TCP 裝載，並且保持原狀使用它。
    - 這是連線的處理站 WebApp 內部的步驟。

6. OPC Proxy (用戶端元件) 會查閱 IoT 中樞裝置登錄中的 OPC Proxy (伺服器元件) 裝置。 然後呼叫 IoT 中樞中 OPC Proxy (伺服器元件) 裝置的裝置方法。
    - 使用 HTTPS over TCP/TLS 來查閱 OPC Proxy。
    - 使用 HTTPS over TCP/TLS，建立與 OPC UA 伺服器的 TCP 通訊端連線。
    - 這個步驟是在資料中心內部。

7. IoT 中樞會呼叫 OPC Proxy (伺服器元件) 裝置的裝置方法。
    - 使用已建立的 MQTT over TLS over Socket 或 Secure Websocket 連線，以建立與 OPC UA 伺服器的 TCP 通訊端連線。

8. OPC Proxy (伺服器元件) 會將 TCP 裝載傳送至工廠網路。

9. OPC UA 伺服器會處理裝載，並且傳回回應。

10. OPC Proxy (伺服器元件) 的通訊端會收到回應。
    - OPC Proxy 會以裝置方法的傳回值將資料傳送到 IoT 中樞與 OPC Proxy (用戶端元件)。
    - 這項資料會傳遞到連線的處理站應用程式中的 OPC UA 堆疊。

11. 連線的處理站 WebApp 會將接收自 OPC UA 伺服器、以 OPC UA 特定資訊增補的 OPC 瀏覽器 UX 傳回至網頁瀏覽器，加以轉譯。
    - 在瀏覽 OPC 位址空間以及將函式套用至 OPC 位址空間中的節點時，OPC 瀏覽器 UX 用戶端組件會透過以防偽權杖保護的 HTTPS 使用 AJAX 呼叫，從連線的處理站 WebApp 取得資料。
    - 必要時，用戶端會使用步驟 4 到 10 所述的通訊，與 OPC UA 伺服器交換資訊。

> [!NOTE]
> OPC Proxy (伺服器元件) 和 OPC Proxy (用戶端) 元件會針對與 OPC UA 通訊有關的所有 TCP 流量，完成步驟 #4 到 #10。

> [!NOTE]
> 對於連線的處理站 WebApp 內的 OPC UA 伺服器和 OPC UA 堆疊，OPC Proxy 通訊是透明的，且會套用所有適用於驗證和加密的 OPC UA 安全性功能。

## <a name="next-steps"></a>後續步驟

您可以繼續閱讀下列文章，了解如何開始使用 IoT 解決方案加速器︰

* [azureiotsuite.com 網站的權限][lnk-permissions]
* [在 Windows 或 Linux 上部署連線的處理站解決方案加速器的閘道](iot-accelerators-connected-factory-gateway-deployment.md)
* [OPC 發行者參考實作](https://github.com/Azure/iot-edge-opc-publisher/blob/master/README.md)。

[connected-factory-logical]:media/iot-accelerators-connected-factory-sample-walkthrough/cf-logical-architecture.png

[lnk-preconfigured-solutions]:about-iot-accelerators.md
[lnk-customize]: iot-accelerators-connected-factory-customize.md
[lnk-IoT Hub]: https://azure.microsoft.com/documentation/services/iot-hub/
[lnk-direct-methods]: ../iot-hub/iot-hub-devguide-direct-methods.md
[lnk-OPC-UA-NET-Standard]:https://github.com/OPCFoundation/UA-.NETStandardLibrary
[lnk-Azure-IoT-Gateway]: https://github.com/azure/iot-edge
[lnk-permissions]: iot-accelerators-faq.md
