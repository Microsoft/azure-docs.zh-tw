---
title: API 參考 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 了解 Content Moderator 的內容審核和審查 API。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: reference
ms.date: 06/25/2017
ms.author: sajagtap
ms.openlocfilehash: f065310e3afeaf95af602e513421da6770c9583f
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222399"
---
# <a name="content-moderator-api-reference"></a>Content Moderator API 參考

您可透過下列方式開始使用 Azure Content Moderator API：(另請參閱[管理認證](review-tool-user-guide/credentials.md))。

- 在 Azure 入口網站中，[訂閱 Content Moderator API](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator)。
- 註冊 [Content Moderator 審查工具](http://contentmoderator.cognitive.microsoft.com/)。 請參閱[快速入門](quick-start.md)。

## <a name="moderation-apis"></a>仲裁 API

您可以使用下列 Content Moderator API 來設定審核後的工作流程。

| 說明 | 參考 |
| -------------------- |-------------|
| **影像審核 API**<br /><br />掃描影像，並使用標記、信賴分數及其他擷取的資訊來偵測潛在成人和不雅內容。 <br /><br />使用此資訊，在審核後的工作流程中發佈、拒絕或審查內容。 <br /><br />| [影像審核 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c "影像審核 API 參考")   |
| **文字審核 API**<br /><br />掃描文字內容。 傳回粗話字詞和個人識別資訊 (PII)。 <br /><br />使用此資訊，在審核後的工作流程中發佈、拒絕或審查內容。<br /><br /> | [文字審核 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f "文字審核 API 參考")   |
| **影片審核 API**<br /><br />掃描影片並偵測潛在成人和不雅內容。 <br /><br />使用此資訊，在審核後的工作流程中發佈、拒絕或審查內容。<br /><br /> | [影片審核 API 概觀](video-moderation-api.md "影片審核 API 概觀")   |
| **清單管理 API**<br /><br />建立和管理影像和文字的自訂排除或包含清單。 若已啟用，[影像 - 比對] 和 [文字 - 畫面] 作業會根據您的自訂清單，執行所提交內容的模糊比對。 <br /><br />為了提高效率，您可以略過電腦學習式審核步驟。<br /><br /> | [清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f675 "清單管理 API 參考")   |

## <a name="review-api"></a>審查 API

審查 API 具有下列元件：

| 說明 | 參考 |
| -------------------- |-------------|
| **作業**<br /><br /> 針對影像和文字內容起始掃描並審查的審核工作流程。 審核作業會使用影像審核 API 和文字審核 API 來掃描您的內容。 審核作業會使用已定義和預設的工作流程來產生審查。 <br /><br />在人力審核者已審查自動指派的標記和預測資料並提交內容審核決策之後，審查 API 會將所有資訊提交至您的 API 端點。<br /><br /> | [作業參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5 "作業參考")   |
| **審查**<br /><br />使用審查工具，直接為人力審核者建立影像或文字審查。<br /><br /> 在人力審核者已審查自動指派的標記和預測資料並提交內容審核決策之後，審查 API 會將所有資訊提交至您的 API 端點。<br /><br /> | [審查參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4 "審查參考")   |
| **工作流程**<br /><br />建立、更新並取得您小組建立的自訂工作流程詳細資料。 您可以使用審查工具來定義工作流程。 <br /> <br />工作流程通常會使用 Content Moderator，但是也可以使用某些可作為審查工具中連接器的其他 API。<br /><br /> | [工作流程參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59 "工作流程參考")   |


