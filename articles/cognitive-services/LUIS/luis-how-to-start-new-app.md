---
title: 使用 LUIS 建立新應用程式
description: 在 Language Understanding (LUIS) 網頁上建立及管理應用程式。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 401c4fb8942aee73c036ae2b248a030eaea4917a
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031249"
---
# <a name="create-an-app"></a>建立應用程式
有好幾種方法可建立 LUIS 應用程式。 您可以在 [LUIS](https://www.luis.ai) 入口網站中建立 LUIS 應用程式，或透過 LUIS 撰寫 [API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f)。

## <a name="using-the-luis-portal"></a>使用 LUIS 入口網站
您可在 LUIS 入口網站中以數種方式建立新的應用程式：

* 從空的應用程式[著手](#create-new-app)，並建立意圖、語句和實體。
* 從空的應用程式[著手](#create-new-app)，並新增[預建網域](luis-how-to-use-prebuilt-domains.md)。
* 從已經包含意圖、語句和實體的 JSON 檔案[匯入 LUIS 應用程式](#import-new-app)。

## <a name="using-the-authoring-apis"></a>使用撰寫 API
您可使用撰寫 API 以數種方式建立新的應用程式：

* 從空的應用程式[著手](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f)，並建立意圖、語句和實體。
* 從預建網域[開始](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/59104e515aca2f0b48c76be5)。  


<a name="export-app"></a>
<a name="import-new-app"></a>
<a name="delete-app"></a>
 

## <a name="create-new-app-in-luis"></a>在 LUIS 中建立新的應用程式

1. 在 [我的應用程式] 頁面上，選取 [建立新的應用程式]。

    ![LUIS 應用程式清單](./media/luis-create-new-app/apps-list.png)


2. 在對話方塊中，將您的應用程式命名為 "TravelAgent"。

    ![建立新的應用程式對話方塊](./media/luis-create-new-app/create-app.png)

3. 選擇您的應用程式文化特性 (對於 TravelAgent 應用程式，選擇 [英文])，然後選取 [完成]。 

    > [!NOTE]
    > 建立應用程式之後便無法變更文化特性 (Culture)。 

    

<!--

## Import new app
You can set the name (50 char max), version (10 char max), and description of an app in the JSON file. Examples of application JSON files are available at [LUIS-Samples](https://github.com/Microsoft/LUIS-Samples/tree/master/documentation-samples/Examples-BookFlight).

1. On **My Apps** page, select **Import new app**.
2. In the **Import new app** dialog, select the JSON file defining the LUIS app.

    ![Import a new app dialog](./media/luis-create-new-app/import-app.png)

## Export app
1. On **My Apps** page, select the ellipsis (***...***) at the end of the app row.

    [![](media/luis-create-new-app/apps-list.png "Screenshot of pop-up dialog of per-app actions")](media/luis-create-new-app/three-dots.png#lightbox)

2. Select **Export app** from the menu. 

## Rename app

1. On **My Apps** page, select the ellipsis (***...***) at the end of the app row. 
2. Select **Rename** from the menu.
3. Enter the new name of the app and select **Done**.

## Delete app

> [!CAUTION]
> You are deleting the app for all collaborators and the owner. [Export](#export-app) the app before deleting it. 

1. On **My Apps** page, select the ellipsis (***...***) at the end of the app row. 
2. Select **Delete** from the menu.
3. Select **Ok** in the confirmation window.

## Export endpoint logs
The logs contain the Query, UTC time, and LUIS JSON response.

1. On **My Apps** page, select the ellipsis (***...***) at the end of the app row. 
2. Select **Export endpoint logs** from the menu.

```
Query,UTC DateTime,Response
text i'm driving and will be 30 minutes late to the meeting,02/13/2018 15:18:43,"{""query"":""text I'm driving and will be 30 minutes late to the meeting"",""intents"":[{""intent"":""None"",""score"":0.111048922},{""intent"":""SendMessage"",""score"":0.987501}],""entities"":[{""entity"":""i ' m driving and will be 30 minutes late to the meeting"",""type"":""Message"",""startIndex"":5,""endIndex"":58,""score"":0.162995353}]}"
```
-->
## <a name="next-steps"></a>後續步驟

您在應用程式中的第一項工作是[新增對應方式](luis-how-to-add-intents.md)。