---
title: 模型化 Azure 儲存體資料表設計中的關聯性 | Microsoft Docs
description: 了解設計表格儲存體解決方案時的模型化程序。
services: storage
author: MarkMcGeeAtAquent
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.author: sngun
ms.component: tables
ms.openlocfilehash: ed4399e2d58924f89d4201979ffe1fb903a05d0c
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39522623"
---
# <a name="modeling-relationships"></a>將關聯性模型化
本文討論的模型化程序有助於您設計 Azure 表格儲存體解決方案。

建置網域模型是設計複雜系統時的關鍵步驟。 一般而言，您可以使用模型化程序來識別實體及其之間的關聯性，藉此了解商業網域，並告知您的系統設計。 本節主要說明如何轉譯網域模型中一些常見的關聯性類型，以設計資料表服務。 從邏輯資料模型對應至實體 NoSQL 架構資料模型的程序，與設計關聯式資料庫時使用的程序不同。 關聯式資料庫設計通常會採用針對最小化備援性而最佳化的資料正規化程序，再搭配宣告式查詢功能以說明如何實作資料庫的運作方式。  

## <a name="one-to-many-relationships"></a>一對多關聯性
商業網域物件之間常會產生一對多關聯性：例如，一個部門有許多員工。 有數種方式可用來實作資料表服務中的一對多關聯性，每種方式都有其可能與特定狀況相關的優缺點。  

舉例來說，假設某家大型跨國公司有數十萬個部門和員工實體，其中每個部門都有許多員工，且每位員工都與一個特定部門相關聯。 其中一個方法是儲存個別部門和這些員工實體，如下所示：  


![儲存個別的部門和員工實體](media/storage-table-design-guide/storage-table-design-IMAGE01.png)

此範例會根據 **PartitionKey** 值說明類型之間的隱含一對多關聯性。 每個部門可以有許多員工。  

此範例也說明相同資料分割中的部門實體及其相關聯的員工實體。 您可以選擇為不同的實體類型使用不同的資料分割、資料表甚或儲存體帳戶。  

替代方式是將資料去正規化，並只使用去正規化的部門資料儲存員工實體，如下列範例所示。 在此特定案例中，如果您需要能夠變更部門經理的詳細資料，這個去正規化方法可能不是最佳選擇，因為此時您必須更新部門中的每一位員工。  

![員工實體](media/storage-table-design-guide/storage-table-design-IMAGE02.png)

如需詳細資訊，請參閱本指南下半部的 [去正規化模式](table-storage-design-patterns.md#denormalization-pattern) 。  

下表摘要說明前述用來儲存具有一對多關聯性的員工和部門實體的各種方法有何優缺點。 您也應考慮您要執行各種作業的頻率：如果不常執行作業，則使包含高成本作業的設計，或許是可行的。  

<table>
<tr>
<th>方法</th>
<th>優點</th>
<th>缺點</th>
</tr>
<tr>
<td>個別的實體類型、相同的資料分割、相同的資料表</td>
<td>
<ul>
<li>您可以使用單一作業更新部門實體。</li>
<li>如果您需要在每次更新/插入/刪除員工實體時修改部門實體，您可以使用 EGT 來維持一致性。 例如，假設您維護每個部門的部門員工計數。</li>
</ul>
</td>
<td>
<ul>
<li>您可能需要針對某些用戶端活動同時擷取員工和部門實體。</li>
<li>儲存作業會在相同資料分割中執行。 在交易量偏高時，這可能會導致熱點。</li>
<li>您無法使用 EGT 將員工移至新部門。</li>
</ul>
</td>
</tr>
<tr>
<td>個別的實體類型、不同的資料分割或資料表或儲存體帳戶</td>
<td>
<ul>
<li>您可以使用單一作業更新部門實體或員工實體。</li>
<li>在交易量偏高時，這可能有助於將負載分散到多個資料分割。</li>
</ul>
</td>
<td>
<ul>
<li>您可能需要針對某些用戶端活動同時擷取員工和部門實體。</li>
<li>當您更新/插入/刪除員工和更新部門時，您無法使用 EGT 維護一致性。 例如，更新某個部門實體中的員工計數。</li>
<li>您無法使用 EGT 將員工移至新部門。</li>
</ul>
</td>
</tr>
<tr>
<td>去正規化為單一實體類型</td>
<td>
<ul>
<li>您可以透過單一要求擷取您所需的所有資訊。</li>
</ul>
</td>
<td>
<ul>
<li>如果您需要更新部門資訊，維持一致性可能會很昂貴 (您可能必須更新部門中的所有員工)。</li>
</ul>
</td>
</tr>
</table>

您選擇這些選項的方式及其優缺點是最重要的，這取決於您的應用程式案例。 例如，您多久修改一次部門實體；您所有的員工查詢是否都需要其他部門資訊；您的磁碟分割或儲存體帳戶有多接近延展性限制？  

## <a name="one-to-one-relationships"></a>一對一關聯性
網域模型的實體之間可能會包含一對一關聯性。 如果您需要在資料表服務中實作一對一關聯性，您在需要同時擷取兩個相關的實體時，也必須選擇如何連結兩者。 此連結可以是隱含的 (根據索引鍵值中的慣例) 或明確的 (藉由以 **PartitionKey** 和 **RowKey** 值的形式，將每個實體中的連結儲存至其相關的實體)。 如需是否應將相關實體儲存於相同資料分割中的討論，請參閱 [一對多關聯性](#one-to-many-relationships)一節。  

也有可能會導致您在表格服務中實作一對一關聯性的實作考量：  

* 處理大型實體 (如需詳細資訊，請參閱 [大型實體模式](table-storage-design-patterns.md#large-entities-pattern))。  
* 實作存取控制 (如需詳細資訊，請參閱 [使用共用存取簽章控制存取](#controlling-access-with-shared-access-signatures))。  

## <a name="join-in-the-client"></a>用戶端中的聯結
雖然有多種方式可以模型化資料表服務中的關聯性，但您應該記住，使用資料表服務的兩個主要原因是延展性和效能。 如果您發現您將許多包含方案的效能和延展性的關聯性模型化，您應該自問是否需要在資料表設計中建置所有的資料關聯性。 如果您讓用戶端應用程式執行任何必要的聯結，或許可以簡化設計並改善方案的效能與延展性。  

例如，如果您有小型資料表，且其中包含未經常變更的資料，您就可以擷取此資料一次，並在用戶端上加以快取。 這可以避免重複擷取相同資料的往返。 在我們已於本指南中討論的範例中，小型組織中的部門集合很可能較小且不常變更，因此適合做為可由用戶端應用程式下載一次並快取為查閱資料的資料。  

## <a name="inheritance-relationships"></a>繼承關聯性
如果您的用戶端應用程式使用一組形成部分繼承關聯性的類別來代表商業實體，您可以輕鬆地在資料表服務中保存這些實體。 例如，您可能會在用戶端應用程式中定義下列一組類別，其中 **Person** 是抽象類別。

![抽象人員類別](media/storage-table-design-guide/storage-table-design-IMAGE03.png)

您可以透過使用實體如下所示的單一 Person 資料表，在資料表服務中保存兩個固定類別的執行個體：  

![人員資料表](media/storage-table-design-guide/storage-table-design-IMAGE04.png)

如需在用戶端程式碼中於相同資料表上使用多個實體類型的詳細資訊，請參閱本指南稍後的 [使用異質性實體類型](#working-with-heterogeneous-entity-types) 一節。 其中提供如何在用戶端程式碼中辨識實體類型的範例。  


## <a name="next-steps"></a>後續步驟

- [資料表設計模式](table-storage-design-patterns.md)
- [查詢的設計](table-storage-design-for-query.md)
- [加密資料表資料](table-storage-design-encrypt-data.md)
- [資料修改的設計](table-storage-design-for-modification.md)