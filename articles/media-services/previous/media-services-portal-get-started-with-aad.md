---
title: 利用 Azure 入口網站開始使用 Azure AD 驗證 | Microsoft Docs
description: 了解如何使用 Azure 入口網站存取 Azure Active Directory (Azure AD) 驗證，以使用 Azure 媒體服務 API。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/17/2017
ms.author: juliako
ms.openlocfilehash: 6267dd8dca4c932d4a4d96b34a8eaa26f6a59c20
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785797"
---
# <a name="get-started-with-azure-ad-authentication-by-using-the-azure-portal"></a>利用 Azure 入口網站開始使用 Azure AD 驗證

了解如何使用 Azure 入口網站存取 Azure Active Directory (Azure AD) 驗證，以存取 Azure 媒體服務 API。

## <a name="prerequisites"></a>先決條件

- 一個 Azure 帳戶。 如果您沒有帳戶，請先從 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)開始。 
- 媒體服務帳戶。 如需詳細資訊，請參閱[使用 Azure 入口網站建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)。
- 請先複習[使用 Azure AD 驗證存取 Azure 媒體服務 API 概觀](media-services-use-aad-auth-to-access-ams-api.md)。 

當您使用 Azure AD 驗證搭配 Azure 媒體服務 時，會有兩個驗證選項：

- **使用者驗證**。 驗證使用應用程式與媒體服務資源互動的人員。 互動式應用程式應該會先提示使用者輸入認證。 例如，授權的使用者用來監控編碼工作或即時串流的管理主控台應用程式。 
- **服務主體驗證**。 驗證服務。 通常使用這種驗證方法的應用程式有執行精靈服務、中介層服務或排程的工作的應用程式：Web 應用程式、函數應用程式、邏輯應用程式、API 或微服務。

> [!IMPORTANT]
> 目前，媒體服務支援 Azure 存取控制服務驗證模型。 不過，存取控制授權將在 2018 年 6 月 1 日被取代。 建議您儘速移轉至 Azure AD 驗證模型。

## <a name="select-the-authentication-method"></a>選取驗證方法

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的媒體服務帳戶。
2. 選取如何連線到媒體服務 API。

    ![選取連線方式的頁面](./media/media-services-portal-get-started-with-aad/media-services-portal-get-started01.png)

## <a name="user-authentication"></a>使用者驗證

若要利用使用者驗證選項連線到媒體服務 API，用戶端應用程式必須要求具有下列參數的 Azure AD 權杖：  

* Azure AD 租用戶端點
* 媒體服務資源 URI
* 媒體服務 (原生) 應用程式用戶端識別碼 
* 媒體服務 (原生) 應用程式重新導向 URI 
* REST 媒體服務的資源 URI

您可以在**使用使用者驗證的媒體服務 API**取得這些參數的值。 

![使用使用者驗證頁面連線](./media/media-services-portal-get-started-with-aad/media-services-portal-get-started02.png)

如果您使用媒體服務的 Microsoft .NET SDK 連線到媒體服務 API，此 SDK 會提供您所需的值。 如需詳細資訊，請參閱[使用 Azure AD 驗證搭配 .NET 存取 Azure 媒體服務 API](media-services-dotnet-get-started-with-aad.md)。

如果您不是使用媒體服務 .NET 用戶端 SDK，您必須使用先前討論的參數手動建立 Azure AD 權杖要求。 如需詳細資訊，請參閱[如何使用 Azure AD 驗證程式庫取得 Azure AD 權杖](../../active-directory/develop/active-directory-authentication-libraries.md)。

## <a name="service-principal-authentication"></a>服務主體驗證

若要利用服務主體選項連線到媒體服務 API，您的中介層應用程式 (Web API 或 Web 應用程式) 必須要求具有下列參數的 Azure AD 權杖：  

* Azure AD 租用戶端點
* 媒體服務資源 URI 
* REST 媒體服務的資源 URI
* Azure AD 應用程式的值：**用戶端識別碼**和**用戶端祕密**

您可以在**使用服務主體連線到媒體服務 API** 頁面取得這些參數的值。 使用此頁面可建立新的 Azure AD 應用程式或選取現有的 Azure AD 應用程式。 選取 Azure AD 應用程式之後，您可以取得用戶端識別碼 (應用程式識別碼)，並產生用戶端祕密 (金鑰) 值。 

![使用服務主體連線的頁面](./media/media-services-portal-get-started-with-aad/media-services-portal-get-started04.png)

當服務主體的刀鋒視窗開啟時，會選取符合下列準則的第一個 Azure AD 應用程式：

- 它是已註冊的 Azure AD 應用程式。
- 它對帳戶具有參與者或擁有者角色型存取控制權限。

建立或選取 Azure AD 應用程式之後，您可以建立並複製用戶端祕密 (金鑰) 和用戶端識別碼 (應用程式識別碼)。 在此案例中，需要有用戶端祕密和用戶端識別碼才能取得存取權杖。

如果您沒有在網域中建立 Azure AD 應用程式的權限，則不會顯示此刀鋒視窗的 Azure AD 應用程式控制項，並會顯示警告訊息。

如果您使用媒體服務 .NET SDK 連線到媒體服務 API，請參閱[使用 Azure AD 驗證搭配 .NET 存取 Azure 媒體服務 API](media-services-dotnet-get-started-with-aad.md)。

如果您不是使用媒體服務 .NET 用戶端 SDK，您必須使用先前討論的參數手動建立 Azure AD 權杖要求。 如需詳細資訊，請參閱[如何使用 Azure AD 驗證程式庫取得 Azure AD 權杖](../../active-directory/develop/active-directory-authentication-libraries.md)。

### <a name="get-the-client-id-and-client-secret"></a>取得用戶端識別碼和用戶端祕密

選取現有的 Azure AD 應用程式或選取建立新的 Azure AD 應用程式的選項後，會顯示下列按鈕：

![[Manage permissions] \(管理權限\) 和 [Manage application] \(管理應用程式\) 按鈕](./media/media-services-portal-get-started-with-aad/media-services-portal-manage.png)

若要開啟 Azure AD 應用程式刀鋒視窗，請按一下 [Manage application] \(管理應用程式\)。 在 [Manage application] \(管理應用程式\) 刀鋒視窗中，您可以取得應用程式的用戶端識別碼 (應用程式識別碼)。 若要產生用戶端祕密 (金鑰)，請選取 [金鑰]。

![管理應用程式刀鋒視窗的 [Keys] \(金鑰\) 選項](./media/media-services-portal-get-started-with-aad/media-services-portal-get-started06.png) 

### <a name="manage-permissions-and-the-application"></a>管理權限和應用程式

選取 Azure AD 應用程式之後，您可以管理應用程式和權限。 若要設定您的 Azure AD 應用程式以存取其他應用程式，請按一下 [Manage permissions] \(管理權限\)。 對於管理工作，例如變更金鑰和回覆 URL，或編輯應用程式的資訊清單，請按一下 [Manage application] \(管理應用程式\)。

### <a name="edit-the-apps-settings-or-manifest"></a>編輯應用程式的設定或資訊清單

若要編輯應用程式的設定或資訊清單，請按一下 [Manage application] \(管理應用程式\)。

![管理應用程式頁面](./media/media-services-portal-get-started-with-aad/media-services-portal-get-started05.png)

## <a name="next-steps"></a>後續步驟

開始使用[上傳檔案至您的帳戶](media-services-portal-upload-files.md)。
