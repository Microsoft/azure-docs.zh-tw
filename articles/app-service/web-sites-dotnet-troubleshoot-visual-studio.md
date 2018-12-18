---
title: 使用 Visual Studio 疑難排解 Azure App Service 中的 Web 應用程式
description: 了解如何使用 Visual Studio 2013 內建的遠端偵錯、追蹤和記錄工具，來疑難排解 Azure Web 應用程式。
services: app-service
documentationcenter: .net
author: cephalin
manager: cfowler
editor: ''
ms.assetid: def8e481-7803-4371-aa55-64025d116c97
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 08/29/2016
ms.author: cephalin
ms.openlocfilehash: ba84d297420ca5a9b75b4cfa432373d3070e0d01
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125239"
---
# <a name="troubleshoot-a-web-app-in-azure-app-service-using-visual-studio"></a>使用 Visual Studio 疑難排解 Azure App Service 中的 Web 應用程式
## <a name="overview"></a>概觀
本教學課程示範如何使用 Visual Studio 工具，協助針對 [App Service](http://go.microsoft.com/fwlink/?LinkId=529714) 中的 Web 應用程式進行偵錯，方法是以[偵錯模式](https://docs.microsoft.com/visualstudio/debugger/)從遠端執行，或者檢視應用程式記錄與 Web 伺服器記錄。

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

您將了解：

* Visual Studio 中提供的 Azure Web 應用程式管理功能有哪些。
* 如何使用 Visual Studio 遠端檢視，對遠端 Web 應用程式進行快速變更。
* 在 Azure 中執行專案時，如何同時針對 Web 應用程式和 WebJob 從遠端執行偵錯模式。
* 如何建立應用程式追蹤記錄，並在應用程式建立記錄時加以檢視。
* 如何檢視 Web 伺服器記錄，包括詳細的錯誤訊息與失敗的要求追蹤。
* 如何將診斷記錄傳送至 Azure 儲存體帳戶並在該處檢視。

如果您有 Visual Studio Ultimate，您也可以使用 [IntelliTrace](http://msdn.microsoft.com/library/vstudio/dd264915.aspx) 進行偵錯。 本教學課程未涵蓋 IntelliTrace。

## <a name="prerequisites"></a>必要條件
本教學課程可運用於開發環境、Web 專案與您在[開始使用 Azure 和 ASP.NET](app-service-web-get-started-dotnet-framework.md) 中所設定的 Azure Web 應用程式。 針對 WebJobs 區段，您將會用到您在[開始使用 Azure WebJobs SDK][GetStartedWJ] 中建立的應用程式。

本教學課程中所提供的程式碼範例適用於 C# MVC Web 應用程式，但是疑難排解程序則是與 Visual Basic 和 Web Form 應用程式一樣。

此教學課程假設您使用 Visual Studio 2017。 

串流記錄功能僅適用於鎖定 .NET Framework 4 或更新版本的應用程式。

## <a name="sitemanagement"></a>Web 應用程式組態與管理
Visual Studio 可讓您存取 [Azure 入口網站](http://go.microsoft.com/fwlink/?LinkId=529715)中可用的 Web 應用程式管理功能與組態設定的子集。 本節將說明使用**伺服器總管**可用的項目。 若要查看最新的 Azure 整合功能，也請試試 **雲端總管** 。 您可以同時從 [檢視]  功能表開啟這兩個視窗。

1. 如果您尚未在 Visual Studio 中登入 Azure，請在 [伺服器總管] 中以滑鼠右鍵按一下 [Azure]，然後選取 [連接到 Microsoft Azure 訂用帳戶]。

    替代方式為安裝可讓您存取帳戶的管理憑證。 如果您選擇安裝憑證，請以滑鼠右鍵按一下 [伺服器總管] 中的 [Azure] 節點，然後選取內容功能表中的 [管理與篩選訂用帳戶]。 在 [管理 Microsoft Azure 訂用帳戶] 對話方塊中，按一下 [憑證] 索引標籤，再按一下 [匯入]。 請依照指示下載，然後匯入您 Azure 帳戶的訂用帳戶檔案 (亦稱為 *.publishsettings* 檔案)。

   > [!NOTE]
   > 如果您下載了訂用帳戶檔案，請將其儲存到原始程式碼目錄以外的資料夾 (例如在 Downloads 資料夾)，然後在匯入完成後刪除該檔案。 惡意使用者一旦能夠存取此訂閱檔案，就能夠編輯、建立和刪除您的 Azure 服務。
   >
   >

    如需從 Visual Studio 連線至 Azure 資源的詳細資訊，請參閱 [管理帳戶、訂閱和系統管理角色](http://go.microsoft.com/fwlink/?LinkId=324796#BKMK_AccountVCert)。
2. 在 [伺服器總管] 中，展開 [Azure]，然後展開 [App Service]。
3. 展開包含 Web 應用程式的資源群組，該 Web 應用程式是您在＜[在 Azure 中建立 ASP.NET Web 應用程式](app-service-web-get-started-dotnet-framework.md)＞中建立的，然後使用滑鼠右鍵按一下 Web 應用程式節點，然後按一下 [檢視設定]。

    ![在伺服器總管中檢視設定](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewsettings.png)

    [Azure Web App]  索引標籤隨即顯示，而您會看到 Visual Studio 中所提供的 Web 應用程式管理與組態工作。

    ![Azure Web 應用程式視窗](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configtab.png)

    在本教學課程中，您將使用記錄與追蹤下拉式清單。 您也會使用遠端偵錯功能，但是將以不同的方式來加以啟用。

    如需此視窗中 [應用程式設定] 與 [連接字串] 方塊的詳細資訊，請參閱 [Azure Web Apps：應用程式設定與連接字串如何運作](https://azure.microsoft.com/blog/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)。

    若您想要執行無法在此視窗中完成的 Web 應用程式管理工作，請按一下 [在管理入口網站中開啟]  ，以開啟 Azure 入口網站的瀏覽器視窗。

## <a name="remoteview"></a>在伺服器總管中存取 Web 應用程式檔案
部署 Web 專案時，通常會將 Web.config 檔案中的 `customErrors` 旗標設為 `On` 或 `RemoteOnly`，這表示出現問題時，您將不會收到有用的錯誤訊息。 對許多錯誤而言，您只會看到如下列之一的頁面：

**'/' 應用程式中有伺服器錯誤：**

![Unhelpful error page](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror.png)

**發生錯誤：**

![Unhelpful error page](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror1.png)

**網站無法顯示頁面**

![Unhelpful error page](./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror2.png)

要找到錯誤原因最簡單的方式，往往就是啟用詳細的錯誤訊息，而以上第一個螢幕擷取畫面說明的是其做法。 該做法需要在部署的 Web.config 檔案中進行變更。 您可以編輯專案中的 *Web.config* 檔案並重新部署專案，或建立 [Web.config 轉換](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/web-config-transformations)並部署偵錯組建，但還有更快的方法：在 [方案總管] 中使用 [遠端檢視] 功能，直接檢視及編輯遠端 Web 應用程式中的檔案。

1. 在 [伺服器總管] 中，依序展開 [Azure]、[App Service] 和 Web 應用程式所在的資源群組，然後展開 Web 應用程式的節點。

    您會看到可供您存取 Web 應用程式內容檔案與記錄檔的節點。
2. 展開 [檔案]  節點，然後按兩下 *Web.config* 檔案。

    ![開啟 Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfig.png)

    Visual Studio 會從遠端 Web 應用程式開啟 Web.config 檔案，然後在標題列中的檔案名稱旁邊顯示 [遠端]。
3. 將下列字行新增至 `system.web` 元素：

    `<customErrors mode="Off"></customErrors>`

    ![編輯 Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfigedit.png)
4. 重新整理顯示沒有幫助的錯誤訊息的瀏覽器，現在您就會看到詳細的錯誤訊息，例如以下範例：

    ![詳細的錯誤訊息](./media/web-sites-dotnet-troubleshoot-visual-studio/detailederror.png)

    (顯示的錯誤是透過將紅色字行新增至 *Views\Home\Index.cshtml* 中來加以建立。)

能夠讀取與編輯您的 Azure Web 應用程式上的檔案，使得疑難排解作業更為輕鬆，編輯 Web.config 檔案僅僅是其中一個範例案例。

## <a name="remotedebug"></a>遠端偵錯 Web 應用程式
如果詳細的錯誤訊息無法提供足夠的資訊，而且您無法在本機重現錯誤，另一個疑難排解的方式則是在遠端於偵錯模式中執行。 您可以設定中斷點、直接操作記憶體、逐步執行程式碼，甚至變更程式碼路徑。

遠端偵錯無法在 Visual Studio 的 Express 版本中運作。

本節示範如何使用您在[在 Azure 中建立 ASP.NET Web 應用程式](app-service-web-get-started-dotnet-framework.md)中建立的專案進行遠端偵錯。

1. 開啟您在[在 Azure 中建立 ASP.NET Web 應用程式](app-service-web-get-started-dotnet-framework.md)中建立的 Web 專案。

2. 開啟 *Controllers\HomeController.cs*。

3. 刪除 `About()` 方法，然後插入以下程式碼加以取代。

``` c#
public ActionResult About()
{
    string currentTime = DateTime.Now.ToLongTimeString();
    ViewBag.Message = "The current time is " + currentTime;
    return View();
}
```

4. [在 `ViewBag.Message`這行設定中斷點](https://docs.microsoft.com/visualstudio/debugger/)。

5. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，再按一下 [發行]。

6. 在 [設定檔] 下拉式清單中，選取您在[在 Azure 中建立 ASP.NET Web 應用程式](app-service-web-get-started-dotnet-framework.md)中所使用的同一設定檔。 然後，按一下 [設定]。

7. 在 [發佈] 對話方塊中，按一下 [設定] 索引標籤，然後將 [設定] 變更為 [偵錯]，接著按一下 [儲存]。

    ![於偵錯模式中發行](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-publishdebug.png)

8. 按一下 [發佈] 。 當部署完成且您的瀏覽器開啟至 Web 應用程式的 Azure URL 之後，請關閉瀏覽器。

9. 在 [伺服器總管] 中，以滑鼠右鍵按一下您的 Web 應用程式，接著按一下 [連結偵錯工具]。

    ![連結偵錯工具](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-attachdebugger.png)

    瀏覽器會自動開啟至您在 Azure 中執行的首頁。 您可能需要等候 20 秒左右的時間，讓 Azure 設定要偵錯的伺服器。 通常只有當您第一次在 Web 應用程式上執行偵錯模式的 48 小時內，才會出現這個延遲現象。 當您於相同期間內再次啟動偵錯時，就不會發生延遲。

    > [!NOTE] 
    > 如果您有任何啟動偵錯工具的問題，請嘗試使用 **Cloud Explorer** 執行，而不是**伺服器總管**。
    >

10. 按一下功能表中的 [關於]  。

     Visual Studio 會在中斷點處停止，而程式碼是在 Azure 中執行，而不是在您的本機電腦上執行。

11. 將滑鼠移至 `currentTime` 變數上，以查看時間值。

     ![檢視 Azure 偵錯模式中執行的變數](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugviewinwa.png)

     您在 Azure 伺服器時間上看到的時間，可能與您的本機電腦所在的時區不同。

12. 為 `currentTime` 變數輸入新的值，例如「現在於 Azure 中執行」。

13. 按 F5 繼續執行。

     Azure 中執行的 [關於] 頁面會顯示您輸入到 currentTime 變數中的新值。

     ![含有新值的關於頁面](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugchangeinwa.png)

## <a name="remotedebugwj"></a> 遠端偵錯 WebJobs
本節說明如何使用您在 [開始使用 Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/wiki)中建立的專案和 Web 應用程式進行遠端偵錯。

本節顯示的功能僅適用於 Visual Studio 2013 含 Update 4 或更新版本。

遠端偵錯僅適用於連續的 WebJobs。 排程與隨選 WebJobs 不支援偵錯。

1. 開啟您在[開始使用 Azure WebJobs SDK][GetStartedWJ] 中建立的 Web 專案。

2. 在 ContosoAdsWebJob 專案中，開啟 *Functions.cs*。

3. 在 `GnerateThumbnail` 方法的第一個陳述式上[設定中斷點](https://docs.microsoft.com/visualstudio/debugger/)。

    ![設定中斷點](./media/web-sites-dotnet-troubleshoot-visual-studio/wjbreakpoint.png)

4. 在 [方案總管] 中，以滑鼠右鍵按一下 Web 專案 (不是 WebJob 專案)，然後按一下 [發佈]。

5. 在 [設定檔]  下拉式清單中，選取您在 [開始使用 Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/wiki)中所使用的相同設定檔。

6. 按一下 [設定] 索引標籤，然後將 [組態] 變更為 [偵錯]，然後按一下 [發行]。

    Visual Studio 會部署 Web 和 WebJob 專案，且在瀏覽器中開啟您 Web 應用程式的 Azure URL。

7. 在 [伺服器總管] 中，依序展開 [Azure] > [App Service] > 您的資源群組 > 您的 Web 應用程式 > [WebJobs] > [連續]，然後以滑鼠右鍵按一下 [ContosoAdsWebJob]。

8. 按一下 [連結偵錯工具] 。

    ![連結偵錯工具](./media/web-sites-dotnet-troubleshoot-visual-studio/wjattach.png)

    瀏覽器會自動開啟至您在 Azure 中執行的首頁。 您可能需要等候 20 秒左右的時間，讓 Azure 設定要偵錯的伺服器。 通常只有當您第一次在 Web 應用程式上執行偵錯模式的 48 小時內，才會出現這個延遲現象。 當您於相同期間內再次啟動偵錯時，就不會發生延遲。

9. 在開啟至 Contoso 廣告首頁的 Web 瀏覽器中，建立新的廣告。

    建立廣告會建立由 WebJob 挑選並處理的佇列訊息。 當 WebJobs SDK 呼叫函式來處理佇列訊息時，程式碼便會觸及中斷點。

10. 當偵錯工具在中斷點出現中斷時，您可以檢查並變更在雲端執行程式時的變數值。 在下圖中，偵錯工具顯示已傳遞到 `GenerateThumbnail` 方法之 blobInfo 物件的內容。

     ![偵錯工具中的 blobInfo 物件](./media/web-sites-dotnet-troubleshoot-visual-studio/blobinfo.png)

11. 按 F5 繼續執行。

     `GenerateThumbnail` 方法完成建立縮圖。

12. 在瀏覽器中重新整理索引頁面，您便會看見縮圖。

13. 在 Visual Studio 中，按 SHIFT+F5 停止偵錯。

14. 在 [伺服器總管] 中，以滑鼠右鍵在 [伺服器總管] 中，以滑鼠右鍵按一下 ContosoAdsWebJob 節點，然後按一下 [檢視儀表板]。

15. 使用您的 Azure 認證登入，然後按一下 WebJob 名稱，可前往 WebJob 的頁面。

     ![按一下 ContosoAdsWebJob](./media/web-sites-dotnet-troubleshoot-visual-studio/clickcaw.png)

     儀表板會顯示最近執行的 `GenerateThumbnail` 函式。

     (下次按一下 [檢視儀表板] 時，您無需登入，瀏覽器便會直接進入您的 WebJob 頁面。)

16. 按一下函數名稱可查看執行函數的詳細資料。

     ![函數詳細資料](./media/web-sites-dotnet-troubleshoot-visual-studio/funcdetails.png)

如果函數會[撰寫記錄](https://github.com/Azure/azure-webjobs-sdk/wiki)，您可以按一下 [ToggleOutput] 查看這些記錄。

## <a name="notes-about-remote-debugging"></a>遠端偵錯注意事項

* 不建議在生產環境中執行偵錯模式。 如果您的生產 Web 應用程式並未相應放大到多個伺服器執行個體，偵錯就會防止 Web 伺服器回應其他要求。 如果您確實有多個 Web 伺服器執行個體，當您連結至偵錯工具時，會取得一個隨機執行個體，而且您無法確保後續瀏覽器要求會傳送到相同的執行個體。 此外，通常我們不會將偵錯組建部署到生產環境中，而且版本組建的編譯器最佳化作業將會使系統無法逐行顯示您的來源程式碼中所發生的事情。 針對生產環境問題的疑難排解，您的最佳資源將是應用程式追蹤與 Web 伺服器記錄。
* 進行遠端偵錯時，避免長時間在中斷點停止運作。 Azure 會將停止超過幾分鐘的處理序視為無回應的處理序，並將其關閉。
* 在偵錯期間，伺服器會將資料傳送至 Visual Studio，進而影響頻寬付費情況。 如需關於頻寬費率的詳細資訊，請參閱 [Azure 定價](https://azure.microsoft.com/pricing/calculator/)。
* 確保 *Web.config* 檔案裡 `compilation` 元素中的 `debug` 屬性設為 true。 在發行偵錯組建組態時，該值預設會設為 true。

``` xml
<system.web>
  <compilation debug="true" targetFramework="4.5" />
  <httpRuntime targetFramework="4.5" />
</system.web>
```
* 如果您發現偵錯工具不會逐步執行您要進行偵錯的程式碼，可能需要變更 [Just My Code] 設定。  如需詳細資訊，請參閱[指定是否使用 Visual Studio 中的 Just My Code 只對使用者程式碼進行偵錯](https://docs.microsoft.com/visualstudio/debugger/just-my-code)。
* 當您啟用遠端偵錯功能時，伺服器上會啟動計時器，並在 48 小時後自動關閉此功能。 此 48 小時的限制是為了安全性與效能起見而設計的功能。 若需要，您可以輕鬆開啟這項功能，次數不限。 當您不需要偵錯時，建議您將其保持為停用。
* 您可以手動將偵錯工具附加至任何處理序，不僅止於 Web 應用程式處理序 (w3wp.exe)。 如需如何在 Visual Studio 中使用偵錯模式的詳細資訊，請參閱 [Visual Studio 偵錯](http://msdn.microsoft.com/library/vstudio/sc65sadd.aspx)。

## <a name="logsoverview"></a>診斷記錄概觀
在 Azure Web 應用程式中執行的 ASP.NET 應用程式，可建立下列各種記錄：

* **應用程式追蹤記錄**<br/>
  此應用程式會呼叫 [System.Diagnostics.Trace](http://msdn.microsoft.com/library/system.diagnostics.trace.aspx) 類別的方法來建立這些記錄。
* **Web 伺服器記錄**<br/>
  Web 伺服器會為每個通往 Web 應用程式的 HTTP 要求建立記錄項目。
* **詳細的錯誤訊息記錄**<br/>
  Web 伺服器會針對失敗的 HTTP 要求 (產生狀態碼 400 或以上的要求) 建立含有一些額外資訊的 HTML 頁面。
* **失敗要求追蹤記錄**<br/>
  Web 伺服器會針對失敗的 HTTP 要求建立含有詳細追蹤資訊的 XML 檔案。 Web 伺服器會一併提供 XSL 檔案，在瀏覽器中格式化 XML。

記錄功能會影響 Web 應用程式效能，因此 Azure 可讓您視需要啟用或停用每一種記錄類型。 對於應用程式記錄，您可以指定只寫入高於特定嚴重性層級的記錄。 當您建立新的 Web 應用程式時，預設會停用所有記錄功能。

記錄會寫入 Web 應用程式的檔案系統中 *LogFiles* 資料夾內的檔案，而且可以透過 FTP 存取此檔案。 Web 伺服器記錄與應用程式記錄也可以寫入至 Azure 儲存體帳戶。 您可以在儲存體帳戶中保留大於檔案系統可容納的記錄檔數量。 當您使用檔案系統時，最大的記錄檔大小為 100 MB。 (檔案系統記錄僅適用於短期保留之用。 Azure 會在達到此上限之後刪除舊的記錄檔以騰出空間給新的記錄檔使用。)  

## <a name="apptracelogs"></a>建立並檢視應用程式追蹤記錄
在本節中，您將會執行下列工作：

* 將追蹤陳述式新增至您在[開始使用 Azure 和 ASP.NET](app-service-web-get-started-dotnet-framework.md) 中建立的 Web 專案。
* 當您在本機上執行專案時檢視記錄。
* 依原樣檢視 Azure 中執行的應用程式所產生的記錄。

如需如何在 WebJobs 中建立應用程式記錄的詳細資訊，請參閱 [如何運用 WebJobs SDK 來使用 Azure 佇列儲存體 - 如何寫入記錄](https://github.com/Azure/azure-webjobs-sdk/wiki)。 下列有關在 Azure 中檢視記錄和控制記錄儲存方式的指示也同樣適用於 WebJobs 所建立的應用程式記錄。

### <a name="add-tracing-statements-to-the-application"></a>將追蹤陳述式新增至應用程式
1. 開啟 *Controllers\HomeController.cs*，然後使用下列程式碼來取代`Index`、`About` 和 `Contact` 方法，以便為 `System.Diagnostics` 加入 `Trace` 陳述式與 `using` 陳述式：

```c#
public ActionResult Index()
{
    Trace.WriteLine("Entering Index method");
    ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
    Trace.TraceInformation("Displaying the Index page at " + DateTime.Now.ToLongTimeString());
    Trace.WriteLine("Leaving Index method");
    return View();
}

public ActionResult About()
{
    Trace.WriteLine("Entering About method");
    ViewBag.Message = "Your app description page.";
    Trace.TraceWarning("Transient error on the About page at " + DateTime.Now.ToShortTimeString());
    Trace.WriteLine("Leaving About method");
    return View();
}

public ActionResult Contact()
{
    Trace.WriteLine("Entering Contact method");
    ViewBag.Message = "Your contact page.";
    Trace.TraceError("Fatal error on the Contact page at " + DateTime.Now.ToLongTimeString());
    Trace.WriteLine("Leaving Contact method");
    return View();
}        
```

2. 將 `using System.Diagnostics;` 陳述式新增至檔案頂端。

### <a name="view-the-tracing-output-locally"></a>在本機檢視追蹤輸出
1. 按 F5 以在偵錯模式中執行應用程式。

    預設的追蹤接聽程式會將所有追蹤輸出連同其他「偵錯」輸出，一起寫入到 [輸出]  視窗。 下圖顯示您加入 `Index` 方法的追蹤陳述式的輸出。

    ![偵錯視窗中的追蹤](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugtracing.png)

    以下步驟說明如何在網頁中檢視追蹤輸出，而不需在偵錯模式中編譯。
2. 開啟應用程式 Web.config 檔 (專案資料夾中的那個) 並將 `<system.diagnostics>` 元素新增至檔案結尾 `</configuration>` 元素關閉處前：

``` xml
<system.diagnostics>
<trace>
  <listeners>
    <add name="WebPageTraceListener"
        type="System.Web.WebPageTraceListener,
        System.Web,
        Version=4.0.0.0,
        Culture=neutral,
        PublicKeyToken=b03f5f7f11d50a3a" />
  </listeners>
</trace>
</system.diagnostics>
```

`WebPageTraceListener` 可讓您藉由瀏覽至 `/trace.axd` 來檢視追蹤輸出。
3. 在 Web.config 檔案的 `<system.web>` 下方，加入<a href="http://msdn.microsoft.com/library/vstudio/6915t83k(v=vs.100).aspx">追蹤元素</a>，如以下範例所示：

``` xml
<trace enabled="true" writeToDiagnosticsTrace="true" mostRecent="true" pageOutput="false" />
```       

4. 按 CTRL+F5 執行應用程式。
5. 在瀏覽器視窗的網址列中，將 *trace.axd* 新增至 URL，然後按 Enter (URL 類似於 http://localhost:53370/trace.axd))。
6. 在 [應用程式追蹤] 頁面上，按一下第一行 (不是 BrowserLink 行) 上的 [檢視詳細資料]。

    ![trace.axd](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd1.png)

    [要求詳細資訊] 頁面隨即顯示，而且在 [追蹤資訊] 區段中，您會看到先前加入 `Index` 方法的追蹤陳述式輸出。

    ![trace.axd](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd2.png)

    根據預設， `trace.axd` 只能在本機使用。 如果您想從遠端 Web 應用程式使用它，可以將 `localOnly="false"` 加入 *Web.config* 檔案中的 `trace` 元素，如以下範例所示：

        <trace enabled="true" writeToDiagnosticsTrace="true" localOnly="false" mostRecent="true" pageOutput="false" />

    不過基於安全性考量，建議您不要在生產 Web 應用程式中啟用 `trace.axd`。 在以下各節中，您會看到讀取 Azure Web 應用程式中追蹤記錄的更輕鬆方式。

### <a name="view-the-tracing-output-in-azure"></a>在 Azure 中檢視追蹤輸出
1. 在 [方案總管] 中，於 Web 專案上按一下滑鼠右鍵，再按一下 [發行]。
2. 在 [發佈 Web] 對話方塊中，按一下 [發佈]。

    當 Visual Studio 成功發行您的更新後，將會開啟瀏覽器視窗至您的首頁 (假設您並未清除 [連線] 索引標籤上的 [目的地 URL])。
3. 在 [伺服器總管] 中，以滑鼠右鍵按一下您的 Web 應用程式，然後選取 [檢視串流記錄]。

    ![在內容功能表中檢視串流記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewlogsmenu.png)

    [輸出]  視窗會顯示您已連線至記錄串流服務，並每一分鐘將沒有要顯示的記錄新增一行通知文字。

    ![在內容功能表中檢視串流記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-nologsyet.png)
4. 在顯示您的應用程式首頁的瀏覽器視窗中，按一下 [連絡人] 。

    幾秒鐘後，您加入 `Contact` 方法的錯誤層級追蹤輸出便會顯示在 [輸出] 視窗。

    ![輸出視窗中的錯誤追蹤](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-errortrace.png)

    Visual Studio 只會顯示錯誤層級追蹤，因為那是您在啟用記錄監視服務時的預設設定。 建立新的 Azure Web 應用程式時，預設會停用所有記錄功能，正如同稍早您在開啟設定頁面時所見：

    ![應用程式記錄功能關閉](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-apploggingoff.png)

    不過，當您選取 [檢視串流記錄] 時，Visual Studio 會自動將 [Application Logging (File System)] 變更為 [錯誤]，代表回報的會是錯誤層級記錄。 為了查看所有的追蹤記錄，您可以將此設定變更為 [詳細資訊] 。 當您選取低於錯誤的嚴重性層級時，將一併回報較高嚴重性層級的所有記錄。 因此當您選取詳細資訊時，您會同時看到資訊、警告與錯誤記錄。  

1. 在 [伺服器總管] 中，以滑鼠右鍵按一下 Web 應用程式，然後按一下 [檢視設定] \(如同您稍早所做的動作)。
2. 將 [Application Logging (File System)] 變更為 [詳細資訊]，然後按一下 [儲存]。

    ![將追蹤層級設定為詳細資訊](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-applogverbose.png)
3. 在顯示您的 [連絡人] 頁面的瀏覽器視窗中，依序按一下 [首頁]、[關於]、[連絡人]。

    在幾秒鐘內，[輸出]  視窗就會顯示您的所有追蹤輸出。

    ![詳細資訊追蹤輸出](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-verbosetraces.png)

    在本節中，您已使用 Azure Web 應用程式設定來啟用及停用記錄功能。 您也可以修改 Web.config 檔案，來啟用與停用追蹤接聽程式。 不過，修改 Web.config 檔案會導致應用程式網域回收，而透過 Web 應用程式設定來啟用記錄功能則不會有這種現象。 如果此問題需要經過長時間才會重現或是間歇性出現，則回收應用程式網域可能「修正」此問題，並強迫您等候問題再次發生。 在 Azure 中啟用診斷功能可讓您立即開始擷取錯誤資訊，而不需回收應用程式定義域。

### <a name="output-window-features"></a>輸出視窗功能
[輸出] 視窗的 [Microsoft Azure 記錄] 索引標籤具有多個按鈕與一個文字方塊：

![記錄索引標籤按鈕](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-icons.png)

這些物件可執行下列功能：

* 清除 [輸出]  視窗。
* 啟用或停用自動換行。
* 啟動或停止監視記錄。
* 指定要監視的記錄。
* 下載記錄。
* 依據搜尋字串或規則運算式篩選記錄。
* 關閉 [輸出]  視窗。

如果您輸入搜尋字串或是規則運算式，則 Visual Studio 會篩選用戶端的記錄資訊。 亦即，您可以等到 [輸出]  視窗顯示記錄後輸入條件，這樣您不需重新產生記錄便能直接變更篩選條件。

## <a name="webserverlogs"></a>檢視 Web 伺服器記錄
Web 伺服器記錄會記下 Web 應用程式的所有 HTTP 活動。 為了在 [輸出] 視窗中看見這些記錄，您必須針對 Web 應用程式啟用它們，然後告訴 Visual Studio 您想要監視它們。

1. 在您從 [伺服器總管] 開啟的 [Azure Web 應用程式設定] 索引標籤中，將 [Web 伺服器記錄] 變更為 [開啟]，然後按一下 [儲存]。

    ![啟用 Web 伺服器記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-webserverloggingon.png)
2. 在 [輸出] 視窗中，按一下 [指定要監視的 Microsoft Azure 記錄] 按鈕。

    ![指定要監視的 Azure 記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-specifylogs.png)
3. 在 [Microsoft Azure 記錄選項] 對話方塊中，選取 [Web 伺服器記錄]，然後按一下 [確定]。

    ![監視 Web 伺服器記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorwslogson.png)
4. 在顯示 Web 應用程式的瀏覽器視窗中，依序按一下 [首頁]、[關於] 和 [連絡人]。

    通常會先產生應用程式記錄，然後才是 Web 伺服器記錄。 您可能需要等候一小段時間，記錄才會顯示。

    ![輸出視窗中的 Web 伺服器記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-wslogs.png)

根據預設，當您第一次使用 Visual Studio 啟用 Web 伺服器記錄時，Azure 會將記錄寫入檔案系統。 作為替代方式，您可以使用 Azure 入口網站來指定應該將 Web 伺服器記錄寫入儲存體帳戶中的某個 Blob 容器。

如果您使用入口網站對 Azure 儲存體帳戶啟用 Web 伺服器記錄功能，然後在 Visual Studio 中停用記錄功能，當您在 Visual Studio 中重新啟用記錄功能時，將會還原您的儲存體帳戶設定。

## <a name="detailederrorlogs"></a>檢視詳細的錯誤訊息記錄
針對導致錯誤回應代碼 (400 或以上) 之 HTTP 要求，詳細的錯誤訊息記錄可提供部分額外資訊。 為了在 [輸出]  視窗中看見這些記錄，您必須針對 Web 應用程式啟用它們，然後告訴 Visual Studio 您想要監視它們。

1. 在您從 [伺服器總管] 開啟的 [Azure Web 應用程式組態] 索引標籤中，將 [詳細錯誤訊息] 變更為 [開啟]，然後按一下 [儲存]。

    ![啟用詳細的錯誤訊息](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailedlogson.png)

2. 在 [輸出] 視窗中，按一下 [指定要監視的 Microsoft Azure 記錄] 按鈕。

3. 在 [Microsoft Azure 記錄選項] 對話方塊中，按一下 [所有記錄]，然後按一下 [確定]。

    ![監視所有記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorall.png)

4. 在瀏覽器視窗的網址列中，為 URL 加入額外的字元以引發 404 錯誤 (例如， `http://localhost:53370/Home/Contactx`)，然後按 Enter 鍵。

    幾秒鐘之後，詳細的錯誤記錄就會顯示在 Visual Studio 的 [輸出] 視窗。

    ![輸出視窗：詳細的錯誤記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorlog.png)

    按住 Ctrl 鍵並按一下該連結，可在瀏覽器中查看格式化的記錄輸出：

    ![瀏覽器視窗：詳細的錯誤記錄](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorloginbrowser.png)

## <a name="downloadlogs"></a>下載檔案系統記錄
任何您可在 [輸出]  視窗中監視的記錄，也能下載為 *.zip* 檔案。

1. 在 [輸出] 視窗中，按一下 [Download Streaming Logs]。

    ![記錄索引標籤按鈕](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadicon.png)

    [檔案總管] 會開啟至您的 *Downloads* 資料夾，並選取下載的檔案。

    ![下載的檔案](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadedfile.png)
2. 將 *.zip* 檔案解壓縮後，您會看到下列資料夾結構：

    ![下載的檔案](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilefolders.png)

   * 應用程式追蹤記錄位於 LogFiles\Application 資料夾的 .txt 檔案中。
   * Web 伺服器記錄位於 LogFiles\http\RawLogs 資料夾的 .log 檔案中。 您可以使用 [記錄檔剖析器](http://www.microsoft.com/download/details.aspx?displaylang=en&id=24659) (英文) 之類的工具來檢視與操作這些檔案。
   * 詳細的錯誤訊息記錄位於 LogFiles\DetailedErrors 資料夾的 .html 檔案中。

    (deployments 資料夾用於存放來源控制發行功能所建立的檔案，它與 Visual Studio 發行功能沒有任何關聯。 Git 資料夾則用於存放與來源控制發行功能相關的追蹤記錄，以及記錄檔案串流服務。)  

<!-- ## <a name="storagelogs"></a>View storage logs
Application tracing logs can also be sent to an Azure storage account, and you can view them in Visual Studio. To do that you'll create a storage account, enable storage logs in the Azure portal, and view them in the **Logs** tab of the **Azure Web App** window.

You can send logs to any or all of three destinations:

* The file system.
* Storage account tables.
* Storage account blobs.

You can specify a different severity level for each destination.

Tables make it easy to view details of logs online, and they support streaming; you can query logs in tables and see new logs as they are being created. Blobs make it easy to download logs in files and to analyze them using HDInsight, because HDInsight knows how to work with blob storage. For more information, see **Hadoop and MapReduce** in [Data Storage Options (Building Real-World Cloud Apps with Azure)](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options).

You currently have file system logs set to verbose level; the following steps walk you through setting up information level logs to go to storage account tables. Information level means all logs created by calling `Trace.TraceInformation`, `Trace.TraceWarning`, and `Trace.TraceError` will be displayed, but not logs created by calling `Trace.WriteLine`.

Storage accounts offer more storage and longer-lasting retention for logs compared to the file system. Another advantage of sending application tracing logs to storage is that you get some additional information with each log that you don't get from file system logs.

1. Right-click **Storage** under the Azure node, and then click **Create Storage Account**.

![Create Storage Account](./media/web-sites-dotnet-troubleshoot-visual-studio/createstor.png)

1. In the **Create Storage Account** dialog, enter a name for the storage account.

    The name must be must be unique (no other Azure storage account can have the same name). If the name you enter is already in use you'll get a chance to change it.

    The URL to access your storage account will be *{name}*.core.windows.net.
2. Set the **Region or Affinity Group** drop-down list to the region closest to you.

    This setting specifies which Azure datacenter will host your storage account. For this tutorial your choice won't make a noticeable difference, but for a production web app you want your web server and your storage account to be in the same region to minimize latency and data egress charges. The web app (which you'll create later) should run in a region as close as possible to the browsers accessing your web app in order to minimize latency.
3. Set the **Replication** drop-down list to **Locally redundant**.
   
    When geo-replication is enabled for a storage account, the stored content is replicated to a secondary datacenter to enable failover to that location in case of a major disaster in the primary location. Geo-replication can incur additional costs. For test and development accounts, you generally don't want to pay for geo-replication. For more information, see [Create, manage, or delete a storage account](../storage/common/storage-create-storage-account.md).
4. Click **Create**.

    ![New storage account](./media/web-sites-dotnet-troubleshoot-visual-studio/newstorage.png)    
5. In the Visual Studio **Azure Web App** window, click the **Logs** tab, and then click **Configure Logging in Management Portal**.

     <!-- todo:screenshot of new portal if the VS page link goes to new portal -- >
    ![Configure logging](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configlogging.png)

    This opens the **Configure** tab in the portal for your web app.
6. In the portal's **Configure** tab, scroll down to the application diagnostics section, and then change **Application Logging (Table Storage)** to **On**.
7. Change **Logging Level** to **Information**.
8. Click **Manage Table Storage**.

    ![Click Manage TableStorage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-stgsettingsmgmtportal.png)

    In the **Manage table storage for application diagnostics** box, you can choose your storage account if you have more than one. You can create a new table or use an existing one.

    ![Manage table storage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-choosestorageacct.png)
9. In the **Manage table storage for application diagnostics** box, click the check mark to close the box.
10. In the portal's **Configure** tab, click **Save**.
11. In the browser window that displays the application web app, click **Home**, then click **About**, and then click **Contact**.

     The logging information produced by browsing these web pages is written to the storage account.
12. In the **Logs** tab of the **Azure Web App** window in Visual Studio, click **Refresh** under **Diagnostic Summary**.

     ![Click Refresh](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-refreshstorage.png)

     The **Diagnostic Summary** section shows logs for the last 15 minutes by default. You can change the period to see more logs.

     (If you get a "table not found" error, verify that you browsed to the pages that do the tracing after you enabled **Application Logging (Storage)** and after you clicked **Save**.)

     ![Storage logs](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-storagelogs.png)

     Notice that in this view you see **Process ID** and **Thread ID** for each log, which you don't get in the file system logs. You can see additional fields by viewing the Azure storage table directly.
13. Click **View all application logs**.

     The trace log table appears in the Azure storage table viewer.

     (If you get a "sequence contains no elements" error, open **Server Explorer**, expand the node for your storage account under the **Azure** node, and then right-click **Tables** and click **Refresh**.)

     ![Storage logs in table view](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracelogtableview.png)

     This view shows additional fields you don't see in any other views. This view also enables you to filter logs by using special Query Builder UI for constructing a query. For more information, see Working with Table Resources - Filtering Entities in [Browsing Storage Resources with Server Explorer](http://msdn.microsoft.com/library/ff683677.aspx).
14. To look at the details for a single row, double-click one of the rows.

     ![Trace table in Server Explorer](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracetablerow.png)
 -->
## <a name="failedrequestlogs"></a>檢視失敗要求追蹤記錄
當您需要了解 IIS 如何處理 HTTP 要求的詳細資料時，例如在 URL 重新寫入或是出現驗證問題等情況，失敗要求追蹤記錄就會很有用。

Azure Web 應用程式會使用 IIS 7.0 及更新版本所提供的相同失敗要求追蹤功能。 但您無法存取用來設定哪些錯誤會被記錄下來的 IIS 設定。 當您啟用失敗要求追蹤時，將擷取所有錯誤。

您可以使用 Visual Studio 來啟用失敗要求追蹤，但是您無法在 Visual Studio 中加以檢視。 這些記錄是 XML 檔案。 串流記錄服務只會監視被當作可在純文字模式中讀取的檔案：.txt、.html 與 .log 檔案。

您可以直接在瀏覽器中透過 FTP，或是在本機使用 FTP 工具將記錄下載到本機電腦中之後檢視失敗要求追蹤記錄。 在本節中，您將在瀏覽器中直接檢視它們。

1. 在您從 [伺服器總管] 開啟的 [Azure Web 應用程式] 視窗之 [組態] 索引標籤中，將 [失敗要求追蹤] 變更為 [開啟]，然後按一下 [儲存]。

    ![啟用失敗要求追蹤](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequeston.png)
2. 在顯示 Web 應用程式的瀏覽器視窗網址列中，將額外字元新增至 URL 並按 Enter 鍵以引發 404 錯誤。

    這麼做會讓系統建立失敗要求追蹤記錄，以下步驟將說明如何檢視或下載記錄。

3. 在 Visual Studio 中，於 [Azure Web 應用程式] 視窗的 [組態] 索引標籤中按一下 [在管理入口網站中開啟]。

4. 在 [Azure 入口網站](https://portal.azure.com)適用於您 Web 應用程式的 [設定] 頁面中，按一下 [部署認證]，然後輸入新的使用者名稱和密碼。

    ![新的 FTP 使用者名稱與密碼](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-enterftpcredentials.png)

    > [!NOTE]
    > 當您登入時，必須使用此完整使用者名稱並在前面加上 Web 應用程式名稱。 例如，若您輸入使用者名稱 "myid"，而網站為 "myexample"，則會登入為 "myexample\myid"。
    >

5. 在新的瀏覽器視窗中，前往您 Web 應用程式之 [概觀] 頁面的 [FTP 主機名稱] 或 [FTPS 主機名稱] 下方所示的 URL。

6. 使用您先前建立的 FTP 認證登入 (包括該使用者名稱的 Web 應用程式名稱前置詞)。

    瀏覽器會顯示 Web 應用程式的根資料夾。

7. 開啟 *LogFiles* 資料夾。

    ![開啟 LogFiles 資料夾](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilesfolder.png)

8. 開啟名為 W3SVC 並加上數值的資料夾。

    ![開啟 W3SVC 資料夾](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfolder.png)

    該資料夾包含了一些 XML 檔案 (內含任何您在啟用失敗要求追蹤功能後所記錄的錯誤)，以及一個可供瀏覽器用來格式化該 XML 檔案的 XSL 檔案。

    ![W3SVC 資料夾](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfoldercontents.png)

9. 按一下 XML 檔案，以取得您想要檢視其追蹤資訊的失敗要求。

    下圖顯示錯誤範例追蹤資訊的片段。

    ![瀏覽器中的失敗要求追蹤](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequestinbrowser.png)

## <a name="nextsteps"></a>後續步驟
在了解 Visual Studio 如何讓您輕鬆地檢視 Azure Web 應用程式所建立的記錄之後， 下列各節提供相關主題的更多資源連結：

* Azure Web 應用程式疑難排解
* Visual Studio 偵錯
* 在 Azure 中遠端偵錯
* 在 ASP.NET 應用程式中追蹤
* 分析 Web 伺服器記錄
* 分析失敗要求追蹤記錄
* 偵錯雲端服務

### <a name="azure-web-app-troubleshooting"></a>Azure Web 應用程式疑難排解
如需在 Azure App Service 中疑難排解 Web 應用程式的詳細資訊，請參閱下列資源：

* [如何監視 Web 應用程式](web-sites-monitor.md)
* [使用 Visual Studio 2013 調查 Azure Web 應用程式中的記憶體流失](http://blogs.msdn.com/b/visualstudioalm/archive/2013/12/20/investigating-memory-leaks-in-azure-web-sites-with-visual-studio-2013.aspx)。 Microsoft ALM 部落格文章，討論 Visual Studio 中分析受控記憶體問題的功能。
* [您應該了解的 Azure Web 應用程式線上工具](https://azure.microsoft.com/blog/2014/03/28/windows-azure-websites-online-tools-you-should-know-about-2/)。 取自 Amit Apple 的部落格文章。

如需特定疑難排解問題的說明，請在下列任一個論壇中開啟一段討論串：

* [ASP.NET 網站上的 Azure 論壇](http://forums.asp.net/1247.aspx/1?Azure+and+ASP+NET)。
* [MSDN 上的 Azure 論壇](http://social.msdn.microsoft.com/Forums/windowsazure/)。
* [StackOverflow.com](http://www.stackoverflow.com)。

### <a name="debugging-in-visual-studio"></a>Visual Studio 偵錯
如需如何在 Visual Studio 中使用偵錯模式的詳細資訊，請參閱[在 Visual Studio 中偵錯](http://msdn.microsoft.com/library/vstudio/sc65sadd.aspx)與 [Visual Studio 2010 的偵錯祕訣](http://weblogs.asp.net/scottgu/archive/2010/08/18/debugging-tips-with-visual-studio-2010.aspx) \(英文\)。

### <a name="remote-debugging-in-azure"></a>在 Azure 中遠端偵錯
如需針對 Azure Web 應用程式與 WebJob 進行遠端偵錯的詳細資訊，請參閱下列資源：

* [遠端偵錯 Azure App Service Web Apps 的簡介](https://azure.microsoft.com/blog/2014/05/06/introduction-to-remote-debugging-on-azure-web-sites/)。
* [遠端偵錯 Azure App Service Web Apps 的簡介第 2 部分 - 內部遠端偵錯](https://azure.microsoft.com/blog/2014/05/07/introduction-to-remote-debugging-azure-web-sites-part-2-inside-remote-debugging/)
* [遠端偵錯 Azure App Service Web Apps 的簡介第 3 部分 - 多重執行個體環境和 GIT](https://azure.microsoft.com/blog/2014/05/08/introduction-to-remote-debugging-on-azure-web-sites-part-3-multi-instance-environment-and-git/)
* [WebJobs 偵錯 (影片)](https://www.youtube.com/watch?v=ncQm9q5ZFZs&list=UU_SjTh-ZltPmTYzAybypB-g&index=1)

如果您的 Web 應用程式使用 Azure Web API 或行動服務後端，而您需要加以偵錯，請參閱 [在 Visual Studio 中對 .NET 後端進行偵錯](http://blogs.msdn.com/b/azuremobile/archive/2014/03/14/debugging-net-backend-in-visual-studio.aspx)。

### <a name="tracing-in-aspnet-applications"></a>在 ASP.NET 應用程式中追蹤
網際網路上找不到關於 ASP.NET 追蹤功能詳盡且具時效性的說明。 您所能做的，就是從專為 Web Form 所撰寫的一些舊有簡介資料下手，因為 MVC 是最近才問世的技術，並以著重在特定議題的較新的部落格文章來做為補充。 以下資源是您開始了解這項技術的一些好去處：

* [監視與遙測 (運用 Azure 建構真實的雲端應用程式) (英文)](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry)。<br>
  針對追蹤 Azure 雲端應用程式所建議的電子書章節。
* [ASP.NET 追蹤](http://msdn.microsoft.com/library/ms972204.aspx)<br/>
  舊有但仍是該主題的基本簡介的良好資源。
* [追蹤接聽程式](http://msdn.microsoft.com/library/4y5y10s7.aspx)<br/>
  內含有關追蹤接聽程式的資訊，但是沒有提到 [WebPageTraceListener](http://msdn.microsoft.com/library/system.web.webpagetracelistener.aspx)。
* [逐步解說︰整合 ASP.NET 追蹤與 System.Diagnostics 追蹤](http://msdn.microsoft.com/library/b0ectfxd.aspx)<br/>
  本文同樣為舊有的資料，但是內含簡介文章沒有提到的一些額外資訊。
* [追蹤 ASP.NET MVC Razor 檢視](http://blogs.msdn.com/b/webdev/archive/2013/07/16/tracing-in-asp-net-mvc-razor-views.aspx)<br/>
  除了追蹤 Razor 檢視之外，該文同時說明了如何建立錯誤篩選條件以便記錄 MVC 應用程式所出現的所有未處理的例外。 如需如何記錄 Web Form 應用程式中所有未處理的例外項目的詳細資訊，請參閱 MSDN 上 [完整的錯誤處理常式範例](http://msdn.microsoft.com/library/bb397417.aspx) (英文) 的 Global.asax 範例。 無論是 MVC 還是 Web Form，如果您想要記錄特定例外，但是讓預設的架構處理功能生效，則您可以如以下範例所示捕捉並重新擲回這些例外：

``` c#
try
{
   // Your code that might cause an exception to be thrown.
}
catch (Exception ex)
{
    Trace.TraceError("Exception: " + ex.ToString());
    throw;
}
```

* [從 Azure 命令列串流診斷追蹤記錄 (加上 Glimpse！)](http://www.hanselman.com/blog/StreamingDiagnosticsTraceLoggingFromTheAzureCommandLinePlusGlimpse.aspx)<br/>
  如何使用命令列來執行本教學課程所示範的 Visual Studio 步驟。 [Glimpse](http://www.hanselman.com/blog/IfYoureNotUsingGlimpseWithASPNETForDebuggingAndProfilingYoureMissingOut.aspx) (英文) 工具可供您偵錯 ASP.NET 應用程式。
* [使用 Azure Web Apps 記錄和診斷功能 - 與 David Ebbo 合作](https://azure.microsoft.com/documentation/videos/azure-web-site-logging-and-diagnostics/)以及[來自 Web Apps 的串流記錄 - 與 David Ebbo 合作](https://azure.microsoft.com/documentation/videos/log-streaming-with-azure-web-sites/)<br>
  (英文) 影片，由 Scott Hanselman 與 David Ebbo 共同錄製。

針對錯誤記錄，做為撰寫自己的追蹤程式碼的替代方法，便是使用開放原始碼的記錄架構，例如 [ELMAH](http://nuget.org/packages/elmah/)。 如需詳細資訊，請參閱 [Scott Hanselman 關於 ELMAH 的部落格文章](http://www.hanselman.com/blog/NuGetPackageOfTheWeek7ELMAHErrorLoggingModulesAndHandlersWithSQLServerCompact.aspx)(英文)。

此外，您不需要使用 ASP.NET 或 `System.Diagnostics` 追蹤從 Azure 取得串流記錄。 Azure Web 應用程式串流記錄服務會串流它在 [LogFiles] 資料夾所找到的任何 *.txt*、*.html* 或 *.log* 檔案。 因此，您可以建立自己的記錄系統以寫入 Web 應用程式的檔案系統，而您的檔案會自動進行串流與下載。 您只需撰寫會在 d:\home\logfiles 資料夾中建立相關檔案的應用程式碼。

### <a name="analyzing-web-server-logs"></a>分析 Web 伺服器記錄
如需分析 Web 伺服器記錄的詳細資訊，請參閱下列資源：

* [LogParser](http://www.microsoft.com/download/details.aspx?id=24659)<br/>
  用於檢視 Web 伺服器記錄 (*.log* 檔案) 中資料的工具。
* [疑難排解 IIS 效能問題或使用 LogParser 的應用程式錯誤](http://www.iis.net/learn/troubleshoot/performance-issues/troubleshooting-iis-performance-issues-or-application-errors-using-logparser)<br/>
  此篇介紹可以用來分析 Web 伺服器記錄的 Log Parser 工具。
* [Robert McMurray 關於使用 LogParser 的部落格文章](http://blogs.msdn.com/b/robert_mcmurray/archive/tags/logparser/)<br/>
* [IIS 7.0、IIS 7.5 與 IIS 8.0 中的 HTTP 狀態碼](http://support.microsoft.com/kb/943891)

### <a name="analyzing-failed-request-tracing-logs"></a>分析失敗要求追蹤記錄
Microsoft TechNet 網站內的 [使用失敗要求追蹤](http://www.iis.net/learn/troubleshoot/using-failed-request-tracing) \(英文\) 小節可能有助您了解如何使用這些記錄。 不過，本文主要著重在 IIS 內設定失敗要求追蹤功能，這是您無法在 Azure Web Apps 中執行的功能。

[GetStarted]: app-service-web-get-started-dotnet.md
[GetStartedWJ]: https://github.com/Azure/azure-webjobs-sdk/wiki
