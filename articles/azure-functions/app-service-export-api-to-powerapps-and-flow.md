---
title: 將 Azure 裝載 API 匯出至 PowerApps 和 Microsoft Flow | Microsoft Docs
description: 如何向 PowerApps 和 Microsoft Flow 公開 App Service 中裝載之 API 的概觀
services: app-service
documentationcenter: ''
author: ggailey777
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 12/15/2017
ms.author: glenga
ms.reviewer: sunayv
ms.openlocfilehash: ef3fe5002a28c66478a10909a7e9556449cd9712
ms.sourcegitcommit: 4e36ef0edff463c1edc51bce7832e75760248f82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35234762"
---
# <a name="exporting-an-azure-hosted-api-to-powerapps-and-microsoft-flow"></a>將 Azure 裝載 API 匯出至 PowerApps 和 Microsoft Flow

[PowerApps](https://powerapps.microsoft.com/guided-learning/learning-introducing-powerapps/) 是一種服務，可用來建置和使用自訂商務 App，以便跨越平台連接到您的資料和工作。 [Microsoft Flow](https://flow.microsoft.com/guided-learning/learning-introducing-flow/) 可讓您輕易將最愛的應用程式與服務之間的工作流程和商業程序自動化。 PowerApps 和 Microsoft Flow 隨附資料來源 (如 Office 365、Dynamics 365、Salesforce 等等) 的各種內建連接器。 在某些情況下，應用程式和流程建置者也想要連接他們組織建置的資料來源和 API。

同樣地，想要在組織內廣泛公開 API 的開發人員，可以讓應用程式和流程建置者使用他們的 API。 本主題說明如何匯出以 [Azure Functions](../azure-functions/functions-overview.md) 或 [Azure App Service](../app-service/app-service-web-overview.md)API。 匯出的 API 能成為「自訂連接器」，它在 PowerApps and Microsoft Flow just 中的使用方法就像內建連接器一樣。

## <a name="create-and-export-an-api-definition"></a>建立及匯出 API 定義
在匯出 API 之前，您必須使用 OpenAPI 定義 (前身為 [Swagger](http://swagger.io/) 檔案) 描述 API。 此定義包含有關 API 中可以使用哪些作業，以及應該如何結構化 API 之要求和回應資料的資訊。 PowerApps 和 Microsoft Flow 可以為任何 OpenAPI 2.0 定義建立自訂連接器。 Azure Functions 和 Azure App Service 有內建支援，可供建立、裝載及管理 OpenAPI 定義。 如需詳細資訊，請參閱[在 Azure App Service 中裝載具有 CORS 支援的 RESTful API](../app-service/app-service-web-tutorial-rest-api.md)。

> [!NOTE]
> 您也可以不使用 OpenAPI 定義在 PowerApps 和 Microsoft Flow 中建置自訂連接器。 如需詳細資訊，請參閱[註冊及使用自訂連接器 (PowerApps)](https://powerapps.microsoft.com/tutorials/register-custom-api/) 和[註冊及使用自訂連接器 (Microsoft Flow)](https://flow.microsoft.com/documentation/register-custom-api/)。

若要匯出 API 定義，請遵循以下步驟：

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 Azure Functions 或另一個 App Service 應用程式。

    如果使用 Azure Functions，請選取您的函式應用程式，然後依序選擇 [平台功能]、[API 定義]。

    ![Azure Functions API 定義](media/app-service-export-api-to-powerapps-and-flow/api-definition-function.png)

    如果使用 Azure App Service，請從設定清單選取 [API 定義]。

    ![App Service API 定義](media/app-service-export-api-to-powerapps-and-flow/api-definition-app.png)

2. [匯出至 PowerApps + Microsoft Flow] 按鈕應可使用 (如果無法使用，您必須先建立 OpenAPI 定義)。 按一下此按鈕，開始匯出程序。

    ![[匯出至 PowerApps 和 Microsoft Flow] 按鈕](media/app-service-export-api-to-powerapps-and-flow/export-apps-flow.png)

3. 選取 [匯出模式]：

    [快速] 可讓您從 Azure 入口網站建立自訂連接器。 您必須登入 PowerApps 或 Microsoft Flow，而且擁有在目標環境中建立連接器的權限。 如果可符合這兩項需求，則這是建議的方法。 如果使用此模式，請依照下面的[使用快速匯出](#express)指示執行。

    [手動] 可讓您匯出 API 定義，稍後再使用 PowerApps 或 Microsoft Flow 入口網站匯入。 如果 Azure 使用者和具有建立連接器權限的使用者是不同的人，或連接器必須在另一個 Azure 租用戶中建立，則這是建議的方法。 如果使用此模式，請依照下面的[使用手動匯出](#manual)指示執行。

    ![匯出模式](media/app-service-export-api-to-powerapps-and-flow/export-mode.png)

> [!NOTE]
> 自訂連接器使用 API 定義的「副本」，因此 PowerApps 和 Microsoft Flow 無法立即得知您是否變更應用程式與其 API 定義。 如果您確實變更，請針對新版本重複匯出步驟。

<a name="express"></a>
## <a name="use-express-export"></a>使用快速匯出

若要以**快速**模式完成匯出，請依照下列步驟執行：

1. 確認您已登入要匯出的目標 PowerApps 或 Microsoft Flow 租用戶。 

2. 使用表格中所指定的設定。

    |設定|說明|
    |--------|------------|
    |**環境**|選取應該儲存自訂連接器的目標環境。 如需詳細資訊，請參閱[環境概觀](https://powerapps.microsoft.com/tutorials/environments-overview/)。|
    |**自訂 API 名稱**|輸入名稱，也就是 PowerApps 和 Microsoft Flow 建置者在連接器清單中看到的名稱。|
    |**準備安全性設定**|必要時，請提供授與使用者存取 API 權限所需的安全性設定詳細資料。 本範例示範 API 金鑰。 如需詳細資訊，請參閱下面的[指定驗證類型](#auth)。|
 
    ![快速匯出至 PowerApps 和 Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-express.png)

3. 按一下 [確定]。 自訂連接現已建置並新增至您指定的環境。

如需使用「快速」模式搭配 Azure Functions 的範例，請參閱[從 PowerApps 呼叫函式](functions-powerapps-scenario.md)和[從 Microsoft Flow 呼叫函式](functions-flow-scenario.md)。

<a name="manual"></a>
## <a name="use-manual-export"></a>使用手動匯出

若要以**手動**模式完成匯出，請依照下列步驟執行：

1. 按一下 [下載] 並儲存檔案，或按一下複製按鈕儲存 URL。 您將在匯入期間使用下載檔案或 URL。
 
    ![手動匯出至 PowerApps 和 Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-manual.png)
 
2. 如果您的 API 定義包含任何安全性定義，則會在步驟 2 中叫出這些定義。 在匯入期間，PowerApps 和 Microsoft Flow 會偵測這些定義並提示輸入安全性資訊。 收集與每個定義相關的認證，以便使用於下一節中。 如需詳細資訊，請參閱下面的[指定驗證類型](#auth)。

    ![手動匯出的安全性](media/app-service-export-api-to-powerapps-and-flow/export-manual-security.png)

    本範例展示 OpenAPI 定義中包含的 API 金鑰安全性定義。

既然您已經匯出 API 定義，可以在 PowerApps 和 Microsoft Flow 中匯入以建立自訂連接器。 自訂連接器會由這兩項服務共用，因此您只需要匯入定義一次。

若要將 API 定義匯入 PowerApps 和 Microsoft Flow 中，請依照下列步驟執行：

1. 移至 [powerapps.com](https://web.powerapps.com) 或 [flow.microsoft.com](https://flow.microsoft.com)。

2. 在右上角，按一下齒輪圖示，然後按一下 [Custom connectors] \(自訂連接器\)。

   ![服務中的齒輪圖示](media/app-service-export-api-to-powerapps-and-flow/icon-gear.png)

3. 按一下 [Create custom connector] \(建立自訂連接器\)，然後按一下 [Import an OpenAPI definition] \(匯入 OpenAPI 定義\)。

   ![建立自訂連接器](media/app-service-export-api-to-powerapps-and-flow/flow-apps-create-connector.png)

4. 輸入自訂連接器的名稱，然後瀏覽至您所匯出的 OpenAPI 定義，並按一下 [Continue] \(繼續\)。

   ![上傳 OpenAPI 定義](media/app-service-export-api-to-powerapps-and-flow/flow-apps-upload-definition.png)

4. 在 [General] \(一般\) 索引標籤上，檢閱來自 OpenAPI 定義的資訊。

5. 在 [安全性] 索引標籤上，如果系統提示您提供驗證詳細資訊，請輸入適當的驗證類型值。 按一下 [繼續]。

    ![[安全性] 索引標籤](media/app-service-export-api-to-powerapps-and-flow/tab-security.png)

    本範例展示 API 金鑰驗證的必要欄位。 驗證類型不同，欄位也會不同。

6. 在 [定義] 索引標籤上，會自動填入在 OpenAPI 檔案定義的所有作業。 如果所有的必要作業皆已定義，您便可以前往下個步驟。 如果沒有，您可以在此新增與修改作業。

    ![[定義] 索引標籤](media/app-service-export-api-to-powerapps-and-flow/tab-definitions.png)

    本範例有一項名為 `CalculateCosts`。 諸如 [說明] 等中繼資料全部來自 OpenAPI 檔案。

7. 按一下頁面頂端的 [建立連接器]。

您現在可以在 PowerApps 和 Microsoft Flow 中連接自訂連接器了。 如需在 PowerApps 和 Microsoft Flow 入口網站中建立連接器的詳細資訊，請參閱[註冊自訂連接器 (PowerApps)](https://powerapps.microsoft.com/tutorials/register-custom-api/#register-your-custom-connector) 和[註冊自訂連接器 (Microsoft Flow)](https://flow.microsoft.com/documentation/register-custom-api/#register-your-custom-connector)。

<a name="auth"></a>
## <a name="specify-authentication-type"></a>指定驗證類型

PowerApps 和 Microsoft Flow 支援一系列提供自訂連接器驗證的識別提供者。 如果您的 API 需要驗證，請確保它會依照您 OpenAPI 文件中的「安全性定義」擷取，如以下範例所示：

```json
"securityDefinitions": {
    "AAD": {
    "type": "oauth2",
    "flow": "accessCode",
    "authorizationUrl": "https://login.windows.net/common/oauth2/authorize",
    "scopes": {}
    }
}
``` 
在匯出期間，您需要提供設定值，允許 PowerApps 和 Microsoft Flow 驗證使用者。

本節涵蓋的驗證類型是**快速**所支援的類型：API 金鑰、Azure Active Directory 和 Generic OAuth 2.0。 對於 Dropbox、Facebook 及 SalesForce 等特定服務，PowerApps 和 Microsoft Flow 亦支援基本驗證和 OAuth 2.0。

### <a name="api-key"></a>API 金鑰
在使用 API 金鑰時，當連接器的使用者建立連線時，系統會提示他們提供金鑰。 您可以指定 API 金鑰名稱，協助他們了解所需的金鑰為何。 在先前的範例中，我們使用 `API Key (contact meganb@contoso.com)` 的名稱，讓使用者知道該去哪裡取得 API 金鑰的相關資訊。 針對 Azure Functions，金鑰通常是涵蓋函式應用程式內幾種功能的其中一個主機金鑰。

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
在使用 Azure AD 時，需要兩個 Azure AD 應用程式註冊：一個用於 API 本身，還有一個用於自訂連接器：

- 若要設定 API 的註冊，請使用 [App Service 驗證/授權](../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md)功能。

- 若要設定連接器的註冊，請依照[新增 Azure AD 應用程式](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications#adding-an-application)中的步驟執行。 註冊必須具有 API 和 `https://msmanaged-na.consent.azure-apim.net/redirect` 之回覆 URL 的委派存取權。 

如需詳細資訊，請參閱 [PowerApps](https://powerapps.microsoft.com/tutorials/customapi-azure-resource-manager-tutorial/) 和 [Microsoft Flow](https://flow.microsoft.com/documentation/customapi-azure-resource-manager-tutorial/) 中的 Azure AD 註冊範例。 這些範例將 Azure Resource Manager 當作 API。如果您遵循這些步驟，請替換 API。

下列設定值是必要的：
- **用戶端識別碼**：連接器 Azure AD 註冊的用戶端識別碼
- **用戶端祕密**：連接器 Azure AD 註冊的用戶端祕密
- **登入 URL**：Azure AD 的基底 URL。 在 Azure 中通常是 `https://login.windows.net`。
- **租用戶識別碼**：要用於登入的租用戶識別碼。 這應該是 "common"，或建立連接器所在之租用戶的識別碼。
- **資源 URL**：API 的 Azure AD 註冊資源 URL

> [!IMPORTANT]
> 如果其他人會將 API 定義匯入到 PowerApps 和 Microsoft Flow 作為手動流程的一部分，您必須提供他們「連接器註冊」的用戶端識別碼和用戶端密碼，以及 API 的資源 URL。 確定這些密碼都已受到安全地管理。 **請勿共用 API 本身的安全性認證。**

### <a name="generic-oauth-20"></a>Generic OAuth 2.0
在使用 Generic OAuth 2.0 時，您可以整合任何 OAuth 2.0 提供者。 這項功能可讓您使用非原生支援的自訂提供者。

下列設定值是必要的：
- **用戶端識別碼**：OAuth 2.0 用戶端識別碼
- **用戶端祕密**：OAuth 2.0 用戶端祕密
- **授權 URL**：OAuth 2.0 授權 URL
- **權杖 URL**：OAuth 2.0 權杖 URL
- **重新整理 URL**：OAuth 2.0 重新整理 URL


