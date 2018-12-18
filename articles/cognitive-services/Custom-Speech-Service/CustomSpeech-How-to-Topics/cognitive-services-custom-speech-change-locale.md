---
title: Azure 上自訂語音服務中支援的地區設定和語言 | Microsoft Docs
description: 認知服務中自訂語音服務支援的語言概觀。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 02/08/2017
ms.author: panosper
ms.openlocfilehash: 8493aeb3c1c2436900ae626ad0f34cbe8b060163
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342163"
---
# <a name="supported-locales-in-custom-speech-service"></a>自訂語音服務中支援的地區設定

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

自訂語音服務目前在下列地區設定中支援模型的自訂：

| 模型類型 | 語言支援 |
|----|-----|
| 原音模型 | 美式英文 (en-US) |
| 語言模型 | 美式英文 (en-US)、中文 (zh-CN) |

雖然僅支援美式英文的原音模型自訂，但是為了進行自訂中文語言模型的離線測試，而支援匯入中文原音資料。

採取任何動作之前，必須先選取適當的地區設定。 目前的地區設定會標示在所有資料、模型和部署頁面上的資料表標題中。 若要變更地區設定，請按一下資料表標題下的 [變更地區設定] 按鈕。 這會帶您前往地區設定確認頁面。 按一下 [確定] 即可返回資料表。

您應該遵循後續步驟
* 了解[如何建立自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)來提升辨識準確度
* 了解[如何建立自訂語言模型](cognitive-services-custom-speech-create-language-model.md)來提升辨識率
* 請遵循[轉譯指導方針](cognitive-services-custom-speech-transcription-guidelines.md)來準備您的資料
