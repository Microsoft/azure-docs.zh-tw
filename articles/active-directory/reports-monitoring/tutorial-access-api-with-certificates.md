---
title: 教學課程：使用 Azure AD 報告 API 配合憑證來取得資料 | Microsoft Docs
description: 本教學課程說明如何使用 Azure AD 報告 API 配合憑證認證來取得目錄中的資料，而不需使用者介入。
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.component: report-monitor
ms.date: 05/07/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: 5c54af76fc1e145ea062c6bcb37f354a7de94781
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364167"
---
# <a name="tutorial-get-data-using-the-azure-active-directory-reporting-api-with-certificates"></a>教學課程：使用 Azure Active Directory 報告 API 配合憑證來取得資料

[Azure Active Directory (Azure AD) 報告 API](concept-reporting-api.md) 透過一組以 REST 為基礎的 API 為您提供資料的程式設計方式存取。 您可以從各種程式設計語言和工具呼叫這些 API。 如果您想在沒有使用者介入的情況下存取 Azure AD 報告 API，請務必設定您的存取權以使用憑證。

在本教學課程中，您會了解如何建立測試憑證，並使用該憑證存取 MS 圖形 API 來進行報告。 我們不建議在生產環境中使用測試憑證。 

## <a name="prerequisites"></a>必要條件

1. 首先，請完成[存取 Azure Active Directory 報告 API 的必要條件](howto-configure-prerequisites-for-reporting-api.md)。 

2. 下載並安裝 [Azure AD PowerShell V2](https://github.com/Azure/azure-docs-powershell-azuread/blob/master/docs-conceptual/azureadps-2.0/install-adv2.md)。

3. 安裝 [MSCloudIdUtils](https://www.powershellgallery.com/packages/MSCloudIdUtils/)。 此模組會提供數個公用程式 Cmdlet，包括︰
    - 驗證所需的 ADAL 程式庫
    - 從使用者、應用程式金鑰和憑證存取權杖 (使用 ADAL)
    - 處理分頁結果的圖形 API

4. 如果您第一次使用模組執行 **Install-MSCloudIdUtilsModule**，則改用 **Import-Module** Powershell 命令將其匯入。

您的工作階段看起來應該類似此畫面：

  ![Windows Powershell](./media/tutorial-access-api-with-certificates/module-install.png)
  
## <a name="create-a-test-certificate"></a>建立測試憑證

1. 使用 **New-SelfSignedCertificate** Powershell commandlet 建立測試憑證。

   ```
   $cert = New-SelfSignedCertificate -Subject "CN=MSGraph_ReportingAPI" -CertStoreLocation "Cert:\CurrentUser\My" -KeyExportPolicy Exportable -KeySpec Signature -KeyLength 2048 -KeyAlgorithm RSA -HashAlgorithm SHA256
   ```

2. 使用 **Export-Certificate** commandlet 將其匯出至憑證檔案。

   ```
   Export-Certificate -Cert $cert -FilePath "C:\Reporting\MSGraph_ReportingAPI.cer"

   ```

## <a name="register-the-certificate-in-your-app"></a>在您的應用程式中註冊憑證

1. 導覽至 [Azure 入口網站](https://portal.azure.com)，選取 [Azure Active Directory]，接著選取 [應用程式註冊]，然後從清單中選擇您的應用程式。 

2. 選取 [設定] > [金鑰]，然後選取 [上載公開金鑰]。

3. 從上一個步驟中選取憑證檔案，然後選取 [儲存]。 

4. 請記下應用程式識別碼，以及您剛剛向應用程式註冊的憑證指紋。 若要尋找指紋，請從入口網站的應用程式頁面中前往 [設定]，然後按一下 [金鑰]。 指紋將會在 [公開金鑰] 清單下方。

5. 在內嵌資訊清單編輯器中開啟應用程式資訊清單，然後使用下列結構描述將 *keyCredentials* 屬性取代為新的憑證資訊。 

   ```
   "keyCredentials": [
        {
            "customKeyIdentifier": "$base64Thumbprint", //base64 encoding of the certificate hash
            "keyId": "$keyid", //GUID to identify the key in the manifest
            "type": "AsymmetricX509Cert",
            "usage": "Verify",
            "value":  "$base64Value" //base64 encoding of the certificate raw data
        }
    ]
   ```

6. 儲存資訊清單。 
  
## <a name="get-an-access-token-for-ms-graph-api"></a>取得 MS 圖形 API 的存取權杖

1. 使用 MSCloudIdUtils PowerShell 模組中的 **Get-MSCloudIdMSGraphAccessTokenFromCert** Cmdlet，然後傳入您從上一個步驟中所取得的應用程式識別碼和指紋。 

 ![Azure 入口網站](./media/tutorial-access-api-with-certificates/getaccesstoken.png)

## <a name="use-the-access-token-to-call-the-graph-api"></a>使用存取權杖來呼叫圖形 API

1. 現在，您可以使用 Powershell 指令碼中的存取權杖來查詢圖形 API。 使用 MSCloudIDUtils 中的 **Invoke-MSCloudIdMSGraphQuery** 指令程式來列舉 signins 和 directoryAudits 端點。 此指令程式可處理多個分頁結果，並將這些結果傳送至 PowerShell 管道。

2. 查詢 directoryAudits 端點以擷取稽核記錄。 
 ![Azure 入口網站](./media/tutorial-access-api-with-certificates/query-directoryAudits.png)

3. 查詢 signins 端點以擷取登入記錄。
 ![Azure 入口網站](./media/tutorial-access-api-with-certificates/query-signins.png)

4. 您現在可以選擇將此資料匯出為 CSV，並儲存至 SIEM 系統。 您也可以在排定的工作中包裝您的指令碼，以便定期從租用戶取得 Azure AD 資料，而不必將應用程式金鑰儲存在原始程式碼中。 

## <a name="next-steps"></a>後續步驟

* [取得報告 API 的第一印象](concept-reporting-api.md)
* [稽核 API 參考](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/directoryaudit) 
* [登入活動報告 API 參考](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/signin)



