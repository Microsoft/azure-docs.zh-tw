---
title: Azure AD 應用程式 Proxy 中的自訂網域 | Microsoft Docs
description: 管理 Azure AD 應用程式 Proxy 中的自訂網域，讓不論使用者從何處存取都能看到相同的應用程式 URL。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/31/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 090df19861e00a99f0bb63bf20eb2f26dc6761fd
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39368476"
---
# <a name="working-with-custom-domains-in-azure-ad-application-proxy"></a>使用 Azure AD 應用程式 Proxy 中的自訂網域

當您透過 Azure Active Directory 應用程式 Proxy 發佈應用程式時，您會建立可供使用者在遠端工作時移至的外部 URL。 此 URL 會取得預設網域 *yourtenant.msappproxy.net*。 例如，如果您發佈一個名為 Expenses 的應用程式，且您的租用戶名為 Contoso，則外部 URL 會是 https://expenses-contoso.msappproxy.net。 如果您想要使用自己的網域名稱，請為您的應用程式設定自訂網域。 

建議您盡可能為應用程式設定自訂網域。 自訂網域的一些優點包括：

- 使用者可以透過相同的 URL 移至應用程式，不論他們是在網路內部或外部作業。
- 如果所有應用程式有相同的內部和外部 URL，則某一個應用程式中指向另一個應用程式的連結會繼續在公司網路外部運作。 
- 您可控制您的品牌，並建立您想要的 URL。 


## <a name="configure-a-custom-domain"></a>設定自訂網域

### <a name="prerequisites"></a>必要條件

設定自訂網域之前，請確定您已備妥下列需求： 
- [新增至 Azure Active Directory 的已驗證網域](../fundamentals/add-custom-domain.md)。
- 網域的自訂憑證 (採用 PFX 檔格式)。 
- [透過應用程式 Proxy 發佈](application-proxy-publish-azure-portal.md)的內部部署應用程式。

### <a name="configure-your-custom-domain"></a>設定自訂網域

當您備妥這三項需求時，請遵循下列步驟來設定自訂網域：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式]，然後選擇您要管理的應用程式。
3. 選取 [應用程式 Proxy]。 
4. 在 [外部 URL] 欄位中，使用下拉式清單來選取您的自訂網域。 如果您未在清單中看到您的網域，則它尚未經過驗證。 
5. 選取 [儲存]。
5. 已停用的 [憑證] 欄位會變成已啟用。 選取此欄位。 

   ![按一下以上傳憑證](./media/application-proxy-configure-custom-domain/certificate.png)

   如果您已上傳此網域的憑證，則 [憑證] 欄位會顯示憑證資訊。 

6. 上傳 PFX 憑證，然後輸入憑證的密碼。 
7. 選取 [儲存] 來儲存變更。 
8. 新增 [DNS 記錄](../../dns/dns-operations-recordsets-portal.md)，此記錄會將新的外部 URL 重新導向至 msappproxy.net 網域。 

>[!TIP] 
>您只需要針對每個自訂網域上傳一個憑證。 一旦上傳憑證，您即可在發佈新應用程式時選擇自訂網域，而不需要進行額外的設定 (DNS 記錄除外)。 

## <a name="manage-certificates"></a>管理憑證

### <a name="certificate-format"></a>憑證格式
憑證簽章方法沒有任何限制。 橢圓曲線密碼編譯 (ECC)、主體別名 (SAN) 和其他常見的憑證類型均可支援。 

只要萬用字元符合所需的外部 URL，便可使用萬用字元憑證。 

### <a name="changing-the-domain"></a>變更網域
所有已驗證的網域會出現在您應用程式的外部 URL 下拉式清單中。 若要變更的網域，只要更新應用程式的該欄位。 如果您想要的網域不在清單中，請[將它新增為已驗證的網域](../fundamentals/add-custom-domain.md)。 如果您選取的網域還沒有相關聯的憑證，請遵循步驟 5-7 來新增憑證。 接著，務必更新 DNS 記錄以從新的外部 URL 重新導向。 

### <a name="certificate-management"></a>憑證管理
除非應用程式共用外部主機，否則您可以將相同的憑證使用於多個應用程式。 

您會在憑證過期時收到警告，告知您透過入口網站上傳另一個憑證。 如果已撤銷憑證，使用者在存取應用程式時可能會看到安全性警告。 我們不會執行憑證的撤銷檢查。  若要更新指定應用程式的憑證，請巡覽至該應用程式並遵循步驟 5-7 以在已發佈的應用程式上設定自訂網域，進而上傳新的憑證。 如果其他應用程式並未使用舊憑證，則系統會自動將它刪除。 

目前所有憑證管理都是透過個別的應用程式頁面進行，所以您必須在相關應用程式的環境中管理憑證。 

## <a name="next-steps"></a>後續步驟
* [啟用單一登入](application-proxy-configure-single-sign-on-with-kcd.md)以登入您使用 Azure AD 驗證發佈的應用程式。
* [啟用條件式存取](application-proxy-integrate-with-sharepoint-server.md)以存取您發佈的應用程式。
* [將自訂網域名稱新增至 Azure AD](../fundamentals/add-custom-domain.md)


