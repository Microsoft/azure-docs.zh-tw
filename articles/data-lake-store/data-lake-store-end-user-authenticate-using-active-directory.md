---
title: 使用者驗證︰使用 Azure Active Directory 向 Azure Data Lake Storage Gen1 進行驗證 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 向 Azure Data Lake Storage Gen1 完成使用者驗證
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 2f0638b2449bfd582cb68e26d2043b7bc85342b6
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46125947"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-azure-active-directory"></a>使用 Azure Active Directory 向 Azure Data Lake Storage Gen1 進行使用者驗證
> [!div class="op_single_selector"]
> * [使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)
> * [服務對服務驗證](data-lake-store-service-to-service-authenticate-using-active-directory.md)
> 
> 

Azure Data Lake Storage Gen1 使用 Azure Active Directory 進行驗證。 撰寫配合使用 Data Lake Storage Gen1 或 Azure Data Lake Analytics 的應用程式之前，必須決定如何以 Azure Active Directory (Azure AD) 驗證應用程式。 兩個主要選項為︰

* 使用者驗證 (本文)
* 服務對服務驗證 (從上方的下拉式清單選擇這個選項)

兩個選項都要透過 OAuth 2.0 權杖來提供您的應用程式，權杖會附加到每個對 Data Lake Storage Gen1 或 Azure Data Lake Analytics 提出的要求。

本文說明如何建立 **Azure AD 原生應用程式以進行使用者驗證**。 如需服務對服務驗證的 Azure AD 應用程式設定的指示，請參閱[使用 Azure Active Directory 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-authenticate-using-active-directory.md)。

## <a name="prerequisites"></a>必要條件
* Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* 您的訂用帳戶識別碼。 您可以在 Azure 入口網站擷取。 例如，可從 [Data Lake Storage Gen1] 帳戶刀鋒視窗取得。
  
    ![取得訂用帳戶識別碼](./media/data-lake-store-end-user-authenticate-using-active-directory/get-subscription-id.png)

* 您的 Azure AD 網域名稱。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 在以下螢幕擷取畫面中，網域名稱是 **contoso.onmicrosoft.com**，括號內的 GUID 是租用戶識別碼。 
  
    ![取得 AAD 網域](./media/data-lake-store-end-user-authenticate-using-active-directory/get-aad-domain.png)

* 您的 Azure 租用戶識別碼。 如需有關如何擷取租用戶識別碼的指示，請參閱[取得租用戶識別碼](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-tenant-id)。

## <a name="end-user-authentication"></a>使用者驗證
如果您需要讓終端使用者透過 Azure AD 登入您的應用程式，建議使用這個驗證機制。 您的應用程式接著能夠以與登入的終端使用者相同的存取層級，來存取 Azure 資源。 您的終端使用者必須定期提供其認證，您的應用程式才能繼續存取。

讓終端使用者登入的結果，是系統會提供一個存取權杖和一個重新整理權杖給您的應用程式。 存取權杖會附加到每個對 Data Lake Storage Gen1 或 Data Lake Analytics 提出的要求，預設的有效期是一小時。 重新整理權杖可用來取得新的存取權杖，預設的有效期最多為兩週。 您可以使用兩種不同方法讓終端使用者登入。

### <a name="using-the-oauth-20-pop-up"></a>使用 OAuth 2.0 快顯視窗
您的應用程式可以觸發 OAuth 2.0 授權快顯視窗，讓終端使用者輸入其認證。 如有必要，這個快顯視窗也適用於 Azure AD 雙因素驗證 (2FA) 程序。 

> [!NOTE]
> 適用於 Python 或 Java 的 Azure AD 驗證程式庫 (ADAL) 尚未支援這個方法。
> 
> 

### <a name="directly-passing-in-user-credentials"></a>直接傳遞使用者認證
您的應用程式可以直接提供對 Azure AD 的使用者認證。 這個方法僅適用於有組織識別碼的使用者帳戶；不適用於個人/「Live ID」使用者帳戶 (包括以 @outlook.com 或 @live.com 結尾的帳戶)。 此外，這個方法與需要 Azure AD 雙因素驗證 (2FA) 的使用者帳戶不相容。

### <a name="what-do-i-need-for-this-approach"></a>這個方法需要什麼？
* Azure AD 網域名稱。 此需求已列在本文的先決條件中。
* Azure AD 租用戶識別碼。 此需求已列在本文的先決條件中。
* Azure AD **原生應用程式**
* Azure AD 原生應用程式的應用程式識別碼
* Azure AD 原生應用程式的重新導向 URI
* 設定委派權限


## <a name="step-1-create-an-active-directory-native-application"></a>步驟 1：建立 Active Directory 原生應用程式

建立和設定 Azure AD 原生應用程式，以便使用 Azure Active Directory 向 Data Lake Storage Gen1 進行使用者驗證。 如需指示，請參閱[建立 Azure AD 應用程式](../azure-resource-manager/resource-group-create-service-principal-portal.md)。

遵循連結中的指示進行時，請確定如以下螢幕擷取畫面所示，選取 [原生] 應用程式類型：

![建立 Web 應用程式](./media/data-lake-store-end-user-authenticate-using-active-directory/azure-active-directory-create-native-app.png "建立原生應用程式")

## <a name="step-2-get-application-id-and-redirect-uri"></a>步驟 2：取得應用程式識別碼和重新導向 URI

若要擷取應用程式識別碼，請參閱[取得應用程式識別碼](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-application-id-and-authentication-key)。

若要擷取重新導向 URI，請進行下列步驟。

1. 在 Azure 入口網站中，選取 [Azure Active Directory]，按一下 [應用程式註冊]，然後尋找並按一下您已建立的 Azure AD 原生應用程式。

2. 在應用程式的 [設定] 刀鋒視窗中，按一下 [重新導向 URI]。

    ![取得重新導向 URI](./media/data-lake-store-end-user-authenticate-using-active-directory/azure-active-directory-redirect-uri.png)

3. 複製顯示的值。


## <a name="step-3-set-permissions"></a>步驟 3：設定權限

1. 在 Azure 入口網站中，選取 [Azure Active Directory]，按一下 [應用程式註冊]，然後尋找並按一下您已建立的 Azure AD 原生應用程式。

2. 在應用程式的 [設定] 刀鋒視窗中，按一下 [必要的權限]，然後按一下 [新增]。

    ![用戶端識別碼](./media/data-lake-store-end-user-authenticate-using-active-directory/aad-end-user-auth-set-permission-1.png)

3. 在 [加入 API 存取權] 刀鋒視窗中，依序按一下 [選取 API]、[Azure Data Lake]，然後按一下 [選取]。

    ![用戶端識別碼](./media/data-lake-store-end-user-authenticate-using-active-directory/aad-end-user-auth-set-permission-2.png)
 
4.  在 [加入 API 存取權] 刀鋒視窗中，按一下 [選取權限]，選取核取方塊以提供 **Data Lake Store 完整的存取權**，然後按一下 [選取]。

    ![用戶端識別碼](./media/data-lake-store-end-user-authenticate-using-active-directory/aad-end-user-auth-set-permission-3.png)

    按一下 [完成] 。

5. 重複最後兩個步驟，以便將權限也授與 **Windows Azure 服務管理 API**。
   
## <a name="next-steps"></a>後續步驟
在本文中，您已建立 Azure AD 原生應用程式，並收集您使用 .NET SDK、Java SDK、REST API 等撰寫之用戶端應用程式中所需的資訊。您現在可以繼續進行下列文章，這些文章說明如何使用 Azure AD Web 應用程式先向 Data Lake Storage Gen1 進行驗證，然後再於儲存區上執行其他作業。

* [使用 Java SDK 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-java-sdk.md)
* [使用 .NET SDK 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-net-sdk.md)
* [使用 Python 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-python.md)
* [使用 REST API 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-rest-api.md)

