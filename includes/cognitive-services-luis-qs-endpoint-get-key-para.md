---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: luis
ms.topic: include
ms.custom: include file
ms.date: 08/16/2018
ms.author: diberry
ms.openlocfilehash: fcf3bf29e2cdf89bdc93c7ebac313e5d9a9c18f0
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47043982"
---
預測端點的存取權會與端點金鑰一起提供。 基於本快速入門的目的，請使用與您的 LUIS 帳戶相關聯的免費入門金鑰。 
 
1. 使用您的 LUIS 帳戶登入。 

    [![Language Understanding (LUIS) 應用程式清單的螢幕擷取畫面](media/cognitive-services-luis/app-list.png "Language Understanding (LUIS) 應用程式清單的螢幕擷取畫面")](media/cognitive-services-luis/app-list.png)

2. 在右上功能表中選取您的名稱，然後選取 [設定]。

    ![LUIS 使用者設定功能表存取](media/cognitive-services-luis/get-user-settings-in-luis.png)

3. 複製 [撰寫金鑰] 的值。 您稍後會在快速入門中使用此值。 

    [![Language Understanding (LUIS) 使用者設定的螢幕擷取畫面](media/cognitive-services-luis/get-user-authoring-key.png "Language Understanding (LUIS) 使用者設定的螢幕擷取畫面")](media/cognitive-services-luis/get-user-authoring-key.png)

    撰寫金鑰允許對撰寫 API 提出無限制的免費要求，而且每個月可對所有 LUIS 應用程式的預測端點 API 進行多達 1000 次查詢。 <!--Once the prediction endpoint quota from the authoring key is used for the month, you need to create a **Language Understanding** key from the Azure portal. The key created in the portal is known as the endpoint key. The endpoint key is used _only_ for prediction endpoint queries.-->