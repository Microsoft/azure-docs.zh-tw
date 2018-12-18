---
title: 在 Azure Log Analytics 中收集自訂記錄 | Microsoft Docs
description: Log Analytics 可從 Windows 和 Linux 電腦上的文字檔收集事件。  本文說明如何定義新的自訂記錄檔，以及它們在 Log Analytics 工作區中建立的記錄詳細資料。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.assetid: aca7f6bb-6f53-4fd4-a45c-93f12ead4ae1
ms.service: log-analytics
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/27/2018
ms.author: bwren
ms.component: ''
ms.openlocfilehash: 5eab8e4bf6b1aa90a9eef3e26dfc3020e3e3179b
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423505"
---
# <a name="custom-logs-in-log-analytics"></a>Log Analytics 中的自訂記錄檔
Log Analytics 中的「自訂記錄檔」資料來源可讓您從 Windows 和 Linux 電腦上的文字檔案收集事件。 許多應用程式會將資訊記錄到文字檔而非標準的記錄服務，例如 Windows 事件記錄檔或 Syslog。  在收集之後，您就可以使用 Log Analytics 的[自訂欄位](log-analytics-custom-fields.md)功能將記錄中的每一筆記錄剖析成個別欄位。

![自訂記錄檔收集](media/log-analytics-data-sources-custom-logs/overview.png)

要收集的記錄檔必須符合下列準則。

- 記錄檔必須是每行一個項目，或在每個項目開頭使用符合下列其中一種格式的時間戳記。

    YYYY-MM-DD HH:MM:SS <br>M/D/YYYY HH:MM:SS AM/PM<br>Mon DD, YYYY HH:MM:SS<br />yyMMdd HH:mm:ss<br />ddMMyy HH:mm:ss<br />MMM d hh:mm:ss<br />dd/MMM/yyyy:HH:mm:ss zzz<br />yyyy-MM-ddTHH:mm:ssK

- 記錄檔不得使用會以新的項目覆寫檔案的循環記錄或記錄輪替。
- 記錄檔必須使用 ASCII 或 UTF-8 編碼。  不支援其他格式，例如 UTF-16。

>[!NOTE]
>如果記錄檔中有重複的項目，Log Analytics 會收集這些項目。  不過，搜尋結果會不一致，篩選結果所顯示的事件會比結果計數更多。  您必須驗證記錄，以判定建立該記錄的應用程式是否導致此行為，若可以的話，請先處理此問題，再建立自訂記錄集合定義。  
>
  
>[!NOTE]
> 如果您的應用程式每天建立新記錄檔，或記錄檔已到達特定大小，則適用於 Linux 的 Log Analytics 代理程式在重新啟動之前，並不會探索記錄檔。 這是因為代理程式只會在啟動時，針對有指定記錄的模式進行列舉及開始監視，因此，您必須藉由讓代理程式自動重新啟動來因應此狀況。  適用於 Windows 的 Log Analytics 代理程式沒有此限制。  
>

## <a name="defining-a-custom-log"></a>定義自訂記錄檔
使用下列程序來定義自訂記錄檔。  如需新增自訂記錄檔之範例的逐步解說，請捲動到本文結尾處。

### <a name="step-1-open-the-custom-log-wizard"></a>步驟 1. 開啟自訂記錄檔精靈
自訂記錄檔精靈會在 Azure 入口網站中執行，並可讓您定義要收集的新自訂記錄檔。

1. 在 Azure 入口網站中，選取 [Log Analytics] > 您的工作區 > [進階設定]。
2. 按一下 [資料] > [自訂記錄]。
3. 根據預設，所有組態變更都會自動發送給所有代理程式。  若是 Linux 代理程式，組態檔會傳送給 Fluentd 資料收集器。  如果您想以手動方式在每個 Linux 代理程式上修改這個檔案，只要取消核取 *[Apply below configuration to my Linux machines]* \(將下列設定套用至我的 Linux 機器) 方塊即可。
4. 按一下 [新增+]  開啟自訂記錄檔精靈。

### <a name="step-2-upload-and-parse-a-sample-log"></a>步驟 2. 上傳和剖析範例記錄檔
一開始您要上傳自訂記錄檔的範例。  精靈會剖析並顯示此檔案中的項目以供您驗證。  Log Analytics 會使用您指定的分隔符號來識別每一筆記錄。

**新行字元** 是預設的分隔符號，並且會用於每行一個項目的記錄檔。  如果一行的開頭是其中一種可用格式的日期和時間，則您可以指定 **時間戳記** 分隔符號，其可支援跨越多行的多個項目。

如果使用時間戳記分隔符號，則儲存在 Log Analytics 中的每一筆記錄的 TimeGenerated 屬性會填入針對記錄檔中的該項目指定的日期/時間。  如果使用新行字元分隔符號，則 TimeGenerated 會填入 Log Analytics 收集項目時的日期和時間。


1. 按一下 [瀏覽]  並瀏覽至範例檔案。  請注意，在某些瀏覽器中，這個按鈕可能標示為 [選擇檔案]  。
2. 按 [下一步] 。
3. 自訂記錄檔精靈會上傳檔案並列出其識別的記錄。
4. 變更用來識別新記錄的分隔符號，並選取最能識別記錄檔中的記錄的分隔符號。
5. 按 [下一步] 。

### <a name="step-3-add-log-collection-paths"></a>步驟 3. 新增記錄檔收集路徑
您必須在代理程式上定義一個或多個它可以在其中找到自訂記錄檔的路徑。  您可以提供該記錄檔的特定路徑和名稱，或者您可以使用萬用字元為該名稱指定路徑。 這可支援每天會建立一個新檔案的應用程式或在一個檔案到達特定大小時提供支援。 您也可以為單一記錄檔提供多個路徑。

例如，應用程式可能會每天建立一個記錄檔，且名稱中會包含日期，如同 log20100316.txt。 這類記錄檔的模式可能是 *log\*.txt*，而這會套用到任何遵循應用程式命名配置的記錄檔。

>[!NOTE]
> 如果您的應用程式每天建立新記錄檔，或記錄檔已到達特定大小，則適用於 Linux 的 Log Analytics 代理程式在重新啟動之前，並不會探索記錄檔。 這是因為代理程式只會在啟動時，針對有指定記錄的模式進行列舉及開始監視，因此，您必須藉由讓代理程式自動重新啟動來因應此狀況。  適用於 Windows 的 Log Analytics 代理程式沒有此限制。  
>

下表提供可用來指定不同記錄檔的有效模式範例。

| 說明 | Path |
|:--- |:--- |
| Windows 代理程式上的 *C:\Logs* 中，副檔名為 .txt 的所有檔案 |C:\Logs\\\*.txt |
| Windows 代理程式上的 *C:\Logs* 中，名稱開頭為 log 且副檔名為 .txt 的所有檔案 |C:\Logs\log\*.txt |
| Windows 代理程式上的 */var/log/audit* 中副檔名為 .txt 的所有檔案 |/var/log/audit/*.txt |
| Linux 代理程式上的 */var/log/audit* 中，名稱開頭為 log 且副檔名為 .txt 的所有檔案 |/var/log/audit/log\*.txt |

1. 選取 Windows 或 Linux 以指定您要新增的路徑格式。
2. 輸入路徑並按一下 [+] **+** 按鈕。
3. 針對任何其他路徑重複此程序。

### <a name="step-4-provide-a-name-and-description-for-the-log"></a>步驟 4. 提供記錄檔的名稱和描述
您指定的名稱將用於如上所述的記錄檔類型。  它一定會以 _CL 結尾以將自己辨別為自訂記錄檔。

1. 輸入記錄檔的名稱。  **\__CL** 尾碼會自動提供。
2. 新增選擇性的 [描述] 。
3. 按 [下一步]  來儲存自訂記錄檔的定義。

### <a name="step-5-validate-that-the-custom-logs-are-being-collected"></a>步驟 5。 驗證會收集自訂記錄檔
最多可能需要一小時的時間，新的自訂記錄檔中的初始資料才會出現在 Log Analytics 中。  從您定義自訂記錄檔之後，它就會開始從您指定的路徑中所找到的記錄檔收集項目。  它不會保留您在建立自訂記錄檔期間上傳的項目，但它會收集它所找出之記錄檔中既有的項目。

一旦 Log Analytics 開始從自訂記錄檔收集，就能透過「記錄檔搜尋」取得其記錄。  請使用您提供給自訂記錄檔的名稱來做為查詢中的 [類型]  。

> [!NOTE]
> 如果搜尋中遺漏 RawData 屬性，您可能需要先關閉再重新開啟瀏覽器。
>
>

### <a name="step-6-parse-the-custom-log-entries"></a>步驟 6. 剖析自訂記錄檔項目
整個記錄檔項目會儲存在稱為 **RawData**的單一屬性中。  您很可能會想要將每個項目中的不同資訊片段，分成儲存在記錄中的個別屬性。  您可以使用 Log Analytics 的 [自訂欄位](log-analytics-custom-fields.md) 功能來這麼做。

這裡並未提供用來剖析自訂記錄檔項目的詳細步驟。  如需這項資訊，請參閱 [自訂欄位](log-analytics-custom-fields.md) 文件。

## <a name="removing-a-custom-log"></a>移除自訂記錄
在 Azure 入口網站中使用下列程序來移除您先前定義的自訂記錄。

1. 從工作區 [進階設定] 的 [資料] 功能表中，選取 [自訂記錄] 以列出所有自訂記錄。
2. 按一下要移除之自訂記錄旁邊的 [移除]。


## <a name="data-collection"></a>資料收集
Log Analytics 會從每個自訂記錄檔收集新的項目，間隔大約為每 5 分鐘。  代理程式會記錄它在從中收集項目的每個記錄檔中的位置。  如果代理程式離線一段時間，Log Analytics 即會從上次停止的地方收集項目，即使這些項目是在代理程式離線時所建立亦同。

整個記錄檔項目的內容會寫入到稱為 **RawData**的單一屬性。  您可以將此屬性剖析成多個屬性，以在建立自訂記錄檔之後透過定義 [自訂欄位](log-analytics-custom-fields.md) 來個別分析和搜尋。

## <a name="custom-log-record-properties"></a>自訂記錄檔記錄的屬性
自訂記錄檔記錄的類型具有您提供的記錄檔名稱和下表中的屬性。

| 屬性 | 說明 |
|:--- |:--- |
| TimeGenerated |Log Analytics 收集記錄時的日期和時間。  如果記錄使用以時間為基礎的分隔符號，則這是從項目收集到的時間。 |
| SourceSystem |收集記錄的來源代理程式類型。 <br> OpsManager - Windows 代理程式，直接連線或由 System Center Operations Manager 管理 <br> Linux – 所有的 Linux 代理程式 |
| RawData |所收集項目的完整文字。 |
| ManagementGroupName |System Center Operations Manager 代理程式的管理群組名稱。  若為其他代理程式，此為 AOI-\<工作區 ID\> |

## <a name="log-searches-with-custom-log-records"></a>使用自訂記錄檔記錄來記錄搜尋
和來自任何其他資料來源的記錄一樣，來自自訂記錄檔的記錄會儲存在 Log Analytics 工作區中。  其類型會符合您在定義記錄檔時提供的名稱，因此您可以在搜尋中使用 [類型] 屬性，以擷取從特定記錄檔收集而來的記錄。

下表提供從自訂記錄檔擷取記錄之記錄檔搜尋的不同範例。

| 查詢 | 說明 |
|:--- |:--- |
| MyApp_CL |所有事件來自名為 MyApp_CL 的自訂記錄檔。 |
| MyApp_CL &#124; where Severity_CF=="error" |來自 MyApp_CL 自訂記錄檔且在 *Severity_CF* 自訂欄位中的值為 *error* 的所有事件。 |


## <a name="sample-walkthrough-of-adding-a-custom-log"></a>新增自訂記錄檔的範例逐步解說
下面的章節會逐步解說建立自訂記錄檔的範例。  所收集的範例記錄在每一行只有一個項目，其開頭為日期和時間，然後是以逗號分隔的程式碼、狀態及訊息欄位。  下面顯示了幾個範例項目。

    2016-03-10 01:34:36 207,Success,Client 05a26a97-272a-4bc9-8f64-269d154b0e39 connected
    2016-03-10 01:33:33 208,Warning,Client ec53d95c-1c88-41ae-8174-92104212de5d disconnected
    2016-03-10 01:35:44 209,Success,Transaction 10d65890-b003-48f8-9cfc-9c74b51189c8 succeeded
    2016-03-10 01:38:22 302,Error,Application could not connect to database
    2016-03-10 01:31:34 303,Error,Application lost connection to database

### <a name="upload-and-parse-a-sample-log"></a>上傳和剖析範例記錄檔
我們提供其中一個記錄檔，並可以看到將會收集的事件。  在此案例中，新行字元足以做為分隔符號。  但如果記錄檔中的單一項目跨越多行，就必須使用時間戳記分隔符號。

![上傳和剖析範例記錄檔](media/log-analytics-data-sources-custom-logs/delimiter.png)

### <a name="add-log-collection-paths"></a>新增記錄檔收集路徑
記錄檔將位於 *C:\MyApp\Logs*。  每一天都會建立一個新檔案，且其名稱中包括「appYYYYMMDD.log」 模式的日期。  此記錄檔的完整模式為 *C:\MyApp\Logs\\\*.log*。

![記錄檔收集路徑](media/log-analytics-data-sources-custom-logs/collection-path.png)

### <a name="provide-a-name-and-description-for-the-log"></a>提供記錄檔的名稱和描述
我們使用 *MyApp_CL* 這個名稱並輸入 [描述]。

![記錄檔名稱](media/log-analytics-data-sources-custom-logs/log-name.png)

### <a name="validate-that-the-custom-logs-are-being-collected"></a>驗證會收集自訂記錄檔
我們使用 *Type=MyApp_CL* 這個查詢，以從收集到的記錄檔傳回所有記錄。

![無自訂欄位的記錄檔查詢](media/log-analytics-data-sources-custom-logs/query-01.png)

### <a name="parse-the-custom-log-entries"></a>剖析自訂記錄檔項目
我們使用「自訂欄位」來定義 *EventTime*、*Code*、*Status* 和 *Message* 欄位，而且我們可以在查詢所傳回的記錄中看到差異。

![有自訂欄位的記錄檔查詢](media/log-analytics-data-sources-custom-logs/query-02.png)

## <a name="next-steps"></a>後續步驟
* 使用[自訂欄位](log-analytics-custom-fields.md)，以將自訂記錄中的項目剖析至個別欄位。
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。
