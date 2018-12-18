---
title: 為自訂 API 新增驗證 - Azure Logic Apps | Microsoft Docs
description: 為從 Azure Logic Apps 呼叫自訂 API 的作業設定驗證
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 09/22/2017
ms.openlocfilehash: b329fb1416d28b0732e7b9ea4612f5bac8580b3a
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43132670"
---
# <a name="secure-calls-to-custom-apis-from-azure-logic-apps"></a>保護從 Azure Logic Apps 對自訂 API 發出的呼叫

若要保護對 API 發出的呼叫，您可以透過 Azure 入口網站設定 Azure Active Directory (Azure AD) 驗證，如此便不需要更新您的程式碼。 或者，您可以透過您的 API 程式碼要求並強制執行驗證。

## <a name="authentication-options-for-your-api"></a>您 API 的驗證選項

您可以以下列方式保護對自訂 API 的呼叫：

* [無程式碼變更](#no-code)：透過 Azure 入口網站使用 [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) 來保護您的 API，因此您不需要更新程式碼或重新部署您的 API。

  > [!NOTE]
  > 根據預設，您在 Azure 入口網站中開啟的 Azure AD 驗證不提供精細的授權。 例如，這項驗證會將您的 API 鎖定為僅限特定租用戶，而非特定使用者或應用程式。 

* [更新您的 API 程式碼](#update-code)：保護您的 API，方法是透過程式碼強制執行[憑證驗證](#certificate)、[基本驗證](#basic)，或 [Azure AD 驗證](#azure-ad-code)。

<a name="no-code"></a>

### <a name="authenticate-calls-to-your-api-without-changing-code"></a>驗證對 API 的呼叫，不需要變更程式碼

以下是這個方法的一般步驟：

1. 建立兩個 Azure Active Directory (Azure AD) 應用程式身分識別：一個用於邏輯應用程式，一個用於 Web 應用程式 (或 API 應用程式)。

2. 若要驗證對您 API 發出的呼叫，請使用與邏輯應用程式的 Azure AD 應用程式識別碼關聯的服務主體認證 (用戶端識別碼和祕密)。

3. 將應用程式識別碼包含在邏輯應用程式定義中。

#### <a name="part-1-create-an-azure-ad-application-identity-for-your-logic-app"></a>第 1 部分：建立 Azure AD 邏輯應用程式的應用程式識別碼

您的邏輯應用程式會使用這個 Azure AD 應用程式識別碼向 Azure AD 驗證。 您只需要為您的目錄設定一次這個身分識別。 例如，儘管您可以為每個邏輯應用程式建立唯一的身分識別，但可以選擇讓所有的 Logic Apps 使用相同的身分識別。 您可以在 Azure 入口網站或使用 [PowerShell](#powershell) 設定這些身分識別。

**在 Azure 入口網站中建立邏輯應用程式的應用程式識別碼**

1. 在 [Azure 入口網站](https://portal.azure.com "https://portal.azure.com")中，選擇 [Azure Active Directory]。 

2. 請確認您與 web 應用程式或 API 應用程式位於相同的目錄中。

   > [!TIP]
   > 若要切換目錄，請選擇您的設定檔，然後選取另一個目錄。 或者，選擇 [概觀] > [切換目錄]。

3. 在 [目錄] 功能表的 [管理] 下，選擇 [應用程式註冊] > [新增應用程式註冊]。

   > [!TIP]
   > 根據預設，應用程式註冊清單會顯示您目錄中的所有應用程式註冊。 若只要檢視您的應用程式註冊，請在搜尋方塊旁選取 [我的應用程式]。 

   ![建立新的應用程式註冊](./media/logic-apps-custom-api-authentication/new-app-registration-azure-portal.png)

4. 為您的應用程式識別碼指定名稱，並保留**應用程式類型**設為 **Web 應用程式 / API**，提供格式化為**登入 URL** 之網域的唯一字串，然後選擇 [建立]。

   ![提供應用程式識別碼的名稱和登入 URL](./media/logic-apps-custom-api-authentication/logic-app-identity-azure-portal.png)

   您針對邏輯應用程式建立的應用程式識別碼現在會出現在應用程式註冊清單中。

   ![邏輯應用程式的應用程式識別碼](./media/logic-apps-custom-api-authentication/logic-app-identity-created.png)

5. 在應用程式註冊清單中，選取新的應用程式識別碼。 複製並儲存**應用程式識別碼**，用來作為第 3 部分中邏輯應用程式的「用戶端識別碼」。

   ![複製並儲存邏輯應用程式的應用程式識別碼](./media/logic-apps-custom-api-authentication/logic-app-application-id.png)

6. 如果看不到您的應用程式識別碼設定，請選擇 [設定] 或 [所有設定]。

7. 在 [API 存取] 下，選擇 [金鑰]。 在 [描述] 下，提供您金鑰的名稱。 在 [到期時間] 下，選取您金鑰的持續時間。

   您要建立的金鑰會作為應用程式識別碼的「祕密」或邏輯應用程式的密碼。

   ![建立邏輯應用程式身分識別的金鑰](./media/logic-apps-custom-api-authentication/create-logic-app-identity-key-secret-password.png)

8. 在工具列上，選擇 [儲存]。 在 [值] 下，現在會顯示您的金鑰。 
離開 [金鑰] 頁面時，**請務必複製並儲存您的金鑰**以供稍後使用，因為系統將會隱藏此金鑰。

   當您在第 3 部分設定邏輯應用程式時，您要指定這個金鑰作為「祕密」或密碼。

   ![複製並儲存金鑰供稍後使用](./media/logic-apps-custom-api-authentication/logic-app-copy-key-secret-password.png)

<a name="powershell"></a>

**在 PowerShell 中建立邏輯應用程式的應用程式識別碼**

您可以使用 PowerShell，透過 Azure Resource Manager 來執行這項工作中。 在 PowerShell 中，執行以下命令：

1. `Switch-AzureMode AzureResourceManager`

2. `Add-AzureAccount`

3. `New-AzureADApplication -DisplayName "MyLogicAppID" -HomePage "http://mydomain.tld" -IdentifierUris "http://mydomain.tld" -Password "identity-password"`

4. 請務必複製 [租用戶識別碼] \(您 Azure AD 租用戶的 GUID)、[應用程式識別碼] 和您所使用的密碼。

如需詳細資訊，請了解如何[使用 PowerShell 建立用來存取資源的服務主體](../azure-resource-manager/resource-group-authenticate-service-principal.md)。

#### <a name="part-2-create-an-azure-ad-application-identity-for-your-web-app-or-api-app"></a>第 2 部分：建立 Web 應用程式或 API 應用程式的 Azure AD 應用程式識別碼

如果已部署 web 應用程式或 API 應用程式，您就可以開啟驗證，並在 Azure 入口網站中建立應用程式識別碼。 否則，您可以[在您使用 Azure Resource Manager 範本進行部署時開啟驗證](#authen-deploy)。 

**建立應用程式識別碼，並在 Azure 入口網站中開啟已部署應用程式的驗證**

1. 在 [Azure 入口網站](https://portal.azure.com "https://portal.azure.com")中，找出並選取您的 Web 應用程式或 API 應用程式。 

2. 在 [設定] 下，選擇 [驗證/授權]。 在 [App Service 驗證] 下，將驗證 [開啟]。 在 [驗證提供者] 下，選擇 [Azure Active Directory]。

   ![開啟驗證](./media/logic-apps-custom-api-authentication/custom-web-api-app-authentication.png)

3. 現在建立 Web 應用程式或 API 應用程式的應用程式識別碼，如下所示。 在 [Azure Active Directory 設定] 頁面上，將 [管理模式] 設定為 [快速]。 選擇 [建立新的 AD 應用程式]。 為您的應用程式識別碼提供名稱，然後選擇 **[確定]**。 

   ![建立 Web 應用程式或 API 應用程式的應用程式識別碼](./media/logic-apps-custom-api-authentication/custom-api-application-identity.png)

4. 在 [驗證/授權] 頁面上，選擇 [儲存]。

現在您必須尋找的應用程式識別碼之用戶端識別碼和租用戶識別碼，是與您的 web 應用程式或 API 應用程式相關聯。 您可以在第 3 部分中使用這些識別碼。 因此，繼續執行 Azure 入口網站的下列步驟。

**在 Azure 入口網站中尋找 web 應用程式或 API 應用程式的應用程式識別碼之用戶端識別碼和租用戶識別碼**

1. 在 [驗證提供者] 下，選擇 [Azure Active Directory]。 

   ![選擇 [Azure Active Directory]](./media/logic-apps-custom-api-authentication/custom-api-app-identity-client-id-tenant-id.png)

2. 在 [Azure Active Directory 設定] 頁面上，將 [管理模式] 設定為 [進階]。

3. 複製 [用戶端識別碼]，並儲存用於第 3 部分的 GUID。

   > [!TIP] 
   > 如果未出現 [用戶端識別碼] 和 [簽發者 URL]，請嘗試重新整理 Azure 入口網站，然後重複步驟 1。

4. 在 [簽發者 URL] 下，僅複製並儲存用於第 3 部分的 GUID。 您也可以視需要在您的 web 應用程式或 API 應用程式的部署範本中使用此 GUID。

   此 GUID 是您特定租用戶的 GUID (「租用戶 ID」)，且應該會出現在此 URL：`https://sts.windows.net/{GUID}`

5. 無須儲存變更，關閉 [Azure Active Directory 設定] 頁面。

<a name="authen-deploy"></a>

**在您使用 Azure Resource Manager 範本進行部署時開啟驗證**

您還是必須建立 web 應用程式或 API 應用程式的 Azure AD 應用程式識別碼，其與邏輯應用程式的應用程式識別碼不同。 若要建立應用程式識別碼，請遵循第 2 部分中 Azure 入口網站的先前步驟。 

您也可以遵循第 1 部分中的步驟，但請務必使用您的 web 應用程式或 API 應用程式適用於**登入 URL** 和**應用程式識別碼 URI** 的實際 `https://{URL}`。 在這些步驟中，您必須儲存用戶端識別碼和租用戶識別碼，以在您應用程式的部署範本以及用於第 3 部分中使用。

> [!NOTE]
> 當您建立 Web 應用程式或 API 應用程式的 Azure AD 應用程式身分識別時，必須使用 Azure 入口網站，而不是 PowerShell。 PowerShell commandlet 不會設定使用者登入網站的必要權限。

取得用戶端識別碼和租用戶識別碼後，請在部署範本中納入這些識別碼，作為 Web 應用程式或 API 應用程式的子資源：

``` json
"resources": [ {
    "apiVersion": "2015-08-01",
    "name": "web",
    "type": "config",
    "dependsOn": ["[concat('Microsoft.Web/sites/','parameters('webAppName'))]"],
    "properties": {
        "siteAuthEnabled": true,
        "siteAuthSettings": {
            "clientId": "{client-ID}",
            "issuer": "https://sts.windows.net/{tenant-ID}/",
        }
    }
} ]
```

若要自動將空白 web 應用程式和邏輯應用程式與 Azure Active Directory 驗證共同部署，請[在這裡檢視完整範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-custom-api/azuredeploy.json)，或在這裡按一下 [部署至 Azure]：

[![部署至 Azure](media/logic-apps-custom-api-authentication/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-logic-app-custom-api%2Fazuredeploy.json)

#### <a name="part-3-populate-the-authorization-section-in-your-logic-app"></a>第 3 部分：填入邏輯應用程式中的授權區段

前一個範本已設定此授權區段，但如果您要直接撰寫邏輯應用程式，則必須包含完整的授權區段。

在程式碼檢視中開啟您的邏輯應用程式定義、移至 [HTTP] 動作區段、尋找 [授權] 區段，並納入這一行：

`{"tenant": "{tenant-ID}", "audience": "{client-ID-from-Part-2-web-app-or-API app}", "clientId": "{client-ID-from-Part-1-logic-app}", "secret": "{key-from-Part-1-logic-app}", "type": "ActiveDirectoryOAuth" }`

| 元素 | 必要 | 說明 | 
| ------- | -------- | ----------- | 
| tenant | 是 | Azure AD 租用戶的 GUID | 
| audience | 是 | 您想要存取之目標資源的 GUID - 這是來自您 Web 應用程式或 API 應用程式之應用程式識別碼的用戶端識別碼 | 
| clientId | 是 | 要求存取權之用戶端的 GUID - 這是來自您邏輯應用程式之應用程式識別碼的用戶端識別碼 | 
| secret | 是 | 來自要求存取權杖的用戶端之應用程式識別碼的金鑰或密碼 | 
| type | 是 | 驗證類型。 若為 ActiveDirectoryOAuth 驗證，值為 `ActiveDirectoryOAuth`。 | 
|||| 

例如︰

``` json
{
   "actions": {
      "some-action": {
         "conditions": [],
         "inputs": {
            "method": "post",
            "uri": "https://your-api-azurewebsites.net/api/your-method",
            "authentication": {
               "tenant": "tenant-ID",
               "audience": "client-ID-from-azure-ad-app-for-web-app-or-api-app",
               "clientId": "client-ID-from-azure-ad-app-for-logic-app",
               "secret": "key-from-azure-ad-app-for-logic-app",
               "type": "ActiveDirectoryOAuth"
            }
         },
      }
   }
}
```

<a name="update-code"></a>

### <a name="secure-api-calls-through-code"></a>透過程式碼保護 API 呼叫

<a name="certificate"></a>

#### <a name="certificate-authentication"></a>憑證驗證

若要驗證邏輯應用程式傳入 Web 應用程式或 API 應用程式中的要求，您可以使用用戶端憑證。 如需設定程式碼，請了解[如何設定 TLS 相互驗證](../app-service/app-service-web-configure-tls-mutual-auth.md)。

在 [授權] 區段中，納入這一行： 

`{"type": "clientcertificate", "password": "password", "pfx": "long-pfx-key"}`

| 元素 | 必要 | 說明 | 
| ------- | -------- | ----------- | 
| type | 是 | 驗證類型。 若為 SSL 用戶端憑證，值必須是 `ClientCertificate`。 | 
| password | 是 | 用以存取用戶端憑證的密碼 (PFX 檔案) | 
| pfx | 是 | 用戶端憑證的 Base64 編碼內容 (PFX 檔案) | 
|||| 

<a name="basic"></a>

#### <a name="basic-authentication"></a>基本驗證

若要驗證邏輯應用程式傳入 Web 應用程式或 API 應用程式中的要求，您可以使用基本驗證，例如使用者名稱和密碼。 基本驗證的常見模式，任何用來建置 Web 應用程式或 API 應用程式的語言都可以使用此驗證。

在 [授權] 區段中，納入這一行：

`{"type": "basic", "username": "username", "password": "password"}`。

| 元素 | 必要 | 說明 | 
| ------- | -------- | ----------- | 
| type | 是 | 您想要使用的驗證類型。 若為基本驗證，值必須是 `Basic`。 | 
| username | 是 | 您想要用來進行驗證的使用者名稱 | 
| password | 是 | 您想要用來進行驗證的密碼 | 
|||| 

<a name="azure-ad-code"></a>

#### <a name="azure-active-directory-authentication-through-code"></a>透過程式碼驗證 Azure Active Directory

根據預設，您在 Azure 入口網站中開啟的 Azure AD 驗證不提供精細的授權。 例如，這項驗證會將您的 API 鎖定為僅限特定租用戶，而非特定使用者或應用程式。 

若要限制 API 透過程式碼存取您的邏輯應用程式，請擷取具有 JSON Web 權杖 (JWT) 的標頭。 檢查呼叫者的身分識別，並拒絕不相符的要求。

<!-- Going further, to implement this authentication entirely in your own code, 
and not use the Azure portal, learn how to 
[authenticate with on-premises Active Directory in your Azure app](../app-service/app-service-authentication-overview.md).

To create an application identity for your logic app and use that identity to call your API, 
you must follow the previous steps. -->

## <a name="next-steps"></a>後續步驟

* [從邏輯應用程式工作流程部署和呼叫自訂 API](../logic-apps/logic-apps-custom-api-host-deploy-call.md)