---
title: 使用 Azure AD 驗證搭配 REST 存取 Azure 媒體服務 API | Microsoft Docs
description: 了解如何使用 Azure Active Directory 驗證搭配 REST 存取 Azure 媒體服務 API。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: willzhan; johndeu
ms.openlocfilehash: 28719046c9a8ccc65d231244ef8b5b3f8e116282
ms.sourcegitcommit: 77afc94755db65a3ec107640069067172f55da67
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98694725"
---
# <a name="use-azure-ad-authentication-to-access-the-media-services-api-with-rest"></a>使用 Azure AD 驗證搭配 REST 存取媒體服務 API

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> 媒體服務 v2 不會再新增任何新的特性或功能。 <br/>查看最新版本的[媒體服務 v3](../latest/index.yml)。 另請參閱[從 v2 變更為 v3 的移轉指導方針](../latest/migrate-v-2-v-3-migration-introduction.md)

使用 Azure AD 驗證搭配 Azure 媒體服務時，您可以下列其中一種方式進行驗證：

- **使用者驗證** 可驗證使用應用程式與 Azure 媒體服務資源互動的人員。 互動式應用程式應該會先提示使用者輸入認證。 例如，授權的使用者用來監控編碼工作或即時串流的管理主控台應用程式。 
- **服務主體驗證** 會驗證服務。 通常使用這種驗證方法的應用程式有執行精靈服務、中介層服務或排程的工作的應用程式：例如，Web 應用程式、函數應用程式、邏輯應用程式、API 或微服務。

    本教學課程會示範如何使用 Azure AD **服務主體** 驗證來存取使用 REST 的 AMS API。 

    > [!NOTE]
    > 對於大部分連線到 Azure 媒體服務的應用程式，**服務主體** 是建議的最佳做法。 

在本教學課程中，您會了解如何：

> [!div class="checklist"]
> * 從 Azure 入口網站取得驗證資訊
> * 使用 Postman 取得存取權杖
> * 使用存取權杖來測試 **資產** API


> [!IMPORTANT]
> 目前，媒體服務支援 Azure 存取控制服務驗證模型。 不過，存取控制驗證將在 2018 年 6 月 1 日被取代。 建議您儘速移轉至 Azure AD 驗證模型。

## <a name="prerequisites"></a>必要條件

- 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- [使用 Azure 入口網站建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)。
- 請先複習[使用 Azure AD 驗證存取 Azure 媒體服務 API 概觀](media-services-use-aad-auth-to-access-ams-api.md)一文。
- 安裝 [Postman](https://www.getpostman.com/) \(英文\) REST 用戶端，來執行在本文中示範的 REST API。 

    在此教學課程中，我們使用的是 **Postman**，但任何 REST 工具都適用。 其他替代方式為：搭配 REST 外掛程式的 **Visual Studio Code**，或 **Telerik Fiddler**。 

## <a name="get-the-authentication-information-from-the-azure-portal"></a>從 Azure 入口網站取得驗證資訊

### <a name="overview"></a>概觀

若要存取 Media Services API，您必須收集下列資料點。

|設定|範例|描述|
|---|-------|-----|
|Azure Active Directory 租用戶網域|microsoft.onmicrosoft.com|Azure AD 即 Secure Token Service (STS) 端點會透過以下格式建立：<https://login.microsoftonline.com/{your-ad-tenant-name.onmicrosoft.com}/oauth2/token>. Azure AD 會核發存取資源所需的 JWT (存取權杖)。|
|REST API 端點|<https://amshelloworld.restv2.westus.media.azure.net/api/>|您的應用程式中發出之所有媒體服務 REST API 呼叫，都是針對此端點。|
|用戶端識別碼 (應用程式識別碼)|f7fbbb29-a02d-4d91-bbc6-59a2579259d2|Azure AD 應用程式 (用戶端) 識別碼。 需要用戶端識別碼，才能取得存取權杖。 |
|用戶端密碼|+mUERiNzVMoJGggD6aV1etzFGa1n6KeSlLjIq+Dbim0=|Azure AD 應用程式金鑰 (用戶端秘密)。 需要用戶端秘密，才能取得存取權杖。|

### <a name="get-azure-active-directory-auth-info-from-the-azure-portal"></a>從 Azure 入口網站取得 Azure Active Directory auth 資訊

若要取得資訊，請遵循下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至您的 AMS 執行個體。
3. 選取 [ **API 存取**]。
4. 按一下 [使用服務主體連線到 Azure 媒體服務 API]。

    ![顯示從右窗格中選取 [媒體服務] 功能表和 [連接到 Azure 媒體服務 A P I with service principal] 的螢幕擷取畫面。](./media/connect-with-rest/connect-with-rest01.png)

5. 選取現有的 **Azure AD 應用程式** 或建立新的 Azure AD 應用程式 (如下所示)。

    > [!NOTE]
    > 若要讓 Azure 媒體 REST 要求成功，呼叫的使用者必須擁有其嘗試存取之媒體服務帳戶的「 **參與者** 」或「 **擁有** 者」角色。 如果您收到例外狀況指出「遠端伺服器傳回錯誤：(401) 未授權」，請參閱[存取控制](media-services-use-aad-auth-to-access-ams-api.md#access-control)。

    如果您需要建立新的 AD 應用程式，請依照下列步驟操作：
    
   1. 按下 [新建]。
   2. 輸入名稱。
   3. 再按一次 [新建]。
   4. 按下 [儲存] 。

      ![顯示 [建立新的] 對話方塊的螢幕擷取畫面，其中反白顯示 [建立應用程式] 文字方塊，並選取 [儲存] 按鈕。](./media/connect-with-rest/new-app.png)

      新的應用程式會顯示在頁面上。

6. 取得 **用戶端識別碼** (應用程式識別碼)。
    
   1. 選取應用程式。
   2. 從右側視窗取得 **用戶端識別碼**。 

      ![顯示已選取 [Azure A D 應用程式] 和 [管理應用程式] 的螢幕擷取畫面，並在右窗格中反白顯示 [用戶端 I D]。](./media/connect-with-rest/existing-client-id.png)

7. 取得應用程式的 **金鑰** (用戶端秘密)。 

   1. 按一下 [管理應用程式] 按鈕 (請注意，用戶端識別碼資訊位於 **應用程式識別碼** 底下)。 
   2. 按 [金鑰]。
    
       ![顯示已選取 [管理應用程式] 按鈕的螢幕擷取畫面，中間窗格中的 [應用程式 I D] 已反白顯示，並在右窗格中選取 [金鑰]。](./media/connect-with-rest/manage-app.png)
   3. 產生應用程式金鑰 (用戶端秘密)，方法是填入 **DESCRIPTION** 和 **EXPIRES**，然後按 [儲存]。
    
       一旦按下 [儲存] 按鈕後，隨即出現金鑰值。 離開刀鋒視窗之前，請複製金鑰值。

   ![API 存取](./media/connect-with-rest/connect-with-rest03.png)

您可以將 AD 連線參數的值新增至您的 web.config 或 app.config 檔案，以便稍後在您的程式碼中使用。

> [!IMPORTANT]
> **用戶端金鑰** 是重要的祕密，務必在金鑰保存庫中適當保護，或是在生產環境中加密。

## <a name="get-the-access-token-using-postman"></a>使用 Postman 取得存取權杖

本節示範如何使用 **Postman** 來執行 REST API，從而傳回 JWT 持有人權杖 (存取權杖)。 若要呼叫任何媒體服務 REST API，您必須將「授權」標頭新增至呼叫，並將 "Bearer *your_access_token*" 的值新增至每個呼叫 (如本教學課程的下一節所示)。 

1. 開啟 **Postman**。
2. 選取 [POST]  。
3. 輸入 URL，其中包含您的租用戶名稱，且使用下列格式：租用戶名稱結尾應為 **.onmicrosoft.com**，且 URL 結尾應為 **oauth2/token**： 

    `https://login.microsoftonline.com/{your-aad-tenant-name.onmicrosoft.com}/oauth2/token`

4. 選取 [標頭] 索引標籤。
5. 使用「索引鍵/值」資料格線輸入 **標頭** 資訊。 

    ![顯示已選取 [標頭] 索引標籤和 [大量編輯] 動作的螢幕擷取畫面。](./media/connect-with-rest/headers-data-grid.png)

    或者，按一下 Postman 視窗右邊的 [大量編輯] 連結，並將下列程式碼貼上。

    ```javascript
    Content-Type:application/x-www-form-urlencoded
    Keep-Alive:true
    ```

6. 按 [主體] 索引標籤。
7. 使用「索引鍵/值」資料格線 (取代用戶端識別碼和祕密值) 輸入主體資訊。 

    ![資料格線](./media/connect-with-rest/data-grid.png)

    或者，按一下 Postman 視窗右邊的 [大量編輯]，並將下列主體貼上 (取代用戶端識別碼和祕密值)：

    ```javascript
    grant_type:client_credentials
    client_id:{Your Client ID that you got from your Azure AD Application}
    client_secret:{Your client secret that you got from your Azure AD Application's Keys}
    resource:https://rest.media.azure.net
    ```

8. 按 [傳送]  。

    ![顯示 [貼文] 文字方塊、[標頭] 和 [主體] 索引標籤和 [access_token] 的螢幕擷取畫面，並已偵測到 [傳送] 按鈕。](./media/connect-with-rest/connect-with-rest04.png)

傳回的回應包含您存取任何 AMS API 時所需使用的 **存取權杖**。

## <a name="test-the-assets-api-using-the-access-token"></a>使用存取權杖來測試 **資產** API

本節示範如何使用 **Postman** 存取 **資產** API。

1. 開啟 **Postman**。
2. 選取 [ **取得**]。
3. 貼上 REST API 端點 (例如 https://amshelloworld.restv2.westus.media.azure.net/api/Assets)
4. 選取 [授權] 索引標籤。 
5. 選取 [持有人權杖]。
6. 將上一節中建立的權杖貼上。

    ![取得權杖](./media/connect-with-rest/connect-with-rest05.png)

    > [!NOTE]
    > Mac 與 PC 之間的 Postman UX 可能有所不同。 如果 Mac 版本 [驗證] 區段下拉式清單中沒有「持有人權杖」選項，您應該在 Mac 用戶端上手動新增 [授權] 標頭。

   ![授權標頭](./media/connect-with-rest/auth-header.png)

7. 選取 [標頭]。
5. 按一下 Postman 視窗右邊的 [大量編輯] 連結。
6. 將下列標頭貼上：

    ```javascript
    x-ms-version:2.19
    Accept:application/json
    Content-Type:application/json
    DataServiceVersion:3.0
    MaxDataServiceVersion:3.0
    ```

7. 按 [傳送]  。

傳回的回應會包含您帳戶中的資產。

## <a name="next-steps"></a>後續步驟

* 在[存取 Azure 媒體服務要進行的 Azure AD 驗證：均透過 REST API](https://github.com/willzhan/WAMSRESTSoln) \(英文\) 中嘗試此範例程式碼
* [使用 .NET 上傳檔案](media-services-dotnet-upload-files.md)
