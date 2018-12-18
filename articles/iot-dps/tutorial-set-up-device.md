---
title: 為裝置設定 Azure IoT 中樞裝置佈建服務
description: 在裝置製造過程中將裝置設定為透過 IoT 中樞裝置佈建服務進行佈建
author: wesmc7777
ms.author: wesmc
ms.date: 04/02/2018
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: f0b62f73650294349e1879b306beebabdaf974a7
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45633396"
---
# <a name="set-up-a-device-to-provision-using-the-azure-iot-hub-device-provisioning-service"></a>將裝置設定為使用 Azure IoT 中樞裝置佈建服務進行佈建

在上一個教學課程中，您已了解如何將 Azure IoT 中樞裝置佈建服務設定為自動將裝置佈建到 IoT 中樞。 本教學課程示範如何在製造過程中設定您的裝置，讓它能透過 IoT 中樞自動佈建。 第一次開機並連線至佈建服務時，您的裝置會根據其[證明機制](concepts-device.md#attestation-mechanism)進行佈建。 本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 建置平台特定裝置佈建服務用戶端 SDK
> * 擷取安全構件
> * 建立裝置註冊軟體

本教學課程預期您已使用[設定雲端資源](tutorial-set-up-cloud.md)教學課程中的指示，建立裝置佈建服務執行個體和 IoT 中樞。

本教學課程使用 [Azure IoT SDK 及適用於 C 存放庫的程式庫](https://github.com/Azure/azure-iot-sdk-c)，其中包含適用於 C 的裝置佈建服務用戶端 SDK。此 SDK 目前針對在 Windows 或 Ubuntu 實作上執行的裝置，提供 TPM 和 X.509 支援。 本教學課程是以使用 Windows 開發用戶端為基礎，而且假設熟悉 Visual Studio 2017 基本功能。 

如果您不熟悉自動佈建程序，請務必先檢閱[自動佈建概念](concepts-auto-provisioning.md)才能繼續。 


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

* 啟用[「使用 C++ 進行桌面開發」](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/)工作負載的 Visual Studio 2015 或 [Visual Studio 2017](https://www.visualstudio.com/vs/)。
* 已安裝最新版的 [Git](https://git-scm.com/download/)。



## <a name="build-a-platform-specific-version-of-the-sdk"></a>建置 SDK 的平台特定版本

裝置佈建服務用戶端 SDK 可協助您實作裝置註冊軟體。 但是，您必須先建置您的開發用戶端平台和證明機制特有的 SDK 版本，才可以使用該軟體。 在本教學課程中，您會針對支援的證明類型，在 Windows 開發平台上建置使用 Visual Studio 2017 的 SDK：

1. 下載 [CMake 建置系統](https://cmake.org/download/)的 3.11.4 版。 請確認下載的二進位檔使用對應的密碼編譯雜湊值。 下列範例使用 Windows PowerShell 來驗證 x64 MSI 發行版本 3.11.4 的密碼編譯雜湊：

    ```PowerShell
    PS C:\Downloads> $hash = get-filehash .\cmake-3.11.4-win64-x64.msi
    PS C:\Downloads> $hash.Hash -eq "56e3605b8e49cd446f3487da88fcc38cb9c3e9e99a20f5d4bd63e54b7a35f869"
    True
    ```
    
    在撰寫本文時，CMake 網站上列出了3.11.4 版的下列雜湊值：

    ```
    6dab016a6b82082b8bcd0f4d1e53418d6372015dd983d29367b9153f1a376435  cmake-3.11.4-Linux-x86_64.tar.gz
    72b3b82b6d2c2f3a375c0d2799c01819df8669dc55694c8b8daaf6232e873725  cmake-3.11.4-win32-x86.msi
    56e3605b8e49cd446f3487da88fcc38cb9c3e9e99a20f5d4bd63e54b7a35f869  cmake-3.11.4-win64-x64.msi
    ```

    在開始安裝 `CMake` **之前**，請務必將 Visual Studio 先決條件 (Visual Studio 和「使用 C++ 進行桌面開發」工作負載) 安裝在您的機器上。 在符合先決條件，並且驗證過下載項目之後，請安裝 CMake 建置系統。

1. 開啟命令提示字元或 Git Bash 殼層。 執行下列命令以複製 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) GitHub 存放庫：
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive
    ```
    此存放庫的大小目前約為 220 MB。 預期此作業需要幾分鐘的時間才能完成。


1. 在 git 存放庫的根目錄中建立 `cmake` 子目錄，並瀏覽至該資料夾。 

    ```cmd/sh
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

1. 根據您所將使用的證明機制，建置開發平台的 SDK。 請使用下列其中一個命令 (也請注意應在每個命令結尾處加上兩個句點字元)。 完成時，CMake 會建置出含有您裝置特有內容的 `/cmake` 子目錄：
 
    - 對於使用 TPM 模擬器進行證明的裝置：

        ```cmd/sh
        cmake -Duse_prov_client:BOOL=ON -Duse_tpm_simulator:BOOL=ON ..
        ```

    - 對於任何其他裝置 (實體 TPM/HSM/X.509，或模擬 X.509 憑證)：

        ```cmd/sh
        cmake -Duse_prov_client:BOOL=ON ..
        ```


您現在即可使用 SDK 來建置您的裝置註冊程式碼。 
 
<a id="extractsecurity"></a> 

## <a name="extract-the-security-artifacts"></a>擷取安全構件 

下一個步驟是為您的裝置所用的證明機制擷取安全構件。 

### <a name="physical-devices"></a>實體裝置 

根據您所建置的 SDK 是使用實體 TPM/HSM 的證明還是使用 X.509 憑證，依照下列方式收集安全性構件：

- 對於 TPM 裝置，您必須從 TPM 晶片製造商找出與其相關聯的**簽署金鑰**。 您可以將簽署金鑰進行雜湊處理，以衍生 TPM 裝置的唯一**註冊識別碼**。  

- 針對 X.509 裝置，您必須取得核發給裝置的憑證。 佈建服務會公開兩種類型的註冊項目，用來對使用 X.509 證明機制的裝置進行存取控制。 所需的憑證取決於您要使用的註冊類型。

    1. 個別註冊：特定單一裝置的註冊。 此類型的註冊項目需要[終端實體「分葉」憑證](concepts-security.md#end-entity-leaf-certificate)。
    1. 註冊群組：此類型的註冊項目需要中繼或根憑證。 如需詳細資訊，請參閱[使用 X.509 憑證控制對於佈建服務的裝置存取](concepts-security.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates)。

### <a name="simulated-devices"></a>模擬的裝置

根據您所建置的 SDK 是透過 TPM 還是 X.509 憑證來使用模擬裝置的證明，依照下列方式收集安全性構件：

- 對於模擬 TPM 裝置：

   1. 開啟 Windows 命令提示字元，瀏覽至 `azure-iot-sdk-c` 子目錄，然後執行 TPM 模擬器。 它會透過連接埠 2321年和 2322 上的通訊端接聽。 請勿關閉此命令視窗；您必須讓此模擬器保持執行，直到下列快速入門結束為止。 

      從 `azure-iot-sdk-c` 子目錄，執行下列命令以啟動模擬器：

      ```cmd/sh
      .\provisioning_client\deps\utpm\tools\tpm_simulator\Simulator.exe
      ```

      > [!NOTE]
      > 如果在此步驟中您使用 Git Bash 命令提示字元，您必須將反斜線變更為正斜線，例如：`./provisioning_client/deps/utpm/tools/tpm_simulator/Simulator.exe`。

   1. 使用 Visual Studio，開啟在 cmake 資料夾中產生的方案 (名為 `azure_iot_sdks.sln`)，然後使用 [建置] 功能表上的 [建置方案] 命令加以建置。

   1. 在 Visual Studio 的 [方案總管] 窗格中，瀏覽至 **Provision\_Tools** 資料夾。 以滑鼠右鍵按一下 **tpm_device_provision** 專案，然後選取 [設為起始專案]。 

   1. 使用 [偵錯] 功能表上的任一 [啟動] 命令來執行方案。 輸出視窗會顯示裝置註冊所需之 TPM 模擬器的 [登錄識別碼] 和 [簽署金鑰]。 複製這些值，供後續使用。 您可以關閉此視窗 (含有註冊識別碼和簽署金鑰)，但是讓您在步驟 1 中啟動的 TPM 模擬器視窗持續執行。

- 對於模擬 X.509 裝置：

  1. 使用 Visual Studio，開啟在 cmake 資料夾中產生的方案 (名為 `azure_iot_sdks.sln`)，然後使用 [建置] 功能表上的 [建置方案] 命令加以建置。

  1. 在 Visual Studio 的 [方案總管] 窗格中，瀏覽至 **Provision\_Tools** 資料夾。 以滑鼠右鍵按一下 **dice\_device\_enrollment** 專案，然後選取 [設為起始專案]。 
  
  1. 使用 [偵錯] 功能表上的任一 [啟動] 命令來執行方案。 在輸出視窗中，當出現提示時，針對個別註冊輸入 **i**。 輸出視窗會顯示針對您的模擬裝置在本機產生的 X.509 憑證。 將輸出 (從 -----BEGIN CERTIFICATE----- 開始並於第一個 -----END PUBLIC KEY----- 結束) 複製到剪貼簿，並確定包含這兩行文字。 您只需要輸出視窗中的第一個憑證。
 
  1. 建立名為 **X509testcert.pem** 的檔案，在您選擇的文字編輯器中開啟該檔案，並將剪貼簿內容複製到這個檔案。 儲存檔案，因為稍後註冊裝置時會用到它。 當您註冊軟體執行時，它會在自動佈建期間使用相同的憑證。    

向裝置佈建服務註冊裝置期間需要這些安全構件。 佈建服務會等候裝置開機，並於稍後的任何時間點與裝置連線。 裝置第一次開機時，用戶端 SDK 邏輯會與晶 (或模擬器) 片互動以從裝置擷取安全構件，並向裝置佈建服務確認註冊情形。 

## <a name="create-the-device-registration-software"></a>建立裝置註冊軟體

最後一個步驟是撰寫註冊應用程式，以使用裝置佈建服務用戶端 SDK 來向 IoT 中樞服務註冊裝置。 

> [!NOTE]
> 在此步驟中，我們會假設使用模擬裝置，並藉由從您的工作站執行 SDK 範例註冊應用程式來完成。 不過，如果您要建置註冊應用程式以供部署到實體裝置，則適用相同的概念。 

1. 在 Azure 入口網站中，選取您裝置佈建服務的 [概觀] 刀鋒視窗，並複製 [識別碼範圍] 值。 「識別碼範圍」會由服務產生，以保證唯一性。 識別碼範圍永遠不變，因此可用來唯一識別註冊識別碼。

    ![從入口網站刀鋒視窗擷取裝置佈建服務端點資訊](./media/tutorial-set-up-device/extract-dps-endpoints.png) 

1. 在您電腦上 Visual Studio 的 [方案總管] 中，瀏覽至 **Provision\_Samples** 資料夾。 選取名為 **prov\_dev\_client\_sample** 的範例專案，並開啟來源檔案 **prov\_dev\_client\_sample.c**。

1. 將在步驟 1 取得的 [識別碼範圍] 值指派給 `id_scope` 變數 (移除左 `[` 和右 `]` 方括號)： 

    ```c
    static const char* global_prov_uri = "global.azure-devices-provisioning.net";
    static const char* id_scope = "[ID Scope]";
    ```

    提供參考，`global_prov_uri` 變數可讓 IoT 中樞用戶端註冊 API`IoTHubClient_LL_CreateFromDeviceAuth` 與指定的裝置佈建服務執行個體進行連線。

1. 在相同檔案的 **main()** 函式中，請註解/取消註解 `hsm_type` 變數，以符合您裝置的註冊軟體 (TPM 或 X.509) 所使用的證明機制： 

    ```c
    hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    ```

1. 儲存您的變更，並藉由從 [建置] 功能表選取 [建置方案] 來重建 **prov\_dev\_client\_sample** 範例。 

1. 以滑鼠右鍵按一下 **Provision\_Samples** 資料夾底下的 **prov\_dev\_client\_sample** 專案，然後選取 [設定為起始專案]。 先不要執行範例應用程式。

> [!IMPORTANT]
> 先不要執行/啟動裝置！ 在啟動裝置之前，您必須先向裝置佈建服務註冊裝置，才能完成此程序。 下面的「後續步驟」一節將引導您前往下一篇文章。

### <a name="sdk-apis-used-during-registration-for-reference-only"></a>在註冊期間使用的 SDK API (僅供參考)

提供參考，此 SDK 會提供下列 API 以供應用程式在註冊期間使用。 這些 API 可協助裝置在開機時與裝置佈建服務進行連線和註冊。 然而，您的裝置會收到建立 IoT 中樞執行個體連線所需的資訊：

```C
// Creates a Provisioning Client for communications with the Device Provisioning Client Service.  
PROV_DEVICE_LL_HANDLE Prov_Device_LL_Create(const char* uri, const char* scope_id, PROV_DEVICE_TRANSPORT_PROVIDER_FUNCTION protocol)

// Disposes of resources allocated by the provisioning Client.
void Prov_Device_LL_Destroy(PROV_DEVICE_LL_HANDLE handle)

// Asynchronous call initiates the registration of a device.
PROV_DEVICE_RESULT Prov_Device_LL_Register_Device(PROV_DEVICE_LL_HANDLE handle, PROV_DEVICE_CLIENT_REGISTER_DEVICE_CALLBACK register_callback, void* user_context, PROV_DEVICE_CLIENT_REGISTER_STATUS_CALLBACK reg_status_cb, void* status_user_ctext)

// Api to be called by user when work (registering device) can be done
void Prov_Device_LL_DoWork(PROV_DEVICE_LL_HANDLE handle)

// API sets a runtime option identified by parameter optionName to a value pointed to by value
PROV_DEVICE_RESULT Prov_Device_LL_SetOption(PROV_DEVICE_LL_HANDLE handle, const char* optionName, const void* value)
```

您也會發現您需要先使用模擬裝置，然後測試服務安裝程式，以改進裝置佈建服務用戶端註冊應用程式。 當應用程式可以在測試環境中運作後，您可以針對特定裝置來建置此應用程式，並將可執行檔複製到裝置映像中。 

## <a name="clean-up-resources"></a>清除資源

至此，您可能有在入口網站中執行的裝置佈建和 IoT 中樞服務。 如果您想要放棄裝置佈建設定及 (或) 延遲完成本教學課程系列，建議您將其關閉以避免產生不必要的成本。

1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取您的裝置佈建服務。 在 [所有資源] 刀鋒視窗的頂端，按一下 [刪除]。  
1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取您的 IoT 中樞。 在 [所有資源] 刀鋒視窗的頂端，按一下 [刪除]。  

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建置平台特定裝置佈建服務用戶端 SDK
> * 擷取安全構件
> * 建立裝置註冊軟體

前進到下一個教學課程，以了解如何藉由向 Azure IoT 中樞裝置佈建服務註冊要讓裝置進行自動佈建，來將裝置佈建到 IoT 中樞。

> [!div class="nextstepaction"]
> [將裝置佈建到 IoT 中樞](tutorial-provision-device-to-hub.md)

