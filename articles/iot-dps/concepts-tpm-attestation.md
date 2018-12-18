---
title: Azure IoT 中樞裝置佈建服務 - TPM 證明
description: 本文將在概念上略述使用 IoT 中樞裝置佈建服務時的 TPM 證明流程。
author: nberdy
ms.author: nberdy
ms.date: 04/23/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: briz
ms.openlocfilehash: cb763327eb292feb9d58fb21b1ca808a3f2909aa
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42141293"
---
# <a name="tpm-attestation"></a>TPM 證明

IoT 中樞裝置佈建服務是 IoT 中樞適用的協助程式服務，用於設定在指定 IoT 中樞上的全自動佈建裝置作業。 這項裝置佈建服務可以讓您以安全的方式佈建數百萬個裝置。

本文將說明使用 [TPM](./concepts-device.md) 時的識別證明程序。 TPM 代表的是「信賴平台模組 (Trusted Platform Module)」，而且是一種硬體安全模組 (HSM)。 本文假設您使用的是個別、韌體或整合式 TPM。 模擬軟體的 TPM 非常適合用來建立原型或測試，但是其不提供與個別、韌體或整合式 TPM 相同的安全性等級。 我們不建議將軟體 TPM 用在生產環境中。 如需 TPM 類型的詳細資訊，請參閱 [TPM 簡短介紹](http://trustedcomputinggroup.org/wp-content/uploads/TPM-2.0-A-Brief-Introduction.pdf)。

本文僅與使用 TPM 2.0 及具有 HMAC 金鑰支援的裝置和其簽署金鑰有關。 不適用於使用 X.509 憑證進行驗證的裝置。 在信賴運算群組中，TPM 是業界廣泛使用的 ISO 標準，您可以在[完整 TPM 2.0 規格](https://trustedcomputinggroup.org/tpm-library-specification/)或 [ISO/IEC 11889 規格](https://www.iso.org/standard/66510.html)中深入了解 TPM。本文也假設您熟悉公用和私密金鑰組，以及其用於加密的方式。

裝置佈建服務的裝置 SDK 會為您處理本文所述的所有項目。 如果您在裝置上使用 SDK，那麼您就不需要實作其他任何項目。 本文可協助您在概念上了解 TPM 安全性晶片在裝置佈建時的作用，以及為什麼它如此安全。

## <a name="overview"></a>概觀

TPM 會使用所謂的簽署金鑰 (EK) 作為安全的信任根。 EK 對 TPM 而言是唯一的，而且改變 EK 等於是從本質上將裝置變成一個新的裝置。

TPM 有另一種類型的金鑰，稱為儲存根金鑰 (SRK)。 TPM 的擁有者可以在取得 TPM 擁有權之後產生 SRK。 取得 TPM 擁有權是 TPM 專屬的說法，意思是「有人在 HSM 上設定密碼」。 如果將 TPM 裝置賣給新的擁有者，則新的擁有者即可取得 TPM 擁有權來產生新 SRK。 產生新的 SRK 可確保先前的擁有者無法使用 TPM。 因為 SRK 是 TPM 擁有者獨有的，所以該擁有者可使用 SRK 將資料密封到 TPM 本身。 SRK 會提供沙箱讓擁有者儲存金鑰，並在裝置或 TPM 售出時提供存取權撤銷功能。 這就像是搬到新家：取得擁有權就是變更門鎖，並破壞先前擁有者留下的所有家俱 (SRK)，但是您無法變更房子的地址 (EK)。

裝置一旦設定好且可供使用後，它也會讓 EK 和 SRK 可供使用。

![取得 TPM 擁有權](./media/concepts-tpm-attestation/tpm-ownership.png)

關於取得 TPM 擁有權，有一點需要注意：能否取得 TPM 擁有權取決於很多事情，包括 TPM 製造商、正在使用的 TPM 工具集，以及裝置作業系統。 請遵循適用於您系統的指示來取得擁有權。

裝置佈建服務會使用 EK (EK_pub) 的公開部分來識別和註冊裝置。 裝置廠商可以在製造或最終測試期間讀取 EK_pub，並將 EK_pub 上傳至佈建服務，以便裝置連線到佈建時，佈建可辨識出此裝置。 裝置佈建服務不會檢查 SRK 或擁有者，因此「清除」TPM 會清除客戶資料，但會保留 EK (和其他廠商資料)，而且裝置佈建服務仍可在裝置連線至佈建時辨識出裝置。

## <a name="detailed-attestation-process"></a>詳細的證明程序

當具有 TPM 的裝置第一次連線到裝置佈建服務時，服務會先根據儲存在註冊清單中的 EK_pub 來檢查提供的 EK_pub。 如果 EK_pub 不相符，則裝置不會允許進行佈建。 如果 EK_pub 是相符的，則服務會要求裝置透過 nonce 查問 (用來證明身分識別的安全查問)，來證明其具有 EK 私密部分的擁有權。 裝置佈建服務會產生 nonce，然後依序使用 SRK 和 EK_pub 對其加密，這兩者皆是在初始註冊呼叫期間由裝置提供的。 TPM 會確保 EK 私密部分永遠是安全的。 這樣可防止仿冒，並可確保 SAS 權杖能安全地佈建到已授權的裝置。

讓我們來詳細地逐步解說證明程序。

### <a name="device-requests-an-iot-hub-assignment"></a>裝置要求 IoT 中樞指派

首先，裝置會連線到裝置佈建服務，並要求進行佈建。 在此情況下，裝置會將其註冊識別碼、識別碼範圍與來自 TPM 的 EK_pub 和 SRK_pub 提供給服務。 服務會將加密的 nonce 傳回給裝置，並要求裝置解密 nonce，然後使用它來簽署 SAS 權杖，以重新連線並完成佈建。

![裝置要求佈建](./media/concepts-tpm-attestation/step-one-request-provisioning.png)

### <a name="nonce-challenge"></a>Nonce 查問

裝置取得 nonce 並用 EK 和 SRK 的私密部分來將 nonce 解密到 TPM；nonce 加密的命令會將信任從 EK (此項目不會改變) 委派給 SRK (如果新擁有者取得 TPM 擁有權即可變更此項目)。

![解密 nonce](./media/concepts-tpm-attestation/step-two-nonce.png)

### <a name="validate-the-nonce-and-receive-credentials"></a>驗證 nonce 和接收認證

接著，裝置可以使用解密的 nonce 來簽署 SAS 權杖，並使用已簽署的 SAS 權仗來重新與裝置佈建服務建立連線。 完成 Nonce 查問後，服務就會允許裝置進行佈建。

![裝置會重新建立與裝置佈建服務的連線以驗證 EK 擁有權](./media/concepts-tpm-attestation/step-three-validation.png)

## <a name="next-steps"></a>後續步驟

現在，裝置會連線到 IoT 中樞，您可以放心地相信裝置的金鑰已安全地儲存。 現在，您知道裝置佈建服務如何使用 TPM 安全地驗證裝置的身分識別，若要深入了解，請參閱下列文章：

* [了解中自動佈建的所有概念](./concepts-auto-provisioning.md)
* [開始使用自動佈建](./quick-setup-auto-provision.md)，使用 SDK 來完成流程。
