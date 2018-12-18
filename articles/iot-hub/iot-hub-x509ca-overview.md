---
title: Azure IoT 中樞 X.509 CA 安全性概觀 | Microsoft Docs
description: 概觀 - 如何使用 X.509 憑證授權單位向 IoT 中樞進行裝置驗證。
author: eustacea
manager: arjmands
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/18/2017
ms.author: eustacea
ms.openlocfilehash: 6a9b4fc5479dda58dd024cdf93cbdf4853f9c965
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42140680"
---
# <a name="device-authentication-using-x509-ca-certificates"></a>使用 X.509 CA 憑證進行裝置驗證

本文說明如何使用 X.509 憑證授權單位 (CA) 憑證來驗證連線至「IoT 中樞」的裝置。  在本文中，您將了解：

* 如何取得 X.509 CA 憑證
* 如何向 IoT 中樞註冊 X.509 CA 憑證
* 如何使用 X.509 CA 憑證來簽署裝置
* 以 X.509 CA 簽署之裝置的驗證方式為何

## <a name="overview"></a>概觀

X.509 CA 功能可讓您使用「憑證授權單位」(CA) 向「IoT 中樞」進行裝置驗證。 這大幅簡化了初始裝置註冊程序，以及裝置製造期間的供應鏈物流。 如需了解裝置驗證，請[從這篇有關使用 X.509 CA 憑證之價值的案例文章進行深入了解](iot-hub-x509ca-concept.md)。  建議您先閱讀這篇案例文章再繼續操作，因為此文章會說明為什麼會有後續這些步驟。

## <a name="prerequisite"></a>必要條件

若要使用 X.509 CA 功能，您必須擁有「IoT 中樞」帳戶。  如果您還沒有該帳戶，請[了解如何建立 IoT 中樞執行個體](quickstart-send-telemetry-dotnet.md)。

## <a name="how-to-get-an-x509-ca-certificate"></a>如何取得 X.509 CA 憑證

X.509 CA 憑證位於您每個裝置的憑證鏈結頂端。  您可以購買或建立一個這樣的憑證，視您打算如何使用此憑證而定。

針對生產環境，建議您從公用根憑證授權單位購買 X.509 CA 憑證。 購買 CA 憑證的好處在於，有根 CA 作為信任的協力廠商來擔保您裝置的合法性。 如果您打算將裝置加入開放式 IoT 網路，讓它們與協力廠商產品或服務進行互動，請考量此選項。

您也可以建立自我簽署的 X.509 CA 來進行測試，或在封閉式 IoT 網路中使用。

不論您如何取得 X.509 CA 憑證，請務必在任何時候都將其對應的私密金鑰妥善保密及保護。  這對於在 X.509 CA 驗證中建立信任來說是必要的。 

了解如何[建立自我簽署的 CA 憑證](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)，您可以使用此憑證在這整個功能描述中進行測試。

## <a name="sign-devices-into-the-certificate-chain-of-trust"></a>將裝置簽署至憑證信任鏈結中

X.509 CA 憑證的擁有者可以透過密碼編譯方式簽署中繼 CA，此 CA 可再簽署另一個中繼 CA，依此類推，直到最後一個中繼 CA 藉由簽署裝置來終止此程序為止。 結果會是串聯的憑證鏈結，稱為憑證信任鏈結。 在現實生活中，這最後會變成朝向簽署裝置發展的信任委派。 這個委派相當重要，因為它會建立一個在密碼編譯上多變的保管鏈結，以避免共用簽署金鑰。

![img-generic-cert-chain-of-trust](./media/generic-cert-chain-of-trust.png)

了解如何[建立憑證鏈結](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) (會在簽署裝置時完成)。

## <a name="how-to-register-the-x509-ca-certificate-to-iot-hub"></a>如何向 IoT 中樞註冊 X.509 CA 憑證

請向「IoT 中樞」註冊 X.509 CA 憑證，在此中樞將會使用此憑證，在裝置註冊和連線的期間，進行裝置驗證。  註冊 X.509 CA 憑證的程序包含兩個步驟，亦即上傳憑證檔案和證明所有權。

上傳程序需要上傳一個包含您憑證的檔案。  此檔案一律不應包含任何私密金鑰。

證明所有權步驟涉及一個您與「IoT 中樞」之間的密碼編譯查問和回應程序。  由於數位憑證內容是公用的，因而容易遭到竊聽，因此「IoT 中樞」會想要確定您是否真的擁有 CA 憑證。  其做法應該是產生一個您必須以 CA 憑證的對應私密金鑰簽署的隨機查問。  如果您如先前建議的，將私密金鑰妥善保密及保護，則只有您會知道如何完成此步驟。 在此方法中，私密金鑰的保密性就是信任的來源。  簽署查問之後，請上傳包含結果的檔案來完成此步驟。

請從這裡了解如何[註冊您的 CA 憑證](iot-hub-security-x509-get-started.md#registercerts)。

## <a name="how-to-create-a-device-on-iot-hub"></a>如何在 IoT 中樞上建立裝置

為了杜絕裝置模擬，「IoT 中樞」會要求您讓它知道所要預期的裝置。  您可以在「IoT 中樞」的裝置登錄中建立裝置項目，來達到此目的。  當使用「IoT 中樞」[裝置佈建服務](https://azure.microsoft.com/blog/azure-iot-hub-device-provisioning-service-preview-automates-device-connection-configuration/)時，會自動執行此程序。 

請從這裡了解如何[在 IoT 中樞中手動建立裝置](iot-hub-security-x509-get-started.md#createdevice)。

## <a name="authenticating-devices-signed-with-x509-ca-certificates"></a>驗證以 X.509 CA 憑證簽署的裝置

在註冊 X.509 CA 憑證且裝置已簽署至憑證信任鏈結之後，剩下就是在裝置連線 (即使是第一次連線) 時進行裝置驗證。  當以 X.509 CA 簽署的裝置連線時，它會上傳其憑證鏈結來進行驗證。 此鏈結包含所有中繼 CA 和裝置憑證。  有了此資訊，「IoT 中樞」就會以一個有兩步驟的程序來驗證裝置。  「IoT 中樞」會以密碼編譯方式驗證憑證鏈結來了解內部是否一致，然後向裝置發出所有權證明查問。  「IoT 中樞」會在從裝置獲得成功的所有權證明回應時，宣告裝置驗證。  此宣告會假設裝置的私密金鑰已受保護，而只有裝置能夠成功回應這個查問。  建議您在裝置中使用安全晶片 (例如「硬體安全模組」(HSM)) 來保護私密金鑰。

裝置與「IoT 中樞」的成功連線既會完成驗證程序，同時也象徵設定正確。

從這裡了解如何[完成此裝置連線步驟](iot-hub-security-x509-get-started.md#authenticatedevice)。

## <a name="next-steps"></a>後續步驟

了解 IoT 中 [X.509 CA 驗證的價值](iot-hub-x509ca-concept.md)。

開始使用「IoT 中樞」[裝置佈建服務](https://docs.microsoft.com/azure/iot-dps/)。
