---
title: 建立 Azure 時間序列深入解析單一頁面 Web 應用程式
description: 了解如何建立會從 TSI 環境查詢及轉譯資料的單一頁面 Web 應用程式。
author: ashannon7
ms.service: time-series-insights
ms.topic: tutorial
ms.date: 06/14/2018
ms.author: anshan
manager: cshankar
ms.openlocfilehash: 312e15f976a6782e3f39cfcc5ce0721ac6357a16
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39626750"
---
# <a name="tutorial-create-an-azure-time-series-insights-single-page-web-app"></a>教學課程：建立 Azure 時間序列深入解析單一頁面 Web 應用程式

本教學課程將逐步引導您建立自己的單一頁面 Web 應用程式 (SPA) 來存取 TSI 資料 (仿造自[時間序列深入解析 (TSI) 範例應用程式](https://insights.timeseries.azure.com/clientsample))。 在本教學課程中，您將了解：

> [!div class="checklist"]
> * 應用程式設計
> * 如何向 Azure Active Directory (AD) 註冊您的應用程式
> * 如何建置、發佈和測試您的 Web 應用程式 

## <a name="prerequisites"></a>必要條件

如果您沒有帳戶，請註冊[免費 Azure 訂用帳戶](https://azure.microsoft.com/free/)。 

如果您尚未安裝 Visual Studio，您也必須加以安裝。 在本教學課程中，您可以[下載/安裝免費 Community 版本或免費試用版](https://www.visualstudio.com/downloads/)。

## <a name="application-design-overview"></a>應用程式設計概觀

如前所述，TSI 範例應用程式會提供本教學課程中使用的設計和程式碼的基礎。 該程式碼會使用 TSI 用戶端 JavaScript 程式庫。 TSI 用戶端程式庫會提供兩種主要 API 類別的概念：

- **用來呼叫 TSI 查詢 API 的包裝函式方法**：可讓您使用 JSON 型運算式來查詢 TSI 資料的 REST API。 這些方法會組織在程式庫的 `TsiClient.server` 命名空間下。
- **用來建立和填入數種圖表控制項類型的方法**：用來將網頁中的 TSI 資料視覺化的方法。 這些方法會組織在程式庫的 `TsiClient.ux` 命名空間下。

本教學課程也使用來自範例應用程式 TSI 環境的資料。 如需 TSI 範例應用程式的結構及其如何使用 TSI 用戶端程式庫的詳細資訊，請參閱[探索 Azure 時間序列深入解析 JavaScript 用戶端程式庫](tutorial-explore-js-client-lib.md)教學課程。

## <a name="register-the-application-with-azure-ad"></a>向 Azure AD 註冊應用程式 

在建置應用程式之前，您必須先向 Azure AD 註冊該應用程式。 註冊可提供應用程式的身分識別組態，讓它能夠使用 OAuth 支援進行單一登入。 要使用 OAuth，SPA 必須要使用「隱含」授權，您可以在應用程式資訊清單中加以更新。 應用程式資訊清單是應用程式身分識別組態的 JSON 表示法。 

1. 使用您的 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com)。  
1. 在左側導覽窗格中，依序選取 [Azure Active Directory] 資源、[應用程式註冊] 和 [+ 新增應用程式註冊]：  
   
   ![Azure 入口網站 Azure AD 應用程式註冊](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration.png)

1. 在 [建立] 頁面上填入必要參數：
   
   參數|說明
   ---|---
   **名稱** | 提供有意義的註冊名稱。  
   **應用程式類型** | 由於您要建置 SPA Web 應用程式，因此請保留為「Web 應用程式/API」。
   **登入 URL** | 輸入應用程式首頁/登入頁面的 URL。 由於應用程式將會裝載在 Azure App Service 中 (後續)，您必須在 "https://azurewebsites.net" 網域中使用 URL。 在此範例中，名稱會以註冊名稱為基礎。

   完成後，請按一下 [建立] 以建立新的應用程式註冊。

   ![Azure 入口網站 Azure AD 應用程式註冊 - 建立](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-create.png)

1. 資源應用程式會提供讓其他應用程式使用的 REST API，並且也會向 Azure AD 註冊。 API 可藉由公開「範圍」，提供對用戶端應用程式的細微/安全存取。 由於您的應用程式會呼叫「Azure 時間序列深入解析」API，因此您必須指定 API 和範圍，而其相關權限可在執行階段要求/授與。 依序選取 [設定]、[必要權限] 和 [+ 新增]：

   ![Azure 入口網站 Azure AD 新增權限](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-add-perms.png)

1. 在 [新增 API 存取權] 頁面中，按一下 [1 選取 API] 以指定 TSI API。 在 [選取 API] 頁面上，在 [搜尋] 欄位中輸入 "azure time"。 接著，在結果清單中選取「Azure 時間序列深入解析」API，然後按一下 [選取]： 

   ![Azure 入口網站 Azure AD 和權限 - API](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-add-perms-api.png)

1. 現在，您可以在 API 上指定範圍。 同樣在 [新增 API 存取權] 頁面上，按一下 [2 選取權限]。 在 [啟用存取] 頁面上，選取 [存取 Azure 時間序列深入解析服務] 範圍。 按一下 [選取]，此時您會返回 [新增 API 存取權] 頁面，請在其中按一下 [完成]：

   ![Azure 入口網站 Azure AD 和權限 - 範圍](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-add-perms-api-scopes.png)

1. 當您返回 [必要權限] 頁面時，請留意此時會列出「Azure 時間序列深入解析」API。 您也需要預先同意權限，應用程式才能存取所有使用者的 API 和範圍。 按一下最上方的 [授與權限] 按鈕，然後選取 [是]：

   ![Azure 入口網站 Azure AD 必要權限 - 同意](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-required-permissions-consent.png)

1. 如前所述，您也需要更新應用程式資訊清單。 按一下階層連結中的應用程式名稱，返回 [註冊的應用程式] 頁面。 選取 [資訊清單]，將 `oauth2AllowImplicitFlow` 屬性變更為 `true`，然後按一下 [儲存]：

   ![Azure 入口網站 Azure AD 更新資訊清單](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-update-manifest.png)

1. 最後，按一下階層連結重新回到 [註冊的應用程式] 頁面，並複製應用程式的 [首頁] URL 和 [應用程式識別碼] 屬性。 您將在後續步驟中使用這些屬性：

   ![Azure 入口網站 Azure AD 屬性](media/tutorial-create-tsi-sample-spa/ap-aad-app-registration-application.png)

## <a name="build-and-publish-the-web-application"></a>建置和發佈 Web 應用程式

1. 建立用來儲存應用程式專案檔的資料夾。 然後，瀏覽至下列各個 URL，以滑鼠右鍵按一下頁面右上方區域中的 [原始] 連結，並「另存新檔」到您的專案目錄：

   > [!NOTE]
   > 根據瀏覽器，您在儲存檔案前可能必須先更正副檔名 (變更為 HTML 或 CSS)。

   - **index.html**：頁面的 HTML 和 JavaScript：https://github.com/Microsoft/tsiclient/blob/tutorial/pages/tutorial/index.html
   - **sampleStyles.css**：CSS 樣式表：https://github.com/Microsoft/tsiclient/blob/tutorial/pages/tutorial/sampleStyles.css
    
1. 啟動並登入 Visual Studio，以建立 Web 應用程式的專案。 在 [檔案] 功能表上，選取 [開啟]、[網站] 選項。 在 [開啟網站] 對話方塊上，選取您用來儲存 HTML 和 CSS 檔案的工作目錄，然後按一下 [開啟]：

   ![VS - 檔案開啟網站](media/tutorial-create-tsi-sample-spa/vs-file-open-web-site.png)

1. 從 Visual Studio [檢視] 功能表中開啟 [方案總管]。 您應該會看到新的方案中包含網站專案 (地球圖示)，其中包含 HTML 和 CSS 檔案：

   ![VS - 方案總管新增解決方案](media/tutorial-create-tsi-sample-spa/vs-solution-explorer.png)

1. 您必須先在 **index.html** 中更新 JavaScript 程式碼的某些部分，才能夠發佈應用程式： 

   a. 首先，變更 `<head>` 元素中的 Javascript 和樣式表檔案參考的路徑。 在 Visual Studio 解決方案中開啟 **index.html** 檔案，並尋找 JavaScript 程式碼的以下幾行。 將 "PROD RESOURCE LINKS" 下方的三行取消註解，並註解化 "DEV RESOURCE LINKS" 下方的三行：
   
      [!code-javascript[head-sample](~/samples-javascript/pages/tutorial/index.html?range=2-20&highlight=10-13,15-18)]

      變更後的程式碼看起來應類似於下列範例：

      ```javascript
      <!-- PROD RESOURCE LINKS -->
      <link rel="stylesheet" type="text/css" href="sampleStyles.css"></link>
      <script src="https://unpkg.com/tsiclient@1.1.4/tsiclient.js"></script>
      <link rel="stylesheet" type="text/css" href="https://unpkg.com/tsiclient@1.1.4/tsiclient.css"></link>

      <!-- DEV RESOURCE LINKS -->
      <!-- <link rel="stylesheet" type="text/css" href="pages/samples/sampleStyles.css"></link>
      <script src="dist/tsiclient.js"></script>
      <link rel="stylesheet" type="text/css" href="dist/tsiclient.css"></link> -->
      ```

   b. 接著，將存取權杖邏輯變更為使用您新的 Azure AD 應用程式註冊。 分別變更下列 `clientID` 和 `postLogoutRedirectUri` 變數，使其使用您在[向 Azure AD 註冊應用程式](#register-the-application-with-azure-ad)一節中的步驟 9 複製的應用程式識別碼和首頁 URL：

      [!code-javascript[head-sample](~/samples-javascript/pages/tutorial/index.html?range=147-153&highlight=4-5)]

      變更後的程式碼看起來應類似於下列範例：

      ```javascript
      clientId: '8884d4ca-b9e7-403a-bd8a-366d0ce0d460',
      postLogoutRedirectUri: 'https://tsispaapp.azurewebsites.net',
      ``` 

   c. 編輯完成後，請儲存 **index.html**。

1. 現在，將 Web 應用程式發佈至您的 Azure 訂用帳戶作為 Azure App Service：  

   > [!NOTE]
   > 下列對話方塊上的數個欄位會填入您 Azure 訂用帳戶中的資料。 此時，每個對話方塊可能需要幾秒鐘才會完整載入，其後您才能繼續操作。  

   a. 在 [方案總管] 中，以滑鼠右鍵按一下網站專案節點，並選取 [發佈 Web 應用程式]：  

      ![VS - 方案總管發佈 Web 應用程式](media/tutorial-create-tsi-sample-spa/vs-solution-explorer-publish-web-app.png)

   b. 選取 [Microsoft Azure App Service] 以建立發佈目標：  

      ![VS - 發佈設定檔](media/tutorial-create-tsi-sample-spa/vs-publish-profile-target.png)  

   c. 選取您要用來發佈應用程式的訂用帳戶，然後按一下 [新增]： 

      ![VS - 發佈設定檔 - App Service](media/tutorial-create-tsi-sample-spa/vs-publish-profile-app-service.png)  

   d. 等候數秒讓 [建立 App Service] 對話方塊載入所有欄位，然後修改下列欄位：
   
      欄位 | 說明
      ---|---
      **應用程式名稱** | 變更為您在[向 Azure AD 註冊應用程式](#register-the-application-with-azure-ad)中的步驟 3 使用的 Azure AD 應用程式註冊名稱。 
      **資源群組** | 使用 [新增...] 按鈕進行變更，以符合 [應用程式名稱] 欄位。
      **App Service 方案** | 使用 [新增...] 按鈕進行變更，以符合 [應用程式名稱] 欄位。

      完成後，按一下 [建立]。 在建立 Azure 資源時，左下方的 [匯出] 按鈕會取代「部署中」數秒。 在資源建立後，您將會返回先前的對話方塊。 

      ![VS - 發佈設定檔 - 新增 App Service](media/tutorial-create-tsi-sample-spa/vs-publish-profile-app-service-create.png)  

   e. 當您返回 [發佈] 對話方塊時，請確定 [發佈方法] 設定為 [Web 部署]。 同時請將 [目的地 URL] 變更為使用 `https` (而不是 `http`)，以符合 Azure AD 應用程式註冊。 然後，按一下 [發佈] 以部署 Web 應用程式：  

      ![VS - 發佈 Web 應用程式 - 發佈 App Service](media/tutorial-create-tsi-sample-spa/vs-publish-publish.png)  

   f. 您應該會在 Visual Studio [輸出] 視窗中看到成功發佈記錄。 部署完成後，Visual Studio 也會在瀏覽器索引標籤中開啟 Web 應用程式，提示您進行登入。 成功登入後，您會看到所有 TSI 控制項皆已填入資料：  

      [![VS - 發佈 Web 應用程式 - 發佈記錄輸出](media/tutorial-create-tsi-sample-spa/vs-publish-output.png)](media/tutorial-create-tsi-sample-spa/vs-publish-output.png#lightbox)

      ![TSI SPA 應用程式 - 登入](media/tutorial-create-tsi-sample-spa/tsispaapp-azurewebsites-net.png)  

## <a name="troubleshooting"></a>疑難排解  

錯誤碼/情況 | 說明
---------------------| -----------
*AADSTS50011：未註冊應用程式的回覆位址。* | Azure AD 註冊遺漏「回覆 URL」屬性。 請移至 [設定] / [回覆 URL] 頁面檢視您的 Azure AD 應用程式註冊。 確認先前在[向 Azure AD 註冊應用程式](#register-the-application-with-azure-ad)的步驟 3 中指定的**登入** URL 存在於該處。 
*AADSTS50011：在要求中指定的回覆 URL 不符合為應用程式設定的回覆 URL：'<Application ID GUID>'。* | 在[建置和發佈 Web 應用程式](#build-and-publish-the-web-application)的步驟 4.b 中指定的 `postLogoutRedirectUri`，必須符合在您 Azure AD 應用程式中的 [設定] / [回覆 URL] 屬性下指定的值。 也請務必將 [目的地 URL] 變更為使用 `https` (根據 [建置和發佈 Web 應用程式](#build-and-publish-the-web-application)的步驟 5.e)。
Web 應用程式會載入，但使用無樣式的文字登入頁面，和白色背景。 | 請確認[建置和發佈 Web 應用程式](#build-and-publish-the-web-application)的步驟 4.a 中討論的路徑正確無誤。 如果 Web 應用程式找不到 .css 檔案，頁面將無法正確套用樣式。

## <a name="clean-up-resources"></a>清除資源

本教學課程建立了數個執行中的 Azure 服務。 如果您不打算完成本教學課程系列，建議您刪除所有資源，以避免產生不必要的成本。 

從 Azure 入口網站的左側功能表：

1. 按一下 [資源群組] 圖示，然後選取您為 TSI 環境建立的資源群組。 在頁面頂端按一下 [刪除資源群組]、輸入資源群組的名稱，然後按一下 [刪除]。 
1. 按一下 [資源群組] 圖示，然後選取裝置模擬解決方案加速器所建立的資源群組。 在頁面頂端按一下 [刪除資源群組]、輸入資源群組的名稱，然後按一下 [刪除]。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 應用程式設計
> * 如何向 Azure Active Directory (AD) 註冊您的應用程式
> * 如何建置、發佈和測試您的 Web 應用程式 

本教學課程與 Azure AD 整合，使用登入使用者的身分識別取得存取權杖。 若要了解如何使用服務/精靈應用程式的身分識別存取 TSI API，請參閱：

> [!div class="nextstepaction"]
> [Azure 時間序列深入解析 API 的驗證和授權](time-series-insights-authentication-and-authorization.md)
