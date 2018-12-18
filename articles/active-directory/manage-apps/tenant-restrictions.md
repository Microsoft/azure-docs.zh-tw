---
title: 透過限制租用戶管理對雲端應用程式的存取 | Microsoft Docs
description: 如何使用「租用戶限制」以根據使用者的 Azure AD 租用戶來管理可存取應用程式的使用者。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: yossib
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/15/2018
ms.author: barbkess
ms.reviewer: richagi
ms.openlocfilehash: 4a5cb7d15627ac3e48b72567ec3740377b2a0264
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39365665"
---
# <a name="use-tenant-restrictions-to-manage-access-to-saas-cloud-applications"></a>使用租用戶限制來管理對 SaaS 雲端應用程式的存取

注重安全性的大型組織想要移到 Office 365 之類的雲端服務，但需要確知其使用者只能存取獲得核准的資源。 傳統上，當公司想要管理存取時，會限制網域名稱或 IP 位址。 在 SaaS 應用程式裝載於公用雲端而在共用網域名稱 (例如 outlook.office.com 和 login.microsoftonline.com) 上執行的環境中，這個方法行不通。 封鎖這些位址會讓使用者無法存取整個網路上的 Outlook，而不僅是限制他們只能存取已核准的身分識別和資源。

Azure Active Directory 對這項挑戰所提出的解決方案是一個稱為「租用戶限制」的功能。 「租用戶限制」可讓組織根據應用程式用於單一登入的 Azure AD 租用戶來控制對 SaaS 雲端應用程式的存取。 例如，您可能想要允許使用者存取您組織的 Office 365 應用程式，但又防止他們存取其他組織的這些相同應用程式執行個體。  

「租用戶限制」可讓組織指定允許其使用者存取的租用戶清單。 Azure AD 接著便可只授與對這些已允許之租用戶的存取權。

本文將焦點放在 Office 365 的「租用戶限制」，但此功能應該適用於使用新式驗證通訊協定搭配 Azure AD 來進行單一登入的所有 SaaS 雲端應用程式。 如果您使用 SaaS 應用程式搭配與 Office 365 所用租用戶不同的 Azure AD 租用戶，請務必核准所有必要的租用戶。 如需有關 SaaS 雲端應用程式的詳細資訊，請參閱 [Active Directory Marketplace](https://azure.microsoft.com/marketplace/active-directory/)。

## <a name="how-it-works"></a>運作方式

整體解決方案包含下列元件： 

1. **Azure AD** – 如果有 `Restrict-Access-To-Tenants: <permitted tenant list>` 存在，Azure AD 便只會針對已允許的租用戶發出安全性權杖。 

2. **內部部署 Proxy 伺服器基礎結構** – 一個能夠執行 SSL 檢查的 Proxy 裝置，經設定後能將包含已允許之租用戶清單的標頭插入目的地為 Azure AD 的流量中。 

3. **用戶端軟體** – 為了支援「租用戶限制」，用戶端軟體必須直接從 Azure AD 要求權杖，以便 Proxy 基礎結構能夠攔截流量。 目前瀏覽器型 Office 365 應用程式和使用新式驗證 (例如 OAuth 2.0) 的 Office 用戶端都支援「租用戶限制」。 

4. **新式驗證** – 雲端服務必須使用新式驗證，才能使用「租用戶限制」並封鎖對所有非允許之租用戶的存取。 必須將 Office 365 雲端服務設定為預設使用新式驗證通訊協定。 如需有關 Office 365 對新式驗證之支援的最新資訊，請參閱[更新的 Office 365 新式驗證 (英文)](https://blogs.office.com/2015/11/19/updated-office-365-modern-authentication-public-preview/)。

下圖說明概略的流量流程。 只有傳送到 Azure AD 的流量才需要進行 SSL 檢查，傳送到 Office 365 雲端服務的流量則不需要。 這項區別很重要，因為傳送到 Azure AD 以進行驗證的流量通常比傳送到 SaaS 應用程式 (例如 Exchange Online 和 SharePoint Online) 的流量少很多。

![租用戶限制流量流程 - 圖表](./media/tenant-restrictions/traffic-flow.png)

## <a name="set-up-tenant-restrictions"></a>設定租用戶限制

開始使用「租用戶限制」的步驟有兩個。 第一個步驟是確定您的用戶端可以連線到正確的位址。 第二個步驟是設定您的 Proxy 基礎結構。

### <a name="urls-and-ip-addresses"></a>URL 和 IP 位址

若要使用「租用戶限制」，您的用戶端必須要能夠連線到下列 Azure AD URL 來進行驗證：login.microsoftonline.com、login.microsoft.com 及 login.windows.net。 此外，若要存取 Office 365，您的用戶端必須也要能夠連線到 [Office 365 URL 與 IP 位址範圍](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所定義的 FQDN/URL 和 IP 位址。 

### <a name="proxy-configuration-and-requirements"></a>Proxy 組態和需求

以下是透過 Proxy 基礎結構啟用「租用戶限制」的必要組態。 本指導方針是通用的，因此如需了解特定的實作步驟，您應該參考您 Proxy 廠商的文件。

#### <a name="prerequisites"></a>必要條件

- Proxy 必須要能夠執行 SSL 攔截、HTTP 標頭插入，以及使用 FQDN/URL 來篩選目的地。 

- 用戶端必須信任 Proxy 針對 SSL 通訊所出示的憑證鏈結。 例如，如果使用來自內部 PKI 的憑證，就必須信任內部發行的根憑證授權單位憑證。

- Office 365 訂用帳戶中已包含此功能，但如果您想要使用「租用戶限制」來控制對其他 SaaS 應用程式的存取，則需要 Azure AD「進階 1」授權。

#### <a name="configuration"></a>組態

針對每個傳送到 login.microsoftonline.com、login.microsoft.com 及 login.windows.net 的連入要求，請插入兩個 HTTP 標頭：*Restrict-Access-To-Tenants* 和 *Restrict-Access-Context*。

這些標頭應該包含下列元素︰ 
- 就 *Restrict-Access-To-Tenants* 而言，需包含 \<permitted tenant list\> 的值，這是您想要允許使用者存取的租用戶清單 (以逗號分隔)。 與租用戶一起註冊的任何網域都可用來在此清單中識別該租用戶。 例如，若要允許存取 Contoso 和 Fabrikam 租用戶，名稱/值組看起來會像這樣：`Restrict-Access-To-Tenants: contoso.onmicrosoft.com,fabrikam.onmicrosoft.com` 
- 就 *Restrict-Access-Context* 而言，需包含單一目錄識別碼的值，用來宣告設定「租用戶限制」的是哪一個租用戶。 例如，若要宣告 Contoso 是設定「租用戶限制」原則的租用戶，名稱/值組看起來會像這樣：`Restrict-Access-Context: 456ff232-35l2-5h23-b3b3-3236w0826f3d`  

> [!TIP]
> 您可以在 [Azure 入口網站](https://portal.azure.com)中找到您的目錄識別碼。 請以系統管理員身分登入，選取 [Azure Active Directory]，然後選取 [屬性]。

若要防止使用者插入自己的含有非核准租用戶的 HTTP 標頭，Proxy 就必須在連入要求中已經有 Restrict-Access-To-Tenants 標頭時取代此標頭。 

必須強制用戶端針對所有傳送到 login.microsoftonline.com、login.microsoft.com 及 login.windows.net 的要求使用 Proxy。 例如，如果使用 PAC 檔案來指示用戶端使用 Proxy，使用者應該要不能夠編輯或停用 PAC 檔案。

## <a name="the-user-experience"></a>使用者體驗

本節說明使用者和系統管理員的體驗。

### <a name="end-user-experience"></a>使用者體驗

範例使用者位於 Contoso 網路上，但正在嘗試存取 Fabrikam 的共用 SaaS 應用程式執行個體 (例如 Outlook Online)。 如果 Contoso 不是該執行個體所允許的租用戶，使用者就會看到以下頁面：

![針對非允許之租用戶中使用者的拒絕存取頁面](./media/tenant-restrictions/end-user-denied.png)

### <a name="admin-experience"></a>管理員體驗

雖然設定「租用戶限制」時是在公司 Proxy 基礎結構上完成設定，但系統管理員可以直接在 Azure 入口網站中存取「租用戶限制」報告。 若要檢視這些報告，請移至 Azure Active Directory 的 [概觀] 頁面，然後查看 [其他功能] 底下。

如果將租用戶指定為 Restricted-Access-Context 租用戶，其系統管理員便可以使用此報告來查看因「租用戶限制」原則而遭封鎖的登入，包括所使用的身分識別及租用戶目錄識別碼。 如果設定限制的租用戶為登入的使用者租用戶或資源租用戶，則會包含登入。

![使用 Azure 入口網站來檢視受限制的登入嘗試](./media/tenant-restrictions/portal-report.png)

與 Azure 入口網站中的其他報告相同，您可以使用篩選來指定報告的範圍。 您可以依據特定使用者、應用程式、用戶端或時間間隔進行篩選。

## <a name="office-365-support"></a>Office 365 支援

Office 365 應用程式必須符合兩項準則，才能完全支援「租用戶限制」：

1. 所使用的用戶端支援新式驗證
2. 啟用新式驗證作為雲端服務的預設驗證通訊協定。

如需有關哪些 Office 用戶端目前支援新式驗證的最新資訊，請參閱[更新的 Office 365 新式驗證 (英文)](https://blogs.office.com/2015/11/19/updated-office-365-modern-authentication-public-preview/)。 該頁面也包含在特定 Exchange Online 和「商務用 Skype Online」租用戶上啟用新式驗證的指示連結。 SharePoint Online 中已經預設啟用新式驗證。

目前 Office 365 瀏覽器型應用程式 (Office Portal、Yammer、SharePoint 網站、Outlook 網頁版等) 都支援「租用戶限制」。 針對豐富型用戶端 (Outlook、商務用 Skype、Word、Excel、PowerPoint 等)，只有在使用新式驗證的情況下，才能強制執行「租用戶限制」。  

支援新式驗證的 Outlook 和「商務用 Skype」用戶端仍然能夠針對未啟用新式驗證的租用戶使用傳統通訊協定，有效地略過「租用戶限制」。 使用傳統通訊協定的應用程式如果在驗證期間連絡 login.microsoftonline.com、login.microsoft.com 或 login.windows.net，可能會遭到「租用戶限制」封鎖。

針對 Windows 上的 Outlook，客戶可以選擇實作可防止使用者將非已核准郵件帳戶新增至其設定檔的限制。 例如，請參閱[防止新增非預設 Exchange 帳戶](http://gpsearch.azurewebsites.net/default.aspx?ref=1)群組原則設定。

## <a name="testing"></a>測試

如果您想要先試用「租用戶限制」，然後才為整個組織實作此功能，則有兩個選項：運用使用 Fiddler 這類工具的主機型方法，或是分段推出 Proxy 設定。

### <a name="fiddler-for-a-host-based-approach"></a>適用於主機型方法的 Fiddler

Fiddler 是一個免費的 Web 偵錯 Proxy，可用來擷取和修改 HTTP/HTTPS 流量，包括插入 HTTP 標頭。 若要設定 Fiddler 以測試「租用戶限制」，請執行下列步驟：

1.  [下載並安裝 Fiddler](http://www.telerik.com/fiddler)。
2.  依照 [Fiddler 的說明文件 (英文)](http://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS) 操作，設定 Fiddler 以將 HTTPS 流量解密。
3.  設定 Fiddler 以使用自訂規則來插入 *Restrict-Access-To-Tenants* 和 *Restrict-Access-Context* 標頭：
  1. 在「Fiddler Web 偵錯工具」中，選取 [Rules] \(規則) 功能表，然後選取 [Customize Rules] \(自訂規則) 以開啟 CustomRules 檔案。
  2. 在 *OnBeforeRequest* 函式開頭新增下列幾行。 使用與您租用戶一起註冊的網域來取代 \<tenant domain\>，例如 contoso.onmicrosoft.com。 使用您租用戶的 Azure AD GUID 識別碼來取代 \<directory ID\>。

  ```
  if (oSession.HostnameIs("login.microsoftonline.com") || oSession.HostnameIs("login.microsoft.com") || oSession.HostnameIs("login.windows.net")){      oSession.oRequest["Restrict-Access-To-Tenants"] = "<tenant domain>";      oSession.oRequest["Restrict-Access-Context"] = "<directory ID>";}
  ```

  如果您需要允許多個租用戶，請使用逗號來分隔租用戶名稱。 例如︰

  ```
  oSession.oRequest["Restrict-Access-To-Tenants"] = "contoso.onmicrosoft.com,fabrikam.onmicrosoft.com";
  ```

4. 儲存並關閉 CustomRules 檔案。

設定 Fiddler 之後，您可以移至 [File] \(檔案) 功能表並選取 [Capture Traffic] \(擷取流量)，來擷取流量。

### <a name="staged-rollout-of-proxy-settings"></a>分段推出 Proxy 設定

視您 Proxy 基礎結構的功能而定，您可能可以向使用者分段推出設定。 以下是一些可供考量的概略選項：

1.  使用 PAC 檔案將測試使用者指向測試 Proxy 基礎結構，同時又讓一般使用者繼續使用生產環境 Proxy 基礎結構。
2.  有些 Proxy 伺服器可以使用群組來支援不同的組態。

如需具體的詳細資料，請參閱您的 Proxy 伺服器文件。

## <a name="next-steps"></a>後續步驟

- 閱讀[更新的 Office 365 新式驗證 (英文)](https://blogs.office.com/2015/11/19/updated-office-365-modern-authentication-public-preview/)

- 檢閱 [Office 365 URL 與 IP 位址範圍](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)
