---
title: 了解 Azure IoT Edge 憑證 | Microsoft Docs
description: 深入了解 Azure IoT Edge 憑證及其使用方式。
author: stevebus
manager: timlt
ms.author: stevebus
ms.date: 09/13/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: a782a7341e53420dbc31fefc86007951df967a9e
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46307977"
---
# <a name="azure-iot-edge-certificate-usage-detail"></a>Azure IoT Edge 憑證使用方式詳細資料

IoT Edge 憑證用於各種模組和下游 IoT 裝置上，可驗證其連線到的[ Edge 中樞](iot-edge-runtime.md#iot-edge-hub)執行階段模組的身分識別和合法性。 這類驗證作業可在執行階段、模組和 IoT 裝置之間建立 TLS (傳輸層安全性) 安全連線。 和 IoT 中樞本身一樣，IoT Edge 也需要 IoT 下游 (或分葉) 裝置和 IoT Edge 模組的安全加密連線。 為了建立安全的 TLS 連線，Edge 中樞模組會提供連線用戶端的伺服器信任鏈結，以便用戶端驗證其身分識別。

為了釐清 IoT Edge 使用的憑證，本文會說明 IoT Edge 憑證在大規模案例以及開發測試情境下的運作方式。 雖然指令碼不同 (Powershell 和 bash)，但在 Linux 和 Windows 上的概念皆相同。

## <a name="iot-edge-certificates"></a>IoT Edge 憑證

在大部分情況下，對於 Edge 裝置的終端使用者而言，製造商是獨立的實體。 有時候，製造商和營運商之間的唯一關聯性是市售 Edge 裝置的購買關係。 而有時候，製造商會簽訂合約，代表營運商組建 Edge 裝置。 IoT Edge 憑證在設計上嘗試將這兩種案例均納入考量。

下圖說明了 IoT Edge 使用憑證的情況。 根 CA 憑證與裝置 CA 憑證之間可能會有一個以上的中繼簽署憑證，也可能完全沒有。 根據參與的實體數量，在這裡先示範其中一種情況。

![一般憑證關聯性圖表](./media/iot-edge-certs/edgeCerts-general.png)

### <a name="certificate-authority"></a>憑證授權單位

憑證授權單位，簡稱 CA，是核發數位憑證的實體。 憑證授權單位是做為憑證擁有者和憑證接收者之間信任的第三方。 數位憑證會確認憑證接收者對公開金鑰的所有權。 憑證信任鏈結一開始會先核發根憑證，這是憑證授權單位核發的所有憑證獲得信任的基礎。 此後，擁有者即可使用根憑證核發其他中繼憑證 (「分葉」憑證)。

### <a name="root-ca-certificate"></a>根 CA 憑證

根 CA 憑證是整個程序的信任根憑證。 在生產案例中，通常是向信任的商業憑證授權單位 (例如 Baltimore、Verisign、DigiCert 等) 購買 CA 憑證。至於連線到您 Edge 裝置的裝置，如果您有完整控制權，就可以使用公司層級的憑證授權單位。 在上述任一情況下，Edge 中樞註冊的整個信任鏈結會積存至中樞，因此分葉 IoT 裝置必須信任根憑證。 您可以將根 CA 憑證儲存在信任的根憑證授權單位存放區中，或在 IoT 用戶端連接到 IoT Edge 時，於您的程式碼中提供根 CA 憑證的詳細資訊。

### <a name="intermediate-certificates"></a>中繼憑證

在建立安全裝置的一般製造程序中，很少直接使用根 CA 憑證，主要是因為有外洩或暴露的風險。 一般而言，會建立一個或多個中繼 CA 憑證，並由根 CA 憑證以數位方式簽署。 可能只會有一個中繼憑證，也可能會有由數個這類中繼憑證組成的鏈結。 需要中繼憑證鏈結的案例包括：

* 製造商的部門階層。

* 參與同款裝置生產流程的多家公司。

* 購買根 CA 並以此獲得簽署憑證的客戶，以提供給製造商簽署代客戶製造的裝置。

在任何情況下，製造商都會使用這個鏈結尾端的中繼 CA 憑證簽署終端裝置上的裝置 CA 憑證。 一般而言，這些中繼憑證會受到製造工廠嚴密保護。 這些憑證在實體和電子的使用上均會經過嚴格的流程。

### <a name="device-ca-certificate"></a>裝置 CA 憑證

裝置 CA 憑證是流程內最終的中繼 CA 憑證所產生和簽署的憑證。 此憑證會安裝在 Edge 裝置上，最好是放置在安全儲存體中，例如硬體安全性模組 (HSM)。 此外，裝置 CA 憑證可唯一識別出 IoT Edge 裝置。 對於 IoT Edge，裝置 CA 憑證可核發其他憑證。 例如，裝置 CA 憑證會核發分葉裝置憑證，用來向 [Azure IoT 裝置佈建服務](..\iot-dps\about-iot-dps.md)驗證裝置。

### <a name="iot-edge-workload-ca"></a>IoT Edge 工作負載 CA

此憑證是流程中「營運商」端的第一個憑證，會在 IoT Edge 第一次啟動時由 [IoT Edge 安全性管理員](iot-edge-security-manager.md) 產生。 此憑證是「裝置 CA 憑證」產生並簽署的憑證。 此憑證又是另一個中繼簽署憑證，用來產生和簽署 IoT Edge 執行階段所使用的其他所有憑證。 目前這主要是會在下節討論到的 Edge 中樞伺服器憑證，但未來可能會加入用來驗證 IoT Edge 元件的其他憑證。

### <a name="edge-hub-server-certificate"></a>Edge 中樞伺服器憑證

Edge 中樞伺服器憑證是提供給分葉裝置和模組的實際憑證，用於在建立 IoT Edge 所需的 TLS 連線期間進行身分識別驗證。 此憑證會提供簽署憑證的完整鏈結，用來向分葉 IoT 裝置必須信任的根 CA 憑證產生簽署憑證。 由 IoT Edge 安全性管理員產生此 Edge 中樞憑證時，憑證的一般名稱 (CN) 會設定為由 config.yaml 檔案中的「hostname」屬性提供名稱，並轉換成小寫。 這是與 IoT Edge 產生混淆的常見原因。

## <a name="production-implications"></a>生產環境影響

有個問題可能很有道理：「IoT Edge 為何需要『工作負載 CA』額外憑證？ 無法使用裝置 CA 憑證來直接產生 Edge 中樞伺服器憑證嗎？」 這在技術上可行。 不過，此「工作負載」的中繼憑證是用來區隔裝置製造商和裝置營運商彼此的考量。 假設一位客戶將 IoT Edge 裝置售予或轉讓予另一位客戶。 就可能會希望製造商提供的裝置 CA 憑證維持不變。 不過，應該抹除裝置作業特定的「工作負載」憑證，並針對新的部署重新建立該憑證。

由於製造商和營運商的流程彼此分隔，因此在生產裝置上推出 IoT Edge 時有些需要考量的影響因素。

* 若有任何以憑證為基礎的流程，在推出 IoT Edge 裝置的整個流程中，根 CA 憑證和所有的中繼 CA 憑證均應受到保護和監控。 IoT Edge 裝置製造商應要備有穩健的流程，以妥善儲存和使用中繼憑證。 此外，裝置 CA 憑證應盡可能留存在裝置的安全儲存體中，最好是硬體安全性模組。

* 因為Edge 中樞會向連線的用戶端裝置與模組提供 Edge 中樞伺服器憑證，因此裝置 CA 憑證的 CN **不可**與 IoT Edge 裝置上的 config.yaml 中使用的「主機名稱」相同。 用戶端用來連線到 IoT Edge 的名稱 (例如，透過連接字串的 GatewayHostName 參數或 MQTT 中的 CONNECT 命令) **不可**與裝置 CA 憑證中使用的一般名稱相同。 這是因為 Edge 中樞會提供整個憑證鏈結由用戶端驗證。 如果 Edge 中樞伺服器憑證和裝置 CA 憑證兩者都有相同的 CN 憑證，則會進入驗證迴圈中，而導致憑證無效。

* 因為 IoT Edge 安全性精靈會使用裝置 CA 憑證來產生最終 IoT Edge 憑證，因此裝置 CA 憑證必須是簽署憑證，也就是本身具有憑證簽署功能。 若要將之設定為簽署 CA 憑證，將「V3 Basic constraints CA:True」套用於裝置 CA 憑證後，基本上會自動設定必要的金鑰使用屬性。

>[!Tip]
> 如果您已經在開發/測試情況中使用我們的「便利性指令碼」 (請參閱下一節) 安裝 IoT Edge 做為透明的閘道，而且您在建立主機 CA 憑證時使用的主機名稱與 config.yaml 中的主機名稱相同，您可能會很好奇為何可正常運作。 為了簡化開發人員作業的過程，便利性指令碼會在您傳遞至指令碼的名稱後面附加「.ca」。 因此，比方說您指令碼中的裝置名稱和 config.yaml 中主機名稱都是「mygateway」，前者會轉換為 mygateway.ca，然後才會做為裝置 CA 憑證的 CN 使用。

## <a name="devtest-implications"></a>開發/測試影響

為了簡化開發和測試過程，Microsoft 提供了一組[便利性指令碼](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates)，用來產生透明閘道案例中試用 IoT Edge 的非生產憑證。

這些指令碼產生的憑證會遵循本文所述的憑證鏈結結構。 對於 [Linux](how-to-create-transparent-gateway-linux.md#certificate-creation) 或 [Windows](how-to-create-transparent-gateway-windows.md#certificate-creation)。 下列命令會產生「根 CA 憑證 」和單一「中繼 CA 憑證」。

```bash
./certGen.sh create_root_and_intermediate 
```

```Powershell
New-CACertsCertChain rsa 
```

同樣地，這些命令會產生「裝置 CA 憑證」。

```bash
./certGen.sh create_edge_device_certificate "<gateway device name>"
```

```Powershell
New-CACertsEdgeDevice "<gateway device name>"
```

* 傳入這些指令碼的**\<閘道裝置名稱\>****不可**與 config.yaml 中的「hostname」參數相同。 如上所述，如果使用者在兩個位置中使用相同的名稱設定 Edge，指令碼，則可將「.ca」字串附加到**\<閘道裝置名稱\>** 上來避免名稱衝突，以免發生問題。不過，最好避免使用相同的名稱。

>[!Tip]
> 若要將裝置的 IoT「分葉」裝置連線到透過 IoT Edge 使用我們的 IoT 裝置 SDK 的應用程式，必須將選擇性的「GatewayHostName」參數家到裝置連接字串的尾端。 產生 Edge 中樞伺服器憑證時，會以小寫的 config.yaml 主機名稱為基礎，因此，若要名稱相符並且成功通過 TLS 憑證驗證，請輸入小寫的 GatewayHostName 參數。

## <a name="example-of-iot-edge-certificate-hierarchy"></a>IoT Edge 憑證階層的範例

為了說明這個憑證路徑的範例，下列螢幕擷取畫面取自於設為透明閘道的運作中 IoT Edge 裝置。 OpenSSL 用來連線到 Edge 中樞、驗證和傾印出憑證。

![憑證階層各層級的螢幕擷取畫面](./media/iot-edge-certs/iotedge-cert-chain.png)

可以在螢幕擷取畫面看到憑證深度階層：

| 根 CA 憑證         | 僅限 Azure IoT 中樞 CA 憑證測試                                                                           |
|-----------------------------|-----------------------------------------------------------------------------------------------------------|
| 中繼 CA 憑證 | 僅限 Azure IoT 中樞中繼憑證測試                                                                 |
| 裝置 CA 憑證       | iotgateway.ca (「iotgateway」作為 <閘道器主機名稱> 傳入便利性指令碼)      |
| 工作負載 CA 憑證     | IoT Edge 工作負載 CA                                                                                       |
| Edge 中樞伺服器憑證 | iotedgegw.local (與 config.yaml 的「hostname」相符)                                                |

## <a name="next-steps"></a>後續步驟

[了解 Azure IoT Edge 模組](iot-edge-modules.md)

[建立作為透明閘道的 Linux IoT Edge 裝置](how-to-create-transparent-gateway-linux.md)

[建立作為透明閘道的 Windows IoT Edge 裝置](how-to-create-transparent-gateway-windows.md)
