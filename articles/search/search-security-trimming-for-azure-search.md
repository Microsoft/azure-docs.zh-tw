---
title: 在 Azure 搜尋服務中進行安全性篩選以調整結果 | Microsoft Docs
description: 在 Azure 搜尋服務內容上使用安全性篩選和使用者身分識別進行存取控制。
ms.service: search
ms.topic: conceptual
services: search
ms.date: 08/07/2017
author: revitalbarletz
ms.author: revitalb
manager: jlembicz
ms.openlocfilehash: dd26676b74431566b3631b8a79cd06bcf3022518
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31792795"
---
# <a name="security-filters-for-trimming-results-in-azure-search"></a>在 Azure 搜尋服務中進行安全性篩選以調整結果

您可以在 Azure 搜尋服務中，依據使用者身分識別套用安全性篩選以調整搜尋結果。 在此種搜尋體驗中，通常需要將要求搜尋之人員的身分識別與含有具文件權限主體的欄位互相比較。 找到相符結果時，使用者或主體 (例如群組或角色) 即具有該文件的存取權。

達成安全性篩選的其中一種方式，是透過等號比較運算式的負責分離執行：例如 `Id eq 'id1' or Id eq 'id2'` 等等。 此作法很容易發生錯誤且難以維護，若清單包含數以百計或千計的值，則會使查詢回應時間慢上數秒鐘。 

透過 `search.in` 函式執行是更為簡單快速的作法。 若您使用 `search.in(Id, 'id1, id2, ...')` 取代等號比較運算式，則僅需不到一秒鐘的回應時間。

本文將示範如何使用下列步驟完成安全性篩選：
> [!div class="checklist"]
> * 建立包含主體識別碼的欄位 
> * 推送或更新具有相關主體識別碼的現有文件
> * 透過 `search.in` `filter` 發出搜尋要求

>[!NOTE]
> 本文件未多加說明擷取主體識別碼的流程。 您應從身分識別服務提供者處取得主體識別碼。

## <a name="prerequisites"></a>先決條件

本文假定您已具有 [Azure 訂用帳戶](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)、[Azure 搜尋服務](https://docs.microsoft.com/azure/search/search-create-service-portal)和 [Azure 搜尋索引](https://docs.microsoft.com/azure/search/search-create-index-portal)。  

## <a name="create-security-field"></a>建立安全性欄位

您的文件必須有一個欄位指定哪些群組具有存取權限。 此資訊會成為篩選準則，決定要從傳回給簽發者之結果集選取或拒絕哪些文件。
假設有個受保護檔案的索引，每個檔案皆可由一組不同的使用者存取。
1. 將欄位 `group_ids` (可於此選擇任何名稱) 新增為 `Collection(Edm.String)`。 確定欄位中的 `filterable` 屬性已設為 `true`，如此才能根據使用者具有的存取權限來篩選搜尋結果。 例如，若您具有 `file_name` "secured_file_b" 的文件的 `group_ids` 欄位設為 `["group_id1, group_id2"]`，則僅有屬於 "group_id1" 或 "group_id2" 群組識別碼的使用者才具備該檔案的讀取權限。
   確定將欄位的 `retrievable` 屬性設為 `false`，使之不會被當作搜尋要求的一部分而傳回。
2. 為了本範例起見，亦請新增 `file_id` 和 `file_name` 欄位。  

```JSON
{
    "name": "securedfiles",  
    "fields": [
        {"name": "file_id", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false},
        {"name": "file_name", "type": "Edm.String"},
        {"name": "group_ids", "type": "Collection(Edm.String)", "filterable": true, "retrievable": false}
    ]
}
```

## <a name="pushing-data-into-your-index-using-the-rest-api"></a>將資料推送至使用 REST API 的索引
  
將 HTTP POST 要求發送至您索引的 URL 端點。 HTTP 要求主體是包含要新增之文件的 JSON 物件：

```
POST https://[search service].search.windows.net/indexes/securedfiles/docs/index?api-version=[api-version]  
Content-Type: application/json
api-key: [admin key]
```

在要求主體中，指定您文件的內容：

```JSON
{
    "value": [
        {
            "@search.action": "upload",
            "file_id": "1",
            "file_name": "secured_file_a",
            "group_ids": ["group_id1"]
        },
        {
            "@search.action": "upload",
            "file_id": "2",
            "file_name": "secured_file_b",
            "group_ids": ["group_id1", "group_id2"]
        },
        {
            "@search.action": "upload",
            "file_id": "3",
            "file_name": "secured_file_c",
            "group_ids": ["group_id5", "group_id6"]
        }
    ]
}
```

若您需要透過群組清單更新現有文件，則可使用 `merge` 或 `mergeOrUpload` 動作：

```JSON
{
    "value": [
        {
            "@search.action": "mergeOrUpload",
            "file_id": "3",
            "group_ids": ["group_id7", "group_id8", "group_id9"]
        }
    ]
}
```

如需與新增或更新文件相關的完整詳細資料，請參閱[編輯文件](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)\(英文\)。
   
## <a name="apply-the-security-filter"></a>套用安全性篩選條件

若要根據 `group_ids` 存取權限調整文件，您應透過 `group_ids/any(g:search.in(g, 'group_id1, group_id2,...'))` 篩選條件發出搜尋查詢，其中 'group_id1, group_id2,...' 是搜尋要求簽發者的所屬群組。
`group_ids` 欄位包含其中一個指定識別碼的所有文件均符合此篩選條件。
如需關於使用 Azure 搜尋服務來搜尋文件的完整詳細資料，請參閱[搜尋文件](https://docs.microsoft.com/rest/api/searchservice/search-documents)\(英文\)。
請注意，此範例會示範如何使用 POST 要求搜尋文件。

發出 HTTP POST 要求：

```
POST https://[service name].search.windows.net/indexes/securedfiles/docs/search?api-version=[api-version]  
Content-Type: application/json  
api-key: [admin or query key]
```

在要求主體中指定篩選條件：

```JSON
{
   "filter":"group_ids/any(g:search.in(g, 'group_id1, group_id2'))"  
}
```

應會取回 `group_ids` 包含 "group_id1" 或 "group_id2" 的文件。 換言之，您會取得要求簽發者具有讀取權限的文件。

```JSON
{
 [
   {
    "@search.score":1.0,
     "file_id":"1",
     "file_name":"secured_file_a",
   },
   {
     "@search.score":1.0,
     "file_id":"2",
     "file_name":"secured_file_b"
   }
 ]
}
```
## <a name="conclusion"></a>結論

以此作法，可根據使用者身分識別與 Azure 搜尋服務 `search.in()` 函式來篩選結果。 您可使用此函式傳入要求使用者的主體識別碼，向每個目標文件比對關聯的主體識別碼。 處理搜尋要求時，`search.in` 函式會篩選出沒有任何使用者主體具備讀取權限的搜尋結果。 主體識別碼可代表安全性群組、角色等等，甚至可代表使用者的專屬身分識別。
 
## <a name="see-also"></a>另請參閱

+ [使用 Azure 搜尋服務篩選來進行 Active Directory 身分識別型存取控制](search-security-trimming-for-azure-search-with-aad.md)
+ [Azure 搜尋服務中的篩選條件](search-filters.md)
+ [Azure 搜尋服務作業中的資料安全性和存取控制](search-security-overview.md)