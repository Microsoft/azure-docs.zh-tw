---
title: Azure Machine Learning 資料準備之執行與資料環境的支援組合 | Microsoft Docs
description: 本文件提供 Azure Machine Learning 資料準備的不同執行階段與資料來源之支援組合的完整清單
services: machine-learning
author: euangMS
ms.author: euang
manager: lanceo
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: ''
ms.devlang: ''
ms.topic: article
ms.date: 02/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 9168ac1d26432ca3eee5a59b63aa0cec3ae72856
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989051"
---
# <a name="supported-matrix-for-this-release"></a>此版本支援的矩陣 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


當您的程式碼使用 Azure Machine Learning 資料來源或 Azure Machine Learning 資料準備載入資料，取得 Pandas 或 Spark 資料框架時，支援以下實驗計算環境和資料位置的組合：

|     |本機檔案  |Azure Blob 儲存體  |SQL Server 資料庫***  |
|---------|---------|---------|---------|---------|
|本機 Python    |     支援    |不支援         | 不支援        |         |
|Docker (Linux VM) Python     |僅在專案檔中支援*         | 不支援        |        不支援 |         |
|Docker (Linux VM) PySpark     |僅在專案檔中支援*     |支援         | 支援\*\*        |         |
|Azure 資料科學虛擬機器 Python     |僅在專案檔中支援*         |不支援         |不支援         |         |
|Azure 資料科學虛擬機器 PySPark     | 僅在專案檔中支援*        |不支援         |不支援         |         |
|Azure HDInsight PySpark     | 不支援        |支援         |支援\*\*         |         |
|Azure HDInsight Python     | 不支援        | 不支援        | 不支援        |         |

Azure Data Lake Store 目前不支援任何計算目標。

*使用本機檔案路徑時，會將專案中的檔案複製到計算環境，然後在該環境中讀取。 不會複製專案外部的檔案，也不會在計算環境中解析路徑。 請考慮使用「資料來源取代」，使您的程式碼可以在本機執行時使用本機檔案。 然後切換至 Azure 儲存體 Blob 以進行不同的執行設定。 您也可以使用資料來源支援的取樣，以管理僅在某些執行設定中執行的大型資料。

\*\*使用 Maven JDBC SQL Server 驅動程式 6.2.1。 您必須確定該計算環境的 spark_dependencies.yml 檔案包含此封裝 (或相容的封裝)。

***支援 Azure SQL Database 或 SQL Server，前提是資料庫可從計算環境中連線。 
