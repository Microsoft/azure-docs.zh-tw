---
title: 執行資料科學工作 - Azure Machine Learning | Microsoft Docs
description: 資料科學家如何以可追蹤、版本控制，以及共同作業的方式執行資料科學專案。
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
ms.date: 11/28/2017
ms.author: deguhath
ms.openlocfilehash: 19267429510207129b0229dc55fbd46f12977d5d
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39112639"
---
# <a name="execute-data-science-tasks-exploration-modeling-and-deployment"></a>執行資料科學工作：探索、模型化和部署

一般資料科學工作包括資料探索、模型化和部署。 本文示範如何使用**互動式資料探索、分析與報告 (IDEAR)** 和**自動模型化與報告 (AMAR)** 公用程式來完成數個常見的資料科學工作，例如互動式資料探索、資料分析、報告和模型建立。 它也會列出使用各種工具組和資料平台，將模型部署到生產環境的選項，如下所示：

- [Azure Machine Learning](../service/index.yml)
- [SQL-Server 與 ML 服務](https://docs.microsoft.com/sql/advanced-analytics/r/r-services#in-database-analytics-with-sql-server)
- [Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server)


## 1.<a name='DataQualityReportUtility-1'></a> 探索 

資料科學家可以利用許多方式執行探索和報告：使用 Python 可用的程式庫和套件 (例如 matplotlib) 或使用 R (例如 ggplot 或 lattice)。 資料科學家可以自訂這類程式碼，以符合特定情節的資料探索需求。 處理結構化資料與非結構化資料的需求不同，例如文字或影像。 

諸如 Azure Machine Learning Workbench 等產品也提供[進階資料準備](../desktop-workbench/tutorial-bikeshare-dataprep.md)以進行資料討論及探索，包括功能建立。 使用者應決定最符合其需求的工具、程式庫和套件。 

在此階段結尾處的交付項目是資料探索報告。 報告應該提供相當完整的資料檢視，以用於模型化及評估資料是否適合繼續進行模型化步驟。 下列各節中針對半自動探索、模型化和報告所討論的 Team Data Science Process (TDSP) 公用程式也提供標準化的資料探索和模型化報告。 

### <a name="interactive-data-exploration-analysis-and-reporting-using-the-idear-utility"></a>使用 IDEAR 公用程式的互動式資料探索、分析和報告

此 R Markdown 型或 Python Notebook 型公用程式提供彈性且互動式的工具，以評估及瀏覽資料集。 使用者可以使用最少的編碼，從資料集快速產生報告。 使用者可以按一下按鈕，將互動式工具中的探索結果匯出至最終報告，這份報告可以傳遞給用戶端，或用來決定要在後續模型化步驟中包含哪些變數。

目前，此工具只適用於記憶體中的資料框架。 若要指定要探索之資料集的參數，需要 YAML 檔案。 如需詳細資訊，請參閱 [TDSP 資料科學公用程式中的 IDEAR](https://github.com/Azure/Azure-TDSP-Utilities/tree/master/DataScienceUtilities/DataReport-Utils)。


## 2.<a name='ModelingUtility-2'></a> 模型化

以各種不同的語言提供多種定型模型的工具組和套件。 資料科學家應該可以自由使用任何其習慣的項目，只要針對相關商務使用案例與實際執行情節符合有關精確度和延遲的效能考量。

下一節顯示如何使用半自動模型化的 R 型 TDSP 公用程式。 此 AMAR 公用程式可用來快速產生基準線模型，以及需要進行微調，以提供更佳執行模型的參數。
下列模型管理章節會顯示如何具備可註冊和管理多個模型的系統。


### <a name="model-training-modeling-and-reporting-using-the-amar-utility"></a>模型定型：使用 AMAR 公用程式模型化和報告

此[自動模型化與報告 (AMAR) 公用程式](https://github.com/Azure/Azure-TDSP-Utilities/tree/master/DataScienceUtilities/Modeling)提供可自訂的半自動工具，可執行具有超參數清除的模型建立，並且比較這些模型的精確度。 

模型建立公用程式是 R Markdown 檔案，可執行以產生自封式 HTML 輸出，具有目錄以輕鬆瀏覽其不同區段。 執行 Markdown 檔案時會執行三個演算法 (knit)：使用 glmnet 套件的正規化迴歸、使用 randomForest 套件的隨機樹系，以及使用 xgboost 套件的提升樹狀結構)。 每個演算法都會產生定型模型。 然後會比較這些模型的準確度，並且報告相對功能重要性繪圖。 目前有兩個公用程式：一個適用於二進位分類工作，另一個適用於迴歸工作。 兩者之間的主要差異是為這些學習工作指定控制參數和精確度計量的方式。 

YAML 檔案是用來指定：

- 資料輸入 (SQL 來源或 R-資料檔案) 
- 資料的哪個部分用於定型，哪個部分用於測試
- 要執行哪些演算法 
- 模型最佳化之控制參數的選擇：
    - 交叉驗證 
    - 啟動程序
    - 摺疊交叉驗證
- 每個演算法的超參數集合。 

演算法數目、最佳化的摺疊數目、超參數，以及要執行清除之超參數集合的數目，也可以在 Yaml 檔案中修改以快速地執行模型。 例如，可以使用較低數目的 CV 摺疊、較低數目的參數集合來執行。 如果獲得保證，也可以使用較高數目的 CV 摺疊或較大數目的參數集合，從而更全面地執行。

如需詳細資訊，請參閱 [TDSP 資料科學公用程式中的自動化模型化和報告公用程式](https://github.com/Azure/Azure-TDSP-Utilities/tree/master/DataScienceUtilities/Modeling)。

### <a name="model-management"></a>模型管理
在多個模型都已建置完成之後，您通常需要有可註冊和管理模型的系統。 通常您需要指令碼或 API 以及後端資料庫或版本控制系統的組合。 針對這些管理工作，您可以考慮以下的一些選項：

1. [Azure Machine Learning - 模型管理服務](../service/index.yml)
2. [從 MIT ModelDB](https://mitdbg.github.io/modeldb/) 
3. [SQL-Server 即模型管理系統](https://blogs.technet.microsoft.com/dataplatforminsider/2016/10/17/sql-server-as-a-machine-learning-model-management-system/) \(英文\)
4. [Microsoft Machine Learning Server](https://docs.microsoft.com/sql/advanced-analytics/r/r-server-standalone)

## 3.<a name='Deployment-3'></a> 部署

生產環境部署可讓模型在企業中扮演主動角色。 從已部署模型的預測可用於商業決策。

### <a name="production-platforms"></a>生產環境平台
有許多方法和平台可將模型置入生產環境。 以下提供一些選項：


- [在 Azure Machine Learning 中的模型部署](../desktop-workbench/model-management-overview.md)
- [在 SQL-Server 中部署模型](https://docs.microsoft.com/sql/advanced-analytics/tutorials/sqldev-py6-operationalize-the-model)
- [Microsoft Machine Learning Server](https://docs.microsoft.com/sql/advanced-analytics/r/r-server-standalone)

> [!NOTE]
> 在部署之前，必須確保模型計分的延遲夠低，以便在生產環境中使用。
>
>

逐步解說還會提供進一步範例，說明**特定情節**之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)文章中會列出這些逐步解說以及簡短說明的連結。 這些逐步解說說明如何將雲端、內部部署工具及服務組合成工作流程或管線，以建立智慧型應用程式。

> [!NOTE]
> 如需使用 Azure Machine Learning Studio 進行部署，請參閱[部署 Azure Machine Learning Web 服務](../studio/publish-a-machine-learning-web-service.md)。
>
>

### <a name="ab-testing"></a>A/B 測試
多個模型在生產環境中時，執行 [A/B 測試](https://en.wikipedia.org/wiki/A/B_testing)來比較模型的效能會很有用。 

 
## <a name="next-steps"></a>後續步驟

[追蹤資料科學專案的進度](track-progress.md)會顯示資料科學家可如何追蹤資料科學專案的進度。

[模型作業和 CI/CD](ci-cd-flask.md) 會示範如何使用開發模型執行 CI/CD。


