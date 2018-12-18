---
title: 如何使用 Azure IoT 中樞的裝置佈建服務執行 X.509 CA 憑證擁有權證明 | Microsoft Docs
description: 如何使用裝置佈建服務驗證 X.509 CA 憑證
author: wesmc7777
ms.author: wesmc
ms.date: 02/26/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 6a6782c5e1cd7b5f42b619cd52470b640813a905
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42144817"
---
# <a name="how-to-do-proof-of-possession-for-x509-ca-certificates-with-your-device-provisioning-service"></a>如何使用您的裝置佈建服務執行 X.509 CA 憑證擁有權證明

已驗證的 X.509 憑證授權單位 (CA) 憑證是指該 CA 憑證已上傳至您的佈建服務並且已註冊，而且已透過服務進行擁有權證明。 

擁有權證明包含下列步驟：
1. 取得佈建服務 (適用於 X.509 CA 憑證) 所產生的唯一驗證碼。 您可以從 Azure 入口網站執行此動作。
2. 以此驗證碼作為 X.509 驗證憑證的主體，建立驗證憑證，然後使用與 X.509 CA 憑證相關聯的私密金鑰簽署該憑證。
3. 將已簽署的驗證憑證上傳至服務。 服務會使用要驗證之 CA 憑證的公開部分來驗證此驗證憑證，藉此證明您擁有 CA 憑證的私密金鑰。

使用註冊群組時，已驗證的憑證扮演著重要角色。 驗證憑證擁有權可提供額外的安全層，因為可確保憑證的上傳者擁有憑證的私密金鑰。 驗證可防止探查您流量的惡意動作擷取中繼憑證，並使用該憑證在他們自己的佈建服務中建立註冊群組，進而有效劫持您的裝置。 透過證明信任鏈結中的根憑證或中繼憑證擁有權，您可以證明您有權產生裝置的分葉憑證，以將其註冊為該註冊群組的一部分。 基於這個理由，註冊群組中設定的根憑證或中繼憑證，必須是信任鏈 中已驗證的憑證或必須積存為已驗證的憑證，信任鏈結需在裝置向服務進行驗證時出示。 若要深入了解註冊群組，請參閱 [X.509 憑證](concepts-security.md#x509-certificates)和[使用 X.509 憑證控制裝置對佈建服務的存取](concepts-security.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates)。

## <a name="register-the-public-part-of-an-x509-certificate-and-get-a-verification-code"></a>註冊 X.509 憑證的公開部分，並取得驗證碼

若要向佈建服務註冊 CA 憑證並取得驗證碼，以在擁有權證明期間使用，請遵循下列步驟。 

1. 在 Azure 入口網站中，瀏覽至您的佈建服務，並從左側功能表開啟 [憑證]。 
2. 按一下 [新增] 以新增新的憑證。
3. 為您的憑證輸入易記的顯示名稱。 瀏覽至代表 X.509 憑證公開部分的 .cer 或 .pem 檔案。 按一下 [上傳] 。
4. 一旦取得您的憑證已成功上傳的通知後，請按一下 [儲存]。

    ![Upload certificate](./media/how-to-verify-certificates/add-new-cert.png)  

   [憑證總管] 清單中會顯示您的憑證。 請注意，這個憑證的 [狀態] 是 [未驗證]。

5. 按一下您在上一個步驟中新增的憑證。

6. 在 [憑證詳細資料] 中，按一下 [產生驗證碼]。

7. 佈建服務會建立**驗證碼**，您可以用它來驗證憑證擁有權。 將程式碼複製到剪貼簿。 

   ![驗證憑證](./media/how-to-verify-certificates/verify-cert.png)  

## <a name="digitally-sign-the-verification-code-to-create-a-verification-certificate"></a>以數位方式簽署驗證碼，以建立驗證憑證

現在，您必須使用與 X.509 CA 憑證相關聯的私密金鑰來簽署這個驗證碼，如此就會產生簽章。 這稱為[擁有權證明](https://tools.ietf.org/html/rfc5280#section-3.1)，且會產生已簽署的驗證憑證。

Microsoft 提供了工具和範例，協助您建立已簽署的驗證憑證： 

- **Azure IoT 中樞 C SDK**提供 PowerShell (Windows) 及 Bash (Linux) 指令碼，協助您建立 CA 憑證和分葉憑證來進行開發，以及使用驗證碼執行擁有權證明。 您可以將適用您系統的[檔案](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates)下載至工作資料夾，並遵循[管理 CA 憑證讀我檔案](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)中的指示，對 CA 憑證執行擁有權證明。 
- **Azure IoT 中樞 C# SDK** 包含[群組憑證驗證範例](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service/samples/GroupCertificateVerificationSample)，可用來執行擁有權證明。
 
> [!IMPORTANT]
> 除了執行擁有權證明外，先前所提到的 PowerShell 和 Bash 指令碼也可讓您建立根憑證、中繼憑證和分葉憑證，以用於驗證和佈建裝置。 這些憑證應只限用於開發。 一律不能用在生產環境中。 

文件中提供的 PowerShell 和 Bash 指令碼及 SDK 皆依賴 [OpenSSL](https://www.openssl.org/)。 您也可以使用 OpenSSL 或其他第三方工具，來協助您執行擁有權證明。 如需深入了解如何使用與 SDK 一起提供的工具，請參閱[如何使用 SDK 中提供的工具](how-to-use-sdk-tools.md)。 


## <a name="upload-the-signed-verification-certificate"></a>上傳已簽署的驗證憑證

1. 在入口網站中，將產生的簽章當作驗證憑證上傳至佈建服務。 在 Azure 入口網站的 [憑證詳細資料] 中，使用 [驗證憑證 .pem 或.cer 檔案] 欄位旁的 [檔案總管] 圖示，從您的系統上傳已簽署的驗證憑證。

2. 一旦憑證上傳成功後，按一下 [確認]。 憑證的 [狀態] 在 [憑證總管]清單中會變更為**_已驗證_**。 如果無法自動更新，請按一下 [重新整理]。

   ![上傳憑證驗證](./media/how-to-verify-certificates/upload-cert-verification.png)  

## <a name="next-steps"></a>後續步驟

- 若要深入了解如何使用入口網站來建立註冊群組，請參閱[使用 Azure 入口網站管理裝置註冊](how-to-manage-enrollments.md)。
- 若要深入了解如何使用服務 SDK 來建立註冊群組，請參閱[使用服務 SDK 管理裝置註冊](how-to-manage-enrollments-sdks.md)。










