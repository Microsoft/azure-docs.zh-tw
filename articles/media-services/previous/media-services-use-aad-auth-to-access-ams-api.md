---
title: 使用 Azure Active Directory 驗證存取 Azure 媒體服務 API | Microsoft Docs
description: 深入了解使用 Azure Active Directory (Azure AD) 驗證 Azure 媒體服務 API 之存取的概念和所需的步驟。
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
ms.openlocfilehash: d6fe69588c44e489f37ad0908df569c2b5a4b96c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967807"
---
# <a name="access-the-azure-media-services-api-with-azure-ad-authentication"></a>使用 Azure AD 驗證存取 Azure 媒體服務 API
 
Azure 媒體服務 API 是一種 RESTful API。 您可以使用 REST API 或使用可用的用戶端 SDK 來執行媒體資源的作業。 Azure 媒體服務提供適用於 Microsoft .NET 的媒體服務用戶端 SDK。 在獲得存取媒體服務資源和媒體服務 API 的授權之前，您必須先進行驗證。 

媒體服務支援 [Azure Active Directory (Azure AD) 型驗證](../../active-directory/fundamentals/active-directory-whatis.md)。 Azure 媒體 REST 服務規定，發出 REST API 要求的使用者或應用程式需具備**參與者**或**擁有者**角色才能存取資源。 如需詳細資訊，請參閱[在 Azure 入口網站中開始使用角色型存取控制](../../role-based-access-control/overview.md)。  

> [!IMPORTANT]
> 目前，媒體服務支援 Azure 存取控制服務驗證模型。 不過，存取控制授權將在 2018 年 6 月 1 日被取代。 建議您儘速移轉至 Azure AD 驗證模型。

本文件提供如何使用 REST 或 .NET API 來存取媒體服務 API 的概觀。

## <a name="access-control"></a>存取控制

Azure 媒體 REST 要求若要成功，呼叫的使用者必須擁有嘗試存取之媒體服務帳戶的「參與者」或「擁有者」角色。  
只有具備「擁有者」角色的使用者，可以授與媒體資源 (帳戶) 的存取權給新的使用者或應用程式。 「參與者」角色只能存取媒體資源。
未經授權的要求會失敗，狀態碼為 401。 如果您看到此錯誤碼，請確認您的使用者是否具有針對該使用者的媒體服務帳戶指派的「參與者」或「擁有者」角色。 您可以在 Azure 入口網站查看。 搜尋您的媒體帳戶，然後按一下 [Access control] \(存取控制\) 索引標籤。 

![[Access control] \(存取控制\) 索引標籤](./media/media-services-use-aad-auth-to-access-ams-api/media-services-access-control.png)

## <a name="types-of-authentication"></a>驗證類型 
 
當您使用 Azure AD 驗證搭配 Azure 媒體服務 時，會有兩個驗證選項：

- **使用者驗證**。 驗證使用應用程式與媒體服務資源互動的人員。 互動式應用程式應該會先提示使用者輸入使用者的認證。 例如，授權的使用者用來監控編碼工作或即時串流的管理主控台應用程式。 
- **服務主體驗證**。 驗證服務。 通常使用這種驗證方法的應用程式有執行精靈服務、中介層服務或排程的工作的應用程式。 例如，Web 應用程式、函數應用程式、邏輯應用程式、API 或微服務。

### <a name="user-authentication"></a>使用者驗證 

應使用使用者驗證方法的應用程式有管理或監控原生應用程式：行動應用程式、Windows 應用程式和主控台應用程式。 當您想和下列其中一個案例中的服務有人為互動時，這種類型的解決方案非常有用：

- 編碼工作的監控儀表板。
- 即時串流的監控儀表板。
- 讓傳統型或行動使用者管理媒體服務帳戶中之資源的管理應用程式。

> [!NOTE]
> 這種驗證方法不應用於消費者對向應用程式。 

原生應用程式必須先從 Azure AD 取得存取權杖，然後在您向媒體服務 REST API 發出 HTTP 要求時使用它。 將存取權杖加入要求標頭。 

下圖顯示典型的互動式應用程式驗證流程： 

![原生應用程式圖](./media/media-services-use-aad-auth-to-access-ams-api/media-services-native-aad-app1.png)

在上圖中，數字代表依時間順序的要求流量。

> [!NOTE]
> 當您使用使用者驗證方法時，所有應用程式會共用相同的 (預設的) 原生應用程式用戶端識別碼和原生應用程式重新導向 URI。 

1. 提示使用者輸入認證。
2. 使用下列參數要求 Azure AD 存取權杖：  

    * Azure AD 租用戶端點。

        租用戶資訊可從 Azure 入口網站擷取。 將游標放在右上角登入的使用者名稱上方。
    * 媒體服務資源 URI。 

        同一 Azure 環境中的媒體服務帳戶的這個 URI 都相同 (例如， https://rest.media.azure.net)。

    * 媒體服務 (原生) 應用程式用戶端識別碼。
    * 媒體服務 (原生) 應用程式重新導向 URI。
    * REST 媒體服務的資源 URI。
        
        URI 表示 REST API 端點 (例如 https://test03.restv2.westus.media.azure.net/api/)。

    若要取得這些參數的值，請參閱[使用 Azure 入口網站存取 Azure AD 驗證設定](media-services-portal-get-started-with-aad.md) (使用使用者驗證選項)。

3. Azure AD 存取權杖會傳送至用戶端。
4. 用戶端會使用此 Azure AD 存取權杖將要求傳送至 Azure 媒體 REST API。
5. 媒體服務回傳資料給用戶端。

如需如何使用媒體服務 .NET 用戶端 SDK 以利用 Azure AD 驗證和 REST 要求通訊的資訊，請參閱[使用 Azure AD 驗證搭配 .NET 存取媒體服務 API](media-services-dotnet-get-started-with-aad.md)。 

如果您不是使用媒體服務 .NET 用戶端 SDK，您必須使用步驟 2 中描述的參數手動建立 Azure AD 存取權杖要求。 如需詳細資訊，請參閱[如何使用 Azure AD 驗證程式庫取得 Azure AD 權杖](../../active-directory/develop/active-directory-authentication-libraries.md)。

### <a name="service-principal-authentication"></a>服務主體驗證

通常使用這種驗證方法的應用程式有執行中介層服務和排程的工作的應用程式：Web 應用程式、函數應用程式、邏輯應用程式、API 和微服務。 此驗證方法也適用於您可能想使用服務帳戶管理資源的互動式應用程式。

當您使用服務主體驗證方法來建立取用者案例時，驗證通常在中介層 (透過某些 API) 而非直接在行動或傳統型應用程式中處理。 

若要使用此方法，請在其自己的租用戶中建立 Azure AD 應用程式和服務主體。 建立應用程式之後，請將媒體服務帳戶的「參與者」或「擁有者」角色存取權授與應用程式。 您可以在 Azure 入口網站中、使用 Azure CLI 或使用 PowerShell 指令碼執行此動作。 您也可以使用現有的 Azure AD 應用程式。 您可以[在 Azure 入口網站](media-services-portal-get-started-with-aad.md)註冊及管理您的 Azure AD 應用程式和服務主體。 您也可以使用 [Azure CLI](media-services-use-aad-auth-to-access-ams-api.md) 或 [PowerShell](media-services-powershell-create-and-configure-aad-app.md) 執行此動作。 

![中介層應用程式](./media/media-services-use-aad-auth-to-access-ams-api/media-services-principal-service-aad-app1.png)

建立您的 Azure AD 應用程式之後，您會取得下列設定的值。 您需要這些值以進行驗證：

- 用戶端識別碼 
- 用戶端密碼 

在上圖中，數字代表依時間順序的要求流量：
    
1. 中介層應用程式 (Web API 或 Web 應用程式) 會要求具有下列參數的 Azure AD 存取權杖：  

    * Azure AD 租用戶端點。

        租用戶資訊可從 Azure 入口網站擷取。 將游標放在右上角登入的使用者名稱上方。
    * 媒體服務資源 URI。 

        位於同一 Azure 環境中的媒體服務帳戶的這個 URI 都相同 (例如， https://rest.media.azure.net)。

    * REST 媒體服務的資源 URI。

        URI 表示 REST API 端點 (例如 https://test03.restv2.westus.media.azure.net/api/)。

    * Azure AD 應用程式的值：用戶端識別碼和用戶端祕密。
    
    若要取得這些參數的值，請參閱[使用 Azure 入口網站存取 Azure AD 驗證設定](media-services-portal-get-started-with-aad.md) (使用服務主體驗證選項)。

2. Azure AD 存取權杖會傳送至中介層。
4. 中介層使用該 Azure AD 權杖傳送要求至 Azure 媒體 REST API。
5. 媒體服務回傳資料給中介層。

如需如何使用媒體服務 .NET 用戶端 SDK 以利用 Azure AD 驗證和 REST 要求通訊的詳細資訊，請參閱[使用 Azure AD 驗證搭配 .NET 存取 Azure 媒體服務 API](media-services-dotnet-get-started-with-aad.md)。 

如果您不是使用媒體服務 .NET 用戶端 SDK，您必須使用步驟 1 中描述的參數手動建立 Azure AD 權杖要求。 如需詳細資訊，請參閱[如何使用 Azure AD 驗證程式庫取得 Azure AD 權杖](../../active-directory/develop/active-directory-authentication-libraries.md)。

## <a name="troubleshooting"></a>疑難排解

例外狀況：「遠端伺服器傳回錯誤：(401) 未經授權。」

解決方案：媒體服務 REST 要求若要成功，呼叫的使用者必須是嘗試存取之媒體服務帳戶的「參與者」或「擁有者」角色。 如需詳細資訊，請參閱[存取控制](media-services-use-aad-auth-to-access-ams-api.md#access-control)一節。

## <a name="resources"></a>資源

下列文章是 Azure AD 驗證概念的概觀： 

- [Azure AD 的驗證案例](../../active-directory/develop/authentication-scenarios.md)
- [在 Azure AD 新增、更新或移除應用程式](../../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md)
- [使用 PowerShell 設定及 管理角色型存取控制](../../role-based-access-control/role-assignments-powershell.md)

## <a name="next-steps"></a>後續步驟

* 使用 Azure 入口網站[存取 Azure AD 驗證以取用 Azure 媒體服務 API](media-services-portal-get-started-with-aad.md)。
* 使用 Azure AD 驗證[搭配 .NET 存取 Azure 媒體服務 API](media-services-dotnet-get-started-with-aad.md)。

