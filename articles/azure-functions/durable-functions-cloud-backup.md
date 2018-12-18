---
title: Durable Functions 中的展開傳送/收合傳送情節 - Azure
description: 了解如何在 Azure Functions 的 Durable Functions 擴充中實作展開傳送/收合傳送情節。
services: functions
author: cgillum
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 03/19/2018
ms.author: azfuncdf
ms.openlocfilehash: 4e7b7b6af1f41eb0077d8a8605eb2a553c251f8e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33763843"
---
# <a name="fan-outfan-in-scenario-in-durable-functions---cloud-backup-example"></a>Durable Functions 中的展開傳送/收合傳送情節 - 雲端備份範例

「展開傳送/收合傳送」是指同時執行多個函式，然後對結果執行一些彙總的模式。 本文以一個範例說明如何使用 [Durable Functions](durable-functions-overview.md) 來實作展開傳送/收合傳送情節。 範例是一個永久性函式，可將應用程式的所有或部分網站內容備份至 Azure 儲存體。

## <a name="prerequisites"></a>先決條件

* [安裝 Durable Functions](durable-functions-install.md)。
* 完成 [Hello 序列](durable-functions-sequence.md)逐步解說。

## <a name="scenario-overview"></a>案例概觀

在此範例中，函式會將指定目錄中的所有檔案，以遞迴方式上傳至 blob 儲存體。 還會計算已上傳的位元組總數。

您可以只撰寫一個函式來處理這一切。 您會遇到的主要問題是**延展性**。 單一函式執行只能在單一虛擬機器上執行，所以輸送量受限於該單一虛擬機器的輸送量。 另一個問題是**可靠性**。 如果半途失敗，或整個程序耗時超過 5 分鐘，備份可能會失敗，而處於只有部分完成的狀態。 於是就必須重新啟動。

更強固的方法是撰寫兩個一般函式：其中一個會列舉檔案，並將檔案名稱加入佇列中，另一個會讀取佇列，並將檔案上傳至 blob 儲存體。 這樣會有較高的輸送量和可靠性，但需要您佈建及管理佇列。 更重要的是，如果您想要再多一些功能，例如報告已上傳的位元組總數，則在**狀態管理**和**協調**方面會變得相當複雜。

Durable Functions 方法提供上述所有優點，而且額外負荷極低。

## <a name="the-functions"></a>函式

本文說明範例應用程式中的函式如下：

* `E2_BackupSiteContent`
* `E2_GetFileList`
* `E2_CopyFileToBlob`

下列各節說明用於 C# 指令碼的設定和程式碼。 適用於 Visual Studio 開發的程式碼顯示在本文結尾。

## <a name="the-cloud-backup-orchestration-visual-studio-code-and-azure-portal-sample-code"></a>雲端備份協調流程 (Visual Studio Code 和 Azure 入口網站範例程式碼)

`E2_BackupSiteContent` 函式使用協調器函式的標準  *function.json*。

[!code-json[Main](~/samples-durable-functions/samples/csx/E2_BackupSiteContent/function.json)]

以下是實作協調器函式的程式碼：

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E2_BackupSiteContent/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_BackupSiteContent/index.js)]

此協調器函式基本上會執行下列動作：

1. 接受 `rootDirectory` 值作為輸入參數。
2. 呼叫函式以取得 `rootDirectory` 下的檔案遞迴清單。
3. 執行多次平行函式呼叫，將每個檔案上傳到 Azure Blob 儲存體。
4. 等候所有上傳完成。
5. 傳回已上傳到 Azure Blob 儲存體的位元組總數。

請注意 `await Task.WhenAll(tasks);` (C#) 和 `yield context.df.Task.all(tasks);` (JS) 行。 呼叫 `E2_CopyFileToBlob` 函式時全部都「不會」等候。 這是為了平行執行而刻意設計。 將這一批工作傳給 `Task.WhenAll` 時，將會傳回一個「直到所有複製作業都完成」才會完成的工作。 如果您熟悉 .NET 中的工作平行程式庫 (TPL)，則對此不會感到陌生。 差別在於，這些工作可以在多個虛擬機器上同時執行，而 Durable Functions 擴充可確保端對端執行在處理序回收的情況下迅速恢復。

工作非常類似於承諾的 JavaScript 概念。 不過，`Promise.all` 與 `Task.WhenAll` 有些差異。 `Task.WhenAll` 的概念已成為 `durable-functions` JavaScript 模組的一部分且專屬於它。

結束等候 `Task.WhenAll` (或暫止 `context.df.Task.all`) 之後，就可知道所有函式呼叫已完成，而值也已傳回給我們。 每次呼叫 `E2_CopyFileToBlob` 都會傳回已上傳的位元組數，因此，只要合計所有這些傳回值，就能算出位元組總數。

## <a name="helper-activity-functions"></a>協助程式活動函式

如同其他範例一樣，協助程式活動函式只不過是使用 `activityTrigger` 觸發程序繫結的一般函式。 例如，`E2_GetFileList` 的 function.json 檔案看起來像下面這樣：

[!code-json[Main](~/samples-durable-functions/samples/csx/E2_GetFileList/function.json)]

實作如下：

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E2_GetFileList/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_GetFileList/index.js)]

`E2_GetFileList` 的 JavaScript 實作會使用 `readdirp` 模組以遞迴方式讀取目錄結構。

> [!NOTE]
> 您可能覺得奇怪，為何不能直接將此程式碼放入協調器函式中。 您可以這樣做，但這會違反協調器函式的基本規則之一，也就是永遠都不該執行 I/O，包括本機檔案系統存取。

`E2_CopyFileToBlob`的 *function.json* 檔案也一樣簡單：

[!code-json[Main](~/samples-durable-functions/samples/csx/E2_CopyFileToBlob/function.json)]

C# 實作也相當直接。 碰巧用到 Azure Functions 繫結的一些進階功能 (也就是使用 `Binder` 參數)，但基於本逐步解說的目的，您不必擔心這些細節。

### <a name="c"></a>C#

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E2_CopyFileToBlob/run.csx)]

### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

JavaScript 實作無法存取 Azure Functions 的 `Binder` 功能，所以 [Azure Storage SDK for Node](https://github.com/Azure/azure-storage-node) 取代之。 請注意 SDK 需要 `AZURE_STORAGE_CONNECTION_STRING` 應用程式設定。

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_CopyFileToBlob/index.js)]

實作會從磁碟載入檔案，並以非同步方式將內容串流至 "backups" 容器中相同名稱的 blob。 傳回值是複製到儲存體的位元組數，協調器函式接著會利用此值來計算的總數。

> [!NOTE]
> 這是將 I/O 作業移入 `activityTrigger` 函式中的最佳範例。 不僅可將工作分散至許多不同的虛擬機器，還可讓您享受到檢查點檢查進度的好處。 如果主機處理序由於任何原因而終止，您會知道哪些上傳已完成。

## <a name="run-the-sample"></a>執行範例

您可以藉由傳送下列 HTTP POST 要求來啟動協調流程。

```
POST http://{host}/orchestrators/E2_BackupSiteContent
Content-Type: application/json
Content-Length: 20

"D:\\home\\LogFiles"
```

> [!NOTE]
> 您叫用的 `HttpStart`函式僅適用於 JSON 格式的內容。 因此，需要 `Content-Type: application/json` 標頭，而且目錄路徑會編碼為 JSON 字串。

此 HTTP 要求將會觸發 `E2_BackupSiteContent` 協調器並傳遞字串 `D:\home\LogFiles` 當作參數。 回應提供的連結可取得備份作業的狀態：

```
HTTP/1.1 202 Accepted
Content-Length: 719
Content-Type: application/json; charset=utf-8
Location: http://{host}/admin/extensions/DurableTaskExtension/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

(...trimmed...)
```

根據您在應用程式函式中有多少記錄檔而定，這項作業可能需要幾分鐘才能完成。 您可以在前一個 HTTP 202 回應的 `Location` 標頭中查詢 URL，以取得最新狀態。

```
GET http://{host}/admin/extensions/DurableTaskExtension/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

```
HTTP/1.1 202 Accepted
Content-Length: 148
Content-Type: application/json; charset=utf-8
Location: http://{host}/admin/extensions/DurableTaskExtension/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

{"runtimeStatus":"Running","input":"D:\\home\\LogFiles","output":null,"createdTime":"2017-06-29T18:50:55Z","lastUpdatedTime":"2017-06-29T18:51:16Z"}
```

在此情況下，函式仍在執行中。 您可以看到已儲存至協調器狀態的輸入，以及上次更新時間。 您可以繼續使用 `Location` 標頭值來輪詢是否已完成。 當狀態為 "Completed" 時，您會看到類似下列的 HTTP 回應值：

```
HTTP/1.1 200 OK
Content-Length: 152
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":"D:\\home\\LogFiles","output":452071,"createdTime":"2017-06-29T18:50:55Z","lastUpdatedTime":"2017-06-29T18:51:26Z"}
```

現在，您可以看到協調流程已完成，以及大約耗費多少時間完成。 您還會看到 `output` 欄位的值，這指出大約已上傳 450 KB 的記錄。

## <a name="visual-studio-sample-code"></a>Visual Studio 範例程式碼

以下是 Visual Studio 專案中的單一 C# 檔案所示範的協調流程：

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs)]

## <a name="next-steps"></a>後續步驟

此範例已說明如何實作展開傳送/收合傳送模式。 下一個範例示範如何使用[長期計時器](durable-functions-timers.md)來實作監視模式。

> [!div class="nextstepaction"]
> [執行監視範例](durable-functions-monitor.md)
