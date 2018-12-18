---
title: 設定 Content Moderator 的審查工具設定 | Microsoft Docs
description: 設定或取得小組、標記、連接器、工作流程及認證。
services: cognitive-services
author: sanjeev3
manager: mikemcca
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: article
ms.date: 06/25/2017
ms.author: sajagtap
ms.openlocfilehash: a3432a1d8f424fbe78570f47b774c6e7942e16b1
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367926"
---
# <a name="about-review-tool-settings"></a>關於審查工具設定 #

使用 [審查工具儀表板] 上的 [設定] 索引標籤，可以輕鬆地定義和變更許多元件。

![Content Moderator 審核設定](images/settings-1.png)

## <a name="team-and-subteams"></a>小組與子小組 ## 

從這個索引標籤管理您的小組和子小組。您只能有一個小組，但是您可以[建立多個子小組](subteams.md)並將邀請傳送給未來成員。 在您送出邀請之後，您可加以監視、變更小組成員的權限，以及邀請其他使用者。 在小組成員接受您的邀請之後，您可以將這些成員指派給不同的子小組。 您可以將小組成員的角色設定為系統管理員或審查者：系統管理員可以邀請其他使用者，而審查者無法這麼做。

![Content Moderator 小組設定](images/settings-2-team.png)

## <a name="tags"></a>標記 ##

您可以為標記輸入簡短代碼、名稱和描述，即可在其中[定義自訂標記](tags.md)。 建立標籤之後，即可在審核期間使用。 開啟和關閉可見性，即可針對不同的審核使用不同的標記。

![Content Moderator 標記設定](images/settings-3-tags.png)

## <a name="connectors"></a>連接器 ##

工作流程可使用連接器來與審查工具通訊，進而新增功能。 審查工具會使用預設工作流程呼叫 Content Moderator API，以便審查內容。 當您註冊審查工具時，它會為您自動佈建 Moderator API 認證。 它也支援整合其他連接器 API (只要連接器可用)。 我們提供了一些立即可用的連接器。

您可以在 [[連接器]](connectors.md) 索引標籤中管理連接器。 您可以新增或刪除連接器，以及尋找特定連接器的訂用帳戶金鑰。 按一下 [連線] 將這些項目新增到您的自訂工作流程。 

![Content Moderator 連接器設定](images/settings-4-connectors.png)

## <a name="workflows"></a>工作流程 ##

從 [工作流程] 索引標籤管理工作流程。您可上傳範例檔案，以測試工作流程。 您也可以使用可用的 API 連接器 (在 [連接器] 索引標籤上找到)，針對影像和文字[定義自訂工作流程](workflows.md)。 

![Content Moderator 工作流程設定](images/settings-5-workflows.png)

## <a name="credentials"></a>認證 ##

此索引標籤可供快速存取您的 Content Moderator 訂用帳戶金鑰，您需有該金鑰才能使用 Content Moderator 隨附的 API (影像審核、文字審核、清單管理、工作流程和審核 API) 的內容。
 
![Content Moderator 認證](images/settings-6-credentials.png)
