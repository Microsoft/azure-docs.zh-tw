---
title: 使用 Azure IoT 中樞裝置佈建服務 (.NET) 佈建裝置 | Microsoft Docs
description: 使用 Azure IoT 中樞裝置佈建服務 (.NET) 將裝置佈建到單一 IoT 中樞
author: wesmc7777
ms.author: wesmc
ms.date: 09/05/2017
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.devlang: csharp
ms.custom: mvc
ms.openlocfilehash: 1d82ccdf85b34416dc630b9fcad969d87cc53ff1
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39520661"
---
# <a name="enroll-the-device-to-an-iot-hub-using-the-azure-iot-hub-provisioning-service-client-net"></a>使用 Azure IoT 中樞佈建服務用戶端 (.NET) 將裝置註冊到 IoT 中樞

在上一個教學課程中，您已了解如何將裝置設定為連線到裝置佈建服務。 在本教學課程中，您將了解如何使用這項服務，透過**_個別註冊_** 和**_註冊群組_** 將裝置佈建到單一 IoT 中樞。 本教學課程說明如何：

> [!div class="checklist"]
> * 註冊裝置
> * 啟動裝置
> * 確認裝置已註冊

## <a name="prerequisites"></a>必要條件

在繼續之前，請務必依照[將裝置設定為使用 Azure IoT 中樞裝置佈建服務來進行佈建](./tutorial-set-up-device.md)教學課程所討論的方法，來設定裝置和其*硬體安全模組*。

* Visual Studio 2015 或 Visual Studio 2017

> [!NOTE]
> 您不需要使用 Visual Studio。 安裝 [.NET](https://www.microsoft.com/net) 即已足夠，且開發人員可以在 Windows 或 Linux 上使用其偏好的編輯器。  

本教學課程會模擬裝置資訊新增至佈建服務時，硬體製造過程中或之後的期間。 此程式碼通常會在可執行.NET 程式碼的電腦或原廠裝置上執行，且不應新增至裝置本身。


## <a name="enroll-the-device"></a>註冊裝置

此步驟涉及將裝置的唯一安全性構件新增至裝置佈建服務。 這些安全性構件如下所示：

- 若為 TPM 架構的裝置：
    - 簽署金鑰，每個 TPM 晶片或模擬都會有唯一的簽署金鑰。 如需詳細資訊，請參閱[了解 TPM 簽署金鑰](https://technet.microsoft.com/library/cc770443.aspx)。
    - 註冊識別碼，用來唯一識別命名空間/範圍中的裝置。 註冊識別碼與裝置識別碼不一定相同。 每個裝置都必須有註冊識別碼。 若為 TPM 架構的裝置，註冊識別碼可能會衍生自該 TPM 自身，例如，TPM 簽署金鑰的 SHA-256 雜湊。

- 若為 X.509 架構的裝置：
    - [核發給裝置的 X.509 憑證](https://msdn.microsoft.com/library/windows/desktop/bb540819.aspx)，其格式為 .pem 或 .cer 檔案。 若要進行個別註冊，您必須對 X.509 系統使用「分葉憑證」，若要進行群組註冊，則必須使用「根憑證」或等同的「簽署者憑證」。
    - 註冊識別碼，用來唯一識別命名空間/範圍中的裝置。 註冊識別碼與裝置識別碼不一定相同。 每個裝置都必須有註冊識別碼。 對於以 X.509 為基礎的裝置，註冊識別碼會衍生自憑證的通用名稱 (CN)。 如需這些需求的詳細資訊，請參閱[裝置概念](https://docs.microsoft.com/azure/iot-dps/concepts-device)。

有兩種方式可向裝置佈建服務註冊裝置：

- **個別註冊** 這代表可能會向裝置佈建服務註冊的單一裝置項目。 個別申請可使用 X.509 憑證或 SAS 權杖 (適用於實際或虛擬 TPM) 作為證明機制。 對於需要唯一初始設定的裝置，或是只能透過 TPM 使用 SAS 權杖作為證明機制的裝置，我們建議您使用個別註冊。 個別申請可能會指定所需的 IoT 中樞裝置識別碼。

- **註冊群組** 這代表一組共用特定證明機制的裝置。 對於共用所需初始設定的大量裝置，或是全都要進入同一個租用戶的裝置，建議您使用註冊群組。 註冊群組僅適用於 X.509，且全部共用其 X.509 信任鏈結中的簽署憑證。

### <a name="enroll-the-device-using-individual-enrollments"></a>使用個別註冊為裝置註冊

1. 在 Visual Studio 中，使用 [主控台應用程式] 專案範本建立 Visual C# 主控台應用程式專案。 將專案命名為 **DeviceProvisioning**。
    
1. 在 [方案總管] 中，以滑鼠右鍵按一下 **DeviceProvisioning** 專案，然後按一下 [管理 NuGet 套件...]。

1. 在 [NuGet 套件管理員] 視窗中選取 [瀏覽]，並搜尋 **microsoft.azure.devices.provisioning.service**。 選取項目並按一下 [安裝] 以安裝 **Microsoft.Azure.Devices.Provisioning.Service** 套件，並接受使用條款。 此程序會下載及安裝 [Azure IoT 裝置佈建服務 SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) NuGet 套件及其相依項目，並新增對它的參考。

1. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：
   
    ```csharp
    using Microsoft.Azure.Devices.Provisioning.Service;
    ```

1. 將下列欄位新增到 **Program** 類別。 將預留位置的值取代為在上一節中記下的 DPS 連接字串。
   
    ```csharp
    static readonly string ServiceConnectionString = "{DPS connection string}";

    private const string SampleRegistrationId = "sample-individual-csharp";
    private const string SampleTpmEndorsementKey =
            "AToAAQALAAMAsgAgg3GXZ0SEs/gakMyNRqXXJP1S124GUgtk8qHaGzMUaaoABgCAAEMAEAgAAAAAAAEAxsj2gUS" +
            "cTk1UjuioeTlfGYZrrimExB+bScH75adUMRIi2UOMxG1kw4y+9RW/IVoMl4e620VxZad0ARX2gUqVjYO7KPVt3d" +
            "yKhZS3dkcvfBisBhP1XH9B33VqHG9SHnbnQXdBUaCgKAfxome8UmBKfe+naTsE5fkvjb/do3/dD6l4sGBwFCnKR" +
            "dln4XpM03zLpoHFao8zOwt8l/uP3qUIxmCYv9A7m69Ms+5/pCkTu/rK4mRDsfhZ0QLfbzVI6zQFOKF/rwsfBtFe" +
            "WlWtcuJMKlXdD8TXWElTzgh7JS4qhFzreL0c1mI0GCj+Aws0usZh7dLIVPnlgZcBhgy1SSDQMQ==";
    private const string OptionalDeviceId = "myCSharpDevice";
    private const ProvisioningStatus OptionalProvisioningStatus = ProvisioningStatus.Enabled;
    ```

1. 新增下列項目，以實作裝置的註冊：

    ```csharp
    static async Task SetRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetRegistrationData");

        Attestation attestation = new TpmAttestation(SampleTpmEndorsementKey);

        IndividualEnrollment individualEnrollment = new IndividualEnrollment(SampleRegistrationId, attestation);

        individualEnrollment.DeviceId = OptionalDeviceId;
        individualEnrollment.ProvisioningStatus = OptionalProvisioningStatus;

        Console.WriteLine("\nAdding new individualEnrollment...");
        var serviceClient = ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString);

        IndividualEnrollment individualEnrollmentResult =
            await serviceClient.CreateOrUpdateIndividualEnrollmentAsync(individualEnrollment).ConfigureAwait(false);

        Console.WriteLine("\nIndividualEnrollment created with success.");
        Console.WriteLine(individualEnrollmentResult);
    }
    ```

1. 最後，在 **Main** 方法中新增下列程式碼以開啟 IoT 中樞的連線，並開始進行註冊︰
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Provisioning example");

        SetRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
        Console.ReadLine();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```
        
1. 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下您的方案，然後按一下 [設定啟始專案...]選取 [單一啟始專案]，然後選取下拉式功能表中的 **DeviceProvisioning** 專案。  

1. 執行 .NET 裝置應用程式 **DeviceProvisiong**。 它應該會設定裝置的佈建： 

    ![個別註冊執行](./media/tutorial-net-provision-device-to-hub/individual.png)

在成功註冊裝置後，您應該會看到它顯示在入口網站中，如下所示：

   ![入口網站中的成功註冊](./media/tutorial-net-provision-device-to-hub/individual-portal.png)

### <a name="enroll-the-device-using-enrollment-groups"></a>使用註冊群組為裝置註冊

> [!NOTE]
> 註冊群組範例需要 X.509 憑證。

1. 在 Visual Studio 方案總管中，開啟先前建立的 **DeviceProvisioning** 專案。 

1. 在 **Program.cs** 檔案開頭處新增下列 `using` 陳述式：
    
    ```csharp
    using System.Security.Cryptography.X509Certificates;
    ```

1. 將下列欄位新增到 **Program** 類別。 將預留位置的值取代為 X509 憑證位置。
   
    ```csharp
    private const string X509RootCertPathVar = "{X509 Certificate Location}";
    private const string SampleEnrollmentGroupId = "sample-group-csharp";
    ```

1. 將下列項目新增至 **Program.cs**，以實作群組的註冊：

    ```csharp
    public static async Task SetGroupRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetGroupRegistrationData");

        using (ProvisioningServiceClient provisioningServiceClient =
                ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString))
        {
            Console.WriteLine("\nCreating a new enrollmentGroup...");

            var certificate = new X509Certificate2(X509RootCertPathVar);

            Attestation attestation = X509Attestation.CreateFromRootCertificates(certificate);

            EnrollmentGroup enrollmentGroup = new EnrollmentGroup(SampleEnrollmentGroupId, attestation);

            Console.WriteLine(enrollmentGroup);
            Console.WriteLine("\nAdding new enrollmentGroup...");

            EnrollmentGroup enrollmentGroupResult =
                await provisioningServiceClient.CreateOrUpdateEnrollmentGroupAsync(enrollmentGroup).ConfigureAwait(false);

            Console.WriteLine("\nEnrollmentGroup created with success.");
            Console.WriteLine(enrollmentGroupResult);
        }
    }
    ```

1. 最後，在 **Main** 方法中放回下列程式碼以開啟 IoT 中樞的連線，並開始進行群組註冊︰
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Group Provisioning example");

        SetGroupRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
        Console.ReadLine();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. 執行 .NET 裝置應用程式 **DeviceProvisiong**。 它應該會設定裝置的群組佈建： 

    ![群組註冊執行](./media/tutorial-net-provision-device-to-hub/group.png)

    在成功註冊裝置群組後，您應該會看到它顯示在入口網站中，如下所示：

   ![入口網站中的成功群組註冊](./media/tutorial-net-provision-device-to-hub/group-portal.png)


## <a name="start-the-device"></a>啟動裝置

至此，下列設定已可用來註冊裝置：

1. 您的裝置或裝置群組已註冊至裝置佈建服務，而且 
2. 您的裝置已設定安全性而做好準備，且可透過應用程式使用裝置佈建服務用戶端 SDK 加以存取。

請啟動裝置，讓用戶端應用程式開始向裝置佈建服務進行註冊。  


## <a name="verify-the-device-is-registered"></a>確認裝置已註冊

裝置開機後應該會發生下列動作。 如需詳細資訊，請參閱 TPM 模擬器應用程式範例 [dps_client_sample](https://github.com/Azure/azure-iot-device-auth/blob/master/dps_client/samples/dps_client_sample/dps_client_sample.c)。 

1. 裝置會向裝置佈建服務傳送註冊要求。
2. 若為 TPM 裝置，裝置佈建服務會回傳註冊挑戰供您的裝置回應。 
3. 註冊成功後，裝置佈建服務會對裝置回傳 IoT 中樞 URI、裝置識別碼和加密金鑰。 
4. 裝置上的 IoT 中樞用戶端應用程式接著會連線到您的中樞。 
5. 成功連線至中樞後，您應該會看到裝置出現在 IoT 中樞的 **Device Explorer** 中。 

    ![在入口網站中成功連線至中樞](./media/tutorial-net-provision-device-to-hub/hub-connect-success.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 註冊裝置
> * 啟動裝置
> * 確認裝置已註冊

前進到下一個教學課程，以了解如何跨負載平衡中樞來佈建多個裝置。 

> [!div class="nextstepaction"]
> [跨負載平衡 IoT 中樞來佈建裝置](./tutorial-provision-multiple-hubs.md)
