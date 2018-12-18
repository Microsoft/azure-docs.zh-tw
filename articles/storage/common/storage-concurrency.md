---
title: 管理 Microsoft Azure 儲存體中的並行存取
description: 如何管理 Blob、佇列、資料表和檔案服務的並行存取
services: storage
author: jasontang501
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 05/11/2017
ms.author: jasontang501
ms.component: common
ms.openlocfilehash: 9c36347db2d1678e79e5ad80cda491f77850c4a6
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525234"
---
# <a name="managing-concurrency-in-microsoft-azure-storage"></a>管理 Microsoft Azure 儲存體中的並行存取
## <a name="overview"></a>概觀
現代以網際網路為基礎的應用程式通常會有多個使用者同時檢視及更新資料。 這使得應用程式開發人員不得不認真思考如何為其使用者提供可預測的使用經驗，尤其是有多個使用者可更新相同資料的案例。 開發人員通常會考量三個主要的資料並行存取策略：  

1. 開放式並行存取 – 執行更新的應用程式將在其更新的過程中，驗證在應用程式上次讀取資料後，該資料是否有所變更。 例如，如果兩個檢視 wiki 頁面的使用者對相同的頁面進行更新，則 wiki 平台必須確定第二個更新並未覆寫第一個更新，並確定兩個使用者都了解其更新是否成功。 此策略最常用在 Web 應用程式中。
2. 封閉式並行存取 – 要執行更新的應用程式會鎖定物件以防止其他使用者更新資料，直到鎖定解除為止。 例如，在只有 master 會執行更新的主從式資料複寫案例中，master 通常會長時間保有資料的獨佔鎖定，以確保他人無法更新該資料。
3. 最後寫入為準 – 此方法會直接允許任何更新作業的執行，而不會先驗證在應用程式第一次讀取資料後，是否有任何其他應用程式更新過該資料。 此策略 (或非正式策略) 通常用在因資料分割的方式而不可能有多個使用者存取相同資料的情況中。 在處理暫時性資料串流的情況中，也可以利用此策略。  

本文將概略說明 Azure 儲存體平台如何為這三種並行存取策略提供絕佳支援，以簡化開發工作。  

## <a name="azure-storage--simplifies-cloud-development"></a>Azure 儲存體 – 簡化雲端開發
Azure 儲存體服務對這三種策略都可支援，但此服務依其設計主要是要支援增強式一致性模型，以確保在儲存體服務認可資料插入或更新作業時，所有對該資料的進一步存取都可看見最新的更新，因此這項服務在為開放式和封閉式並行存取提供完整支援的能力上是有差別的。 使用最終一致性模型的儲存體平台在一個使用者執行寫入與其他使用者可看見更新資料的時間上會有延遲，因而會使用戶端應用程式的開發複雜化，以防止使用者受到不一致的影響。  

除了選取適當的並行存取策略以外，開發人員也應注意儲存體平台隔離變更的方式，尤其是在不同交易間對相同物件的變更。 Azure 儲存體服務會使用快照隔離，讓讀取作業與寫入作業在單一資料分割內同時執行。 不同於其他隔離層級，快照隔離可確保所有讀取皆可看見資料的一致快照，即使在更新執行時亦然 – 基本上是藉由在更新交易進行處理時傳回最新的認可值。  

## <a name="managing-concurrency-in-blob-storage"></a>管理 Blob 儲存體中的並行存取
您可以選擇使用開放式或封閉式並行存取模型，來管理對 Blob 服務中的 Blob 和容器的存取。 如果您未明確指定策略，依預設會採用「最後寫入為準」。  

### <a name="optimistic-concurrency-for-blobs-and-containers"></a>Blob 和容器的開放式並行存取
儲存體服務會為每個儲存的物件指派識別碼。 此識別碼會在每次對物件執行更新作業時更新。 此識別碼會使用在 HTTP 通訊協定內定義的 ETag (實體標記) 標頭，隨附在 HTTP GET 回應中傳回至用戶端。 對此類物件執行更新的使用者可將原始 ETag 連同條件式標頭一併傳入，以確保只有在符合特定條件時才會執行更新。在此案例中，條件是 "If-Match" 標頭，它會要求儲存體服務必須確定在更新要求中指定的 ETag 值，與儲存體服務中儲存的值是相同的。  

此程序大致如下：  

1. 從儲存體服務中擷取 Blob，回應中會包含 HTTP ETag 標頭值，用來識別儲存體服務中目前的物件版本。
2. 當您更新 Blob 時，請將您在步驟 1 中接收到的 ETag 值納入您傳送至服務之要求的 **If-Match** 條件式標頭中。
3. 服務會比較要求中的 ETag 值與 Blob 目前的 ETag 值。
4. 如果 Blob 目前的 ETag 值與要求中的 **If-Match** 條件式標頭內的 ETag 不是相同版本，服務會將 412 錯誤傳回至用戶端。 這會向用戶端指出在用戶端擷取 Blob 後，有另一個程序更新過該 Blob。
5. 如果 Blob 目前的 ETag 值與要求中的 **If-Match** 條件式標頭內的 ETag 是相同版本，服務將會執行要求的作業，並更新 Blob 目前的 ETag 值，以顯示它已建立新版本。  

下列 C# 程式碼片段 (使用用戶端儲存體程式庫 4.2.0) 所顯示的簡易範例，說明如何根據從先前擷取或插入之 Blob 的屬性中存取的 ETag 值，來建構 **If-Match AccessCondition** 。 接著它會在更新 blob 時使用 **AccessCondition** 物件：**AccessCondition** 物件會將 **If-Match** 標頭新增至要求。 如果有其他程序更新了 Blob，Blob 服務將會傳回 HTTP 412 (預先指定的條件失敗) 狀態訊息。 此處可下載完整的範例： [使用 Azure 儲存體管理並行存取](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)。  

```csharp
// Retrieve the ETag from the newly created blob
// Etag is already populated as UploadText should cause a PUT Blob call
// to storage blob service which returns the etag in response.
string orignalETag = blockBlob.Properties.ETag;

// This code simulates an update by a third party.
string helloText = "Blob updated by a third party.";

// No etag, provided so orignal blob is overwritten (thus generating a new etag)
blockBlob.UploadText(helloText);
Console.WriteLine("Blob updated. Updated ETag = {0}",
blockBlob.Properties.ETag);

// Now try to update the blob using the orignal ETag provided when the blob was created
try
{
    Console.WriteLine("Trying to update blob using orignal etag to generate if-match access condition");
    blockBlob.UploadText(helloText,accessCondition:
    AccessCondition.GenerateIfMatchCondition(orignalETag));
}
catch (StorageException ex)
{
    if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
    {
        Console.WriteLine("Precondition failure as expected. Blob's orignal etag no longer matches");
        // TODO: client can decide on how it wants to handle the 3rd party updated content.
    }
    else
        throw;
}  
```

儲存體服務也包含對其他條件式標頭的支援，例如 **If-Modified-Since**、**If-Unmodified-Since** 和 **If-None-Match** 及其組合。 如需詳細資訊，請參閱 MSDN 上的 [為 Blob 服務作業指定條件式標頭](http://msdn.microsoft.com/library/azure/dd179371.aspx) 。  

下表彙總了在要求中接受條件式標頭 (例如 **If-Match** ) 以及在回應中傳回 ETag 值的容器作業。  

| 作業 | 傳回容器 ETag 值 | 接受條件式標頭 |
|:--- |:--- |:--- |
| 建立容器 |是 |否 |
| 取得容器屬性 |是 |否 |
| 取得容器中繼資料 |是 |否 |
| 設定容器中繼資料 |是 |是 |
| 取得容器 ACL |是 |否 |
| 設定容器 ACL |是 |是 (*) |
| 刪除容器 |否 |是 |
| 租用容器 |是 |是 |
| 列出 Blob |否 |否 |

(*) 系統會快取由 SetContainerACL 定義的權限，而這些權限的更新約需 30 秒來完成填入，在此期間，更新並不一定會一致。  

下表彙總了在要求中接受條件式標頭 (例如 **If-Match** ) 以及在回應中傳回 ETag 值的 Blob 作業。

| 作業 | 傳回 ETag 值 | 接受條件式標頭 |
|:--- |:--- |:--- |
| 放置 Blob |是 |是 |
| 取得 Blob |是 |是 |
| 取得 Blob 屬性 |是 |是 |
| 設定 Blob 屬性 |是 |是 |
| 取得 Blob 中繼資料 |是 |是 |
| 設定 Blob 中繼資料 |是 |是 |
| 租用 Blob (*) |是 |是 |
| 快照 Blob |是 |是 |
| 複製 Blob |是 |是 (針對來源及目的地 Blob) |
| 中止複製 Blob |否 |否 |
| 刪除 Blob |否 |是 |
| 放置區塊 |否 |否 |
| 放置區塊清單 |是 |是 |
| 取得區塊清單 |是 |否 |
| 放置頁面 |是 |是 |
| 取得頁面範圍 |是 |是 |

(*) 租用 Blob 並不會變更 Blob 上的 ETag。  

### <a name="pessimistic-concurrency-for-blobs"></a>Blob 的封閉式並行存取
若要鎖定 Blob 以進行獨佔使用，您可以取得其 [租用](http://msdn.microsoft.com/library/azure/ee691972.aspx) 。 您取得租用時，可指定需要租用的時間長度：這可以是 15 至 60 秒，也可以是無限期 (等於獨佔鎖定)。 您可以更新有限租用而加以延伸，而且您可以在用完任何租用後加以釋放。 Blob 服務會在有限租用到期時自動加以釋放。  

租用可讓不同的同步處理策略獲得支援，包括獨佔寫入/共用讀取、獨佔寫入/獨佔讀取和共用寫入/獨佔讀取。 只要有租用存在，儲存體服務就會強制執行獨佔寫入 (放置、設定和刪除作業)，但要確保讀取作業的獨佔性，開發人員必須確定所有的用戶端應用程式都使用同一個租用識別碼，並確定同一時間只有一個用戶端具有有效的租用識別碼。 未包含租用識別碼的讀取作業將會導致共用讀取。  

下列 C# 程式碼片段說明對某個 Blob 取得獨佔租用 30 秒、更新該 Blob 的內容，然後釋放租用的範例。 當您嘗試取得新租用時，若 Blob 上已存在有效租用，則 Blob 服務會傳回「HTTP (409) 衝突」狀態結果。 下列程式碼片段在要求更新儲存體服務中的 Blob 時，使用 **AccessCondition** 物件封裝租用資訊。  此處可下載完整的範例： [使用 Azure 儲存體管理並行存取](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)。

```csharp
// Acquire lease for 15 seconds
string lease = blockBlob.AcquireLease(TimeSpan.FromSeconds(15), null);
Console.WriteLine("Blob lease acquired. Lease = {0}", lease);

// Update blob using lease. This operation will succeed
const string helloText = "Blob updated";
var accessCondition = AccessCondition.GenerateLeaseCondition(lease);
blockBlob.UploadText(helloText, accessCondition: accessCondition);
Console.WriteLine("Blob updated using an exclusive lease");

//Simulate third party update to blob without lease
try
{
    // Below operation will fail as no valid lease provided
    Console.WriteLine("Trying to update blob without valid lease");
    blockBlob.UploadText("Update without lease, will fail");
}
catch (StorageException ex)
{
    if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        Console.WriteLine("Precondition failure as expected. Blob's lease does not match");
    else
        throw;
}  
```

如果您直接嘗試對已租用的 Blob 執行寫入作業，而未傳送租用識別碼，要求將會失敗，並出現 412 錯誤。 請注意，如果在呼叫 **UploadText** 方法之前租用已過期，但您仍傳送租用識別碼，要求也會失敗，並出現 **412** 錯誤。 如需管理租用到期時間和租用識別碼的詳細資訊，請參閱 [租用 Blob](http://msdn.microsoft.com/library/azure/ee691972.aspx) REST 文件。  

下列 Blob 作業可使用租用來管理封閉式並行存取：  

* 放置 Blob
* 取得 Blob
* 取得 Blob 屬性
* 設定 Blob 屬性
* 取得 Blob 中繼資料
* 設定 Blob 中繼資料
* 刪除 Blob
* 放置區塊
* 放置區塊清單
* 取得區塊清單
* 放置頁面
* 取得頁面範圍
* 快照 Blob - 若有租用存在，租用識別碼則是選用的
* 複製 Blob - 若有租用存在於目的地 Blob 上，則必須要有租用識別碼
* 中止複製 Blob - 若有無限制租用存在於目的地 Blob 上，則必須要有租用識別碼
* 租用 Blob  

### <a name="pessimistic-concurrency-for-containers"></a>容器的封閉式並行存取
容器的租用可讓與 Blob 相同的同步處理策略獲得支援 (獨佔寫入/共用讀取、獨佔寫入/獨佔讀取和共用寫入/獨佔讀取)，但不同於 Blob，儲存體服務只會對刪除作業強制執行獨佔性。 若要刪除具有作用中租用的容器，用戶端必須使用刪除要求納入作用中的租用識別碼。 其他所有的容器作業皆可在已租用的容器上成功執行，而無須納入租用識別碼；在此情況下，這些作業屬於共用作業。 如果更新 (放置或設定) 或讀取作業需要獨佔性，則開發人員應確定所有的用戶端都使用同一個租用識別碼，並確定同一時間只有一個用戶端具有有效的租用識別碼。  

下列容器作業可使用租用來管理封閉式並行存取：  

* 刪除容器
* 取得容器屬性
* 取得容器中繼資料
* 設定容器中繼資料
* 取得容器 ACL
* 設定容器 ACL
* 租用容器  

如需詳細資訊，請參閱  

* [指定 Blob 服務作業的條件式標頭](http://msdn.microsoft.com/library/azure/dd179371.aspx)
* [租用容器](http://msdn.microsoft.com/library/azure/jj159103.aspx)
* [租用 Blob ](http://msdn.microsoft.com/library/azure/ee691972.aspx)

## <a name="managing-concurrency-in-the-table-service"></a>管理資料表服務中的並行存取
當您使用實體時，資料表服務會使用開放式並行存取檢查作為預設行為，這一點不同於必須明確選擇執行開放式並行存取檢查的 Blob 服務。 資料表服務與 Blob 服務的另一個差異是您只能管理實體的並行存取行為，而在使用 Blob 服務時，您可以同時管理容器和 Blob 的並行存取。  

若要使用開放式並行存取，並檢查在您從資料表儲存體服務中擷取實體後是否有其他程序修改了該實體，您可以使用您在資料表服務傳回實體時所接收的 ETag 值。 此程序大致如下：  

1. 從資料表儲存體服務中擷取實體，回應中會包含 ETag 值，用來識別與儲存體服務中的該實體相關聯的現行識別碼。
2. 當您更新實體時，請將您在步驟 1 中接收到的 ETag 值納入您傳送至服務之要求的必要 **If-Match** 標頭中。
3. 服務會比較要求中的 ETag 值與實體目前的 ETag 值。
4. 如果實體目前的 ETag 值與要求中的必要 **If-Match** 標頭內的 ETag 不同，服務會將 412 錯誤傳回至用戶端。 這會向用戶端指出在用戶端擷取實體後，有另一個程序更新過該實體。
5. 如果實體目前的 ETag 值與要求中的必要 **If-Match** 標頭內的 ETag 相同，或是 **If-Match** 標頭包含萬用字元 (*)，服務將會執行要求的作業，並更新實體目前的 ETag 值，以顯示它已更新。  

請注意，不同於 Blob 服務，在使用資料表服務時，用戶端必須在更新要求中納入 **If-Match** 標頭。 但如果用戶端將要求中的 **If-Match** 標頭設為萬用字元 (*)，則有可能強制執行非條件式更新 (「最後寫入為準」策略)，並略過並行存取檢查。  

下列 C# 程式碼片段說明先前建立或擷取的客戶實體更新了其電子郵件地址。 初始插入或擷取作業將 ETag 值儲存在客戶物件中，且因為範例在執行取代作業時使用了相同的物件執行個體，因此自動將 ETag 值傳回至資料表服務，使服務能夠檢查是否有並行存取違規。 如果有其他程序更新了資料表儲存體中的實體，服務將會傳回 HTTP 412 (預先指定的條件失敗) 狀態訊息。  此處可下載完整的範例： [使用 Azure 儲存體管理並行存取](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)。

```csharp
try
{
    customer.Email = "updatedEmail@contoso.org";
    TableOperation replaceCustomer = TableOperation.Replace(customer);
    customerTable.Execute(replaceCustomer);
    Console.WriteLine("Replace operation succeeded.");
}
catch (StorageException ex)
{
    if (ex.RequestInformation.HttpStatusCode == 412)
        Console.WriteLine("Optimistic concurrency violation – entity has changed since it was retrieved.");
    else
        throw;
}  
```

若要明確停用並行存取檢查，您應在執行取代作業之前，將 **employee** 物件的 **ETag** 屬性設為 "*"。  

```csharp
customer.ETag = "*";  
```

下表彙總了資料表實體作業使用 ETag 值的情形：

| 作業 | 傳回 ETag 值 | 需要 If-Match 要求標頭 |
|:--- |:--- |:--- |
| 查詢實體 |是 |否 |
| 插入實體 |是 |否 |
| 更新實體 |是 |是 |
| 合併實體 |是 |是 |
| 刪除實體 |否 |是 |
| 插入或取代實體 |是 |否 |
| 插入或合併實體 |是 |否 |

請注意，**插入或取代實體**和**插入或合併實體**作業並「不會」執行任何並行存取檢查，因為這些作業不會將 ETag 值傳送至資料表服務。  

一般而言，使用資料表的開發人員在開發可擴充的應用程式時，應該會採用開放式並行存取。 如果需要封閉式鎖定，開發人員在存取資料表時可採用的方法之一，是為每個資料表指派一個指定 Blob，在且在操作資料表之前嘗試租用 Blob。 要使用此方法，應用程式必須確定所有資料存取路徑都在操作資料表之前取得租用。 您也應注意，最短租用時間為 15 秒，您應謹慎考量這一點以維持擴充性。  

如需詳細資訊，請參閱  

* [實體上的作業](http://msdn.microsoft.com/library/azure/dd179375.aspx)  

## <a name="managing-concurrency-in-the-queue-service"></a>管理佇列服務中的並行存取
在使用佇列服務時必須考量並行存取的案例之一，是有多個用戶端從一個佇列擷取訊息時。 從佇列擷取訊息時，回應中會包含訊息，以及刪除訊息所需的 pop receipt 值。 訊息並不會從佇列中自動刪除，但在擷取之後，其他用戶端在 visibilitytimeout 參數所指定的時間間隔內將看不到此訊息。 擷取訊息的用戶端應會在訊息完成處理後、回應的 TimeNextVisible 元素所指定的時間之前刪除訊息；這段時間會根據 visibilitytimeout 參數值計算得出。 visibilitytimeout 的值會加到擷取訊息的時間上，以決定 TimeNextVisible 的值。  

佇列服務並不支援開放式或封閉式並行存取，因此，用戶端在處理擷取自佇列的訊息時，應確定訊息是以冪等方式進行處理的。 「最後寫入為準」策略可用於更新作業，例如 SetQueueServiceProperties、SetQueueMetaData、SetQueueACL 和 UpdateMessage。  

如需詳細資訊，請參閱  

* [佇列服務 REST API](http://msdn.microsoft.com/library/azure/dd179363.aspx)
* [取得訊息](http://msdn.microsoft.com/library/azure/dd179474.aspx)  

## <a name="managing-concurrency-in-the-file-service"></a>管理檔案服務中的並行存取
檔案服務可使用兩種不同的通訊協定端點來存取 – SMB 和 REST。 REST 服務不支援開放式鎖定或封閉式鎖定，且所有更新都將遵循「最後寫入為準」策略。 裝載檔案共用的 SMB 用戶端可利用檔案系統鎖定機制，來管理對共用檔案的存取 – 包括執行封閉式鎖定的功能。 SMB 用戶端在開啟檔案時，會同時指定檔案存取和共用模式。 將 [檔案存取] 選項設為 [寫入] 或 [讀取/寫入]，並將 [檔案共用] 模式設為 [無]，會使檔案被 SMB 用戶端鎖定，直到檔案關閉為止。 如果嘗試對已由 SMB 用戶端鎖定的檔案執行 REST 作業，REST 服務將會傳回狀態碼 409 (衝突) 和錯誤碼 SharingViolation。  

SMB 用戶端在開啟檔案以進行刪除時，會將檔案標示為「擱置刪除」，直到所有對該檔案的其他 SMB 用戶端開啟控制代碼關閉為止。 在檔案標示為「擱置刪除」時，任何對該檔案的 REST 作業都將傳回狀態碼 409 (衝突) 和錯誤碼 SMBDeletePending。 此時並不會傳回狀態碼 404 (找不到)，因為 SMB 用戶端有可能在關閉檔案之前移除擱置刪除旗標。 換句話說，狀態碼 404 (找不到) 只有可能在檔案已移除時出現。 請注意，處於 SMB 擱置刪除狀態的檔案不會包含在「列出檔案」結果中。 另請注意，「REST 刪除檔案」和「REST 刪除目錄」作業會以不可分割的方式認可，因此不會導致擱置刪除狀態。  

如需詳細資訊，請參閱  

* [管理檔案鎖定](http://msdn.microsoft.com/library/azure/dn194265.aspx)  

## <a name="summary-and-next-steps"></a>摘要和後續步驟
Microsoft Azure 儲存體服務的設計已符合最複雜的線上應用程式的需求，而不會迫使開發人員犧牲或再三考量主要的設計假設，例如已被開發人員視為理所當然的並行存取和資料一致性。  

如需此部落格中參考的完整範例應用程式：  

* [使用 Azure 儲存體管理並行存取 - 範例應用程式](http://code.msdn.microsoft.com/Managing-Concurrency-using-56018114)  

如需 Azure 儲存體的詳細資訊，請參閱：  

* [Microsoft Azure 儲存體首頁](https://azure.microsoft.com/services/storage/)
* [Azure 儲存體簡介](storage-introduction.md)
* [Blob](../blobs/storage-dotnet-how-to-use-blobs.md)、[資料表](../../cosmos-db/table-storage-how-to-use-dotnet.md)、[佇列](../storage-dotnet-how-to-use-queues.md)及[檔案](../storage-dotnet-how-to-use-files.md)的儲存體入門
* 儲存體架構 – [Azure 儲存體：具有高度一致性的高可用性雲端儲存體服務](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

