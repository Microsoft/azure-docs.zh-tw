---
title: IoT 中樞裝置佈建服務 - 自動佈建概念
description: 本文提供使用「IoT 裝置佈建服務」、「IoT 中樞」及用戶端 SDK 來進行之裝置自動佈建階段的概念性概觀。
author: wesmc7777
ms.author: wesmc
ms.date: 06/01/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 9a45f8269e0ca2bc4188016af0ace06831c762b6
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521273"
---
# <a name="auto-provisioning-concepts"></a>自動佈建概念

如[概觀](about-iot-dps.md)中所述，「裝置佈建服務」是一項協助程式服務，可將裝置以 Just-In-Time 方式佈建到 IoT 中樞，而不需要人為介入。 在成功佈建之後，裝置會直接與指定的「IoT 中樞」連線。 此程序稱為自動佈建，可為裝置提供一個現成的登錄和初始設定體驗。

## <a name="overview"></a>概觀

Azure IoT 自動佈建可分成三個階段：

1. **服務設定** -「Azure IoT 中樞」和「IoT 中樞裝置佈建服務」執行個體的一次性設定，可建立這些執行個體並在它們之間建立連結。

   > [!NOTE]
   > 不論您的 IoT 解決方案規模為何，即使您打算支援數百萬個裝置，這仍然是**一次性設定**。

2. **裝置註冊** - 讓「裝置佈建服務」執行個體知道未來會嘗試註冊之裝置的程序。 [註冊](concepts-service.md#enrollment)時會一併在佈建服務中設定裝置身分識別資訊，針對單一裝置，會採用「個別註冊」的形式，針對多個裝置，則會採用「群組註冊」的形式。 身分識別會根據已指定裝置使用的[證明機制](concepts-security.md#attestation-mechanism)，這可讓佈建服務在登錄期間證明裝置的真確性：

   - **TPM**：以「個別註冊」的形式設定，裝置身分識別會根據 TPM 註冊識別碼和公開簽署金鑰。 由於 TPM 是一個[規格](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/)，因此不論 TPM 實作 (硬體或軟體) 為何，服務只會依據此規格來證明。 如需有關 TPM 型證明的詳細資料，請參閱[裝置佈建：使用 TPM 進行身分識別證明](https://azure.microsoft.com/blog/device-provisioning-identity-attestation-with-tpm/) \(英文\)。 

   - **X509**：以「個別註冊」或「群組註冊」的形式設定，裝置身分識別會根據 X.509 數位憑證 (以 .pem 或 .cer 檔案的形式上傳至註冊)。

   > [!IMPORTANT]  
   > 雖然「硬體安全模組」(HSM) 不是使用「裝置佈建服務」的先決條件，但強烈建議您的裝置使用該模組來儲存敏感性裝置身分識別資訊，例如金鑰和 X.509 憑證。

3. **裝置登錄與設定** - 由登錄軟體在開機時所起始，此軟體是使用適合裝置和證明機制的「裝置佈建服務」用戶端 SDK 來建置的軟體。 此軟體會與佈建服務建立連線來驗證裝置，以及在「IoT 中樞」中進行後續的登錄。 登錄成功之後，系統會為裝置提供其「IoT 中樞」唯一裝置識別碼和連線資訊，讓它能夠提取其初始設定並開始遙測程序。 在生產環境中，此階段可能會在前兩個階段後的數周或數月才會發生。

## <a name="roles-and-operations"></a>角色和作業

上一節中所討論的階段可能會因生產實況 (例如製造時間、運送、海關流程等) 而跨數周或數月。此外，如果涉及各種實體，它們還可能讓活動跨多個角色執行。 本節將深入探討與每個階段相關的各種角色和作業，然後以循序圖說明流程。 

自動佈建也針對啟用證明機制方面，為裝置製造商設下需求。 製造作業的進行也可能不受自動佈建階段的時機影響，特別是在已經建立自動佈建之後才購得新裝置的情況下。

左邊目錄中提供了一系列快速入門，可透過實際操作體驗來協助說明自動佈建。 為了協助/簡化學習程序，會使用軟體來模擬實體裝置以進行註冊和登錄。 由於快速入門的模擬本質緣故，有些快速入門會要求您執行多個角色的作業，包括不存在之角色的作業。

| 角色 | 作業 | 說明 | 
|------| --------- | ------------| 
| 製造商 | 將身分識別和登錄 URL 編碼 | 根據所使用的證明機制，製造商需負責將裝置身分識別資訊及「裝置佈建服務」登錄 URL 編碼。<br><br>**快速入門**：由於裝置是模擬裝置，因此沒有「製造商」角色。 如需有關如何取得此資訊的詳細資料，請參閱「開發人員」角色，撰寫範例登錄應用程式的程式碼時會使用此角色。 |  
| | 提供裝置身分識別 | 作為裝置身分識別資訊的來源，製造商需負責向操作員 (或指定的代理人) 傳達此資訊，或透過 API 將直接向「裝置佈建服務」註冊此資訊。<br><br>**快速入門**：由於裝置是模擬裝置，因此沒有「製造商」角色。 如需有關如何取得裝置身分識別的詳細資料，請參閱「操作員」角色，在於「裝置佈建服務」執行個體中註冊模擬裝置時會使用此角色。 | 
| 運算子 | 設定自動佈建 | 此作業與自動佈建的第一個階段對應。<br><br>**快速入門**：您需執行「操作員」角色，在您的 Azure 訂用帳戶中設定「裝置佈建服務」和「IoT 中樞」執行個體。 | 設定自動佈建 |
|  | 註冊裝置身分識別 | 此作業與自動佈建的第二個階段對應。<br><br>**快速入門**：您需執行「操作員」角色，在您的「裝置佈建服務」執行個體中註冊模擬裝置。 裝置身分識別取決於快速入門中所模擬的證明方法 (TPM 或 X.509)。 如需了解證明詳細資料，請參閱「開發人員」角色。 | 
| 裝置佈建服務、<br>IoT 中樞 | \<所有作業\> | 不論是使用實體裝置進行的生產環境實作，還是使用模擬裝置進行的快速入門，都是透過您在 Azure 訂用帳戶中所設定的 IoT 服務來執行這些角色。 這些角色/作業完全相同，因為 IoT 服務並不在乎佈建的是實體裝置還是模擬裝置。 | 
| 開發人員 | 建置/部署登錄軟體 | 此作業與自動佈建的第三個階段對應。 「開發人員」需負責使用適當的 SDK 來建置登錄軟體並部署至裝置。<br><br>**快速入門**：您所建置的範例登錄應用程式會針對您選擇的平台/語言來模擬真實裝置，此應用程式會在您的工作站上執行 (而不會部署至實體裝置)。 此登錄應用程式所執行的作業，會與部署至實體裝置之應用程式所執行的作業相同。 您需指定證明方法 (TPM 或 X.509 憑證)，再加上「裝置佈建服務」執行個體的登錄 URL 及「識別碼範圍」。 裝置身分識別取決於執行階段的 SDK 證明邏輯，根據的是您所指定的方法： <ul><li>**TPM 證明** - 您的開發工作站會執行 [TPM 模擬器應用程式](how-to-use-sdk-tools.md#trusted-platform-module-tpm-simulator)。 一旦執行，就會使用個別的應用程式來擷取 TPM 的「簽署金鑰」和「註冊識別碼」，以用於註冊和裝置識別碼。 SDK 證明邏輯也會在登錄期間使用模擬器，以針對驗證和註冊驗證出示已簽署的 SAS 權杖。</li><li>**X509 證明** - 您需使用工具來[產生憑證](how-to-use-sdk-tools.md#x509-certificate-generator)。 產生憑證之後，您需建立要在註冊作業中使用的必要憑證檔。 SDK 證明邏輯也會在登錄期間使用憑證，以針對驗證和註冊驗證出示憑證。</li></ul> | 
| 裝置 | 開機和註冊 | 此作業與自動佈建的第三個階段對應，會由「開發人員」所建置的裝置登錄軟體來執行。 如需詳細資料，請參閱「開發人員」角色。 第一次開機時： <ol><li>應用程式會依據在開發期間所指定的全域 URL 和服務「識別碼範圍」，與「裝置佈建服務」執行個體連線。</li><li>連線之後，會依據在註冊期間所指定的證明方法和身分識別來驗證裝置。</li><li>驗證之後，就會向佈建服務執行個體所指定的「IoT 中樞」執行個體註冊裝置。</li><li>登錄成功時，會將唯一裝置識別碼和「IoT 中樞」端點傳回給登錄應用程式，以與「IoT 中樞」進行通訊。</li><li> 裝置可以從該處下拉其初始[裝置對應項](~/articles/iot-hub/iot-hub-devguide-device-twins.md)狀態來進行設定，並開始回報遙測資料的程序。</li></ol>**快速入門**：由於裝置是模擬裝置，因此登錄軟體會在您的開發工作站上執行。| 

下圖摘要說明裝置自動佈建期間的角色和作業順序：
<br><br>
[![裝置的自動佈建順序](./media/concepts-auto-provisioning/sequence-auto-provision-device-vs.png)](./media/concepts-auto-provisioning/sequence-auto-provision-device-vs.png#lightbox) 

> [!NOTE]
> 製造商也可以視需要使用「裝置佈建服務 API」(而不透過「操作員」) 來執行「註冊裝置身分識別」作業。 如需此排序及更多的詳細討論，請觀看[向 Azure IoT 進行全自動裝置登錄](https://myignite.microsoft.com/sessions/55087)影片 (從 41:00 標記開始)

## <a name="next-steps"></a>後續步驟

隨著您逐步進行對應的自動佈建快速入門，您可能會發現將本文加入書籤以作為參考點相當有幫助。 

請從最適合您管理工具喜好設定的「設定自動佈建」快速入門開始著手，這會引導您完成「服務設定」階段：

- [使用 Azure CLI 來設定自動佈建](quick-setup-auto-provision-cli.md)
- [使用 Azure 入口網站來設定自動佈建](quick-setup-auto-provision.md)
- [使用 Resource Manager 範本來設定自動佈建](quick-setup-auto-provision-rm.md)

接著，繼續進行適合您裝置證明機制和「裝置佈建服務」SDK/語言喜好設定的「自動佈建模擬裝置」快速入門。 在此快速入門中，您將逐步完成「裝置註冊」和「裝置登錄與設定」階段： 

|  | 模擬裝置證明機制 | 快速入門 SDK/語言 |  |
|--|--|--|--|
|  | 信賴平台模組 (TPM) | [C](quick-create-simulated-device.md)<br>[Java](quick-create-simulated-device-tpm-java.md)<br>[C#](quick-create-simulated-device-tpm-csharp.md)<br>[Python](quick-create-simulated-device-tpm-python.md) |  |
|  | X.509 憑證 | [C](quick-create-simulated-device-x509.md)<br>[Java](quick-create-simulated-device-x509-java.md)<br>[C#](quick-create-simulated-device-x509-csharp.md)<br>[Node.js](quick-create-simulated-device-x509-node.md)<br>[Python](quick-create-simulated-device-x509-python.md) |  |




