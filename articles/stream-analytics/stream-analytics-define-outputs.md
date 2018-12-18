---
title: 了解來自 Azure 串流分析的輸出
description: 本文說明 Azure 串流分析中所提供的資料輸出選項，包括適用於分析結果的 Power BI。
services: stream-analytics
author: jasonwhowell
ms.author: jasonh
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/14/2018
ms.openlocfilehash: e9c09d31af1b6ea214ae2d0fc6fd7399c07fd8c0
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434533"
---
# <a name="understand-outputs-from-azure-stream-analytics"></a>了解來自 Azure 串流分析的輸出
本文說明適用於 Azure 串流分析作業的不同輸出類型。 輸出可讓您存放並儲存串流分析作業的結果。 透過使用輸出資料，您可以對資料進行進一步的商務分析及資料倉儲處理。 

在您設計串流分析查詢時，請使用 [INTO 子句](https://msdn.microsoft.com/azure/stream-analytics/reference/into-azure-stream-analytics) \(英文\) 參考輸出的名稱。 您可以針對每個作業使用單一輸出，或視需要在查詢中提供多個 INTO 子句來針對每個串流作業使用多個輸出。

若要建立、編輯及測試串流分析作業輸出，您可以使用 [Azure 入口網站](stream-analytics-quick-create-portal.md#configure-output-to-the-job)、[Azure PowerShell](stream-analytics-quick-create-powershell.md#configure-output-to-the-job)、[.Net API](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.streamanalytics.ioutputsoperations?view=azure-dotnet) \(英文\)、[REST API](https://docs.microsoft.com/rest/api/streamanalytics/stream-analytics-output) \(英文\)，以及 [Visual Studio](stream-analytics-quick-create-vs.md)。

某些輸出類型支援[資料分割](#partitioning)，且[輸出批次大小](#output-batch-size)會變化以達到最佳輸送量。


## <a name="azure-data-lake-store"></a>Azure Data Lake Store
串流分析支援 [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/)。 Azure Data Lake Store 是容納巨量資料分析工作負載的企業級超大規模存放庫。 Data Lake Store 可讓您存放任何大小、類型和擷取速度的資料，以便進行運作和探究分析。 串流分析需要經過授權，才能存取 Data Lake Store。

Azure 中國 (21Vianet) 和 Azure 德國 (T-Systems International) 區域目前無法從串流分析產生 Azure Data Lake Store 輸出。

### <a name="authorize-an-azure-data-lake-store-account"></a>授權 Azure Data Lake Store 帳戶

1. 在 Azure 入口網站中選取 Data Lake Storage 作為輸出時，系統會提示您授權與現有 Data Lake Store 的連線。  

   ![授權 Data Lake Store](./media/stream-analytics-define-outputs/06-stream-analytics-define-outputs.png)  

2. 如果您已經可以存取 Data Lake Store，請選取 [立即授權]，螢幕會出現一個頁面，指出「正在重新導向至授權」。 授權成功之後，您會看到設定 Data Lake Store 輸出的頁面。

3. 驗證 Data Lake Store 帳戶之後，您可以設定 Data Lake Store 輸出的屬性。 下表是設定 Data Lake Store 輸出的屬性名稱及其描述的清單。

   ![授權 Data Lake Store](./media/stream-analytics-define-outputs/07-stream-analytics-define-outputs.png)  

| 屬性名稱 | 說明 | 
| --- | --- |
| 輸出別名 | 此為易記名稱，用於在查詢中將查詢輸出指向這個 Data Lake Store。 | 
| 帳戶名稱 | 您傳送輸出的 Data Lake Storage 帳戶名稱。 您會看到您的訂用帳戶中可用的 Data Lake Store 帳戶的下拉式清單。 |
| 路徑前置詞模式 | 用來在指定的 Data Lake Store 帳戶中寫入檔案的檔案路徑。 您可以指定一或多個 {date} 和 {time} 變數的執行個體。</br><ul><li>範例 1：folder1/logs/{date}/{time}</li><li>範例 2：folder1/logs/{date}</li></ul><br>所建立的資料夾結構時間戳記遵循 UTC 而非當地時間。</br><br>如果檔案路徑模式不包含尾端的 "/"，則會將檔案路徑中的最後一個模式視為檔案名稱前置詞。 </br></br>系統會在下列情況下建立新檔案：<ul><li>輸出結構描述中出現變更</li><li>從外部或內部重新啟動作業。</li></ul> |
| 日期格式 | 選用。 如果前置詞路徑中使用日期權杖，您可以選取組織檔案要用的日期格式。 範例：YYYY/MM/DD |
|時間格式 | 選用。 如果前置詞路徑中使用時間權杖，請指定組織檔案要用的時間格式。 目前唯一支援的值為 HH。 |
| 事件序列化格式 | 輸出資料的序列化格式。 支援 JSON、CSV 和 Avro。| 
| 編碼 | 如果使用 CSV 或 JSON 格式，則必須指定編碼。 UTF-8 是目前唯一支援的編碼格式。|
| 分隔符號 | 僅適用於 CSV 序列化。 串流分析可支援多種序列化 CSV 資料常用的分隔符號。 支援的值是逗號、分號、空格、索引標籤和分隔號。|
| 格式 | 僅適用於 JSON 序列化。 分隔的行會指定輸出的格式化方式為利用新行分隔每個 JSON 物件。 陣列會指定輸出將會格式化為 JSON 物件的陣列。 只有在作業停止或串流分析已移動到下一個時間範圍時，才會關閉這個陣列。 一般情況下，最好使用分行的 JSON，因為它不需要任何特殊處理，同時仍會寫入輸出檔案。|

### <a name="renew-data-lake-store-authorization"></a>更新 Data Lake Store 授權
如果您在建立作業之後或上次驗證過後變更了密碼，則需要重新驗證您的 Data Lake Store 帳戶。 如果您不重新驗證，您的作業將不會產生輸出結果，並在作業記錄檔中會顯示錯誤，指出需要重新授權。 目前有一個限制，即每隔 90 天必須針對 Data Lake Store 輸出的所有工作，以手動方式更新驗證 Token。 

若要更新授權，請 [停止] 您的工作 > 移至 Data Lake Store 輸出 > 按一下 [更新授權] 連結，螢幕很快就會出現一個頁面，指出「正在重新導向至授權...」。此頁面會自動關閉，並在成功的情況下指出「已成功更新授權」。 接著，您必須按一下頁面底部的 [儲存]，然後可以從**上次停止的時間**重新開始您的工作繼續，以避免資料遺失。

![授權 Data Lake Store](./media/stream-analytics-define-outputs/08-stream-analytics-define-outputs.png)  

## <a name="sql-database"></a>SQL Database
[Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 做為資料輸出。 串流分析作業會寫入至 Azure SQL Database 中的現有資料表。  資料表結構描述必須完全符合作業輸出的欄位及其類型。 您也可以透過 SQL Database 輸出選項，將 [Azure SQL 資料倉儲](https://azure.microsoft.com/documentation/services/sql-data-warehouse/)指定為輸出。 若要深入了解如何改善寫入輸送量，請參閱[使用 Azure SQL DB 作為輸出的串流分析](stream-analytics-sql-output-perf.md)一文。 下表列出屬性名稱及其描述以建立 SQL Database 輸出。

| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 |此為易記名稱，用於在查詢中將查詢輸出指向這個資料庫。 |
| 資料庫 | 您傳送輸出的目標資料庫名稱。 |
| 伺服器名稱 | SQL Database 伺服器名稱。 |
| 使用者名稱 | 具有寫入資料庫存取權限的使用者名稱。 |
| 密碼 | 連接到資料庫的密碼 |
| 資料表 | 要在其中寫入輸出的資料表名稱。 資料表名稱會區分大小寫，且資料表的結構描述應該完全符合作業輸出所產生的欄位數目和其類型。 |

> [!NOTE]
> 目前「串流分析」中的工作輸出支援 Azure SQL Database 供應項目。 不過，不支援執行已連結資料庫之 SQL Server 的「Azure 虛擬機器」。 這在未來的版本中有可能變更。
> 

## <a name="blob-storage"></a>Blob 儲存體
若要將大量非結構化資料儲存於雲端，Blob 儲存體提供具有成本效益且可擴充的解決方案。  如需 Azure Blob 儲存體及其使用方式的簡介，請參閱 [如何使用 Blob](../storage/blobs/storage-dotnet-how-to-use-blobs.md)中的文件。

下表列出屬性名稱及其描述以建立 blob 輸出。

| 屬性名稱       | 說明                                                                      |
| ------------------- | ---------------------------------------------------------------------------------|
| 輸出別名        | 此為易記名稱，用於在查詢中將查詢輸出指向這個 blob 儲存體。 |
| 儲存體帳戶     | 您傳送輸出的儲存體帳戶名稱。               |
| 儲存體帳戶金鑰 | 與儲存體帳戶相關聯的密碼金鑰。                              |
| 儲存體容器   | 容器提供邏輯分組給儲存在 Microsoft Azure Blob 服務中的 blob。 當您將 blob 上傳至 Blob 服務時，您必須指定該 blob 的容器。 |
| 路徑格式 | 選用。 用來在指定容器中寫入 Blob 的檔案路徑模式。 <br /><br /> 在路徑模式中，您也可以選擇使用日期時間變數的一或多個執行個體，來指定 blob 的寫入頻率： <br /> {date}、{time} <br /><br />如果您已註冊[預覽](https://aka.ms/ASAPreview)，可以從事件資料指定一個自訂 {field} 名稱來分割 Blob。 欄位名稱是英數字元，可以包含空格、連字號和底線。 自訂欄位的限制包含下列各項： <ul><li>不區分大小寫 (無法分辨資料行 "ID" 和資料行 "id")</li><li>不允許巢狀欄位 (改為在作業查詢中使用別名來「壓平」欄位)</li><li>運算式不能作為欄位名稱使用。</li></ul> <br /><br /> 預覽也允許在路徑中使用自訂日期/時間格式的指定名稱設定。 自訂日期和時間格式一次只能指定一項，兩側必須加上 {datetime:\<specifier>} 關鍵字。 允許輸入的 \<specifier> 為 yyyy、MM、M、dd、d、HH、H、mm、m、ss 或 s。 {datetime:\<specifier>} 關鍵字能在路徑中使用多次，以構成自訂日期/時間設定。 <br /><br />範例： <ul><li>範例 1：cluster1/logs/{date}/{time}</li><li>範例 2：cluster1/logs/{date}</li><li>範例 3 (預覽)：cluster1/{client_id}/{date}/{time}</li><li>範例 4 (預覽)：cluster1/{datetime:ss}/{myField} 其中查詢為：SELECT data.myField AS myField FROM Input;</li><li>範例 5 (預覽)：cluster1/year={datetime:yyyy}/month={datetime:MM}/day={datetime:dd}</ul><br /><br />所建立的資料夾結構時間戳記遵循 UTC 而非當地時間。<br /><br/>檔案命名會遵循下列慣例： <br /><br />{路徑前置詞模式}/schemaHashcode_Guid_Number.extension<br /><br />範例輸出檔案︰<ul><li>Myoutput/20170901/00/45434_gguid_1.csv</li>  <li>Myoutput/20170901/01/45434_gguid_1.csv</li></ul> |
| 日期格式 | 選用。 如果前置詞路徑中使用日期權杖，您可以選取組織檔案要用的日期格式。 範例：YYYY/MM/DD |
| 時間格式 | 選用。 如果前置詞路徑中使用時間權杖，請指定組織檔案要用的時間格式。 目前唯一支援的值為 HH。 |
| 事件序列化格式 | 輸出資料的序列化格式。  支援 JSON、CSV 和 Avro。 |
| 編碼    | 如果使用 CSV 或 JSON 格式，則必須指定編碼。 UTF-8 是目前唯一支援的編碼格式。 |
| 分隔符號   | 僅適用於 CSV 序列化。 串流分析可支援多種序列化 CSV 資料常用的分隔符號。 支援的值是逗號、分號、空格、索引標籤和分隔號。 |
| 格式      | 僅適用於 JSON 序列化。 分隔的行會指定輸出的格式化方式為利用新行分隔每個 JSON 物件。 陣列會指定輸出將會格式化為 JSON 物件的陣列。 只有在作業停止或串流分析已移動到下一個時間範圍時，才會關閉這個陣列。 一般情況下，最好使用分行的 JSON，因為它不需要任何特殊處理，同時仍會寫入輸出檔案。 |

使用 blob 儲存體作為輸出時，在下列情況下 blob 中會建立新檔案：

* 如果檔案超過允許的區塊數目上限 (目前為 50,000 個)。 可以在未達到允許 blob 大小上限的情況下，達到允許的區塊數目上限。 比方說，如果輸出速率很高，您可看到每個區塊有更多位元組，且檔案大小更大。 如果輸出速率很低，每個區塊都會具有較少的資料，且檔案大小會較小。
* 如果輸出中有結構描述變更，且輸出格式需要固定的結構描述 (CSV 與 Avro)。  
* 如果作業已重新啟動，無論是由使用者於外部停止並啟動它，或是於內部重新啟動以進行系統維護或錯誤復原。  
* 如果完全分割查詢，則系統會為每個輸出分割區建立新檔案。  
* 如果使用者已刪除儲存體帳戶的檔案或容器。  
* 如果使用路徑前置詞模式將輸出進行時間分割，則會在查詢移至下一個小時的時候使用新的 Blob。
* 如果輸出是由自訂欄位進行分割，系統會針對每個分割區索引鍵建立新的 blob (若不存在的話)。
* 如果輸出是由自訂欄位進行分割，其中分割區索引鍵基數超過 8000，則系統可能會針對每個分割區索引鍵建立新的 blob。

## <a name="event-hub"></a>事件中樞
[Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/) 服務是具高延展性的發佈-訂閱事件擷取器。 它每秒可以收集數百萬個事件。 當串流分析作業成為另一個串流作業的輸入時，就會使用事件中樞作為輸出。

設定事件中樞串流做為輸出時，需要幾個參數。

| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 | 此為易記名稱，用於在查詢中將查詢輸出指向這個事件中樞。 |
| 事件中樞命名空間 |事件中樞命名空間是一組訊息實體的容器。 建立新的事件中樞時，也會建立事件中樞命名空間。 |
| 事件中樞名稱 | 事件中樞輸出的名稱。 |
| 事件中樞原則名稱 | 共用的存取原則，可以在事件中樞的 [設定] 索引標籤上建立。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰。 |
| 事件中樞原則金鑰 | 用來驗證事件中樞命名空間存取權的共用存取金鑰。 |
| 分割區索引鍵資料行 [選擇性] | 這個資料行包含事件中樞輸出的資料分割索引鍵。 |
| 事件序列化格式 | 輸出資料的序列化格式。  支援 JSON、CSV 和 Avro。 |
| 編碼 | 對於 CSV 和 JSON 而言，UTF-8 是目前唯一支援的編碼格式。 |
| 分隔符號 | 僅適用於 CSV 序列化。 串流分析可支援多種以 CSV 格式序列化資料常用的分隔符號。 支援的值是逗號、分號、空格、索引標籤和分隔號。 |
| 格式 | 僅適用於 JSON 序列化。 分隔的行會指定輸出的格式化方式為利用新行分隔每個 JSON 物件。 陣列會指定輸出將會格式化為 JSON 物件的陣列。 只有在作業停止或串流分析已移動到下一個時間範圍時，才會關閉這個陣列。 一般情況下，最好使用分行的 JSON，因為它不需要任何特殊處理，同時仍會寫入輸出檔案。 |

## <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com/) 當做串流分析工作的輸出，來為分析結果提供豐富的視覺體驗。 這項功能可以用於可運作的儀表板、產生報告，以及度量驅動的報告。

Azure 中國 (21Vianet) 和 Azure 德國 (T-Systems International) 區域目前無法從串流分析產生 Power BI 輸出。

### <a name="authorize-a-power-bi-account"></a>授權 Power BI 帳戶
1. 在 Azure 入口網站中選取 Power BI 作為輸出時，您會收到提示，要授權現有的 Power BI 使用者或建立新的 Power BI 帳戶。  
   
   ![授權 Power BI 使用者](./media/stream-analytics-define-outputs/01-stream-analytics-define-outputs.png)  

2. 如果您尚未擁有帳戶，請建立新帳戶，然後按一下 [立即授權]。  會顯示下列頁面：
   
   ![Azure 帳戶 Power BI](./media/stream-analytics-define-outputs/02-stream-analytics-define-outputs.png)  

3. 在此步驟中，請提供工作或學校帳戶以授權 Power BI 輸出。 如果您尚未註冊 Power BI，請選擇 [立即註冊]。 您用於 Power BI 的工作或學校帳戶可能與您目前用來登入的 Azure 訂用帳戶不同。

### <a name="configure-the-power-bi-output-properties"></a>設定 Power BI 輸出屬性
驗證 Power BI 帳戶之後，您可以設定 Power BI 輸出的屬性。 下表是設定 Power BI 輸出的屬性名稱清單及其描述。

| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 |此為易記名稱，用於在查詢中將查詢輸出指向這個 Power BI 輸出。 |
| 群組工作區 |若要與其他 Power BI 使用者共用資料，您可以選取 Power BI 帳戶內的群組，若您不希望寫入群組，請選擇「我的工作區」。  更新現有的群組需要更新 Power BI 驗證。 |
| 資料集名稱 |提供 Power BI 輸出需要使用的資料集名稱 |
| 資料表名稱 |提供 Power BI 輸出資料集的資料表名稱。 目前，串流分析工作的 Power BI 輸出中，一個資料集只能有一個資料表 |

如需設定 Power BI 輸出和儀表板的逐步解說，請參閱 [Azure 串流分析與 Power BI](stream-analytics-power-bi-dashboard.md) 文章。

> [!NOTE]
> 未在 Power BI 儀表板中明確建立資料集和資料表。 當作業開始並將輸出提取至 Power BI 時，就會自動填入資料集和資料表。 請注意，如果作業查詢沒有產生任何結果，則不會建立資料集和資料表。 請注意，如果 Power BI 已經具有與串流分析作業中提供的名稱相同的資料集和資料表，則會覆寫現有的資料。
> 

### <a name="schema-creation"></a>建立結構描述
如果 Power BI 資料集和資料表尚不存在，則 Azure 串流分析會代表使用者建立一個。 在其他情況下，則會以新的值更新資料表。 目前的限制是一個資料集內只能存在一個資料表。

### <a name="data-type-conversion-from-stream-analytics-to-power-bi"></a>從串流分析至 Power BI 的資料類型轉換
如果輸出結構描述變更，則 Azure 串流分析會在執行階段動態更新資料模型。 所有資料行名稱變更、資料行類型變更以及資料行新增或移除都會加以追蹤。

如果 POWER BI 資料集和資料表不存在，此資料表包含從[串流分析資料類型](https://msdn.microsoft.com/library/azure/dn835065.aspx)至 Power BI [實體資料模型 (EDM) 類型](https://powerbi.microsoft.com/documentation/powerbi-developer-walkthrough-push-data/)的資料類型轉換。

從串流分析 | 至 Power BI
-----|-----|------------
bigint | Int64
nvarchar(max) | 字串
Datetime | DateTime
float | 兩倍
記錄陣列 | 字串類型、常數值 "IRecord" 或 "IArray"

### <a name="schema-update"></a>更新結構描述
串流分析會根據輸出中的第一組事件來推斷資料模型結構描述。 之後會視需要更新資料模型結構描述，以容納原始結構描述放不下的連入事件。

應該避免 `SELECT *` 查詢，以防止跨越資料列的動態結構描述更新。 除了潛在的效能影響以外，也可能導致結果所花費的時間不定。 應選取必須顯示在 Power BI 儀表板上的確切欄位。 此外，資料值應該與所選的資料類型相符。


先前/目前 | Int64 | 字串 | DateTime | 兩倍
-----------------|-------|--------|----------|-------
Int64 | Int64 | 字串 | 字串 | 兩倍
兩倍 | 兩倍 | 字串 | 字串 | 兩倍
字串 | 字串 | 字串 | 字串 |  | 字串 | 
DateTime | 字串 | 字串 |  DateTime | 字串


### <a name="renew-power-bi-authorization"></a>更新 Power BI 授權
如果您的 Power BI 帳戶密碼在建立或最後一次驗證串流分析作業之後出現變更，便必須重新驗證串流分析。 如果您的 Azure Active Directory (AAD) 租用戶上有設定多重要素驗證 (MFA)，則也需要每兩週更新一次 Power BI 授權。 此問題發生時的徵兆就是沒有工作輸出，且作業記錄檔中出現「驗證使用者錯誤」：

  ![Power BI 重新整理權杖錯誤](./media/stream-analytics-define-outputs/03-stream-analytics-define-outputs.png)  

若要解決這個問題，請停止執行工作並移至 Power BI 輸出。  選取 [更新授權] 連結，並從 [上次停止時間] 重新啟動作業以避免資料遺失。

  ![Power BI 更新授權](./media/stream-analytics-define-outputs/04-stream-analytics-define-outputs.png)  

## <a name="table-storage"></a>表格儲存體
[Azure 表格儲存體](../storage/common/storage-introduction.md)提供高可用性且可大幅擴充的儲存體，可讓應用程式自動調整來滿足使用者需求。 資料表儲存體是 Microsoft 的 NoSQL 索引鍵/屬性存放區，其中可以使用結構化資料，但結構描述的限制較少。 使用 Azure 資料表儲存資料時，資料可長期儲存而且調閱方便。

下表列出屬性名稱及其描述以建立資料表輸出。

| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 |此為易記名稱，用於在查詢中將查詢輸出指向這個資料表儲存體。 |
| 儲存體帳戶 |您傳送輸出的儲存體帳戶名稱。 |
| 儲存體帳戶金鑰 |與儲存體帳戶相關聯的存取金鑰。 |
| 資料表名稱 |資料表的名稱。 如果資料表不存在，則會建立資料表。 |
| 資料分割索引鍵 |包含資料分割索引鍵的輸出資料行名稱。 在構成實體主索引鍵第一個部分的指定資料表內，資料分割索引鍵是資料分割的唯一識別碼。 大小最高為 1 KB 的字串值。 |
| 列索引鍵 |包含資料列索引鍵的輸出資料行名稱。 資料列索引鍵是指定資料分割內實體的唯一識別碼。 它可構成實體主索引鍵的第二個部分。 資料列索引鍵是大小可能高達 1 KB 的字串值。 |
| 批次大小 |批次作業的記錄數目。 預設值 (100)通常足以應付大部分的作業。 如需修改此設定的詳細資料，請參閱[資料表批次作業規格](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.tablebatchoperation.aspx) \(英文\)。 |
 
## <a name="service-bus-queues"></a>服務匯流排佇列
[服務匯流排佇列](https://msdn.microsoft.com/library/azure/hh367516.aspx) 會採用「先進先出」(FIFO) 訊息傳遞機制。 通常會預期由接收者依訊息加入佇列的時間順序來接收和處理訊息，而且每則訊息只能由一個訊息取用者接收和處理。

下表列出屬性名稱及其描述以建立佇列輸出。

| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 |此為易記名稱，用於在查詢中將查詢輸出指向這個服務匯流排佇列。 |
| 服務匯流排命名空間 |服務匯流排命名空間是一個容器，包含一組訊息實體。 |
| Queue name |服務匯流排佇列的名稱。 |
| 佇列原則名稱 |當您建立佇列時，您也可以在 [佇列設定] 索引標籤上建立共用的存取原則。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰。 |
| 佇列原則金鑰 |用來驗證服務匯流排命名空間之存取權的共用存取金鑰 |
| 事件序列化格式 |輸出資料的序列化格式。  支援 JSON、CSV 和 Avro。 |
| 編碼 |對於 CSV 和 JSON 而言，UTF-8 是目前唯一支援的編碼格式 |
| 分隔符號 |僅適用於 CSV 序列化。 串流分析可支援多種以 CSV 格式序列化資料常用的分隔符號。 支援的值是逗號、分號、空格、索引標籤和分隔號。 |
| 格式 |僅適用於 JSON 類型。 分隔的行會指定輸出的格式化方式為利用新行分隔每個 JSON 物件。 陣列會指定輸出將會格式化為 JSON 物件的陣列。 |

分割區數目是[根據服務匯流排 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分割區索引鍵是每個分割區的唯一整數值。

## <a name="service-bus-topics"></a>服務匯流排主題
「服務匯流排佇列」提供從傳送者到接收者的一對一通訊方法，而 [服務匯流排主題](https://msdn.microsoft.com/library/azure/hh367516.aspx) 則是提供一對多的通訊形式。

下表列出屬性名稱及其描述以建立資料表輸出。

| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 |此為易記名稱，用於在查詢中將查詢輸出指向這個服務匯流排主題。 |
| 服務匯流排命名空間 |服務匯流排命名空間是一個容器，包含一組訊息實體。 建立新的事件中樞時，也會建立服務匯流排命名空間 |
| 主題名稱 |主題為訊息實體，類似於事件中樞和佇列。 它們可以收集各種裝置和服務的事件資料流。 建立主題時也會給予其特定名稱。 除非已建立訂用帳戶，否則無法使用傳送至主題的訊息，所以請確保該主題內有一或多個訂用帳戶 |
| 主題原則名稱 |當您建立主題時，您也可以在 [主題設定] 索引標籤上建立共用的存取原則。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰 |
| 主題原則金鑰 |用來驗證服務匯流排命名空間之存取權的共用存取金鑰 |
| 事件序列化格式 |輸出資料的序列化格式。  支援 JSON、CSV 和 Avro。 |
| 編碼 |如果使用 CSV 或 JSON 格式，則必須指定編碼。 UTF-8 是目前唯一支援的編碼格式 |
| 分隔符號 |僅適用於 CSV 序列化。 串流分析可支援多種以 CSV 格式序列化資料常用的分隔符號。 支援的值是逗號、分號、空格、索引標籤和分隔號。 |

分割區數目是[根據服務匯流排 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分割區索引鍵是每個分割區的唯一整數值。

## <a name="azure-cosmos-db"></a>Azure Cosmos DB
[Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/) 是全域散發的多模型資料庫服務，在全球各地提供無限的彈性調整、透過無從驗證結構描述資料模型的豐富查詢和自動索引、保證低延遲以及領先業界的完整 SLA。 若要了解串流分析的 Cosmos DB 集合選項，請參閱[以 Cosmos DB 做為輸出的串流分析](stream-analytics-documentdb-output.md)一文。

Azure 中國 (21Vianet) 和 Azure 德國 (T-Systems International) 區域目前無法從串流分析產生 Azure Cosmos DB 輸出。

> [!Note]
> 此時 Azure 串流分析僅支援使用 **SQL API** 連線至 CosmosDB。
> 尚不支援其他 Azure Cosmos DB API。 如果您將 Azure Stream Analytics 指向使用其他 API 建立的 Azure Cosmos DB 帳戶，可能無法正確儲存資料。 

下表描述用來建立 Azure Cosmos DB 輸出的屬性。
| 屬性名稱 | 說明 |
| --- | --- |
| 輸出別名 | 在您的串流分析查詢中參照此輸出時所用的別名。 |
| 接收 | Cosmos DB |
| 匯入選項 | 選擇 [從您的訂用帳戶選取 Cosmos DB]，或 [手動提供 Cosmos DB 設定]。
| 帳戶識別碼 | Cosmos DB 帳戶的名稱或端點 URI。 |
| 帳戶金鑰 | Cosmos DB 帳戶的共用存取金鑰。 |
| 資料庫 | Cosmos DB 資料庫名稱。 |
| 集合名稱模式 | 所要使用集合的集合名稱或其模式。 <br/>您可以使用選用的 {partition} 語彙基元來建構集合名稱的格式，其中的資料分割會從 0 開始。 兩個範例：  <br/>1._MyCollection_ – 必須要有一個名為 "MyCollection" 的集合。  <br/>2._MyCollection{partition}_ – 根據分割資料行。 <br/>分割資料行集合必須存在 – "MyCollection0"、"MyCollection1"、"MyCollection2" 等，依此類推。 |
| 資料分割索引鍵 | 選用。 只有當您在集合名稱模式中使用 {partition} 權杖時，才需要此索引鍵。<br/> 分割索引鍵是輸出事件中的欄位名稱，用來指定跨集合分割輸出的索引鍵。<br/> 針對單一集合輸出，可使用任何任意輸出資料行。 例如，PartitionId。 |
| 文件識別碼 |選用。 輸出事件中的欄位名稱會用來指定主索引鍵，此為插入或更新作業的依據。  

## <a name="azure-functions"></a>Azure Functions
Azure Functions 是無伺服器計算服務，可讓您依需求執行程式碼，無需明確佈建或管理基礎結構。 它可讓您實作在 Azure 或協力廠商服務中發生之事件所觸發的程式碼。  Azure Functions 回應觸發程序的這個功能使其自然輸出 Azure 串流分析。 此輸出配接器可讓使用者將串流分析連接至 Azure Functions，然後執行指令碼或程式碼片段，以回應各種不同的事件。

Azure 中國 (21Vianet) 和 Azure 德國 (T-Systems International) 區域目前無法從串流分析產生 Azure Functions 輸出。

Azure 串流分析會透過 HTTP 觸發程序叫用 Azure Functions。 新的 Azure Functions 輸出配接器可搭配下列可設定屬性使用：

| 屬性名稱 | 說明 |
| --- | --- |
| 函式應用程式 |Azure Functions 應用程式的名稱 |
| 函式 |Azure Functions 應用程式中函式的名稱 |
| Key |如果您想要使用另一個訂用帳戶中的 Azure Function，可以藉由提供存取函式的金鑰來達到這個目的 |
| 批次大小上限 |此屬性可用來針對傳送到您 Azure Function 的每個輸出批次，設定大小上限。 根據預設，此值為 256 KB |
| 批次計數上限  |正如其名稱所示，此屬性可讓您在傳送至 Azure Functions 的每個批次中，指定事件數目上限。 預設的最大批次計數值為 100 |

當 Azure 串流分析從 Azure 函式收到 413 (http 要求實體太大) 例外狀況時，它會縮減傳送至 Azure Functions 批次的大小。 在 Azure 函式程式碼中，使用這個例外狀況可確保 Azure 串流分析不會傳送過大的批次。 此外，請確認函式中使用的最大批次計數和大小值都符合串流分析入口網站中輸入的值。 

此外，如果在某個時間範圍內沒有登陸任何事件，則不會產生任何輸出。 如此一來，就不會呼叫 computeResult 函式。 此行為與內建的視窗型彙總函式一致。

## <a name="partitioning"></a>分割

下表摘要說明分割支援，和每個輸出類型的輸出寫入器數目：

| 輸出類型 | 支援分割 | 資料分割索引鍵  | 輸出寫入器數目 | 
| --- | --- | --- | --- |
| Azure Data Lake Store | 是 | 在路徑前置詞模式中使用 {date} 和 {time} 權杖。 選擇日期格式，例如 YYYY/MM/DD、DD/MM/YYYY、MM-DD-YYYY。 HH 用於時間格式。 | 遵循[完整可平行化查詢](stream-analytics-scale-jobs.md)的輸入資料分割。 | 
| 連接字串 | 是 | 以查詢中的 PARTITION BY 子句為依據 | 遵循[完整可平行化查詢](stream-analytics-scale-jobs.md)的輸入資料分割。 | 
| Azure Blob 儲存體 | 是 | 使用來自路徑模式中事件欄位的 {date} 和 {time} 權杖。 選擇日期格式，例如 YYYY/MM/DD、DD/MM/YYYY、MM-DD-YYYY。 HH 用於時間格式。 在[預覽](https://aka.ms/ASAPreview)當中，可依照單一自訂事件屬性 {fieldname} 或 {datetime:\<specifier>} 分割 Blob 輸出。 | 遵循[完整可平行化查詢](stream-analytics-scale-jobs.md)的輸入資料分割。 | 
| Azure 事件中樞 | 是 | 是 | 根據分割區對齊方式而有所不同。</br> 當輸出事件中樞分割區索引鍵與上游 (先前的) 查詢步驟同等對齊時，寫入器的數目將會和輸出事件中樞分割區的數目相同。 每個寫入器都會使用事件中樞的 [EventHubSender 類別](/dotnet/api/microsoft.servicebus.messaging.eventhubsender?view=azure-dotnet)來將事件傳送至特定的分割區。 </br> 當輸出事件中樞分割區索引鍵沒有與上游 (先前的) 查詢步驟同等對齊時，寫入器的數目將會和先前步驟中的分割區數目相同。 每個寫入器會使用 EventHubClient [SendBatchAsync 類別](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventhubclient.sendasync?view=azure-dotnet) \(英文\) 來將事件傳送至所有輸出分割區。 |
| Power BI | 否 | None | 不適用。 | 
| Azure 資料表儲存體 | 是 | 任何輸出資料行。  | 遵循[完整平行化查詢](stream-analytics-scale-jobs.md)的輸入資料分割。 | 
| Azure 服務匯流排主題 | 是 | 自動選擇。 分割區數目是根據[服務匯流排 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分割區索引鍵是每個分割區的唯一整數值。| 與輸出主題中的分割區數目相同。  |
| Azure 服務匯流排佇列 | 是 | 自動選擇。 分割區數目是根據[服務匯流排 SKU 和大小](../service-bus-messaging/service-bus-partitioning.md)。 分割區索引鍵是每個分割區的唯一整數值。| 與輸出佇列中的分割區數目相同。 |
| Azure Cosmos DB | 是 | 在集合名稱模式中使用 {partition} 權杖。 {partition} 值是根據查詢中讀得 PARTITION BY 子句。 | 遵循[完整平行化查詢](stream-analytics-scale-jobs.md)的輸入資料分割。 |
| Azure Functions | 否 | None | 不適用。 | 

## <a name="output-batch-size"></a>輸出批次大小
Azure 串流分析使用可變大小批次來處理事件及寫入至輸出。 串流分析引擎通常不會一次撰寫一個訊息，並會使用批次以提升效率。 當傳入和傳出事件頻率較高時，它便會使用較大的批次。 當輸出速率較低時，它會使用較小的批次來降低延遲。 

下表說明輸出批次處理的部分考量：

| 輸出類型 | 訊息大小上限 | 批次大小最佳化 |
| :--- | :--- | :--- | 
| Azure Data Lake Store | 請參閱 [Data Lake 儲存體限制](../azure-subscription-service-limits.md#data-lake-store-limits) | 每個寫入作業最多 4 MB |
| 連接字串 | 每個單一大量插入最多 10,000 個資料列</br>每個單一大量插入最少 100 個資料列 </br>另請參閱 [Azure SQL 限制](../sql-database/sql-database-resource-limits.md) |  每個批次一開始都會搭配批次大小上限進行大量插入，並可能會根據來自 SQL 的可重試錯誤，將批次分割成一半 (以批次大小下限為限)。 |
| Azure Blob 儲存體 | 請參閱 [Azure 儲存體限制](../azure-subscription-service-limits.md#storage-limits) | Blob 區塊大小上限為 4 MB</br>Blob 區塊計數上限為 50000 |
| Azure 事件中樞   | 每個訊息 256 KB </br>另請參閱[事件中樞限制](../event-hubs/event-hubs-quotas.md) |    當輸入輸出資料分割沒有對齊時，每個事件會個別封裝於 EventData 中，並在以訊息大小上限 (針對進階 SKU 為 1 MB) 為限的情況下，以批次的方式傳送。 </br></br>  當輸入-輸出資料分割有對齊時，多個事件會封裝為單一 EventData (以訊息大小上限為限) 並傳送。    |
| Power BI | 請參閱 [Power BI Rest API 限制](https://msdn.microsoft.com/library/dn950053.aspx) |
| Azure 資料表儲存體 | 請參閱 [Azure 儲存體限制](../azure-subscription-service-limits.md#storage-limits) | 預設值為每個單一交易 100 個實體，並可以視需要設定為較小的值。 |
| Azure 服務匯流排佇列   | 每個訊息 256 KB</br> 另請參閱[服務匯流排限制](../service-bus-messaging/service-bus-quotas.md) | 每個訊息單一事件 |
| Azure 服務匯流排主題 | 每個訊息 256 KB</br> 另請參閱[服務匯流排限制](../service-bus-messaging/service-bus-quotas.md) | 每個訊息單一事件 |
| Azure Cosmos DB   | 請參閱 [Azure Cosmos DB 限制](../azure-subscription-service-limits.md#azure-cosmos-db-limits) | 批次大小和寫入頻率會依 CosmosDB 回應進行動態調整。 </br> 串流分析不會做任何預先決定的限制。 |
| Azure Functions   | | 預設批次大小為 246 KB。 </br> 每個批次的預設事件計數為 100。 </br> 批次大小可以設定，並可以在串流分析的[輸出選項](#azure-functions)中增加或減少。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [快速入門：使用 Azure 入口網站建立串流分析作業](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301
