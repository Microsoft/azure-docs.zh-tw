---
title: 使用 SQL 資料倉儲建置整合式解決方案 | Microsoft Docs
description: '工具以及提供可與 SQL 資料倉儲整合之解決方案的合作夥伴。 '
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 221d5d05906e7e162013c0d4cdddc01a95f4024c
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302697"
---
# <a name="integrate-other-services-with-sql-data-warehouse"></a>將其他服務與 SQL 資料倉儲整合
除了其核心功能，SQL 資料倉儲可讓使用者整合許多其他 Azure 中的服務。 其中一些服務包括：

* Power BI
* Azure Data Factory
* Azure Machine Learning
* Azure 串流分析

SQL 資料倉儲會持續與整個 Azure 中的更多服務，還有更多[整合合作夥伴](sql-data-warehouse-partner-data-integration.md)整合。

## <a name="power-bi"></a>Power BI
Power BI 整合可讓您結合 SQL 資料倉儲的計算能力以及 Power BI 的動態報告和視覺效果。 目前 Power BI 整合包括：

* **直接連接**：針對 SQL 資料倉儲進行之邏輯下推的更進階連接。 下推可提供更快速且更大規模的分析。
* **在 Power BI 中開啟**：[在 Power BI 中開啟] 按鈕會傳遞執行個體資訊給 Power BI，提供簡化的連接方式。

如需詳細資訊，請參閱[與 Power BI 整合](sql-data-warehouse-get-started-visualize-with-power-bi.md)或 [Power BI 文件](http://blogs.msdn.com/b/powerbi/archive/2015/06/24/exploring-azure-sql-data-warehouse-with-power-bi.aspx)。

## <a name="azure-data-factory"></a>Azure Data Factory
Azure Data Factory 提供使用者一個受控平台，以建立複雜的擷取和載入管線。 SQL 資料倉儲與 Azure Data Factory 的整合包含：

* **預存程序**：協調 SQL 資料倉儲上的預存程序的執行。
* **複製**：使用 ADF 將資料移到「SQL 資料倉儲」中。 這項作業可以在幕後使用 ADF 的標準資料移動機制或 PolyBase。 

如需詳細資訊，請參閱[與 Azure Data Factory 整合](sql-data-warehouse-get-started-visualize-with-power-bi.md)。

## <a name="azure-machine-learning"></a>Azure Machine Learning
Azure Machine Learning 是完全受控的分析服務，可讓您使用一組大型預測工具建立複雜模型。 SQL 資料倉儲可支援做為模型的來源和目的地，具備下列功能：

* **讀取資料：** 針對 SQL 資料倉儲透過 T-SQL 大規模驅動模型。
* **寫入資料：** 認可從任何模型回到 SQL 資料倉儲的變更。

如需詳細資訊，請參閱[與 Azure Machine Learning 整合](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)。

## <a name="azure-stream-analytics"></a>Azure 串流分析
Azure 串流分析是複雜、完全受控的基礎結構，可處理和取用產生自 Azure 事件中樞的事件資料。  與 SQL 資料倉儲的整合可讓串流資料有效地處理並與相關資料一起儲存以啟用更深入、更進階的分析。  

* **工作輸出：** 將來自資料流分析工作的輸出直接傳送到 SQL 資料倉儲。

如需詳細資訊，請參閱[與 Azure 串流分析整合](sql-data-warehouse-integrate-azure-stream-analytics.md)。

## <a name="next-steps"></a>後續步驟
若要與 Azure SQL Database 整合，請參閱[設定 SQL Database 彈性查詢](tutorial-elastic-query-with-sql-datababase-and-sql-data-warehouse.md)

