---
title: Azure IoT 中樞裝置佈建服務的安全性概念 | Microsoft Docs
description: 說明具有裝置佈建服務和 IoT 中樞之裝置的特定安全性佈建概念
author: nberdy
ms.author: nberdy
ms.date: 03/30/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: briz
ms.openlocfilehash: 92a30f0754decc3052bf53a64da13325ddc4f954
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946556"
---
# <a name="iot-hub-device-provisioning-service-security-concepts"></a>IoT 中樞裝置佈建服務的安全性概念 

IoT 中樞裝置佈建服務是 IoT 中樞適用的協助程式服務，用於設定在指定 IoT 中樞上的全自動佈建裝置作業。 這項裝置佈建服務可以讓您以安全且可調整的方式[自動佈建](concepts-auto-provisioning.md)數百萬個裝置。 本文提供關於裝置佈建*安全性*概念的概觀。 本文與為裝置進行部署準備工作的所有角色相關。

## <a name="attestation-mechanism"></a>證明機制

證明機制是確認裝置身分識別的方法。 證明機制也與申請清單相關，該清單會讓佈建服務知道各裝置對應的證明方法。

> [!NOTE]
> IoT 中樞會針對該服務中的類似概念使用「驗證配置」。

裝置佈建服務支援下列形式的證明：
* 以標準 X.509 憑證驗證流程為基礎的 **X.509 憑證**。
* **信賴平台模組 (TPM)** 是以 nonce 挑戰為基礎，使用金鑰的 TPM 標準，提供已簽署的共用存取簽章 (SAS) 權杖。 使用此證明形式時，裝置不需要有實體 TPM，但該服務會根據 [TPM 規格](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/)，而預期裝置使用簽署金鑰進行證明。
* 以共用存取簽章 (SAS) [安全性權杖](../iot-hub/iot-hub-devguide-security.md#security-tokens)為基礎的**對稱金鑰**，其包含雜湊簽章與內嵌到期日。 如需詳細資訊，請參閱[對稱金鑰證明](concepts-symmetric-key-attestation.md)。


## <a name="hardware-security-module"></a>硬體安全模組

我們將硬體安全模組 (也稱為 HSM) 作為安全的硬體式裝置密碼儲存體使用，這是最安全的密碼儲存體。 X.509 憑證和 SAS 權杖都可以儲存在硬體安全模組 (HSM) 中。 硬體安全模組 (HSM) 可以和佈建支援的兩種證明機制搭配使用。

> [!TIP]
> 我們強烈建議裝置使用硬體安全模組 (HSM) 以在裝置上安全地儲存密碼。

您也可以將裝置密碼儲存在軟體 (記憶體) 上，但相較於硬體安全模組 (HSM)，該儲存方式比較不安全。

## <a name="trusted-platform-module"></a>信賴平台模組

TPM 可以參考安全地儲存驗證平台所用之金鑰的標準，以及與實作標準之模組互動的 I/O 介面。 TPM 可以是獨立硬體、整合式硬體、韌體或軟體。 深入了解 [TPM 和 TPM 證明](/windows-server/identity/ad-ds/manage/component-updates/tpm-key-attestation)。 裝置佈建服務僅支援 TPM 2.0。

TPM 證明是以 nonce 挑戰為基礎，會使用簽署和儲存根金鑰來出示共用存取簽章 (SAS) 權杖。

### <a name="endorsement-key"></a>簽署金鑰

簽署金鑰是 TPM 內建的非對稱金鑰，該金鑰在製造 TPM 時內部產生或插入，且專屬於該 TPM。 您無法變更或移除簽署金鑰。 簽署金鑰的私密部分絕不會釋放到 TPM 外部，而簽署金鑰的公開部分則用於識別正版 TPM。 深入了解[簽署金鑰](https://technet.microsoft.com/library/cc770443(v=ws.11).aspx)。

### <a name="storage-root-key"></a>儲存根金鑰

儲存根金鑰儲存在 TPM 中，可保護應用程式建立的 TPM 金鑰，這樣一來，若沒有 TPM，就無法使用這些金鑰。 取得 TPM 擁有權後，會產生儲存根金鑰；而若您清除 TPM 以讓新的使用者取得擁有權，便又會產生新的儲存根金鑰。 深入了解[儲存根金鑰](https://technet.microsoft.com/library/cc753560(v=ws.11).aspx)。

## <a name="x509-certificates"></a>X.509 憑證

使用 X.509 憑證作為證明機制是調整生產環境並簡化裝置佈建的絕佳方式。 X.509 憑證通常會排列在信任鏈結中，其中每個憑證是由下一個較高憑證的私密金鑰簽署，依此類推，於自我簽署的根憑證終止。 這種方式會從受信任根憑證授權單位 (CA) 產生的根憑證，透過每個中繼 CA 將委派的信任鏈結建立至裝置上安裝之終端實體「分葉」憑證。 若要深入了解，請參閱[使用 X.509 CA 憑證進行裝置驗證](/azure/iot-hub/iot-hub-x509ca-overview)。 

通常憑證鏈結代表與裝置相關聯的某些邏輯或實體階層。 例如，製造商可能：
- 發出自我簽署根 CA 憑證
- 使用根憑證來產生每個處理站的唯一中繼 CA 憑證
- 使用每個處理站的憑證來產生工廠中每個生產線的唯一中繼 CA 憑證
- 最後使用生產線憑證來產生該生產線上所製造之每個裝置的唯一裝置 (終端實體) 憑證。 

若要深入了解，請參閱[概念性了解 IoT 產業中的 X.509 CA 憑證](/azure/iot-hub/iot-hub-x509ca-concept)。 

### <a name="root-certificate"></a>根憑證

根憑證是自我簽署 X.509 憑證，代表憑證授權單位 (CA)。 它是憑證鏈結的終點或信任錨點。 根憑證可以由組織自我發行，或是向根憑證授權單位購買。 若要深入了解，請參閱[取得 X.509 CA 憑證](/azure/iot-hub/iot-hub-security-x509-get-started#get-x509-ca-certificates)。 根憑證也可以稱為根 CA 憑證。

### <a name="intermediate-certificate"></a>中繼憑證

中繼憑證是根憑證 (或其鏈結中具有根憑證的另一個中繼憑證) 簽署的一種 X.509 憑證。 鏈結中的最後一個中繼憑證會用來簽署分葉憑證。 中繼憑證也可以稱為中繼 CA 憑證。

### <a name="end-entity-leaf-certificate"></a>終端實體「分葉」憑證

分葉憑證 (或終端實體憑證) 會識別憑證持有者。 它在其憑證鏈結中有根憑證，以及零或多個中繼憑證。 分葉憑證無法用來簽署其他任何憑證。 它會唯一識別裝置來佈建服務，且有時稱為裝置憑證。 在驗證期間，裝置會使用與此憑證相關聯的私密金鑰，回應以證明來自服務的持有挑戰。 若要深入了解，請參閱[驗證以 X.509 CA 憑證簽署的裝置](/azure/iot-hub/iot-hub-x509ca-overview#authenticating-devices-signed-with-x509-ca-certificates)。

## <a name="controlling-device-access-to-the-provisioning-service-with-x509-certificates"></a>使用 X.509 憑證控制對於佈建服務的裝置存取

佈建服務會公開兩種類型的註冊項目，您可以用來控制對於使用 X.509 證明機制之裝置的存取：  

- [個別註冊](./concepts-service.md#individual-enrollment)項目是以與特定裝置相關聯的裝置憑證來設定的。 這些項目會控制特定裝置的註冊。
- [註冊群組](./concepts-service.md#enrollment-group)項目是與特定的中繼或根 CA 憑證相關聯的。 這些項目會控制所有裝置的註冊，這些裝置在其憑證鏈結中具有中繼或根憑證。 

當裝置連線到佈建服務時，服務會給予更特定註冊項目比起較不特定註冊項目更高的優先順序。 也就是說，如果有裝置的個別註冊存在，佈建服務會套用該項目。 如果沒有裝置的任何個別註冊，且裝置的憑證鏈結中有第一個中繼憑證的註冊群組存在，則服務會套用該項目，依此類推到根目錄鏈結。 服務會套用它所找到的第一個適用項目，例如：

- 如果找到的第一個註冊項目已啟用，則服務會佈建裝置。
- 如果找到的第一個註冊項目已停用，則服務不會佈建裝置。  
- 如果在裝置的憑證鏈結中找不到任何憑證的註冊項目，則服務不會佈建裝置。 

這項機制和憑證鏈結的階層式結構提供功能強大具彈性的方式，讓您可以控制個別裝置以及裝置群組的存取。 例如，假設有五個裝置具有下列憑證鏈結： 

- 裝置 1：根憑證 -> 憑證 A -> 裝置 1 憑證
- 裝置 2：根憑證 -> 憑證 A -> 裝置 2 憑證
- 裝置 3：根憑證 -> 憑證 A -> 裝置 3 憑證
- 裝置 4：根憑證 -> 憑證 B -> 裝置 4 憑證
- 裝置 5：根憑證 -> 憑證 B -> 裝置 5 憑證

一開始，您可以為根憑證建立單一已啟用群組註冊項目，以啟用所有五個裝置的存取。 如果憑證 B 稍後洩露，您可以為憑證 B 建立已停用註冊群組項目，以防止裝置 4 和裝置 5 註冊。 如果稍後裝置 3 也遭到入侵，您可以針對其憑證建立已停用個別註冊項目。 這樣會撤銷裝置 3 的存取權，但是仍然允許裝置 1 和裝置 2 註冊。