---
title: 使用 Azure 串流分析事件處理的即時事件處理
description: 本文將說明參考架構，以使用 Azure 串流分析達成即時事件處理和分析。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 01/24/2017
ms.openlocfilehash: 8a5d426d67916e010c7fff048eebdc77b93c5c38
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30902577"
---
# <a name="reference-architecture-real-time-event-processing-with-microsoft-azure-stream-analytics"></a>參考架構：使用 Microsoft Azure 串流分析的即時事件處理
使用 Microsoft Azure 串流分析的即時事件處理之參考架構主要用來提供一般的藍圖，供使用 Microsoft Azure 部署即時的平台即服務 (PaaS) 串流處理解決方案。

## <a name="summary"></a>總結
傳統上，分析解決方案為基於例如 ETL (擷取、轉換、載入) 和資料倉儲等功能運作，其中資料在分析前就已儲存。 不斷變化的要求，包含更快速到達的資料，將此現有的模型推到了極限。 在移動串流到儲存體之前分析資料的能力是一種解決方式，雖然這並非新的功能，但此方式並未廣泛受到所有產業縱向市場採用。 

Microsoft Azure 提供分析技術的全面目錄，可以支援一系列不同的解決方案案例和要求。 當產品相當多元時，選取要為端對端解決方案部署哪一個 Azure 服務可說是項挑戰。 本文件主要用來描述支援事件串流解決方案之各種 Azure 服務的功能和交互操作。 這也說明客戶可從此類方式中獲益的某些案例。

## <a name="contents"></a>內容
* 執行摘要
* 即時分析簡介
* 在 Azure 中的即時資料價值定位
* 即時分析的常見案例
* 架構與元件
  * 資料來源
  * 資料整合層
  * 即時分析層
  * 資料儲存層
  * 展示/使用層
* 結論

**作者：** Charles Feddersen, Solution Architect, Data Insights Center of Excellence, Microsoft Corporation

**發行日期：** 2015 年 1 月

**修訂版本：** 1.0

**下載：** [使用 Microsoft Azure 串流分析的即時事件處理](http://download.microsoft.com/download/6/2/3/623924DE-B083-4561-9624-C1AB62B5F82B/real-time-event-processing-with-microsoft-azure-stream-analytics.pdf)

## <a name="get-help"></a>取得說明
如需進一步的協助，請參閱 [Azure 串流分析論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

