---
title: 開始在 Java 中使用 Azure 搜尋服務 | Microsoft Docs
description: 如何使用 Java 做為程式設計語言，在 Azure 上建置雲端託管搜尋應用程式。
services: search
author: jj09
manager: jlembicz
ms.service: search
ms.topic: conceptual
ms.date: 08/26/2018
ms.author: jjed
ms.openlocfilehash: 6fed5a8afe6030038752d540bfabbc446c57db2c
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43124146"
---
# <a name="get-started-with-azure-search-in-java"></a>開始在 Java 中使用 Azure 搜尋服務
> [!div class="op_single_selector"]
> * [入口網站](search-get-started-portal.md)
> * [.NET](search-howto-dotnet-sdk.md)
> 
> 

瞭解如何建置使用 Azure 搜尋服務提供搜尋體驗的自訂 Java 搜尋應用程式。 本教學課程利用 [Azure 搜尋服務 REST API](https://msdn.microsoft.com/library/dn798935.aspx) 來建構在此練習中所使用的物件和作業。

若要執行此範例，必須要有 Azure 搜尋服務，您可以在 [Azure 入口網站](https://portal.azure.com)註冊此服務。 如需逐步指示，請參閱 [在入口網站中建立 Azure 搜尋服務](search-create-service-portal.md) 。

我們使用了以下軟體建置及測試此範例：

* [Eclipse IDE for Java EE Developers](https://www.eclipse.org/downloads/packages/release/photon/r/eclipse-ide-java-ee-developers)。 請務必下載 EE 版本。 其中一個驗證步驟所需的功能只有在此版本中才能找到。
* [JDK 8u181](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [Apache Tomcat 8.5.33](https://tomcat.apache.org/download-80.cgi#8.5.33)

## <a name="about-the-data"></a>關於資料
此範例應用程式使用的 [美國地理服務中心 (USGS)](http://geonames.usgs.gov/domestic/download_data.htm)資料已依據羅德島州進行篩選，藉此減少資料集的大小。 我們將使用此資料建置可傳回地標建築物 (例如醫院和學校) 及地理特徵 (例如河流、湖泊和山峰) 的搜尋應用程式。

在此應用程式中， **SearchServlet.java** 程式會使用 [索引子](https://msdn.microsoft.com/library/azure/dn798918.aspx) 建構來建置及載入索引，以從公用 Azure SQL Database 擷取篩選過的 USGS 資料集。 程式碼中提供線上資料來源的預先定義認證和連接資訊。 關於資料存取，不需要進一步設定。

> [!NOTE]
> 我們在此資料集套用了一個篩選，以維持不超過免費版定價層的 10,000 個文件的數量上限。 如果使用標準版定價層，則不套用此限制，可以修改此程式碼以使用更大的資料集。 如需各個定價層的容量詳細資料，請參閱 [限制和條件約束](search-limits-quotas-capacity.md)。
> 
> 

## <a name="about-the-program-files"></a>關於程式檔
以下清單說明與此範例相關的檔案。

* Search.jsp：提供使用者介面
* SearchServlet.java：提供方法 (類似 MVC 中的控制器)
* SearchServiceClient.java：處理 HTTP 要求
* SearchServiceHelper.java：提供靜態方法的協助程式類別
* Document.java：提供資料模型
* config.properties：設定搜尋服務 URL 和 API 金鑰
* pom.xml：Maven 相依性

<a id="sub-2"></a>

## <a name="find-the-service-name-and-api-key-of-your-azure-search-service"></a>尋找 Azure 搜尋服務的服務名稱和 API 金鑰
所有對 Azure 搜尋服務的 REST API 呼叫都會要求您提供服務 URL 和 API 金鑰。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在導向列中，按一下 [搜尋服務]  列出為您的訂用帳戶佈建的所有 Azure 搜尋服務。
3. 選取您要使用的服務。
4. 您會在服務儀表板上看到基本資訊磚，以及存取系統管理金鑰的鑰匙圖示。
   
      ![][3]
5. 複製服務 URL 和系統管理金鑰， 稍後會需要將它們加到 **config.properties** 檔案中。

## <a name="download-the-sample-files"></a>下載範例檔案
1. 移至 GitHub 上的 [search-java-indexer-demo](https://github.com/Azure-Samples/search-java-indexer-demo)。
2. 按一下 [下載 ZIP] ，將 .zip 檔案儲存至磁碟，然後解壓縮其中所含的所有檔案。 可以考慮將檔案解壓縮到 Java 工作區，以便之後可以輕鬆找到專案。
3. 範例檔案為唯讀。 請以滑鼠右鍵按一下資料夾內容，然後清除唯讀屬性。

所有後續的檔案修改及執行陳述式都會用到此資料夾中的檔案。  

## <a name="import-project"></a>匯入專案
1. 在 Eclipse 中，選擇 [檔案] > [匯入] > [一般] > [將現有專案匯入工作區]。
   
    ![][4]
2. 在 [Select root directory (選取根目錄)] 中，瀏覽至含有範例檔案的資料夾。 請選取含有 .project 資料夾的資料夾。 此專案應會在 [Projects (專案)]  清單中顯示為選取的項目。
   
    ![][12]
3. 按一下 [完成] 。
4. 使用 **專案總管** 檢視及編輯檔案。 如果還沒開啟，請按一下 [視窗] > [顯示檢視] > [專案總管]，或使用捷徑來開啟檔案。

## <a name="configure-the-service-url-and-api-key"></a>設定服務 URL 和 API 金鑰
1. 在**專案總管**中按兩下 **config.properties** 以編輯含有伺服器名稱和 API 金鑰的組態設定。
2. 請參閱本文中稍早的步驟，其中提及如何在 [Azure 入口網站](https://portal.azure.com)中找到服務 URL 和 API 金鑰，藉此取得您現在要輸入 **config.properties** 中的值。
3. 在 **config.properties**中，以您服務的 API 金鑰取代 "Api Key"。 接著，在同一個檔案中以服務名稱 (URL http://servicename.search.windows.net) 的第一個部分) 取代 "service name"。
   
    ![][5]

## <a name="configure-the-project-build-and-runtime-environments"></a>設定專案、組建和執行階段環境。
1. 在 Eclipse 的 [專案總管] 中，以滑鼠右鍵按一下專案 > [屬性] > [專案 Facet]。
2. 選取 [Dynamic Web Module (動態網路模組)]、[Java] 及 [JavaScript]。
   
    ![][6]
3. 按一下 [套用] 。
4. 選取 [視窗] > [喜好設定] > [伺服器] > [執行階段環境] > [新增..]。
5. 展開 [Apache] 並選取您先前安裝的 Apache Tomcat 伺服器版本。 我們的系統安裝了第 8 版。
   
    ![][7]
6. 在下一頁指定 Tomcat 的安裝目錄。 在 Windows 電腦中，這通常為 C:\Program Files\Apache Software Foundation\Tomcat *版本*。
7. 按一下 [完成] 。
8. 選取 [視窗] > [喜好設定] > [Java] > [已安裝的 JRE] > [新增]。
9. 在 [Add JRE (新增 JRE)] 中，選取 [Standard VM (標準 VM)]。
10. 按 [下一步] 。
11. 在 [JRE Definition (JRE 定義)] 的 [JRE home (JRE 主資料夾)] 中按一下 [Directory (目錄)] 。
12. 瀏覽至 [程式檔案] > [Java]，然後選取您先前安裝的 JDK。 請務必選取 JDK 做為 JRE。
13. 在 [Installed JREs (安裝的 JRE)] 中選擇 **JDK**。 您的設定應該類似以下的螢幕擷取畫面。
    
    ![][9]
14. 您可以選擇性地選取 [視窗] > [網頁瀏覽器] > [Internet Explorer] 以在外部瀏覽器視窗開啟應用程式。 使用外部瀏覽器可給您更好的 Web 應用程式體驗。
    
    ![][8]

您已經完成設定工作。 接著要建置並執行專案。

## <a name="build-the-project"></a>建置專案
1. 在 [專案總管] 中，以滑鼠右鍵按一下專案名稱，然後選擇 [執行身分] > [Maven 組建...] 以設定專案。
   
    ![][10]
2. 在 [Edit Configuration (編輯組態)] 的 [Goals (目標)] 中輸入 "clean install"，然後按一下 [Run (執行)] 。

狀態訊息會輸出至主控台視窗。 您應該會看見 BUILD SUCCESS 訊息，指出專案已建置完成，未發生任何錯誤。

## <a name="run-the-app"></a>執行應用程式
在此最後步驟中，您要在本機伺服器執行階段環境中執行應用程式。

如果您尚未在 Eclipse 中指定伺服器執行階段環境，請務必先行指定。

1. 在 Project Explorer 中，展開 [WebContent] 。
2. 以滑鼠右鍵依序按一下 [Search.jsp] > [執行身分] > [在伺服器上執行]。 選取 Apache Tomcat 伺服器，然後按一下 [Run (執行)] 。

> [!TIP]
> 如果您使用非預設工作區來儲存專案，則需要修改 [執行組態]，使其指向專案位置，以避免伺服器啟動錯誤。 在 [專案總管] 中，以滑鼠右鍵依序按一下 [Search.jsp] > [執行身分] > [執行組態]。 選取 Apache Tomcat 伺服器。 按一下 [Arguments (引數)] 。 按一下 [Workspace (工作區)] 或 [File System (檔案系統)]，以設定包含專案的資料夾。
> 
> 

執行應用程式時，您應該會看見瀏覽器視窗，並在其中提供可輸入字詞的搜尋方塊。

等待約一分鐘後再按一下 [Search (搜尋)]  ，讓服務有時間建立並載入索引。 如果遇到 HTTP 404 錯誤，只要多等一下，即可再重試一次。

## <a name="search-on-usgs-data"></a>搜尋 USGS 資料
USGS 資料集包含與羅德島州相關的記錄。 如果您在空白的搜尋方塊中按一下 [搜尋]  ，預設會出現前 50 個項目。

輸入搜尋詞彙，讓搜尋引擎運作一下。 試著輸入區域名稱。 "Roger Williams" 是羅德島州的第一任州長。 許多公園、建築物和學校都是以他的姓名命名。

![][11]

此外，您也可以試著使用這些字詞：

* Pawtucket
* Pembroke
* goose +cape

## <a name="next-steps"></a>後續步驟
這是第一個以 Java 和 USGS 資料集為基礎的 Azure 搜尋服務教學課程。 我們會漸漸擴充本教學課程，以示範其他您可能會想用在自訂方案中的搜尋功能。

如果您已有一些 Azure 搜尋服務的背景知識，可以利用此範例做為進一步實驗的跳板，例如擴充[搜尋頁面](search-pagination-page-layout.md)或實作[多面向導覽](search-faceted-navigation.md)。 您也可以新增計數和批次處理文件，讓使用者可以逐頁查看結果，藉此改進搜尋結果頁面。

不熟悉 Azure 搜尋服務嗎？ 建議您嘗試學習其他教學課程，深入了解您還可以建立哪些東西。 請瀏覽我們的 [文件頁面](https://azure.microsoft.com/documentation/services/search/) 以尋找更多資源。 

<!--Image references-->
[1]: ./media/search-get-started-java/create-search-portal-1.PNG
[2]: ./media/search-get-started-java/create-search-portal-21.PNG
[3]: ./media/search-get-started-java/create-search-portal-31.PNG
[4]: ./media/search-get-started-java/AzSearch-Java-Import1.PNG
[5]: ./media/search-get-started-java/AzSearch-Java-config1.PNG
[6]: ./media/search-get-started-java/AzSearch-Java-ProjectFacets1.PNG
[7]: ./media/search-get-started-java/AzSearch-Java-runtime1.PNG
[8]: ./media/search-get-started-java/AzSearch-Java-Browser1.PNG
[9]: ./media/search-get-started-java/AzSearch-Java-JREPref1.PNG
[10]: ./media/search-get-started-java/AzSearch-Java-BuildProject1.PNG
[11]: ./media/search-get-started-java/rogerwilliamsschool1.PNG
[12]: ./media/search-get-started-java/AzSearch-Java-SelectProject.png
