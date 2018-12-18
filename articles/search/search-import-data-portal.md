---
title: 在入口網站中將資料匯入 Azure 搜尋服務 | Microsoft Docs
description: 了解如何在 Azure 入口網站中使用匯入資料精靈，以從 Cosmos DB、Blob 儲存體、表格儲存體、SQL Database 和 Azure VM 上的 SQL Server 對 Azure 資料進行編目。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 07/10/2018
ms.author: heidist
ms.openlocfilehash: dcdc0501d94191cf2c281a4f880ddab3db023fc0
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39004933"
---
# <a name="how-to-import-data-into-azure-search-index-using-the-azure-portal"></a>如何使用 Azure 入口網站將資料匯入 Azure 搜尋服務索引

Azure 入口網站在 Azure 搜尋服務儀表板上提供 [匯入資料] 精靈來將資料載入至索引。 

  ![命令列上的 [匯入資料]][1]

就內部而言，精靈會設定並叫用 *索引子*，將編製索引程序的幾個步驟自動化︰ 

* 從相同的 Azure 訂用帳戶連接至外部資料來源
* 根據來源資料結構產生可修改的索引結構描述
* 使用從資料來源擷取到的資料列集將 JSON 文件載入索引中

> [!NOTE]
> 您可以從 Azure Cosmos DB 儀表板啟動**匯入資料**精靈，以簡化該資料來源的索引建立作業。 在左側導覽中，移至 [集合] > [新增 Azure 搜尋服務] 以便開始使用。

## <a name="data-sources-supported-by-the-import-data-wizard"></a>匯入資料精靈所支援的資料來源
匯入資料精靈支援下列資料來源︰ 

* 連接字串
* Azure VM 上的 SQL Server 關聯式資料
* Azure Cosmos DB
* Azure Blob 儲存體
* Azure 資料表儲存體

必須輸入扁平化資料集。 您可以只從單一資料表、資料庫檢視或對等的資料結構匯入。 您應在執行精靈之前建立此資料結構。

## <a name="connect-to-your-data"></a>連接到您的資料
1. 登入 [Azure 入口網站](https://portal.azure.com)並開啟服務儀表板。 您可以按一下導向列中的 [所有服務] 以搜尋目前訂用帳戶中的現有「搜尋服務」。 

1. 按一下命令列上的 [匯入資料]  即可滑動開啟 [匯入資料] 刀鋒視窗。

1. 按一下 [連接到您的資料]  可指定索引子所使用的資料來源定義。 對於訂用帳戶內的資料來源，精靈通常可以偵測並讀取連線資訊，將整體的設定需求降至最低。

|  |  |
| --- | --- |
| **現有的資料來源** |如果您的搜尋服務中已經定義索引子，您可以針對另一次匯入選取現有的資料來源定義。 |
| **Azure SQL Database** |您可以在頁面上或透過 ADO.NET 連接字串指定服務名稱、具有讀取權限的資料庫使用者認證以及資料庫名稱。 選擇連接字串選項以檢視或自訂屬性。 <br/><br/>提供資料列集的資料表或檢視必須在頁面上指定。 連線成功後會出現此選項，並透過下拉式清單供您選取。 |
| **在 Azure VM 上的 SQL Server** |指定完整服務名稱、使用者識別碼與密碼以及資料庫作為連接字串。 若要使用此資料來源，您必須先前就已在本機存放區中安裝用來加密連線的憑證。 如需指示，請參閱 [SQL VM 到 Azure 搜尋服務的連線](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md)。 <br/><br/>提供資料列集的資料表或檢視必須在頁面上指定。 連線成功後會出現此選項，並透過下拉式清單供您選取。 |
| **Azure Cosmos DB** |需求包括帳戶、資料庫和集合。 集合中的所有文件將會包含在索引中。 您可以定義查詢以壓平合併或篩選資料列集，或者偵測已變更的文件以用於後續資料重新整理作業。 |
| **Azure Blob 儲存體** |需求包括儲存體帳戶和容器。 如果 Blob 名稱為了分組而遵循虛擬命名慣例，您可以指定名稱的虛擬目錄部分做為容器下的資料夾 (選擇性)。 如需詳細資訊，請參閱[為 Blob 儲存體編製索引](search-howto-indexing-azure-blob-storage.md)。 |
| **Azure 資料表儲存體** |需求包括儲存體帳戶和資料表名稱。 您可以指定要擷取資料表子集的查詢 (選擇性)。 如需詳細資訊，請參閱[為表格儲存體編製索引](search-howto-indexing-azure-tables.md)。 |

## <a name="customize-target-index"></a>自訂目標索引
初步的索引通常是由資料集推斷而來。 新增、編輯或刪除欄位以完成結構描述。 此外，在欄位層級設定屬性可決定其後續的搜尋行為。

1. 在 [自訂目標索引] 中，指定名稱和用來對每個文件進行唯一識別的**金鑰**。 金鑰必須是字串。 如果欄位值包含空格或連字號，請務必在 [匯入資料]  中設定進階選項以隱藏這些字元的驗證檢查。

1. 檢閱並修改其餘的欄位。 通常會為您填入欄位名稱和類型。 在索引建立完成之前，您都可以變更資料類型。 之後若要變更，就需要重建。

1. 設定每個欄位的索引屬性：
   
   * Retrievable 會在搜尋結果中傳回欄位。
   * Filterable 允許在篩選運算式中參考欄位。
   * Sortable 允許欄位用於排序。
   * Facetable 可讓欄位進行多面向導覽。
   * Searchable 能夠進行全文檢索搜尋。

1. 如果您要在欄位層級指定語言分析器，請按一下 [分析器]  索引標籤。 此時只能指定語言分析器。 使用自訂分析器或非語言分析器 (如關鍵字、模式等等) 需要有程式碼。
   
   * 按一下 [Searchable]  在欄位上指定全文檢索搜尋，並啟用分析器下拉式清單。
   * 選擇您要的分析器。 如需詳細資訊，請參閱[以多種語言建立文件的索引](search-language-support.md)。

1. 按一下 [建議工具]  以在選取的欄位上啟用預先輸入的查詢建議。

## <a name="import-your-data"></a>匯入資料
1. 在 [匯入資料] 中提供索引子名稱。 請記得，[匯入資料] 精靈的產出是索引子。 稍後如果您想要加以檢視或編輯，請從入口網站來選取，而不是重新執行精靈。 

1. 指定排程，排程是依據服務佈建所在的區域時區。

1. 設定進階選項，以指定文件卸除時是否可以繼續編製索引的臨界值。 此外，您可以指定 **金鑰** 欄位是否允許包含空格和斜線。  

1. 按一下 [確定] 以建立索引並匯入資料。

您可以在入口網站中監視索引。 隨著文件的載入，您所定義之索引的文件計數將會增加。 有時候入口網站頁面需要幾分鐘的時間來取得最近期的更新。

所有文件載入後，索引便立即可供查詢。

## <a name="query-an-index-using-search-explorer"></a>使用搜尋總管查詢索引

入口網站包含 [搜尋總管]，可供您查詢索引，而不需撰寫任何程式碼。 您可以將[搜尋總管](search-explorer.md)用在任何索引上。

搜尋體驗是以預設設定為基礎，例如[簡單語法](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search)和預設 [searchMode 查詢參數](https://docs.microsoft.com/rest/api/searchservice/search-documents)。 

結果會以 JSON 詳細資訊格式傳回，以便您檢查整份文件。

## <a name="edit-an-existing-indexer"></a>編輯現有的索引子
如前所述，匯入資料精靈會建立**索引子**，您可以在入口網站中以獨立建構的形式進行修改。

在服務儀表板中，按兩下 [索引子] 圖格即可顯示針對您訂用帳戶所建立的所有索引子清單。 按兩下其中一個索引子，加以執行、編輯或刪除。 您可以將索引替換為另一個現有索引、變更資料來源，以及設定編製索引期間的錯誤臨界值選項。

## <a name="edit-an-existing-index"></a>編輯現有索引
精靈也會建立**索引**。 在 Azure 搜尋服務中，若要更新索引的結構，就必須重建該索引。 重新時必須刪除索引，使用內含所需變更的已修改結構描述建立索引，然後重新載入資料。 結構化更新包含變更資料類型和重新命名或刪除欄位。

不需要重建的編輯作業包含新增欄位、變更評分設定檔、變更建議，或變更語言分析器。 如需詳細資訊，請參閱 [更新索引](https://msdn.microsoft.com/library/azure/dn800964.aspx) 。


## <a name="next-steps"></a>後續步驟
檢閱下列連結以深入了解索引子：

* [為 Azure SQL Database 編製索引](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [Azure Cosmos DB 編製索引](search-howto-index-cosmosdb.md)
* [為 Blob 儲存體編製索引](search-howto-indexing-azure-blob-storage.md)
* [為表格儲存體編製索引](search-howto-indexing-azure-tables.md)

<!--Image references-->
[1]: ./media/search-import-data-portal/search-import-data-command.png

