---
title: 使用對話學習模組管理使用者資料 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何使用對話學習模組管理使用者資料。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: f9de4377857188a8cf483321654fb857e428c7f5
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171634"
---
# <a name="managing-user-data"></a>管理使用者資料

此頁面說明對話學習模組雲端服務在與終端使用者對話時記錄的內容。  這也說明如何使對話學習模組記錄與使用者識別碼產生關聯，以便您能夠擷取或刪除與特定使用者相關聯的所有記錄。

## <a name="overview-of-end-user-data-logging"></a>使用者資料記錄的概觀

對話學習模組雲端服務預設會記錄終端使用者與您的聊天機器人之間的互動。  這些記錄對於改進您的聊天機器人很重要，能夠讓您識別聊天機器人擷取錯誤實體或選取錯誤動作的情況。  然後您可以移至 UI 的 [Log Dialogs]\(記錄對話\) 進行修正，並將此修正的對話另存為新的訓練對話，以便修正這些錯誤。 如需詳細資訊，請參閱「記錄對話」的教學課程。

## <a name="how-to-disable-logging"></a>如何停用記錄

您可以控制與終端使用者進行的對話是否會出現在對話學習模組模型的 [設定] 頁面上。  有 [記錄對話] 的核取方塊。  取消核取此方塊時，將不會記錄與使用者的對話。

## <a name="what-is-logged"></a>會記錄什麼內容 

在記錄對話中，對話學習模組會儲存每次對話的使用者輸入、實體值、選取的動作和時間戳記。  這些記錄會在儲存一段時間後捨棄 (如需詳細資訊，請參閱「預設值和界限」的說明頁面)。  

對話學習模組會對於每次記錄的對話建立唯一識別碼。  對話學習模組*不*會對於記錄的對話儲存使用者識別碼。  

## <a name="associating-logged-dialogs-with-a-user-id"></a>使記錄的對話與使用者識別碼產生關聯

能夠將記錄的對話與使用者的識別碼產生關聯很重要，例如，能夠擷取或刪除對於特定使用者記錄的對話。  由於對話學習模組不會儲存使用者識別碼，因此這個關聯必須由開發人員的程式碼維護。  

若要建立這個對應，請在 `EntityDetectionCallback` 中對於記錄的對話取得其識別碼，然後，在聊天機器人的儲存體中，儲存使用者識別碼與此已記錄對話之間的關聯。  

```
cl.EntityDetectionCallback(async (text: string, memoryManager: ClientMemoryManager): Promise<void> => {

    // Get user and session info
    var sessionData = memoryManager.SessionInfo();

    // sessionData.sessionId is the ID of this logged dialog.
    // In your bot-specific datastore, store an association
    // bewteen your user identifier and this session ID.
    console.log(sessionData.logDialogId)

    // sessionData.userId and sessionData.userName are the 
    // user ID and user name as defined by the channel the user
    // is on (eg, Skype, Teams, Facebook Messenger, etc.).
    // For some channels, this will be undefined.
    // This information is NOT stored with the logged dialog.
    console.log(sessionData.userId);
    console.log(sessionData.userName);

})
```

## <a name="headers-for-http-calls"></a>HTTP 呼叫的標頭

在以下的每個 HTTP 呼叫中，加入下列標頭：

```
Ocp-Apim-Subscription-Key=<LUIS_AUTHORING_KEY>
```

其中 `<LUIS_AUTHORING_KEY>` 是 LUIS 撰寫金鑰，用於存取對話學習模組應用程式。

## <a name="how-to-obtain-raw-data-for-a-logged-dialog"></a>如何取得記錄對話的未經處理資料

若要取得記錄對話的未經處理資料，您可以使用這個 HTTP 呼叫：

```
GET https://westus.api.cognitive.microsoft.com/conversationlearner/v1.0/app/<appId>/logdialog/<logDialogId>
```

其中 `<appId>` 是此對話學習模組模型的 GUID，而 `<logDialgoId>` 是您要擷取的記錄對話識別碼。  

> [!NOTE]
> 開發人員可能會編輯記錄對話，然後另存為定型對話。  完成後，對話學習模組會連同訓練對話儲存「來源」記錄識別碼。  此外，訓練對話可在 UI 中「分支」；如果訓練對話有相關聯的來源記錄對話識別碼，則訓練對話的分支將標示相同的記錄對話識別碼。

若要取得記錄對話衍生的所有訓練對話，請遵循下列步驟。

首先，擷取所有訓練對話：

```
GET https://westus.api.cognitive.microsoft.com/conversationlearner/v1.0/app/<appId>/traindialogs
```

其中 `<appId>` 是此對話學習模組模型的 GUID。  

這會傳回所有訓練對話。  搜尋此清單中相關聯的 `sourceLogDialogId`，並記下相關聯的 `trainDialogId`。 

對於依識別碼的單一訓練對話：

```
GET https://westus.api.cognitive.microsoft.com/conversationlearner/v1.0/app/<appId>/traindialog/<trainDialogId>
```

其中 `<appId>` 是此對話學習模組模型的 GUID，而 `<trainDialogId>` 是您要擷取的定型對話識別碼。  

## <a name="how-to-delete-a-logged-dialog"></a>如何刪除記錄對話

如果您想要刪除特定識別碼的記錄對話，您可以使用這個 HTTP 呼叫：

```
DELETE https://westus.api.cognitive.microsoft.com/conversationlearner/v1.0/app/<appId>/logdialog/<logDialogId>
```

其中 `<appId>` 是此對話學習模組模型的 GUID，而 `<logDialogId>` 是您要刪除的記錄對話識別碼。 

如果您想要刪除特定識別碼的訓練對話，您可以使用這個 HTTP 呼叫：

```
DELETE https://westus.api.cognitive.microsoft.com/conversationlearner/v1.0/app/<appId>/traindialog/<trainDialogId>
```

其中 `<appId>` 是此對話學習模組模型的 GUID，而 `<trainDialogId>` 是您要刪除的定型對話識別碼。 
