---
title: 識別案例和規劃分析處理 - Azure | Microsoft Docs
description: 考慮一系列重要問題來規劃進階分析環境。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 421520dd-7728-4d29-889c-ebe6a0a6fb07
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 7754fccdfe8a2c4fad24f61fc22c722d2640ffa6
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37064401"
---
# <a name="how-to-identify-scenarios-and-plan-for-advanced-analytics-data-processing"></a>如何識別案例和規劃進階分析資料處理
在設定環境以對資料集執行進階分析處理時，您應該規劃包含哪些資源？ 本文提出一系列問題，有助於識別您的案例相關的工作和資源。 [何謂 Team Data Science Process (TDSP)？](overview.md)中概述預測性分析的高階步驟順序。 針對您的案例相關的工作，每個步驟會需要特定的資源。 用以識別案例的主要問題是關於資料邏輯、特性、資料集的品質，以及您想要進行分析的工具和語言。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="logistic-questions-data-locations-and-movement"></a>邏輯問題：資料位置和移動
邏輯問題是關於**資料來源**的位置、Azure 中的**目標目的地**，以及移動資料的需求，包括排程、數量和涉及的資源。 在分析程序期間，資料可能需要移動數次。 常見的案例是將本機資料移到 Azure 上的某種儲存體，再移至 Machine Learning Studio。

1. **資料來源是什麼？** 在本機還是在雲端？ 例如︰
   
   * 資料可在 HTTP 位址上公開使用。
   * 資料位於本機或網路檔案位置中。
   * 資料是在 SQL Server 資料庫中。
   * 資料儲存於 Azure 儲存體容器中
2. **Azure 目的地是什麼？** 需要在哪裡處理或建立模型？ 例如︰
   
   * Azure Blob 儲存體
   * SQL Azure 資料庫
   * 在 Azure VM 上的 SQL Server
   * HDInsight (Azure 上的 Hadoop) 或 Hive 資料表
   * Azure Machine Learning
   * 可裝載的 Azure 虛擬硬碟。
3. **您要如何移動資料？** 下列文章概述可用來將資料內嵌或載入至各種不同儲存體和處理環境的程序和資源：
   
   * [將資料載入至儲存體環境以便進行分析](ingest-data.md)
   * [從各種資料來源將定型資料匯入 Azure Machine Learning Studio](../studio/import-data.md)。
4. **資料是否需要定期移動或在移轉期間修改？** 當資料需要持續移轉時，特別是在涉及同時存取內部部署和雲端資源的混合式案例中，或當資料為交易式或在移轉過程中需要修改或新增商務邏輯時，請考慮使用 Azure Data Factory (ADF)。 如需進一步資訊，請參閱[使用 Azure Data Factory 從內部部署 SQL Server 移動資料至 SQL Azure](move-sql-azure-adf.md)
5. **有多少資料要移至 Azure？** 極大型的資料集可能會超過某些環境的儲存體容量。 如需相關範例，請參閱下一節關於 Machine Learning studio 大小限制的討論。 在這種情況下，分析期間可能使用資料樣本。 如需如何在各種 Azure 環境中縮減取樣資料集的詳細資訊，請參閱 [在 Team Data Science Process 中進行資料取樣](sample-data.md)。

## <a name="data-characteristics-questions-type-format-and-size"></a>資料特性問題：類型、格式和大小
這些問題是規劃儲存體和處理環境的關鍵，各適用於不同類型的資料，也各有其特定的限制。

1. **資料類型是什麼？** 例如：
   
   * 數值
   * 類別
   * 字串
   * Binary
2. **如何將資料格式化？** 例如：
   
   * 逗號分隔 (CSV) 或定位鍵分隔 (TSV) 一般檔案
   * 已壓縮或未壓縮
   * Azure Blob
   * Hadoop Hive 資料表
   * SQL Server 資料表
3. **您的資料大小為何？**
   
   * 小型：小於 2 GB
   * 中型：大於 2 GB 且小於 10 GB
   * 大型：大於 10 GB

以 Azure Machine Learning Studio 環境為例子：

* 如需 Azure Machine Learning Studio 支援的資料格式和 類型的清單，請參閱 [支援的資料格式和資料類型](../studio/import-data.md#data-formats-and-data-types-supported) 一節。
* 如需 Azure Machine Learning Studio 資料限制的相關資訊，請參閱 **匯入和匯出機器學習服務的資料** 的 [我的模組適用多大的資料集？](../studio/faq.md#machine-learning-studio-questions)

如需分析程序中使用的其他 Azure 服務限制的相關資訊，請參閱 [Azure 訂用帳戶和服務限制、配額與限制](../../azure-subscription-service-limits.md)。

## <a name="data-quality-questions-exploration-and-pre-processing"></a>資料品質問題：探索和前置處理
1. **您對資料的熟悉程度為何？** 請探索資料，以了解資料的基本特性。 資料呈現什麼模式或趨勢、有什麼極端值或遺漏多少值。 這個步驟很重要，將會決定所需的前置處理程度、形成假設來建議最適合的功能分析類型，以及擬定計劃來收集其他資料。 計算描述性統計資料和繪製視覺效果是很有用的資料檢查技術。 如需如何在各種 Azure 環境中探索資料集的詳細資訊，請參閱 [在 Team Data Science Process 中探索資料](explore-data.md)。
2. **資料是否需要前置處理或清除？**
   前置處理和清除資料是很重要的工作，必須先執行這些工作，才能有效地將資料集用於機器學習服務。 未經處理的資料通常會有雜訊且不可靠，還可能會有遺漏值。 使用這類資料進行模型化可能會產生誤導的結果。 如需說明，請參閱 [準備增強機器學習服務的資料的工作](prepare-data.md)。

## <a name="tools-and-languages-questions"></a>工具和語言的問題
根據您需要或最喜歡使用的語言和開發環境或工具而定，這裡有許多選項可選擇。

1. **您想要使用哪些語言進行分析？**  
   
   * R
   * Python
   * SQL
2. **您應該使用哪些工具進行資料分析？**
   
   * [Microsoft Azure Powershell](/powershell/azure/overview) - 在指令碼語言中用來管理 Azure 資源的指令碼語言。
   * [Azure Machine Learning Studio](../studio/what-is-ml-studio.md)
   * [Revolution Analytics](https://www.microsoft.com/sql-server/machinelearningserver)
   * [RStudio](http://www.rstudio.com)
   * [適用於 Visual Studio 的 Python 工具](http://aka.ms/ptvsdocs)
   * [Anaconda](https://www.continuum.io/why-anaconda)
   * [Jupyter 筆記本](http://jupyter.org/)
   * [Microsoft Power BI](http://powerbi.microsoft.com)

## <a name="identify-your-advanced-analytics-scenario"></a>識別您的進階分析案例
一旦回答上一區段中的問題之後，您便已準備好決定哪個案例最符合您的情況。 [Azure 機器學習服務中的進階分析案例](plan-sample-scenarios.md)中概述範例案例。

