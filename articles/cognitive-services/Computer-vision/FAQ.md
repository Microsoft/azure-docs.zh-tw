---
title: 電腦視覺 API 常見問題集 | Microsoft Docs
description: 取得有關 Microsoft 認知服務中電腦視覺 API 常見問題集的解答。
services: cognitive-services
author: KellyDF
manager: corncar
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: article
ms.date: 01/26/2017
ms.author: kefre
ms.openlocfilehash: 5c862dd2fb26a005f4e785673a4e9358ecf9286f
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367950"
---
# <a name="computer-vision-api-frequently-asked-questions"></a>電腦視覺 API 常見問題集
### <a name="if-you-cant-find-answers-to-your-questions-in-this-faq-try-asking-the-computer-vision-api-community-on-stackoverflowhttpsstackoverflowcomquestionstaggedproject-oxfordormicrosoft-cognitive-or-contact-help-and-support-on-uservoicehttpscognitiveuservoicecom"></a>如果您在此常見問題集中找不到問題的解答，請嘗試在 [StackOverflow](https://stackoverflow.com/questions/tagged/project-oxford+or+microsoft-cognitive) 上詢問電腦視覺 API 社群，或連絡 [UserVoice 協助與支援](https://cognitive.uservoice.com/)。

-----

**問題**：*我可以訓練電腦視覺 API 使用自訂標記嗎？例如，我想要饋入貓咪品種的圖片來「訓練」AI，則會收到 AI 要求的品種值。*

**解答**：這項功能目前無法使用。 不過，我們的工程師正努力將這項功能放入電腦視覺中。

-----

**問題**：*在沒有網際網路連線的情況下，可以在本機使用電腦視覺嗎？*

**解答**：我們目前不提供內部部署或本機解決方案。

-----

**問題**：*電腦視覺支援哪些語言？*

**解答**：支援的語言包括：

| | | 支援的語言 | | |
|---------------- |------------------ |------------------ |--------------------------- |--------------------
| 丹麥文 (da-DK)  | 荷蘭文 (nl-NL)     | English           | 芬蘭文 (fi-FI)            |法文 (fr-FR)
| 德文 (de-DE)  | 希臘文 (el-GR)     | 匈牙利文 (hu-HU) | 義大利文 (it-IT)            | 日文 (ja-JP)
| 韓文 (ko-KR)  | 挪威文 (nb-NO) | 波蘭文 (pl-PL)    | 葡萄牙文 (pt-BR) (pt-PT) | 俄文 (ru-RU)
| 西班牙文 (es-ES)   | 瑞典文 (sv-SV)     | 土耳其文 (tr-TU)   |                            |

-----

**問題**：*電腦視覺可用來讀取車牌嗎？*

**解答**：視覺 API 提供採用 OCR 的良好文字偵測功能，但目前並未針對車牌進行最佳化。 我們持續不斷地嘗試改善我們的服務，並已將適用於自動車牌辨識的 OCR 新增至我們的功能要求清單。

-----

**問題**：*手寫辨識支援哪些語言？*

**解答**：目前僅支援英文。

-----

**問題**：*手寫辨識支援哪些類型的撰寫表面？*

**解答**：此技術可搭配不同種類的介面運作，包括白板、白紙和黃色自黏便箋。

-----

**問題**：*手寫辨識作業需花費多少時間？*

**解答**：所花費的時間量取決於文字的長度。 若為較長的文字，可能需要數秒鐘的時間。 因此，在「辨識手寫文字」作業完成後，您可能需要先行等待，才能使用「取得手寫文字作業結果」作業擷取結果。

-----

**問題**：*手寫辨識技術如何處理一行中使用插入號插入的文字？*

**解答**：手寫辨識作業會以個別行方式傳回這類文字。

-----

**問題**：*手寫辨識技術如何處理已刪去的字或行？*

**解答**：如果以多條線刪去字組來呈現它們無法辨識，則手寫辨識作業不會挑選這些字組。 不過，如果使用單條線刪去字組，則會將該刪除線視為雜訊，而手寫辨識作業仍會挑選這些字組。

-----

**問題**：*手寫辨識技術支援哪些文字方向？*

**解答**：手寫辨識作業可能會挑選角度最多約 30 度到 40 度的文字。

-----
