---
title: 使用 Azure IoT 中樞裝置佈建服務來佈建裝置 | Microsoft Docs
description: 使用 Azure IoT 中樞裝置佈建服務將裝置佈建到單一 IoT 中樞
author: wesmc7777
ms.author: wesmc
ms.date: 04/12/2018
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 1b9d6342d30c5f5e9ef80213664447c48a62494c
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521895"
---
# <a name="provision-the-device-to-an-iot-hub-using-the-azure-iot-hub-device-provisioning-service"></a>使用 Azure IoT 中樞裝置佈建服務將裝置佈建到 IoT 中樞

在上一個教學課程中，您已了解如何將裝置設定為連線到裝置佈建服務。 在本教學課程中，您會了解如何使用這項服務，以透過自動佈建和**_註冊清單_** 將裝置佈建到單一 IoT 中樞。 本教學課程說明如何：

> [!div class="checklist"]
> * 註冊裝置
> * 啟動裝置
> * 確認裝置已註冊

## <a name="prerequisites"></a>必要條件

在繼續之前，請務必依照[將裝置設定為使用 Azure IoT 中樞裝置佈建服務來進行佈建](./tutorial-set-up-device.md)教學課程所討論的方法來設定裝置。

如果您不熟悉自動佈建程序，請務必先檢閱[自動佈建概念](concepts-auto-provisioning.md)才能繼續。

<a id="enrolldevice"></a>
## <a name="enroll-the-device"></a>註冊裝置

此步驟涉及將裝置的唯一安全性構件新增至裝置佈建服務。 這些安全性成品是以裝置的[證明機制](concepts-device.md#attestation-mechanism)為基礎，如下所示：

- 對於以 TPM 為基礎的裝置，您需要：
    - 「簽署金鑰」是每個 TPM 晶片或模擬從 TMP 晶片製造商取得唯一的簽署金鑰。  如需詳細資訊，請參閱[了解 TPM 簽署金鑰](https://technet.microsoft.com/library/cc770443.aspx)。
    - 註冊識別碼，用來唯一識別命名空間/範圍中的裝置。 此識別碼與裝置識別碼不一定相同。 每個裝置都必須有註冊識別碼。 若為 TPM 架構的裝置，註冊識別碼可能會衍生自該 TPM 自身，例如，TPM 簽署金鑰的 SHA-256 雜湊。

    [![入口網站中的 TPM 註冊資訊](./media/tutorial-provision-device-to-hub/tpm-device-enrollment.png)](./media/tutorial-provision-device-to-hub/tpm-device-enrollment.png#lightbox)  

- 對於以 X.509 為基礎的裝置，您需要：
    - [核發給 X.509 晶片或模擬的憑證](https://msdn.microsoft.com/library/windows/desktop/bb540819.aspx)，格式為 .pem 或 .cer 檔案。 若要進行個別註冊，您必須對 X.509 系統使用每一裝置的「簽章者憑證」，若要進行群組註冊，則必須使用「根憑證」。 

    [![在入口網站中新增 X.509 證明的個別註冊](./media/tutorial-provision-device-to-hub/individual-enrollment.png)](./media/tutorial-provision-device-to-hub/individual-enrollment.png#lightbox)

有兩種方式可向裝置佈建服務註冊裝置：

- **註冊群組** 這代表一組共用特定證明機制的裝置。 對於共用所需初始設定的大量裝置，或是全都要進入同一個租用戶的裝置，建議您使用註冊群組。 如需與註冊群組的身分識別證明有關的詳細資訊，請參閱[安全性](concepts-security.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates)。

    [![在入口網站中新增 X.509 證明的群組註冊](./media/tutorial-provision-device-to-hub/group-enrollment.png)](./media/tutorial-provision-device-to-hub/group-enrollment.png#lightbox)

- **個別註冊** 這代表可能會向裝置佈建服務註冊的單一裝置項目。 個別註冊可使用 x509 憑證或 SAS 權杖 (在實際或虛擬的 TPM 中) 來作為證明機制。 對於需要唯一初始設定的裝置，以及只能透過 TPM 或虛擬 TPM 使用 SAS 權杖作為證明機制的裝置，我們建議您使用個別申請。 個別申請可能會指定所需的 IoT 中樞裝置識別碼。

現在使用以裝置的證明機制為基礎的必要安全性成品，向您的裝置佈建服務執行個體註冊裝置： 

1. 登入 Azure 入口網站，按一下左側功能表上的 [所有資源] 按鈕，然後開啟您的裝置佈建服務。

2. 在裝置佈建服務摘要刀鋒視窗上，選取 [管理註冊]。 根據您的裝置設定選取 [個別註冊] 索引標籤或 [註冊群組] 索引標籤。 按一下位於頂端的 [新增] 按鈕。 選取 [TPM] 或 [X.509] 來作為識別證明「機制」，然後如先前所討論的輸入適當的安全性構件。 您可以輸入新的 [IoT 中樞裝置識別碼]。 完成後，按一下 [儲存] 按鈕。 

3. 在成功註冊裝置後，您應該會看到它顯示在入口網站中，如下所示：

    ![入口網站中的成功 TPM 註冊](./media/tutorial-provision-device-to-hub/tpm-enrollment-success.png)

註冊之後，佈建服務會等候裝置開機，並於稍後的任何時間點與裝置連線。 裝置第一次開機時，用戶端 SDK 程式庫會與晶片互動以從裝置擷取安全構件，並向裝置佈建服務確認註冊情形。 

## <a name="start-the-iot-device"></a>啟動 IoT 裝置

您的 IoT 裝置可以是實際裝置，或是模擬裝置。 由於 IoT 裝置已向裝置佈建服務執行個體註冊，因此裝置現在可以開啟，並使用證明機制呼叫要辨識的佈建服務。 在佈建服務已辨識裝置後，裝置即會指派給 IoT 中樞。 

針對 C、Java、C#、Node.js 和 Python，都會包含使用 TPM 和 X.509 證明的模擬裝置範例。 例如，使用 TPM 和 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) 的模擬裝置會遵循[模擬裝置的第一個開機順序](quick-create-simulated-device.md#simulate-first-boot-sequence-for-the-device)一節中說明的程序。 使用 X.509 憑證證明的相同裝置也會參考這個[開機順序](quick-create-simulated-device-x509.md#simulate-first-boot-sequence-for-the-device)小節。

如需實際裝置的範例，請參閱 [MXChip Iot DevKit 的操作說明指南](how-to-connect-mxchip-iot-devkit.md)。

請啟動裝置，讓裝置的用戶端應用程式開始向裝置佈建服務進行註冊。  

## <a name="verify-the-device-is-registered"></a>確認裝置已註冊

在您的裝置開機後，應該會發生下列動作：

1. 裝置會向裝置佈建服務傳送註冊要求。
2. 若為 TPM 裝置，裝置佈建服務會回傳註冊挑戰供您的裝置回應。 
3. 註冊成功後，裝置佈建服務會對裝置回傳 IoT 中樞 URI、裝置識別碼和加密金鑰。 
4. 裝置上的 IoT 中樞用戶端應用程式接著會連線到您的中樞。 
5. 成功連線至中樞後，您應該會看到裝置出現在 IoT 中樞的 **IoT 裝置**總管中。 

    ![在入口網站中成功連線至中樞](./media/tutorial-provision-device-to-hub/hub-connect-success.png)

如需詳細資訊，請參閱 TPM 模擬器範例應用程式 [dps_client_sample](https://github.com/Azure/azure-iot-device-auth/blob/master/dps_client/samples/dps_client_sample/dps_client_sample.c)。 

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 註冊裝置
> * 啟動裝置
> * 確認裝置已註冊

前進到下一個教學課程，以了解如何跨負載平衡中樞來佈建多個裝置。 

> [!div class="nextstepaction"]
> [跨負載平衡 IoT 中樞來佈建裝置](./tutorial-provision-multiple-hubs.md)
