---
title: 服務對服務驗證︰使用 Azure Active Directory 以 Data Lake Store 進行 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 以 Data Lake Store 完成服務對服務驗證
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
ms.openlocfilehash: 1e59ed093417d8761135b946e2fa3f183bb085c9
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39215966"
---
# <a name="service-to-service-authentication-with-data-lake-store-using-azure-active-directory"></a>使用 Azure Active Directory 以 Data Lake Store 進行服務對服務驗證
> [!div class="op_single_selector"]
> * [使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)
> * [服務對服務驗證](data-lake-store-service-to-service-authenticate-using-active-directory.md)
> 
>  

Azure Data Lake Store 使用 Azure Active Directory 進行驗證。 撰寫搭配 Azure Data Lake Store 的應用程式之前，必須決定要如何向 Azure Active Directory (Azure AD) 驗證應用程式。 兩個主要選項為︰

* 使用者驗證 
* 服務對服務驗證 (本文) 

兩個選項都要靠 OAuth 2.0 權杖來提供您的應用程式，權杖會連接到每個對 Azure Data Lake Store 提出的要求。

本文說明如何建立 **Azure AD Web 應用程式，以進行服務對服務驗證**。 如需有關適用於終端使用者驗證的 Azure AD 應用程式設定的指示，請參閱[使用 Azure Active Directory 以 Data Lake Store 進行終端使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)。

## <a name="prerequisites"></a>必要條件
* Azure 訂用帳戶。 請參閱 [取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="step-1-create-an-active-directory-web-application"></a>步驟 1：建立 Active Directory Web 應用程式

建立和設定 Azure AD Web 應用程式，以便使用 Azure Active Directory 以 Azure Data Lake Store 進行服務對服務驗證。 如需指示，請參閱[建立 Azure AD 應用程式](../azure-resource-manager/resource-group-create-service-principal-portal.md)。

依照上面連結中的指示進行時，請確定如以下螢幕擷取畫面所示，選取 [Web 應用程式 / API] 應用程式類型：

![建立 Web 應用程式](./media/data-lake-store-authenticate-using-active-directory/azure-active-directory-create-web-app.png "建立 Web 應用程式")

## <a name="step-2-get-application-id-authentication-key-and-tenant-id"></a>步驟 2：取得應用程式識別碼、驗證金鑰及租用戶識別碼
以程式設計方式登入時，您需要應用程式的識別碼。 如果應用程式是在自己的認證下執行，則您還需要一個驗證金鑰。

* 如需有關如何為應用程式擷取應用程式識別碼和驗證金鑰 (也稱為用戶端祕密) 的指示，請參閱[取得應用程式識別碼和驗證金鑰](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-application-id-and-authentication-key)。

* 如需有關如何擷取租用戶識別碼的指示，請參閱[取得租用戶識別碼](../azure-resource-manager/resource-group-create-service-principal-portal.md#get-tenant-id)。

## <a name="step-3-assign-the-azure-ad-application-to-the-azure-data-lake-store-account-file-or-folder"></a>步驟 3：將 Azure AD 應用程式指派給 Azure Data Lake Store 帳戶檔案或資料夾


1. 登入 [Azure 入口網站](https://portal.azure.com)。 開啟要與您稍早建立的 Azure Active Directory 應用程式建立關聯的 Azure Data Lake Store 帳戶。
2. 在您的 [Data Lake Store 帳戶] 刀鋒視窗中，按一下 [資料總管] 。
   
    ![在 Data Lake Store 帳戶中建立目錄](./media/data-lake-store-authenticate-using-active-directory/adl.start.data.explorer.png "在 Data Lake Store 帳戶中建立目錄")
3. 在 [資料總管] 刀鋒視窗中，按一下您要將其存取權提供給 Azure AD 應用程式的檔案或資料夾，然後按一下 [存取]。 若要設定對檔案的存取權，您必須從 [檔案預覽] 刀鋒視窗按一下 [存取]。
   
    ![設定 Data Lake 檔案系統上的 ACL](./media/data-lake-store-authenticate-using-active-directory/adl.acl.1.png "設定 Data Lake 檔案系統上的 ACL")
4. [存取]  刀鋒視窗會列出已指派至根的標準存取和自訂存取。 按一下 [新增]  圖示以新增自訂層級的 ACL。
   
    ![列出標準和自訂存取](./media/data-lake-store-authenticate-using-active-directory/adl.acl.2.png "列出標準和自訂的存取")
5. 按一下 [新增] 圖示，以開啟 [新增自訂存取] 刀鋒視窗。 在此刀鋒視窗中，按一下 [選取使用者或群組]，然後在 [選取使用者或群組] 刀鋒視窗中，尋找您稍早建立的 Azure Active Directory 應用程式。 若您需要搜尋大量的群組，請使用頂端的文字方塊來篩選群組名稱。 按一下您要新增的群組，然後按一下 [選取] 。
   
    ![加入群組](./media/data-lake-store-authenticate-using-active-directory/adl.acl.3.png "加入群組")
6. 按一下 [選取權限]，選取權限及權限的指派方式 (例如預設 ACL、存取 ACL 或兩者並用)。 按一下 [確定]。
   
    ![將權限指派至群組](./media/data-lake-store-authenticate-using-active-directory/adl.acl.4.png "將權限指派至群組")
   
    如需有關 Data Lake Store 中的權限及預設/存取 ACL 的詳細資訊，請參閱 [Data Lake Store 中的存取控制](data-lake-store-access-control.md)。
7. 在 [新增自訂存取] 刀鋒視窗中，按一下 [確定]。 具有相關權限的新增群組會列在 [存取] 刀鋒視窗中。
   
    ![將權限指派至群組](./media/data-lake-store-authenticate-using-active-directory/adl.acl.5.png "將權限指派至群組")

> [!NOTE]
> 如果您打算將 Azure Active Directory 應用程式限制在特定資料夾中，則還需要提供根與 Azure Active Directory 應用程式相同的**執行**權限，才能透過 .NET SDK 進行檔案建立存取。

> [!NOTE]
> 如果您需要使用 SDK 來建立 Data Lake Store 帳戶，必須將 Azure AD web 應用程式作為角色指派給您在其中建立 Data Lake Store 帳戶的資源群組。
> 
>

## <a name="step-4-get-the-oauth-20-token-endpoint-only-for-java-based-applications"></a>步驟 4：取得 OAuth 2.0 權杖端點 (只適用於 Java 型應用程式)

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後按一下左窗格中的 [Active Directory]。

2. 從左窗格，按一下 [應用程式註冊]。

3. 從 [應用程式註冊] 刀鋒視窗頂端，按一下 [端點]。

    ![OAuth 權杖端點](./media/data-lake-store-authenticate-using-active-directory/oauth-token-endpoint.png "OAuth 權杖端點")

4. 從端點清單，複製 OAuth 2.0 權杖端點。

    ![OAuth 權杖端點](./media/data-lake-store-authenticate-using-active-directory/oauth-token-endpoint-1.png "OAuth 權杖端點")   

## <a name="next-steps"></a>後續步驟
在本文中，您已建立 Azure AD Web 應用程式，並收集您使用 .NET SDK、Java、Python、REST API 等撰寫的用戶端應用程式中所需的資訊。您現在可以繼續進行下列文章，這些文章說明如何使用 Azure AD 原生應用程式先以 Data Lake Store 進行驗證，然後再於存放區上執行其他作業。

* [使用 Java 向 Data Lake Store 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-java.md)
* [使用 .NET SDK 向 Data Lake Store 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-net-sdk.md)
* [使用 Python 向 Data Lake Store 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-python.md)
* [使用 REST API 向 Data Lake Store 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-rest-api.md)


