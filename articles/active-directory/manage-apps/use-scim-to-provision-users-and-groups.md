---
title: 在 Azure Active Directory 中使用 SCIM 自動佈建應用程式 | Microsoft Docs
description: Azure Active Directory 會利用 SCIM 通訊協定規格中定義的介面，自動佈建使用者和群組到 Web 服務前端的任何應用程式或身分識別存放區
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: barbkess
ms.reviewer: asmalser
ms.custom: aaddev;it-pro;seohack1
ms.openlocfilehash: d99996bb4177a11342f0897d2cf69f82a5b81bf7
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44721223"
---
# <a name="using-system-for-cross-domain-identity-management-scim-to-automatically-provision-users-and-groups-from-azure-active-directory-to-applications"></a>使用 System for Cross-Domain Identity Management (SCIM) 自動將使用者和群組從 Azure Active Directory 佈建到應用程式

## <a name="overview"></a>概觀
Azure Active Directory (Azure AD) 會利用 [System for Cross-Domain Identity Management (SCIM) 2.0 通訊協定規格](https://tools.ietf.org/html/draft-ietf-scim-api-19)中定義的介面，自動佈建使用者和群組到 Web 服務前端的任何應用程式或身分識別存放區。 Azure Active Directory 可以傳送要求以建立、修改或刪除指派給 Web 服務的使用者和群組。 然後 Web 服務可以將這些要求轉譯成目標身分識別存放區上的作業。 

![][0]
圖 1：透過 Web 服務從 Azure Active Directory 佈建到身分識別存放區

這項功能可用來與 Azure AD 中的「自備應用程式」功能搭配使用。 這項功能可為 SCIM Web 服務前方的應用程式啟用單一登入和自動使用者佈建。

在 Azure Active Directory 中使用 SCIM 有兩個使用案例：

* **將使用者與群組佈建至支援 SCIM 的應用程式** - 應用程式若支援 SCIM 2.0，而且使用 OAuth 持有人權杖進行驗證，將可直接與 Azure AD 搭配運作，不需其他設定。
  
* **為支援其他 API 型佈建的應用程式建置您自己的佈建解決方案** - 對於非 SCIM 應用程式，您可以建立能夠在 Azure AD SCIM 端點與應用程式為使用者佈建支援的任何 API 之間進行轉譯的 SCIM 端點。 為了協助您開發 SCIM 端點，通用語言基礎結構 (CLI) 程式庫中的程式碼範例可向您說明如何提供 SCIM 端點及轉譯 SCIM 訊息。  

## <a name="provisioning-users-and-groups-to-applications-that-support-scim"></a>將使用者與群組佈建至支援 SCIM 的應用程式
Azure AD 可以設定為將已指派的使用者和群組佈建至實作 [System for Cross-domain Identity Management 2 (SCIM)](https://tools.ietf.org/html/draft-ietf-scim-api-19) Web 服務、並接受以 OAuth 持有人權杖進行驗證的應用程式。 在 SCIM 2.0 規格中，應用程式必須符合下列需求：

* 支援根據 SCIM 通訊協定 3.3 小節建立使用者和 (或) 群組的作業。  
* 支援根據 SCIM 通訊協定 3.5.2 小節修改具有修補要求的使用者和 (或) 群組的作業。  
* 支援根據 SCIM 通訊協定 3.4.1 小節擷取已知資源的作業。  
* 支援根據 SCIM 通訊協定 3.4.2 小節查詢使用者和 (或) 群組的作業。  依預設會依 externalId 來查詢使用者，並依 displayName 查詢群組。  
* 支援根據 SCIM 通訊協定 3.4.2 小節，依 ID 和管理員查詢使用者的作業。  
* 支援根據 SCIM 通訊協定 3.4.2 小節，依 ID 和成員查詢群組的作業。  
* 接受根據 SCIM 通訊協定 2.1 小節以 OAuth 持有人權杖進行授權的作法。

洽詢應用程式提供者，或參閱應用程式提供者文件中的相關陳述，以了解是否符合這些需求。

### <a name="getting-started"></a>開始使用
支援本文所述 SCIM 設定檔的應用程式，可使用 Azure AD 應用程式庫中的「不在資源庫內的應用程式」功能連線到 Azure Active Directory。 連線之後，Azure AD 會每隔 40 分鐘執行一次同步處理程序，此程序會為指派的使用者和群組查詢應用程式的 SCIM 端點，並根據指派詳細資料加以建立或修改。

**若要連接支援 SCIM 的應用程式：**

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 瀏覽至 [Azure Active Directory] > [企業應用程式]，然後選取 [新增應用程式] > [全部] > [不在資源庫內的應用程式]。
3. 輸入您的應用程式名稱，然後按一下 [新增] 圖示，以建立應用程式物件。
    
  ![][1]
  圖 2：使用 Azure AD 應用程式庫
    
4. 在結果畫面中，選取左側資料行中的 [佈建] 索引標籤。
5. 在 [佈建模式] 功能表上，選取 [自動]。
    
  ![][2]
  圖 3：在 Azure 入口網站中設定佈建
    
6. 在 [租用戶 URL] 欄位中，輸入應用程式 SCIM 端點的 URL。 範例： https://api.contoso.com/scim/v2/
7. 如果 SCIM 端點需要來自非 Azure AD 簽發者的 OAuth 持有人權杖，那麼便將所需的 OAuth 持有人權杖複製到選擇性 [祕密權杖] 欄位。 如果此欄位保留空白，則 Azure AD 會在每個要求包含從 Azure AD 簽發的 OAuth 持有人權杖。 使用 Azure AD 作為識別提供者的應用程式，可以驗證此 Azure AD 簽發的權杖。
8. 按一下 [測試連線] 按鈕，讓 Azure Active Directory 嘗試連線到 SCIM 端點。 如果嘗試失敗，則會顯示錯誤資訊。  
9. 如果嘗試連線到應用程式成功，則按一下 [儲存] 以儲存管理員認證。
10. 在 [對應] 區段中，有兩組可選取的屬性對應：一個用於使用者物件，一個用於群組物件。 選取其中一個以檢閱從 Azure Active Directory 同步處理至應用程式的屬性。 選取為 [比對] 屬性的屬性會用來比對應用程式中的使用者和群組以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

    >[!NOTE]
    >您可以選擇性地藉由停用「群組」對應以停用同步處理群組物件。 

11. 在 [設定] 底下的 [範圍] 欄位定義哪些使用者或群組會進行同步處理。 選取 [僅同步處理指派的使用者和群組]\(建議選項) 只會同步處理 [使用者和群組] 索引標籤中指派的使用者和群組。
12. 一旦您的設定完成，請將 [佈建狀態] 變更為 [開啟]。
13. 按一下 [儲存] 以啟動 Azure AD 佈建服務。 
14. 如果僅同步處理指派的使用者和群組 (建議選項)，請務必選取 [使用者和群組] 索引標籤，並且指派您想要同步處理的使用者和/或群組。

一旦啟動初始同步處理，您可以使用 [稽核記錄] 索引標籤以監視進度，進步會顯示佈建服務在您的應用程式上執行的所有動作。 如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](check-status-user-account-provisioning.md)。

>[!NOTE]
>初始同步處理會比後續同步處理花費更多時間執行，只要服務正在執行，這大約每 40 分鐘便會發生一次。 


## <a name="building-your-own-provisioning-solution-for-any-application"></a>為任何應用程式建置您自己的佈建解決方案
建立可與 Azure Active Directory 互動的 SCIM Web 服務後，您即可為提供 REST 或 SOAP 使用者佈建 API 的絕大多數應用程式啟用單一登入和自動使用者佈建。

其運作方式如下：

1. Azure AD 提供名為 [Microsoft.SystemForCrossDomainIdentityManagement](https://www.nuget.org/packages/Microsoft.SystemForCrossDomainIdentityManagement/)的通用語言基礎結構程式庫。 系統整合業者和開發人員可以使用此程式庫來建立及部署能夠將 Azure AD 連線到任何應用程式的身分識別存放區的 SCIM 式 Web 服務端點。
2. 對應會在 Web 服務中實作，以將標準的使用者結構描述對應到使用者結構描述和應用程式所需的通訊協定。
3. 端點 URL 會在 Azure AD 中註冊，作為應用程式資源庫中自訂應用程式的一部分。
4. 使用者和群組會在 Azure AD 中指派給此應用程式。 指派時，它們會被放入佇列，以同步處理至目標應用程式。 同步處理程序會每隔 40 分鐘處理佇列的執行。

### <a name="code-samples"></a>程式碼範例
為了讓這個程序更簡單，我們提供了[程式碼範例](https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master)，該範例會建立 SCIM Web 服務端點並示範自動佈建。 其中一個範例是維護代表使用者和群組、具有逗號分隔值資料列檔案的提供者。  另一個是在 Amazon Web 服務身分識別與存取管理服務上運作的提供者。  

**先決條件**

* Visual Studio 2013 或更新版本
* [Azure SDK for .NET](https://azure.microsoft.com/downloads/)
* 支援將 ASP.NET Framework 4.5 用作 SCIM 端點的 Windows 電腦。 必須能夠自雲端存取這台電腦
* [具有 Azure AD Premium 試用版或授權版的 Azure 訂用帳戶](https://azure.microsoft.com/services/active-directory/)

### <a name="getting-started"></a>開始使用
實作可以接受來自 Azure AD 的佈建要求的 SCIM 端點的最簡單的方式是建置和部署會將佈建的使用者輸出至以逗號分隔值 (CSV) 檔案的程式碼範例。

**若要建立範例 SCIM 端點：**

1. 請至 [https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master](https://github.com/Azure/AzureAD-BYOA-Provisioning-Samples/tree/master) 下載程式碼範例套件。
2. 將套件解壓縮並將放在 Windows 電腦上的位置，例如 C:\AzureAD-BYOA-Provisioning-Samples\。
3. 在此資料夾中，啟動 Visual Studio 中的 FileProvisioning\Host\FileProvisioningService.csproj 專案。
4. 選取 [工具] > [NuGet 套件管理員] > [套件管理員主控台]，然後執行以下命令，讓 FileProvisioningService 專案解析方案參考：
  ```` 
   Update-Package -Reinstall
  ````
5. 建置 FileProvisioningService 專案。
6. 在 Windows 中啟動「命令提示字元」應用程式 (以系統管理員身分)，然後使用 **cd** 命令將目錄變更為您的 **\AzureAD-BYOA-Provisioning-Samples\FileProvisioning\Host\bin\Debug** 資料夾。
7. 執行以下命令，以 Windows 電腦的 IP 位址或網域名稱取代 <ip-address>：
  ````   
   FileSvc.exe http://<ip-address>:9000 TargetFile.csv
  ````
8. 在 Windows 中，於 [Windows 設定] > [網路和網際網路設定] 底下，選取 [Windows 防火牆] > [進階設定]，然後建立允許對連接埠 9000 進行輸入存取的「輸入規則」。
9. 如果 Windows 電腦位於路由器背後，必須將路由器設定為在公開到網際網路的連接埠 9000 和 Windows 電腦上的連接埠 9000 之間執行網路存取轉譯。 為了讓 Azure AD 能夠在雲端中存取這個端點，這是必要的組態。

**若要在 Azure AD 中註冊範例 SCIM 端點：**

1. 登入 [Azure 入口網站](https://portal.azure.com)。 
2. 瀏覽至 [Azure Active Directory] > [企業應用程式]，然後選取 [新增應用程式] > [全部] > [不在資源庫內的應用程式]。
3. 輸入您的應用程式名稱，然後按一下 [新增] 圖示，以建立應用程式物件。 建立的應用程式物件要代表您要佈建和實作登一登入的目標應用程式，而不只是 SCIM 端點。
4. 在結果畫面中，選取左側資料行中的 [佈建] 索引標籤。
5. 在 [佈建模式] 功能表上，選取 [自動]。
    
  ![][2]
  圖 4：在 Azure 入口網站中設定佈建
    
6. 在 [租用戶 URL] 欄位中，輸入網際網路公開的 URL 和 SCIM 端點的連接埠。 此項目就像是 http://testmachine.contoso.com:9000 或 http://<ip-address>:9000/，其中 <ip-address> 是網際網路公開 IP 位址。  
7. 如果 SCIM 端點需要來自非 Azure AD 簽發者的 OAuth 持有人權杖，那麼便將所需的 OAuth 持有人權杖複製到選擇性 [祕密權杖] 欄位。 如果此欄位保留空白，則 Azure AD 將在每個要求包含從 Azure AD 簽發的 OAuth 持有人權杖。 使用 Azure AD 作為識別提供者的應用程式，可以驗證此 Azure AD 簽發的權杖。
8. 按一下 [測試連線] 按鈕，讓 Azure Active Directory 嘗試連線到 SCIM 端點。 如果嘗試失敗，則會顯示錯誤資訊。  
9. 如果嘗試連線到應用程式成功，則按一下 [儲存] 以儲存管理員認證。
10. 在 [對應] 區段中，有兩組可選取的屬性對應：一個用於使用者物件，一個用於群組物件。 選取其中一個以檢閱從 Azure Active Directory 同步處理至應用程式的屬性。 選取為 [比對] 屬性的屬性會用來比對應用程式中的使用者和群組以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。
11. 在 [設定] 底下的 [範圍] 欄位定義哪些使用者或群組會進行同步處理。 選取 [僅同步處理指派的使用者和群組]\(建議選項) 只會同步處理 [使用者和群組] 索引標籤中指派的使用者和群組。
12. 一旦您的設定完成，請將 [佈建狀態] 變更為 [開啟]。
13. 按一下 [儲存] 以啟動 Azure AD 佈建服務。 
14. 如果僅同步處理指派的使用者和群組 (建議選項)，請務必選取 [使用者和群組] 索引標籤，並且指派您想要同步處理的使用者和/或群組。

一旦啟動初始同步處理，您可以使用 [稽核記錄] 索引標籤以監視進度，進步會顯示佈建服務在您的應用程式上執行的所有動作。 如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](check-status-user-account-provisioning.md)。

確認此範例的最後一個步驟是開啟 Windows 電腦上 \AzureAD-BYOA-Provisioning-Samples\ProvisioningAgent\bin\Debug 資料夾中的 TargetFile.csv 檔案。 一旦執行佈建程序，此檔案會顯示所有指派和佈建的使用者和群組的詳細資料。

### <a name="development-libraries"></a>開發程式庫
若要開發自己符合 SCIM 規格的 Web 服務，請先熟悉下列 Microsoft 所提供、有助於加速開發程序的程式庫： 

1. 提供通用語言基礎結構 (CLI) 程式庫，可與以該基礎結構為基礎的語言 (例如 C#) 搭配使用。 其中一個程式庫，[Microsoft.SystemForCrossDomainIdentityManagement.Service](https://www.nuget.org/packages/Microsoft.SystemForCrossDomainIdentityManagement/)，會宣告介面 (Microsoft.SystemForCrossDomainIdentityManagement.IProvider)，如下圖所示：使用程式庫的開發人員會以參考 (通常是作為提供者) 的類別來實作該介面。 程式庫可讓開發人員部署符合 SCIM 規格的 Web 服務。 Web 服務可以裝載在 Internet Information Services 或任何可執行的通用語言基礎結構組件。 要求會轉譯成對提供者的方法呼叫，該呼叫會由開發人員以程式方式設計，以對某些身分識別存放區進行操作。
  
  ![][3]
  
2. [ExpressRoute 處理常式](http://expressjs.com/guide/routing.html)可剖析代表對 node.js Web 服務發出之呼叫 (如 SCIM 規格所定義) 的 node.js 要求物件。   

### <a name="building-a-custom-scim-endpoint"></a>建置自訂 SCIM 端點
使用 CLI 程式庫，使用這些程式庫的開發人員可以將其服務託管在任何可執行的通用語言基礎結構組件內，或在網際網路資訊服務內。 以下是範例程式碼，此程式碼可將服務裝載於位於 http://localhost:9000: 位址的可執行組件內： 

    private static void Main(string[] arguments)
    {
    // Microsoft.SystemForCrossDomainIdentityManagement.IMonitor, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IProvider and 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service are all defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service.dll.  

    Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor = 
      new DevelopersMonitor();
    Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider = 
      new DevelopersProvider(arguments[1]);
    Microsoft.SystemForCrossDomainIdentityManagement.Service webService = null;
    try
    {
        webService = new WebService(monitor, provider);
        webService.Start("http://localhost:9000");

        Console.ReadKey(true);
    }
    finally
    {
        if (webService != null)
        {
            webService.Dispose();
            webService = null;
        }
    }
    }

    public class WebService : Microsoft.SystemForCrossDomainIdentityManagement.Service
    {
    private Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor;
    private Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider;

    public WebService(
      Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitoringBehavior, 
      Microsoft.SystemForCrossDomainIdentityManagement.IProvider providerBehavior)
    {
        this.monitor = monitoringBehavior;
        this.provider = providerBehavior;
    }

    public override IMonitor MonitoringBehavior
    {
        get
        {
            return this.monitor;
        }

        set
        {
            this.monitor = value;
        }
    }

    public override IProvider ProviderBehavior
    {
        get
        {
            return this.provider;
        }

        set
        {
            this.provider = value;
        }
    }
    }

這項服務必須具有 HTTP 位址，而其伺服器驗證憑證的根憑證授權單位是下列其中一個名稱： 

* CNNIC
* Comodo
* CyberTrust
* DigiCert
* GeoTrust
* GlobalSign
* Go Daddy
* VeriSign
* WoSign

伺服器驗證憑證可以使用網路殼層公用程式繫結到 Windows 主機上的連接埠： 

    netsh http add sslcert ipport=0.0.0.0:443 certhash=0000000000003ed9cd0c315bbb6dc1c08da5e6 appid={00112233-4455-6677-8899-AABBCCDDEEFF}  

此處，對 certhash 引數提供的值為憑證指紋，而對 appid 引數提供的值為任意的全域唯一識別碼。  

若要將服務託管在網際網路資訊服務內，開發人員會建置 CLA 程式碼程式庫組件，並在組件的預設命名空間中使用名為 Startup 的類別。  以下是這種類別的範例： 

    public class Startup
    {
    // Microsoft.SystemForCrossDomainIdentityManagement.IWebApplicationStarter, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IMonitor and  
    // Microsoft.SystemForCrossDomainIdentityManagement.Service are all defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Service.dll.  

    Microsoft.SystemForCrossDomainIdentityManagement.IWebApplicationStarter starter;

    public Startup()
    {
        Microsoft.SystemForCrossDomainIdentityManagement.IMonitor monitor = 
          new DevelopersMonitor();
        Microsoft.SystemForCrossDomainIdentityManagement.IProvider provider = 
          new DevelopersProvider();
        this.starter = 
          new Microsoft.SystemForCrossDomainIdentityManagement.WebApplicationStarter(
            provider, 
            monitor);
    }

    public void Configuration(
      Owin.IAppBuilder builder) // Defined in Owin.dll.  
    {
        this.starter.ConfigureApplication(builder);
    }
    }

### <a name="handling-endpoint-authentication"></a>處理端點驗證
來自 Azure Active Directory 的要求包括 OAuth 2.0 持有人權杖。   接收要求的任何服務應該代表預期的 Azure Active Directory 租用戶，將簽發者驗證為 Azure Active Directory，以存取 Azure Active Directory 圖形 Web 服務。  在權杖中，是用 iss 宣告識別簽發者，例如："iss":"https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/"。  在此範例中，宣告值的基礎位址 https://sts.windows.net 會將 Azure Active Directory 識別為簽發者，而相對位址區段 cbb1a5ac-f33b-45fa-9bf5-f37db0fed422 則是簽發權杖時所代表之 Azure Active Directory 租用戶的唯一識別碼。  如果發出的權杖要用於存取 Azure Active Directory 圖形 Web 服務，則該服務的識別項 00000002-0000-0000-c000-000000000000，應該位於權杖的 aud 宣告中的值。  

開發人員若使用 Microsoft 所提供的 CLA 程式庫來建置 SCIM 服務，可以依照下列步驟使用 Microsoft.Owin.Security.ActiveDirectory 套件以驗證來自 Azure Active Directory 的要求： 

1. 在提供者中，透過讓 Microsoft.SystemForCrossDomainIdentityManagement.IProvider.StartupBehavior 屬性在每次服務啟動時傳回要呼叫的方法，來實作此屬性： 

  ````
    public override Action\<Owin.IAppBuilder, System.Web.Http.HttpConfiguration.HttpConfiguration\> StartupBehavior
    {
      get
      {
        return this.OnServiceStartup;
      }
    }

    private void OnServiceStartup(
      Owin.IAppBuilder applicationBuilder,  // Defined in Owin.dll.  
      System.Web.Http.HttpConfiguration configuration)  // Defined in System.Web.Http.dll.  
    {
    }
  ````

2. 將下列程式碼新增到該方法中，以將對任何服務端點發出的任何要求，都驗證為持有 Azure Active Directory 代表指定租用戶簽發的權杖，以存取 Azure AD 圖形 Web 服務： 

  ````
    private void OnServiceStartup(
      Owin.IAppBuilder applicationBuilder IAppBuilder applicationBuilder, 
      System.Web.Http.HttpConfiguration HttpConfiguration configuration)
    {
      // IFilter is defined in System.Web.Http.dll.  
      System.Web.Http.Filters.IFilter authorizationFilter = 
        new System.Web.Http.AuthorizeAttribute(); // Defined in System.Web.Http.dll.configuration.Filters.Add(authorizationFilter);

      // SystemIdentityModel.Tokens.TokenValidationParameters is defined in    
      // System.IdentityModel.Token.Jwt.dll.
      SystemIdentityModel.Tokens.TokenValidationParameters tokenValidationParameters =     
        new TokenValidationParameters()
        {
          ValidAudience = "00000002-0000-0000-c000-000000000000"
        };

      // WindowsAzureActiveDirectoryBearerAuthenticationOptions is defined in 
      // Microsoft.Owin.Security.ActiveDirectory.dll
      Microsoft.Owin.Security.ActiveDirectory.
      WindowsAzureActiveDirectoryBearerAuthenticationOptions authenticationOptions =
        new WindowsAzureActiveDirectoryBearerAuthenticationOptions()    {
        TokenValidationParameters = tokenValidationParameters,
        Tenant = "03F9FCBC-EA7B-46C2-8466-F81917F3C15E" // Substitute the appropriate tenant’s 
                                                      // identifier for this one.  
      };

      applicationBuilder.UseWindowsAzureActiveDirectoryBearerAuthentication(authenticationOptions);
    }
  ````


## <a name="user-and-group-schema"></a>使用者和群組結構描述
Azure Active Directory 可以佈建兩種類型的資源至 SCIM Web 服務。  這些類型的資源是使用者和群組。  

用結構描述識別碼 "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" 識別使用者資源，此識別碼包含在下列通訊協定規格中： http://tools.ietf.org/html/draft-ietf-scim-core-schema 。  以下的表 1 提供相對於 "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" 資源的屬性，Azure Active Directory 中使用者屬性的預設對應。  

用結構描述識別碼 http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group 識別群組資源。  下面的表 2 顯示 Azure Active Directory 中的群組屬性與 http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group 資源屬性的預設對應。  

### <a name="table-1-default-user-attribute-mapping"></a>表 1：預設使用者屬性對應
| Azure Active Directory 使用者 | "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" |
| --- | --- |
| IsSoftDeleted |作用中 |
| displayName |displayName |
| Facsimile-TelephoneNumber |phoneNumbers[type eq "fax"].value |
| givenName |name.givenName |
| jobTitle |title |
| mail |emails[type eq "work"].value |
| mailNickname |externalId |
| manager |manager |
| mobile |phoneNumbers[type eq "mobile"].value |
| objectId |ID |
| postalCode |addresses[type eq "work"].postalCode |
| proxy-Addresses |emails[type eq "other"].Value |
| physical-Delivery-OfficeName |addresses[type eq "other"].Formatted |
| streetAddress |addresses[type eq "work"].streetAddress |
| surname |name.familyName |
| telephone-Number |phoneNumbers[type eq "work"].value |
| user-PrincipalName |userName |

### <a name="table-2-default-group-attribute-mapping"></a>表 2：預設群組屬性對應
| Azure Active Directory 群組 | http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group |
| --- | --- |
| displayName |externalId |
| mail |emails[type eq "work"].value |
| mailNickname |displayName |
| members |members |
| objectId |ID |
| proxyAddresses |emails[type eq "other"].Value |

## <a name="user-provisioning-and-de-provisioning"></a>使用者佈建和取消佈建
下圖顯示 Azure Active Directory 會傳送至 SCIM 服務的訊息，以管理使用者在其他身分識別存放區中的生命週期。 圖表也會示範使用 Microsoft 提供、用於建置此類服務的 CLI 程式庫所實作之 SCIM 服務如何將這些要求轉譯為對提供者的方法呼叫。  

![][4]
圖 5：使用者佈建和取消佈建順序

1. Azure Active Directory 會查詢服務是否有 externalId 屬性值與 Azure AD 中使用者的 mailNickname 屬性值相符的使用者。 查詢會以類似於此範例的超文字傳輸通訊協定 (HTTP) 要求表示，其中，jyoung 是 Azure Active Directory 中使用者的 mailNickname 範例： 
  ````
    GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
    Authorization: Bearer ...
  ````
  如果是使用 Microsoft 所提供、用於實作 SCIM 服務的通用語言基礎結構程式庫建置服務，會將要求轉譯為對服務提供者的 Query 方法的呼叫。  以下是該方法的簽章： 
  ````
    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Protocol.  

    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource[]> Query(
      Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters parameters, 
      string correlationIdentifier);
  ````
  以下是 Microsoft.SystemForCrossDomainIdentityManagement.IQueryParameters 介面的定義： 
  ````
    public interface IQueryParameters: 
      Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
    {
        System.Collections.Generic.IReadOnlyCollection <Microsoft.SystemForCrossDomainIdentityManagement.IFilter> AlternateFilters 
        { get; }
    }

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IRetrievalParameters
    {
      system.Collections.Generic.IReadOnlyCollection<string> ExcludedAttributePaths 
      { get; }
      System.Collections.Generic.IReadOnlyCollection<string> RequestedAttributePaths 
      { get; }
      string SchemaIdentifier 
      { get; }
    }

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IFilter
    {
        Microsoft.SystemForCrossDomainIdentityManagement.IFilter AdditionalFilter 
          { get; set; }
        string AttributePath 
          { get; } 
        Microsoft.SystemForCrossDomainIdentityManagement.ComparisonOperator FilterOperator 
          { get; }
        string ComparisonValue 
          { get; }
    }

    public enum Microsoft.SystemForCrossDomainIdentityManagement.ComparisonOperator
    {
        Equals
    }
  ````
  在查詢具有給定值 externalId 屬性的使用者的下列範例中，傳遞至 Query 方法的引數值為： 
  * parameters.AlternateFilters.Count: 1
  * parameters.AlternateFilters.ElementAt(0).AttributePath: "externalId"
  * parameters.AlternateFilters.ElementAt(0).ComparisonOperator: ComparisonOperator.Equals
  * parameters.AlternateFilter.ElementAt(0).ComparisonValue: "jyoung"
  * correlationIdentifier: System.Net.Http.HttpRequestMessage.GetOwinEnvironment["owin.RequestId"] 

2. 如果向 Web 服務查詢是否有 externalId 屬性值與使用者的 mailNickname 值相符的使用者時，回應未傳回任何使用者，Azure Active Directory 就會要求服務佈建與 Azure Active Directory 中的使用者對應的使用者。  以下是這類要求的範例： 
  ````
    POST https://.../scim/Users HTTP/1.1
    Authorization: Bearer ...
    Content-type: application/scim+json
    {
      "schemas":
      [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0User"],
      "externalId":"jyoung",
      "userName":"jyoung",
      "active":true,
      "addresses":null,
      "displayName":"Joy Young",
      "emails": [
        {
          "type":"work",
          "value":"jyoung@Contoso.com",
          "primary":true}],
      "meta": {
        "resourceType":"User"},
       "name":{
        "familyName":"Young",
        "givenName":"Joy"},
      "phoneNumbers":null,
      "preferredLanguage":null,
      "title":null,
      "department":null,
      "manager":null}
  ````
  Microsoft 所提供、用於實作 SCIM 服務的通用語言基礎結構程式庫，會將要求轉譯為對服務提供者的 Create 方法的呼叫。  Create 方法具有此簽章： 
  ````
    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource is defined in 
    // Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  

    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource> Create(
      Microsoft.SystemForCrossDomainIdentityManagement.Resource resource, 
      string correlationIdentifier);
  ````
  在佈建使用者的要求中，資源引數的值會是 Microsoft.SystemForCrossDomainIdentityManagement 的執行個體。 Microsoft.SystemForCrossDomainIdentityManagement.Schemas 程式庫中定義的 Core2EnterpriseUser 類別。  如果佈建使用者的要求成功，則方法的實作應該會傳回 Microsoft.SystemForCrossDomainIdentityManagement 的執行個體。 Core2EnterpriseUser 類別，且其識別碼屬性值設定為新佈建使用者的唯一識別碼。  

3. 為了更新已知存在於前端為 SCIM 之身分識別存放區中的使用者，Azure Active Directory 會以類似下方的要求向服務要求該使用者的目前狀態，來繼續執行： 
  ````
    GET ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...
  ````
  如果是使用 Microsoft 所提供、用於實作 SCIM 服務的通用語言基礎結構程式庫建置服務，會將要求轉譯為對服務提供者的 Retrieve 方法的呼叫。  以下是 Retrieve 方法的簽章： 
  ````
    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.Resource and 
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters 
    // are defined in Microsoft.SystemForCrossDomainIdentityManagement.Schemas.  
    System.Threading.Tasks.Task<Microsoft.SystemForCrossDomainIdentityManagement.Resource> 
       Retrieve(
         Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters 
           parameters, 
           string correlationIdentifier);

    public interface 
      Microsoft.SystemForCrossDomainIdentityManagement.IResourceRetrievalParameters:   
        IRetrievalParameters
        {
          Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier 
            ResourceIdentifier 
              { get; }
    }
    public interface Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier
    {
        string Identifier 
          { get; set; }
        string Microsoft.SystemForCrossDomainIdentityManagement.SchemaIdentifier 
          { get; set; }
    }
  ````
  在擷取使用者目前狀態之要求的範例中，提供作為參數引數值的物件具有的屬性值如下所示： 
  
  * 識別碼："54D382A4-2050-4C03-94D1-E769F1D15682"
  * SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

4. 如果要更新某個參考屬性，Azure Active Directory 就會查詢服務，以判斷以該服務作為前端之身分識別存放區中參考屬性目前的值，是否已經與 Azure Active Directory 中該屬性的值相符。 對於使用者，以這種方式可查詢目前值的唯一屬性會是管理員屬性。 要判斷特定使用者物件的管理員屬性目前是否具有某個值之要求的範例如下： 
  ````
    GET ~/scim/Users?filter=id eq 54D382A4-2050-4C03-94D1-E769F1D15682 and manager eq 2819c223-7f76-453a-919d-413861904646&attributes=id HTTP/1.1
    Authorization: Bearer ...
  ````
  屬性查詢參數 "ID" 的值，表示如果滿足提供作為篩選查詢參數值的運算式的使用者物件存在，則服務應該以 "urn:ietf:params:scim:schemas:core:2.0:User" 或 "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" 資源回應，包僅括該資源 "ID" 屬性的值。  要求者知道 **ID** 屬性的值。 它包含在篩選查詢參數的值中；要求它的目的只是實際要求滿足篩選運算式作為任何這類物件是否存在指示之資源的最小表示。   

  如果是使用 Microsoft 所提供、用於實作 SCIM 服務的通用語言基礎結構程式庫建置服務，會將要求轉譯為對服務提供者的 Query 方法的呼叫。 提供物件的屬性值作為參數引數的值，如下所示： 
  
  * parameters.AlternateFilters.Count: 2
  * parameters.AlternateFilters.ElementAt(x).AttributePath: "ID"
  * parameters.AlternateFilters.ElementAt(x).ComparisonOperator: ComparisonOperator.Equals
  * parameters.AlternateFilter.ElementAt(x).ComparisonValue: "54D382A4-2050-4C03-94D1-E769F1D15682"
  * parameters.AlternateFilters.ElementAt(y).AttributePath: "manager"
  * parameters.AlternateFilters.ElementAt(y).ComparisonOperator: ComparisonOperator.Equals
  * parameters.AlternateFilter.ElementAt(y).ComparisonValue: "2819c223-7f76-453a-919d-413861904646"
  * parameters.RequestedAttributePaths.ElementAt(0): "ID"
  * parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

  在此處，索引 x 的值可能會是 0，而索引 y 的值可能是 1，或 x 值可能是 1 而 y 的值可能是 0，視篩選查詢參數運算式的順序而定。   

5. 以下是由 Azure Active Directory 對 SCIM 服務發出要求來更新使用者的範例： 
  ````
    PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...
    Content-type: application/scim+json
    {
      "schemas": 
      [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"],
      "Operations":
      [
        {
          "op":"Add",
          "path":"manager",
          "value":
            [
              {
                "$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646",
                "value":"2819c223-7f76-453a-919d-413861904646"}]}]}
  ````
  用於實作 SCIM 服務的 Microsoft 通用語言基礎結構程式庫，會將要求轉譯為對服務提供者的 Update 方法的呼叫。 以下是更新方法的簽章： 
  ````
    // System.Threading.Tasks.Tasks and 
    // System.Collections.Generic.IReadOnlyCollection<T>
    // are defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IPatch, 
    // Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
    // Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation, 
    // Microsoft.SystemForCrossDomainIdentityManagement.OperationName, 
    // Microsoft.SystemForCrossDomainIdentityManagement.IPath and 
    // Microsoft.SystemForCrossDomainIdentityManagement.OperationValue 
    // are all defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 

    System.Threading.Tasks.Task Update(
      Microsoft.SystemForCrossDomainIdentityManagement.IPatch patch, 
      string correlationIdentifier);

    public interface Microsoft.SystemForCrossDomainIdentityManagement.IPatch
    {
    Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase 
      PatchRequest 
        { get; set; }
    Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier 
      ResourceIdentifier 
        { get; set; }        
    }

    public class PatchRequest2: 
      Microsoft.SystemForCrossDomainIdentityManagement.PatchRequestBase
    {
    public System.Collections.Generic.IReadOnlyCollection
      <Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation> 
        Operations
        { get;}

    public void AddOperation(
      Microsoft.SystemForCrossDomainIdentityManagement.PatchOperation operation);
    }

    public class PatchOperation
    {
    public Microsoft.SystemForCrossDomainIdentityManagement.OperationName 
      Name
      { get; set; }

    public Microsoft.SystemForCrossDomainIdentityManagement.IPath 
      Path
      { get; set; }

    public System.Collections.Generic.IReadOnlyCollection
      <Microsoft.SystemForCrossDomainIdentityManagement.OperationValue> Value
      { get; }

    public void AddValue(
      Microsoft.SystemForCrossDomainIdentityManagement.OperationValue value);
    }

    public enum OperationName
    {
      Add,
      Remove,
      Replace
    }

    public interface IPath
    {
      string AttributePath { get; }
      System.Collections.Generic.IReadOnlyCollection<IFilter> SubAttributes { get; }
      Microsoft.SystemForCrossDomainIdentityManagement.IPath ValuePath { get; }
    }

    public class OperationValue
    {
      public string Reference
      { get; set; }

      public string Value
      { get; set; }
    }
  ````
    在更新使用者之要求的範例中，提供作為修補程式引數值的物件具有這些屬性值： 
  
  * ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
  * ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
  * (PatchRequest as PatchRequest2).Operations.Count: 1
  * (PatchRequest as PatchRequest2).Operations.ElementAt(0).OperationName: OperationName.Add
  * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Path.AttributePath: "manager"
  * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.Count: 1
  * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Reference: http://.../scim/Users/2819c223-7f76-453a-919d-413861904646
  * (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Value: 2819c223-7f76-453a-919d-413861904646

6. 若要將使用者從前端為 SCIM 服務的身分識別存放區中取消佈建，Azure AD 會傳送像以下的要求： 
  ````
    DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
    Authorization: Bearer ...
  ````
  如果是使用 Microsoft 所提供、用於實作 SCIM 服務的通用語言基礎結構程式庫建置服務，會將要求轉譯為對服務提供者的 Delete 方法的呼叫。   該方法具有此簽章： 
  ````
    // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
    // Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier, 
    // is defined in Microsoft.SystemForCrossDomainIdentityManagement.Protocol. 
    System.Threading.Tasks.Task Delete(
      Microsoft.SystemForCrossDomainIdentityManagement.IResourceIdentifier  
        resourceIdentifier, 
      string correlationIdentifier);
  ````
  提供作為 resourceIdentifier 引數值的物件，在要取消佈建使用者之要求的範例中，會具有這些屬性值： 
  
  * ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
  * ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

## <a name="group-provisioning-and-de-provisioning"></a>群組佈建和取消佈建
下圖顯示 Azure AD 會傳送至 SCIM 服務的訊息，以管理群組在其他身分識別存放區中的生命週期。  這些訊息與使用者的訊息不同，有下列三個方面： 

* 群組資源的結構描述會識別為 `http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/Group`。  
* 擷取群組的要求會規定將成員屬性從回應要求中提供的任何資源中排除。  
* 要求判斷參考屬性是否具有特定值，會是有關成員屬性的要求。  

![][5]
圖 6：群組佈建和取消佈建順序

## <a name="related-articles"></a>相關文章
* [自動化 SaaS 應用程式使用者佈建/解除佈建](user-provisioning.md)
* [自訂使用者佈建的屬性對應](customize-application-attributes.md)
* [撰寫屬性對應的運算式](functions-for-customizing-application-data.md)
* [適用於使用者佈建的範圍篩選器](define-conditional-rules-for-provisioning-user-accounts.md)
* [帳戶佈建通知](user-provisioning.md)
* [如何整合 SaaS 應用程式的教學課程清單](../saas-apps/tutorial-list.md)

<!--Image references-->
[0]: ./media/use-scim-to-provision-users-and-groups/scim-figure-1.png
[1]: ./media/use-scim-to-provision-users-and-groups/scim-figure-2a.png
[2]: ./media/use-scim-to-provision-users-and-groups/scim-figure-2b.png
[3]: ./media/use-scim-to-provision-users-and-groups/scim-figure-3.png
[4]: ./media/use-scim-to-provision-users-and-groups/scim-figure-4.png
[5]: ./media/use-scim-to-provision-users-and-groups/scim-figure-5.png
