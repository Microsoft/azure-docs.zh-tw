---
title: 使用 Azure Application Insights 診斷執行階段例外狀況 | Microsoft Docs
description: 使用 Azure Application Insights 來尋找並診斷應用程式執行階段例外狀況的教學課程。
services: application-insights
keywords: ''
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/19/2017
ms.service: application-insights
ms.custom: mvc
ms.topic: tutorial
manager: carmonm
ms.openlocfilehash: f45cb6a47756fae7b75d8c3df80a0bc754742063
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298994"
---
# <a name="find-and-diagnose-run-time-exceptions-with-azure-application-insights"></a>使用 Azure Application Insights 尋找並診斷執行階段例外狀況

Azure Application Insights 會從您的應用程式收集遙測，以協助識別並診斷執行階段例外狀況。  本教學課程會引導您對應用程式完成此程序。  您會了解如何：

> [!div class="checklist"]
> * 修改您的專案，以啟用例外狀況追蹤
> * 識別應用程式不同元件的例外狀況
> * 檢視例外狀況詳細資料
> * 將例外狀況的快照集下載至 Visual studio 進行偵錯
> * 使用查詢語言分析失敗要求的詳細資料
> * 建立新的工作項目，以更正錯誤程式碼


## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

- 使用下列工作負載安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)：
    - ASP.NET 和 Web 開發
    - Azure 開發
- 下載並安裝 [Visual Studio 快照集偵錯工具](http://aka.ms/snapshotdebugger) \(英文\)。
- 啟用 [Visual Studio 快照集偵錯工具](https://docs.microsoft.com/azure/application-insights/app-insights-snapshot-debugger)
- 將 .NET 應用程式部署至 Azure，並[啟用 Application Insights SDK](app-insights-asp-net.md)。 
- 本教學課程會追蹤您應用程式中的例外狀況識別情形，因此，請修改您開發或測試環境中的程式碼以產生例外狀況。 

## <a name="log-in-to-azure"></a>登入 Azure
在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。


## <a name="analyze-failures"></a>分析失敗
Application Insights 會收集應用程式中的任何失敗，並可讓您檢視錯誤在不同作業間的發生頻率，以協助您專注於影響最大的部分。  您可以再向下鑽研這些失敗的詳細資料，以識別根本原因。   

1. 選取 [Application Insights]，然後選取您的訂用帳戶。  
2. 若要開啟 [失敗] 面板中，請選取 [調查] 功能表下方的 [失敗]，或按一下 [失敗的要求] 圖形。

    ![失敗的要求](media/app-insights-tutorial-runtime-exceptions/failed-requests.png)

3. 針對應用程式的每個作業，[失敗的要求] 面板會顯示失敗的要求計數和受影響的使用者數目。  您可以依使用者排序這項資訊，以識別對使用者影響最大的失敗。  在此範例中，**GET Employees/Create** 和 **GET Customers/Details** 是可能的調查候選項目，因其失敗的數目和受影響的使用者為數眾多。  選取作業會在右面板顯示關於此作業的進一步資訊。

    ![[失敗的要求] 面板](media/app-insights-tutorial-runtime-exceptions/failed-requests-blade.png)

4. 縮小時間範圍，以將失敗率突然增加的期間放大顯示。

    ![[失敗的要求] 視窗](media/app-insights-tutorial-runtime-exceptions/failed-requests-window.png)

5. 按一下附有篩選結果數目的按鈕，以查看相關範例。 「建議」範例會包含來自所有元件的相關遙測資料，儘管取樣的標的可能是其中任一元件。 按一下搜尋結果可查看失敗的詳細資料。

    ![失敗的要求範例](media/app-insights-tutorial-runtime-exceptions/failed-requests-search.png)

6. 失敗要求的詳細資料會顯示甘特圖，說明此交易中有兩個相依性失敗，而這也造就了該交易 50% 以上的總持續時間。 此體驗顯示了所有遙測資料，來源包括與此作業識別碼相關的分散式應用程式的各個元件。 [深入了解新的體驗](app-insights-transaction-diagnostics.md)。 您可以選取任何項目，以在右側查看其詳細資料。 

    ![失敗的要求詳細資料](media/app-insights-tutorial-runtime-exceptions/failed-request-details.png)

7. 作業詳細資料亦顯示 FormatException 似乎是導致失敗的原因。  您可以看到這是因為郵遞區號不正確。 您可以在 Visual Studio 中開啟偵錯快照集，以查看程式碼層級的偵錯資訊。

    ![例外狀況詳細資料](media/app-insights-tutorial-runtime-exceptions/failed-requests-exception.png)

## <a name="identify-failing-code"></a>識別失敗的程式碼
快照集偵錯工具會收集您應用程式中最常見之例外狀況的快照集，以協助您診斷生產環境中的根本原因。  您可以檢視入口網站中的偵錯快照集，以查看呼叫堆疊並檢查每個呼叫堆疊框架的變數。 您可接著下載該快照集並在 Visual Studio 2017 中開啟，以對原始程式碼進行偵錯。

1. 在例外狀況的內容中，按一下 [開啟偵錯快照集]。
2. [偵錯快照集] 面板隨即開啟，並顯示要求的呼叫堆疊。  按一下任一方法，即可檢視所有區域變數在要求時的值。  從這個範例最上方的方法開始，我們可以看到沒有值的區域變數。

    ![偵錯快照集](media/app-insights-tutorial-runtime-exceptions/debug-snapshot-01.png)

4. 具有有效值的第一個呼叫是 **ValidZipCode**，我們可以看到提供的郵遞區號含有無法轉譯成整數的字母。  這似乎是程式碼中需要修正的錯誤。

    ![偵錯快照集](media/app-insights-tutorial-runtime-exceptions/debug-snapshot-02.png)

5. 若要將此快照集下載至 Visual studio，以在其中尋找需要修正的實際程式碼，請按一下 [下載快照集]。
6. 快照集就會載入到 Visual Studio。
7. 您可以立即在 Visual Studio 中執行偵錯工作階段，以快速地識別造成例外狀況的程式碼。

    ![程式碼中的例外狀況](media/app-insights-tutorial-runtime-exceptions/exception-code.png)


## <a name="use-analytics-data"></a>使用分析資料
Application Insights 收集的所有資料都會儲存在 Azure Log Analytics 中，Azure Log Analytics 提供豐富的查詢語言，讓您可以各種方式分析資料。  使用這項資料，就能分析產生我們目前所研究之例外狀況的要求。 

8. 按一下程式碼上方的 CodeLens 資訊，可檢視 Application Insights 所提供的遙測。

    ![代碼](media/app-insights-tutorial-runtime-exceptions/codelens.png)

9. 按一下 [分析影響] 以開啟 Application Insights 分析。  它已填入數個查詢，提供失敗要求的詳細資料，例如受影響的使用者、瀏覽器和區域。<br><br>![分析](media/app-insights-tutorial-runtime-exceptions/analytics.png)<br>

## <a name="add-work-item"></a>新增工作項目
如果您將 Application Insights 與追蹤系統 (例如 Azure DevOps 或 GitHub) 連結，即可直接從 Application Insights 建立工作項目。

1. 返回 Application Insights 中的 [例外狀況屬性] 面板。
2. 按一下 [新增工作項目]。
3. [新增工作項目] 面板隨即開啟，並顯示關於例外狀況的詳細資料。  您可以新增任何額外的資訊，然後再將它儲存。

    ![新增工作項目](media/app-insights-tutorial-runtime-exceptions/new-work-item.png)

## <a name="next-steps"></a>後續步驟
由於您已學會如何識別執行階段例外狀況，請前進到下一個教學課程，了解如何識別並診斷效能問題。

> [!div class="nextstepaction"]
> [識別效能問題](app-insights-tutorial-performance.md)