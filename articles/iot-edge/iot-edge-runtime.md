---
title: 了解 Azure IoT Edge 執行階段 | Microsoft Docs
description: 了解 Azure IoT Edge 執行階段，以及如何有效運用 Edge 裝置
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 08/13/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 9c9c04a8310a46605cf5733131db1418b7cb7f7a
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47218779"
---
# <a name="understand-the-azure-iot-edge-runtime-and-its-architecture"></a>了解 Azure IoT Edge 執行階段和架構

IoT Edge 執行階段是一個程式集合，需安裝於要被視為 IoT Edge 裝置的裝置上。 整體而言，IoT Edge 執行階段的元件讓 IoT Edge 裝置能夠接收要在 Edge 上執行的程式碼並傳達結果。 

IoT Edge 執行階段會在 IoT Edge 裝置上執行下列功能：

* 在裝置上安裝和更新工作負載。
* 在裝置上維護 Azure IoT Edge 安全性標準。
* 確保 [IoT Edge 模組][lnk-modules] 一律執行中。
* 將模組健康情況報告至雲端，以便進行遠端監控。
* 促進下游分葉裝置與 IoT Edge 裝置之間的通訊。
* 促進 IoT Edge 裝置上模組之間的通訊。
* 促進 IoT Edge 裝置與雲端之間的通訊。

![IoT Edge 執行階段會將深入見解和模組健康情況傳達到 IoT 中樞][1]

IoT Edge 執行階段的責任分為兩類：通訊和模組管理。 這兩個角色都是透過組成 IoT Edge 執行階段的兩個元件來執行。 IoT Edge 中樞負責通訊，而 IoT Edge 代理程式會管理部署和監視模組。 

Edge 中樞和 Edge 代理程式都是模組，就像 IoT Edge 裝置上執行的任何其他模組。 

## <a name="iot-edge-hub"></a>IoT Edge 中樞

Edge 中樞是組成 Azure IoT Edge 執行階段的兩個模組之一。 它藉由公開與 IoT 中樞相同的通訊協定端點來作為 IoT 中樞的本機 Proxy。 此一致性表示用戶端 (不論是裝置或模組) 都可連線到 IoT Edge 執行階段，就像它們對 IoT 中樞所做的。 

>[!NOTE]
>Edge 中樞支援使用 MQTT 或 AMQP 連線的用戶端。 並不支援使用 HTTP 的用戶端。 

Edge 中樞不是在本機執行的 IoT 中樞完整版本。 Edge 中樞會以無訊息方式委派幾件事給 IoT 中樞。 例如，當裝置第一次嘗試連線時，Edge 中樞會將驗證要求轉送到 IoT 中樞。 建立第一個連線之後，Edge 中樞就會在本機快取安全性資訊。 允許後續來自該裝置的連線，而不需向雲端驗證。 

>[!NOTE]
>每次嘗試驗證裝置時，都必須連線執行階段。

為了降低 IoT Edge 解決方案所使用的頻寬，Edge 中樞會將要連至雲端的連線數目最佳化。 Edge 中樞會從模組或分葉裝置等用戶端取得邏輯連線，並針對單一實體連線來將它們合併到雲端。 此程序的詳細資料對解決方案的其餘部分而言是透明的。 用戶端認為它們有自己的連線可連至雲端，即使它們全部都會透過相同連線來傳送。 

![Edge 中樞會作為多個實體裝置與雲端之間的閘道][2]

Edge 中樞可以判斷是否已連線到 IoT 中樞。 如果連線中斷，Edge 中樞就會在本機儲存訊息或對應項更新。 一旦連線重新建立之後，就會將所有資料同步。 針對此暫時的快取所使用的位置取決於 Edge 中樞模組對應項的屬性。 快取的大小不會受限，而且只要裝置還有儲存體容量就會持續成長。 

### <a name="module-communication"></a>模組通訊

Edge 中樞可促進模組對模組的通訊。 使用 Edge 中樞作為訊息代理程式，讓模組彼此保持獨立。 模組只需指定它們要在其中接受訊息的輸入，以及要將訊息寫入其中的輸出。 解決方案開發人員接著會將這些輸入和輸出拼接在一起，讓模組能夠依該解決方案特定的順序來處理資料。 

![Edge 中樞可促進模組對模組的通訊][3]

為了將資料傳送至 Edge 中樞，模組會呼叫 SendEventAsync 方法。 第一個引數會指定在哪個輸出上傳送訊息。 下列虛擬程式碼會在 output1 上傳送訊息：

   ```csharp
   ModuleClient client = new ModuleClient.CreateFromEnvironmentAsync(transportSettings); 
   await client.OpenAsync(); 
   await client.SendEventAsync(“output1”, message); 
   ```

若要接收訊息，請註冊會處理來自特定輸入之訊息的回呼。 下列虛擬程式碼會註冊函式 messageProcessor，以用於處理 input1 上接收到的所有訊息：

   ```csharp
   await client.SetEventHandlerAsync(“input1”, messageProcessor, userContext);
   ```

解決方案開發人員會負責指定規則，以判斷 Edge 中樞在模組之間傳遞訊息的方式。 路由傳送規則會定義於雲端，並利用 Edge 中樞的裝置對應項往下推送到該中樞。 適用於 IoT 中樞路由的相同語法，可用來定義 Azure IoT Edge 中模組之間的路由。 

<!--- For more info on how to declare routes between modules, see []. --->   

![模組之間的路由][4]

## <a name="iot-edge-agent"></a>IoT Edge 代理程式

IoT Edge 代理程式是另一個組成 Azure IoT Edge 執行階段的模組。 它負責將模組具現化，確保它們會繼續執行，並將模組的狀態回報至 IoT 中樞。 就像任何其他模組，Edge 代理程式會使用其模組對應項來儲存此設定資料。 

[IoT Edge 安全性精靈](iot-edge-security-manager.md)會在裝置啟動時啟動 Edge 代理程式。 代理程式會從 IoT 中樞擷取其模組對應項，並檢查部署資訊清單。 部署資訊清單是 JSON 檔案，會宣告需要啟動的模組。 

部署資訊清單中的每個項目都包含關於模組的特定資訊，而 Edge 代理程式可用其來控制模組的生命週期。 以下提供一些更令人感興趣的屬性： 

* **settings.image**：Edge 代理程式用來啟動模組的容器映像。 如果映像受到密碼保護，Edge 代理程式就必須使用適用於容器登錄的認證來設定。 若要從遠端設定容器登錄的認證，您可以藉由使用部署資訊清單來進行，也可以在 Edge 裝置本身上藉由在 IoT Edge 方案資料夾中更新 `config.yaml` 檔案來進行。
* **settings.createOptions**：啟動容器模組時直接傳遞至 Docker 精靈的字串。 在這個屬性中新增 Docker 選項，可允許使用連接埠轉送或將磁碟區掛接到模組容器等進階選項。  
* **status**：Edge 代理程式放置模組的狀態。 這個值通常會設定為 *running*，因為大多數人都想要 Edge 代理程式在裝置上立即啟動所有模組。 不過，您可以指定要停止之模組的初始狀態，並等候一段時間，以告知 Edge 代理程式啟動模組。 Edge 代理程式會在回報的屬性中，將每個模組的狀態回報至雲端。 所需屬性和回報的屬性之間的差異可能表示裝置異常。 支援的狀態如下：
   * 正在下載
   * 執行中
   * 狀況不良
   * Failed
   * 已停止
* **restartPolicy**：Edge 代理程式重新啟動模組的方式。 可能的值包括：
   * Never：Edge 代理程式絕對不會重新啟動模組。
   * onFailure：如果模組損毀，Edge 代理程式就會重新啟動它。 如果模組完全關閉，Edge 代理程式就不會重新啟動它。
   * Unhealthy：如果模組毀損或被視為狀況不良，Edge 代理程式就會重新啟動它。
   * Always：如果模組損毀、被視為狀況不良，或以任何方式關閉，Edge 代理程式就會重新啟動它。 

IoT Edge 代理程式會將執行階段回應傳送到 IoT 中樞。 以下是可能回應的清單：
  * 200 - 確定
  * 400 - 部署設定不正確或無效。
  * 417 - 裝置沒有部署組態集。
  * 412 - 部署設定中的結構描述版本無效。
  * 406 - 邊緣裝置已離線或無法傳送狀態報表。
  * 500 - 邊緣執行階段中發生錯誤。

### <a name="security"></a>安全性

IoT Edge 代理程式在 IoT Edge 裝置的安全性中扮演了關鍵角色。 例如，它會執行像是啟動前先驗證模組映像的動作。 

如需 Azure IoT Edge 安全性架構的詳細資訊，請閱讀 [IoT Edge 安全性管理員](iot-edge-security-manager.md)

## <a name="next-steps"></a>後續步驟

[了解 Azure IoT Edge 認證][lnk-certs]

<!-- Images -->
[1]: ./media/iot-edge-runtime/Pipeline.png
[2]: ./media/iot-edge-runtime/Gateway.png
[3]: ./media/iot-edge-runtime/ModuleEndpoints.png
[4]: ./media/iot-edge-runtime/ModuleEndpointsWithRoutes.png

<!-- Links -->
[lnk-certs]: iot-edge-certs.md
