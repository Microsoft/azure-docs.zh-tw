---
title: 在 Azure CDN 中管理 Web 內容的到期 | Microsoft Docs
description: 了解如何在 Azure CDN 中管理 Azure Web Apps/雲端服務、ASP.NET 或 IIS 內容的到期。
services: cdn
documentationcenter: .NET
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: bef53fcc-bb13-4002-9324-9edee9da8288
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/15/2018
ms.author: magattus
ms.openlocfilehash: d4ae0c4d5924fab8fcdaf1b4da5c8183a3a5fd0f
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092468"
---
# <a name="manage-expiration-of-web-content-in-azure-cdn"></a>在 Azure CDN 中管理 Web 內容的到期
> [!div class="op_single_selector"]
> * [Azure 網頁內容](cdn-manage-expiration-of-cloud-service-content.md)
> * [Azure Blob 儲存體](cdn-manage-expiration-of-blob-content.md)
> 

來自可公開存取的原始 Web 伺服器的檔案均可在 Azure 內容傳遞網路 (CDN) 中加以快取，直到其存留時間 (TTL) 結束。 TTL 是由來自原始伺服器之 HTTP 回應中的 `Cache-Control` 標頭所決定。 本文說明如何設定 Microsoft Azure App Service 之 Web Apps 功能、Azure Cloud Services、ASP.NET 應用程式、Internet Information Services (IIS) 網站的 `Cache-Control` 標頭，上述所有項目的設定方式均類似。 您可以使用組態檔，或以程式設計方式來設定 `Cache-Control` 標頭。 

您也可以藉由設定 [CDN 快取規則](cdn-caching-rules.md)，從 Azure 入口網站控制快取設定。 如果您建立一個或多個快取規則，並將其快取行為設定為 [覆寫] 或 [略過快取]，就會忽略本文所討論之原始提供的快取設定。 如需一般快取概念的相關資訊，請參閱[快取如何運作](cdn-how-caching-works.md)。

> [!TIP]
> 您可以選擇不替檔案設定 TTL。 在此情況下，除非您已在 Azure 入口網站中設定快取規則，否則 Azure CDN 會自動套用七天的預設 TTL。 此預設 TTL 僅會套用至一般 Web 傳遞最佳化。 針對大型檔案的最佳化，預設 TTL 為一天；針對媒體串流最佳化，預設 TTL 則為一年。
> 
> 針對 Azure CDN 如何加快對檔案和其他資源的存取速度，如需詳細資訊請參閱 [Azure 內容傳遞網路概觀](cdn-overview.md)。
> 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>使用 CDN 快取規則設定 Cache-Control 標頭
設定 Web 伺服器 `Cache-Control` 標頭的慣用方法為在 Azure 入口網站中使用快取規則。 如需 CDN 快取規則的詳細資訊，請參閱[使用快取規則控制 Azure CDN 快取行為](cdn-caching-rules.md)。

> [!NOTE] 
> 快取規則僅適用於「**來自 Verizon 的 Azure CDN 標準**」和「**來自 Akamai 的 Azure CDN 標準**」的設定檔。 針對「**來自 Verizon 的 Azure CDN 進階**」設定檔，您必須使用 [管理] 入口網站中的 [Azure CDN 規則引擎](cdn-rules-engine.md)來執行類似功能。

**瀏覽至 CDN 快取規則頁面**：

1. 在 Azure 入口網站中，選取 CDN 設定檔，然後選取 Web 伺服器的端點。

1. 在左窗格的 [設定] 下方，選取 [快取規則]。

   ![CDN 快取規則按鈕](./media/cdn-manage-expiration-of-cloud-service-content/cdn-caching-rules-btn.png)

   [快取規則] 頁面隨即出現。

   ![CDN 快取頁面](./media/cdn-manage-expiration-of-cloud-service-content/cdn-caching-page.png)


**若要使用全域快取規則設定 Web 伺服器的 Cache-Control 標頭：**

1. 在 [全域快取規則] 下方，將 [查詢字串快取行為] 設定為 [忽略查詢字串]，並將 [快取行為] 設定為 [覆寫]。
      
1. 在 [快取到期期間] 的 [秒鐘] 方塊中輸入 3600 或在 [小時] 方塊中輸入 1。 

   ![CDN 全域快取規則範例](./media/cdn-manage-expiration-of-cloud-service-content/cdn-global-caching-rules-example.png)

   這個全域快取規則會設定一小時的快取期間，並影響針對端點的所有要求。 它會覆寫由端點指定之原始伺服器所傳送的任何 `Cache-Control` 或 `Expires` HTTP 標頭。   

1. 選取 [ **儲存**]。

**使用自訂快取規則設定 Web 伺服器檔案的 Cache-Control 標頭：**

1. 在 [自訂快取規則] 下，建立兩個比對條件：

     a. 在第一個比對條件，將 [比對條件] 設為 [路徑]，並在 [符合值] 輸入 `/webfolder1/*`。 將 [快取行為] 設定為 [覆寫]，並在 [時數] 方塊中輸入 4。

     b. 在第二個比對條件，將 [比對條件] 設為 [路徑]，並在 [符合值] 輸入 `/webfolder1/file1.txt`。 將 [快取行為] 設定為 [覆寫]，並在 [時數] 方塊中輸入 2。

    ![CDN 自訂快取規則範例](./media/cdn-manage-expiration-of-cloud-service-content/cdn-custom-caching-rules-example.png)

    第一個自訂快取規則會替您的端點指定之原始伺服器上 `/webfolder1` 資料夾中的所有檔案，設定四個小時的快取期間。 第二個規則只會針對 `file1.txt` 檔案覆寫第一個規則，並為其設定兩個小時的快取期間。

1. 選取 [ **儲存**]。


## <a name="setting-cache-control-headers-by-using-configuration-files"></a>使用組態檔設定 Cache-Control 標頭
針對影像和樣式表之類的靜態內容，您可以藉由修改 Web 應用程式的 **applicationHost.config** 或 **web.config** 組態檔來控制更新頻率。 若要為內容設定 `Cache-Control` 標頭，請使用檔案中的 `<system.webServer>/<staticContent>/<clientCache>` 元素。

### <a name="using-applicationhostconfig-files"></a>使用 ApplicationHost.config 檔案
**ApplicationHost.config** 檔案是 IIS 組態系統的根檔案。 **ApplicationHost.config** 檔案中的組態設定會影響網站上的所有應用程式，但該組態設定會由 Web 應用程式中現有的任何 **Web.config** 檔案設定覆寫。

### <a name="using-webconfig-files"></a>使用 Web.config 檔案
您可以使用 **Web.config** 檔案，自訂整個 Web 應用程式或 Web 應用程式上特定目錄的行為。 通常來說，您的 Web 應用程式根資料夾中至少會有一個 **Web.config** 檔案。 針對特定資料夾中的每個 **Web.config** 檔案，組態設定會影響資料夾及子資料夾中的所有項目，除非這些項目在子資料夾層級就由另一 **Web.config** 檔案覆寫。 

例如，您可以在 Web 應用程式中根資料夾的 **Web.config** 檔案中設定 `<clientCache>` 元素。 您也可以在子資料夾中新增具有更多變數內容的 **Web.config** 檔案 (例如 `\frequent`) 並設定其 `<clientCache>` 元素以快取子資料夾的內容達六小時。 最終結果是整個網站上的內容會進行三天的快取，除了 `\frequent` 目錄中的所有內容僅會進行六小時的快取。  

下列 XML 設定檔範例會示範如何設定 `<clientCache>` 元素，以指定最多三天期間：  

```xml
<configuration>
    <system.webServer>
        <staticContent>
            <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="3.00:00:00" />
        </staticContent>
    </system.webServer>
</configuration>
```

若要使用 **cacheControlMaxAge** 屬性，您必須將 **cacheControlMode** 屬性的值設為 `UseMaxAge`。 此設定會產生要新增至回應的 HTTP 標頭及指示詞，`Cache-Control: max-age=<nnn>`。 **cacheControlMaxAge** 屬性的時間範圍值格式為 `<days>.<hours>:<min>:<sec>`。 此值會轉換為秒，且會當做 `Cache-Control` `max-age` 指示詞使用。 如需 `<clientCache>` 元素的詳細資訊，請參閱[用戶端快取<clientCache>](http://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)。  

## <a name="setting-cache-control-headers-programmatically"></a>以程式設計方式設定 Cache-Control 標頭
針對 ASP.NET 應用程式，設定 .NET API 的 **HttpResponse.Cache** 屬性即可透過程式設計方式控制 CDN 快取行為。 如需 **HttpResponse.Cache** 屬性的資訊，請參閱 [HttpResponse.Cache 屬性](http://msdn.microsoft.com/library/system.web.httpresponse.cache.aspx)和 [HttpCachePolicy 類別](http://msdn.microsoft.com/library/system.web.httpcachepolicy.aspx)。  

若要以程式設計方式快取 ASP.NET 中的應用程式內容，遵循下列步驟：
   1. 確認內容已標示為可快取(將 `HttpCacheability` 設定為 `Public`)。 
   1. 呼叫以下其中一個 `HttpCachePolicy` 方法來設定快取驗證程式：
      - 呼叫 `SetLastModified` 以設定 `Last-Modified` 標頭的時間戳記。
      - 呼叫 `SetETag` 以設定 `ETag` 標頭值。
   1. 您也可以選擇性地呼叫 `SetExpires`，設定 `Expires` 標頭值，以指定快取到期時間。 否則，預設快取會套用本文件先前所述的啟發學習法。

例如，若要快取一個小時的內容，請加入下列 C# 程式碼：  

```csharp
// Set the caching parameters.
Response.Cache.SetExpires(DateTime.Now.AddHours(1));
Response.Cache.SetCacheability(HttpCacheability.Public);
Response.Cache.SetLastModified(DateTime.Now);
```

## <a name="testing-the-cache-control-header"></a>測試 Cache-Control 標頭
您可以輕鬆地驗證網頁內容的 TTL 設定。 使用瀏覽器的[開發人員工具](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/)，測試網頁內容是否包含 `Cache-Control` 回應標頭。 您也可以使用 **wget**、[Postman](https://www.getpostman.com/) 或 [Fiddler](http://www.telerik.com/fiddler) 之類的工具來檢查回應標頭。

## <a name="next-steps"></a>後續步驟
* [深入了解 **clientCache** 項目](http://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)
* [閱讀 **HttpResponse.Cache** 屬性的文件](http://msdn.microsoft.com/library/system.web.httpresponse.cache.aspx) 
* [閱讀 **HttpCachePolicy 類別**的文件](http://msdn.microsoft.com/library/system.web.httpcachepolicy.aspx)  
* [深入了解快取概念](cdn-how-caching-works.md)
