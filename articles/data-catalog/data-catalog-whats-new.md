---
title: Azure 資料目錄的新功能
description: 本文提供新增至 Azure 資料目錄之新功能的概觀。
services: data-catalog
author: steelanddata
ms.author: maroche
ms.assetid: 1201f8d4-6f26-4182-af3f-91e758a12303
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 61f837fbe7f53cdf9f313e5498c6f99ea14203a7
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047000"
---
# <a name="whats-new-in-azure-data-catalog"></a>Azure 資料目錄的新功能
**Azure 資料目錄** 的更新會定期發行。 並非所有發行版本都會包含新的使用者對應功能，某些版本會著重在後端服務功能。 本頁特別強調已加入 Azure 資料目錄服務的新使用者對應功能。

## <a name="whats-new-for-november-2017"></a>2017 年 11 月的新功能 
自 2017 年 11 月起，Azure 資料目錄已新增下列功能：

* 支援直接連結到資料目錄入口網站中的特定商務詞彙字詞。 使用者可以從商務詞彙複製連結，並將其內嵌在文件、電子郵件、報告或其他位置中，以便直接連結至詞彙的字詞定義。
* 支援 Azure Active Directory 服務主體。 資料目錄管理員可以授權用戶端應用程式使用服務主體存取類別目錄，並可以授與這些應用程式特定的權限，就像他們可以授與權限給使用者和安全性群組一樣。 如需詳細資訊，請參閱 [Azure Active Directory 中的應用程式和服務主體物件](../active-directory/develop/app-objects-and-service-principals.md)。
* 當使用資料目錄的資料來源註冊工具連線到 Azure SQL Database 和 Azure SQL 資料倉儲資料來源時，支援 Azure Active Directory 驗證。 如需詳細資訊，請參閱[利用 SQL Database 和 SQL 資料倉儲使用 Azure Active Directory 驗證來驗證](../sql-database/sql-database-aad-authentication.md)。


## <a name="whats-new-for-september-2017"></a>2017 年 9 月的新功能 
Azure 資料目錄於 2017 年 9 月新增了下列功能：

* 支援在使用資料來源註冊工具註冊相關的資料表時，擷取 DB2 資料來源中的聯結關聯性中繼資料。
* 支援使用資料來源註冊工具註冊 MongoDB 3.4 版的資料來源。
* 支援在移除資料庫或資料目錄中的其他容器時，刪除單一作業中之物件的所有中繼資料。
* 支援在資料目錄入口網站中縮小搜尋範圍時，檢視最多 1000 個標籤、商務字彙表字詞或其他搜尋 Facet。


## <a name="whats-new-for-august-2017"></a>2017 年 8 月的新功能 
自 2017 年 8 月起，Azure 資料目錄已新增下列功能：

*   新的開發人員範例可用於建立及管理關聯性的中繼資料，方法是使用資料目錄 REST API。 將關聯性資訊匯入資料目錄中範例位於[資料目錄程式碼範例頁面](https://azure.microsoft.com/resources/samples/?service=data-catalog&sort=0)。 
* 使用資料來源註冊工具註冊相關的資料表時，支援從 Teradata 資料來源擷取聯結關聯性的中繼資料。
* 使用資料來源註冊工具註冊 SQL Server 資料來源時，支援 SQL Server 資料表值函式 (TVF) 物件。
* 多個更新和細分可提高資料目錄入口網站的效能和可用性。

## <a name="whats-new-for-july-2017"></a>2017 年 7 月的新功能 
自 2017 年 7 月起，Azure 資料目錄已新增下列功能：
*   對更精確控制允許之中繼資料作業的支援包括：
    - 目錄系統管理員可以限制使用者提供標記和相關中繼資料的能力，從而啟用目錄的唯讀存取。
    - 目錄系統管理員可以限制使用者在目錄中註冊新資料來源的能力。
    - 目錄系統管理員可以限制使用者在目錄中擁有資料資產中繼資料的能力。
    - 可以將權限授與 Azure Active Directory 安全性群組和使用者，以簡化管理的權限。
* 已註冊資料資產之間的關聯性，以及在資料目錄入口網站中探索相關資料資產的支援包括：
    - 使用資料目錄資料來源註冊工具時，從 SQL Server (包括 Azure SQL Database)、Oracle 和 MySQL 資料來源擷取關聯性中繼資料。
    - 在資料目錄入口網站中檢視資產中繼資料時，探索相關的資料資產。
    - 使用資料目錄 REST API 在資料資產之間定義、探索及管理關聯性的作業。

如需在資料目錄中管理權限的其他詳細資料，請參閱[如何保護安全存取資料目錄和資料資產](data-catalog-how-to-secure-catalog.md)。
如需資料目錄中關聯性的其他詳細資料，請參閱[如何在 Azure 資料目錄中檢視相關的資料資產](data-catalog-how-to-view-related-data-assets.md)。

## <a name="whats-new-for-june-2017"></a>2017 年 6 月的新功能 
自 2017 年 6 月起，Azure 資料目錄已新增下列功能：
*   支援 Sybase、Apache Cassandra 和 MongoDB 資料來源。 使用者現在可以註冊及探索 Cassandra 與 MongoDB 資料庫和資料表，以及 Sybase 資料庫、資料表和檢視表。

> [!NOTE]
> 當註冊的 MongoDB 和 Cassandra 資料表包括內含記錄和陣列等複雜資料類型的資料行時，將不會在新增到資料目錄的中繼資料中包含這些資料行。

## <a name="whats-new-for-may-2017"></a>2017 年 5 月的新功能 
自 2017 年 5 月起，Azure 資料目錄已新增下列功能：
*   新的開發人員範例可用於大量匯入商務詞彙。 詞彙大量匯入範例位於[資料目錄開發人員範例頁面](https://docs.microsoft.com/azure/data-catalog/data-catalog-samples)。 
*   支援在 Azure 資料目錄入口網站中編輯 ODBC 連線資訊。 資料資產擁有者和資料目錄管理員現在可以編輯已註冊之 ODBC 資料來源的連線資訊，而無需重新註冊資料來源。
*   支援商務詞彙定義和描述中的可點選 URL。 當商務詞彙的描述和定義屬性中包含 URL 時，這些 URL 將在資料目錄入口網站中顯示為超連結。
*   除了專家的電子郵件地址之外，還支援顯示專家名稱。 在資料目錄入口網站中檢視資料資產屬性中的專家時，來自 Azure Active Directory 的專家完整名稱將顯示在工具提示中。

## <a name="whats-new-for-april-2017"></a>2017 年 4 月的新功能 
自 2017 年 4 月起，Azure 資料目錄已新增下列功能：
*   支援 ODBC 資料來源。 使用者現在可以使用資料目錄的資料來源註冊工具來註冊及探索 ODBC 資料庫、資料表和檢視表。

## <a name="whats-new-for-march-2017"></a>2017 年 3 月的新功能 
自 2017 年 3 月起，Azure 資料目錄已新增下列功能：
*   支援對資料目錄管理員使用 AAD 安全性群組。
*   Azure 資料目錄目前已在新的 Azure 區域中提供。 可讓客戶佈建 Azure 資料目錄的區域除了美國東部、美國西部、西歐及澳大利亞東部、北歐及東南亞之外，還增加了美國中西部區域。 如需詳細資訊，請參閱 [Azure 地區](https://azure.microsoft.com/regions/)。

## <a name="whats-new-for-february-2017"></a>2017 年 2 月的新功能 
自 2017 年 2 月起，Azure 資料目錄已新增下列功能：
*   支援資料目錄資料來源註冊工具中的進階設定。 註冊大型資料來源時，使用者可以指定命令逾時值。
*   支援 FTP 和 PostgreSQL 資料來源。 使用者現在可以註冊及探索 FTP 檔案和目錄，以及 PostgreSQL 資料庫、資料表和檢視表。

## <a name="whats-new-for-january-2017"></a>2017 年 1 月的新功能 
自 2017 年 1 月起，Azure 資料目錄已新增下列功能：
*   Azure 資料目錄現在符合 [CSA STAR](https://www.microsoft.com/en-us/trustcenter/compliance/csa-star-certification) 標準。
*   與 [Excel 2016 和 Power Query for Excel 中的取得及轉換](https://support.office.com/article/Introduction-to-Microsoft-Power-Query-for-Excel-6E92E2F4-2079-4E1F-BAD5-89F6269CD605)整合。 Excel 使用者可以從 Excel 內部使用 Azure 資料目錄來共用查詢及探索查詢。 這項功能可供具有 Power BI Pro 授權的使用者使用。

## <a name="whats-new-for-december-2016"></a>2016 年 12 月的新功能
自 2016 年 12 月起，Azure 資料目錄已新增下列功能：
*   Azure 資料目錄現在符合 [HIPAA](https://www.microsoft.com/en-us/TrustCenter/Compliance/hipaa) 和[歐盟示範條款](https://www.microsoft.com/en-us/TrustCenter/Compliance/EU-Model-Clauses)標準。
*   支援編輯資料來源連線資訊。 資料資產擁有者和資料目錄管理員現在可以編輯已註冊之資料來源的連線資訊，而無需重新註冊資料來源。
*   支援 Salesforce.com 資料來源。 使用者現在可以註冊及探索 Salesforce 物件。


## <a name="whats-new-for-november-2016"></a>2016 年 11 月的新功能
自 2016 年 11 月起，Azure 資料目錄已新增下列功能：
*   Azure 資料目錄現在符合 [ISO/IEC 27001](https://www.microsoft.com/en-us/trustcenter/compliance/iso-iec-27001) 和 [ISO/IEC 27018](https://www.microsoft.com/en-us/TrustCenter/Compliance/iso-iec-27018) 標準。
*   支援使用資料目錄入口網站和 REST API 手動註冊 ODBC 資料來源。

## <a name="whats-new-for-september-2016"></a>2016 年 9 月的新功能
自 2016 年 9 月起，Azure 資料目錄已新增下列功能：

* 支援 IBM DB2 資料來源。 使用者現在可以註冊並探索 DB2 資料庫、資料表和檢視。
* 支援 Azure Cosmos DB 資料來源。 使用者現在可以註冊並探索 Cosmos DB 資料庫和集合。
* 支援在資料目錄入口網站中自訂目錄名稱。 目錄管理員現在可以提供顯示在入口網站標題的文字，例如組織名稱。

## <a name="whats-new-for-august-2016"></a>2016 年 8 月的新功能
自 2016 年 8 月起，Azure 資料目錄已新增下列功能：

* 增強 SQL Server Master Data Services (MDS) 資料來源的註冊功能。 使用者現在可以在使用資料目錄的資料來源註冊工具註冊 MDS 實體時，包含預覽及資料設定檔。
* 支援管理員定義的組織已儲存的搜尋。 在資料目錄入口網站中儲存搜尋時，資料目錄管理員現在可以選擇儲存搜尋以供個人使用或所有目錄使用者使用。 組織已儲存的搜尋會與所有目錄使用者共用，而且可以提供標準化的起點來探索資料來源。
* 更新資料目錄入口網站中的屬性檢視。 所有的資料資產屬性現已在單一、可調整大小的窗格中顯示和管理，以提供更一致且可探索的體驗。

## <a name="whats-new-for-july-2016"></a>2016 年 7 月的新功能
自 2016 年 7 月起，Azure 資料目錄已新增下列功能：

* 支援 SQL Server Master Data Services (MDS) 資料來源。 使用者現在可以註冊並探索 MDS 模型和實體。
* 支援 SQL Server 預存程序。 使用者現在可以註冊並探索 SQL Server 資料來源中的預存程序物件。
* Azure 資料目錄入口網站及資料來源註冊工具支援其他語言，總共支援 18 種語言。 Azure 資料目錄會根據 Windows 或網頁瀏覽器中設定的語言喜好設定，提供當地語系化的使用者體驗。
* 更新和細分資料目錄入口網站首頁，包括效能改進和簡化的使用者體驗。

## <a name="whats-new-for-june-2016"></a>2016 年 6 月的新功能
自 2016 年 6 月起，Azure 資料目錄已新增下列功能：

* 在探索資料目錄入口網站中的資料資產時，支援在清單檢視中將資料行重新調整大小。 使用者現在可以將個別資料行的大小調整為更容易閱讀的冗長資產中繼資料，例如標籤和描述。
* 資料目錄入口網站的 [開啟於] 功能表中已新增 Power Query for Excel。 使用者現在可以在安裝了 [Power Query for Excel](https://support.office.com/article/Introduction-to-Microsoft-Power-Query-for-Excel-6E92E2F4-2079-4E1F-BAD5-89F6269CD605) 增益集的 Excel 2016 或 Excel 2010 和 Excel 2013 中開啟支援的資料來源。
* 支援 Azure 表格儲存體的資料來源。 使用者現在可以註冊並探索 Azure 儲存體資料來源中的資料表物件。

## <a name="whats-new-for-may-2016"></a>2016 年 5 月的新功能
自 2016 年 5 月起，Azure 資料目錄已新增下列功能：

* 允許目錄系統管理員定義商務詞彙與階層以建立常用商務詞彙的商務詞彙。 使用者可以標記包含詞彙的已註冊資料資產，讓您更輕鬆地探索和了解目錄的內容。 如需詳細資訊，請參閱 [如何設定控管標籤的商務詞彙](data-catalog-how-to-business-glossary.md)  
* 「資料目錄商務字彙」增強功能，可讓使用者以單一作業更新多個字彙字詞。 使用者可以選取多個字詞來編輯下列欄位︰
  * 父系字詞︰使用者可以選取新的父系字詞，而所有選取的字詞都會更新為所選父系字詞的子系。 如果選取的字詞都具有相同的父系，該父系就會顯示在文字方塊中，否則父系字詞欄位會設為空白。   
  * 標記和專案關係人︰使用者可以使用與標記多個資料資產相同的體驗，來新增和移除多個字彙字詞的標記和專案關係人。

> [!NOTE]
> 商務詞彙只能在標準版 Azure 資料目錄中使用。 免費版不提供控管標記或商務詞彙的功能。

## <a name="whats-new-for-march-2016"></a>2016 年 3 月的新功能
自 2016 年 3 月起，Azure 資料目錄已新增下列功能：

* 合併的 REST API 端點，以程式設計方式存取 Azure 資料目錄服務的搜尋功能和目錄資產管理功能。 此搜尋 API 端點和目錄 API 端點已在 2016 年 3 月 21 日被取代並終止。 沒有針對 API 的語意做出任何變更。 僅變更端點 URI。 如需詳細資訊，請參閱 [Azure 資料目錄 REST API 參考](https://msdn.microsoft.com/library/azure/mt267595.aspx)。 如需 API 範例，請參閱 [Azure 資料目錄開發人員範例](data-catalog-samples.md)。

## <a name="whats-new-for-february-2016"></a>2016 年 2 月的新功能
自 2016 年 2 月起，Azure 資料目錄已新增下列功能：

* Azure 資料目錄的資料來源註冊工具會提供您最近重新設計過的資料來源選取體驗。 資料來源註冊工具已更新，讓您可以更輕鬆地找到並選取 Azure 資料目錄支援的資料來源。
* Azure 資料目錄入口網站及資料來源註冊工具支援額外 10 種語言。 除了英文之外，Azure 資料目錄現已支援德文、西班牙文、法文、義大利文、日文、韓文、葡萄牙文、俄文、簡體中文及繁體中文。 Azure 資料目錄會根據 Windows 或使用者的網頁瀏覽器中設定的語言喜好設定，提供當地語系化的使用者體驗。
* 針對商務持續性及災害復原，支援 Azure 資料目錄資料的異地複寫功能。 現在所有 Azure 資料目錄的內容 (包括資料來源元資料及群眾外包註解)，會在兩個 Azure 區域之間複寫，且客戶不需支付額外的成本。 Azure 區域是預先配對的，兩個區域至少相隔 500 英哩，且遵循 [業務持續性和災害復原 (BCDR)：Azure 配對的區域](../best-practices-availability-paired-regions.md)一文中所述的對應。
* 支援變更 Azure 資料目錄所使用的 Azure 訂用帳戶。 Azure 資料目錄的系統管理員可以使用 Azure 資料目錄入口網站中的 [設定] 頁面，針對計費功能選取不同的 Azure 訂用帳戶。

## <a name="whats-new-for-january-2016"></a>2016 年 1 月的新功能
自 2016 年 1 月起，Azure 資料目錄已新增下列功能：

* 支援手動註冊其他資料來源。 您現在可以使用 Azure 資料目錄入口網站中的 [建立手動輸入]，或使用 Azure 資料目錄 REST API 來為下列資料來源註冊：
  * OData - 函式、實體集和實體容器
  * HTTP - 檔案、端點、報告和網站
  * 檔案系統 - 檔案
  * SharePoint - 清單
  * FTP - 檔案和目錄
  * Salesforce.com - 物件
  * DB2 - 資料表、檢視和資料庫
  * PostgreSQL - 資料表、檢視和資料庫
* 針對 SQL Server (包含 Azure SQL DB 和 Azure SQL 資料倉儲) 資料來源支援「在 SQL Server Data Tools 中開啟」。  
* 支援註冊與探索 SAP HANA 檢視和封裝。 您可以利用 Azure 資料目錄資料來源註冊工具，為 SAP HANA 資料來源註冊，並且可以利用 Azure 資料目錄入口網站，對已註冊的 SAP HANA 資料來源加上註解及進行探索。
* 在 Azure 資料目錄入口網站中，釘選及取消釘選資料資產的能力。 您可以選擇釘選資料資產，方便更輕鬆地重新探索及重複使用。
* Azure 資料目錄入口網站的首頁已經過重新設計。 新的首頁包含針對目前使用者活動的深入解析 (包含最近發佈的資產、釘選的資產和儲存的搜尋)，以及針對整體目錄中活動的深入解析。
* 支援 Azure 資料目錄入口網站中永續性的使用者設定。 使用者體驗設定 - 包括方格檢視或並排顯示檢視、每個頁面上的結果數，以及開啟或關閉搜尋結果醒目提示 - 會在使用者工作階段之間保存。
* Azure 資料目錄目前已在兩個新的 Azure 區域中提供。 可讓客戶佈建 Azure 資料目錄的區域除了美國東部、美國西部、西歐及澳大利亞東部之外，還增加了北歐及東南亞區域。 如需詳細資訊，請參閱 [Azure 地區](https://azure.microsoft.com/regions/)。

> [!NOTE]
> 「在 SQL Server Data Tools 中開啟」需要安裝 Visual Studio 2013 (含 Update 4) 與 SQL Server 工具。 若要安裝最新版的 SQL Server Data Tools，請瀏覽 [下載 SQL Server Data Tools (SSDT)](https://msdn.microsoft.com/library/mt204009.aspx)。


## <a name="whats-new-for-december-2015"></a>2015 年 12 月的新功能
自 2015 年 12 月起，Azure 資料目錄已新增下列功能：

* 「Azure SQL 資料倉儲」資料來源支援資料設定檔。 註冊 Azure SQL 資料倉儲資料表和檢視時，使用者可以選擇包含資料設定檔度量和從資料來源擷取的中繼資料。
* 支援註冊與探索 MySQL 物件和資料庫。 使用者可以使用 Azure 資料目錄資料來源註冊工具來註冊 MySQL 資料來源，並且可以使用 Azure 資料目錄入口網站對已註冊的 MySQL 資料來源加上註解及進行探索。
* 支援針對 Teradata 資料來源進行 SPNEGO 和 Windows 驗證。 註冊 Teradata 資料表和檢視時，使用者可以選擇使用 SPNEGO 和 Windows，以及 LDAP 和 TD2 驗證連線到 Teradata。
* 支援 Azure Data Lake Store 資料來源。 使用者現在可以使用 Azure 資料目錄來註冊及探索 Azure Data Lake Store 資料來源。
* 支援以手動方式在 Azure 資料目錄的資料來源註冊工具中指定網路 Proxy 設定。 使用者可以從工具的 [歡迎使用] 頁面上選取 [修改 Proxy 設定]，而且可以指定該工具所使用的 Proxy 位址和連接埠。

## <a name="whats-new-for-november-2015"></a>2015 年 11 月的新功能
自 2015 年 11 月起，Azure 資料目錄已新增下列功能：

* 能夠在 Azure 資料目錄入口網站中檢視和複製適用於 SQL Server (包括 Azure SQL Database) 和 Oracle 資料來源的連接字串。 使用者可以在 SQL Server 或 Oracle 資料表、檢視或資料庫的連接資訊中按一下 [檢視連接字串] 連結，查看用來連線到資料來源的連接字串。 已針對 SQL Server 資料來源提供 ADO.NET、ODBC、OLEDB 及 JDBC 連接字串。 已針對 Oracle 資料來源提供 ODBC 與 OLEDB 連接字串。
* 支援在註冊 Teradata 資料表和檢視時納入資料設定檔。
* 針對 SQL Server (包含 Azure SQL DB 和 Azure SQL 資料倉儲)、SQL Server Analysis Services、Azure 儲存體，以及 HDFS 來源支援「在 Power BI Desktop 中開啟」。  
* 支援針對 Teradata 資料來源進行 LDAP 驗證。 註冊 Teradata 資料表和檢視時，使用者可以選擇使用 LDAP 及 TD2 驗證連線到 Teradata。
* 針對 Teradata 資料來源支援「在 Excel 中開啟」。
* 在 Azure 資料目錄入口網站中支援最近搜尋的字詞。 在入口網站中搜尋時，使用者可以從最近使用的搜尋字詞中進行選擇以加速探索體驗。
* 支援預覽 Teradata 資料來源。 註冊 Teradata 資料表和檢視時，使用者可以選擇包含快照記錄和從資料來源擷取的中繼資料。
* 「Azure SQL 資料倉儲」資料來源支援「在 Excel 中開啟」。
* 手動註冊的資料資產支援定義和編輯資料行層級的結構描述。 使用 Azure 資料目錄入口網站手動建立資料資產之後，使用者可以在資料資產屬性中新增資料行定義。
* 搜尋 Azure 資料目錄時支援 "has" 查詢，以便能探索擁有特定元資料的已註冊資料資產。 Azure 資料目錄查詢語法現在包含下列項目：

| 查詢語法 | 目的 |
| --- | --- |
| `has:previews` |尋找包含預覽的資料資產 |
| `has:documentation` |尋找已提供文件的資料資產 |
| `has:tableDataProfiles` |尋找具有資料表層級之資料設定檔資訊的資料資產 |
| `has:columnsDataProfiles` |尋找具有資料行層級之資料設定檔資訊的資料資產 |


> [!NOTE]
> 「在 Power BI Desktop 中開啟」需要安裝目前的 Power BI Desktop 應用程式版本。 如果您在使用此功能時遇到問題或發生錯誤，請確定您已從 [PowerBI.com](https://powerbi.com) 取得最新的 Power BI Desktop 版本。


## <a name="whats-new-for-october-2015"></a>2015 年 10 月的新功能
自 2015 年 10 月起，Azure 資料目錄已新增下列功能：

* 支援已註冊資料來源的資料預覽和資料設定檔的靜止狀態加密。 Azure 資料目錄會透明地針對已向服務註冊之資料來源的任何預覽記錄和資料設定檔進行加密，而不需要目錄系統管理員管理金鑰。
* 支援 Teradata 資料來源。 使用者現在可以註冊並探索 Teradata 資料表和檢視。
* 支援內部部署 Hive 資料來源。 使用者現可在內部部署資料來源的 Hadoop 中，註冊並探索適用於 Apache Hive 的 Hive 資料表。
* 支援 Azure 資料目錄入口網站中已儲存的搜尋。 使用者可以儲存的搜尋詞彙和篩選器選取項目，而輕鬆地重複先前的搜尋，及定義目錄內容的實用檢視。 使用者也可以將已儲存的搜尋標示為其預設搜尋。 當使用者從 Azure 資料目錄入口網站首頁或從「入門」頁面中按一下「放大鏡」搜尋圖示時，該使用者會直接進入標示為預設值的已儲存搜尋。
* 支援 Azure Data Catalog 入口網站中已註冊之資料資產和容器的 RTF 文件。 對於標籤和描述不足的案例，使用者現在可以提供資料資產的文件，例如資料表、檢視和報告，以及提供容器的文件，例如資料庫和模型。
* 支援手動註冊已知的資料來源類型。 對於 Azure資料目錄支援的所有資料來源類型，使用者可以使用 Azure 資料目錄入口網站手動輸入資料來源資訊。
* 支援授權 Azure Active Directory 安全性群組。 目錄系統管理員可以讓您存取安全性群組及使用者帳戶的目錄，使得您易於管理 Azure 資料目錄的存取。
* 支援透過 Azure 資料目錄入口網站在 Excel 中開啟 Hive 資料來源。

> [!NOTE]
> 目前版本中，僅支援 Teradata TD2 驗證。 在未來發行的版本中會支援更多驗證機制。

> [!NOTE]
> 若要對 Hive 資料來源使用「在 Excel 中開啟」功能，使用者必須已安裝 Hive 適用的 ODBC 驅動程式。

## <a name="whats-new-for-september-2015"></a>2015 年 9 月的新功能
自 2015 年 9 月起，Azure 資料目錄已新增下列功能：

* 在登錄 Hive 資料來源時納入資料設定檔的支援。
* 支援以程式設計方式探索目錄 API，更輕易地整合應用程式與 Azure 資料目錄。
* 在 Azure 資料目錄入口網站中新的「入門」資料來源探索體驗。 當使用者進入 Azure 資料目錄入口網站的「探索」頁面而未輸入搜尋字詞時，他們會看見目錄內容的概觀，包括最常使用的標記、專家、資料來源類型及物件類型。
* 支援註冊與探索 Azure SQL 資料倉儲物件和資料庫。 如需 Azure SQL 資料倉儲的詳細資訊，請參閱 [SQL 資料倉儲](https://azure.microsoft.com/services/sql-data-warehouse/)。
* 支援將 SQL Server Analysis Services 模型和 SQL Server Reporting Services 伺服器做為容器來進行註冊與探索。 註冊 SSAS 和 SSRS 物件時，Azure 資料目錄會針對 SSAS 模型和 SSRS 伺服器，以及報告和其他物件建立項目。 您可以使用 Azure 資料目錄入口網站來探索容器，並為其加上註解。 除了搜尋和篩選目錄的內容之外，使用者也可以搜尋和篩選模型或伺服器的內容。
* 支援透過 HTTP/HTTPS 註冊和探索 SQL Server Analysis Servics 物件。 使用者現在可以使用 URL (例如 https://servername/olap/msmdpump.dll)) 連線至 SSAS 伺服器，而無需使用伺服器名稱，而且可使用 Windows 驗證以外的基本驗證與匿名連線。 如需 HTTP/HTTPS 連線至 SSAS 的其他資訊，請參閱 [設定 Analysis Services 的 HTTP 存取](https://msdn.microsoft.com/library/gg492140.aspx)。
* 針對 HDInsight 提供 Hive 資料來源支援。 使用者現可在 HDInsight 資料來源的 Hadoop 中，註冊並探索適用於 Apache Hive 的 Hive 資料表。 如需 HDInsight 上 Hive 的其他資訊，請參閱 [HDInsight 文件中心](../hdinsight/hadoop/hdinsight-use-hive.md)。
* 支援註冊與探索 Oracle 資料庫及 HDFS 叢集以做為容器。 註冊 Oracle 資料表和檢視或 HDFS 時，Azure 資料目錄會建立資料庫、資料表及檢視的項目。 您可以使用 Azure 資料目錄入口網站來探索資料庫，並為其加上註解。 除了搜尋和篩選目錄內容之外，使用者亦可搜尋和篩選資料庫或叢集的內容。
* 支援手動註冊未知的資料來源類型。 使用者可透過 Azure 資料目錄入口網站手動輸入資料來源資訊，如此即可為資料來源註冊工具未明確支援的資料來源加上註解並加以探索。
* 支援註冊與探索 SQL Server 資料庫以做為容器。 註冊 SQL Server 資料表和檢視時，Azure 資料目錄會建立資料庫、資料表及檢視的項目。 您可以使用 Azure 資料目錄入口網站來探索資料庫，並為其加上註解。 除了搜尋和篩選目錄的內容之外，使用者也可以搜尋和篩選資料庫的內容。

> [!NOTE]
> 在 9 月 18 日發行之前註冊的 SSAS 和 SSRS 物件，必須在模型或伺服器項目加入目錄之前，使用資料來源註冊工具重新加以註冊。 重新註冊資料來源並不會影響使用者在 Azure 資料目錄入口網站中所加入的任何註解。

> [!NOTE]
> 在 9 月 11 日發行之前註冊的 Oracle 資料表和檢視及 HDFS 檔案和目錄，必須在資料庫或叢集項目加入目錄之前，使用資料來源註冊工具重新加以註冊。 重新註冊資料來源並不會影響使用者在 Azure 資料目錄入口網站中所加入的任何註解。

> [!NOTE]
> 在 9 月 4 日發行之前註冊的 SQL Server 資料表和檢視，必須在資料庫項目加入目錄之前，使用資料來源註冊工具重新加以註冊。 重新註冊資料來源並不會影響使用者在 Azure 資料目錄入口網站中所加入的任何註解。

## <a name="whats-new-for-august-2015"></a>2015 年 8 月的新功能
自 2015 年 8 月起，Azure 資料目錄已新增下列功能：

* 支援 SQL Server 和 Oracle 資料來源的資料分析。 在註冊 SQL Server 和 Oracle 資料表和檢視時，使用者可以選擇要包含正在註冊之物件的資料設定檔資訊。 資料設定檔包含物件層級與資料行層級的統計資料。
* 支援 Hadoop HDFS 資料來源。 使用者現在可以註冊並探索 HDFS 檔案和目錄。
* 支援針對註冊的資料來源提供存取要求資訊。 使用者現在可以針對任何已註冊的資料資產，提供要求存取權的指令 (包括電子郵件連結或 URL)，以輕鬆整合現有工具和程序。
* 標籤和專家的工具提示，讓您可以更輕鬆探索哪些使用者已為註冊的資料資產提供何種中繼資料。
* 我們已將新的「使用者」按鈕和功能表加入我們的上方導覽列中。 此功能表可讓使用者查看用來登入 Azure 資料目錄的帳戶，以及在必要時登出。 此功能表也會顯示目錄名稱，這可為使用 Azure 資料目錄 REST API 的開發人員提供實用資訊。
* 僅限標準版：將擁有者新增至資料資產時，Azure 資料目錄現在可同時支援將使用者帳戶和安全性群組做為擁有者。 若要將安全性群組加入並使其成為所選資料資產的擁有者，您可以輸入群組的顯示名稱或群組的 UPN 電子郵件地址 (如果有的話)。
* 支援 Azure Blob 儲存體的資料來源。 使用者現在可以註冊並探索 Azure 儲存體 Blob 和目錄。

