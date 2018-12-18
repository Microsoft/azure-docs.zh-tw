---
title: Azure SQL Database 的資料探索與分類 | Microsoft Docs
description: Azure SQL Database 的資料探索與分類
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: giladmit
ms.author: giladm
ms.reviewer: vanto
manager: craigg
ms.date: 09/10/2018
ms.openlocfilehash: d34bb54729fe0adc4b26d213bfaa4ad4fb210ab7
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064170"
---
# <a name="azure-sql-database-data-discovery-and-classification"></a>Azure SQL Database 的資料探索與分類
資料探索與分類 (目前處於預覽階段) 提供內建於 Azure SQL Database 的進階功能，可用於**探索**、**分類**、**標記** & **保護**資料庫中的敏感性資料。
對於最具敏感性的資料 (商業、財務、醫療保健與個人識別資料 (PII) 等) 進行探索與分類，在組織的資訊保護方面扮演著關鍵角色。 它可以作為以下的基礎結構：
* 協助符合資料隱私標準和法規合規性需求。
* 各種安全性案例，例如針對敏感性資料異常存取的監視 (稽核) 及警示。
* 控制對包含高度敏感性資料之資料庫的存取，並強化安全性。

資料探索與分類是 [SQL 進階威脅防護](sql-advanced-threat-protection.md) (ATP) 供應項目的一部分，該供應項目是進階 SQL 安全性功能的整合套件。 資料探索與分類可以透過中央 SQL ATP 入口網站存取及管理。

> [!NOTE]
> 這份文件只與 Azure SQL Database 相關。 若是 SQL Server (內部部署)，請參閱 [SQL 資料探索與分類](https://go.microsoft.com/fwlink/?linkid=866999)。

## <a id="subheading-1"></a>什麼是資料探索與分類？
資料探索與分類導入一組進階服務和新的 SQL 功能，構成目標是保護資料而不只是資料庫的全新 SQL 資訊保護典範：
* **探索與建議**：分類引擎會掃描您的資料庫，並識別包含可能是敏感性資料的資料行。 接著，它能提供輕鬆的方式，讓您透過 Azure 入口網站檢閱並套用適當的分類建議。
* **標記**：使用導入 SQL 引擎的全新分類中繼資料屬性，可以在資料行上持續標記敏感性分類標籤。 此中繼資料還可利用在進階的敏感性稽核和保護案例上。
* **查詢結果集敏感度**：為執行稽核，查詢結果集的敏感度是即時計算的。
* **可見性**：資料庫分類狀態可以在入口網站的詳細儀表板中檢視。 此外，您可以下載 Excel 格式的報表以用於合規性、稽核用途及其他需求。

## <a id="subheading-2"></a>針對機密資料行進行探索、分類與設定標籤
下一節描述關於探索、分類及標記資料庫中包含敏感性資料之資料行，以及檢視資料庫目前的分類狀態和匯出報告的步驟。

分類包含兩個中繼資料屬性：
* 標籤：主要分類屬性，用來定義儲存在資料行中的資料敏感度等級。  
* 資訊類型：為儲存在資料行中的資料類型提供額外的細微性。

## <a name="define-and-customize-your-classification-taxonomy"></a>定義及自訂您的類別分類法

SQL 資料探索與分類隨附一組內建的敏感度標籤與一組內建的資訊類型和探索邏輯。 您現在可以自訂此分類法，並定義專門針對您的環境建構之類別的集合和順位。

類別分類法的定義及自訂會在您整個 Azure 租用戶的一個集中位置完成。 該位置位於 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)中，做為您的安全性原則的一部分。 只有具備租用戶根管理群組系統管理權限的人可以執行此工作。

作為資訊保護原則管理的一部分，您可以定義自訂標籤、對它們進行排名，並將它們與一組選取的資訊類型相關聯。 您也可以新增自己的自訂資訊類型，並使用字串模式對其進行設定，字串模式將新增至探索邏輯中，以便在資料庫中識別此類型的資料。
在[資訊保護原則操作指南](https://go.microsoft.com/fwlink/?linkid=2009845&clcid=0x409) \(英文\) 中深入了解有關自訂及管理您原則的詳細資訊。

一旦定義整個租用戶的原則，您就可以使用您的自訂原則，繼續對個別資料庫進行分類。

## <a name="classify-your-sql-database"></a>將您的 SQL Database 分類

1. 移至 [Azure 入口網站](https://portal.azure.com)。

2. 瀏覽至 [Azure SQL Database] 窗格中 [安全性] 標題下的 [進階威脅防護]。 按一下以啟用進階威脅防護，然後按一下 [資料探索與分類 (預覽)] 卡片。

   ![掃描資料庫](./media/sql-data-discovery-and-classification/data_classification.png)

3. [概觀] 索引標籤包含資料庫目前分類狀態的摘要，包括所有已分類資料行的詳細清單，您也可以篩選這些資料行來只檢視特定的結構描述組件、資訊類型與標籤。 如果您尚未分類任何資料行，請[跳至步驟 5](#step-5)。

   ![目前分類狀態的摘要](./media/sql-data-discovery-and-classification/2_data_classification_overview_dashboard.png)

4. 若要下載 Excel 格式的報表，請按一下視窗頂端功能表中的 [匯出] 選項。

   ![匯出至 Excel](./media/sql-data-discovery-and-classification/3_data_classification_export_report.png)

5.  <a id="step-5"></a>若要開始分類資料，請按一下視窗頂端的 [分類] 索引標籤。

    ![分類您的資料](./media/sql-data-discovery-and-classification/4_data_classification_classification_tab_click.png)

6. 分類引擎會掃描您的資料庫，以尋找包含可能是敏感性資料的資料行，並提供 [建議的資料行分類] 清單。 檢視並套用分類建議：

    * 若要檢視建議的資料行分類清單，請按一下視窗底部的建議面板：

      ![分類您的資料](./media/sql-data-discovery-and-classification/5_data_classification_recommendations_panel.png)

    * 檢閱建議清單：若要接受針對特定資料行的建議，請選取相關資料列左側資料行中的核取方塊。 您也可以選取建議資料表標頭中的核取方塊，將「所有建議」標記為接受。

       ![檢閱建議清單](./media/sql-data-discovery-and-classification/6_data_classification_recommendations_list.png)

    * 若要套用選取的建議，請按一下藍色的 [接受選取的建議] 按鈕。

      ![套用建議](./media/sql-data-discovery-and-classification/7_data_classification_accept_selected_recommendations.png)

7. 您也可以選擇將資料行「手動分類」，或同時採用手動分類和建議分類：

    * 按一下視窗頂端功能表中的 [新增分類]。

      ![手動新增分類](./media/sql-data-discovery-and-classification/8_data_classification_add_classification_button.png)

    * 在開啟的內容視窗中，選取您想分類的 [結構描述] > [資料表] > [資料行]，以及資訊類型和敏感度標籤。 然後按一下內容視窗底部的藍色 [新增分類] 按鈕。

      ![選取要分類的資料行](./media/sql-data-discovery-and-classification/9_data_classification_manual_classification.png)

8. 若要完成分類，並持續以新的分類中繼資料標記資料庫資料行，請按一下視窗頂端功能表中的 [儲存]。

   ![儲存](./media/sql-data-discovery-and-classification/10_data_classification_save.png)

## <a id="subheading-3"></a>對敏感性資料的存取進行稽核

資訊保護範例的重要層面是能夠監視對敏感性資料的存取。 我們已強化 [Azure SQL Database 稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)，並在稽核記錄中包含新欄位 *data_sensitivity_information*，其中會記錄查詢所傳回之實際資料的敏感度分類 (標籤)。

![稽核記錄檔](./media/sql-data-discovery-and-classification/11_data_classification_audit_log.png)

## <a id="subheading-4"></a>自動/程式設計分類

您可以使用 T-SQL 新增/移除資料行分類，以及擷取整個資料庫的所有分類。

> [!NOTE]
> 使用 T-SQL 管理標籤時，系統不會驗證新增到資料行的標籤是否存在於組織資訊保護原則 (顯示於入口網站建議中的那組標籤)。 因此，這項驗證會由您執行。

* 新增/更新一或多個資料行的分類：[ADD SENSITIVITY CLASSIFICATION](https://docs.microsoft.com/sql/t-sql/statements/add-sensitivity-classification-transact-sql)
* 從一或多個資料行移除分類：[DROP SENSITIVITY CLASSIFICATION](https://docs.microsoft.com/sql/t-sql/statements/drop-sensitivity-classification-transact-sql)
* 檢視資料庫上的所有分類：[sys.sensitivity_classifications](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-sensitivity-classifications-transact-sql)

您也可以使用 REST API 以程式設計方式管理分類。 已發行的 REST API 支援下列作業：
* [建立或更新](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/createorupdate) - 建立或更新指定資料行的敏感度標籤
* [刪除](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/delete) - 刪除指定資料行的敏感度標籤
* [取得](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/get) - 取得指定資料行的敏感度標籤
* [依資料庫列出](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/listbydatabase) - 取得指定資料庫的敏感度標籤


## <a id="subheading-5"></a>後續步驟

- 深入了解 [SQL 進階威脅防護](sql-advanced-threat-protection.md)。
- 請考慮設定 [Azure SQL Database 稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)，以監視和稽核對已分類敏感性資料的存取。

<!--Anchors-->
[SQL Data Discovery & Classification overview]: #subheading-1
[Discovering, classifying & labeling sensitive columns]: #subheading-2
[Auditing access to sensitive data]: #subheading-3
[Automated/Programmatic classification]: #subheading-4
[Next Steps]: #subheading-5
