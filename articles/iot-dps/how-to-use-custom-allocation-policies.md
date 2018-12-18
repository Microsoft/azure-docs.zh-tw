---
title: 如何將自訂配置原則用於 Azure IoT 中樞裝置佈建服務 | Microsoft Docs
description: 如何將自訂配置原則用於 Azure IoT 中樞裝置佈建服務
author: wesmc7777
ms.author: wesmc
ms.date: 08/15/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 89cb44366d4752052d990a1506482c9108cde103
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161693"
---
# <a name="how-to-use-custom-allocation-policies"></a>如何使用自訂配置原則


自訂配置原則可讓您進一步掌控將裝置指派給 IoT 中樞的方式。 此作業可藉由在 [Azure 函式](../azure-functions/functions-overview.md)中使用自訂程式碼將裝置指派給 IoT 中樞來完成。 裝置佈建服務會呼叫您的 Azure 函式程式碼，提供關於裝置和註冊的所有資訊。 您的函式程式碼會執行並傳回用來佈建裝置的 IoT 中樞資訊。

藉由使用自訂配置原則，您可在裝置佈建服務所提供的原則不符合案例的需求時，定義您自己的配置原則。

例如，或許您想要檢查裝置在佈建期間所使用的憑證，並根據憑證屬性將裝置指派給 IoT 中樞。 或許您將裝置的資訊儲存在資料庫中，而需要查詢資料庫以判斷應將裝置指派給哪個 IoT 中樞。


本文將示範如何使用以 C# 撰寫的 Azure 函式設定自訂配置原則。 我們會建立兩個新的 IoT 中樞，分別代表 *Contoso 烤箱部門*和 *Contoso 熱泵部門*。 要求佈建的裝置必須具有包含下列其中一個尾碼的註冊識別碼，才能進行佈建：

- **-contoso-tstrsd-007**：Contoso 烤箱部門
- **-contoso-hpsd-088**：Contoso 熱泵部門

裝置會根據註冊識別碼的前述必要尾碼之一進行佈建。 我們將使用 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) 中包含的佈建範例來模擬這些裝置。 

您將在本文中執行下列步驟：

* 使用 Azure CLI 建立兩個 Contoso 部門 IoT 中樞 (**Contoso 烤箱部門**和 **Contoso 熱泵部門**)
* 使用 Azure 函式為自訂配置原則建立新的群組註冊
* 建立兩個裝置模擬的裝置金鑰。
* 設定適用於 Azure IoT C SDK 的開發環境
* 模擬裝置，以觀察它們根據自訂配置原則的範例程式碼進行佈建的情形


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

* 完成[使用 Azure 入口網站設定 IoT 中樞裝置佈建服務](./quick-setup-auto-provision.md)快速入門。
* 啟用[「使用 C++ 進行桌面開發」](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/)工作負載的 Visual Studio 2015 或 [Visual Studio 2017](https://www.visualstudio.com/vs/)。
* 已安裝最新版的 [Git](https://git-scm.com/download/)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-two-divisional-iot-hubs"></a>建立兩個部門 IoT 中樞

在本節中，您將使用 Azure Cloud Shell 建立兩個新的 IoT 中樞，分別代表 **Contoso 烤箱部門**和 **Contoso 熱泵部門**。

1. 在 Azure Cloud Shell 中使用 [az group create](/cli/azure/group#az-group-create) 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

    下列範例會在 eastus 區域中建立名為 contoso-us-resource-group 的資源群組。 建議您將此群組用於在本文中建立的所有資源。 透過此方法，您在完成作業後將可更容易清除資源。

    ```azurecli-interactive 
    az group create --name contoso-us-resource-group --location eastus
    ```

2. 在 Azure Cloud Shell 中使用 [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create) 命令建立 **Contoso 烤箱部門** IoT 中樞。 此 IoT 中樞將會新增至 *contoso-us-resource-group*。

    下列範例會在 eastus 位置中建立名為 contoso-toasters-hub-1098 的 IoT 中樞。 您必須使用自己的唯一中樞名稱。 請在中樞名稱中設定您自己的尾碼來取代 **1098**。 自訂配置原則的範例程式碼在中樞名稱中必須要有 `-toasters-`。

    ```azurecli-interactive 
    az iot hub create --name contoso-toasters-hub-1098 --resource-group contoso-us-resource-group --location eastus --sku S1
    ```
    
    此命令可能需要數分鐘才能完成。

3. 在 Azure Cloud Shell 中使用 [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create) 命令建立 **Contoso 熱泵部門** IoT 中樞。 此 IoT 中樞也會新增至 *contoso-us-resource-group*。

    下列範例會在 eastus 位置中建立名為 contoso-heatpumps-hub-1098 的 IoT 中樞。 您必須使用自己的唯一中樞名稱。 請在中樞名稱中設定您自己的尾碼來取代 **1098**。 自訂配置原則的範例程式碼在中樞名稱中必須要有 `-heatpumps-`。

    ```azurecli-interactive 
    az iot hub create --name contoso-heatpumps-hub-1098 --resource-group contoso-us-resource-group --location eastus --sku S1
    ```
    
    此命令可能也需要數分鐘才能完成。




## <a name="create-the-enrollment"></a>建立註冊

在本節中，您將建立使用自訂配置原則的新註冊群組。 為了簡單起見，本文將在註冊中使用[對稱金鑰證明](concepts-symmetric-key-attestation.md)。 如需更安全的解決方案，請考慮使用 [X.509 憑證證明](concepts-security.md#x509-certificates)與信任鏈結。

1. 登入 [Azure 入口網站](http://portal.azure.com)，並開啟您的裝置佈建服務執行個體。

2. 選取 [管理註冊] 索引標籤，然後按一下頁面頂端的 [新增註冊群組] 按鈕。 

3. 在 [新增註冊群組] 上輸入下列資訊，然後按一下 [儲存] 按鈕。

    **群組名稱**：輸入 **contoso-custom-allocated-devices**。

    **證明類型**：選取 [對稱金鑰]。

    **自動產生金鑰**：此核取方塊應已勾選。

    **選取要如何將裝置指派給中樞**：選取 [自訂 (使用 Azure 函式)]。

    ![為對稱金鑰證明新增自訂配置註冊群組](./media/how-to-use-custom-allocation-policies/create-custom-allocation-enrollment.png)


4. 在 [新增註冊群組] 上按一下 [連結新的 IoT 中樞]，以連結您新的部門 IoT 中樞。 

    您必須為這兩個部門 IoT 中樞執行此步驟。

    **訂用帳戶**：如果您有多個訂用帳戶，請選擇您用來建立部門 IoT 中樞的訂用帳戶。

    **IoT 中樞**：選取您建立的其中一個部門中樞。

    **存取原則**：選擇 **iothubowner**。

    ![連結部門 IoT 中樞與佈建服務](./media/how-to-use-custom-allocation-policies/link-divisional-hubs.png)


5. 在 [新增註冊群組] 上，在兩個部門 IoT 中樞都已連結後，您必須以 IoT 中樞群組的形式將其選取作為註冊群組，如下所示：

    ![建立註冊的部門中樞群組](./media/how-to-use-custom-allocation-policies/enrollment-divisional-hub-group.png)


6. 在 [新增註冊群組] 上向下捲動至 [選取 Azure 函式] 區段，然後按一下 [建立新的函式應用程式]。

7. 在開啟的 [函式應用程式] 建立頁面上，為您的新函式輸入下列設定，然後按一下 [建立]：

    **應用程式名稱**：輸入唯一的函式應用程式名稱。 在此範例中顯示為 **contoso-function-app-1098**。

    **資源群組**：選取 [使用現有的] 和**contoso-us-resource-group**，以將本文中建立的所有資源保存在一起。

    **Application Insights**：在此練習中，您可以關閉這項功能。

    ![建立函式應用程式](./media/how-to-use-custom-allocation-policies/function-app-create.png)


8. 回到您的 [新增註冊群組] 頁面，確定已選取新的函式應用程式。 您可能必須重新選取訂用帳戶，以重新整理函式應用程式清單。

    在選取新的函式應用程式之後，請按一下 [建立新的函式]。

    ![建立函式應用程式](./media/how-to-use-custom-allocation-policies/click-create-new-function.png)

    您新的函式應用程式將會開啟。

9. 在函式應用程式上按一下，以建立新的函式

    ![建立函式應用程式](./media/how-to-use-custom-allocation-policies/new-function.png)

    針對新的函式，使用 **CSharp** 語言以預設設定建立新的 **Webhook + API**。 按一下 [建立此函式]。

    這會建立新的 C# 函式，名為 **HttpTriggerCSharp1**。

10. 將新 C# 函式的程式碼取代為下列程式碼，然後按一下 [儲存]：    

    ```C#
    #r "Newtonsoft.Json"
    using System.Net;
    using System.Text;
    using Newtonsoft.Json;

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        // Just some diagnostic logging
        log.Info("C# HTTP trigger function processed a request.");
        log.Info("Request.Content:...");    
        log.Info(req.Content.ReadAsStringAsync().Result);

        // Get request body
        dynamic data = await req.Content.ReadAsAsync<object>();

        // Get registration ID of the device
        string regId = data?.deviceRuntimeContext?.registrationId;

        string message = "Uncaught error";
        bool fail = false;
        ResponseObj obj = new ResponseObj();

        if (regId == null)
        {
            message = "Registration ID not provided for the device.";
            log.Info("Registration ID : NULL");
            fail = true;
        }
        else
        {
            string[] hubs = data?.linkedHubs.ToObject<string[]>();

            // Must have hubs selected on the enrollment
            if (hubs == null)
            {
                message = "No hub group defined for the enrollment.";
                log.Info("linkedHubs : NULL");
                fail = true;
            }
            else
            {
                // This is a Contoso Toaster Model 007
                if (regId.Contains("-contoso-tstrsd-007"))
                {
                    //Find the "-toasters-" IoT hub configured on the enrollment
                    foreach(string hubString in hubs)
                    {
                        if (hubString.Contains("-toasters-"))
                            obj.iotHubHostName = hubString;
                    }

                    if (obj.iotHubHostName == null)
                    {
                        message = "No toasters hub found for the enrollment.";
                        log.Info(message);
                        fail = true;
                    }
                }
                // This is a Contoso Heat pump Model 008
                else if (regId.Contains("-contoso-hpsd-088"))
                {
                    //Find the "-heatpumps-" IoT hub configured on the enrollment
                    foreach(string hubString in hubs)
                    {
                        if (hubString.Contains("-heatpumps-"))
                            obj.iotHubHostName = hubString;
                    }

                    if (obj.iotHubHostName == null)
                    {
                        message = "No heat pumps hub found for the enrollment.";
                        log.Info(message);
                        fail = true;
                    }
                }
                // Unrecognized device.
                else
                {
                    fail = true;
                    message = "Unrecognized device registration.";
                    log.Info("Unknown device registration");
                }
            }
        }

        return (fail)
            ? req.CreateResponse(HttpStatusCode.BadRequest, message)
            : new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new StringContent(JsonConvert.SerializeObject(obj, Formatting.Indented), Encoding.UTF8, "application/json")
            };
    }    

    public class DeviceTwinObj
    {
        public string deviceId {get; set;}
    }

    public class ResponseObj
    {
        public string iotHubHostName {get; set;}
        public string IoTHub {get; set;}
        public DeviceTwinObj initialTwin {get; set;}
        public string[] linkedHubs {get; set;}
        public string enrollment {get; set;}
    }
    ```


11. 回到您的 [新增註冊群組] 頁面，並確定已選取新函式。 您可能必須重新選取函式應用程式，以重新整理函式清單。

    在選取新的函式後，請按一下 [儲存] 以儲存註冊群組。

    ![最後，儲存註冊群組](./media/how-to-use-custom-allocation-policies/save-enrollment.png)


12. 儲存註冊之後，請重新加以開啟，並記下 [主要金鑰]。 您必須先儲存註冊，才能產生金鑰。 此金鑰將在後續用來產生模擬裝置的唯一裝置金鑰。


## <a name="derive-unique-device-keys"></a>衍生唯一的裝置金鑰

在本節中，您將建立兩個裝置金鑰。 一個金鑰將用於模擬的烤箱裝置。 另一個金鑰將用於模擬的熱泵裝置。

為了產生裝置金鑰，您將使用先前記下的 [主要金鑰]，為每個裝置計算裝置註冊識別碼的 [HMAC-SHA256](https://wikipedia.org/wiki/HMAC)，並將結果轉換為 Base64 格式。 如需關於為註冊群組建立衍生裝置金鑰的詳細資訊，請參閱[對稱金鑰證明](concepts-symmetric-key-attestation.md)的群組註冊小節。

針對本文的範例，請使用下列兩個裝置註冊識別碼，並計算這兩個裝置的裝置金鑰。 這兩種註冊識別碼都具有有效的尾碼，可用於自訂配置原則的範例程式碼：

- **breakroom499-contoso-tstrsd-007**
- **mainbuilding167-contoso-hpsd-088**

#### <a name="linux-workstations"></a>Linux 工作站

如果您使用 Linux 工作站，您可以使用 openssl 來產生衍生的裝置金鑰，如下列範例所示。

1. 將 **KEY** 的值取代為您先前記下的 [主要金鑰]。

    ```bash
    KEY=oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA==

    REG_ID1=breakroom499-contoso-tstrsd-007
    REG_ID2=mainbuilding167-contoso-hpsd-088

    keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
    devkey1=$(echo -n $REG_ID1 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)
    devkey2=$(echo -n $REG_ID2 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)

    echo -e $"\n\n$REG_ID1 : $devkey1\n$REG_ID2 : $devkey2\n\n"
    ```

    ```bash
    breakroom499-contoso-tstrsd-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
    mainbuilding167-contoso-hpsd-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
    ```


#### <a name="windows-based-workstations"></a>以 Windows 為基礎的工作站

如果您使用以 Windows 為基礎的工作站，您可以使用 PowerShell 來產生衍生的裝置金鑰，如下列範例所示。

1. 將 **KEY** 的值取代為您先前記下的 [主要金鑰]。

    ```PowerShell
    $KEY='oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA=='

    $REG_ID1='breakroom499-contoso-tstrsd-007'
    $REG_ID2='mainbuilding167-contoso-hpsd-088'

    $hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
    $hmacsha256.key = [Convert]::FromBase64String($key)
    $sig1 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID1))
    $sig2 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID2))
    $derivedkey1 = [Convert]::ToBase64String($sig1)
    $derivedkey2 = [Convert]::ToBase64String($sig2)

    echo "`n`n$REG_ID1 : $derivedkey1`n$REG_ID2 : $derivedkey2`n`n"
    ```

    ```PowerShell
    breakroom499-contoso-tstrsd-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
    mainbuilding167-contoso-hpsd-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
    ```


模擬裝置會使用每個註冊識別碼的衍生裝置金鑰來執行對稱金鑰證明。




## <a name="prepare-an-azure-iot-c-sdk-development-environment"></a>準備 Azure IoT C SDK 開發環境

在此節中，您會準備用來建置 [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) 的開發環境。 SDK 包含模擬裝置的範例程式碼。 這個模擬的裝置將會嘗試在裝置開機順序期間進行佈建。

此節以 Windows 工作站為基礎來說明。 如需 Linux 範例，請參閱[如何針對多組織用戶佈建](how-to-provision-multitenant.md)中的 VM 設定。



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

2. 開啟命令提示字元或 Git Bash 殼層。 執行下列命令以複製 Azure IoT C SDK GitHub 存放庫：
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive
    ```
    此存放庫的大小目前約為 220 MB。 預期此作業需要幾分鐘的時間才能完成。


3. 在 git 存放庫的根目錄中建立 `cmake` 子目錄，並瀏覽至該資料夾。 

    ```cmd/sh
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

4. 請執行下列命令，以建置您開發用戶端平台特有的 SDK 版本。 `cmake` 目錄中會產生模擬裝置的 Visual Studio 解決方案。 

    ```cmd
    cmake -Duse_prov_client:BOOL=ON ..
    ```
    
    如果 `cmake` 找不到 C++ 編譯，您在執行上述命令時，可能會收到建置錯誤。 如果發生這種情況，請嘗試在 [Visual Studio 命令提示字元](https://docs.microsoft.com/dotnet/framework/tools/developer-command-prompt-for-vs)中執行此命令。 

    建置成功後，最後幾行輸出會類似於下列輸出：

    ```cmd/sh
    $ cmake -Duse_prov_client:BOOL=ON ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```




## <a name="simulate-the-devices"></a>模擬裝置

在本節中，您將在先前設定的 Azure IoT C SDK 中更新名為 **prov\_dev\_client\_sample** 的佈建範例。 

此範例程式碼會模擬將佈建要求傳送至裝置佈建服務執行個體的裝置開機順序。 此開機順序會使烤箱裝置接受辨識，並使用自訂配置原則指派給 IoT 中樞。

1. 在 Azure 入口網站中，選取您裝置佈建服務的 [概觀] 索引標籤，並記下 [識別碼範圍] 值。

    ![從入口網站刀鋒視窗擷取裝置佈建服務端點資訊](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. 在 Visual Studio 中，開啟先前藉由執行 CMake 而產生的 **azure_iot_sdks.sln** 解決方案檔案。 該方案檔案應該位於下列位置：

    ```
    \azure-iot-sdk-c\cmake\azure_iot_sdks.sln
    ```

3. 在 Visual Studio 的 [方案總管] 視窗中，瀏覽至 **Provision\_Samples** 資料夾。 展開名為 **prov\_dev\_client\_sample** 的範例專案。 展開 [來源檔案]，然後開啟 **prov\_dev\_client\_sample.c**。

4. 尋找 `id_scope` 常數，並以您稍早複製的**識別碼範圍**值取代該值。 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. 在相同的檔案中找出 `main()` 函式的定義。 確定 `hsm_type` 變數已設定為 `SECURE_DEVICE_TYPE_SYMMETRIC_KEY`，如下所示：

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. 以滑鼠右鍵按一下 **prov\_dev\_client\_sample** 專案，然後選取 [設定為起始專案]。 

#### <a name="simulate-the-contoso-toaster-device"></a>模擬 Contoso 烤箱裝置

1. 在 Visual Studio 的 [方案總管] 視窗中，瀏覽至 **hsm\_security\_client** 專案並將它展開。 展開 [原始程式檔]，然後開啟 **hsm\_client\_key.c**。 

    尋找 `REGISTRATION_NAME` 與 `SYMMETRIC_KEY_VALUE` 常數的宣告。 對檔案進行下列變更並儲存檔案。

    使用烤箱裝置的註冊識別碼 **breakroom499-contoso-tstrsd-007** 來更新 `REGISTRATION_NAME` 常數的值。
    
    使用您為烤箱裝置產生的裝置金鑰更新 `SYMMETRIC_KEY_VALUE` 常數的值。 此處提供的值 **JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=** 只是範例。

    ```c
    static const char* const REGISTRATION_NAME = "breakroom499-contoso-tstrsd-007";
    static const char* const SYMMETRIC_KEY_VALUE = "JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=";
    ```

2. 在 Visual Studio 功能表中，選取 [偵錯] > [啟動但不偵錯] 以執行解決方案。 出現重新建置專案的提示時，按一下 [是]，以在執行前重新建置專案。

    以下顯示輸出範例，說明模擬的烤箱裝置在成功開機後連線至佈建服務執行個體，並依自訂配置原則指派給烤箱 IoT 中樞的情形：

    ```cmd
    Provisioning API Version: 1.2.9

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-toasters-hub-1098.azure-devices.net, deviceId: breakroom499-contoso-tstrsd-007

    Press enter key to exit:
    ```


#### <a name="simulate-the-contoso-heat-pump-device"></a>模擬 Contoso 熱泵裝置

1. 回到 Visual Studio 的 [方案總管] 視窗中，瀏覽至 **hsm\_security\_client** 專案並將它展開。 展開 [原始程式檔]，然後開啟 **hsm\_client\_key.c**。 

    尋找 `REGISTRATION_NAME` 與 `SYMMETRIC_KEY_VALUE` 常數的宣告。 對檔案進行下列變更並儲存檔案。

    使用熱泵裝置的註冊識別碼 **mainbuilding167-contoso-hpsd-088** 來更新 `REGISTRATION_NAME` 常數的值。
    
    使用您為烤箱裝置產生的裝置金鑰更新 `SYMMETRIC_KEY_VALUE` 常數的值。 此處提供的值 **6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=** 只是範例。

    ```c
    static const char* const REGISTRATION_NAME = "mainbuilding167-contoso-hpsd-088";
    static const char* const SYMMETRIC_KEY_VALUE = "6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=";
    ```

7. 在 Visual Studio 功能表中，選取 [偵錯] > [啟動但不偵錯] 以執行解決方案。 出現重新建置專案的提示時，按一下 [是]，以在執行前重新建置專案。

    以下顯示輸出範例，說明模擬的熱泵裝置在成功開機後連線至佈建服務執行個體，並依自訂配置原則指派給 Contoso 熱泵 IoT 中樞的情形：

    ```cmd
    Provisioning API Version: 1.2.9

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-heatpumps-hub-1098.azure-devices.net, deviceId: mainbuilding167-contoso-hpsd-088

    Press enter key to exit:
    ```




## <a name="troubleshooting-custom-allocation-policies"></a>對自訂配置原則進行疑難排解

下表顯示預期的案例和您可能會看到的結果錯誤碼。 您可以利用此表格對 Azure Functions 的自訂配置原則失敗進行疑難排解。


| 案例 | 佈建服務的註冊結果 | 佈建 SDK 結果 |
| -------- | --------------------------------------------- | ------------------------ |
| Webhook 傳回「200 確定」，且 'iotHubHostName' 設定為有效的 IoT 中樞主機名稱 | 結果狀態：已指派  | SDK 傳回 PROV_DEVICE_RESULT_OK 與中樞資訊 |
| Webhook 傳回「200 確定」，且回應中包含 'iotHubHostName'，但設定為空字串或 Null | 結果狀態：失敗<br><br> 錯誤碼：CustomAllocationIotHubNotSpecified (400208) | SDK 傳回 PROV_DEVICE_RESULT_HUB_NOT_SPECIFIED |
| Webhook 傳回「401 未授權」 | 結果狀態：失敗<br><br>錯誤碼：CustomAllocationUnauthorizedAccess (400209) | SDK 傳回 PROV_DEVICE_RESULT_UNAUTHORIZED |
| 建立了個別註冊以停用裝置 | 結果狀態：已停用 | SDK 傳回 PROV_DEVICE_RESULT_DISABLED |
| Webhook 傳回錯誤碼 >= 429 | DPS 的協調流程將會重試特定次數。 目前的重試原則為：<br><br>&nbsp;&nbsp;- 重試計數：10<br>&nbsp;&nbsp;- 初始間隔：1s<br>&nbsp;&nbsp;- 增量：9s | SDK 將會忽略錯誤，並在指定的時間內提交另一個取得狀態訊息 |
| Webhook 傳回任何其他狀態碼 | 結果狀態：失敗<br><br>錯誤碼：CustomAllocationFailed (400207) | SDK 傳回 PROV_DEVICE_RESULT_DEV_AUTH_ERROR |


## <a name="clean-up-resources"></a>清除資源

如果您打算繼續使用在本文中建立的資源，您可加以保留。 如果您不打算繼續這些資源，請使用下列步驟刪除本文所建立的所有資源，以避免產生非必要的費用。

以下步驟假設您依照指示在名為 **contoso-us-resource-group** 的相同資源群組中建立了本文中的所有資源。

> [!IMPORTANT]
> 刪除資源群組是無法回復的動作。 資源群組和其中包含的所有資源都將永久刪除。 請確定您不會不小心刪除錯誤的資源群組或資源。 如果您在現有的資源群組內建立了 IoT 中樞，而該群組中包含您想要保留的資源，則您只需刪除 IoT 中樞本身即可，而不要刪除資源群組。
>

若要依名稱刪除資源群組：

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後按一下 [資源群組]。

2. 在 [依名稱篩選...] 文字方塊中，輸入您的資源所屬的資源群組名稱 **contoso-us-resource-group**。 

3. 在結果清單中的資源群組右側，按一下 **...**，然後按一下 [刪除資源群組]。

4. 系統將會要求您確認是否刪除資源。 再次輸入您的資源群組名稱進行確認，然後按一下 [刪除]。 片刻過後，系統便會刪除該資源群組及其所有內含的資源。

## <a name="next-steps"></a>後續步驟

- 若要深入了解如何重新佈建，請參閱 [IoT 中樞裝置重新佈建的概念](concepts-device-reprovision.md) 
- 若要深入了解如何取消佈建，請參閱[如何取消佈建先前自動佈建的裝置](how-to-unprovision-devices.md) 











