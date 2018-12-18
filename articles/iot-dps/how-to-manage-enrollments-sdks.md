---
title: 使用 Azure 裝置佈建服務 SDK 管理裝置註冊 | Microsoft Docs
description: 如何在 IoT 中樞裝置佈建服務中使用服務 SDK 管理裝置註冊
author: yzhong94
ms.author: yizhon
ms.date: 04/04/18
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: arjmands
ms.openlocfilehash: 9a68d928c70e1e233f6de7df13441a1f688f456a
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34629842"
---
# <a name="how-to-manage-device-enrollments-with-azure-device-provisioning-service-sdks"></a>如何使用 Azure 裝置佈建服務 SDK 管理裝置註冊
「裝置註冊」會建立單一裝置或裝置群組的記錄，這些裝置可能會在某個時間點向裝置佈建服務進行註冊。 註冊記錄包含屬於該註冊一部分之裝置一開始所需的設定，包括所需的 IoT 中樞。 本文會示範如何針對您的佈建服務使用 Azure IoT 佈建服務 SDK，以程式設計方式管理裝置註冊。  這些 SDK 可從 GitHub 上取得，其位在與 Azure IoT SDK 相同的存放庫中。

## <a name="prerequisites"></a>先決條件
* 從裝置佈建服務執行個體取得連接字串。
* 取得所使用之[證明機制](concepts-security.md#attestation-mechanism)的裝置安全性構件：
    * [**信賴平台模組 (TPM)**](/azure/iot-dps/concepts-security#trusted-platform-module)：
        * 個別註冊：來自實體裝置或 TPM 模擬器的註冊識別碼和 TPM 簽署金鑰。
        * 註冊群組不適用 TPM 證明。
    * [**X.509**](/azure/iot-dps/concepts-security)：
        * 個別註冊：來自實體裝置或 SDK [DICE](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) 模擬器的[分葉憑證](/azure/iot-dps/concepts-security#leaf-certificate)。
        * 註冊群組：[CA/根憑證](/azure/iot-dps/concepts-security#root-certificate)或[中繼憑證](/azure/iot-dps/concepts-security#intermediate-certificate)，用來在實體裝置上產生裝置憑證。  也可以從 SDK DICE 模擬器產生。
* 確切的 API 呼叫可能因語言不同而有所不同。 請檢閱 GitHub 上提供的範例以取得詳細資訊：
   * [JAVA 佈建服務用戶端範例](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-samples)
   * [Node.js 佈建服務用戶端範例](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service/samples)
   * [.NET 佈建服務用戶端範例](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service/samples)

## <a name="create-a-device-enrollment"></a>建立裝置註冊
有兩種方式可供您向佈建服務註冊裝置：

* **註冊群組**是裝置群組項目，這些裝置會共用由[根憑證](https://docs.microsoft.com/azure/iot-dps/concepts-security#root-certificate)或[中繼憑證](https://docs.microsoft.com/azure/iot-dps/concepts-security#intermediate-certificate)所簽署之 X.509 憑證的通用證明機制。 對於一大批共用所需初始設定的裝置，或是全都設定為相同租用戶的裝置，我們建議使用申請群組。 請注意，您只能將使用 X.509 證明機制的裝置註冊為「註冊群組」。 

    您可以依照此工作流程使用 SDK 建立註冊群組：

    1. 註冊群組的證明機制使用 X.509 根憑證。  使用根憑證呼叫服務 SDK API ```X509Attestation.createFromRootCertificate``` 以建立註冊的證明。  X.509 根憑證會以 PEM 檔案或以字串形式提供。
    1. 使用已建立的 ```attestation``` 及唯一的 ```enrollmentGroupId``` 建立新 ```EnrollmentGroup``` 變數。  (選擇性) 您可以設定參數如 ```Device ID```、```IoTHubHostName```、```ProvisioningStatus```。
    2. 使用 ```EnrollmentGroup``` 在後端應用程式中呼叫服務 SDK API ```createOrUpdateEnrollmentGroup```，以建立註冊群組。

* **個別註冊**是可能會註冊之單一裝置的項目。 個別註冊可使用 X.509 憑證或 SAS 權杖 (從實際或虛擬的 TPM) 來作為證明機制。 對於需要唯一初始設定的裝置，或是只能透過 TPM 或虛擬 TPM 使用 SAS 權杖作為證明機制的裝置，建議您使用個別註冊。 個別申請可能會指定所需的 IoT 中樞裝置識別碼。

    您可以依照此工作流程使用 SDK 建立個別註冊：
    
    1. 選擇您的 ```attestation``` 機制，可以是 TPM 或 X.509。
        1. **TPM**：從實體裝置或 TPM 模擬器使用簽署金鑰作為輸入，您可以呼叫服務 SDK API ```TpmAttestation``` 以建立註冊證明。 
        2. **X.509**：使用用戶端憑證作為輸入，您可以呼叫服務 SDK API ```X509Attestation.createFromClientCertificate``` 以建立註冊證明。
    2. 使用已建立的 ```attestation``` 及唯一的 ```registrationId``` 建立新 ```IndividualEnrollment``` 變數作為輸入，其位於您的裝置上或從 TPM 模擬器所產生。  (選擇性) 您可以設定參數如 ```Device ID```、```IoTHubHostName```、```ProvisioningStatus```。
    3. 使用 ```IndividualEnrollment``` 在後端應用程式中呼叫服務 SDK API ```createOrUpdateIndividualEnrollment```，以建立個別註冊。

在您成功建立註冊之後，裝置佈建服務會傳回註冊結果。 此工作流程會在[先前所述的](#prerequisites)範例中示範。

## <a name="update-an-enrollment-entry"></a>更新註冊項目

在您建立註冊項目之後，建議您更新註冊。  可能的情況包括更新所需屬性、更新證明方法或撤銷裝置存取權。  有針對個別註冊及群組註冊的不同 API，但證明機制則沒有差別。

您可以依照此工作流程更新註冊項目：
* **個別註冊**：
    1. 首先使用服務 SDK API ```getIndividualEnrollment``` 從佈建服務取得最新註冊。
    2. 視需要修改最新註冊的參數。 
    3. 使用最新註冊，呼叫服務 SDK API ```createOrUpdateIndividualEnrollment``` 以更新您的註冊項目。
* **群組註冊**：
    1. 首先使用服務 SDK API ```getEnrollmentGroup``` 從佈建服務取得最新註冊。
    2. 視需要修改最新註冊的參數。
    3. 使用最新註冊，呼叫服務 SDK API ```createOrUpdateEnrollmentGroup``` 以更新您的註冊項目。

此工作流程會在[先前所述的](#prerequisites)範例中示範。

## <a name="remove-an-enrollment-entry"></a>移除註冊項目

* **個別註冊**可藉由使用 ```registrationId``` 呼叫服務 SDK API ```deleteIndividualEnrollment``` 來予以刪除。
* **群組註冊**可藉由使用 ```enrollmentGroupId``` 呼叫服務 SDK API ```deleteEnrollmentGroup``` 來予以刪除。

此工作流程會在[先前所述的](#prerequisites)範例中示範。

## <a name="bulk-operation-on-individual-enrollments"></a>個別註冊上的大量作業

您可以依照此工作流程執行大量作業，以建立、更新或移除多項個別註冊：

1. 建立包含多個 ```IndividualEnrollment``` 的變數。  針對每種語言，此變數的實作皆不相同。  檢閱 GitHub 上的大量作業範例以取得詳細資訊。
2. 使用所需作業的 ```BulkOperationMode``` 與個別註冊的變數來呼叫服務 SDK API ```runBulkOperation```。 支援下列四種模式：建立、更新、updateIfMatchEtag 和刪除。

在您成功執行作業之後，裝置佈建服務會傳回大量作業的結果。

此工作流程會在[先前所述的](#prerequisites)範例中示範。
