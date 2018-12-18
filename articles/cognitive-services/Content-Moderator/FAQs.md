---
title: 常見問題集 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 獲得 Content Moderator 常見問題的解答。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: faq
ms.date: 11/21/2016
ms.author: sajagtap
ms.openlocfilehash: ce61a89069caf90695a2cb6c54f61de8fa8cb112
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226927"
---
# <a name="frequently-asked-questions-faq"></a>常見問題集 (FAQ)

#### <a name="what-does-my-content-moderator-subscription-include"></a>Content Moderator 訂用帳戶包含哪些功能？
Content Moderator 訂用帳戶包含檢閱工具和 API 的存取權。 您可以根據商務需求來決定要使用其中某個或兩者都要使用。

#### <a name="what-are-the-limitsrestrictions-of-the-content-that-can-be-moderated-by-using-the-api"></a>可使用此 API 審核的內容有哪些限制？
使用此 API 時，影像大小最小必須為 128 像素，最大不得超過 4 MB。 文字長度最多可為 1024 個字元。 影片持續時間沒有限制。

#### <a name="what-happens-if-the-content-passed-to-the-text-api-or-the-image-api-exceeds-the-size-limits"></a>當傳遞給文字 API 或影像 API 的內容超過大小限制時會如何？
當文字長度超過允許的上限時，文字 API 會傳回錯誤碼通知您。 當影像大小不合要求時，影像 API 也會傳回錯誤碼通知您。

#### <a name="do-you-keep-the-images-text-or-videos-that-are-submitted-for-moderation"></a>提交審核的影像、文字或影片是否會保存起來？
內容為您自己所有，未經您的同意，Microsoft 不得保存起來。 Microsoft 會使用業界領先的安全性措施來協助保護您的內容。

#### <a name="can-i-use-content-moderator-to-screen-for-illegal-child-exploitation-images"></a>是否可以使用 Content Moderator 來篩選出不合法的孩童利用影像？
否。 不過，合格組織可以使用[PhotoDNA Cloud Service](https://www.microsoft.com/photodna "Microsoft PhotoDNA Cloud Service") 來篩選出這方面的內容。

#### <a name="up-to-how-many-review-teams-can-a-user-join-can-the-user-switch-between-teams"></a>使用者最多可以加入幾個檢閱小組？ 使用者是否可以轉換到不同小組？
使用者一次可以加入一個小組。

#### <a name="what-kind-of-team-member-roles-are-supported-by-the-review-tool-how-are-they-different"></a>檢閱工具支援何種小組成員角色？ 兩者有何不同？
Studio 目前允許指派系統管理員和檢閱者角色。 系統管理員可以邀請其他使用者，並可存取組態設定，檢閱者則只能檢閱審核結果和標記或將內容解除標記。

#### <a name="what-is-a-tag-does-the-review-tool-support-custom-tags"></a>何謂標記？ 檢閱工具是否支援自訂標記？
標記是一到兩個字母的短碼，用來表示審核旗標，例如 'a' 代表成人、'r' 代表猥褻等等。 系統管理員可以視需要為其業務定義新標記。

#### <a name="how-many-team-members-can-i-have-in-my-review-team"></a>檢閱小組可以有幾位成員？
連同小組中的系統管理員在內，您最多可有五位小組成員。
