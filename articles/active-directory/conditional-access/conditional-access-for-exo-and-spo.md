---
title: 設定 SharePoint Online 和 Exchange Online 以採用 Azure Active Directory 條件式存取 | Microsoft Docs
description: 了解如何設定 SharePoint Online 和 Exchange Online，以便採用 Azure Active Directory 條件式存取。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 62349fba-3cc0-4ab5-babe-372b3389eff6
ms.service: active-directory
ms.component: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/15/2018
ms.author: markvi
ms.reviewer: calebb
ms.openlocfilehash: a5f0a17682f3481ad9a2dff5f144aa3d97f29920
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063841"
---
# <a name="how-to-set-up-sharepoint-online-and-exchange-online-for-azure-active-directory-conditional-access"></a>作法：設定 SharePoint Online 和 Exchange Online，以便採用 Azure Active Directory 條件式存取 

透過 [Azure Active Directory (Azure AD) 條件式存取](overview.md)，您可以控制使用者如何存取您的雲端應用程式。 如果您想要使用條件式存取來控制 SharePoint Online 和 Exchange Online 的存取權限，您需要：

- 檢閱您的條件式存取案例是否受到支援
- 預防用戶端應用程式略過強制執行的條件式存取原則。   

本文說明如何解決這兩種情況。


## <a name="what-you-need-to-know"></a>您所需了解的事情

您可以使用 Azure AD 條件式存取來保護雲端應用程式，因應來自以下來源的驗證嘗試：

- 網頁瀏覽器

- 使用[新式驗證](https://support.office.com/article/Using-Office-365-modern-authentication-with-Office-clients-776c0036-66fd-41cb-8928-5495c0f9168a)的用戶端應用程式

- Exchange ActiveSync 

有些雲端應用程式也支援舊版驗證通訊協定。 SharePoint Online 和 Exchange Online 即是一例。 當用戶端應用程式可以使用舊版驗證通訊協定來存取雲端應用程式時，Azure AD 無法針對該存取嘗試強制執行條件式存取原則。 若要預防用戶端應用程式略過強制執行的原則，您應該確認是否可以只針對受影響的雲端應用程式啟用新式驗證。 

不適用用戶端應用程式條件式存取的範例包括：

- Office 2010 和先前版本

- Office 2013 (未啟用新式驗證時)



 
## <a name="control-access-to-sharepoint-online"></a>控制 SharePoint Online 的存取

除了新式驗證之外，SharePoint Online 也支援舊版驗證通訊協定。 如果舊版驗證通訊協定已啟用，對於不使用新式驗證的用戶端，您的 SharePoint 條件式存取原則便不會強制執行。

您可以使用 **[Set-SPOTenant](https://technet.microsoft.com/library/fp161390.aspx)** Cmdlet 停用 SharePoint 存取的舊版驗證通訊協定： 

    Set-SPOTenant -LegacyAuthProtocolsEnabled $false

## <a name="control-access-to-exchange-online"></a>控制 Exchange Online 的存取

在設定 Exchange Online 的條件式存取原則時，您需要檢視下列各項：

- Exchange ActiveSync

- 舊版驗證通訊協定



### <a name="exchange-activesync"></a>Exchange ActiveSync

Exchange Active Sync 雖然支援新式驗證，不過有些與條件式存取案例支援相關的限制：

- 您只能設定裝置平台條件  

    ![裝置平台](./media/conditional-access-for-exo-and-spo/05.png)

- 不支援設定多重要素驗證要求  

    ![條件式存取](./media/conditional-access-for-exo-and-spo/01.png)

若要有效保護從 Exchange ActiveSync 到 Exchange Online 的存取，您可以：

- 按照以下步驟設定支援的條件式存取原則：

    a. 只將 [Office 365 Exchange Online] 選為雲端應用程式。  

    ![條件式存取](./media/conditional-access-for-exo-and-spo/04.png)

    b. 將 [Exchange Active Sync] 選為**用戶端應用程式**，然後選取 [只將原則套用到支援的平台]。  

    ![裝置平台](./media/conditional-access-for-exo-and-spo/03.png)

- 使用 Active Directory 同盟服務 (AD FS) 規則以封鎖 Exchange ActiveSync。

        @RuleName = "Block Exchange ActiveSync"
        c1:[Type == "http://schemas.microsoft.com/2012/01/requestcontext/claims/x-ms-client-application", Value == "Microsoft.Exchange.ActiveSync"]
        => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "false");




### <a name="legacy-authentication-protocols"></a>舊版驗證通訊協定

除了新式驗證之外，Exchange Online 也支援舊版驗證通訊協定。 如果舊版驗證通訊協定已啟用，對於不使用新式驗證的用戶端，您的 Exchange Online 條件式存取原則便不會強制執行。

您可以設定 AD FS 規則，以停用 Exchange Online 的舊版驗證通訊協定。 如此將能封鎖來自以下來源的存取：

- 未啟用新式驗證的舊版 Office 用戶端 (如 Office 2013) 

- 先前版本的 Office


## <a name="set-up-ad-fs-rules"></a>設定 AD FS 規則

您可以使用下列發行授權規則，在 AD FS 層級啟用或封鎖流量。 

### <a name="block-legacy-traffic-from-the-extranet"></a>封鎖來自外部網路的舊版流量

套用以下三個規則： 

- 啟用以下項目的存取：
    - Exchange ActiveSync 流量
    - 瀏覽器流量
    - 新式驗證流量
- 封鎖以下項目的存取： 
    - 來自外部網路的舊版用戶端應用程式

**規則 1：**

    @RuleName = "Allow all intranet traffic"
    c1:[Type == "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork", Value == "true"]
    => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "true");

**規則 2：**

    @RuleName = "Allow Exchange ActiveSync"
    c1:[Type == "http://schemas.microsoft.com/2012/01/requestcontext/claims/x-ms-client-application", Value == "Microsoft.Exchange.ActiveSync"]
    => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "true");

**規則 3：**

    @RuleName = "Allow extranet browser and browser dialog traffic"
    c1:[Type == "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork", Value == "false"] &&
    c2:[Type == "http://schemas.microsoft.com/2012/01/requestcontext/claims/x-ms-endpoint-absolute-path", Value =~ "(/adfs/ls)|(/adfs/oauth2)"]
    => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "true");

### <a name="block-legacy-traffic-from-anywhere"></a>封鎖來自任何位置的舊版流量

套用以下三個規則：

- 啟用以下項目的存取： 
    - Exchange ActiveSync 流量
    - 瀏覽器流量
    - 新式驗證流量
- 封鎖以下項目的存取： 
    - 來自任何位置的舊版應用程式

##### <a name="rule-1"></a>規則 1
    @RuleName = "Allow all intranet traffic only for browser and modern authentication clients"
    c1:[Type == "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork", Value == "true"] &&
    c2:[Type == "http://schemas.microsoft.com/2012/01/requestcontext/claims/x-ms-endpoint-absolute-path", Value =~ "(/adfs/ls)|(/adfs/oauth2)"]
    => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "true");


##### <a name="rule-2"></a>規則 2
    @RuleName = "Allow Exchange ActiveSync"
    c1:[Type == "http://schemas.microsoft.com/2012/01/requestcontext/claims/x-ms-client-application", Value == "Microsoft.Exchange.ActiveSync"]
    => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "true");


##### <a name="rule-3"></a>規則 3
    @RuleName = "Allow extranet browser and browser dialog traffic"
    c1:[Type == "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork", Value == "false"] &&
    c2:[Type == "http://schemas.microsoft.com/2012/01/requestcontext/claims/x-ms-endpoint-absolute-path", Value =~ "(/adfs/ls)|(/adfs/oauth2)"]
    => issue(Type = "http://schemas.microsoft.com/authorization/claims/permit", Value = "true");

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱[什麼是 Azure Active Directory 中的條件式存取](overview.md)。

如需設定宣告規則的相關指示，請參閱[設定宣告規則](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-claim-rules)。 






