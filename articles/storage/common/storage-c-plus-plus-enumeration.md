---
title: 使用 Storage Client Library for C++ 列出 Azure 儲存體資源 | Microsoft Docs
description: 了解如何使用 Microsoft Azure Storage Client Library for C++ 中的列表 API 來列舉容器、Blob、佇列、資料表和實體。
services: storage
author: dineshmurthy
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: dineshm
ms.component: common
ms.openlocfilehash: 0a105122eccdfa671c806389908762941f23200c
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525302"
---
# <a name="list-azure-storage-resources-in-c"></a>以 C++ 列出 Azure 儲存體資源
列表作業是許多使用 Azure 儲存體的開發案例的關鍵。 本文說明如何使用 Microsoft Azure Storage Client Library for C++ 中提供的列表 API，以最有效率的方式列舉 Azure 儲存體中的物件。

> [!NOTE]
> 本指南以 Azure Storage Client Library for C++ 2.x 版為對象 (其可透過 [NuGet](http://www.nuget.org/packages/wastorage) 或 [GitHub](https://github.com/Azure/azure-storage-cpp) 取得)。
> 
> 

Storage Client Library 提供各種方法來列出或查詢 Azure 儲存體中的物件。 本文說明下列案例：

* 列出帳戶中的容器
* 列出容器或虛擬 Blob 目錄中的 Blob
* 列出帳戶中的佇列
* 列出帳戶中的資料表
* 查詢資料表中的實體

每個方法會使用不同案例的不同多載來顯示。

## <a name="asynchronous-versus-synchronous"></a>同步與非同步
因為 Storage Client Library for C++ 的建置基礎為 [C++ REST 程式庫](https://github.com/Microsoft/cpprestsdk)，所以我們原本就使用 [pplx::task](http://microsoft.github.io/cpprestsdk/classpplx_1_1task.html) 支援非同步作業。 例如︰

```cpp
pplx::task<list_blob_item_segment> list_blobs_segmented_async(continuation_token& token) const;
```

同步作業會包裝對應的非同步作業：

```cpp
list_blob_item_segment list_blobs_segmented(const continuation_token& token) const
{
    return list_blobs_segmented_async(token).get();
}
```

如果您正在使用多個執行緒的應用程式或服務，建議您直接使用非同步 API，而不是建立執行緒來呼叫同步 API，這會大幅影響您的效能。

## <a name="segmented-listing"></a>分段列表
雲端儲存體的級別需要分段列表。 例如，在 Azure blob 容器中可有超過 100 萬個 Blob，或在 Azure 資料表中可有超過 10 億個實體。 這些不是理論上的數字，而是實際的客戶使用量案例。

因此，列出單一回應中的所有物件並不切實際。 反而，您可以使用分頁列出物件。 每個列表 API 都有 *分段* 多載。

分段列表作業的回應包含：

* <i>_segment</i>，其中包含針對列表 API 的單一呼叫所傳回的結果集。
* continuation_token，其會傳遞給下一個呼叫，以便取得下一頁的結果。 沒有可傳回的結果時，接續 Token 為 null。

例如，列出容器中所有 Blob 的典型呼叫可能如下列程式碼片段所示。 此程式碼可在我們的 [範例](https://github.com/Azure/azure-storage-cpp/blob/master/Microsoft.WindowsAzure.Storage/samples/BlobsGettingStarted/Application.cpp)中取得：

```cpp
// List blobs in the blob container
azure::storage::continuation_token token;
do
{
    azure::storage::list_blob_item_segment segment = container.list_blobs_segmented(token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_diretory(it->as_directory());
    }
}

    token = segment.continuation_token();
}
while (!token.empty());
```

請注意，一個頁面傳回的結果數目可由每個 API 的多載中的參數 max_results 所控制，例如：

```cpp
list_blob_item_segment list_blobs_segmented(const utility::string_t& prefix, bool use_flat_blob_listing,
    blob_listing_details::values includes, int max_results, const continuation_token& token,
    const blob_request_options& options, operation_context context)
```

如果您未指定 max_results 參數，則會在單一頁面中傳回多達 5000 筆結果的預設最大值。

也請注意，對 Azure 資料表儲存體的查詢可能不會傳回任何記錄，或傳回少於您指定之 max_results 參數值的記錄 (即使接續 Token 不是空的)。 其中一個原因可能是查詢無法在五秒內完成。 只要接續 Token 不是空的，查詢就應該繼續進行，而您的程式碼不得假設區段結果的大小。

大多數案例的建議編碼模式為分段列表，可以提供明確的列表或查詢進度，以及服務回應每個要求的方式。 尤其是 C++ 應用程式或服務，列表進度的較低層級控制項有助於控制記憶體和效能。

## <a name="greedy-listing"></a>窮盡列表
舊版的 Storage Client Library for C++ (0.5.0 預覽版或更早版本) 包含了適用於資料表和佇列的非分段列表 API，如下列範例所示：

```cpp
std::vector<cloud_table> list_tables(const utility::string_t& prefix) const;
std::vector<table_entity> execute_query(const table_query& query) const;
std::vector<cloud_queue> list_queues() const;
```

這些方法已做為分段 API 的包裝函式實作。 對於分段列表的每個回應，程式碼會將結果附加至向量，並傳回掃描完整容器後的所有結果。

當儲存體帳戶或資料表包含少量物件時，這個方法可能有用。 不過，隨著物件數目的增加，所需的記憶體可能會無所限制的增加，因為所有結果都會保留在記憶體中。 一項列表作業可能需要很長的時間，在這段期間內呼叫端沒有其進度的相關資訊。

SDK 中的這些窮盡列表 API 不存在於 C#、Java 或 JavaScript Node.js 環境中。 為了避免使用這些窮盡 API 的潛在問題，我們已在 0.6.0 預覽版中予以移除。

如果您的程式碼呼叫這些窮盡 API：

```cpp
std::vector<azure::storage::table_entity> entities = table.execute_query(query);
for (auto it = entities.cbegin(); it != entities.cend(); ++it)
{
    process_entity(*it);
}
```

您就應該修改程式碼以使用分段列表 API：

```cpp
azure::storage::continuation_token token;
do
{
    azure::storage::table_query_segment segment = table.execute_query_segmented(query, token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
    {
        process_entity(*it);
    }

    token = segment.continuation_token();
} while (!token.empty());
```

指定區段的 max_results 參數，即可平衡要求數目與記憶體使用量，以符合您的應用程式的效能考量。

此外，如果您使用分段列表 API，但以「窮盡」樣式將資料儲存在本機集合中，也強烈建議您重整您的程式碼，以便仔細地將資料大規模儲存在本機集合中。

## <a name="lazy-listing"></a>延遲列表
雖然窮盡列表引發了潛在的問題，但如果容器中沒有太多物件，則很方便。

如果您也使用 C# 或 Oracle Java SDK，您應該熟悉可提供延遲樣式列表的「可列舉」程式設計模型，其中特定位移的資料只會在必要時提取。 在 C++ 中，以迭代器為基礎的範本也會提供類似的方法。

典型的延遲列表 API (以 **list_blobs** 為例) 如下所示：

```cpp
list_blob_item_iterator list_blobs() const;
```

使用延遲列表模式的典型程式碼片段可能如下所示：

```cpp
// List blobs in the blob container
azure::storage::list_blob_item_iterator end_of_results;
for (auto it = container.list_blobs(); it != end_of_results; ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}
```

請注意，延遲列表僅可用於同步模式。

相較於窮盡列表，延遲列表只會在必要時提取資料。 實際上，只有在下一個迭代器移至下一個區段時，它才會從 Azure 儲存體提取資料。 因此，記憶體使用量會控制在有限的大小內，且作業速度很快。

延遲列表 API 已包含在 Storage Client Library for C++ 2.2.0 版中。

## <a name="conclusion"></a>結論
在本文中，我們針對 Storage Client Library for C++ 中的各種物件，討論了列表 API 的不同多載。 總結：

* 在多個執行緒的案例中，強烈建議使用非同步 API。
* 在大部分的案例中，建議使用分段列表。
* 程式庫中提供的延遲列表是同步案例中的方便包裝函式。
* 不建議使用窮盡列表，並已從程式庫中移除。

## <a name="next-steps"></a>後續步驟
如需 Azure Storage Client Library for C++ 的詳細資訊，請參閱下列資源。

* [如何使用 C++ 的 Blob 儲存體](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [如何使用 C++ 的資料表儲存體](../../cosmos-db/table-storage-how-to-use-c-plus.md)
* [如何使用 C++ 的佇列儲存體](../storage-c-plus-plus-how-to-use-queues.md)
* [Azure Storage Client Library for C++ API 文件。](http://azure.github.io/azure-storage-cpp/)
* [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage/)
* [Azure 儲存體文件](https://azure.microsoft.com/documentation/services/storage/)

