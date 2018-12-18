---
title: SQL 資料倉儲建議 - 概念 | Microsoft Docs
description: 深入了解 SQL 資料倉儲的建議與其產生方式
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 07/27/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 57bce631a570f549d46a9b0beefcb5adce4decfc
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44380109"
---
# <a name="sql-data-warehouse-recommendations"></a>SQL 資料倉儲建議

本文說明由 SQL 資料倉儲透過 Azure Advisor 所提供的建議。  

SQL 資料倉儲提供的建議可確保為了效能持續最佳化資料倉儲。 資料倉儲建議與 [Azure Advisor](https://docs.microsoft.com/azure/advisor/advisor-performance-recommendations) 緊密整合，直接在 [Azure 入口網站](https://aka.ms/Azureadvisor)內為您提供最佳做法。 SQL 資料倉儲會每日針對作用中的工作負載，分析資料倉儲目前的狀態、收集遙測資料並顯示建議。 下面說明支援的資料倉儲建議案例以及如何套用建議的動作。

如果您有關於 SQL 資料倉儲 Advisor 的任何意見反應，或遇到任何問題，請連絡 [sqldwadvisor@service.microsoft.com](mailto:sqldwadvisor@service.microsoft.com)。   

按一下[這裡](https://aka.ms/Azureadvisor)，以立即查看建議！ 此功能目前僅適用於 Gen2 資料倉儲。 

## <a name="data-skew"></a>資料扭曲

當執行工作負載時，資料扭曲可能會造成額外的資料移動或資源瓶頸。 下列文件說明如何識別資料扭曲，並透過選取最佳的散發索引鍵來防止發生資料扭曲。

- [識別並移除扭曲](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute#how-to-tell-if-your-distribution-column-is-a-good-choice) 

## <a name="no-or-outdated-statistics"></a>沒有統計資料或已過期

若只有次佳的統計資料，會造成 SQL 資料倉儲查詢最佳化工具產生次佳的查詢計劃，而嚴重影響查詢效能。 下列文件說明建立及更新統計資料的最佳做法：

- [建立及更新資料表統計資料](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-statistics)

若要查看這些建議列出的受影響資料表，請執行下列 [T-SQL 指令碼](https://github.com/Microsoft/sql-data-warehouse-samples/blob/master/samples/sqlops/MonitoringScripts/ImpactedTables)。 Advisor 會持續執行相同的 T-SQL 指令碼，以產生這些建議。
