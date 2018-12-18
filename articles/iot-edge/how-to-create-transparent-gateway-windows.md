---
title: 使用 Azure IoT Edge 建立透明閘道 - Windows | Microsoft Docs
description: 使用 Azure IoT Edge 建立可以為多個裝置處理資訊的透明閘道
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 6/20/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: e9de037f886db7a48411959ef62e1e6687e54beb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984291"
---
# <a name="create-a-windows-iot-edge-device-that-acts-as-a-transparent-gateway"></a>建立作為透明閘道的 Windows IoT Edge 裝置

本文會詳細說明如何使用 IoT Edge 裝置做為透明閘道。 本文接下來的「IoT Edge 閘道」一詞是指做為透明閘道的 IoT Edge 裝置。 如需詳細資訊，請參閱[如何使用 IoT Edge 裝置作為閘道][lnk-edge-as-gateway]，其中提供概念性的概觀。

>[!NOTE]
>目前狀況：
> * 如果閘道與 IoT 中樞中斷連線，則下游裝置無法透過閘道進行驗證。
> * 已啟用 Edge 的裝置無法連線到 IoT Edge 閘道。 
> * 下游裝置無法使用檔案上傳。

建立透明閘道的困難之處，在於必須安全地將閘道連線至下游裝置。 Azure IoT Edge 可讓您使用 PKI 基礎結構，設定這些裝置之間的安全 TLS 連線。 在此案例中，我們將允許下游裝置連線至作為透明閘道的 IoT Edge 裝置。  為了維護適當的安全性，下游裝置應確認 Edge 裝置的身分識別，因為您應該只希望讓裝置連線至自己的閘道，而不是可能的惡意閘道。

您可以建立任何憑證基礎結構，來啟用裝置閘道拓撲所需的信任。 在本文中，我們假設您會使用在 IoT 中樞內用來啟用 [X.509 CA 安全性][lnk-iothub-x509]的相同憑證安裝程式，其中包含與特定 IoT 中樞相關聯的 X.509 CA 憑證 (IoT 中樞擁有者 CA)，以及使用此 CA 和 Edge 裝置的 CA 簽署的一系列憑證。

![閘道安裝][1]

閘道在連線起始期間，會向下游裝置出具其 Edge 裝置 CA 憑證。 下游裝置會檢查以確認 Edge 裝置 CA 憑證是由擁有者 CA 憑證所簽署。 此程序可讓下游裝置確認閘道是來自信任的來源。

下列步驟會逐步引導您在正確的位置建立憑證並且加以安裝。

## <a name="prerequisites"></a>必要條件
1.  在您想要作為透明閘道的 Windows 裝置上[安裝 Azure IoT Edge 執行階段][lnk-install-windows-x64]。

1. 取得 Windows 的 OpenSSL。 您可以透過多種方式來安裝 OpenSSL：

   >[!NOTE]
   >如果您已在 Windows 裝置上安裝 OpenSSL，則可以略過此步驟，但請確定您的 `%PATH%` 環境變數中有 `openssl.exe`。

   * 下載並安裝任何[第三方 OpenSSL 二進位檔](https://wiki.openssl.org/index.php/Binaries)，例如，從 [SourceForge 上的這個專案](https://sourceforge.net/projects/openssl/)下載並安裝。
   
   * 自行下載 OpenSSL 原始程式碼並在您的電腦上組建二進位檔，或是透過 [vcpkg](https://github.com/Microsoft/vcpkg) 來執行此作業。 下列指示會使用 vcpkg 下載原始程式碼，並在 Windows 電腦上編譯並安裝 OpenSSL，且各項作業都能以非常容易使用的步驟完成。

      1. 瀏覽至要安裝 vcpkg 的目錄。 以下我們將稱之為 $VCPKGDIR。 依照指示下載並安裝 [vcpkg](https://github.com/Microsoft/vcpkg)。
   
      1. vcpkg 安裝後，請從 Powershell 提示字元執行下列命令，以安裝 Windows x64 的 OpenSSL 套件。 此作業通常需要約 5 分鐘的時間。

         ```PowerShell
         .\vcpkg install openssl:x64-windows
         ```
      1. 將 `$VCPKGDIR\installed\x64-windows\tools\openssl` 新增至您的 `PATH` 環境變數，讓 `openssl.exe` 檔案可供叫用。

1. 瀏覽至您要使用的目錄。 以下我們將稱之為 $WRKDIR。  所有檔案都會在此目錄中建立。
   
   cd $WRKDIR

1.  使用下列命令取得相關指令碼，以產生所需的非生產憑證。 這些指令碼可協助您建立用來設定透明閘道的所需憑證。

      ```PowerShell
      git clone https://github.com/Azure/azure-iot-sdk-c.git
      ```

1. 將組態和指令碼檔案複製到您的工作目錄。 此外，請設定環境變數 OPENSSL_CONF 以使用 openssl_root_ca.cnf 組態檔。

   ```PowerShell
   copy azure-iot-sdk-c\tools\CACertificates\*.cnf .
   copy azure-iot-sdk-c\tools\CACertificates\ca-certs.ps1 .
   $env:OPENSSL_CONF = "$PWD\openssl_root_ca.cnf"
   ```

1. 執行下列命令，使 PowerShell 能夠執行指令碼

   ```PowerShell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted
   ```

1. 透過下列命令使用點執行，將指令碼所使用的函式導入 PowerShell 的全域命名空間中
   
   ```PowerShell
   . .\ca-certs.ps1
   ```

1. 執行下列命令以確認 OpenSSL 已正確安裝，並確定名稱不會與現有的憑證名稱衝突。 如果有問題，指令碼應會說明如何在系統上加以修正。

   ```PowerShell
   Test-CACertsPrerequisites
   ```

## <a name="certificate-creation"></a>建立憑證
1.  建立擁有者 CA 憑證和一個中繼憑證。 這些項目都位於 `$WRKDIR` 中。

      ```PowerShell
      New-CACertsCertChain rsa
      ```

1.  使用下列命令建立 Edge 裝置 CA 憑證和私密金鑰。

   >[!NOTE]
   > **請勿**使用與閘道 DNS 主機名稱相同的名稱。 這麼做將導致使用這些憑證的用戶端認證失敗。

   ```PowerShell
   New-CACertsEdgeDevice "<gateway device name>"
   ```

## <a name="certificate-chain-creation"></a>建立憑證鏈結
使用下列命令，從擁有者 CA 憑證、中繼憑證和 Edge 裝置 CA 憑證建立憑證鏈結。 將它放入鏈結檔案中，可讓您輕鬆地將其安裝在作為透明閘道的 Edge 裝置上。

   ```PowerShell
   Write-CACertsCertificatesForEdgeDevice "<gateway device name>"
   ```

   指令碼執行的輸出是下列憑證和金鑰：
   * `$WRKDIR\certs\new-edge-device.*`
   * `$WRKDIR\private\new-edge-device.key.pem`
   * `$WRKDIR\certs\azure-iot-test-only.root.ca.cert.pem`

## <a name="installation-on-the-gateway"></a>安裝在閘道上
1.  從 $WRKDIR 將下列檔案複製到 Edge 裝置上的任一處，我們稱之為 $CERTDIR。 如果您是在 Edge 裝置上產生憑證，請略過此步驟。

   * 裝置 CA 憑證 -  `$WRKDIR\certs\new-edge-device-full-chain.cert.pem`
   * 裝置 CA 私密金鑰 - `$WRKDIR\private\new-edge-device.key.pem`
   * 擁有者 CA - `$WRKDIR\certs\azure-iot-test-only.root.ca.cert.pem`

2.  將安全性精靈設定 yaml 檔案中的 `certificate` 屬性，設定為您放置憑證和金鑰檔案的路徑。

```yaml
certificates:
  device_ca_cert: "$CERTDIR\\certs\\new-edge-device-full-chain.cert.pem"
  device_ca_pk: "$CERTDIR\\private\\new-edge-device.key.pem"
  trusted_ca_certs: "$CERTDIR\\certs\\azure-iot-test-only.root.ca.cert.pem"
```
## <a name="deploy-edgehub-to-the-gateway"></a>將 Edge 中樞部署至閘道
Azure IoT Edge 的主要功能之一，是能夠從雲端將模組部署到您的 IoT Edge 裝置。 在本節中，您建立了看似空白的部署，但即使沒有其他模組存在，Edge 中樞也會自動新增至所有部署。 Edge 中樞是您讓 Edge 裝置作為透明閘道所需的唯一模組，因此建立空白部署就已足夠。 
1. 在 Azure 入口網站中，瀏覽至您的 IoT 中樞。
2. 移至 [IoT Edge]，並選取要作為閘道的 IoT Edge 裝置。
3. 選取 [設定模組]。
4. 選取 [下一步] 。
5. 在 [指定路由] 步驟中，您應該會有可將所有模組的所有訊息傳送至 IoT 中樞的預設路由。 如果沒有，請新增下列程式碼，然後選取 [下一步]。
   ```JSON
   {
       "routes": {
           "route": "FROM /* INTO $upstream"
       }
   }
   ```
6. 在 [檢閱範本] 步驟中，選取 [提交]。

## <a name="installation-on-the-downstream-device"></a>安裝在下游裝置上
下游裝置可以是任何使用 [Azure IoT 裝置 SDK][lnk-devicesdk] 的應用程式，例如[使用 .NET 將您的裝置連線至 IoT 中樞][lnk-iothub-getstarted]中所述的其中一個。 下游裝置應用程式必須信任**擁有者 CA** 憑證，才能驗證對閘道裝置的 TLS 連線。 通常有兩種方式可執行此步驟：在作業系統層級，或在 (適用於特定語言) 應用程式層級。

### <a name="os-level"></a>作業系統層級
在作業系統憑證存放區中安裝此憑證，將允許所有應用程式使用擁有者 CA 憑證作為信任憑證。

* Ubuntu - 以下是如何在 Ubuntu 主機上安裝 CA 憑證的範例。

   ```cmd
   sudo cp $CERTDIR/certs/azure-iot-test-only.root.ca.cert.pem  /usr/local/share/ca-certificates/azure-iot-test-only.root.ca.cert.pem.crt
   sudo update-ca-certificates
   ```
 
    您應該會看到訊息指出：「正在更新 /etc/ssl/certs 中的憑證...已新增 1 個，已移除 0 個；完成。」

* Windows - 以下是如何在 Windows 主機上安裝 CA 憑證的範例。
  * 從 [開始] 功能表中，輸入「管理電腦憑證」。 這應該會啟動名為 `certlm` 的公用程式。
  * 瀏覽至 [憑證本機電腦] --> [受信任的根憑證] --> [憑證] --> 按一下滑鼠右鍵 --> [所有工作] --> [匯入]，以啟動 [憑證匯入精靈]。
  * 依照指示執行步驟，並匯入憑證檔案 $CERTDIR/certs/azure-iot-test-only.root.ca.cert.pem。
  * 完成時，您應該會看到「已成功匯入」訊息。

### <a name="application-level"></a>應用程式層級
針對 .NET 應用程式，您可以新增下列程式碼片段以信任 PEM 格式的憑證。 使用 `$CERTDIR\certs\azure-iot-test-only.root.ca.cert.pem` 初始化變數 `certPath`。

   ```
   using System.Security.Cryptography.X509Certificates;

   ...

   X509Store store = new X509Store(StoreName.Root, StoreLocation.CurrentUser);
   store.Open(OpenFlags.ReadWrite);
   store.Add(new X509Certificate2(X509Certificate2.CreateFromCertFile(certPath)));
   store.Close();
   ```

## <a name="connect-the-downstream-device-to-the-gateway"></a>將下游裝置連線至閘道
您必須使用參照閘道裝置主機名稱的連接字串，來初始化 IoT 中樞裝置 SDK。 這會透過將 `GatewayHostName` 屬性附加至您裝置的連接字串來完成。 例如，以下是裝置的裝置連接字串範例，我們在其中附加 `GatewayHostName` 屬性：

   ```
   HostName=yourHub.azure-devices.net;DeviceId=yourDevice;SharedAccessKey=XXXYYYZZZ=;GatewayHostName=mygateway.contoso.com
   ```

   >[!NOTE]
   >這是範例命令，可以測試所有項目是否都已正確設定。 您應該會看到訊息指出：「驗證完成」。
   >
   >openssl s_client -connect mygateway.contoso.com:8883 -CAfile $CERTDIR/certs/azure-iot-test-only.root.ca.cert.pem -showcerts

## <a name="routing-messages-from-downstream-devices"></a>從下游裝置路由傳送訊息
IoT Edge 執行階段可以路由傳送從下游裝置送來的訊息，就像路由傳送從模組送來的訊息一樣。 這可讓您在將任何資料傳送到雲端之前，在執行於閘道的模組中執行分析。 以下路由會用來將下游裝置 `sensor` 送來的訊息，傳送到名為 `ai_insights` 的模組。

   ```json
   { "routes":{ "sensorToAIInsightsInput1":"FROM /messages/* WHERE NOT IS_DEFINED($connectionModuleId) INTO BrokeredEndpoint(\"/modules/ai_insights/inputs/input1\")", "AIInsightsToIoTHub":"FROM /messages/modules/ai_insights/outputs/output1 INTO $upstream" } }
   ```

請參閱[模組組合文章][lnk-module-composition]，以取得訊息路由的詳細資訊。

[!INCLUDE [](../../includes/iot-edge-extended-offline-preview.md)]

## <a name="next-steps"></a>後續步驟
[了解開發 IoT Edge 模組的需求和工具][lnk-module-dev]。

<!-- Images -->
[1]: ./media/how-to-create-transparent-gateway/gateway-setup.png

<!-- Links -->
[lnk-install-windows-x64]: ./how-to-install-iot-edge-windows-with-windows.md
[lnk-module-composition]: ./module-composition.md
[lnk-devicesdk]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-tutorial1-win]: tutorial-simulate-device-windows.md
[lnk-tutorial1-lin]: tutorial-simulate-device-linux.md
[lnk-edge-as-gateway]: ./iot-edge-as-gateway.md
[lnk-module-dev]: module-development.md
[lnk-iothub-getstarted]: ../iot-hub/quickstart-send-telemetry-dotnet.md
[lnk-iothub-x509]: ../iot-hub/iot-hub-x509ca-overview.md
[lnk-iothub-secure-deployment]: ../iot-hub/iot-hub-security-deployment.md
[lnk-iothub-tokens]: ../iot-hub/iot-hub-devguide-security.md#security-tokens
[lnk-iothub-throttles-quotas]: ../iot-hub/iot-hub-devguide-quotas-throttling.md
[lnk-iothub-devicetwins]: ../iot-hub/iot-hub-devguide-device-twins.md
[lnk-iothub-c2d]: ../iot-hub/iot-hub-devguide-messages-c2d.md
[lnk-ca-scripts]: https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md
[lnk-modbus-module]: https://github.com/Azure/iot-edge-modbus
