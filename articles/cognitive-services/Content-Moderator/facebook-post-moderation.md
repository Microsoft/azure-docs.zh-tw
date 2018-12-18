---
title: 教學課程：Facebook 內容仲裁 - Azure Content Moderator
titlesuffix: Azure Cognitive Services
description: 使用 Content Moderator 來仲裁 Facebook 頁面。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: tutorial
ms.date: 09/18/2017
ms.author: sajagtap
ms.openlocfilehash: ead8c1d445bf32ecaaf236b4e73c2a583c755049
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223933"
---
# <a name="tutorial-facebook-content-moderation-with-content-moderator"></a>教學課程：使用 Content Moderator 進行 Facebook 內容仲裁

在本教學課程中，我們將了解如何使用機器學習型 Content Moderator 來協助仲裁 Facebook 文章和留言。

本教學課程會引導您完成下列步驟：

1. 建立 Content Moderator 小組。
2. 建立接聽來自 Content Moderator 和 Facebook 之 HTTP 事件的 Azure Functions。
3. 建立 Facebook 頁面和應用程式，然後將其連線至 Content Moderator。

完成之後，Facebook 就會將訪客張貼的內容傳送給 Content Moderator。 您的 Content Moderator 工作流程會根據相符閾值，發佈內容或在審核工具內建立審核項目。 

下圖顯示此解決方案的構成要素。

![Facebook 文章調節](images/tutorial-facebook-moderation.png)

## <a name="create-a-content-moderator-team"></a>建立 Content Moderator 小組

請參考[快速入門](quick-start.md)頁面，以註冊 Content Moderator 並建立小組。

## <a name="configure-image-moderation-workflow-threshold"></a>設定影像仲裁工作流程 (閾值)

請參考[工作流程](review-tool-user-guide/workflows.md)頁面，以設定自訂影像工作流程 (閾值)。 請記下工作流程**名稱**。

## <a name="3-configure-text-moderation-workflow-threshold"></a>3.設定文字仲裁工作流程 (閾值)

請使用與[工作流程](review-tool-user-guide/workflows.md)頁面類似的步驟，來設定自訂文字閾值和工作流程。 請記下工作流程**名稱**。

![設定文字工作流程](images/text-workflow-configure.PNG)

使用 [執行工作流程] 按鈕來測試工作流程。

![測試文字工作流程](images/text-workflow-test.PNG)

## <a name="create-azure-functions"></a>建立 Azure Functions

登入 [Azure 管理入口網站](https://portal.azure.com/)，以建立您自己的 Azure Functions。 請遵循下列步驟：

1. 建立「Azure 函數應用程式」，如[Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-create-function-app-portal)所示。
2. 開啟新建立的「函數應用程式」。
3. 在應用程式內，瀏覽至 [平台功能] > [應用程式設定]
4. 定義下列[應用程式設定](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings#settings)：

> [!NOTE]
> **cm:Region** 應該是區域的名稱 (不含任何空格)。
> 例如 **westeurope** 而非 West Europe、**westcentralus** 而非 West Central US 等等。
>

| 應用程式設定 | 說明   | 
| -------------------- |-------------|
| cm:TeamId   | 您的 Content Moderator TeamId  | 
| cm:SubscriptionKey | 您的 Content Moderator 訂用帳戶金鑰 - 請參閱 [Credentials](review-tool-user-guide/credentials.md) | 
| cm:Region | 您的 Content Moderator 區域名稱 (不含空格)。 請參閱上面的注意事項。 |
| cm:ImageWorkflow | 要在「影像」上執行之工作流程的名稱 |
| cm:TextWorkflow | 要在「文字」上執行之工作流程的名稱 |
| cm:CallbackEndpoint | 您稍後在本指南中所建立「CMListener 函數應用程式」的 URL |
| fb:VerificationToken | 祕密權杖，此權杖也用來訂閱 Facebook 摘要事件 |
| fb:PageAccessToken | Facebook 圖形 API 存取權杖永不過期，可讓函式代表您「隱藏/刪除」文章。 |

5. 建立名為 **FBListener** 的新 **HttpTrigger-CSharp** 函式。 此函式會從 Facebook 接收事件。 請依照下列步驟建立此函式：

    1. 將 [Azure Functions 建立](https://docs.microsoft.com/azure/azure-functions/functions-create-function-app-portal) 頁面保持開啟以供參考。
    2. 按一下 [+ 新增]以建立新函式。
    3. 選擇 [自行開始使用/自訂函數] 選項，而不選擇內建範本。
    4. 按一下顯示 **HttpTrigger-CSharp** 的圖格。
    5. 輸入名稱 **FBListener**。 [授權等級] 欄位應該設定為 [函式]。
    6. 按一下頁面底部的 [新增] 。
    7. 使用來自 [**FbListener/run.csx**](https://github.com/MicrosoftContentModerator/samples-fbPageModeration/blob/master/FbListener/run.csx) 的內容取代 **run.csx** 的內容。

6. 建立名為 **CMListener** 的新 **HttpTrigger-CSharp** 函式。 此函式會從 Content Moderator 接收事件。 請依照下列步驟來建立此函式。

    1. 將 [Azure Functions 建立](https://docs.microsoft.com/azure/azure-functions/functions-create-function-app-portal) 頁面保持開啟以供參考。
    2. 按一下 [+ 新增]以建立新函式。
    3. 選擇 [自行開始使用/自訂函數] 選項，而不選擇內建範本。
    4. 按一下顯示 **HttpTrigger-CSharp** 的圖格
    5. 輸入名稱 **CMListener**。 [授權等級] 欄位應該設定為 [函式]。
    6. 按一下頁面底部的 [新增] 。
    7. 使用來自 [**CMListener/run.csx**](https://github.com/MicrosoftContentModerator/samples-fbPageModeration/blob/master/CmListener/run.csx) 的內容取代 **run.csx** 的內容。

## <a name="configure-the-facebook-page-and-app"></a>設定 Facebook 頁面和應用程式
1. 建立 Facebook 應用程式。

    1. 瀏覽至 [Facebook 開發人員入口網站](https://developers.facebook.com/)
    2. 按一下 [我的應用程式]。
    3. 新增應用程式。
    4. 選取 [Webhooks] > [開始使用]
    5. 選取 [Page] \(頁面\) -> [Subscribe to this topic] \(訂閱此主題\)
    6. 提供 **FBListener Url** 作為 [回呼網址]，並提供您已在 [函數應用程式設定] 底下設定的 [驗證權杖]
    7. 訂閱之後，向下捲動以摘要並選取 [訂閱]。

2. 建立 Facebook 頁面。

    1. 瀏覽至 [Facebook](https://www.facebook.com/bookmarks/pages)，然後建立一個**新的 Facebook 頁面**。
    2. 依照下列步驟來允許「Facebook 應用程式」存取此頁面：
        1. 瀏覽至[圖形 API 測試工具](https://developers.facebook.com/tools/explorer/)。
        2. 選取 [應用程式]。
        3. 選取 [粉絲專頁存取權杖]，傳送一個 **Get** 要求。
        4. 按一下回應中的 [粉絲專頁 ID]。
        5. 現在，將 **/subscribed_apps** 附加至 URL，然後傳送一個 **Get** (空回應) 要求。
        6. 提交一個 **Post** 要求。 您會得到像是 **success: true** 的回應。

3. 建立永不過期的「圖形 API」存取權杖。

    1. 瀏覽至[圖形 API 測試工具](https://developers.facebook.com/tools/explorer/)。
    2. 選取 [應用程式] 選項。
    3. 選取 [取得用戶存取權杖] 選項。
    4. 在 [選擇權限] 底下，選取 [manage_pages] 和 [publish_pages] 選項。
    5. 我們將在下一個步驟中使用**存取權杖** (短期權杖)。

4. 我們會在接下來幾個步驟中使用 Postman。

    1. 開啟 **Postman** (或從[這裡](https://www.getpostman.com/)取得)。
    2. 匯入兩個檔案：
        1. [Postman 集合](https://github.com/MicrosoftContentModerator/samples-fbPageModeration/blob/master/Facebook%20Permanant%20Page%20Access%20Token.postman_collection.json)
        2. [Postman 環境](https://github.com/MicrosoftContentModerator/samples-fbPageModeration/blob/master/FB%20Page%20Access%20Token%20Environment.postman_environment.json)       
    3. 更新下列環境變數：
    
    | Key | 值   | 
    | -------------------- |-------------|
    | appId   | 在這裡插入您的「Facebook 應用程式編號」  | 
    | appSecret | 在這裡插入您「Facebook 應用程式」的祕密 | 
    | short_lived_token | 插入您在上一個步驟中產生的短期用戶存取權杖 |
    4. 現在，執行集合中所列的 3 個 API： 
        1. 選取 [Generate Long-Lived Access Token] \(產生長期存取權杖\) ，然後按一下 [Send] \(傳送\)。
        2. 選取 [Get User ID] \(取得使用者識別碼)，然後按一下 [Send] \(傳送\)。
        3. 選取 [Get Permanent Page Access Token] \(取得永久頁面存取權杖\) ，然後按一下 [Send] \(傳送\)。
    5. 從回應中複製 **access_token** 值，然後將其指派給 [fb:PageAccessToken] 應用程式設定。

就這麼簡單！

此解決方案會將張貼在您 Facebook 頁面上的所有影像和文字傳送給 Content Moderator。 將會叫用您先前設定的工作流程。 未通過工作流程中所定義準則的內容，會導致在審核工具中建立審核項目。 其餘內容則會發佈。

## <a name="license"></a>授權

所有 Microsoft 認知服務 SDK 和範例都是透過 MIT 授權來獲得授權的。 如需詳細資料，請參閱[授權](https://microsoft.mit-license.org/) \(英文\)。

## <a name="developer-code-of-conduct"></a>開發人員行為準則

使用「認知服務」(包括此用戶端程式庫和範例) 的開發人員應當遵守 http://go.microsoft.com/fwlink/?LinkId=698895 中的「Microsoft 認知服務的開發人員行為準則」。

## <a name="next-steps"></a>後續步驟

1. [觀看這個來自 Microsoft Build 2017 的解決方案示範 (影片)](https://channel9.msdn.com/Events/Build/2017/T6033) \(英文\)。
1. [Github 上的 Facebook 範例](https://github.com/MicrosoftContentModerator/samples-fbPageModeration) \(英文\)
1. https://docs.microsoft.com/azure/azure-functions/functions-create-github-webhook-triggered-function
2. http://ukimiawz.github.io/facebook/2015/08/12/webhook-facebook-subscriptions/
3. http://stackoverflow.com/questions/17197970/facebook-permanent-page-access-token
