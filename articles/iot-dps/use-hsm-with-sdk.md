---
title: Azure 作法 - 如何在 Azure 中使用不同證明機制搭配裝置佈建服務用戶端 SDK
description: Azure 作法 - 如何在 Azure 中使用不同證明機制搭配裝置佈建服務用戶端 SDK
author: yzhong94
ms.author: yizhon
ms.date: 03/30/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: arjmands
ms.custom: mvc
ms.openlocfilehash: 4ab558b680a0d00d1b9bdfbcb1529219f6c37b37
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319246"
---
# <a name="how-to-use-different-attestation-mechanisms-with-device-provisioning-service-client-sdk-for-c"></a>如何使用不同證明機制搭配適用於 C 的裝置佈建服務用戶端 SDK

本文說明如何使用不同的[證明機制](concepts-security.md#attestation-mechanism)搭配適用於 C 的裝置佈建服務用戶端 SDK。您可以使用實體裝置或模擬器。 佈建服務支援兩種證明機制的驗證：X **.** 509 和信賴平台模組 (TPM)。

## <a name="prerequisites"></a>必要條件

根據[建立及佈建模擬的裝置](./quick-create-simulated-device.md)指南中標題為「準備開發環境」一節，準備您的開發環境。

### <a name="choose-an-attestation-mechanism"></a>選擇證明機制

身為裝置製造商，您必須先選擇以其中一個支援類型為基礎的證明機制。 [適用於 C 的裝置佈建服務用戶端 SDK](https://github.com/Azure/azure-iot-sdk-c/tree/master/provisioning_client) 目前提供下列證明機制的支援： 

- [信賴平台模組 (TPM)](https://en.wikipedia.org/wiki/Trusted_Platform_Module)：TPM 是適用於大部分 Windows 裝置平台以及一些 Linux/Ubuntu 架構裝置的公認標準。 身為裝置製造商，如果您的裝置是執行上述任一作業系統，而且您想要使用公認的標準，則可以選擇這個證明機制。 若使用 TPM 晶片，裝置就只能個別地向裝置佈建服務進行註冊。 若要進行開發，您可以在 Windows 或 Linux 開發機器上使用 TPM 模擬器。

- [X.509](https://cryptography.io/en/latest/x509/)：X.509 憑證可以儲存在相對較新的晶片上，其名為[硬體安全模組 (HSM)](concepts-security.md#hardware-security-module)。 Microsoft 也正著手設計 RIoT 或 DICE 晶片，以便實作 X.509 憑證。 若使用 X.509 晶片，您將可以在入口網站中進行大量裝置註冊。 這種晶片也支援某些非 Windows 的作業系統，例如 embedOS。 針對開發用途，裝置佈建服務用戶端 SDK 可支援 X.509 裝置模擬器。 

如需詳細資訊，請參閱 IoT 中樞裝置佈建服務的[安全性概念](concepts-security.md)和[自動佈建概念](/azure/iot-dps/concepts-auto-provisioning)。

## <a name="enable-authentication-for-supported-attestation-mechanisms"></a>針對支援的證明機制啟用驗證

SDK 驗證模式 (X **.** 509 或 TPM) 必須針對實體裝置或模擬器啟用後，才能在 Azure 入口網站中進行註冊。 首先，瀏覽至 azure-iot-sdk-c 的根目錄。 然後根據您選擇的驗證模式來執行指定的命令：

### <a name="use-x509-with-simulator"></a>使用 X **.** 509 搭配模擬器

佈建服務隨附於裝置識別組合引擎 (DICE) 模擬器，所產生的 X **.** 509 憑證可驗證裝置。 若要啟用 X **.** 509 驗證，請執行下列命令： 

```
cmake -Ddps_auth_type=x509 ..
```

可在[這裡](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/)找到硬體與 DICE 的相關資訊。

### <a name="use-x509-with-hardware"></a>使用 X **.** 509 搭配硬體

可在其他硬體上使用佈建服務搭配 X **.** 509。 需要硬體與 SDK 之間的介面才能建立連線。 如需關於介面的資訊，請連絡您的 HSM 製造商。

### <a name="use-tpm"></a>使用 TPM

佈建服務可以使用 SAS 權杖連線到 Windows 和 Linux 硬體 TPM 晶片。 若要啟用 TPM 驗證，請執行下列命令：

```
cmake -Ddps_auth_type=tpm ..
```

### <a name="use-tpm-with-simulator"></a>使用 TPM 搭配模擬器

如果您沒有搭載 TPM 晶片的裝置，可以在 Windows 作業系統上就開發目的使用模擬器。 若要啟用 TPM 驗證並執行 TPM 模擬器，請執行下列命令：

```
cmake -Ddps_auth_type=tpm_simulator ..
```

## <a name="build-the-sdk"></a>建置 SDK
建立裝置註冊之前，請建置 SDK。

### <a name="linux"></a>Linux
- 若要在 Linux 中建置 SDK：
    ```
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    cmake ..
    cmake --build .  # append '-- -j <n>' to run <n> jobs in parallel
    ```
- 若要建置偵錯二進位檔，請將對應的 CMake 選項新增至專案產生命令，例如：
    ```
    cmake -DCMAKE_BUILD_TYPE=Debug ..
    ```

- 有許多 [CMake 設定選項](https://cmake.org/cmake/help/v3.6/manual/cmake.1.html)可用來建置 SDK。 例如，您可以停用其中一個可用的通訊協定堆疊，方法是將引數新增至 CMake 專案產生命令：
    ```
    cmake -Duse_amqp=OFF ..
    ```
- 您也可以建置並執行單元測試：
    ```
    cmake -Drun_unittests=ON ..
    cmake --build .
    ctest -C "debug" -V
    ```

### <a name="windows"></a>Windows
- 若要在 Windows 中建置 SDK，請採取下列步驟以產生專案檔：
    - 開啟「VS2015 的開發人員命令提示字元」
    - 從存放庫的根目錄執行下列 CMake 命令：
      ```
      cd azure-iot-sdk-c
      mkdir cmake
      cd cmake
      cmake -G "Visual Studio 14 2015" ..
      ```
    此命令會建置 x86 程式庫。 若要建置 x64 的程式庫，請修改 cmake 產生器引數： 
    ```
    cmake .. -G "Visual Studio 14 2015 Win64"
    ```

- 如果專案產生成功完成，應該會在 `cmake` 資料夾下看到 Visual Studio 解決方案檔 (.sln)。 若要建置 SDK：
    - 在 Visual Studio 中開啟 **cmake\azure_iot_sdks.sln** 並加以建置，**或**
    - 在您用來產生專案檔的命令提示字元中執行下列命令：
      ```
      cmake --build . -- /m /p:Configuration=Release
      ```
- 若要建置偵錯二進位檔，請使用對應的 MSBuild 引數： 
    ```
    cmake --build . -- /m /p:Configuration=Debug`
    ```
- 有許多 CMake 設定選項可用來建置 SDK。 例如，您可以停用其中一個可用的通訊協定堆疊，方法是將引數新增至 CMake 專案產生命令：
    ```
    cmake -G "Visual Studio 14 2015" -Duse_amqp=OFF ..
    ```
- 此外，您可以建置並執行單元測試：
    ```
    cmake -G "Visual Studio 14 2015" -Drun_unittests=ON ..
    cmake --build . -- /m /p:Configuration=Debug
    ctest -C "debug" -V
    ```
  
### <a name="libraries-to-include"></a>要包含的程式庫
- 這些程式庫應包含在您的 SDK：
    - 佈建服務：dps_http_transport、dps_client、dps_security_client
    - IoTHub 安全性：iothub_security_client

## <a name="create-a-device-enrollment-entry-in-device-provisioning-services"></a>在裝置佈建服務中建立裝置註冊項目

### <a name="tpm"></a>TPM
如果您是使用 TPM，請遵循[「使用 IoT 中樞裝置佈建服務來建立及佈建模擬的裝置」](./quick-create-simulated-device.md)中的指示，在裝置佈建服務中建立裝置註冊項目，並模擬第一次開機。

### <a name="x509"></a>X **.** 509

1. 若要在佈建服務中註冊裝置，您必須記下每個裝置的簽署金鑰和註冊 ID，這些都會顯示在用戶端 SDK 所提供的佈建工具中。 執行下列命令可列印出根 CA 憑證 (適用於註冊群組) 和分葉憑證 (適用於個別註冊)：
      ```
      ./azure-iot-sdk-c/dps_client/tools/x509_device_provision/x509_device_provision.exe
      ```
2. 登入 Azure 入口網站，按一下左側功能表上的 [所有資源] 按鈕，然後開啟您的裝置佈建服務。
   - X **.** 509 個別註冊：在佈建服務摘要刀鋒視窗上，選取 [管理註冊]。 選取 [個別註冊] 索引標籤，然後按一下頂端的 [新增] 按鈕。 選取 **X**.**509** 作為身分識別證明機制，如刀鋒視窗所要求上傳分葉憑證。 完成後，按一下 [儲存] 按鈕。 
   - X **.** 509 群組註冊：在佈建服務摘要刀鋒視窗上，選取 [管理註冊]。 選取 [群組註冊] 索引標籤，然後按一下頂端的 [新增] 按鈕。 選取 **X**.**509** 作為身分識別證明機制，輸入群組名稱和憑證名稱，如刀鋒視窗所要求上傳 CA/中繼憑證。 完成後，按一下 [儲存] 按鈕。 

## <a name="enable-authentication-for-devices-using-a-custom-attestation-mechanism-optional"></a>針對使用自訂證明機制的裝置啟用驗證 (選擇性)

> [!NOTE]
> 本節僅適用於需要自訂平台或證明機制支援的裝置，若裝置目前由適用於 C 的裝置佈建服務用戶端 SDK 所支援，則不適用。並請注意，SDK 通常會使用 "HSM" 字詞來取代「證明機制」。

您必須先為自訂證明機制開發存放庫和程式庫：

1. 開發程式庫以存取您的證明機制。 此專案必須產生靜態程式庫以供裝置佈建 SDK 取用。

2. 在您的程式庫中實作下列標頭檔所定義的函式： 

    - 若為自訂 TPM：實作 [HSM TPM API](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_custom_hsm.md#hsm-tpm-api) 之下定義的函式。  
    - 若為自訂 X.509：實作 [HSM X509 API](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_custom_hsm.md#hsm-x509-api) 之下定義的函式。 

一旦您的程式庫自行建置成功，您必須藉由與您的程式庫進行連結，將它與裝置佈建服務用戶端 SDK 整合。 ：

1. 在下列 `cmake` 命令中提供自訂的 GitHub 存放庫和程式庫：
    ```cmd/sh
    cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=<path_and_name_of_library> <PATH_TO_AZURE_IOT_SDK>
    ```
   
2. 開啟 CMake 所建置的 Visual Studio 方案檔 (`\azure-iot-sdk-c\cmake\azure_iot_sdks.sln`)，並加以建置。 

    - 建置程序會編譯 SDK 程式庫。
    - SDK 會嘗試與 `cmake` 命令中所定義的自訂程式庫進行連結。

3. 若要確認您的自訂證明機制是否已正確實作，請執行 "Provision_Samples" 之下的 "prov_dev_client_ll_sample" 範例應用程式 (在 `\azure-iot-sdk-c\cmake\provisioning_client\samples\prov_dev_client_ll_sample` 之下)。

## <a name="connecting-to-iot-hub-after-provisioning"></a>佈建之後連線到 IoT 中樞

一旦使用佈建服務來佈建裝置後，此 API 會使用指定的驗證模式 (X **.** 509 或 TPM) 與 IoT 中樞連線： 
  ```
  IOTHUB_CLIENT_LL_HANDLE handle = IoTHubClient_LL_CreateFromDeviceAuth(iothub_uri, device_id, iothub_transport);
  ```
