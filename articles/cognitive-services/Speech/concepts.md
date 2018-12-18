---
title: Bing 語音概念 | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 用於 Microsoft 語音服務的基本概念。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: 6089f053472faaa7fa8c957904f73c7061cb2d3f
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344920"
---
# <a name="basic-concepts"></a>基本概念

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

此頁面將說明 Microsoft 語音辨識服務中的一些基本概念。 在您的應用程式中使用 Microsoft 語音辨識 API 之前，建議您先閱讀此頁面。

## <a name="understanding-speech-recognition"></a>了解語音辨識

如果這是您第一次建立啟用語音的應用程式，或是您第一次將語音功能新增到現有應用程式中，本節可協助您開始執行這些作業。 如果您在啟用語音的應用程式上已有一些相關經驗，可以選擇快速瀏覽本節，或者，如果您在語音方面已有相當豐富的經驗，並且想直接了解通訊協定的詳細資料，您可以完全掠過本節。

### <a name="audio-streams"></a>音訊串流

語音基本概念中最首要的莫過於「音訊串流」。 不同於發生在單一時間點且包含單一資訊片段的按鍵輸入，口語要求會長達數百毫秒，並且包含數 KB 的資訊。 對於開發人員來說，口說語句的持續時間讓他們難以在其應用程式上提供精簡且優雅的語音體驗。 現今電腦和演算法執行語音轉譯的速度大約是語句持續時間的一半，也就是 2 秒的語句大約可在 1 秒左右轉譯完成，但在處理使用者作業時會遇到 1 秒延遲的任何應用程式既不精簡也不優雅。

幸運的是，有些方式可「隱藏」轉譯時間，也就是在使用者說出語句的另一部分時，對一部分的語句執行轉譯。 例如，藉由將 1 秒的語句分割成 10 個 100 毫秒的區塊，並依序對每個區塊執行轉譯，若總計有 500 毫秒的語句需要轉譯，則其中有超過 450 毫秒可「隱藏」，如此一來，使用者在說話時就不會察覺到轉譯正在執行。 思考此範例時，請記住服務是在使用者說出下一個 100 毫秒的音訊時，對前 100 毫秒的音訊執行轉譯，因此當使用者停止說話時，服務將只需要轉譯大約 100毫秒的音訊來產生結果。

為達到此使用者體驗，口語音訊的資訊會在使用者說話時收集到區塊中並進行轉譯。 這些音訊區塊是從「音訊串流」收集來的，而將這些音訊區塊傳送給服務的程序稱為「音訊串流處理」。 在任何啟用語音的應用程式中，音訊串流處理都是很重要的一部分；若要改善您應用程式的使用者體驗，調整區塊大小和最佳化串流實作是最具影響力的方式。

### <a name="microphones"></a>麥克風

人們會使用耳朵來處理口語音訊，但無生命的硬體則會使用麥克風。 若要在任何應用程式中啟用語音，您需要整合麥克風來為應用程式提供音訊串流。

麥克風的 API 必須可讓您開始和停止從麥克風接收音訊位元組。 您必須決定哪些使用者動作會觸發麥克風開始接聽語音。 您可以選擇按下按鈕來觸發開始接聽，或是選擇讓「關鍵字」或「喚醒字」偵查器持續接聽麥克風，並使用該模組的輸出，來觸發將音訊傳送到 Microsoft 語音服務的動作。

### <a name="end-of-speech"></a>語音結束

對人類來說，偵測說話者「停止」說話的「時機」似乎很簡單，但如果沒有實驗室的條件，這可能會變得困難。 要找到語句說完後完全安靜的時間點並不容易，因為通常會有環境中的雜音，這讓事情變得複雜許多。 Microsoft 語音服務在這方面做得很好，它可快速偵測使用者停止說話的時機，而且服務可將這項資訊告知您的應用程式，但這種安排表示應用程式會最後知道使用者何時停止說話。 這完全不像其他形式的輸入，應用程式通常都會「第一個」知道使用者的輸入何時開始和結束。

### <a name="asynchronous-service-responses"></a>非同步服務回應

事實上，應用程式需接收使用者何時完成輸入的通知，並不會造成效能低落或程式設計上的困難，但當您思考語音要求時，必須了解這和您熟悉的輸入要求/回應模式不同。 由於應用程式不會知道使用者何時停止說話，應用程式必須在同時但不同步處理來自服務的回應時，持續將音訊串流傳送給服務。 此模式不同於其他要求/回應的 Web 通訊協定，例如 HTTP。 在這些通訊協定中，您必須先完成要求，才能接收任何回應；在 Microsoft 語音服務通訊協定中，「您仍在串流處理要求的音訊時」，就會收到回應。

> [!NOTE]
> 使用語音 HTTP REST API 時，此功能不受支援。

### <a name="turns"></a>回合

語音負責載運資訊。 當您說話時，就是試著將您擁有的資訊傳遞給接聽該資訊的人。 傳遞資訊時，您通常會說話也會聆聽。 同樣地，啟用語音的應用程式會透過交替進行接聽和回應來與使用者互動，雖然通常您的應用程式大部分都在接聽。 使用者的口語輸入及服務對此輸入的回應稱為「回合」。 「回合」會在使用者說話時開始，並在應用程式完成處理語音服務回應時結束。

### <a name="telemetry"></a>遙測

建立啟用語音的裝置或應用程式可能是項挑戰，即使對資深的開發人員來說也是如此。 以串流為基礎的通訊協定通常會讓人望而生畏，而且像是靜音偵測這類重要細節對您來說可能是完全陌生的。 由於要成功傳送和接收非常多訊息才能完成一組要求/回應，因此完整且正確地收集這些訊息的資料是「非常」重要的。 Microsoft 語音服務通訊協定可用於此資料收集。 您應盡可能精確地提供所需資料；提供完整且正確的資料對您將有所幫助 -- 當您需要 Microsoft 語音服務團隊協助您針對用戶端實作進行疑難排解時，您收集的遙測資料品質會大大影響問題分析。

> [!NOTE]
> 使用語音辨識 REST API 時，此功能不受支援。

### <a name="speech-application-states"></a>語音應用程式狀態

在應用程式中啟用語音輸入的步驟，與其他輸入形式 (例如滑鼠點按或手指點選) 的步驟稍微不同。 您必須追蹤應用程式接聽麥克風並將資料傳送到語音服務的時間、應用程式等候服務回應的時間，以及處於閒置狀態的時間。 下圖顯示這些狀態之間的關聯性。

![語音應用程式狀態圖表](Images/speech-application-state-diagram.png)

由於 Microsoft 語音服務會參與部分狀態，因此服務通訊協定會定義訊息來協助應用程式轉換狀態。 您的應用程式需要解譯及處理這些通訊協定訊息，以追蹤和管理語音應用程式狀態。

## <a name="using-the-speech-recognition-service-from-your-apps"></a>從您的應用程式中使用語音辨識服務

Microsoft 語音辨識服務為開發人員提供兩種可將語音新增到應用程式的方式。

- [REST API](GetStarted/GetStartedREST.md)：開發人員可以從自己的應用程式中對服務使用 HTTP 呼叫，來取得語音辨識服務。
- [用戶端程式庫](GetStarted/GetStartedClientLibraries.md)：如需進階功能，開發人員可以下載 Microsoft Speech 用戶端程式庫，並連結至他們的應用程式。  用戶端程式庫可用於使用不同語言 (C#、Java、JavaScript、ObjectiveC) 的各種平台 (Windows、Android、iOS)。

| 使用案例 | [REST APIs](GetStarted/GetStartedREST.md) | [用戶端程式庫](GetStarted/GetStartedClientLibraries.md) |
|-----|-----|-----|
| 轉換簡短的語音，例如命令 (音訊長度 < 15 秒)，但不提供中期結果 | 是 | 是 |
| 轉換長音訊 (> 15 秒) | 否 | 是 |
| 串流音訊並提供所需的中期結果 | 否 | 是 |
| 使用 LUIS 來理解從音訊轉換的文字 | 否 | 是 |

 如果您的語言或平台還沒有 SDK，您可以根據[通訊協定文件](API-Reference-REST/websocketprotocol.md)建立您自己的實作。

## <a name="recognition-modes"></a>辨識模式

辨識模式有三種：`interactive`、`conversation` 和 `dictation`。 辨識模式會根據使用者可能的說話方式調整語音辨識。 為您的應用程式選擇適當的辨識模式。

> [!NOTE]
> 辨識模式在 [REST 通訊協定](#rest-speech-recognition-api)中的行為可能與在 [WebSocket 通訊協定](#webSocket-speech-recognition-api) 中的行為不同。 例如，REST API 不支援連續辨識，即使在交談或聽寫模式中也是如此。
> [!NOTE]
> 當您直接使用 REST 或 WebSocket 通訊協定時，這些模式皆適用。 [用戶端程式庫](GetStarted/GetStartedClientLibraries.md)會使用不同的參數來指定辨識模式。 如需詳細資訊，請參閱您所選擇的用戶端程式庫。

針對所有辨識模式，Microsoft 語音服務只會傳回一個辨識片語結果。 每個單一語句的限制為 15 秒。

### <a name="interactive-mode"></a>互動模式

在 `interactive` 模式中，使用者會提出簡短要求，並預期應用程式在回應中執行動作。

下列是互動模式應用程式的典型特性：

- 使用者知道他們的說話對象是機器而非另一個人。
- 應用程式使用者會根據他們要應用程式完成的事，事先知道他們要說的內容。
- 語句通常持續 2-3 秒。

### <a name="conversation-mode"></a>交談模式

在 `conversation` 模式中，使用者會參與人與人的對話。

下列是交談模式應用程式的典型特性：

- 使用者知道他們在對某人說話。
- 語音辨識會藉由允許一個或兩個參與者看到口說文字，來加強人員交談。
- 使用者不一定會構想他們想說的話。
- 使用者經常會使用俚語及其他非正式的話語。

### <a name="dictation-mode"></a>聽寫模式

在 `dictation` 模式中，使用者會為應用程式朗誦較長的語句，以進行進一步的處理。

下列是聽寫模式應用程式的典型特性：

- 使用者知道他們在對機器說話。
- 使用者會看到語音辨識文字結果。
- 使用者通常會構想他們要說的話，並使用較正式的語言。
- 使用者採用完整句子，並持續 5-8 秒。

> [!NOTE]
> 在聽寫和交談模式中，Microsoft 語音服務不會傳回部分結果。 相反地，服務會傳回音訊串流中靜音界限後的穩固片語結果。 Microsoft 可能會增強語音通訊協定，以改善這些連續辨識模式中的使用者體驗。

## <a name="recognition-languages"></a>辨識語言

「辨識語言」會指定應用程式使用者所說的語言。 使用 language URL 查詢參數來指定連線上的「辨識語言」。 language 查詢參數的值會使用 IETF 語言標記 [BCP 47](https://en.wikipedia.org/wiki/IETF_language_tag)，並且**必須**是語音辨識 API 支援的其中一種語言。 語音服務所支援的完整語言清單位於＜[支援的語言](API-Reference-REST/supportedlanguages.md)＞頁面。

Microsoft 語音服務會藉由顯示 `HTTP 400 Bad Request` 回應來拒絕無效的連線要求。 無效的要求就是：

- 未包含 language 查詢參數值。
- 包含格式不正確的 language 查詢參數。
- 包含的 language 查詢參數不是支援的語言。

您可以選擇建置應用程式來支援服務所支援的一個或所有語言。

### <a name="example"></a>範例

在下列範例中，應用程式針對說英文 (美國) 的人使用「交談」語音辨識模式。

```HTTP
https://speech.platform.bing.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

## <a name="transcription-responses"></a>轉譯回應

轉譯回應會將從音訊轉換的文字傳回給用戶端。 此轉譯回應包含下列欄位：

- `RecognitionStatus` 會指定辨識的狀態。 下表提供可能的值。

| 狀態 | 說明 |
| ------------- | ---------------- |
| 成功 | 辨識成功則會出現 DisplayText 欄位 |
| NoMatch | 音訊串流中偵測到語音，但目標語言中沒有符合的字組。 請參閱 [NoMatch 辨識狀態(#nomatch-recognition-status) 以取得詳細資訊  |
| InitialSilenceTimeout | 音訊串流的開頭沒有任何聲音，而且等候語音的服務已逾時 |
| BabbleTimeout | 音訊串流的開頭只有雜音，而且等候語音的服務已逾時 |
| Error | 辨識服務發生內部錯誤，無法繼續 |

- `DisplayText` 表示辨識後的片語，其中已套用大小寫、標點符號及反向文字正規化 (inverse-text-normalization)，並且已使用星號遮住不雅內容。 DisplayText 欄位「只」會在 `RecognitionStatus` 欄位有 `Success` 值時才會出現。

- `Offset` 會指定辨識片語後，相對於音訊串流起點的位移 (以 100 奈秒為單位)。

- `Duration` 會指定此語音片語的持續時間 (以 100 奈秒為單位)。

如有需要，轉譯回應會傳回詳細資訊。 請參閱[輸出格式](#output-format)，以了解如何傳回更詳細的輸出。

Microsoft 語音服務支援其他轉譯程序，包括新增大小寫和標點符號、遮罩不雅內容，以及將文字標準化為通用形式。 例如，如果使用者說出的片語以文字表示是 "remind me to buy six iPhones" (提醒我買六支 iPhone)，Microsoft 語音服務傳回的轉譯文字會是 "Remind me to buy 6 iPhones" (提醒我買 6 支 iPhone)。 將文字 "six" (六) 轉換為數字 "6" 的程序稱為「反向文字正規化」 (縮寫為 *ITN*)。

### <a name="nomatch-recognition-status"></a>NoMatch 辨識狀態

當 Microsoft 語音服務在音訊串流中偵測到語音，但無法將此語音與要求中正在使用的語言文法相對應時，轉譯回應會在 `RecognitionStatus` 中傳回 `NoMatch`。 例如，當辨識工具預期英文 (美國) 為說話語言，但使用者卻以德文說出一些句子時，就可能發生「NoMatch」的情況。 語句的波形模式會指出有人在說話，但說出的字組不符合辨識工具正在使用的英文 (美國) 語彙。

另一個會發生「NoMatch」的情況是，辨識演算法無法為含在音訊串流中的聲音找到正確相符項目。 當這種狀況發生時，Microsoft 語音服務可以產生包含「假設文字」的 speech.hypothesis 訊息，但會產生 speech.phrase 訊息，其中 *RecognitionStatus* 是 *NoMatch*。 這種情況是正常現象；切勿對 speech.hypothesis 訊息中的文字精確度進行任何假設。 此外，切勿認為因為 Microsoft 語音服務會產生 speech.hypothesis 訊息，所以服務就能夠產生 RecognitionStatus 為 Success 的 speech.phrase 訊息。

## <a name="output-format"></a>輸出格式

Microsoft 語音服務可以在轉譯回應中傳回各種酬載格式。 所有承載都為 JSON 結構。

您可以藉由指定 `format` URL 查詢參數來控制片語結果的格式。 根據預設，服務會傳回 `simple` 結果。

| 格式 | 說明 |
|-----|-----|
| `simple` | 簡化的片語結果，其中包含辨識狀態和以顯示形式表示的已辨識文字。 |
| `detailed` | 片語結果的辨識狀態和 N 最佳 (N-best) 清單，其中每個片語結果都包含全部四種辨識形式和信賴分數。 |

在回應中，`detailed` 格式除了包含 `RecognitionStatus`、`Offset` 和 `duration` 外，也會包含 [N 最佳值](#n-best-values)。

### <a name="n-best-values"></a>N 最佳值

無論是人類或機器的接聽者，都無法確定能「完全無誤」地聽到說話內容。 接聽者只可以對特定的語句解譯提供「可能性」。 

在一般情況下，如果對經常互動的對象說話，會有較高機率可辨識出說出的文字。 機器類型的語音接聽工具會盡量達到類似的精確度，而且，如果在適當的條件下，[這些接聽工具可達到人類的水準](https://blogs.microsoft.com/next/2016/10/18/historic-achievement-microsoft-researchers-reach-human-parity-conversational-speech-recognition/#sm.001ykosqs14zte8qyxj2k9o28oz5v)。

語音辨識中所使用的演算法會在一般處理程序中探索語句的替代解譯。 通常，替代方案會遭到捨棄就表示單一解譯方式變得很強大。 不過，在沒有最佳方式的情況下，語音辨識工具會使用可能的替代解譯清單來完成作業。 此份清單中的前 N 個替代方式稱為「N 最佳清單」。 每個替代方法皆會獲派[信賴分數](#confidence)。 信賴分數的範圍是從 0 到 1。 分數 1 代表信賴層級最高。 分數 0 代表信賴層級最低。

> [!NOTE]
> 在多個語句間，N 最佳清單中的項目數會有所不同。 「相同」語句的多個辨識方式間，項目數目也可能有所不同。 基於語音辨識演算法的概率本質，此差異是自然和可預期的結果。

在 N 最佳清單中傳回的每個項目會包含

- `Confidence`，表示此項目的[信賴分數](#confidence)。
- `Lexical`，這是已辨識文字的[語彙形式](#lexical-form)。
- `ITN`，這是已辨識文字的 [ITN 形式](#itn-form)。
- `MaskedITN`，這是已辨識文字的[遮罩 ITN 形式](#masked-itn-form)。
- `Display`，這是已辨識文字的[顯示形式](#display-form)。

### 信賴分數 <a id="confidence"></a>

信賴分數是語音辨識系統中不可或缺的。 Microsoft 語音服務會從「信賴分類器」取得信賴分數。 Microsoft 會透過一組功能來訓練信賴分類器，這組功能主要用來最大化分辨正確和不正確辨識的能力。 信賴分數是針對個別字組和整個語句進行評估的結果。

如果您選擇使用服務所傳回的信賴分數，請注意下列行為：

- 只可在相同的辨識模式和語言內比較信賴分數。 請勿在不同語言或不同辨識模式之間比較分數。 例如，互動辨識模式中的信賴分數與聽寫模式中的信賴分數並「沒有」相互關聯。
- 信賴分數適用於一組有限的語句。 針對大量語句，自然會有高度的分數變異性。

如果您選擇使用信賴分數值作為應用程式運作的「閾值」，請使用語音辨識來建立此閾值。

- 在您的應用程式上，對具代表性的語句範例執行語音辨識。
- 收集範例集合中每個辨識的信賴分數。
- 以該範例的部分信賴百分位數為基礎來建立閾值。

沒有適用於所有應用程式的單一閾值。 一個應用程式可接受的信賴分數，可能無法讓另一個應用程式接受。

### <a name="lexical-form"></a>語彙形式

語彙形式是已辨識的文字，完全以語句的原樣產生，沒有標點符號和大小寫。 例如，地址 "1020 Enterprise Way" 的語彙形式會是 "ten twenty enterprise way" (假設說話者就是這麼說的)。 句子 "Remind me to buy 5 pencils" (提醒我買 5 支鉛筆) 的 語彙形式是 "remind me to buy five pencils" (提醒我買五支鉛筆)。

需要執行非標準文字正規化的應用程式最適合使用語彙形式。 如果應用程式需要未處理的辨識字組，也適用語彙形式。

語彙形式中不會遮罩不雅內容。

### <a name="itn-form"></a>ITN 形式

文字正規化是將文字從一個形式轉換成另一個「標準」形式的程序。 例如，電話號碼 "555-1212" 可能會轉換成「五五五一二一二」的標準形式。 「反向」文字正規化 (ITN) 會反轉此程序，將「五五五一二一二」字組轉換成反向的標準形式：555-1212。 辨識結果的 ITN 形式不包含大小寫或標點符號。

要處理已辨識文字的應用程式最適合使用 ITN 形式。 例如，可讓使用者說出搜尋詞彙，然後在 Web 查詢中使用這些詞彙的應用程式可能會使用 ITN 形式。 ITN 形式中不會遮罩不雅內容。 若要遮罩不雅內容，請使用「遮罩的 ITN 形式」。

### <a name="masked-itn-form"></a>遮罩的 ITN 形式

由於不雅內容本來就是說話的一部份，因此 Microsoft 語音服務仍會在這類字組與片語說出時加以辨識。 不過，不雅內容可能不適用於所有應用程式，特別是針對有限制且使用對象不是成人的應用程式。

遮罩的 ITN 形式會對反向文字正規化形式套用不雅內容遮罩。 若要遮罩不雅內容，請將不雅內容參數值的值設定為 `masked`。 遮罩不雅內容後，語言中辨識為不雅內容語彙的字組將會以星號取代。 例如：提醒我購買 5 支****鉛筆。 辨識結果的遮罩 ITN 形式不包含大小寫或標點符號。

> [!NOTE]
> 如果 profanity 查詢參數值設定為 `raw`，則遮罩的 ITN 形式等同於 ITN 形式。 不雅內容「不會」加上遮罩。

### <a name="display-form"></a>顯示形式

標點符號和大小寫可指示何處要強調、何處要暫停等等，讓文字更易於了解。 顯示形式會在辨識結果上加上標點符號和大小寫，使其成為應用程式顯示口說文字的最恰當形式。

由於顯示形式會延伸到遮罩的 ITN 形式，因此您可以將不雅內容參數值設定為 `masked` 或 `raw`。 如果值設定為 `raw`，則辨識結果會包含任何使用者說出的不雅內容。 如果值設定為 `masked`，語言中辨識為不雅內容語彙的字組將會以星號取代。

### <a name="sample-responses"></a>回應範例

所有承載都為 JSON 結構。

`simple` 片語結果的承載格式：

```json
{
  "RecognitionStatus": "Success",
  "DisplayText": "Remind me to buy 5 pencils.",
  "Offset": "1236645672289",
  "Duration": "1236645672289"
}
```

`detailed` 片語結果的承載格式：

```json
{
  "RecognitionStatus": "Success",
  "Offset": "1236645672289",
  "Duration": "1236645672289",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "remind me to buy five pencils",
        "ITN" : "remind me to buy 5 pencils",
        "MaskedITN" : "remind me to buy 5 pencils",
        "Display" : "Remind me to buy 5 pencils.",
      },
      {
        "Confidence" : "0.54",
        "Lexical" : "rewind me to buy five pencils",
        "ITN" : "rewind me to buy 5 pencils",
        "MaskedITN" : "rewind me to buy 5 pencils",
        "Display" : "Rewind me to buy 5 pencils.",
      }
  ]
}
```

## <a name="profanity-handling-in-speech-recognition"></a>語音辨識中的不雅內容處理

Microsoft 語音服務能辨識所有形式的人類話語，包括人們分類為「不雅內容」的字組和片語。 您可以使用 profanity 查詢參數來控制服務處理不雅內容的方式。 根據預設，服務會遮罩 speech.phrase 結果中的不雅內容，而且不會傳回包含不雅內容的 speech.hypothesis 訊息。

| Profanity 值 | 說明 |
| - | - |
| `masked` | 以星號遮罩不雅內容。 這是預設行為。 | 
| `removed` | 從所有結果中移除不雅內容。 |
| `raw` | 辨識不雅內容並在所有結果中傳回。 |

### <a name="profanity-value-masked"></a>Profanity 的 `Masked` 值

若要遮罩不雅內容，請將 profanity 查詢參數的值設定為 masked。 當 profanity 查詢參數使用這個值，或未針對要求指定參數時，服務會「遮罩」不雅內容。 服務執行遮罩的方法是以星號取代辨識結果中的不雅內容。 當您指定使用不雅內容遮罩處理時，服務就不會傳回包含不雅內容的 speech.hypothesis 訊息。

### <a name="profanity-value-removed"></a>Profanity 的 `Removed` 值

當 profanity 查詢參數使用 removed 值時，服務會從 speech.phrase 和 speech.hypothesis 訊息中移除不雅內容。 結果就如同「未說過不雅內容」。

#### <a name="profanity-only-utterances"></a>只有不雅內容的語句

在應用程式已設定服務來移除不雅內容時，使用者可能「只」說了不雅內容。 在這種情況下，如果辨識模式為「聽寫」或「交談」，服務就不會傳回 speech.result。 如果辨識模式為「互動」，則服務會傳回狀態碼為 NoMatch 的 speech.result。 

### <a name="profanity-value-raw"></a>Profanity 的 `Raw` 值

當 profanity 查詢參數使用 raw 值時，服務不會從 speech.phrase 或 speech.hypothesis 訊息中移除或遮罩不雅內容。
