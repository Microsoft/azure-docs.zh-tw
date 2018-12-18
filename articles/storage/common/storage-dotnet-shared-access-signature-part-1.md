---
title: 在 Azure 儲存體中使用共用存取簽章 (SAS) | Microsoft Docs
description: 學習如何使用共用存取簽章 (SAS) 委派存取至 Azure 儲存體資源，包括 Blob、佇列、資料表及檔案。
services: storage
author: craigshoemaker
ms.service: storage
ms.topic: article
ms.date: 04/18/2017
ms.author: cshoe
ms.component: common
ms.openlocfilehash: 315c5a88d16206414b6b81a83963cbb1f8b4424a
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524749"
---
# <a name="using-shared-access-signatures-sas"></a>使用共用存取簽章 (SAS)

若要在無須提供您帳戶金鑰的情況下，將儲存體帳戶中物件的限制存取授與其他用戶端，則可使用共用存取簽章 (SAS) 達成此目標。 本文會提供有關 SAS 模型的概觀、檢閱最佳做法以及查看一些範例。

除了本文所提供的範例外，如果您還需要其他使用 SAS 的程式碼範例，請參閱[在 .NET 中開始使用 Azure Blob 儲存體](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/)和 [Azure 程式碼範例](https://azure.microsoft.com/documentation/samples/?service=storage)程式庫提供的其他範例。 您可以下載範例應用程式並加以執行，或瀏覽 GitHub 上的程式碼。

## <a name="what-is-a-shared-access-signature"></a>共用存取簽章為何？
共用存取簽章可提供您儲存體帳戶中資源的委派存取。 透過 SAS，您可以對用戶端授與儲存體帳戶中資源的存取權，而不必共用帳戶金鑰。 這是在您應用程式中使用共用存取簽章的重點 - SAS 是共用儲存體資源的安全方式，而不會危害您的帳戶金鑰。

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

SAS 可讓您更細微地控制要對擁有 SAS 的用戶端授與何種類型的存取權，包括︰

* SAS 的有效期間，包括開始時間和到期時間。
* SAS 所授與的權限。 例如，Blob 的 SAS 可能會授與該 Blob 的讀取和寫入權限，但不授與刪除權限。
* Azure 儲存體接受的 SAS 所來自的選擇性 IP 位址或 IP 位址範圍。 例如，您可以指定屬於組織的 IP 位址範圍。
* Azure 儲存體接受的 SAS 所透過的通訊協定。 您可以使用這個選擇性參數來限制使用 HTTPS 之用戶端的存取權。

## <a name="when-should-you-use-a-shared-access-signature"></a>使用共用存取簽章的時機？
當您想要將儲存體帳戶中的資源存取權提供給未具有您儲存體帳戶存取金鑰的用戶端時，即可使用 SAS。 您的儲存體帳戶包含主要與次要存取金鑰，兩者皆可授與帳戶及帳戶內所有資源的系統管理存取權。 提供以上任一金鑰，皆有可能讓您的帳戶遭到惡意或粗心使用。 共用存取簽章提供了安全的替代方式，無需帳戶金鑰便可讓用戶端根據獲明確授與的權限，來讀取、寫入及刪除您儲存體帳戶中的資料。

證明 SAS 非常有用的一個常見案例，就是使用者在您的儲存體帳戶中讀取和寫入自己的資料。 在儲存體帳戶儲存使用者資料的案例中，典型的設計模式有兩種：

1. 用戶端通過前端 Proxy 服務 (執行驗證) 來上傳與下載資料。 此前端 Proxy 服務有個好處，那就是允許商務規則的驗證，但在大量資料或大量交易的情況下，建立可調整以符合需求的服務可能十分昂貴或困難。

  ![案例圖表︰前端 Proxy 服務](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png)   

1. 輕量型服務可視需要驗證用戶端，然後產生 SAS。 在用戶端收到 SAS 之後，他們可以使用 SAS 所定義的權限，並在 SAS 允許的間隔內直接存取儲存體帳戶資源。 SAS 可減輕透過前端 Proxy 服務路由所有資料的需求。

  ![案例圖表︰SAS 提供者服務](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png)   

許多實際服務可能會混合運用這兩種方法。 例如，某些資料可能會透過前端 Proxy 處理和驗證，其他資料則會直接使用 SAS 來儲存和/或讀取。

此外，在某些情況下，您必須使用 SAS 來授權存取複製作業中的來源物件：

* 當您將 Blob 複製到另一個位於不同儲存體帳戶的 Blob 時，必須使用 SAS 來授權存取來源 Blob。 您也可以選擇性地使用 SAS 來授權存取目的地 Blob。
* 當您將檔案複製到另一個位於不同儲存體帳戶的檔案時，必須使用 SAS 來授權存取來源檔案。 您也可以選擇性地使用 SAS 來授權存取目的檔案。
* 當您將 Blob 複製到檔案，或將檔案複製到 Blob 時，必須使用 SAS 來授權存取來源物件，即使來源和目的地物件位於相同的儲存體帳戶內也一樣。

## <a name="types-of-shared-access-signatures"></a>共用存取簽章的類型
您可建立兩種類型的共用存取簽章：

* **服務 SAS。** 服務 SAS 只會將存取權限委派給一種儲存體服務資源：Blob、佇列、資料表或檔案服務。 如需有關建構服務 SAS 權杖的深入資訊，請參閱[建構服務 SAS](https://msdn.microsoft.com/library/dn140255.aspx) 和[服務 SAS 範例](https://msdn.microsoft.com/library/dn140256.aspx)。
* **帳戶 SAS。** 帳戶 SAS 則將存取權限委派給一或多個儲存體服務的資源。 可透過服務 SAS 取得的所有作業也可透過帳戶 SAS 取得。 此外，利用帳戶 SAS，您可以委派適用指定的服務作業 (例如：**取得/設定服務屬性**和**取得服務統計資料**) 的存取。您也可以將 Blob 容器、資料表、佇列和檔案共用的讀取、寫入和刪除作業的存取權限，委派給本無權限的服務 SAS。 如需有關建構帳戶 SAS 權杖的深入資訊，請參閱[建構帳戶 SAS](https://msdn.microsoft.com/library/mt584140.aspx)。

## <a name="how-a-shared-access-signature-works"></a>共用存取簽章的運作方式
共用存取簽章是指向一或多個儲存體資源，並包括含有一組特殊的查詢參數權杖的已簽署 URI。 權杖指出用戶端可以如何存取資源。 簽章是查詢參數的其中一個，根據 SAS 參數所建構並使用帳戶金鑰進行簽署。 Azure 儲存體會使用此簽章來授權存取儲存體資源。

以下是 SAS URI 範例，其顯示資源 URI 和 SAS 權杖︰

![SAS URI 的元件](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-uri.png)   

SAS 權杖是*用戶端*所產生的字串 (如需程式碼範例，請參閱〈[SAS 範例](#sas-examples)〉一節)。 譬如 Azure 儲存體不會以任何方式，追蹤您透過儲存體用戶端程式庫所產生的 SAS 權杖。 您可以在用戶端建立不限數量的 SAS 權杖。

當用戶端在要求中提供 SAS URI 給 Azure 儲存體時，服務會檢查 SAS 參數和簽章，以確認它是有效的，可用於驗證要求。 如果服務確認簽章有效，則要求會獲得授權。 否則要求會遭到拒絕，並產生錯誤碼 403 (禁止)。

## <a name="shared-access-signature-parameters"></a>共用存取簽章參數
帳戶 SAS 和服務 SAS 權杖包含一些常見的參數，並且採取幾個不同參數。

### <a name="parameters-common-to-account-sas-and-service-sas-tokens"></a>帳戶 SAS 和服務 SAS 權杖的通用參數
* **API 版本** 選擇性參數，指定要用來執行要求的儲存體服務版本。
* **服務版本** 必要參數，指定要用來授權要求的儲存體服務版本。
* **開始時間。** 這是指 SAS 生效的時間。 共用存取簽章的開始時間為選擇性。 若省略開始時間，SAS 便會立即生效。 開始時間必須以 UTC (國際標準時間) 表示，並包含特殊的 UTC 指示項 ("Z")，例如 `1994-11-05T13:15:30Z`。
* **到期時間。** 這是指 SAS 何時失效的時間。 最佳做法建議您為 SAS 指定過期時間，或將它與預存存取原則建立關聯。 過期時間必須以 UTC (國際標準時間) 表示，並包含特殊的 UTC 指示項 ("Z")，例如 `1994-11-05T13:15:30Z` (參閱以下的更多資訊)。
* **權限。** 在 SAS 上指定的權限表示用戶端可以使用 SAS 來對儲存體資源執行哪些作業。 帳戶 SAS 和服務 SAS 的可用權限不同。
* **IP。** 選用參數，可指定要從中接受要求且位於 Azure 外部的 IP 位址或 IP 位址範圍 (請參閱適用於 Express Route 的 [路由工作階段組態狀態](../../expressroute/expressroute-workflows.md#routing-session-configuration-state) 一節)。
* **通訊協定。** 選擇性參數，指定對要求允許的通訊協定。 可能的值為 HTTPS 和 HTTP (`https,http`)，其為預設值或僅限 HTTPS (`https`)。 請注意，僅 HTTP 是不允許的值。
* **簽章。** 簽章是從其他參數建構，指定為權杖的一部分，然後加密。 此簽章會用來授權存取指定的儲存體資源。

### <a name="parameters-for-a-service-sas-token"></a>服務 SAS 權杖的參數
* **儲存體資源。** 可以委派對服務 SAS 存取的儲存體資源包括：
  * 容器和 Blob
  * 檔案共用及檔案
  * 佇列
  * 資料表和資料表實體範圍。

### <a name="parameters-for-an-account-sas-token"></a>帳戶 SAS 權杖的參數
* **一或多個服務。** 帳戶 SAS 可以委派存取給一或多個儲存體服務。 例如，您可以建立委派存取 Blob 和檔案服務的帳戶 SAS。 或者您可以建立委派存取給全部四個服務 (Blob、佇列、表格和檔案) 的 SAS。
* **儲存體資源類型。** 帳戶 SAS 適用於一或多個類別的儲存體資源，而不是特定資源。 您可以建立帳戶 SAS 來委派存取給：
  * 對儲存體帳戶資源呼叫的服務層級 API。 範例包括**取得/設定服務屬性**、**取得服務統計資料**和**列出容器/佇列/資料表/共用**。
  * 容器層級 API，會針對每個服務的容器物件呼叫：Blob 容器、佇列、資料表和檔案共用。 範例包括**建立/刪除容器**、**建立/刪除佇列**、**建立/刪除資料表**、**建立/刪除共用**和**列出 Blob/檔案和目錄**。
  * 物件層級 API，針對 Blob、佇列訊息、資料表實體和檔案呼叫。 例如，**放置 Blob**、**查詢實體**、**取得訊息**和**建立檔案**。

## <a name="examples-of-sas-uris"></a>SAS URI 的範例

### <a name="service-sas-uri-example"></a>服務 SAS URI 範例

以下是提供讀取和寫入 Blob 權限的服務 SAS URI 範例。 此資料表會細分 URI 的每一部分，以了解它會如何影響 SAS：

```
https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
```

| Name | SAS 部分 | 說明 |
| --- | --- | --- |
| Blob URI |`https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt` |Blob 的位址。 請注意，我們強烈建議您使用 HTTPS。 |
| 儲存體服務版本 |`sv=2015-04-05` |若是儲存體服務版本 2012-02-12 和更新版本，此參數表示要使用的版本。 |
| 開始時間 |`st=2015-04-29T22%3A18%3A26Z` |以 UTC 時間指定。 如果您想要 SAS 立即生效，請略過開始時間。 |
| 過期時間 |`se=2015-04-30T02%3A23%3A26Z` |以 UTC 時間指定。 |
| 資源 |`sr=b` |此資源是 Blob。 |
| 權限 |`sp=rw` |SAS 所授與的權限包括讀取 (r) 和寫入 (w)。 |
| IP 範圍 |`sip=168.1.5.60-168.1.5.70` |將從中接受要求的 IP 位址範圍。 |
| 通訊協定 |`spr=https` |僅允許使用 HTTPS 的要求。 |
| 簽章 |`sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D` |用來授權存取 Blob。 此簽章是 HMAC 根據要簽署字串和金鑰，使用 SHA256 演算法進行計算，然後使用 Base64 方式進行編碼而來的。 |

### <a name="account-sas-uri-example"></a>帳戶 SAS URI 範例

以下是對權杖使用相同通用參數的帳戶 SAS 範例。 由於上面說明了這些參數，在此處將不說明。 下表只會說明帳戶 SAS 的特定參數。

```
https://myaccount.blob.core.windows.net/?restype=service&comp=properties&sv=2015-04-05&ss=bf&srt=s&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=F%6GRVAZ5Cdj2Pw4tgU7IlSTkWgn7bUkkAg8P6HESXwmf%4B
```

| Name | SAS 部分 | 說明 |
| --- | --- | --- |
| 資源 URI |`https://myaccount.blob.core.windows.net/?restype=service&comp=properties` |Blob 服務端點，具有用來取得服務屬性 (使用 GET 呼叫時) 或設定服務屬性 (使用 SET 呼叫時) 的參數。 |
| 服務 |`ss=bf` |SAS 適用於 Blob 和檔案服務 |
| 資源類型 |`srt=s` |SAS 適用於服務層級的作業。 |
| 權限 |`sp=rw` |此權限可授與讀取和寫入作業的存取權。 |

假設權限僅限於服務層級，此 SAS 可存取的作業是**取得 Blob 服務屬性** (讀取) 和**設定 Blob 服務屬性** (寫入)。 不過，利用不同的資源 URI，相同的 SAS 權杖也可以用來委派存取給 **取得 Blob 服務統計資料** (讀取)。

## <a name="controlling-a-sas-with-a-stored-access-policy"></a>使用預存存取原則控制 SAS
共用存取簽章可以接受以下兩種格式其中之一：

* **臨機操作 SAS：** 建立臨機操作 SAS 時，SAS 的開始時間、過期時間和權限都會在 SAS URI 上進行指定 (或暗示，在此情況下則會略過開始時間)。 這種類型的 SAS 可能會建立為帳戶 SAS 或服務 SAS。
* **具有預存存取原則的 SAS：** 預存存取原則會在資源容器 (Blob 容器、資料表、佇列或檔案共用) 中定義，且可用來管理一或多個共用存取簽章的限制。 當您將 SAS 與預存存取原則建立關聯時，SAS 會繼承為該預存存取原則所定義的限制 (開始時間、過期時間和權限)。

> [!NOTE]
> 目前，帳戶 SAS 必須是臨機操作 SAS。 帳戶 SAS 尚不支援預存的存取原則。

這兩種格式間的差異對於以下這一個重要案例而言相當重要：撤銷。 SAS URI 為 URL，因此無論其原始建立者為何，任何人只要取得 SAS 即可自由使用。 如果是公開發佈 SAS，則全世界的人都可以使用此 SAS。 除非發生下述四種情況之一，否則 SAS 會將資源存取權授與具有 SAS 的任何人：

1. 已到達 SAS 上指定的過期時間。
2. 已到達在 SAS 所參考之預存存取原則上所指定的過期時間 (如果參考的是預存存取原則，而且如果此預存存取原則指定了過期時間)。 發生此情況的原因，有可能是因為已超過指定的間隔時間，或是因為您已修改預存存取原則，將過期時間設定為過去的日期，這是撤銷 SAS 的一種方法。
3. 已刪除 SAS 所參考之預存存取原則，這是撤銷 SAS 的另外一種方法。 請注意，如果您使用完全相同的名稱來重新建立預存存取原則，則現有的所有 SAS 權杖會根據與該預存存取原則有關的權限再次有效 (假設 SAS 上的過期時間尚未過去)。 如果您打算撤銷 SAS，且如果您要使用未來的過期時間來重新建立存取原則，則務必使用不同的名稱。
4. 系統會重新產生用來建立 SAS 的帳戶金鑰。 重新產生帳戶金鑰將導致所有使用該金鑰的應用程式元件無法授權，直到它們已更新為使用其他有效帳戶金鑰或最近重新產生的帳戶金鑰為止。

> [!IMPORTANT]
> 共用存取簽章 URI 會與用來建立簽章的帳戶金鑰，以及相關聯的預存的存取原則 (如果有的話) 產生關聯。 如果未指定任何預存的存取原則，則撤銷共用存取簽章的唯一方式是變更帳戶金鑰。

## <a name="authenticating-from-a-client-application-with-a-sas"></a>使用 SAS 從用戶端應用程式進行驗證
擁有 SAS 的用戶端可以使用 SAS 來對他們未擁有帳戶金鑰的儲存體帳戶授權要求。 SAS 可以包含在連接字串，或直接從適當的建構函式或方法來使用。

### <a name="using-a-sas-in-a-connection-string"></a>在連接字串中使用 SAS
[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

### <a name="using-a-sas-in-a-constructor-or-method"></a>在建構函式或方法中使用 SAS
數個 Azure 儲存體用戶端程式庫的建構函式和方法多載會提供 SAS 參數，以便您使用 SAS 來授權對服務的要求。

例如，這裡便使用 SAS URI 來建立區塊 Blob 的參考。 SAS 提供要求所需的唯一認證。 區塊 Blob 參考接著會用來進行寫入作業︰

```csharp
string sasUri = "https://storagesample.blob.core.windows.net/sample-container/" +
    "sampleBlob.txt?sv=2015-07-08&sr=b&sig=39Up9JzHkxhUIhFEjEH9594DJxe7w6cIRCg0V6lCGSo%3D" +
    "&se=2016-10-18T21%3A51%3A37Z&sp=rcw";

CloudBlockBlob blob = new CloudBlockBlob(new Uri(sasUri));

// Create operation: Upload a blob with the specified name to the container.
// If the blob does not exist, it will be created. If it does exist, it will be overwritten.
try
{
    MemoryStream msWrite = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    msWrite.Position = 0;
    using (msWrite)
    {
        await blob.UploadFromStreamAsync(msWrite);
    }

    Console.WriteLine("Create operation succeeded for SAS {0}", sasUri);
    Console.WriteLine();
}
catch (StorageException e)
{
    if (e.RequestInformation.HttpStatusCode == 403)
    {
        Console.WriteLine("Create operation failed for SAS {0}", sasUri);
        Console.WriteLine("Additional error information: " + e.Message);
        Console.WriteLine();
    }
    else
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}

```

## <a name="best-practices-when-using-sas"></a>使用 SAS 時的最佳做法
當您在應用程式中使用共用存取簽章時，您必須留意兩個潛在風險：

* 如果 SAS 洩漏出去，則取得該 SAS 的任何人都可以使用它，這有可能會洩露您的儲存體帳戶。
* 如果提供給用戶端應用程式的 SAS 已過期，且此應用程式無法從您的服務擷取新的 SAS，那麼該應用程式的功能可能會受到影響。

下列關於使用共用存取簽章的建議，將可協助您平衡這些風險：

1. **永遠使用 HTTPS** 來建立或散佈 SAS。 若透過 HTTP 來傳遞 SAS 並遭到攔截，執行攔截式攻擊的攻擊者即可讀取並使用 SAS (就如同預期使用者執行般)，這有可能會洩露敏感資料或允許惡意使用者損毀資料。
2. **可能的話，參考預存存取原則。** 預存存取原則提供了撤銷權限且無需重新產生儲存體帳戶金鑰的選項。 將到期日設在未來 (或無限) 的日期，並確定定期更新到期日以將到期日再往未來的日期移動。
3. **在臨機操作 SAS 上使用短期的到期時間。** 如此一來，即使 SAS 遭到入侵，亦僅會造成短期影響。 如果您無法參考預存存取原則，此做法格外重要。 短期到期時間亦可協助限制可寫入 Blob 的資料量，方法是限制可對其上傳的可用時間。
4. **讓用戶端視需要自動更新 SAS。** 用戶端應在到期日之前就更新 SAS，以便如果提供 SAS 的服務無法使用的話，還有時間可以進行重試。 如果您打算將 SAS 用於少量的即時短期操作 (預計可在到期期限內完成的操作)，則此建議可能沒有必要，因為沒有更新 SAS 的打算。 不過，如果您有定期透過 SAS 做出要求的用戶端，則到期的可能性便有可能發生。 主要考量是要平衡下列兩個需求：短期的 SAS (如先前所述)，與確保用戶端提早要求更新以避免成功更新之前因 SAS 過期而中斷。
5. **請小心使用 SAS 開始時間。** 如果您將 SAS 的開始時間設為 [現在]，則由於時鐘誤差 (根據不同機器會有不同的目前時間)，前幾分鐘可能偶爾會被視為失敗。 一般而言，請將開始時間設為至少 15 分鐘之前的時間。 或是不進行任何設定，這會針對所有案例立即生效。 同樣的道理通常亦適用於過期時間，請記住，您可針對任何要求保留前後多達 15 分鐘的時鐘誤差。 若是用戶端使用 2012-02-12 之前的 REST 版本，則不參考預存存取原則之 SAS 的最大持續期限是 1 個小時，且任何指定比 1 個小時還要長的原則都會失敗。
6. **請具體指出要存取的資源。** 安全性最佳做法是提供使用者最低需求權限。 如果使用者只需要單一實體的讀取存取權，則授與他們該單一實體的讀取存取權，而非授與他們所有實體的讀取/寫入/刪除存取權。 這有助於減輕洩露 SAS 遭受的損害，因為當 SAS 落入攻擊者手中時，即無法發揮固有功能。
7. **了解您帳戶的任何方式將會被收取費用，包括以 SAS 方式完成的部分。** 如果您提供 Blob 的寫入存取權，則使用者可能會選擇上傳 200GB 的 Blob。 若您也同時提供使用者讀取存取權，則他們可能會選擇下載 10 次，而您便會產生 2TB 的出口成本。 再次強調，提供有限的權限有助於減少惡意使用者採取的潛在動作。 使用短期 SAS 以降低此威脅 (但請注意結束時間的時鐘誤差)。
8. **使用 SAS 驗證寫入資料。** 當用戶端應用程式將資料寫入您的儲存體帳戶時，請留意該資料可能會造成問題。 如果您的應用程式要求在開始使用資料之前先驗證或授權資料，則您應在寫入資料之後但應用程式尚未開始使用資料之前執行此驗證。 此做法也可防止正確取得 SAS 的使用者或是利用洩漏 SAS 的使用者，損毀資料或將惡意資料寫入您的帳戶。
9. **請勿一直使用 SAS。** 有時候，在儲存體帳戶中執行特定作業的相關風險可能大過 SAS 的好處。 針對此類作業，請建立一個中介層服務，在執行商務規則驗證、驗證及稽核之後才寫入您的儲存體帳戶。 另外，有時候以其他方式管理存取權可能比較簡單。 例如，如果您要讓容器中的所有 Blob 都可公開讀取，則您可以將此容器設定為 [公用]，而不是將 SAS 提供給每個用戶端進行存取。
10. **使用儲存體分析來監視您的應用程式。** 您可以使用記錄和度量來觀察由於 SAS 提供者服務中斷或不小心移除預存存取原則，而造成的任何驗證失敗急劇增加。 如需額外資訊，請參閱 [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx) (英文)。

## <a name="sas-examples"></a>SAS 範例
下面是兩種類型共用存取簽章 (帳戶 SAS 和服務 SAS) 的一些範例。

若要執行這些 C# 範例，您必須參考專案中的下列 NuGet 封裝︰

* [Azure Storage Client Library for .NET](http://www.nuget.org/packages/WindowsAzure.Storage)，版本 6.x 或更高版本 (以使用 帳戶 SAS)。
* [Azure 資源管理員](http://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager)

如需其他範例，示範如何建立及測試 SAS，請參閱 [Azure 儲存體的程式碼範例](https://azure.microsoft.com/documentation/samples/?service=storage)。

### <a name="example-create-and-use-an-account-sas"></a>範例︰建立和使用帳戶 SAS
下列程式碼範例會建立適用於 Blob 和檔案服務的帳戶 SAS，並提供用戶端權限讀取、寫入和列出權限來存取服務層級 API。 帳戶 SAS 會將通訊協定限制為 HTTPS，因此必須使用 HTTPS 提出要求。

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use your shared key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

若要使用 帳戶 SAS 來存取 Blob 服務的服務層級 API，請使用 SAS 及儲存體帳戶的 Blob 儲存體端點來建構 Blob 用戶端物件。

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "account-name", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

### <a name="example-create-a-stored-access-policy"></a>範例：建立預存的存取原則
下列程式碼會在容器上建立預存的存取原則。 您可以使用存取原則，對於容器上的服務 SAS 或其 Blob 指定條件約束。

```csharp
private static async Task CreateSharedAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new shared access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
        // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create | SharedAccessBlobPermissions.Delete
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

### <a name="example-create-a-service-sas-on-a-container"></a>範例︰在容器上建立服務 SAS
下列程式碼會在容器上建立 SAS。 如果提供現有預存存取原則的名稱，該原則將與 SAS 相關聯。 如果未提供任何預存存取原則，則程式碼會在容器上建立臨機操作 SAS。

```csharp
private static string GetContainerSasUri(CloudBlobContainer container, string storedPolicyName = null)
{
    string sasContainerToken;

    // If no stored policy is specified, create a new access policy and define its constraints.
    if (storedPolicyName == null)
    {
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad-hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocPolicy = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List
        };

        // Generate the shared access signature on the container, setting the constraints directly on the signature.
        sasContainerToken = container.GetSharedAccessSignature(adHocPolicy, null);

        Console.WriteLine("SAS for blob container (ad hoc): {0}", sasContainerToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the container. In this case, all of the constraints for the
        // shared access signature are specified on the stored access policy, which is provided by name.
        // It is also possible to specify some constraints on an ad-hoc SAS and others on the stored access policy.
        sasContainerToken = container.GetSharedAccessSignature(null, storedPolicyName);

        Console.WriteLine("SAS for blob container (stored access policy): {0}", sasContainerToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return container.Uri + sasContainerToken;
}
```

### <a name="example-create-a-service-sas-on-a-blob"></a>範例︰在 Blob 上建立服務 SAS
下列程式碼會在 Blob 上建立 SAS。 如果提供現有預存存取原則的名稱，該原則將與 SAS 相關聯。 如果未提供任何預存存取原則，則程式碼會在 Blob 上建立臨機操作 SAS。

```csharp
private static string GetBlobSasUri(CloudBlobContainer container, string blobName, string policyName = null)
{
    string sasBlobToken;

    // Get a reference to a blob within the container.
    // Note that the blob may not exist yet, but a SAS can still be created for it.
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    if (policyName == null)
    {
        // Create a new access policy and define its constraints.
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad-hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocSAS = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create
        };

        // Generate the shared access signature on the blob, setting the constraints directly on the signature.
        sasBlobToken = blob.GetSharedAccessSignature(adHocSAS);

        Console.WriteLine("SAS for blob (ad hoc): {0}", sasBlobToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the blob. In this case, all of the constraints for the
        // shared access signature are specified on the container's stored access policy.
        sasBlobToken = blob.GetSharedAccessSignature(null, policyName);

        Console.WriteLine("SAS for blob (stored access policy): {0}", sasBlobToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return blob.Uri + sasBlobToken;
}
```

## <a name="conclusion"></a>結論
若要提供您儲存體帳戶的有限權限給沒有帳戶金鑰的用戶端，則共用存取簽章是非常有用的方式。 因此，對於任何使用 Azure 儲存體的應用程式而言，共用存取簽章是安全性模型不可或缺的一部分。 如果您依照此處所列的最佳做法進行，則您可以使用 SAS 來提供更大的彈性以存取儲存體帳戶中的資源，且不會影響應用程式的安全性。

## <a name="next-steps"></a>後續步驟
* [共用存取簽章，第 2 部分：透過 Blob 儲存體來建立與使用 SAS](../blobs/storage-dotnet-shared-access-signature-part-2.md)
* [管理對容器與 Blob 的匿名讀取權限。](../blobs/storage-manage-access-to-resources.md)
* [使用共用存取簽章來委派存取權](http://msdn.microsoft.com/library/azure/ee395415.aspx)
* [資料表與佇列 SAS 簡介](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
