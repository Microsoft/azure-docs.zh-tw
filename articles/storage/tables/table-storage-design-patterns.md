---
title: Azure 儲存體資料表設計模式 | Microsoft Docs
description: 使用適用於 Azure 表格服務解決方案的模式。
services: storage
author: MarkMcGeeAtAquent
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.author: sngun
ms.component: tables
ms.openlocfilehash: b06f5a66566c250eef608ddccc551aaebe24ef74
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39522793"
---
# <a name="table-design-patterns"></a>資料表設計模式
本文將說明一些適用於表格服務方案的模式。 此外，您會了解如何有效處理其他表格儲存體設計文章中討論的一些問題和取捨。 下圖摘要說明不同模式之間的關聯性：  

![查閱相關資料](media/storage-table-design-guide/storage-table-design-IMAGE05.png)


上方的模式圖強調顯示本指南中提及的模式 (藍色) 與反向模式 (橘色) 之間的一些關聯性。 還有許多其他模式也值得考量。 例如，表格服務的其中一個主要案例是從[命令查詢責任隔離 (CQRS)](https://msdn.microsoft.com/library/azure/jj554200.aspx) 模式使用[具體化檢視模式](https://msdn.microsoft.com/library/azure/dn589782.aspx)。  

## <a name="intra-partition-secondary-index-pattern"></a>內部資料分割次要索引模式
為每個實體儲存多個複本且使用不同 **RowKey** 值 (在相同的資料分割內)，透過使用不同的 **RowKey** 值，就能快速且有效率的查閱和替代排序次序。 複本之間的更新可以使用 EGT 保持一致。  

### <a name="context-and-problem"></a>內容和問題
表格服務會使用 **PartitionKey** 和 **RowKey** 值自動編製實體的索引。 這可讓用戶端應用程式使用這些值有效率地擷取實體。 例如，若使用如下的資料表結構，用戶端應用程式將可使用點查詢透過部門名稱和員工識別碼 (**PartitionKey** 和 **RowKey** 值) 來擷取個別員工實體。 用戶端也可以擷取每個部門內以員工識別碼排序的實體。

![Image06](media/storage-table-design-guide/storage-table-design-IMAGE06.png)

如果您也想能夠根據其他屬性 (例如電子郵件地址) 的值尋找員工實體，您必須使用效率較低的資料分割掃描來尋找相符項目。 這是因為資料表服務不會提供次要索引。 此外，您無法要求以 **RowKey** 順序以外的不同順序來排序的員工清單。  

### <a name="solution"></a>解決方法
若要解決缺少次要索引的問題，您可以為每個實體儲存多個複本，且每個複本分別使用不同 **RowKey** 值。 如果您使用如下所示的結構來儲存實體，就能夠根據電子郵件地址或員工識別碼，有效率地擷取員工實體。 **RowKey**、"empid_" 及 "email_" 的前置詞值可讓您使用一組電子郵件地址或員工識別碼的範圍，來查詢單一員工或某個範圍內的員工。  

![員工實體](media/storage-table-design-guide/storage-table-design-IMAGE07.png)

下列兩個篩選準則 (一個依員工識別碼查閱，另一個依電子郵件地址查閱) 都會指定點查詢：  

* $filter=(PartitionKey eq 'Sales') and (RowKey eq 'empid_000223')  
* $filter=(PartitionKey eq 'Sales') and (RowKey eq 'email_jonesj@contoso.com')  

如果您查詢某範圍的員工實體，您可以指定以員工識別碼順序排序的範圍，或藉由查詢在 **RowKey** 中有適當前置詞的實體，指定以電子郵件地址順序排序的範圍。  

* 若要在銷售部門中，找出員工識別碼範圍從 000100 至 000199 的所有員工：請使用 $filter=(PartitionKey eq 'Sales') and (RowKey ge 'empid_000100') and (RowKey le 'empid_000199')  
* 若要在銷售部門中，找出電子郵件地址開頭為字母 'a' 的所有員工：請使用 $filter=(PartitionKey eq 'Sales') and (RowKey ge 'email_a') and (RowKey lt 'email_b')  
  
  請注意，上述範例中使用的篩選語法來自於表格服務 REST API，如需詳細資訊，請參閱 [查詢實體](http://msdn.microsoft.com/library/azure/dd179421.aspx)。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 資料表儲存體的使用成本相對較低廉，因此儲存重複資料的成本負擔應該不是主要的考量。 不過，您一律應根據預期的儲存需求評估設計成本，並僅新增重複實體來支援用戶端應用程式將會執行的查詢。  
* 因為次要索引實體儲存在與原始實體相同的磁碟分割中，因此您應該確定不會超過個別資料分割的延展性目標。  
* 您可以將重複實體彼此保持一致，方法是使用 EGT 自動更新實體的兩個複本。 這表示您應該將實體的所有複本儲存在相同的資料分割中。 如需詳細資訊，請參閱 [使用實體群組交易](table-storage-design.md#entity-group-transactions)一節。  
* 每個實體用於 **RowKey** 的值必須是唯一的。 請考慮使用複合索引鍵值。  
* 在 **RowKey** 中填補數值 (例如，員工識別碼 000223)，可根據上限與下限正確進行排序和篩選。  
* 您不一定需要複製實體的所有屬性。 例如，如果在 **RowKey** 中使用電子郵件地址查閱實體的查詢永遠都不需要員工的年齡，這些實體可能有下列結構：

   ![員工實體結構](media/storage-table-design-guide/storage-table-design-IMAGE08.png)


* 一般而言，儲存重複資料並確保您可以透過單一查詢擷取您需要的所有資料，通常會比使用一個查詢來尋找實體並以另一個查閱所需資料更為理想。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
當用戶端應用程式需要使用各種不同的索引鍵擷取實體時、當用戶端需要擷取不同排序次序的實體時，以及您可以使用不同的唯一值識別每個實體時，請使用此模式。 不過，您應確定在使用不同的 **RowKey** 值執行實體查閱時，您不會超出資料分割延展性限制。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [間分割次要索引模式](#inter-partition-secondary-index-pattern)
* [複合索引鍵模式](#compound-key-pattern)
* [實體群組交易](#entity-group-transactions)
* [使用異質性實體類型](#working-with-heterogeneous-entity-types)

## <a name="inter-partition-secondary-index-pattern"></a>間資料分割次要索引模式
在個別資料分割或個別資料表中為每個實體儲存多個複本且使用不同 **RowKey** 值，透過使用不同的 **RowKey** 值，就能快速且有效率的查閱和替代排序次序。  

### <a name="context-and-problem"></a>內容和問題
表格服務會使用 **PartitionKey** 和 **RowKey** 值自動編製實體的索引。 這可讓用戶端應用程式使用這些值有效率地擷取實體。 例如，若使用如下的資料表結構，用戶端應用程式將可使用點查詢透過部門名稱和員工識別碼 (**PartitionKey** 和 **RowKey** 值) 來擷取個別員工實體。 用戶端也可以擷取每個部門內以員工識別碼排序的實體。  

![員工識別碼](media/storage-table-design-guide/storage-table-design-IMAGE09.png)

如果您也想能夠根據其他屬性 (例如電子郵件地址) 的值尋找員工實體，您必須使用效率較低的資料分割掃描來尋找相符項目。 這是因為資料表服務不會提供次要索引。 此外，您無法要求以 **RowKey** 順序以外的不同順序來排序的員工清單。  

您預期這些實體會有極大量的交易，而想要為您的用戶端節流以將資料表服務的風險降至最低。  

### <a name="solution"></a>解決方法
若要解決缺少次要索引的問題，您可以為每個實體儲存多個複本，且每個複本分別使用不同的 **PartitionKey** 和 **RowKey** 值。 如果您使用如下所示的結構來儲存實體，就能夠根據電子郵件地址或員工識別碼，有效率地擷取員工實體。 **PartitionKey**、"empid_" 及 "email_" 的前置詞值可讓您識別想要用於查詢的索引。  

![主要索引和次要索引](media/storage-table-design-guide/storage-table-design-IMAGE10.png)


下列兩個篩選準則 (一個依員工識別碼查閱，另一個依電子郵件地址查閱) 都會指定點查詢：  

* $filter=(PartitionKey eq 'empid_Sales') and (RowKey eq '000223')
* $filter=(PartitionKey eq 'email_Sales') and (RowKey eq 'jonesj@contoso.com')  

如果您查詢某範圍的員工實體，您可以指定以員工識別碼順序排序的範圍，或藉由查詢在 **RowKey** 中有適當前置詞的實體，指定以電子郵件地址順序排序的範圍。  

* 若要在銷售部門中，找出員工識別碼範圍從 **000100** 至 **000199** 以員工識別碼順序排序的所有員工，請使用：$filter=(PartitionKey eq 'empid_Sales') and (RowKey ge '000100') and (RowKey le '000199')  
* 若要在銷售部門中，找出電子郵件地址以 'a' 開頭的所有員工，請使用：$filter=(PartitionKey eq 'email_Sales') and (RowKey ge 'a') and (RowKey lt 'b')  

請注意，上述範例中使用的篩選語法來自於表格服務 REST API，如需詳細資訊，請參閱 [查詢實體](http://msdn.microsoft.com/library/azure/dd179421.aspx)。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 您可以讓重複實體彼此之間始終保持一致，方法是使用 [最終一致的交易模式](#eventually-consistent-transactions-pattern) 來維護主要和次要索引實體。  
* 資料表儲存體的使用成本相對較低廉，因此儲存重複資料的成本負擔應該不是主要的考量。 不過，您一律應根據預期的儲存需求評估設計成本，並僅新增重複實體來支援用戶端應用程式將會執行的查詢。  
* 每個實體用於 **RowKey** 的值必須是唯一的。 請考慮使用複合索引鍵值。  
* 在 **RowKey** 中填補數值 (例如，員工識別碼 000223)，可根據上限與下限正確進行排序和篩選。  
* 您不一定需要複製實體的所有屬性。 例如，如果在 **RowKey** 中使用電子郵件地址查閱實體的查詢永遠都不需要員工的年齡，這些實體可能有下列結構：
  
   ![員工實體 (次要索引)](media/storage-table-design-guide/storage-table-design-IMAGE11.png)

* 一般而言，儲存重複資料並確保您可以透過單一查詢擷取您需要的所有資料，通常會比使用一個查詢透過次要索引尋找實體、並以另一個查詢來查閱主要索引中的所需資料更為理想。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
當用戶端應用程式需要使用各種不同的索引鍵擷取實體時、當用戶端需要擷取不同排序次序的實體時，以及您可以使用不同的唯一值識別每個實體時，請使用此模式。 當您使用不同的 **RowKey** 值執行實體查閱時，如果要避免超出資料分割延展性限制，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [最終一致的交易模式](#eventually-consistent-transactions-pattern)  
* [內部資料分割次要索引模式](#intra-partition-secondary-index-pattern)  
* [複合索引鍵模式](#compound-key-pattern)  
* [實體群組交易](#entity-group-transactions)  
* [使用異質性實體類型](#working-with-heterogeneous-entity-types)  

## <a name="eventually-consistent-transactions-pattern"></a>最終一致的交易模式
使用 Azure 佇列，跨資料分割界限或儲存體系統界限啟用最終一致的行為。  

### <a name="context-and-problem"></a>內容和問題
EGT 可讓您在共用相的資料分割索引鍵的多個實體之間執行不可部分完成的交易。 基於效能和擴充性的考量，您可能會決定在個別的磁碟分割或不同的儲存體系統中儲存有一致性需求的實體：在這種情況下，您無法使用 EGT 維護一致性。 例如，您可能必須在下列項目間維護最終一致性：  

* 儲存在相同資料表的兩個不同分割區中、不同的資料表中或不同儲存體帳戶中的實體。  
* 儲存在資料表服務中的實體和儲存在 Blob 服務中的 Blob。  
* 儲存在資料表服務中的實體和檔案系統中的檔案。  
* 儲存在資料表服務中、但使用 Azure 搜尋服務編製索引的實體。  

### <a name="solution"></a>解決方法
藉由使用 Azure 佇列，您可以實作解決方案，提供跨兩個或多個資料分割或儲存系統的最終一致性。
為了說明這種方法，我們假設您需要能夠封存舊的員工實體。 舊的員工實體很少受到查詢，應從處理目前員工的任何活動中排除。 若要實作這項需求，您必須將作用中員工儲存在**目前**資料表中，並將舊員工儲存在**封存**資料表中。 要封存員工，您必須先從**目前**資料表中刪除實體，並將實體加入**封存**資料表，但您無法使用 EGT 來執行這兩項作業。 若要避免因失敗而導致實體同時出現或未出現在這兩個資料表中，封存作業必須最終一致。 下列順序圖說明此作業的步驟。 後續文字提供了例外狀況路徑的詳細資料。  

![Azure 佇列解決方案](media/storage-table-design-guide/storage-table-design-IMAGE12.png)

用戶端會藉由將訊息放在 Azure 佇列來起始封存作業，在此範例中會封存員工 #456。 背景工作角色會輪詢佇列中的新訊息；若找到新訊息，它會讀取訊息，並將隱藏的複本保留在佇列上。 背景工作角色接著會擷取來自**目前**資料表的實體複本、將複本插入**封存**資料表中，然後從**目前**資料表中刪除原始複本。 最後，如果前述步驟沒有任何錯誤，背景工作角色會從佇列中刪除隱藏的訊息。  

此範例的步驟 4 會將員工插入 **封存** 資料表。 這可以將員工新增至 Blob 服務中的 Blob 或檔案系統中的檔案。  

### <a name="recovering-from-failures"></a>從失敗復原
步驟 **4** 和 **5** 中的作業務必等冪，免得背景工作角色必須重新啟動封存作業。 使用表格服務時，在步驟 **4** 中，您應使用「插入或取代」作業；在步驟 **5** 中，則應在您使用的用戶端程式庫中使用「如果存在即刪除」作業。 如果您使用其他儲存體系統，您必須使用適當的冪等作業。  

如果背景工作角色一直未完成步驟 **6**，則在逾時後，訊息會重新出現在佇列上，可讓背景工作角色嘗試重新加以處理。 背景工作角色可以檢查訊息在佇列上的已讀取次數，如有必要可將其傳送至不同的佇列，以標示為「有害」訊息接受調查。 如需讀取佇列訊息及檢查清除佇列計數的詳細資訊，請參閱 [取得訊息](https://msdn.microsoft.com/library/azure/dd179474.aspx)。  

來自資料表和佇列服務的某些錯誤是暫時性的，用戶端應用程式應包含適當的重試邏輯來處理它們。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 此方案不提供交易隔離。 例如，用戶端在在背景工作角色介於步驟 **4** 和 **5** 之間時無法讀取**目前**和**封存**資料表，並且會看見不一致的資料檢視。 請注意，資料最終將會一致。  
* 您必須確定步驟 4 和 5 是等冪，以確保最終一致性。  
* 您可以使用多個佇列和背景工作角色執行個體來調整方案。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您想要確保存在於不同磁碟分割或資料表中的實體之間保有最終一致性，請使用此模式。 您可以擴充此模式，以確保整個表格服務與 Blob 服務和其他非 Azure 儲存體資料來源 (例如資料庫或檔案系統) 的作業保有最終一致性。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [實體群組交易](#entity-group-transactions)  
* [合併或取代](#merge-or-replace)  

> [!NOTE]
> 如果交易隔離對您的方案而言很重要，您應該考慮重新設計，讓您能夠使用 EGT 資料表。  
> 
> 

## <a name="index-entities-pattern"></a>索引實體模式
維護索引項目，啟用有效的搜尋以傳回實體清單。  

### <a name="context-and-problem"></a>內容和問題
表格服務會使用 **PartitionKey** 和 **RowKey** 值自動編製實體的索引。 這可讓用戶端應用程式使用點查詢有效率地擷取實體。 例如，若使用如下的資料表結構，用戶端應用程式將可有效率地透過部門名稱和員工識別碼 (**PartitionKey** 和 **RowKey**) 來擷取個別員工實體。  

![員工實體](media/storage-table-design-guide/storage-table-design-IMAGE13.png)

如果您也想能夠根據其他非唯一屬性 (例如其姓氏) 的值擷取員工實體清單，您必須使用效率較低的資料分割掃描來尋找相符項目，而不要使用索引直接加以查閱。 這是因為資料表服務不會提供次要索引。  

### <a name="solution"></a>解決方法
若要透過如上所示的實體結構啟用依據姓氏的查閱，您必須維護員工識別碼清單。 如果您想要擷取具有特定姓氏 (例如 Jones) 的員工實體，您必須先針對姓氏為 Jones 的員工找出員工識別碼清單，然後擷取這些員工實體。 有三個主要的選項可儲存員工識別碼清單：  

* 使用 Blob 儲存體。  
* 在與員工實體相同的磁碟分割中建立索引實體。  
* 在個別的資料分割或資料表中建立索引實體。  

<u>選項 1：使用 Blob 儲存體</u>  

使用第一個選項時，您會為每個唯一的姓氏建立一個 Blob，並在每個 Blob 中，針對具有該姓氏的員工儲存 **PartitionKey** (部門) 和 **RowKey** (員工識別碼) 值的清單。 當您新增或刪除某位員工時，您應該確保相關的 Blob 內容與員工實體最終一致。  

<u>選項 2：</u>在相同的資料分割中建立索引實體  

使用第二個選項時，您會使用儲存下列資料的索引實體：  

![員工索引實體](media/storage-table-design-guide/storage-table-design-IMAGE14.png)

**EmployeeIDs** 屬性包含姓氏儲存在 **RowKey** 中之員工的員工識別碼清單。  

下列步驟概述您在使用第二個選項並且要新增員工時所應遵循的程序。 在此範例中，我們會新增在銷售部門中識別碼為 000152、且姓氏為 Jones 的員工：  

1. 擷取具有 **PartitionKey** 值 "Sales" 和 **RowKey** 值 "Jones" 的索引實體。 儲存此實體的 ETag　以在步驟 2 中使用。  
2. 建立實體群組交易 (也就是批次作業)，插入新的員工實體 (**PartitionKey** 值 "Sales" 和 **RowKey** 值 "000152") 並更新索引實體 (**PartitionKey** 值 "Sales" 和 **RowKey**值 "Jones")，方法是將新員工識別碼加入 EmployeeIDs 欄位中的清單。 如需實體群組交易的詳細資訊，請參閱 [實體群組交易](#entity-group-transactions)。  
3. 如果實體群組交易因為開放式並行存取錯誤而失敗 (其他人剛修改過索引實體)，您就必須從步驟 1 重新執行。  

如果您使用第二個選項，您可以使用類似的方法來刪除某位員工。 變更員工的姓氏會稍微複雜一點，因為您需要執行會更新三個實體的實體群組交易：員工實體、舊姓氏的索引實體，與新姓氏的索引實體。 您必須先擷取每個實體才能進行變更，以擷取接著可以用來透過開放式並行存取執行更新的 ETag 值。  

下列步驟概述您在使用第二個選項、並需要查閱部門中所有具有給定姓氏的員工時所應遵循的程序。 在此範例中，我們會尋找銷售部門中所有姓氏為 Jones 的員工：  

1. 擷取具有 **PartitionKey** 值 "Sales" 和 **RowKey** 值 "Jones" 的索引實體。  
2. 剖析 EmployeeIDs 欄位中的員工識別碼清單。  
3. 如果您需要這些員工的詳細資訊 (例如其電子郵件地址)，請從您在步驟 2 中取得的員工清單使用 **PartitionKey** 值 "Sales" 和 **RowKey** 值來擷取每個員工實體。  

<u>選項 3：</u>在個別的資料分割或資料表中建立索引實體  

使用第三個選項時，您會使用儲存下列資料的索引實體：  

![不同分割區中的員工索引實體](media/storage-table-design-guide/storage-table-design-IMAGE15.png)


**EmployeeIDs** 屬性包含姓氏儲存在 **RowKey** 中之員工的員工識別碼清單。  

使用第三個選項時，您無法使用 EGT 來維持一致性，因為索引實體位於與員工實體不同的磁碟分割中。 您應確保索引實體與員工實體最終一致。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 此方案至少需要兩個查詢來擷取相符實體：一個用來查詢索引實體以取得 **RowKey** 值清單，然後查詢以擷取清單中的每個實體。  
* 假設個別實體的大小上限為 1 MB，方案中的選項 2 和選項 3 會假設任何給定姓氏的員工識別碼清單絕不會大於 1 MB。 如果員工識別碼清單的大小可能大於 1 MB，請使用選項 1，並將索引資料儲存在 Blob 儲存體中。  
* 如果您使用選項 2 (使用 EGT 處理新增和刪除員工以及變更員工姓氏的作業)，您必須評估交易量是否會接近指定資料分割中的延展性限制。 如果會，您應考慮最終一致方案 (選項 1 或選項 3)，以使用佇列來處理更新要求，並讓您將索引實體儲存在與員工實體不同的個別資料分割中。  
* 此方案中的選項 2 會假設您想要依姓氏在某部門內進行查閱：例如，您要擷取銷售部門中姓氏為 Jones 的員工清單。 如果您想要能夠查閱整個組織中姓氏為 Jones 的所有員工，請使用選項 1 或選項 3。
* 您可以實作可提供最終一致性的佇列型方案 (如需詳細資訊，請參閱 [最終一致的交易模式](#eventually-consistent-transactions-pattern) )。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您想要查閱全部共用通用屬性值的一組實體 (例如姓氏為 Jones 的所有員工)，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [複合索引鍵模式](#compound-key-pattern)  
* [最終一致的交易模式](#eventually-consistent-transactions-pattern)  
* [實體群組交易](#entity-group-transactions)  
* [使用異質性實體類型](#working-with-heterogeneous-entity-types)  

## <a name="denormalization-pattern"></a>反正規化模式
將相關資料結合在單一實體中，讓您透過單點查詢擷取所有您所需的資料。  

### <a name="context-and-problem"></a>內容和問題
在關聯式資料庫中，您通常會將資料正規化，以移除會產生查詢而從多個資料表擷取資料的重複資料。 如果您將 Azure 資料表中的資料標準化，您必須從用戶端到伺服器往返多次才能擷取相關的資料。 以如下的資料表結構為例，您需要往返兩次才能擷取某部門的詳細資料：其中一個要求是用來擷取包含經理識別碼的部門實體，另一個要求是用來擷取員工實體中的經理詳細資料。  

![部門實體與員工實體](media/storage-table-design-guide/storage-table-design-IMAGE16.png)

### <a name="solution"></a>解決方法
不要將資料儲存在兩個不同的實體中，而是將資料反正規化，並將管理員詳細資料的複本保存在部門實體中。 例如︰  

![部門實體](media/storage-table-design-guide/storage-table-design-IMAGE17.png)

部門實體連同這些屬性一起儲存後，您現在可以擷取與使用點查詢的部門有關的所有詳細資料。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 儲存資料兩次還有一些相關的成本負擔。 效能優勢 (因對儲存體服務的要求較少而產生) 通常會高於儲存體成本的邊際增值 (而且這項成本有部分會由擷取部門的詳細資料所需的交易量減少而抵銷)。  
* 您必須讓儲存管理員相關資訊的兩個實體保有一致性。 您可以使用 EGT 在單一不可部分完成交易中更新多個實體，以處理一致性問題：在此情況下，部門實體和部門經理的員工實體會儲存在相同的資料分割中。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您經常需要查閱相關資訊，請使用此模式。 此模式可減少用戶端在擷取其所需資料時必須執行的查詢數目。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [複合索引鍵模式](#compound-key-pattern)  
* [實體群組交易](#entity-group-transactions)  
* [使用異質性實體類型](#working-with-heterogeneous-entity-types)

## <a name="compound-key-pattern"></a>複合索引鍵模式
使用複合 **RowKey** 值，讓用戶端可透過單點查詢來查閱相關資料。  

### <a name="context-and-problem"></a>內容和問題
在關聯式資料庫中常會在查詢中使用聯結，將單一查詢中的相關資料片段傳回至用戶端。 例如，您可以使用員工識別碼來查閱包含該員工之績效和考核資料的相關實體清單。  

假設您要使用下列結構在資料表服務中儲存員工實體：  

![員工實體結構](media/storage-table-design-guide/storage-table-design-IMAGE18.png)

您也需要儲存該員工在組織中工作各年度的考核和績效相關歷史資料，而且您必須能夠依年份存取這項資訊。 建立另一個資料表來儲存具有下列結構的實體，是可行選項之一：  

![替代的員工實體結構](media/storage-table-design-guide/storage-table-design-IMAGE19.png)

請注意，使用此方法時，您可以選擇在新的實體中重複某些資訊 (例如名字和姓氏)，以便透過單一要求擷取您的資料。 不過，您無法維護強式一致性，因為您無法使用 EGT 自動更新兩個實體。  

### <a name="solution"></a>解決方法
使用具有下列結構的實體，在您的原始資料表中儲存新的實體類型：  

![員工實體結構的解決方案](media/storage-table-design-guide/storage-table-design-IMAGE20.png)

請注意，**RowKey** 現在是由員工識別碼和考核資料年份所組成的複合索引鍵，可讓您透過單一實體的單一要求擷取員工的績效和考核資料。  

下列範例說明如何擷取特定員工的所有考核資料 (例如銷售部門的員工 000123)：  

$filter=(PartitionKey eq 'Sales') and (RowKey ge 'empid_000123') and (RowKey lt 'empid_000124')&$select=RowKey,Manager Rating,Peer Rating,Comments  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 您應該使用適當的分隔符號字元以便剖析 **RowKey** 值：例如 **000123_2012**。  
* 您也必須將此實體儲存在與包含相同員工之相關資料的其他實體相同的資料分割中，這表示您可以使用 EGT 來維護強式一致性。
* 您應考量您查詢資料的頻率，以判斷此模式是否適用。  比方說，如果您不會經常存取考核資料和主要員工資料，則應將其保存為個別的實體。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您需要儲存一或多個您經常查詢的相關實體，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [實體群組交易](#entity-group-transactions)  
* [使用異質性實體類型](#working-with-heterogeneous-entity-types)  
* [最終一致的交易模式](#eventually-consistent-transactions-pattern)  

## <a name="log-tail-pattern"></a>記錄結尾模式
使用以反向的日期和時間順序排序的 *RowKey* 值，擷取最近加入資料分割的 **n** 個實體。  

### <a name="context-and-problem"></a>內容和問題
常見的需求是要能夠擷取最近建立的實體，例如員工最近提交的費用請款。 資料表查詢支援 **$top** 查詢作業，以從某個集合中傳回前 *n* 個實體：沒有對等的查詢作業可傳回某個集合中的最後 n 個實體。  

### <a name="solution"></a>解決方法
儲存使用可自然以反向的日期/時間順序排序的 **RowKey** 的實體，使最新的項目一律排在資料表中的首位。  

比方說，若要能夠擷取某員工最近提交的十筆費用請款，您可以使用衍生自目前日期/時間的反向刻度值。 下列 C# 程式碼範例說明如何針對從最新排序到最舊的 **RowKey** 建立適當的「反向刻度」值：  

`string invertedTicks = string.Format("{0:D19}", DateTime.MaxValue.Ticks - DateTime.UtcNow.Ticks);`  

您可以使用下列程式碼回到日期時間值：  

`DateTime dt = new DateTime(DateTime.MaxValue.Ticks - Int64.Parse(invertedTicks));`  

資料表查詢如下所示：  

`https://myaccount.table.core.windows.net/EmployeeExpense(PartitionKey='empid')?$top=10`  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 您必須為反向刻度值填補前置零，以確保字串值如預期排序。  
* 您必須知道資料分割層級的延展性目標。 請留意不要建立熱點資料分割。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
當您需要存取反向日期/時間順序的實體時，或當您需要存取最近新增的實體時，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [在前面加上/附加反向模式](#prepend-append-anti-pattern)  
* [擷取實體](#retrieving-entities)  

## <a name="high-volume-delete-pattern"></a>大量刪除模式
藉由將所有要同時刪除的實體儲存在其本身的個別資料表中，讓您能夠刪除大量的實體；您可以藉由刪除資料表來刪除實體。  

### <a name="context-and-problem"></a>內容和問題
許多應用程式刪除用戶端應用程式已無需使用的舊資料，或應用程式已封存至其他儲存媒體的舊資料。 一般而言，您可以依日期來識別這類資料：例如，您需要刪除所有存留期超過 60 天之登入要求的記錄。  

可行的設計之一，就是在 **RowKey**中使用登入要求的日期和時間：  

![登入嘗試的日期和時間](media/storage-table-design-guide/storage-table-design-IMAGE21.png)

這個方法可避免產生資料分割熱點，因為應用程式可以在個別的資料分割中插入和刪除每一位使用者的登入實體。 不過如果您有大量的實體，這種方法可能既昂貴又耗時，因為您必須先執行資料表掃描以識別所有要刪除的實體，然後必須刪除每個舊的實體。 請注意，您可以藉由將多個刪除要求批次處理到 EGT 中，以減少刪除舊實體所需的伺服器往返次數。  

### <a name="solution"></a>方案
為每天的登入嘗試使用個別的資料表。 當您要插入的實體時，您可以使用上述的實體設計來避免熱點，且刪除舊實體目前只不過是每天刪除一個資料表 (單一儲存體作業) 的問題而已，而無須每天尋找和刪除成千上百的個別登入實體。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 您的設計是否支援應用程式使用資料的其他方式，例如查閱特定實體、連結其他資料或產生彙總資訊？  
* 您的設計在插入新實體時是否可避免產生熱點？  
* 如果您在刪除某個資料表名稱後想要加以重複使用，預期應會有延遲。 最好是一律使用唯一的資料表名稱。  
* 當您第一次使用新的資料表時，若資料表服務可辨識存取模式，並將資料分割散發到各節點，預期應會有節流。 您應考量您需要建立新資料表的頻率。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您有大量的實體必須同時刪除，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [實體群組交易](#entity-group-transactions)
* [修改實體](#modifying-entities)  

## <a name="data-series-pattern"></a>資料序列模式
將完整資料序列儲存在單一實體中，以盡可能減少您提出的要求數。  

### <a name="context-and-problem"></a>內容和問題
常見的案例是應用程式需一次儲存必須經常擷取的資料序列。 比方說，您的應用程式可能會記錄每一位員工每小時傳送多少 IM 訊息，然後使用這項資訊來繪製每個使用者在過去 24 小時內傳送的訊息數。 一個設計可為每個員工儲存 24 個實體：  

![為每個員工儲存 24 個實體](media/storage-table-design-guide/storage-table-design-IMAGE22.png)

採用這種設計，您可以輕鬆地找出並更新實體，以在應用程式需要更新訊息計數值時更新每個員工。 不過，若要擷取資訊以繪製過去 24 小時內的活動圖，您必須擷取 24 個實體。  

### <a name="solution"></a>解決方法
使用下列具有個別屬性的設計，儲存每個小時的訊息計數：  

![訊息統計資料實體](media/storage-table-design-guide/storage-table-design-IMAGE23.png)

透過這項設計，您可以使用合併作業來更新員工在特定時段內的訊息計數。 現在，您可以使用單一實體的要求，擷取您要繪圖所需的所有資訊。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 如果您完整資料序列不適用於單一實體 (一個實體可以有 252 個屬性)，請使用替代資料存放區，例如 Blob。  
* 如果您有多個用戶端同時更新實體，您必須使用 **ETag** 實作開放式並行存取。 如果您有許多用戶端，則預期應會有激烈的爭用情況。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您需要更新和擷取與個別實體相關聯的資料序列，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [大型實體模式](#large-entities-pattern)  
* [合併或取代](#merge-or-replace)  
* [最終一致的交易模式](#eventually-consistent-transactions-pattern) (如果您要將資料數列儲存在 Blob 中)  

## <a name="wide-entities-pattern"></a>寬型實體模式
使用多個實際的實體來儲存具有超過 252 個屬性的邏輯實體。  

### <a name="context-and-problem"></a>內容和問題
個別實體可以擁有超過 252 個 (不含必要的系統屬性) 屬性，而且無法儲存總計超過 1 MB 的資料。 在關聯式資料庫中，您會通常可藉由新增資料表並對其施行一對一關聯性，來解決任何資料列的大小限制。  

### <a name="solution"></a>解決方法
使用資料表服務，可讓您儲存多個實體來代表具有超過 252 個屬性的單一大型商業物件。 例如，如果您想要儲存每個員工在 365 天內傳送的 IM 訊息計數，您可以採用下列設計，使用兩個具有不同結構描述的實體：  

![多個實體](media/storage-table-design-guide/storage-table-design-IMAGE24.png)

如果您需要進行必須同時更新兩個實體的變更，讓它們彼此保持同步，您可以使用 EGT。 否則，您可以使用合併作業來更新某天的訊息計數。 若要擷取個別員工的所有資料，您必須都擷取這兩個實體；您可以透過兩個同時使用 **PartitionKey** 和 **RowKey** 值的有效要求來完成此動作。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 擷取完成邏輯實體牽涉到至少兩個儲存體交易：一個用來擷取每個實際的實體。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果需要在資料表服務中儲存屬性的大小或數目超過個別實體限制的實體，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [實體群組交易](#entity-group-transactions)
* [合併或取代](#merge-or-replace)

## <a name="large-entities-pattern"></a>大型實體模式
使用 Blob 儲存體來儲存大型屬性值。  

### <a name="context-and-problem"></a>內容和問題
個別實體無法儲存總計超過 1 MB 的資料。 如果有一或多個屬性所儲存的值會導致您的實體大小總計超過此值，您將無法在資料表服務中儲存整個實體。  

### <a name="solution"></a>解決方法
如果您的實體因為一或多個屬性包含大量資料而使大小超過 1 MB，您可以將資料儲存在 Blob 服務，然後將 Blob 的位址儲存在實體的屬性中。 比方說，您可以在 Blob 儲存體中儲存員工的相片，並將相片的連結儲存在員工實體的 **Photo** 屬性中：  

![相片屬性](media/storage-table-design-guide/storage-table-design-IMAGE25.png)

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 若要讓資料表格服務中的實體與 Blob 服務中的資料之間保有最終一致性，請使用 [最終一致的交易模式](#eventually-consistent-transactions-pattern) 維護您的實體。
* 擷取完整實體牽涉到至少兩個儲存體交易：一個用來擷取實體，另一個擷取 Blob 資料。  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果需要在資料表服務中儲存大小超過個別實體限制的實體，請使用此模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [最終一致的交易模式](#eventually-consistent-transactions-pattern)  
* [寬型實體模式](#wide-entities-pattern)

<a name="prepend-append-anti-pattern"></a>

## <a name="prependappend-anti-pattern"></a>在前面加上/附加反向模式
當您有大量插入作業時，請將插入作業分散到多個資料分割之間，請增加延展性。  

### <a name="context-and-problem"></a>內容和問題
在前面加上或附加實體至您已儲存的實體，通常會導致應用程式將新實體新增至資料分割序列的第一個或最後一個資料分割。 在此情況下，所有在任何給定時間的插入將會在相同的資料分割中執行，而產生使資料表服務無法將插入負載平衡到多個節點間的熱點，且可能會造成您的應用程式達到資料分割的延展性目標。 例如，如果您的應用程式會記錄員工的網路和資源存取，在交易量已達到個別資料分割的延展性目標時，則如下的實體結構可能會導致目前小時的資料分割成為熱點：  

![實體結構](media/storage-table-design-guide/storage-table-design-IMAGE26.png)

### <a name="solution"></a>解決方法
下列替代實體結構可在應用程式記錄事件時避免在任何特定資料分割上產生熱點：  

![替代的實體結構](media/storage-table-design-guide/storage-table-design-IMAGE27.png)

請留意此範例中的 **PartitionKey** 和 **RowKey** 如何成為複合索引鍵。 **PartitionKey** 會同時使用部門和員工識別碼將記錄散發到多個資料分割。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何實作此模式時，請考慮下列幾點：  

* 可避免在插入時產生熱點資料分割的替代索引鍵結構，是否可有效支援用戶端應用程式發出的查詢？  
* 您預期的交易量是否表示您很可能會達到個別資料分割的延展性目標，而遭到儲存體服務的節流？  

### <a name="when-to-use-this-pattern"></a>使用此模式的時機
如果您的交易量在您存取熱點資料分割時很可能會導致儲存體服務執行節流，請避免在前面加上/附加反向模式。  

### <a name="related-patterns-and-guidance"></a>相關的模式和指導方針
在實作此模式時，下列模式和指導方針也可能有所關聯：  

* [複合索引鍵模式](#compound-key-pattern)  
* [記錄檔結尾模式](#log-tail-pattern)  
* [修改實體](#modifying-entities)  

## <a name="log-data-anti-pattern"></a>記錄資料反向模式
一般而言，您應該使用 Blob 服務來儲存記錄資料，而不是資料表服務。  

### <a name="context-and-problem"></a>內容和問題
記錄資料的常見使用案例是要擷取特定日期/時間範圍內的選定記錄項目：例如，您想要尋找應用程式在特定日期的 15:04 到 15:06 之間記錄的所有錯誤和嚴重訊息。 您不應使用記錄訊息的日期和時間來判斷儲存記錄實體的資料分割：這會產生熱點資料分割，因為在任何給定時間，所有的記錄實體都會共用相同的 **PartitionKey** 值 (請參閱 [在前面加上/附加反向模式](#prepend-append-anti-pattern)一節)。 比方說，記錄訊息的下列實體結構描述會產生熱點資料分割，因為應用程式會將目前日期和小時的所有記錄訊息都寫入至資料分割：  

![記錄訊息實體](media/storage-table-design-guide/storage-table-design-IMAGE28.png)

在此範例中， **RowKey** 包含記錄訊息的日期和時間，以確保記錄訊息在儲存時會以日期/時間順序排序，且包含訊息識別碼，以便在有多個記錄訊息共用相同日期和時間時參考。  

另一種方法是使用 **PartitionKey** ，以確保應用程式將訊息寫入一個範圍內的資料分割。 例如，如果記錄訊息的來源可藉由某方式將訊息散發到許多資料分割，您可以使用下列實體結構描述：  

![替代的記錄訊息實體](media/storage-table-design-guide/storage-table-design-IMAGE29.png)

不過，此結構描述的問題是，若要擷取特定時間範圍內的所有記錄訊息，您必須在資料表中搜尋每個資料分割。

### <a name="solution"></a>解決方法
上一節加強說明了嘗試使用資料表服務來儲存記錄項目的問題，並提供了兩個無法令人滿意的設計。 一個方案會導致熱點資料分割，且具有寫入記錄檔訊息效能不佳的風險；另一個方案會導致查詢效能不佳，因為必須要掃描資料表中的每個資料分割，才能擷取特定時間範圍內的記錄訊息。 Blob 儲存體可為這種類型的案例提供更好的方案，Azure Storage Analytics 就是以此方式來儲存它所收集到的記錄資料。  

本節概述 Storage Analytics 將記錄資料儲存在 Blob 儲存體中的方式，以說明如何以此方法儲存您常會依範圍查詢的資料。  

Storage Analytics 會以分隔格式將記錄訊息儲存在多個 Blob 中。 分隔格式可方便用戶端應用程式剖析記錄訊息中的資料。  

Storage Analytics 會為 Blob 使用命名慣例，讓您找出含有您要搜尋之記錄訊息的一或多個 Blob。 例如，名為 "queue/2014/07/31/1800/000001.log" 的 Blob，會包含與始於 2014 年 7 月 31 日 18:00 的佇列服務有關的記錄訊息。 "000001"表示這是這段期間的第一個記錄檔。 儲存體分析也會記錄在檔案中儲存為 Blob 中繼資料的第一個和最後一個記錄訊息的時間戳記。 Blob 儲存體的 API 可讓您根據名稱前置詞找出容器中的 Blob：若要尋找所有內含從 18:00 開始之佇列記錄資料的 Blob，您可以使用前置詞 "queue/2014/07/31/1800"。  

Storage Analytics 會在內部緩衝處理記錄訊息，然後定期更新適當的 Blob，或建立新的 Blob 來容納記錄項目的最新批次。 這會減少它必須對 Blob 服務執行的寫入數目。  

如果您在自己的應用程式中實作類似的方案，您必須考量如何管理可靠性 (在每個記錄項目發生時將其寫入至 Blob 儲存體)、成本和延展性 (在您的應用程式中緩衝處理更新，並將其分批寫入至 Blob 儲存體) 之間的取捨。  

### <a name="issues-and-considerations"></a>問題和考量
當您決定如何儲存記錄資料時，請考慮下列幾點：  

* 如果您建立的資料表設計可避免產生潛在的熱點資料分割，您可能會發現您無法有效率地存取記錄資料。  
* 在處理記錄資料時，用戶端常需要載入多筆記錄。  
* 雖然記錄資料通常是結構化的，Blob 儲存體會是較佳的方案。  

## <a name="implementation-considerations"></a>實作考量
本節討論您在實作前面幾節說明的模式時應謹記在心的注意事項。 本節中的範例大多是以 C# 撰寫，並使用儲存體用戶端程式庫 (撰寫本文時為 4.3.0 版)。  

## <a name="retrieving-entities"></a>擷取實體
如 [查詢的設計](#design-for-querying)一節中所述，最有效率的查詢是點查詢。 但在某些情況下，您可能需要擷取多個實體。 本節說明使用儲存體用戶端程式庫來擷取實體的一些常見方法。  

### <a name="executing-a-point-query-using-the-storage-client-library"></a>使用儲存體用戶端程式庫執行點查詢
要執行點查詢，最簡單的方式是使用下列 C# 程式碼片段中展示的**擷取**資料表作業，來擷取 **PartitionKey** 值為 "Sales" 和 **RowKey** 值為 "212" 的實體：  

```csharp
TableOperation retrieveOperation = TableOperation.Retrieve<EmployeeEntity>("Sales", "212");
var retrieveResult = employeeTable.Execute(retrieveOperation);
if (retrieveResult.Result != null)
{
    EmployeeEntity employee = (EmployeeEntity)retrieveResult.Result;
    ...
}  
```

請注意，此範例預期會擷取的實體屬於 **EmployeeEntity**類型。  

### <a name="retrieving-multiple-entities-using-linq"></a>使用 LINQ 擷取多個實體
您可以搭配使用 LINQ 與儲存體用戶端程式庫，並指定具有 **where** 子句的查詢，以擷取多個實體。 若要避免資料表掃描，您應一律在 where 子句中加入 **PartitionKey** 值，並盡可能加入 **RowKey** 值，以防止資料表和資料分割掃描。 資料表服務支援在 where 子句中使用一組有限的比較運算子 (大於、大於或等於、小於、小於或等於、等於和不等於)。 下列 C# 程式碼片段會在業務部門 (假設 **PartitionKey** 儲存部門名稱) 中，尋找姓氏以 "B" 開頭的所有員工 (假設 **RowKey** 儲存姓氏)：  

```csharp
TableQuery<EmployeeEntity> employeeQuery = employeeTable.CreateQuery<EmployeeEntity>();
var query = (from employee in employeeQuery
            where employee.PartitionKey == "Sales" &&
            employee.RowKey.CompareTo("B") >= 0 &&
            employee.RowKey.CompareTo("C") < 0
            select employee).AsTableQuery();
var employees = query.Execute();  
```

請注意查詢如何同時指定 **RowKey** 和 **PartitionKey** 以確保更好的效能。  

下列程式碼範例說明使用 Fluent API 的對等功能 (如需 Fluent API 的詳細資訊，請參閱 [設計 Fluent API 的最佳做法](http://visualstudiomagazine.com/articles/2013/12/01/best-practices-for-designing-a-fluent-api.aspx))：  

```csharp
TableQuery<EmployeeEntity> employeeQuery = new TableQuery<EmployeeEntity>().Where(
    TableQuery.CombineFilters(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition(
    "PartitionKey", QueryComparisons.Equal, "Sales"),
    TableOperators.And,
    TableQuery.GenerateFilterCondition(
    "RowKey", QueryComparisons.GreaterThanOrEqual, "B")
),
TableOperators.And,
TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "C")
    )
);
var employees = employeeTable.ExecuteQuery(employeeQuery);  
```

> [!NOTE]
> 此範例內嵌了多個 **CombineFilters** 方法，以納入三個篩選條件。  
> 
> 

### <a name="retrieving-large-numbers-of-entities-from-a-query"></a>從查詢擷取大量實體
最佳查詢會根據 **PartitionKey** 值和 **RowKey** 值傳回個別實體。 但在某些情況下，您可能需要從相同的磁碟分割甚或多個資料分割傳回許多實體。  

在此類情況下，您務必要完整測試應用程式的效能。  

對資料表服務的查詢一次最多可傳回 1000 個實體，且最長可執行五秒。 如果結果集包含超過 1000 個實體，且查詢未於五秒內完成，或者查詢跨越資料分割界限，則資料表服務會傳回接續權杖，讓用戶端應用程式能夠要求下一組實體。 如需接續權杖如何運作的詳細資訊，請參閱 [查詢逾時和分頁](http://msdn.microsoft.com/library/azure/dd135718.aspx)。  

如果您使用儲存體用戶端程式庫，它可以在從資料表服務傳回實體時，自動為您處理接續權杖。 下列使用儲存體用戶端程式庫的 C# 程式碼範例會在資料表服務於回應中傳回接續權杖時自動處理接續權杖：  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

var employees = employeeTable.ExecuteQuery(employeeQuery);
foreach (var emp in employees)
{
        ...
}  
```

下列 C# 程式碼會明確處理接續權杖：  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

TableContinuationToken continuationToken = null;

do
{
        var employees = employeeTable.ExecuteQuerySegmented(
        employeeQuery, continuationToken);
    foreach (var emp in employees)
    {
    ...
    }
    continuationToken = employees.ContinuationToken;
} while (continuationToken != null);  
```

藉由明確使用接續權杖，您將可控制應用程式何時會擷取下一個資料區段。 例如，如果用戶端應用程式可讓使用者逐頁查看儲存在資料表中的實體，使用者可以決定不逐頁查看查詢所擷取的所有實體，而在使用者逐頁查看完目前區段中的所有實體時，您的應用程式才會使用接續權杖擷取下一個區段。 這種方法有幾項優點：  

* 它可讓您限制要從資料表服務擷取的資料量，以及在網路上移動的資料量。  
* 它可讓您在 .NET 中執行非同步 IO。  
* 它可讓您序列化永續性儲存體的接續權杖，以便在應用程式當機時繼續作業。  

> [!NOTE]
> 接續權杖通常會傳回包含 1000 個實體的區段，但也可能較少。 如果您使用 **Take** 傳回符合查閱準則的前 n 個實體，以限制查詢可傳回的實體數，也是如此：表格服務可能會傳回包含少於 n 個實體的區段以及接續權杖，讓您能夠擷取剩餘的實體。  
> 
> 

下列 C# 程式碼說明如何修改區段內傳回的實體數目：  

```csharp
employeeQuery.TakeCount = 50;  
```

### <a name="server-side-projection"></a>伺服器端預測
單一實體最多可以有 255 個屬性，且大小上限為 1 MB。 當您查詢資料表並擷取實體時，您可能不需要所有屬性，並可以避免不必要地傳送資料 (以利降低延遲和成本)。 您可以使用伺服器端預測，僅傳送您需要的屬性。 下列範例將只會從查詢選取的實體中擷取 **Email** 屬性 (連同 **PartitionKey**、**RowKey**、**Timestamp** 和 **ETag**)。  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
List<string> columns = new List<string>() { "Email" };
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter).Select(columns);

var entities = employeeTable.ExecuteQuery(employeeQuery);
foreach (var e in entities)
{
        Console.WriteLine("RowKey: {0}, EmployeeEmail: {1}", e.RowKey, e.Email);
}  
```

請注意， **RowKey** 值即使未包含在要擷取的屬性清單中，也可供使用。  

## <a name="modifying-entities"></a>修改實體
儲存體用戶端程式庫可讓您藉由插入、刪除和更新實體，修改您儲存在資料表服務中的實體。 您可以使用 EGT 一併批次處理多個插入、更新和刪除作業，以減少所需的往返次數，並改善方案的效能。  

請注意，當儲存體用戶端程式庫執行 EGT 時，通常預期會包含造成批次失敗的實體索引。 當您偵錯使用 EGT 的程式碼時，這會很有幫助。  

您也應該考量您的設計會如何影響用戶端應用程式處理並行存取和更新作業的方式。  

### <a name="managing-concurrency"></a>管理並行存取
根據預設，表格服務會在個別實體的層級上，針對**插入**、**合併**和**刪除**作業實作開放式並行存取檢查，雖然用戶端也可以強制表格服務略過這些檢查。 如需表格服務如何管理並行存取的詳細資訊，請參閱[管理 Microsoft Azure 儲存體中的並行存取](../../storage/common/storage-concurrency.md)。  

### <a name="merge-or-replace"></a>合併或取代
**TableOperation** 類別的 **Replace** 方法一律會取代表格服務中的完整實體。 如果您未在要求中包含某個屬性，而該屬性存在於已儲存的實體中，要求將會從已儲存的實體中移除該屬性。 除非您想要明確地從已儲存的實體中移除屬性，否則即必須在要求中包含每個屬性。  

您可以使用 **TableOperation** 類別的 **Merge** 方法，減少您想要更新實體時傳送至表格服務的資料量。 **Merge** 方法會將已儲存實體中的任何屬性取代為要求中包含之實體的屬性值，但對於未包含在要求中的已儲存實體，則會完整保留其所有的屬性。 如果您有大型實體，而只需要更新要求中的少數屬性，這就非常有用。  

> [!NOTE]
> 如果實體不存在，**Replace** 和 **Merge** 方法將會失敗。 如果實體不存在，您可以改用 **InsertOrReplace** 和 **InsertOrMerge** 方法建立新實體。  
> 
> 

## <a name="working-with-heterogeneous-entity-types"></a>使用異質性實體類型
表格服務是 *無結構描述* 資料表存放區，這表示單一資料表可以儲存多種類型的實體，並在您的設計中提供絕佳的彈性。 下列範例說明用以儲存員工和部門實體的資料表：  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

請注意，每個實體仍必須要有 **PartitionKey**、**RowKey** 和 **Timestamp** 值，但是可以有任何屬性集。 此外，除非您選擇將實體類型資訊儲存在某處，否則將沒有項目會指出此類型。 有兩個選項可用來識別實體類型：  

* 在 **RowKey** (或可能是 **PartitionKey**) 前面加上實體類型。 例如 **EMPLOYEE_000123**，或以 **DEPARTMENT_SALES** 做為 **RowKey** 值。  
* 使用個別屬性記錄實體類型，如下表所示。  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td>員工</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td>員工</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>department</td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>名字</th>
<th>姓氏</th>
<th>年齡</th>
<th>電子郵件</th>
</tr>
<tr>
<td>員工</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

如果兩個不同類型的實體可能有相同的索引鍵值，第一個選項 (在 **RowKey**前面加上實體類型) 可能會很有用。 它也會將屬於相同類型的實體分組在資料分割中。  

本節討論的方法與本指南稍早的[模型化關聯性](table-storage-design-modeling.md)一文，對於[繼承關聯性](table-storage-design-modeling.md#inheritance-relationships)的討論有密切關聯。  

> [!NOTE]
> 您應考慮在實體類型值中包含版本號碼，讓用戶端應用程式能夠演化 POCO 物件，並與不同版本搭配使用。  
> 
> 

本節的其餘部分說明儲存體用戶端程式庫中一些有助於在相同資料表中使用多個實體類型的功能。  

### <a name="retrieving-heterogeneous-entity-types"></a>擷取異質性實體類型
在使用儲存體用戶端程式庫時，有三個選項可供您使用多個實體類型。  

如果您知道以特定**RowKey** 和 **PartitionKey** 值儲存的實體類型，則在您擷取實體 (如先前兩個擷取實體類型 **EmployeeEntity** 的範例所說明) 時，將可指定實體類型：[使用儲存體用戶端程式庫執行點查詢](#executing-a-point-query-using-the-storage-client-library)和[使用 LINQ 擷取多個實體](#retrieving-multiple-entities-using-linq)。  

第二個選項是使用 **DynamicTableEntity** 類型 (屬性包)，而不是具體的 POCO 實體類型 (這個選項也可改善效能，因為不需要將實體序列化和還原序列化成 .NET 類型)。 下列 C# 程式碼可能會從資料表中擷取多個不同類型的實體，但會傳回所有實體做為 **DynamicTableEntity** 執行個體。 然後，它會使用 **EntityType** 屬性來判斷每個實體的類型：  

```csharp
string filter = TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("PartitionKey",
    QueryComparisons.Equal, "Sales"),
    TableOperators.And,
    TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("RowKey",
                    QueryComparisons.GreaterThanOrEqual, "B"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey",
        QueryComparisons.LessThan, "F")
    )
);
TableQuery<DynamicTableEntity> entityQuery =
    new TableQuery<DynamicTableEntity>().Where(filter);
var employees = employeeTable.ExecuteQuery(entityQuery);

IEnumerable<DynamicTableEntity> entities = employeeTable.ExecuteQuery(entityQuery);
foreach (var e in entities)
{
EntityProperty entityTypeProperty;
if (e.Properties.TryGetValue("EntityType", out entityTypeProperty))
{
    if (entityTypeProperty.StringValue == "Employee")
    {
        // Use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
        }
    }
}  
```

請注意，您必須在 **DynamicTableEntity** 類別的 **Properties** 屬性上使用 **TryGetValue** 方法，才能擷取其他屬性。  

第三個選項是使用 **DynamicTableEntity** 類型和 **EntityResolver** 執行個體進行結合。 這可讓您解析為相同查詢中的多個 POCO 類型。 在此範例中，**EntityResolver** 委派會使用 **EntityType** 屬性來區別查詢傳回的兩個實體類型。 **Resolve** 方法會使用 **resolver** 委派，將 **DynamicTableEntity** 執行個體解析為 **TableEntity** 執行個體。  

```csharp
EntityResolver<TableEntity> resolver = (pk, rk, ts, props, etag) =>
{

        TableEntity resolvedEntity = null;
        if (props["EntityType"].StringValue == "Department")
        {
        resolvedEntity = new DepartmentEntity();
        }
        else if (props["EntityType"].StringValue == "Employee")
        {
        resolvedEntity = new EmployeeEntity();
        }
        else throw new ArgumentException("Unrecognized entity", "props");

        resolvedEntity.PartitionKey = pk;
        resolvedEntity.RowKey = rk;
        resolvedEntity.Timestamp = ts;
        resolvedEntity.ETag = etag;
        resolvedEntity.ReadEntity(props, null);
        return resolvedEntity;
};

string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<DynamicTableEntity> entityQuery =
        new TableQuery<DynamicTableEntity>().Where(filter);

var entities = employeeTable.ExecuteQuery(entityQuery, resolver);
foreach (var e in entities)
{
        if (e is DepartmentEntity)
        {
    ...
        }
        if (e is EmployeeEntity)
        {
    ...
        }
}  
```

### <a name="modifying-heterogeneous-entity-types"></a>修改異質性實體類型
您不需要知道實體的類型即可加以刪除，而在您插入實體時一定會知道其類型。 不過，您可以使用 **DynamicTableEntity** 類型來更新實體，而不需要知道其類型，且無須使用 POCO 實體類別。 下列程式碼範例會擷取單一實體，並確認 **EmployeeCount** 屬性存在，然後加以更新。  

```csharp
TableResult result =
        employeeTable.Execute(TableOperation.Retrieve(partitionKey, rowKey));
DynamicTableEntity department = (DynamicTableEntity)result.Result;

EntityProperty countProperty;

if (!department.Properties.TryGetValue("EmployeeCount", out countProperty))
{
        throw new
        InvalidOperationException("Invalid entity, EmployeeCount property not found.");
}
countProperty.Int32Value += 1;
employeeTable.Execute(TableOperation.Merge(department));  
```

## <a name="controlling-access-with-shared-access-signatures"></a>使用共用存取簽章控制存取
您可以使用共用存取簽章 (SAS) 權杖，讓用戶端應用程式能夠直接修改 (和查詢) 資料表實體，而不需要在您的程式碼中納入儲存體帳戶金鑰。 一般而言，在您的應用程式中使用 SAS 有三大優點：  

* 您不需要為了要讓裝置可存取和修改資料表服務中的實體，而將儲存體帳戶金鑰散發至不安全的平台 (例如行動裝置)。  
* 您可以將 Web 和背景工作角色為了管理實體而執行的某些工作，卸載至使用者電腦和行動裝置之類的用戶端裝置。  
* 您可以為用戶端指派受條件約束和時間限制的權限集 (例如允許對特定資源的唯讀存取)。  

如需搭配使用 SAS 權杖與資料表服務的詳細資訊，請參閱 [使用共用存取簽章 (SAS)](../../storage/common/storage-dotnet-shared-access-signature-part-1.md)。  

不過，您仍必須產生SAS 權杖，讓用戶端應用程式有權使用資料表服務中的實體：您應在可安全存取儲存體帳戶金鑰的環境中執行這個動作。 一般而言，您可以使用 Web 或背景工作角色來產生 SAS 權杖，並將其傳送至需要存取您的實體的用戶端應用程式。 由於產生 SAS 權杖並將其傳遞至用戶端仍會產生額外負荷，因此您應考量怎樣最能降低此負荷，尤其是在大量的案例中。  

您可以產生特定 SAS 權杖，使其授與對資料表中的實體子集進行存取的權限。 根據預設，您會建立適用於整個資料表的 SAS 權杖，但也可以指定 SAS 權杖僅授與存取特定範圍的 **PartitionKey** 值或特定範圍的 **PartitionKey** 和 **RowKey** 值的權限。 您可以選擇為系統的個別使用者產生 SAS 權杖，使每位使用者的 SAS 權杖只允許他們在資料表服務中存取自己的實體。  

## <a name="asynchronous-and-parallel-operations"></a>非同步和平行作業
假設您要跨多個資料分割分散您的要求，您可以使用非同步或平行查詢來改善輸送量和用戶端的回應性。
例如，您可以用兩個或更多背景工作角色執行個體，以平行方式存取您的資料表。 您可以讓個別的背景工作角色負責特定的資料分割集，或僅使用多個背景工作角色執行個體，使其分別都能夠存取資料表中的所有資料分割。  

在用戶端執行個體中，您可以藉由以非同步方式執行儲存作業來提高輸送量。 儲存體用戶端程式庫可讓您輕鬆撰寫非同步查詢並修改。 比方說，首先您可以使用同步方法來擷取資料分割中的所有實體，如下列 C# 程式碼所示：  

```csharp
private static void ManyEntitiesQuery(CloudTable employeeTable, string department)
{
        string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

        TableContinuationToken continuationToken = null;

        do
        {
        var employees = employeeTable.ExecuteQuerySegmented(
                employeeQuery, continuationToken);
        foreach (var emp in employees)
    {
        ...
    }
        continuationToken = employees.ContinuationToken;
        } while (continuationToken != null);
}  
```

您可以輕鬆地修改此程式碼，讓查詢以非同步方式執行，如下所示：  

```csharp
private static async Task ManyEntitiesQueryAsync(CloudTable employeeTable, string department)
{
        string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);
        TableContinuationToken continuationToken = null;

        do
        {
        var employees = await employeeTable.ExecuteQuerySegmentedAsync(
                employeeQuery, continuationToken);
        foreach (var emp in employees)
        {
            ...
        }
        continuationToken = employees.ContinuationToken;
            } while (continuationToken != null);
}  
```

在此非同步範例中，您可以看到下列與同步版本不同的變更：  

* 方法簽章現在包含 **async** 修飾詞，並且會傳回 **Task** 執行個體。  
* 現在方法會呼叫 **ExecuteSegmentedAsync** 方法，並使用 **await** 修飾詞以非同步方式擷取結果，而不會呼叫 **ExecuteSegmented** 方法來擷取結果。  

用戶端應用程式可以呼叫此方法多次 (使用不同的 **department** 參數值)，且每個查詢將會在個別的執行緒上執行。  

請注意，**TableQuery** 類別中的 **Execute** 方法並沒有非同步版本，因為 **IEnumerable** 介面不支援非同步列舉。  

您也可以透過非同步方式插入、更新和刪除實體。 下列 C# 範例說明如何以簡單的同步方法來插入或取代員工實體：  

```csharp
private static void SimpleEmployeeUpsert(CloudTable employeeTable,
        EmployeeEntity employee)
{
        TableResult result = employeeTable
        .Execute(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
}  
```

您可以輕鬆地修改此程式碼，讓更新以非同步方式執行，如下所示：  

```csharp
private static async Task SimpleEmployeeUpsertAsync(CloudTable employeeTable,
        EmployeeEntity employee)
{
        TableResult result = await employeeTable
        .ExecuteAsync(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
}  
```

在此非同步範例中，您可以看到下列與同步版本不同的變更：  

* 方法簽章現在包含 **async** 修飾詞，並且會傳回 **Task** 執行個體。  
* 現在方法會呼叫 **ExecuteAsync** 方法，並使用 **await** 修飾詞以非同步方式擷取結果，而不會呼叫 **Execute** 方法來更新實體。  

用戶端應用程式可以呼叫多個此類的非同步方法，每個方法叫用會在個別的執行緒上執行。  

## <a name="next-steps"></a>後續步驟

- [將關聯性模型化](table-storage-design-modeling.md)
- [查詢的設計](table-storage-design-for-query.md)
- [加密資料表的資料](table-storage-design-encrypt-data.md)
- [資料修改的設計](table-storage-design-for-modification.md)