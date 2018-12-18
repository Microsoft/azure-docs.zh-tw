---
title: 使用 Java 和註冊群組將模擬 X.509 裝置佈建至 Azure IoT 中樞 | Microsoft Docs
description: Azure 教學課程 - 使用適用於 IoT 中樞裝置佈建服務的 Java 裝置和服務 SDK 和註冊群組來建立及佈建模擬 X.509 裝置
author: wesmc7777
ms.author: wesmc
ms.date: 01/04/2018
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.devlang: java
ms.custom: mvc
ms.openlocfilehash: 7f51ac0e1137bf09c220c892e2c21b154f2f2433
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39522640"
---
# <a name="create-and-provision-a-simulated-x509-device-using-java-device-and-service-sdk-and-group-enrollments-for-iot-hub-device-provisioning-service"></a>使用適用於 IoT 中樞裝置佈建服務的 Java 裝置和服務 SDK 以及註冊群組來建立及佈建模擬 X.509 裝置

這些步驟顯示如何在執行 Windows OS 的開發電腦上模擬 X.509 裝置，並使用程式碼範例和註冊群組來連線此模擬裝置與裝置佈建服務和 IoT 中樞。 

繼續之前，請務必完成[使用 Azure 入口網站設定 IoT 中樞裝置佈建服務](./quick-setup-auto-provision.md)中的步驟。


## <a name="prepare-the-environment"></a>準備環境 

1. 請確定您已在電腦上安裝 [Java SE 開發套件 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。

1. 下載並安裝 [Maven](https://maven.apache.org/install.html)。

1. 確定 `git` 已安裝在電腦上，並已新增至命令視窗可存取的環境變數。 請參閱[軟體自由保護協會的 Git 用戶端工具](https://git-scm.com/download/)以取得所要安裝的最新版 `git` 工具，其中包括 **Git Bash** (您可用來與本機 Git 存放庫互動的命令列應用程式)。 

1. 使用下列[憑證概觀](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)建立測試憑證。

    > [!NOTE]
    > 此步驟需要 [OpenSSL](https://www.openssl.org/)，您可以從來源加以建置並安裝，或從[第三方](https://wiki.openssl.org/index.php/Binaries) (例如[這個](https://sourceforge.net/projects/openssl/)) 加以下載並安裝。 如果您已建立_根_、_中繼_和_裝置_憑證，您可以略過此步驟。
    >

    1. 執行前兩個步驟，以建立您的_根_和_中繼_憑證。

    1. 登入 Azure 入口網站，按一下左側功能表上的 [所有資源] 按鈕，然後開啟您的佈建服務。

        1. 在 [裝置佈建服務摘要] 刀鋒視窗中，選取 [憑證]，然後按一下頂端的 [新增] 按鈕。

        1. 在 [新增憑證] 之下，輸入下列資訊：
            - 請輸入唯一的憑證名稱。
            - 選取您剛建立的 **_RootCA.pem_** 檔案。
            - 完成後，按一下 [儲存] 按鈕。

        ![新增憑證](./media/tutorial-group-enrollments/add-certificate.png)

        1. 選取新建立的憑證：
            - 按一下 [產生驗證碼]。 複製已產生的程式碼。
            - 執行驗證步驟。 輸入_驗證碼_，或按一下滑鼠右鍵，在您執行中的 PowerShell 視窗中貼上。  按 **Enter** 鍵。
            - 在 Azure 入口網站中選取新建立的 **_verifyCert4.pem_** 檔案。 按一下 [確認]。

            ![驗證憑證](./media/tutorial-group-enrollments/validate-certificate.png)

    1. 執行這些步驟來建立您的裝置憑證和清除資源，以完成作業。

    > [!NOTE]
    > 建立裝置憑證時，請務必在您的裝置名稱中使用小寫英數字元和連字號。
    >


## <a name="create-a-device-enrollment-entry"></a>建立裝置註冊項目

1. 開啟命令提示字元。 複製 Java SDK 程式碼範例的 GitHub 存放庫：
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```

1. 在下載的原始程式碼中，瀏覽至 _azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_ 範例資料夾。 在您選擇的編輯器中開啟檔案 _/src/main/java/samples/com/microsoft/azure/sdk/iot/ServiceEnrollmentGroupSample.java_，並新增下列詳細資料：

    1. 從入口網站針對佈建服務新增 `[Provisioning Connection String]`，如下所示：

        1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您的佈建服務。 

        1. 開啟 [共用存取原則]，並選取具有 EnrollmentWrite 權限的原則。
    
        1. 複製 [主索引鍵連接字串]。 

            ![從入口網站取得佈建連接字串](./media/tutorial-group-enrollments/provisioning-string.png)  

        1. 在範例程式碼檔案 **_ServiceEnrollmentGroupSample.java_** 中，將 `[Provisioning Connection String]` 取代為**主索引鍵連接字串**。

            ```java
            private static final String PROVISIONING_CONNECTION_STRING = "[Provisioning Connection String]";
            ```

    1. 在文字編輯器中開啟 **_RootCA.pem_** 檔案。 將**根憑證**的值指定為參數 **PUBLIC_KEY_CERTIFICATE_STRING**，如下所示：

        ```java
        private static final String PUBLIC_KEY_CERTIFICATE_STRING =
                "-----BEGIN CERTIFICATE-----\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "-----END CERTIFICATE-----\n";
        ```
 
    1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至連結到您佈建服務的 IoT 中樞。 開啟中樞的 [概觀] 索引標籤，並複製 [主機名稱]。 將此 [主機名稱] 指派給 IOTHUB_HOST_NAME 參數。

        ```java
        private static final String IOTHUB_HOST_NAME = "[Host name].azure-devices.net";
        ```

    1. 研究範例程式碼。 它會建立、更新、查詢並刪除 X.509 裝置的群組註冊。 若要確認入口網站中的註冊成功，請在 _ServiceEnrollmentGroupSample.java_ 檔案的結尾處，將下列幾行程式碼暫時標記為註解：

        ```java
        // ************************************** Delete info of enrollmentGroup ***************************************
        System.out.println("\nDelete the enrollmentGroup...");
        provisioningServiceClient.deleteEnrollmentGroup(enrollmentGroupId);
        ```

    1. 儲存 _ServiceEnrollmentGroupSample.java_ 檔案。 

1. 開啟命令視窗，並瀏覽至 **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_** 資料夾。

1. 使用此命令來建置範例程式碼：

    ```cmd\sh
    mvn install -DskipTests
    ```

1. 在命令視窗中使用這些命令來執行此範例：

    ```cmd\sh
    cd target
    java -jar ./service-enrollment-group-sample-{version}-with-deps.jar
    ```

1. 觀察成功註冊的輸出視窗。

    ![成功註冊](./media/tutorial-group-enrollments/enrollment.png) 

1. 在 Azure 入口網站中，瀏覽至您的佈建服務。 按一下 [管理註冊]。 請注意，您 X.509 裝置的群組會顯示在 [註冊群組] 索引標籤之下，包含自動產生的群組名稱。 


## <a name="simulate-the-device"></a>模擬裝置

1. 在 [裝置佈建服務摘要] 刀鋒視窗中，選取 [概觀]，並記下您的_識別碼範圍_和_佈建服務全域端點_。

    ![服務資訊](./media/tutorial-group-enrollments/extract-dps-endpoints.png)

1. 開啟命令提示字元。 瀏覽至範例專案資料夾。

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-samples/provisioning-X509-sample
    ```

1. 以下列方式輸入註冊群組資訊︰

    - 編輯 `/src/main/java/samples/com/microsoft/azure/sdk/iot/ProvisioningX509Sample.java`，使其包含您前面記下的_識別碼範圍_和_佈建服務全域端點_。 開啟您的 **_{deviceName}-public.pem_** 檔案，並加入此值作為_用戶端憑證_。開啟您的 **_{deviceName}-all.pem_** 檔案，並複製 _-----BEGIN PRIVATE KEY-----_ 與 _-----END PRIVATE KEY-----_ 之間的文字。  請以此作為您的_用戶端憑證私密金鑰_。

        ```java
        private static final String idScope = "[Your ID scope here]";
        private static final String globalEndpoint = "[Your Provisioning Service Global Endpoint here]";
        private static final ProvisioningDeviceClientTransportProtocol PROVISIONING_DEVICE_CLIENT_TRANSPORT_PROTOCOL = ProvisioningDeviceClientTransportProtocol.HTTPS;
        private static final String leafPublicPem = "<Your Public PEM Certificate here>";
        private static final String leafPrivateKey = "<Your Private PEM Key here>";
        ```

        - 您可以使用下列格式來包含憑證及金鑰：
            
            ```java
            private static final String leafPublicPem = "-----BEGIN CERTIFICATE-----\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "+XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "-----END CERTIFICATE-----\n";
            private static final String leafPrivateKey = "-----BEGIN PRIVATE KEY-----\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXX\n" +
                "-----END PRIVATE KEY-----\n";
            ```

1. 建置範例。 瀏覽至目標資料夾並執行建立的 jar 檔案。

    ```cmd/sh
    mvn clean install
    cd target
    java -jar ./provisioning-x509-sample-{version}-with-deps.jar
    ```

    ![註冊成功](./media/tutorial-group-enrollments/registration.png)

1. 在入口網站中，瀏覽到連結至您的佈建服務的 IoT 中樞，並開啟 [Device Explorer] 刀鋒視窗。 X.509 模擬裝置成功佈建到 IoT 中樞時，其裝置識別碼會出現在 [Device Explorer] 刀鋒視窗上，且 [狀態] 顯示為 [已啟用]。 請注意，如果您在執行範例裝置應用程式之前開啟了刀鋒視窗，您可能需要按一下頂端的 [重新整理] 按鈕。 

    ![已向 IoT 中樞註冊裝置](./media/tutorial-group-enrollments/hub-registration.png) 


## <a name="clean-up-resources"></a>清除資源

如果您打算繼續使用並探索裝置用戶端範例，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請使用下列步驟來刪除本快速入門建立的所有資源。

1. 在您的電腦上關閉裝置用戶端範例輸出視窗。
1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取您的裝置佈建服務。 開啟您服務的 [管理註冊] 刀鋒視窗，然後按一下 [個別註冊] 索引標籤。選取您在本快速入門中註冊的裝置之 [註冊識別碼]，然後按一下頂端的 [刪除] 按鈕。 
1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取您的 IoT 中樞。 開啟您中樞的 [IoT 裝置] 刀鋒視窗，選取您在本快速入門中註冊之裝置的 [裝置識別碼]，然後按一下頂端的 [刪除] 按鈕。


## <a name="next-steps"></a>後續步驟

在此教學課程中，您已在 Windows 機器上建立模擬 X.509 裝置，並使用 Azure IoT 中樞裝置佈建服務和註冊群組，將其佈建到 IoT 中樞。 若要深入了解您的 X.509 裝置，請繼續前往裝置概念。 

> [!div class="nextstepaction"]
> [IoT 中樞裝置佈建服務的裝置概念](concepts-device.md)
