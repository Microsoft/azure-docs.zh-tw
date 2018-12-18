---
title: 從行動服務移轉到應用程式服務行動應用程式
description: 了解如何輕鬆地將您的行動服務應用程式移轉至應用程式服務行動應用程式
services: app-service\mobile
documentationcenter: ''
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 07507ea2-690f-4f79-8776-3375e2adeb9e
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile
ms.devlang: na
ms.topic: article
ms.date: 10/03/2016
ms.author: crdun
ms.openlocfilehash: 5001704f47af0c7b07744f1dceb7aa58bdb6448c
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32158863"
---
# <a name="article-top"></a>將您現有的 Azure 行動服務移轉至 Azure App Service
透過 [Azure App Service 的公開上市版]，Azure 行動服務網站將可輕易地就地移轉，以使用 Azure App Service 的所有功能。  本文件說明將您的網站從 Azure 行動服務移轉至 Azure App Service 時的情形。

## <a name="what-does-migration-do"></a>移轉對您的網站有何作用
移轉 Azure 行動服務，會使您的行動服務變成 [Azure App Service] 應用程式，而不會對程式碼造成任何影響。  您通知中樞、SQL 資料連接、驗證設定、排定的作業和網域名稱都會保持不變。  使用您的 Azure 行動服務的行動用戶端仍可正常運作。  移轉會在您的服務轉換為 Azure App Service 後加以重新啟動。

[!INCLUDE [app-service-mobile-migrate-vs-upgrade](../../includes/app-service-mobile-migrate-vs-upgrade.md)]

## <a name="why-migrate"></a>為何您應移轉網站
Microsoft 建議您移轉 Azure 行動服務，以使用 Azure App Service 的各項功能，其中包括：

* 新的主機功能，包括 [WebJobs] 和[自訂網域名稱]。
* 使用 [Application Insights] 進行監視和疑難排解作業。
* 內建的 DevOps 工具，包括[預備位置]、回復和生產環境測試。
* [自動調整]、負載平衡，以及[效能監視]。

若想進一步了解 Azure App Service 的優點，請參閱[比較行動服務與App Service] 主題。

## <a name="before-you-begin"></a>開始之前
網站開始任何主要工作之前，您應該先備份您的行動服務指令碼和 SQL Database。

## <a name="migrating-site"></a>移轉您的網站
移轉程序會移轉單一 Azure 區域內的所有網站。

若要移轉您的網站：

1. 登入 [Azure 傳統入口網站]。
2. 選取區域中您想要移轉的行動服務。
3. 按一下 [移轉至 App Service]  按鈕。

   ![移轉按鈕][0]
4. 閱讀 [移轉至 App Service] 對話方塊。
5. 在提供的方塊中輸入您的行動服務名稱。  例如，如果您的網域名稱是 contoso.azure-mobile.net，請在提供的方塊中輸入 *contoso*。
6. 按一下勾號按鈕。

在活動監視器中監視移轉的狀態。 您的網站會在 Azure 傳統入口網站中列為*移轉中*。

  ![移轉活動監視器][1]

每個移轉的行動服務在每次移轉時可能需要 3 到 15 分鐘不等。  移轉期間仍可使用您的網站。
您的網站會在移轉程序結束時重新啟動。  在重新啟動程序期間無法使用網站，此狀況可能會持續幾秒鐘。

## <a name="finalizing-migration"></a>完成移轉
規劃在移轉程序結束後從行動用戶端測試您的網站。  請確定您可以執行所有的一般用戶端動作，而不會變更行動用戶端。  

### <a name="update-app-service-tier"></a>選取適當的 App Service 定價層
在移轉至 Azure App Service 之後，您在價格方面將會有更多彈性。

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 按一下 [設定] 功能表中的 [App Service 方案] 。
5. 按一下 [定價層] 圖格。
6. 按一下您的需求適用的圖格，然後按一下 [選取]。  您可能需要按一下 [檢視全部]，才能查看可用的定價層。

建議您以下列各層做為起點：

| 行動服務定價層 | App Service 定價層 |
|:--- |:--- |
| 免費 |F1 免費 |
| 基本 |B1 基本 |
| 標準 |S1 標準 |

您有相當大的彈性可為應用程式選擇適當的定價層。  請參閱 [App Service 價格] ，以充分了解新的 App Service 的價格。

> [!TIP]
> App Service 標準層包含您可能想要使用之多種功能的存取權，包括[預備位置]、自動備份和自動調整。  您可以在相關位置查看新功能。
>
>

### <a name="review-migration-scheduler-jobs"></a>檢閱已移轉的排程器作業
排程器作業在移轉後約 30 分鐘內將不會顯示。  排程的作業會持續在背景中執行。
若要在排程的作業再度顯現後加以檢視︰

1. 登入 [Azure 入口網站]。
2. 選取 [瀏覽 >]、在 [篩選] 方塊中輸入 **Schedule**，然後選取 [排程器集合]。

移轉後可用的排程器作業數量將有所限制。  檢閱您的使用量和 [Azure 排程器方案]。

### <a name="configure-cors"></a>視需要設定 CORS
跨原始資源共用是一項可讓網站存取不同網域之 Web API 的技術。  如果您使用的 Azure 行動服務具有相關聯的網站，您就必須在移轉中設定 CORS。  如果您從行動裝置以獨佔方式存取 Azure 行動服務，則在絕大多數的情況下都不需要設定 CORS。

已移轉的 CORS 設定可做為 **MS_CrossDomainWhitelist** 應用程式設定。  若要將您的網站移轉至 App Service CORS 工具：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 按一下 [API] 功能表中的 [CORS]。
5. 在提供的方塊中輸入任何 [允許的原點]，每輸入一個就按一下 Enter 鍵。
6. 如果 [允許的原點] 清單正確無誤，請按一下 [儲存] 按鈕。

> [!TIP]
> 使用 Azure App Service 的優點之一，是您可以在相同網站上執行您的網站和行動服務。  如需詳細資訊，請參閱[後續步驟](#next-steps)一節。
>
>

### <a name="download-publish-profile"></a>下載新的發行設定檔
網站的發行設定檔移轉至 Azure App Service 後，會進行變更。  如果您想要從 Visual Studio 發佈您的網站，您需要新的發行設定檔。  若要下載新的發行設定檔：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 按一下 [取得發行設定檔]。

PublishSettings 檔案會下載至您的電腦。  此檔案通常名為 *sitename*.PublishSettings。  將發佈設定匯入現有的專案中：

1. 開啟 Visual Studio 和您的 Azure 行動服務專案。
2. 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後選取 [發佈...]。
3. 按一下 [匯入]。
4. 按一下 [瀏覽]，然後選取已下載的發佈設定檔案。  按一下 [檔案] &gt; [新增] &gt; [專案] 
5. 按一下 [驗證連接]，確保發佈設定可運作。
6. 選擇 [發佈] 來發佈您的網站。

## <a name="working-with-your-site"></a>在移轉後使用您的網站
移轉之後，在 [Azure 入口網站]中開始使用新的 App Service。  以下是您過去在 [Azure 傳統入口網站]中執行之特定作業的某些注意事項，及其 App Service 的對等項目。

### <a name="publishing-your-site"></a>下載和發佈您已移轉的網站
您的網站可透過 git 或 ftp 來使用，而且可透過各種不同的機制重新發佈，包括 WebDeploy、TFS、Mercurial、GitHub 及 FTP。  部署認證會隨著網站的其餘部分移轉。  如果您未設定部署認證，或您不記得，您可以將其重設：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 按一下 [發佈] 功能表中的 [部署認證]。
5. 在提供的方塊中輸入新的部署認證，然後按一下 [儲存] 按鈕。

您可以使用這些認證透過 git 複製網站，或從 GitHub、TFS 或 Mercurial 設定自動化部署。  如需詳細資訊，請參閱 [Azure App Service 部署文件]。

### <a name="appsettings"></a>應用程式設定
已移轉的行動服務大部分的設定都可透過 [應用程式設定] 來使用。  您可以從 [Azure 入口網站]取得應用程式設定清單。
若要檢視或變更您的應用程式設定：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 按一下 [一般] 功能表中的 [應用程式設定]  。
5. 捲動至 [應用程式設定] 區段，並尋找您的應用程式設定。
6. 按一下應用程式設定的值，以編輯該值。  按一下 [儲存] 以儲存該值。

您可以同時更新多個應用程式設定。

> [!TIP]
> 有兩個 [應用程式設定] 具有相同的值。  例如，您可能會看到 ApplicationKey 和 MS\_ApplicationKey。  同時更新這兩個應用程式設定。
>
>

### <a name="authentication"></a>驗證
所有的驗證設定都可做為已移轉之網站中的 [應用程式設定]。  若要更新您的驗證設定，您必須變更適當的應用程式設定。  下表列出您的驗證提供者所適用的應用程式設定：

| 提供者 | 用戶端識別碼 | 用戶端密碼 | 其他設定 |
|:--- |:--- |:--- |:--- |
| Microsoft 帳戶 |**MS\_MicrosoftClientID** |**MS\_MicrosoftClientSecret** |**MS\_MicrosoftPackageSID** |
| Facebook |**MS\_FacebookAppID** |**MS\_FacebookAppSecret** | |
| Twitter |**MS\_TwitterConsumerKey** |**MS\_TwitterConsumerSecret** | |
| Google |**MS\_GoogleClientID** |**MS\_GoogleClientSecret** | |
| Azure AD |**MS\_AadClientID** | |**MS\_AadTenants** |

注意：**MS\_AadTenants** 會儲存為租用戶網域 ([行動服務入口網站] 中的 [允許的租用戶] 欄位) 的逗號分隔清單。

> [!WARNING]
> **請不要使用 [設定] 功能表中的驗證機制**
>
> Azure App Service 分別在 [驗證/授權設定] 功能表下提供「無程式碼」驗證和授權系統，以及在 [設定] 功能表下提供已被取代的 [行動驗證] 選項。  這些選項與已移轉的 Azure 行動服務不相容。  您可以 [升級您的網站](app-service-mobile-net-upgrading-from-mobile-services.md) ，以利用 Azure App Service 驗證功能。
>
>

### <a name="easytables"></a>資料
行動服務中的 [資料] 索引標籤在 Azure 入口網站中已替換為 [簡單資料表]。  若要存取簡單資料表：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 按一下 [行動] 功能表中的 [簡單資料表]。

您可以按一下 [新增] 按鈕以新增資料表，或按一下資料表名稱以存取現有的資料表。  在此刀鋒視窗中可以執行各種作業，包括：

* 變更資料表權限
* 編輯作業指令碼
* 管理資料表結構描述
* 刪除資料表
* 清除資料表內容
* 刪除資料表的特定資料列

### <a name="easyapis"></a>API
行動服務中的 [API] 索引標籤在 Azure 入口網站中已替換為 [簡單 API]。  若要存取簡單 API：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 按一下 [行動] 功能表中的 [簡單 API]。

移轉的 API 已經列在刀鋒視窗中。  您也可以在此刀鋒視窗中新增 API。  若要管理特定 API，請按一下該 API。
從新的刀鋒視窗中，您可以調整權限，以及編輯 API 的指令碼。

### <a name="on-demand-jobs"></a>排程器作業
所有的排程器作業都可透過 [排程器作業集合] 區段來使用。  若要存取您的排程器作業：

1. 登入 [Azure 入口網站]。
2. 選取 [瀏覽 >]、在 [篩選] 方塊中輸入 **Schedule**，然後選取 [排程器集合]。
3. 選取網站的作業集合。  它的名稱為 *sitename*-Jobs。
4. 按一下 [設定] 。
5. 按一下 [管理] 下的 [排程器作業]。

排程的工作會以您在移轉之前所指定的頻率列出。  隨選作業便已停用。  若要執行隨選作業：

1. 選取您想要執行的作業。
2. 如有必要，請按一下 [啟用] 以啟用作業。
3. 按一下 [設定]，然後按一下 [排程]。
4. 選取 [一次] 的週期性，然後按一下 [儲存]

您的隨需作業會位於 `App_Data/config/scripts/scheduler post-migration`。  建議您將所有隨選作業轉換為 [WebJobs] 或 [Functions]。  將新的排程器作業撰寫成 [WebJobs] 或 [Functions]。

### <a name="notification-hubs"></a>通知中樞
行動服務會使用通知中樞進行推播通知作業。  在移轉之後，會使用下列應用程式設定將通知中樞連結至您的行動服務：

| 應用程式設定 | 說明 |
|:--- |:--- |
| **MS\_PushEntityNamespace** |通知中樞命名空間 |
| **MS\_NotificationHubName** |通知中樞名稱 |
| **MS\_NotificationHubConnectionString** |通知中樞連接字串 |
| **MS\_NamespaceName** |MS_PushEntityNamespace 的別名 |

您的通知中樞會透過 [Azure 入口網站]管理。  請記下通知中樞名稱 (您可以使用 [應用程式設定] 找到此項目)：

1. 登入 [Azure 入口網站]。
2. 選取 [瀏覽>]，然後選取 **通知中樞**
3. 按一下與行動服務相關聯的通知中樞名稱。

> [!NOTE]
> 如果您的通知中樞是「混合」類型，則不會顯示。  「混合」類型的通知中樞會同時使用「通知中樞」和舊版的「服務匯流排」功能。  [轉換混合式命名空間]，再繼續作業。  轉換完成後，您的通知中樞會出現在 [Azure 入口網站]中。
>
>

如需詳細資訊，請檢閱 [通知中樞] 文件。

> [!TIP]
> [Azure 入口網站]中的通知中樞管理功能仍處於預覽階段。  [Azure 傳統入口網站] 仍可用來管理您所有的通知中樞。
>
>

### <a name="legacy-push"></a>舊版推播設定
如果您在通知中樞引入前，即已設定行動服務的推播，您使用的就是「舊版推播」。  如果您使用推播，組態中卻沒有列出通知中樞，您很可能使用的是「舊版推播」。  這項功能會和所有其他功能一起移轉。  不過，建議您完成移轉後盡快升級至通知中樞。

在過渡時期，所有舊版推播設定 (APNS 憑證除外) 都可以在應用程式設定中取得。  將檔案系統上適當的檔案替換掉，以更新 APNS 憑證。

### <a name="app-settings"></a>其他應用程式設定
以下是從您的行動服務移轉的其他應用程式設定，可從 [設定] > [應用程式設定]：

| 應用程式設定 | 說明 |
|:--- |:--- |
| **MS\_MobileServiceName** |您的應用程式名稱 |
| **MS\_MobileServiceDomainSuffix** |網域前置詞。 亦即 azure-mobile.net |
| **MS\_ApplicationKey** |您的應用程式金鑰 |
| **MS\_MasterKey** |您的應用程式主要金鑰 |

應用程式金鑰和主要金鑰會與原始行動服務中的應用程式金鑰完全相同。  特別是，行動用戶端會傳送應用程式金鑰，以驗證他們對行動 API 的使用。

### <a name="cliequivalents"></a>命令列對等項目
您將無法再使用 *azure mobile* 命令來管理您的 Azure 行動服務網站。  有許多函式已替換為 *azure site* 命令。  使用下表來尋找常用命令的對等項目：

| *Azure Mobile* 命令 | 對等的 *Azure site* 命令 |
|:--- |:--- |
| mobile locations |site location list |
| mobile list |site list |
| mobile show *name* |site show *name* |
| mobile restart *name* |site restart *name* |
| mobile redeploy *name* |site deployment redeploy *commitId* *name* |
| mobile key set *name* *type* *value* |site appsetting delete *key* *name* <br/> site appsetting add *key*=*value* *name* |
| mobile config list *name* |site appsetting list *name* |
| mobile config get *name* *key* |site appsetting show *key* *name* |
| mobile config set *name* *key* |site appsetting delete *key* *name* <br/> site appsetting add *key*=*value* *name* |
| mobile domain list *name* |site domain list *name* |
| mobile domain add *name* *domain* |site domain add *domain* *name* |
| mobile domain delete *name* |site domain delete *domain* *name* |
| mobile scale show *name* |site show *name* |
| mobile scale change *name* |site scale mode *mode* *name* <br /> site scale instances *instances* *name* |
| mobile appsetting list *name* |site appsetting list *name* |
| mobile appsetting add *name* *key* *value* |site appsetting add *key*=*value* *name* |
| mobile appsetting delete *name* *key* |site appsetting delete *key* *name* |
| mobile appsetting show *name* *key* |site appsetting delete *key* *name* |

藉由更新適當的應用程式設定，可更新驗證或推播通知設定。
請編輯檔案，並透過 ftp 或 git 發佈您的網站。

### <a name="diagnostics"></a>診斷和記錄
Azure App Service 通常會停用 [診斷記錄]。  若要啟用診斷記錄：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。
4. 選取 [功能] 功能表下的 [診斷記錄]  。
5. 對下列記錄檔按一下 [開啟]：[應用程式記錄 (檔案系統)]、[詳細錯誤訊息] 和 [失敗要求的追蹤]
6. 針對 Web 伺服器記錄，按一下 [檔案系統] 
7. 按一下 [儲存] 

若要檢視記錄檔：

1. 登入 [Azure 入口網站]。
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下已移轉的行動應用程式名稱。
3. 按一下 [工具] 按鈕
4. 選取 [觀察] 功能表下的 [記錄資料流]  。

產生的記錄檔會顯示在視窗中。  您也可以下載的記錄檔，以便後續使用您的部署認證加以分析。 如需詳細資訊，請參閱[記錄]文件。

## <a name="known-issues"></a>已知問題
### <a name="deleting-a-migrated-mobile-app-clone-causes-a-site-outage"></a>刪除移轉的行動應用程式複製會導致網站服務中斷
如果您使用 Azure PowerShell 複製移轉的行動服務，然後又刪除此複製，則會移除生產服務的 DNS 項目。  無法再從網際網路存取您的網站。  

解決方法︰如果您想要複製網站，請透過入口網站進行。

### <a name="changing-webconfig-does-not-work"></a>變更 Web.config 並未發生作用
如果您有 ASP.NET 網站，不會套用 `Web.config` 檔案的變更。  Azure App Service 會在啟動期間建置適合的 `Web.config` 檔案，以支援行動服務執行階段。  您可以使用 XML 轉換檔案來覆寫特定設定 (例如自訂標頭)。  建立名稱為 `applicationHost.xdt` 的檔案 - 這個檔案必須在 Azure 服務上的 `D:\home\site` 目錄中結束。  透過自訂部署指令碼或直接使用 Kudu 上傳 `applicationHost.xdt` 檔案。  下圖顯示範例文件：

```
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.webServer>
    <httpProtocol>
      <customHeaders>
        <add name="X-Frame-Options" value="DENY" xdt:Transform="Replace" />
        <remove name="X-Powered-By" xdt:Transform="Insert" />
      </customHeaders>
    </httpProtocol>
    <security>
      <requestFiltering removeServerHeader="true" xdt:Transform="SetAttributes(removeServerHeader)" />
    </security>
  </system.webServer>
</configuration>
```

如需詳細資訊，請參閱 GitHub 上的 [XDT 轉換範例]文件。

### <a name="migrated-mobile-services-cannot-be-added-to-traffic-manager"></a>移轉的行動服務無法新增至流量管理員
當您建立流量管理員設定檔時，無法直接選擇設定檔的移轉行動服務。  使用「外部端點」。  外部端點只能透過 PowerShell 來新增。  如需詳細資訊，請參閱[流量管理員教學課程](https://azure.microsoft.com/blog/azure-traffic-manager-external-endpoints-and-weighted-round-robin-via-powershell/)。

## <a name="next-steps"></a>後續步驟
您的應用程式現在已移轉至 App Service，有更多功能可供您使用：

* 部署 [預備位置] 可讓您預備網站的變更，並執行 A/B 測試。
* [WebJobs] 可取代隨選排定作業。
* 將網站連結至 GitHub、TFS 或 Mercurial，即可[連續部署]網站。
* 您可以使用 [Application Insights] 監視您的網站。
* 以相同的程式碼為網站和行動 API 提供服務。

### <a name="upgrading-your-site"></a>將您的行動服務網站升級至 Azure Mobile Apps SDK
* 對於以 Node.js 為基礎的伺服器專案，新的 [Mobile Apps Node.js SDK] 提供多項新功能。 例如，您現在可以執行本機開發和偵錯、使用 0.10 以上的任何 Node.js 版本，以及使用任何 Express.js 中介軟體自訂。
* 對於以 .NET 為基礎的伺服器專案，新的 [Mobile Apps SDK NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/)對 NuGet 相依性有更多彈性。  這些套件支援新的 App Service 驗證，並以任何 ASP.NET 專案撰寫。 若要深入了解升級，請參閱 [將您現有的 .NET 行動服務升級為 App Service](app-service-mobile-net-upgrading-from-mobile-services.md)。

<!-- Images -->
[0]: ./media/app-service-mobile-migrating-from-mobile-services/migrate-to-app-service-button.PNG
[1]: ./media/app-service-mobile-migrating-from-mobile-services/migration-activity-monitor.png
[2]: ./media/app-service-mobile-migrating-from-mobile-services/triggering-job-with-postman.png

<!-- Links -->
[App Service 價格]: https://azure.microsoft.com/pricing/details/app-service/
[Application Insights]: ../application-insights/app-insights-overview.md
[自動調整]: ../app-service/web-sites-scale.md
[Azure App Service]: ../app-service/app-service-web-overview.md
[Azure 傳統入口網站]: https://manage.windowsazure.com
[Azure 入口網站]: https://portal.azure.com
[Azure Region]: https://azure.microsoft.com/regions/
[Azure 排程器方案]: ../scheduler/scheduler-plans-billing.md
[連續部署]: ../app-service/app-service-continuous-deployment.md
[轉換混合式命名空間]: https://azure.microsoft.com/blog/updates-from-notification-hubs-independent-nuget-installation-model-pmt-and-more/
[curl]: http://curl.haxx.se/
[自訂網域名稱]: ../app-service/app-service-web-tutorial-custom-domain.md
[Fiddler]: http://www.telerik.com/fiddler
[Azure App Service 的公開上市版]: https://azure.microsoft.com/blog/announcing-general-availability-of-app-service-mobile-apps/
[Hybrid Connections]: ../app-service/app-service-hybrid-connections.md
[記錄]: ../app-service/web-sites-enable-diagnostic-log.md
[Mobile Apps Node.js SDK]: https://github.com/azure/azure-mobile-apps-node
[比較行動服務與App Service]: app-service-mobile-value-prop-migration-from-mobile-services.md
[通知中樞]: ../notification-hubs/notification-hubs-push-notification-overview.md
[效能監視]: ../app-service/web-sites-monitor.md
[Postman]: http://www.getpostman.com/
[預備位置]: ../app-service/web-sites-staged-publishing.md
[VNet]: ../app-service/web-sites-integrate-with-vnet.md
[XDT 轉換範例]: https://github.com/projectkudu/kudu/wiki/Xdt-transform-samples
[Functions]: ../azure-functions/functions-overview.md
