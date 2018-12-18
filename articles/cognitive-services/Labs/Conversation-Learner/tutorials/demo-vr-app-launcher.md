---
title: 示範對話學習模組模型：虛擬實境應用程式啟動器 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解如何建立示範對話學習模組模型。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 896ec007c03e30e5c20a5344430be040271bc00b
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171138"
---
# <a name="demo-virtual-reality-app-launcher"></a>示範：虛擬實境應用程式啟動器

本示範說明虛擬實境模型啟動器，可支援「啟動 Skype 並放在牆上」之類的命令。 使用者需要說出應用程式名稱和位置，才能啟動應用程式。 模型啟動會由 API 呼叫來處理。 辨識出使用者所說的應用程式名稱時，entityDetectionCallback 會檢查要求的應用程式是否符合已安裝應用程式清單上的一個或多個應用程式。 如此即可處理所要求的應用程式並未安裝的情況，以及應用程式名稱不明確 (符合多個已安裝的應用程式) 的情況。

## <a name="video"></a>影片

[![示範 VR 應用程式預覽](http://aka.ms/cl-demo-vrapp-preview)](http://aka.ms/blis-demo-vrapp)

## <a name="requirements"></a>需求

本教學課程需要執行 VRAppLauncher 聊天機器人：

    npm run demo-vrapp
    
### <a name="open-the-demo"></a>開啟示範

在 Web UI 的模型清單中，按一下 [VRAppLauncher]。 

## <a name="entities"></a>實體

我們已經建立 4 個實體：

- AppName：例如 Skype
- PlacementLocation：例如牆
- UnknownAppName：系統無法辨識使用者所說的實體名稱時設定的程式設計實體，比方說尚未安裝。
- DisAmbigAppNames：由兩個或更多個已安裝的應用程式組成的陣列，其名稱符合使用者所說的名稱。 

![](../media/tutorial_vrapplauncher_entities.PNG)

### <a name="actions"></a>動作

我們已建立了一組動作，其中包含稱為 LaunchApp 的 API，其會開始呼叫函式以啟動應用程式。

![](../media/tutorial_vrapplauncher_actions.PNG)

### <a name="training-dialogs"></a>訓練對話
我們已經定義許多訓練對話。

![](../media/tutorial_vrapplauncher_dialogs.PNG)

讓我們試試以教學工作階段為例。

1. 按一下 [Train Dialogs]\(訓練對話\)，然後按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「hi」\(嗨\)。
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [which app do you want to start?]\(您想要啟動哪些應用程式？\)
4. 輸入「outlook」。
    - LUIS 會將它辨識為實體。
5. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [where do you want it placed?]\(您想要將它放在哪裡？\)
4. 輸入「on the wall」(牆上)。
    - LUIS 會將它辨識為 PlacementLocation。
2. 輸入 [Score Actions]\(評分動作\)。
6. 選取「LaunchApp」
7. 系統：「starting outlook on the wall」\(在牆上啟動 outlook\)。
    - 這會觸發 API 呼叫。 此呼叫的程式碼位於 C:\<\installedpath>\src\demos\demoVRAppLauncher.ts。 不過，它並非真的在此示範內包含啟動 Outlook 的程式碼。
    - 這會清除 AppName 和 PlacementLocation 實體。 會傳回上列的字串做為回應。
4. 按一下 [Done Teaching]\(完成教學\)。

![](../media/tutorial_vrapplauncher_callbackcode.PNG)

讓我們開始處理未知實體和實體不明確的另一個訓練階段。

1. 按一下 [New Train Dialog]\(新增訓練對話\)。
1. 輸入「start OneNote」(啟動 OneNote)。 
    - 模型會將 OneNote 辨識為應用程式名稱。 程式碼中定義的 `EntityDetectionCallback` 函式會藉由將使用者輸入為應用程式名稱的名稱與程式碼中定義的應用程式清單進行比對來解析該名稱。 然後，這會傳回所有相符的應用程式所組成的集合。 
    - 如果相符項目的清單為零，表示未安裝應用程式。 這會將其放置於 unknownAppName。
    - 如果找到一個以上的應用程式，就會將它們複製到 `DisambigAppNames` 並清除 AppName 實體。
2. 按一下 [Score Actions]\(評分動作\)。
3. 按一下以選取 [Sorry, I don't know the app $UknownAppName.]\(很抱歉，我不知道應用程式 $UknownAppName。\)
4. 輸入 'start Amazon' (啟動 Amazon)。 我們將嘗試另一個路徑。
5. 按一下 [Score Actions]\(評分動作\)。
    - Amazon Video 和 Amazon Music 現在會在 `DisambigAppNames` 記憶體中，並已清除 OneNote。
3. 按一下以選取 [There are few apps that sound like that...]\(有一些應用程式似乎...\)
    - 分數並不高，因為我們到目前為止只定義了一些定型對話。 定義更多定型對話，讓模型更具決定性。
2. 輸入 [Score Actions]\(評分動作\)。
4. 按一下 [Done Teaching]\(完成教學\)。

現在您已了解如何進行實體解析。 此示範也說明了 API 回呼，並示範如何使用範本來收集資訊、檢查是否存在且是否模稜兩可，並據此採取適當動作。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [部署對話學習模組聊天機器人](../deploy-to-bf.md) (英文)
