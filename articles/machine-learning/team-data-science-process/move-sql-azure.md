---
title: 移動資料至 Azure 機器學習的 Azure SQL Database | Microsoft Docs
description: 建立 SQL 資料表以及將資料載入 SQL 資料表
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 50f8b862-4d32-44b2-a1e2-4fbc8024acaa
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 5/04/2018
ms.author: deguhath
ms.openlocfilehash: 99141b7542cc9adadadda6ada670b644d3aba5f3
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393638"
---
# <a name="move-data-to-an-azure-sql-database-for-azure-machine-learning"></a>移動資料至 Azure 機器學習的 Azure SQL Database
本主題概述從一般檔案 (CSV 或 TSV 格式) 或儲存在內部部署 SQL Server 中的資料，將資料移動至 Azure SQL 資料庫的選項。 將資料移到雲端的這些工作是 Team Data Science Process 的一部分。

如需概述用來將資料移至機器學習服務的內部部署 SQL Server 之選項的主題，請參閱[移動資料至 Azure 虛擬機器上的 SQL Server](move-sql-server-virtual-machine.md)。

下列 **功能表** 所連結的主題會說明如何將資料內嵌至目標環境，以在 Team Data Science Process (TDSP) 期間儲存和處理該資料。

[!INCLUDE [cap-ingest-data-selector](../../../includes/cap-ingest-data-selector.md)]

下表摘要說明移動資料至 Azure SQL Database 的選項。

| <b>來源</b> | <b>目的地：Azure SQL Database</b> |
| --- | --- |
| <b>一般檔案 (CSV 或 TSV 格式)</b> |[大量插入 SQL 查詢](#bulk-insert-sql-query) |
| <b>內部部署 SQL Server</b> |1.[匯出至一般檔案](#export-flat-file)<br> 2.[SQL Database 移轉精靈](#insert-tables-bcp)<br> 3.[資料庫備份和還原](#db-migration)<br> 4.[Azure Data Factory](#adf) |

## <a name="prereqs"></a>必要條件
此處概述的程序要求您須擁有：

* **Azure 訂用帳戶**。 如果您沒有訂用帳戶，可以註冊 [免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure 儲存體帳戶**。 在本教學課程中，您會使用 Azure 儲存體帳戶來儲存資料。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md) 一文。 建立儲存體帳戶之後，您必須取得用來存取儲存體的帳戶金鑰。 請參閱[管理儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)。
* 存取 **Azure SQL Database**。 如果您必須設定 Azure SQL Database， [開始使用 Microsoft Azure SQL Database](../../sql-database/sql-database-get-started.md) 一文中提供如何佈建 Azure SQL Database 之新執行個體的相關資訊。
* 已在本機上安裝和設定 **Azure PowerShell** 。 如需指示，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

**資料**：移轉程序會使用 [NYC 計程車資料集](http://chriswhong.com/open-data/foil_nyc_taxi/)進行示範。 在 Azure Blob 儲存體上，NYC 計程車資料集包含且提供有關車程資料與費用的相關資訊：[NYC 計程車資料](http://www.andresmh.com/nyctaxitrips/)。 這些檔案的範例和說明都會在 [NYC 計程車車程資料集說明](sql-walkthrough.md#dataset)中提供。

您可以將上述程序調整為自己的資料集，或者遵循上述步驟使用 NYC 計程車資料集。 若要將 NYC 計程車資料集上傳至您的內部部署 SQL Server 資料庫，請遵循[大量匯入資料至 SQL Server 資料庫](sql-walkthrough.md#dbload)中概述的程序進行。 這些指示適用於 Azure 虛擬機器上的 SQL Server，但將資料上傳至內部部署 SQL Server 的程序是相同的。

## <a name="file-to-azure-sql-database"></a> 從一般檔案來源移動資料至 Azure SQL 資料庫
一般檔案 (CSV 或 TSV 格式) 中的資料可以透過大量插入 SQL 查詢移動至 Azure SQL Database。

### <a name="bulk-insert-sql-query"></a> 大量插入 SQL 查詢
程序中使用大量插入 SQL 查詢的步驟，與從一般檔案來源移動資料至 Azure VM 上的 SQL Server 各節涵蓋之步驟類似。 如需詳細資料，請參閱 [大量插入 SQL 查詢](move-sql-server-virtual-machine.md#insert-tables-bulkquery)。

## <a name="sql-on-prem-to-sazure-sql-database"></a> 從內部部署 SQL Server 將資料移至 Azure SQL 資料庫
如果來源資料儲存在內部部署的 SQL Server，則移動資料至 Azure SQL 資料庫就擁有各種可能性：

1. [匯出至一般檔案](#export-flat-file)
2. [SQL Database 移轉精靈](#insert-tables-bcp)
3. [資料庫備份和還原](#db-migration)
4. [Azure Data Factory](#adf)

前三個步驟與涵蓋這些相同程序的 [移動資料至 Azure 虛擬機器上的 SQL Server](move-sql-server-virtual-machine.md) 中各節所說明的程序非常類似。 下列指示會提供該主題的適當章節連結。

### <a name="export-flat-file"></a>匯出至一般檔案
此匯出為一般檔案的步驟與 [匯出為一般檔案](move-sql-server-virtual-machine.md#export-flat-file)中所說明的步驟類似。

### <a name="insert-tables-bcp"></a>SQL Database 移轉精靈
使用 [SQL Database 移轉精靈] 的步驟與 [ [SQL Database 移轉精靈](move-sql-server-virtual-machine.md#sql-migration)] 所說明的步驟類似。

### <a name="db-migration"></a>資料庫備份和還原
使用資料庫備份和還原的步驟與 [資料庫備份和還原](move-sql-server-virtual-machine.md#sql-backup)所說明的步驟類似。

### <a name="adf"></a>Azure Data Factory
[使用 Azure Data Factory 將資料從內部部署 SQL Server 移至 SQL Azure](move-sql-azure-adf.md) 主題會提供使用 Azure Data Factory (ADF) 將資料移至 Azure SQL 資料庫的程序。 本主題說明如何使用 ADF，透過 Azure Blob 儲存體將資料從內部部署的 SQL Server 資料庫移動至 Azure SQL 資料庫。

若同時存取內部部署和雲端資源的混合式案例需要持續移轉資料，或是資料有交易、需要修改，或者在移轉時新增了商務邏輯，請考慮使用 ADF。 ADF 允許使用定期管理資料移動的簡易 JSON 指令碼，來進行排程和監視的工作。 ADF 也有其他功能，例如支援複雜作業。
