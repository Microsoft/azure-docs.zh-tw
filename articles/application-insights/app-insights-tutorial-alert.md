---
title: 透過 Azure Application Insights 傳送警示 | Microsoft Docs
description: 使用 Azure Application Insights 傳送警示以回應應用程式錯誤的教學課程。
keywords: ''
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/20/2017
ms.service: application-insights
ms.custom: mvc
ms.topic: tutorial
manager: carmonm
ms.openlocfilehash: 39e2f136e30ebb6dcfc003c435382f3384af1052
ms.sourcegitcommit: e462e5cca2424ce36423f9eff3a0cf250ac146ad
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/01/2017
ms.locfileid: "23659341"
---
# <a name="monitor-and-alert-on-application-health-with-azure-application-insights"></a>使用 Azure Application Insights 監視應用程式健康情況並傳送警示

Azure Application Insights 可讓您監視您的應用程式，並在應用程式無法使用、發生失敗或有效能問題時傳送警示給您。  本教學課程會引導您完成以下程序：建立測試以持續檢查您應用程式的可用性，並傳送不同類型警示以回應偵測到的問題。  您會了解如何：

> [!div class="checklist"]
> * 建立可用性測試，以持續檢查應用程式的回應
> * 發生問題時，傳送郵件給系統管理員
> * 建立以效能計量為依據的警示 
> * 使用邏輯應用程式，依排程傳送摘要的遙測。


## <a name="prerequisites"></a>先決條件

若要完成本教學課程：

- 使用下列工作負載安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)：
    - ASP.NET 和 Web 開發
    - Azure 開發
    - 將 .NET 應用程式部署至 Azure，並[啟用 Application Insights SDK](app-insights-asp-net.md)。 


## <a name="log-in-to-azure"></a>登入 Azure
登入 Azure 入口網站，網址為 [https://portal.azure.com](https://portal.azure.com)。

## <a name="create-availability-test"></a>建立可用性測試
Application Insights 中的可用性測試，可讓您從世界各地的不同位置自動測試您的應用程式。   在本教學課程中，您會執行簡單的測試，以確定應用程式可以使用。  您也可以建立完整的檢測程序來測試詳細作業。 

1. 選取 [Application Insights]，然後選取您的訂用帳戶。  
1. 選取 [調查] 功能表下方的 [可用性]，然後按一下 [加入測試]。
 
    ![加入可用性測試](media/app-insights-tutorial-alert/add-test.png)

2. 輸入測試的名稱，並保留其他預設值。  這樣會每隔 5 分鐘從 5 個不同的地理位置要求應用程式的首頁。 
3. 選取 [警示] 以開啟 [警示] 面板，您可在該處定義測試失敗時如何回應的詳細資料。 輸入符合警示準則時要傳送到哪個電子郵件地址。  您也可以輸入符合警示準則時要呼叫的 Webhook 位址。

    ![建立測試](media/app-insights-tutorial-alert/create-test.png)
 
4. 返回測試面板，幾分鐘您應該會開始看到可用性測試的結果。  按一下測試名稱以檢視來自每個位置的詳細資料。  此散佈圖顯示成功的測試和每個測試的持續時間。

    ![測試詳細資料](media/app-insights-tutorial-alert/test-details.png)

5.  您可以按一下散佈圖上的任一點，以向下鑽研至該測試的詳細資料。  下列範例顯示失敗要求的詳細資料。

    ![測試結果](media/app-insights-tutorial-alert/test-result.png)
  
6. 如果符合警示準則，將傳送類似以下的郵件至您指定的位址。

    ![警示郵件](media/app-insights-tutorial-alert/alert-mail.png)


## <a name="create-an-alert-from-metrics"></a>建立以計量為依據的警示
除了傳送以可用性測試為依據的警示，還可以依據應用程式所收集的任何效能計量來建立警示。

2. 從 [設定] 功能表選取 [警示]。  這樣會開啟 Azure [警示] 面板。  此處可能有針對其他服務設定的其他警示規則。
3. 按一下 [新增計量警示]。  這樣會開啟建立新警示規則的面板。

    ![新增計量警示](media/app-insights-tutorial-alert/add-metric-alert.png)

4. 輸入警示規則的 [名稱]，然後在 [資源] 下拉式清單中選取您的應用程式。
5. 選取一個 [計量] 進行抽樣。  隨即顯示一個圖形，指示此要求在過去 24 小時的值。  這可協助您設定計量的條件。

    ![新增警示規則](media/app-insights-tutorial-alert/add-alert-01.png)

6. 指定警示的 [條件] 和 [閾值]。 這是建立警示前必須超過計量的次數。 
6. 在 [透過下列方式通知] 下方，核取 [將電子郵件寄給擁有者、參與者及讀者] 方塊，以在符合警示條件時傳送電子郵件給這些使用者，並加入任何額外的收件者電子郵件地址。  您也可以在此指定符合條件時執行的 Webhook 或邏輯應用程式。  採用這些方式可用來減緩偵測到的問題 

    ![新增警示規則](media/app-insights-tutorial-alert/add-alert-02.png)


## <a name="proactively-send-information"></a>主動傳送資訊
建立警示可依據在您應用程式中發現的特定問題作回應，通常只會針對需要立即注意的重要問題傳送警示。  您可透過依排程自動執行的邏輯應用程式，主動接收關於應用程式的資訊。  例如，您可以每天傳送郵件給系統管理員，其中包含需進一步評估的摘要資訊。

如需使用 Application Insights 建立邏輯應用程式的詳細資訊，請參閱[使用 Logic Apps 自動執行 Application Insights 程序](automate-with-logic-apps.md)

## <a name="next-steps"></a>後續步驟
由於您已學會如何針對問題傳送警示，請前進到下一個教學課程，了解如何分析使用者和您應用程式的互動方式。

> [!div class="nextstepaction"]
> [了解使用者](app-insights-tutorial-users.md)