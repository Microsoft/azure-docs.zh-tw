---
title: 無伺服器 SQL 集區
description: 了解 Azure Synapse Analytics 中的無伺服器 SQL 集區。
services: synapse analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 87f592f0cff8504fbafba392d20f405640e2578a
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98682965"
---
# <a name="serverless-sql-pool-in-azure-synapse-analytics"></a>Azure Synapse Analytics 中的無伺服器 SQL 集區 

每個 Azure Synapse Analytics 工作區都會隨附無伺服器 SQL 集區端點，供您用來查詢資料湖中的資料。

無伺服器 SQL 集區是針對資料湖資料進行的查詢服務。 其可讓您透過下列功能存取您的資料：
 
- 已有可供查詢資料的熟悉 T-SQL 語法，讓您不必將資料複製或載入至特製化的存放區。 
- 透過 T-SQL 介面所實現的整合式連線能力，可提供各式各樣的商業智慧和臨機操作查詢工具，包括最熱門的驅動程式。 

無伺服器 SQL 集區是一種分散式資料處理系統，針對大規模資料和計算函式而建置的。 視工作負載而定，無伺服器 SQL 集區可讓您在幾秒到幾分鐘內就完成巨量資料的分析。 由於內建了查詢執行容錯功能，即使是涉及大型資料集的長時間執行查詢，系統也可提供高度的可靠性和成功率。

無伺服器 SQL 集合沒有伺服器，因此不必設置基礎結構，也不用維護叢集。 每個 Azure Synapse 工作區內都會提供用於此服務的預設端點，因此您可以在工作區建立好之後就立即開始查詢資料。 

所保留的資源不會收費，您只需就執行查詢時所處理的資料來支付費用，因此此模型是真正的按使用次數付費模型。  

如果您在資料管線中使用 Apache Spark for Azure Synapse 來進行資料準備、清理或擴充，則可以直接從無伺服器 SQL 集區[查詢您在程序中所建立的外部 Spark 資料表](develop-storage-files-spark-tables.md)。 使用 [Private Link](../security/how-to-connect-to-workspace-with-private-links.md) 則可將無伺服器 SQL 集區端點帶入到[受控工作區 VNet](../security/synapse-workspace-managed-vnet.md)。  

## <a name="serverless-sql-pool-benefits"></a>無伺服器 SQL 集區優點

如果您需要探索資料湖中的資料、取得資料的深入解析，或將現有的資料轉換管線最佳化，則可使用無伺服器 SQL 集區來獲益。 這適用於下列案例：

- 基本探索和探勘 - 快速推論資料湖中各種格式 (Parquet、CSV、JSON) 的資料，以便規劃如何從中擷取深入解析。
- 邏輯資料倉儲 - 在原始或不同資料之上提供關聯式抽取，而不需要重新放置和轉換資料，讓您隨時獲得資料的最新觀點。
- 資料轉換 - 簡單、可擴縮且高效能的方法，讓您可以使用 T-SQL 來轉換資料湖中的資料，以便能夠將資料饋送至 BI 和其他工具，或載入到關聯式資料存放區 (Synapse SQL 資料庫、Azure SQL Database 等)。

不同的專業角色皆可從無伺服器 SQL 集區受益：

- 資料工程師可以使用此服務來探索資料湖、轉換和準備資料，並簡化其資料轉換管線。 如需詳細資訊，請參閱此[教學課程](tutorial-data-analyst.md)。
- 由於有 OPENROWSET 和自動結構描述推斷等功能，資料科學家可以快速地推論資料湖所含資料的內容和結構。
- 資料分析師可以使用連線至無伺服器 SQL 集區的熟悉 T-SQL 語言或其最愛的工具，來[探索資料科學家或資料工程師所建立的資料和 Spark 外部資料表](develop-storage-files-spark-tables.md)。
- BI 專業人員可以快速地[根據資料湖所含的資料建立 Power BI 報告](tutorial-connect-power-bi-desktop.md)和 Spark 資料表。

## <a name="how-to-start-using-serverless-sql-pool"></a>如何開始使用無伺服器 SQL 集區

每個 Azure Synapse 工作區內都會提供無伺服器 SQL 集區端點。 您可以使用熟悉的工具立即建立工作區並開始查詢資料。

## <a name="client-tools"></a>用戶端工具

無伺服器 SQL 集區可讓現有的 SQL 臨機操作查詢和商業智慧工具利用資料湖。 其提供了熟悉的 T-SQL 語法，因此只要是能夠建立 TDS 連線 SQL 供應項目的工具，都可以[連線到 Synapse SQL 隨選並進行查詢](connect-overview.md)。 您可以與 Azure Data Studio 連線並執行臨機操作查詢，也可以與 Power BI 連線以在幾分鐘內取得深入解析。

## <a name="t-sql-support"></a>T-SQL 支援

無伺服器 SQL 集區提供 T-SQL 查詢介面區，其會在某些方面稍微強化/擴充，以配合半結構化和非結構化資料的查詢體驗。 此外，由於無伺服器 SQL 集區的設計，因此並不支援 T-SQL 語言的某些方面，例如，目前就不支援 DML 功能。

- 您可以使用熟悉的概念來組織工作負載：
- 資料庫 - 無伺服器 SQL 集區端點可以有多個資料庫。
- 結構描述 - 資料庫內可存在一或多個物件擁有權群組 (稱為結構描述)。
- 檢視
- 外部資源 - 資料來源、檔案格式和資料表

您可以使用下列項目來強制執行安全性：

- 登入和使用者
- 用來控制儲存體帳戶存取權的認證
- 按照物件層級來授與、拒絕及撤銷權限
- Azure Active Directory 整合

支援的 T-SQL：

- 支援完整的 [SELECT](/sql/t-sql/queries/select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) 介面區，包括大部分的 SQL 函式
- CETAS - CREATE EXTERNAL TABLE AS SELECT
- 僅與檢視和安全性相關的 DDL 陳述式

無伺服器 SQL 集區不具有本機儲存體，只有中繼資料物件會儲存在資料庫中。 因此，不支援與下列概念有關的 T-SQL：

- 資料表
- 觸發程序
- 具體化檢視
- 與檢視和安全性不相關的 DDL 陳述式
- DML 陳述式

### <a name="extensions"></a>延伸模組

為了讓您能夠順暢地查詢位於資料湖檔案中的資料，無伺服器 SQL 集區藉由新增下列功能來擴充現有的 [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) 函式：

[查詢多個檔案或資料夾](query-data-storage.md#query-multiple-files-or-folders)

[PARQUET 檔案格式](query-data-storage.md#query-parquet-files)

[使用分隔文字的其他選項 (欄位結束字元、資料列結束字元、逸出字元)](query-data-storage.md#query-csv-files)

[讀取選擇的資料行子集](query-data-storage.md#read-a-chosen-subset-of-columns)

[結構描述推斷](query-data-storage.md#schema-inference)

[filename 函式](query-data-storage.md#filename-function)

[filepath 函式](query-data-storage.md#filepath-function)

[使用複雜類型和巢狀或重複的資料結構](query-data-storage.md#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="security"></a>安全性

無伺服器 SQL 集區有機制可以保護資料的存取。

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Azure Active Directory 整合和多重要素驗證

無伺服器 SQL 集區可讓您透過 [Azure Active Directory 整合](../../azure-sql/database/authentication-aad-configure.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)，集中管理資料庫使用者和其他 Microsoft 服務的身分識別。 這項功能簡化了權限管理並增強安全性。 Azure Active Directory (Azure AD) 支援[多重要素驗證](../../azure-sql/database/authentication-mfa-ssms-configure.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (MFA)，以提高資料和應用程式安全性，同時支援單一登入程序。

#### <a name="authentication"></a>驗證

無伺服器 SQL 集區驗證是指使用者連線到端點時如何證明他們的身分識別。 支援兩種驗證：

- **SQL 驗證**

  此驗證方法會使用使用者名稱和密碼。

- **Azure Active Directory 驗證**：

  此驗證方法會使用由 Azure Active Directory 管理的身分識別。 您可以為 Azure AD 使用者啟用多重要素驗證。 [盡可能](/sql/relational-databases/security/choose-an-authentication-mode?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)使用 Active Directory 驗證 (整合式安全性)。

#### <a name="authorization"></a>授權

授權是指使用者可以在無伺服器 SQL 集區資料庫內執行的動作，這是由使用者帳戶的資料庫角色成員資格和物件層級權限所控制。

如果使用 SQL 驗證，則 SQL 使用者只會存在於無伺服器 SQL 集區中，且權限會限縮在無伺服器 SQL 集區中的物件。 您無法直接對 SQL 使用者授與其他服務 (例如 Azure 儲存體) 中安全性實體物件的存取權，因為其只存在於無伺服器 SQL 集區的範圍內。 SQL 使用者必須使用其中一個[支援的授權類型](develop-storage-files-storage-access-control.md#supported-storage-authorization-types)才能存取檔案。

如果使用 Azure AD 驗證，則使用者可以登入無伺服器 SQL 集區及其他服務 (例如 Azure 儲存體)，而且您可以對 Azure AD 使用者授與權限。

### <a name="access-to-storage-accounts"></a>儲存體帳戶的存取權

已登入無伺服器 SQL 集區服務的使用者必須獲得授權，才能存取及查詢 Azure 儲存體中的檔案。 無伺服器 SQL 集區支援下列授權類型：

- **共用存取簽章 (SAS)** 可提供您儲存體帳戶中資源的委派存取權。 透過 SAS，您可以對用戶端授與儲存體帳戶中資源的存取權，而不必共用帳戶金鑰。 針對具有 SAS 的用戶端，SAS 可讓您精準控制這些用戶端應獲得的存取類型：有效性間隔、授與權限、可接受的 IP 位址範圍，以及可接受的通訊協定 (HTTPs/HTTP)。

- **使用者身分識別** (也稱為「傳遞」) 是一種授權類型，系統會使用已登入無伺服器 SQL 集區的 Azure AD 使用者身分識別來授權資料的存取。 存取資料之前，Azure 儲存體管理員必須對 Azure AD 使用者授與用來存取資料的權限。 此授權類型會使用已登入無伺服器 SQL 集區的 Azure AD 使用者，因此不支援 SQL 使用者類型。

## <a name="next-steps"></a>後續步驟
您可以在下列文章中找到有關端點連線和查詢檔案的其他資訊： 
- [連線到您的端點](connect-overview.md)
- [查詢您的檔案](develop-storage-files-overview.md)
