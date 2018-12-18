---
title: Azure IoT 中樞中的 X.509 安全性教學課程 | Microsoft Docs
description: 開始在模擬環境中使用您 Azure IoT 中樞中以 X.509 作為基礎的安全性。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/10/2017
ms.author: dobett
ms.openlocfilehash: 19f6f5d360981c743d819da81eb2f68db1853c8b
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43783112"
---
# <a name="set-up-x509-security-in-your-azure-iot-hub"></a>在您的 Azure IoT 中樞中設定 X.509 安全性

本教學課程中會使用 X.509 憑證驗證來模擬您保護 Azure IoT 中樞所需的步驟。 為了說明起見，我們將示範如何使用開放原始碼工具 OpenSSL 在 Windows 電腦上本機建立憑證。 建議您僅將本教學課程用於測試目的。 針對生產環境，您應該向根憑證授權單位 (CA) 購買憑證。 

## <a name="prerequisites"></a>必要條件
本教學課程需要您準備下列資源：

- 您已使用 Azure 訂用帳戶建立 IoT 中樞。 如需詳細步驟，請參閱[透過入口網站建立 IoT 中樞](iot-hub-create-through-portal.md)。 
- 您已在電腦上安裝 Visual Studio 2015 或 [Visual Studio 2017](https://www.visualstudio.com/vs/)。 

<a id="getcerts"></a>

## <a name="get-x509-ca-certificates"></a>取得 X.509 CA 憑證
「IoT 中樞」中的 X.509 憑證型安全性會要求您從 [X.509 憑證鏈結](https://en.wikipedia.org/wiki/X.509#Certificate_chains_and_cross-certification)開始著手，此鏈結包含根憑證，以及向上包含任何中繼憑證，一直到分葉憑證為止。 

您可以選擇下列其中一個方式來取得您的憑證：
- 從根憑證授權單位 (CA) 購買 X.509 憑證。 建議用於生產環境。
或者，
- 使用第三方工具建立您自己的 X.509 憑證，例如 [OpenSSL](https://www.openssl.org/)。 這適合進行測試和開發。 如需使用 PowerShell 或 Bash 產生測試 CA 憑證的相關資訊，請參閱[管理用於範例和教學課程的測試 CA 憑證](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)產生的相關資訊。 本教學課程的其餘部分會使用依照[管理用於範例和教學課程的測試 CA 憑證](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)中的指示而產生的測試 CA 憑證。


<a id="registercerts"></a>

## <a name="register-x509-ca-certificates-to-your-iot-hub"></a>向 IoT 中樞註冊 X.509 CA 憑證

這些步驟示範如何透過入口網站將新的「憑證授權單位」新增至您的 IoT 中樞。

1. 在 Azure 入口網站中，瀏覽至您的 IoT 中樞，並開啟 [設定] > [憑證] 功能表。 
2. 按一下 [新增] 以新增新的憑證。
3. 將好記的顯示名稱輸入到您的憑證。 從您的電腦中選取前一個區段中所建立名為 RootCA.cer 的根憑證檔案。 按一下 [上傳] 。
4. 一旦取得您的憑證已成功上傳的通知後，請按一下 [儲存]。

    ![Upload certificate](./media/iot-hub-security-x509-get-started/add-new-cert.png)  

   這會在 [憑證總管] 清單中顯示您的憑證。 請注意，這個憑證的 [狀態] 是 [未驗證]。

5. 按一下您在上一個步驟中新增的憑證。

6. 在 [憑證詳細資料] 刀鋒視窗中，按一下 [產生驗證碼]。

7. 它會建立**驗證碼**來驗證憑證的擁有權。 將程式碼複製到剪貼簿。 

   ![驗證憑證](./media/iot-hub-security-x509-get-started/verify-cert.png)  

8. 現在，您必須使用與 X.509 憑證相關聯的私密金鑰來簽署這個*驗證碼*，如此就會產生簽章。 有一些工具可用來執行此簽署程序，例如 OpenSSL。 這稱為[所有權的證明](https://tools.ietf.org/html/rfc5280#section-3.1)。 [管理用於範例和教學課程的測試 CA 憑證](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)中的步驟 3 會產生驗證碼。
 
9. 將步驟 8 所產生的簽章上傳至入口網站中的 IoT 中樞。 在 Azure 入口網站的 [憑證詳細資料] 刀鋒視窗中，瀏覽至**驗證憑證 .pem 或.cer 檔案**，並選取簽章，例如，使用它旁邊的 [檔案總管] 圖示，由範例 PowerShell 命令所建立的 VerifyCert4.cer。

10. 一旦憑證上傳成功後，按一下 [確認]。 憑證的 [狀態]在 [憑證]刀鋒視窗中會變更為**_已驗證_**。 如果無法自動更新，請按一下 [重新整理]。

   ![上傳憑證驗證](./media/iot-hub-security-x509-get-started/upload-cert-verification.png)  


<a id="createdevice"></a>

## <a name="create-an-x509-device-for-your-iot-hub"></a>建立 IoT 中樞的 X.509 裝置

1. 在 Azure 入口網站中，瀏覽至您 IoT 中樞的 [Device Explorer]。

2. 按一下 [新增] 以新增新的裝置。 

3. 為 [裝置識別碼] 提供好記的顯示名稱，然後選取 [X.509 CA 簽署] 作為 [驗證類型]。 按一下 [檔案] 。

   ![在入口網站中建立 X.509 裝置](./media/iot-hub-security-x509-get-started/create-x509-device.png)



<a id="authenticatedevice"></a>

## <a name="authenticate-your-x509-device-with-the-x509-certificates"></a>使用 X.509 憑證驗證您的 X.509 裝置

若要驗證您的 X.509 裝置，您必須先使用 CA 憑證簽署裝置。 分葉裝置的簽章通常會在製造工廠完成，並在此據以啟用製造工具。 因為裝置會從一個製造商送至另一個製造商，每個製造商的簽署動作都會擷取作為鏈結內的中繼憑證。 最終結果是從 CA 憑證到裝置分葉憑證的憑證鏈結。 [管理用於範例和教學課程的測試 CA 憑證](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)中的步驟 4 會產生裝置憑證。

接下來，我們將示範如何建立 C# 應用程式來模擬為您 IoT 中樞註冊的 X.509 裝置。 我們會將氣溫和溼度值從模擬裝置傳送至您的中樞。 請注意，本教學課程中，我們只會建立裝置應用程式。 它會保留給讀者作為練習建立 IoT 中樞服務應用程式，將回應傳送給此模擬裝置所傳送的事件。 C# 應用程式假設您已依照[管理用於範例和教學課程的測試 CA 憑證](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)中的步驟操作。

1. 在 Visual Studio 中，使用 [主控台應用程式] 專案範本，建立新的 Visual C# Windows 傳統桌面專案。 將專案命名為 **SimulateX509Device**。
   ![在 Visual Studio 中建立 X.509 裝置專案](./media/iot-hub-security-x509-get-started/create-device-project.png)

2. 在 [方案總管] 中，以滑鼠右鍵按一下 **SimulateX509Device** 專案，然後按一下 [管理 NuGet 套件]。在 [NuGet 套件管理員] 視窗中，選取 [瀏覽]，並搜尋 **microsoft.azure.devices.client**。 選取 [安裝] 來安裝 **Microsoft.Azure.Devices.Client** 套件，並接受使用規定。 此程序會下載及安裝 Azure IoT 裝置 SDK NuGet 套件與其相依項目，並新增對它的參考。
   ![在 Visual Studio 中新增裝置 SDK NuGet 套件](./media/iot-hub-security-x509-get-started/device-sdk-nuget.png)

3. 在 Program.cs 檔案開頭處新增下列程式碼：
    
    ```CSharp
        using Microsoft.Azure.Devices.Client;
        using Microsoft.Azure.Devices.Shared;
        using System.Security.Cryptography.X509Certificates;
    ```

4. 在**程式**類別內新增下列程式碼：
    
    ```CSharp
        private static int MESSAGE_COUNT = 5;
        private const int TEMPERATURE_THRESHOLD = 30;
        private static String deviceId = "<your-device-id>";
        private static float temperature;
        private static float humidity;
        private static Random rnd = new Random();
    ```
   使用您在前一節中使用的易記裝置名稱來取代 _<your_device_id>_ 預留位置。

5. 新增下列函式來建立氣溫和溼度的隨機數字，並將這些值傳送至中樞：
    ```CSharp
    static async Task SendEvent(DeviceClient deviceClient)
    {
        string dataBuffer;
        Console.WriteLine("Device sending {0} messages to IoTHub...\n", MESSAGE_COUNT);

        for (int count = 0; count < MESSAGE_COUNT; count++)
        {
            temperature = rnd.Next(20, 35);
            humidity = rnd.Next(60, 80);
            dataBuffer = string.Format("{{\"deviceId\":\"{0}\",\"messageId\":{1},\"temperature\":{2},\"humidity\":{3}}}", deviceId, count, temperature, humidity);
            Message eventMessage = new Message(Encoding.UTF8.GetBytes(dataBuffer));
            eventMessage.Properties.Add("temperatureAlert", (temperature > TEMPERATURE_THRESHOLD) ? "true" : "false");
            Console.WriteLine("\t{0}> Sending message: {1}, Data: [{2}]", DateTime.Now.ToLocalTime(), count, dataBuffer);

            await deviceClient.SendEventAsync(eventMessage);
        }
    }
    ```

6. 最後，將下列程式碼新增至 **Main** 函式，依您的設定所要求，將 _device-id_、_your-iot-hub-name_ 和 _absolute-path-to-your-device-pfx-file_ 等預留位置加以取代。
    ```CSharp
    try
    {
        var cert = new X509Certificate2(@"<absolute-path-to-your-device-pfx-file>", "1234");
        var auth = new DeviceAuthenticationWithX509Certificate("<device-id>", cert);
        var deviceClient = DeviceClient.Create("<your-iot-hub-name>.azure-devices.net", auth, TransportType.Amqp_Tcp_Only);

        if (deviceClient == null)
        {
            Console.WriteLine("Failed to create DeviceClient!");
        }
        else
        {
            Console.WriteLine("Successfully created DeviceClient!");
            SendEvent(deviceClient).Wait();
        }

        Console.WriteLine("Exiting...\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```
   此程式碼會連線到 IoT 中樞，方法為建立 X.509 裝置的連接字串。 一旦成功連線後，就會將氣溫和溼度事件傳送至中樞，並等候其回應。 
7. 此應用程式會存取 .pfx 檔案，因此您可能必須在 Admin 模式中加以執行。 建置 Visual Studio 解決方案。 以**管理員**身分開啟新的命令視窗，並瀏覽至包含此解決方案的資料夾。 瀏覽至 bin/Debug 解決方案資料夾內的路徑。 從 _Admin_ 命令視窗執行應用程式 **SimulateX509Device.exe**。 您應該會看到裝置成功連線至中樞並傳送事件。 
   ![執行裝置應用程式](./media/iot-hub-security-x509-get-started/device-app-success.png)

## <a name="see-also"></a>另請參閱
若要深入了解如何保護您的 IoT 解決方案，請參閱︰

* [IoT 安全性最佳做法][lnk-security-best-practices]
* [IoT 安全性架構][lnk-security-architecture]
* [保護您的 IoT 部署][lnk-security-deployment]

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置][lnk-iotedge]

[lnk-security-best-practices]: ../iot-fundamentals/iot-security-best-practices.md
[lnk-security-architecture]: ../iot-fundamentals/iot-security-architecture.md
[lnk-security-deployment]: ../iot-fundamentals/iot-security-deployment.md

[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
