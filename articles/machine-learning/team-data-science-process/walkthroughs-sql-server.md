---
title: 使用 R、Python 和 T-SQL 的 SQL Server 資料科學逐步解說 | Microsoft Docs
description: 舉例逐步解說如何使用 SQL Server 中的 R、Python 和 T-SQL 來執行預測性分析。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/04/2017
ms.author: deguhath
ms.openlocfilehash: b1b49d9f2f215d08586d4e988fbe5f647da3b937
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394897"
---
# <a name="sql-server-data-science-walkthroughs-using-r-python-and-t-sql"></a>使用 R、Python 和 T-SQL 的 SQL Server 資料科學逐步解說

這些逐步解說會使用 SQL Server、SQL Server R Services 和 SQL Server Python Services 來執行預測性分析。 R 和 Python 程式碼會部署在預存程序中。 其遵循 Team Data Science Process 中所述的步驟。 如需 Team Data Science Process 的概觀，請參閱 [Data Science Process](overview.md)。 

執行 Team Data Science Process 的其他資料科學逐步解說是依據它們使用的**平台**來歸納整理。 如需這些範例的列述，請參閱[執行 Team Data Science Process 的逐步解說](walkthroughs.md)。


## <a name="predict-taxi-tips-using-python-and-sql-queries-with-sql-server"></a>使用 Python 和 SQL 查詢搭配 SQL Server 來預測計程車小費 

[使用 SQL Server](sql-walkthrough.md) 逐步解說會示範如何使用 SQL Server，針對公開使用的 NYC 計程車車程和車資資料集建置和部署機器學習服務的分類和迴歸模型。


## <a name="predict-taxi-tips-using-microsoft-r-with-sql-server"></a>使用 Microsoft R 搭配 SQL Server 來預測計程車小費 

[使用 SQL Server R Services](https://msdn.microsoft.com/library/mt612857.aspx) 逐步解說會對資料科學家提供 R 程式碼、SQL Server 資料和自訂 SQL 函式的組合，以在 SQL Server 建置和部署 R 模型。 此逐步解說是為了向 R 開發人員介紹 R Services (資料庫內) 所設計。


## <a name="predict-taxi-tips-using-r-from-t-sql-or-stored-procedures-with-sql-server"></a>使用 T-SQL 或預存程序中的 R 搭配 SQL Server 來預測計程車小費

[Data science walkthrough for R and SQL Server](https://docs.microsoft.com/sql/advanced-analytics/tutorials/walkthrough-data-science-end-to-end-walkthrough) (適用於 R 和 SQL Server 的資料科學逐步解說) 會對 SQL 程式設計人員提供使用 SQL Server R Services 來建置搭配 Transact-SQL 的進階分析解決方案的體驗，以操作 R 解決方案。 


## <a name="predict-taxi-tips-using-python-in-sql-server-stored-procedures"></a>使用 SQL Server 預存程序中的 Python 來預測計程車小費

[使用 T-SQL 與 SQL Server Python Services](https://docs.microsoft.com/sql/advanced-analytics/tutorials/sqldev-in-database-python-for-sql-developers) 逐步解說可向 SQL 程式設計師提供在 SQL Server 中建置機器學習解決方案的經驗。 該逐步解說會說明如何透過將 Python 程式碼新增至預存程序，來將 Python 合併到應用程式中。


## <a name="next-steps"></a>後續步驟

如需組成 Team Data Science Process 之主要元件的討論，請參閱 [Team Data Science Process 概觀](overview.md)。

如需您可以用來結構化資料科學專案之 Team Data Science Process 生命週期的討論，請參閱 [Team Data Science Process 生命週期](lifecycle.md)。 生命週期會概述專案在執行時通常會遵循之從開始到完成的步驟。 