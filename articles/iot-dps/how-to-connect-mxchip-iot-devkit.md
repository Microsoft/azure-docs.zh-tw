---
title: 如何使用 Azure IoT 中樞裝置佈建服務的自動佈建向 IoT 中樞註冊 MXChip IoT DevKit | Microsoft Docs
description: 如何使用 Azure IoT 中樞裝置佈建服務的自動佈建向 IoT 中樞註冊 MXChip IoT DevKit。
author: liydu
ms.author: liydu
ms.date: 04/04/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: jeffya
ms.openlocfilehash: d8912a5da8c4df2069d8bc53454748b5fb3d5c39
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42140904"
---
# <a name="use-azure-iot-hub-device-provisioning-service-auto-provisioning-to-register-the-mxchip-iot-devkit-with-iot-hub"></a>使用 Azure IoT 中樞裝置佈建服務的自動佈建向 IoT 中樞註冊 MXChip IoT DevKit

本文說明如何使用 Azure IoT 中樞裝置佈建服務的[自動佈建](concepts-auto-provisioning.md)，向 Azure IoT 中樞註冊 MXChip IoT DevKit。 在本教學課程中，您了解如何：

* 在裝置上設定裝置佈建服務的全域端點。
* 使用唯一裝置祕密 (UDS) 來產生 X.509 憑證。
* 註冊個別裝置。
* 確認裝置已註冊。

[MXChip IoT DevKit](https://aka.ms/iot-devkit) 是全方位 Arduino 相容板，具有豐富的週邊設備和感應器。 您可以使用[適用於 Arduino 的 Visual Studio Code 擴充功能](https://aka.ms/arduino)來對它進行開發。 DevKit 隨附不斷增長的[專案目錄](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/)，以引導您設定運用 Azure 服務的物聯網 (IoT) 解決方案原型。

## <a name="before-you-begin"></a>開始之前

若要完成本教學課程中的步驟，請先執行下列工作：

* 遵循[將 IoT DevKit AZ3166 連線到雲端中的 Azure IoT 中樞](/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)中的步驟來準備您的 DevKit。
* 利用[更新 DevKit 韌體](https://microsoft.github.io/azure-iot-developer-kit/docs/firmware-upgrading/)教學課程，升級至最新的韌體 (1.3.0 或更新版本)。
* 遵循[使用 Azure 入口網站設定 IoT 中樞裝置佈建服務](/azure/iot-dps/quick-setup-auto-provision)中的步驟，建立 IoT 中樞並將其與裝置佈建服務執行個體連結。

## <a name="build-and-deploy-auto-provisioning-registration-software-to-the-device"></a>建置自動佈建註冊軟體並且部署到裝置

若要將 DevKit 連線至您所建立的裝置佈建服務服務執行個體：

1. 在 Azure 入口網站中，選取裝置佈建服務的 [概觀] 窗格，並記下 [全域裝置端點] 和 [識別碼範圍] 值。
  ![裝置佈建服務全域端點和識別碼範圍](./media/how-to-connect-mxchip-iot-devkit/dps-global-endpoint.png)

2. 確定 `git` 已安裝在電腦上，並已新增至命令視窗可存取的環境變數。 請參閱[軟體自由保護組織 (Software Freedom Conservancy) 的 Git 用戶端工具](https://git-scm.com/download/) \(英文\) 來安裝最新版本。

3. 開啟命令提示字元。 複製 GitHub 存放庫中的裝置佈建服務範例程式碼：
  ```bash
  git clone https://github.com/DevKitExamples/DevKitDPS.git
  ```

4. 開啟 Visual Studio Code，將 DevKit 連線至電腦，然後開啟包含所複製程式碼的資料夾。

5. 開啟 **DevKitDPS.ino**。 尋找 `[Global Device Endpoint]` 和 `[ID Scope]` 並將其替換為您剛才記下的值。
  ![裝置佈建服務端點](./media/how-to-connect-mxchip-iot-devkit/endpoint.png) 您可以將 **registrationId** 留空。 應用程式會根據 MAC 位址和韌體版本為您產生一個值。 如果您想要自訂「註冊識別碼」，必須只使用英數字元、小寫及連字號的組合，且長度上限為 128 個字元。 如需詳細資訊，請參閱[透過 Azure 入口網站管理裝置註冊](https://docs.microsoft.com/azure/iot-dps/how-to-manage-enrollments)。

6. 在 VS Code 中使用 Quick Open (Windows：`Ctrl+P`、macOS：`Cmd+P`)，然後輸入 task device-upload 來組建程式碼並上傳到 DevKit。

7. 輸出視窗會顯示工作是否成功。

## <a name="save-a-unique-device-secret-on-an-stsafe-security-chip"></a>將唯一裝置祕密儲存在 STSAFE 安全晶片上

可以根據裝置的[證明機制](concepts-security.md#attestation-mechanism)，在裝置上設定自動佈建。 MXChip IoT DevKit 會使用來自 [Trusted Computing Group](https://trustedcomputinggroup.org) 的[裝置身分識別組成引擎 (Device Identity Composition Engine)](https://trustedcomputinggroup.org/wp-content/uploads/Foundational-Trust-for-IOT-and-Resource-Constrained-Devices.pdf)。 儲存在 DevKit 上 STSAFE 安全晶片中的唯一裝置祕密 (UDS)會用來產生裝置的唯一 [X.509 憑證](concepts-security.md#x509-certificates)。 此憑證稍後會用於裝置佈建服務中的註冊程序，以及在執行階段的註冊期間使用。

典型的唯一裝置秘密是 64 個字元的字串，如以下範例所示：

```
19e25a259d0c2be03a02d416c05c48ccd0cc7d1743458aae1cb488b074993eae
```

此字串會分成數個字元組並使用於安全性計算中。 上述範例 UDS 會解析成：`0x19`、`0xe2`、`0x5a`、`0x25`、`0x9d`、`0x0c`、`0x2b`、`0xe0`、`0x3a`、`0x02`、`0xd4`、`0x16`、`0xc0`、`0x5c`、`0x48`、`0xcc`、`0xd0`、`0xcc`、`0x7d`、`0x17`、`0x43`、`0x45`、`0x8a`、`0xae`、`0x1c`、`0xb4`、`0x88`、`0xb0`、`0x74`、`0x99`、`0x3e`、`0xae`。

將唯一裝置祕密儲存在 DevKit 上：

1. 使用 Putty 之類的工具，開啟序列監視器。 如需詳細資訊，請參閱[使用設定模式](https://microsoft.github.io/azure-iot-developer-kit/docs/use-configuration-mode/)。

2. 在 DevKit 已連線至電腦的情況下，按住 **A** 按鈕，然後按下再放開 [重設] 按鈕以進入設定模式。 畫面會顯示 DevKit 識別碼和 [組態]。

3. 使用範例 UDS 字串，然後針對您自己的 UDS 將一或多個字元變更為介於 `0` 與 `f` 之間的其他值。

4. 在序列監視器視窗中，輸入 set_dps_uds [your_own_uds_value]，然後按 Enter 鍵。
  > [!NOTE]
  > 例如，如果您藉由將最後兩個字元變更為 `f` 來設定您自己的 UDS，您必須輸入如下所示的命令：`set_dps_uds 19e25a259d0c2be03a02d416c05c48ccd0cc7d1743458aae1cb488b074993eff`。

5. 在不關閉序列監視器視窗的情況下，按下 DevKit 上的 [重設] 按鈕。

6. 記下 [DevKit MAC 位址] 和 [DevKit 韌體版本] 的值。
  ![韌體版本](./media/how-to-connect-mxchip-iot-devkit/firmware-version.png)

## <a name="generate-an-x509-certificate"></a>產生 X.509 憑證

現在，您需要產生 X.509 憑證。 

### <a name="windows"></a>Windows

1. 開啟檔案總管，並移至包含您稍早複製之裝置佈建服務範例程式碼的資料夾。 在 **.build** 資料夾中，尋找 **DPS.ino.bin** 和 **DPS.ino.map**。
  ![產生的檔案](./media/how-to-connect-mxchip-iot-devkit/generated-files.png)
  > [!NOTE]
  > 如果您將 Arduino 的 `built.path` 組態變更到另一個資料夾，請您設定的資料夾中尋找那些檔案。

2. 將這兩個檔案貼到與 **.build** 資料夾同層的 **tools** 資料夾中。

3. 執行 **dps_cert_gen.exe**。 依照提示輸入您的 **UDS**、DevKit 的 **MAC 位址**及**韌體版本**來產生 X.509 憑證。
  ![執行 dps-cert-gen.exe](./media/how-to-connect-mxchip-iot-devkit/dps-cert-gen.png)

4. 產生 X.509 憑證之後，**.pem** 憑證會儲存到相同的資料夾。

## <a name="create-a-device-enrollment-entry-in-the-device-provisioning-service"></a>在裝置佈建服務中建立裝置註冊項目

1. 在 Azure 入口網站中，移至您的裝置佈建服務執行個體。 選取 [管理註冊]，然後選取 [個別註冊] 索引標籤。![個別註冊](./media/how-to-connect-mxchip-iot-devkit/individual-enrollments.png)

2. 選取 [新增] 。

3. 在 [新增註冊] 面板上：

   - 選取 [機制] 底下的 [X.509]。
   - 按一下 [主要 .pem 或 .cer 憑證檔案] 底下的 [選取檔案]。
   - 在 [開啟舊檔] 對話方塊中，瀏覽至您剛剛產生的 **.pem** 憑證並上傳。
   - 將其他項目保留為預設值，然後按一下 [儲存]。

   ![Upload certificate](./media/how-to-connect-mxchip-iot-devkit/upload-cert.png)

  > [!NOTE]
  > 如果您遇到錯誤並看到這個訊息：
  >
  > `{"message":"BadRequest:{\r\n \"errorCode\": 400004,\r\n \"trackingId\": \"1b82d826-ccb4-4e54-91d3-0b25daee8974\",\r\n \"message\": \"The certificate is not a valid base64 string value\",\r\n \"timestampUtc\": \"2018-05-09T13:52:42.7122256Z\"\r\n}"}`
  >
  > 以純文字方式開啟憑證檔案 **.pem** (使用記事本或任何文字編輯器來開啟)，然後刪除以下這幾行：
  >
  > `"-----BEGIN CERTIFICATE-----"` 和 `"-----END CERTIFICATE-----"`。
  >

## <a name="start-the-devkit"></a>啟動 DevKit

1. 開啟 VS Code 和序列監視器。

2. 按下 DevKit 上的 [Reset] \(重設\) 按鈕。

您會看到 DevKit 開始向裝置佈建服務註冊。

![VS Code 輸出](./media/how-to-connect-mxchip-iot-devkit/vscode-output.png)

## <a name="verify-that-the-devkit-is-registered-with-azure-iot-hub"></a>確認 DevKit 已向 Azure IoT 中樞註冊

在您的裝置開機後，會發生下列動作：

1. 裝置會向裝置佈建服務傳送註冊要求。
2. 裝置佈建服務會回傳註冊挑戰供您的裝置回應。
3. 註冊成功後，裝置佈建服務會對裝置回傳 IoT 中樞 URI、裝置識別碼和加密金鑰。
4. 裝置上的 IoT 中樞用戶端應用程式會連線到您的中樞。
5. 成功連線至中樞後，您會看到裝置出現在 IoT 中樞的 Device Explorer 中。
  ![裝置已註冊](./media/how-to-connect-mxchip-iot-devkit/device-registered.png)

## <a name="problems-and-feedback"></a>問題與意見反應

如果您遇到問題，請參閱 IoT DevKit [常見問題集](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/)，或透過下列管道取得支援：

* [Gitter.im](http://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stackoverflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您會學習如何使用裝置身分識別組合引擎將裝置安全地註冊到裝置佈建服務，讓裝置可以自動向 Azure IoT 中樞註冊。 

總而言之，您學會如何：

> [!div class="checklist"]
> * 在裝置上設定裝置佈建服務的全域端點。
> * 使用唯一裝置祕密來產生 X.509 憑證。
> * 註冊個別裝置。
> * 確認裝置已註冊。

深入了解如何[建立及佈建模擬裝置](./quick-create-simulated-device.md)。

