---
title: 比較 Azure Data Factory 與 Data Factory 第 1 版 | Microsoft Docs
description: 本文將比較 Azure Data Factory 與 Azure Data Factory 第 1 版。
services: data-factory
documentationcenter: ''
author: kromerm
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 04/09/2018
ms.author: makromer
ms.openlocfilehash: ca00a414402c1cfdef55cfbde9291688f091bf77
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43095914"
---
# <a name="compare-azure-data-factory-with-data-factory-version-1"></a>比較 Azure Data Factory 與 Data Factory 第 1 版
本文將比較 Data Factory 與 Data Factory 第 1 版。 如需 Data Factory 的指示，請參閱 [Data Factory 簡介](introduction.md)。如需 Data Factory 第 1 版的指示，請參閱 [Azure Data Factory 簡介](v1/data-factory-introduction.md)。 

## <a name="feature-comparison"></a>功能比較
下表比較 Data Factory 的功能與 Data Factory 第 1 版的功能。 

| 功能 | 第 1 版 | 目前版本 | 
| ------- | --------- | --------- | 
| 資料集 | 一個具名的資料檢視，參考您想要在活動中用來作為輸入或輸出的資料。 資料集可識別資料表、檔案、資料夾和文件等各種資料存放區中的資料。 例如，Blob 資料集會指定活動應從中讀取資料之 Blob 儲存體中的 Blob 容器和資料夾。<br/><br/>**可用性**定義資料集的處理時段切割模型 (例如每小時或每天等)。 | 目前版本中的資料集是相同的。 不過，您不需要定義資料集的**可用性**排程。 您可以定義觸發程序資源，以排程時鐘排程器範例中的管線。 如需詳細資訊，請參閱[觸發程序](concepts-pipeline-execution-triggers.md#triggers)和[資料集](concepts-datasets-linked-services.md)。 | 
| 連結的服務 | 連結的服務非常類似連接字串，可定義 Data Factory 連線到外部資源所需的連線資訊。 | 連結服務與 Data Factory V1 中的相同，但具有新的 **connectVia** 屬性，可使用現行 Data Factory 版本的 Integration Runtime 計算環境。 如需詳細資訊，請參閱 [Azure Data Factory 中的整合執行階段](concepts-integration-runtime.md)和 [Azure Blob 儲存體連結的服務屬性](connector-azure-blob-storage.md#linked-service-properties)。 |
| 管線 | 資料處理站可以有一或多個管線。 管線是一起執行某個工作的活動所組成的邏輯群組。 您可以使用 startTime、endTime 和 isPaused 來排程及執行管線。 | 管線是要對資料執行的活動群組。 不過，管線中活動的排程已分隔成新的觸發程序資源。 您可以將現行 Data Factory 版本 中的管線視為您透過觸發程序個別排程的「工作流程單位」。 <br/><br/>在現行 Data Factory 版本中的管線沒有時間執行「視窗」。 startTime、endTime 及 isPaused 的 Data Factory V1 概念已不存在於目前的 Data Factory 版本中。 如需詳細資訊，請參閱[管道執行和觸發程序](concepts-pipeline-execution-triggers.md)和[管線和活動](concepts-pipelines-activities.md)。 |
| 活動 | 活動會定義要在管線中資料上執行的動作。 支援資料移動 (複製活動) 和資料轉換活動 (例如 Hive、Pig 和 MapReduce)。 | 在目前的 Data Factory 版本中，活動仍是管線中已定義的動作。目前的 Data Factory 版本導入了新的[控制流程活動](concepts-pipelines-activities.md#control-activities)。 您可以在控制流程 (迴圈和分支) 中使用這些活動。 目前的版本可支援 V1 中支援的資料移動和資料轉換活動。 您可以定義轉換活動，而不需使用目前版本中的資料集。 |
| 混合式資料移動和活動分派 | [資料管理閘道](v1/data-factory-data-management-gateway.md)現在稱為整合執行階段，可支援在內部部署與雲端之間移動資料。| 資料管理閘道現在稱為「自我裝載整合執行階段」。 它提供的功能與在 V1 中相同。 <br/><br/> 現行 Data Factory 版本中的 Azure-SSIS Integration Runtime 也支援在雲端部署和執行 SQL Server Integration Services (SSIS) 套件。 如需詳細資訊，請參閱 [Azure Data Factory 中的整合執行階段](concepts-integration-runtime.md)。|
| 參數 | NA | 參數是管線中定義之唯讀組態設定的鍵值組。 當您手動執行管線時，您可以傳遞參數的引數。 如果您使用排程器觸發程序，此觸發程序也可以傳遞參數值。 管線內的活動會取用參數值。  |
| 運算式 | Data Factory V1 可讓您在資料選取項目查詢與活動/資料集屬性中使用函式和系統變數。 | 在目前的 Data Factory 版本中，您可以在 JSON 字串值中的任意處使用運算式。 如需詳細資訊，請參閱[現行 Data Factory 版本中的運算式和函式](control-flow-expression-language-functions.md)。|
| 管線執行 | NA | 管線執行的單一執行個體。 例如，假設您的管線會在上午 8 點、9 點和 10 點執行。 在此情況下，管線會有三個個別的執行 (管線執行)。 每個管線執行都有唯一的管線執行識別碼。 管線執行識別碼是可唯一定義該特定管理執行的 GUID。 通常將引數傳遞給管線中定義的參數，以具現化管線執行。 |
| 活動執行 | NA | 管線內活動執行的執行個體。 | 
| 觸發程序執行 | NA | 觸發程序執行的執行個體。 如需詳細資訊，請參閱[觸發程序](concepts-pipeline-execution-triggers.md)。 |
| 排程 | 排程是以管線開始/結束時間和資料集的可用性為基礎。 | 透過外部排程器的排程器觸發程序或執行。 如需詳細資訊，請參閱[管道執行和觸發程序](concepts-pipeline-execution-triggers.md)。 |

以下各節提供關於現行版本功能的詳細資訊。 

## <a name="control-flow"></a>控制流程  
為了支援現代化資料倉儲中的各種整合流程和模式，目前的 Data Factory 版本已啟用不再與時間序列資料繫結的新式彈性資料管線模型。 現在已啟用幾個先前無法使用的常見流程。 這些流程將於下列各節中說明。   

### <a name="chaining-activities"></a>鏈結活動
在 V1 中，您必須將活動的輸出設定為另一個活動的輸入，才可將它們鏈結起來。 在目前的版本中，您可以循序鏈結管線內的活動。 您可以在活動定義中使用 **dependsOn** 屬性，將它與上游活動鏈結。 如需詳細資訊和範例，請參閱[管線和活動](concepts-pipelines-activities.md#multiple-activities-in-a-pipeline)及[分支和鏈結活動](tutorial-control-flow.md)。 

### <a name="branching-activities"></a>分支活動
在目前的版本中，您可以將管線內的活動分支。 [If-Condition 活動](control-flow-if-condition-activity.md)所提供的功能，與 `if` 陳述式在程式設計語言中提供的功能相同。 它能在條件評估為 `true` 時執行一系列的活動，並在條件評估為 `false` 時執行另一系列的活動。 如需分支活動的範例，請參閱[分支和鏈結活動](tutorial-control-flow.md)教學課程。

### <a name="parameters"></a>參數 
您可以定義管線層級的參數並傳遞引數，同時依需求或透過觸發程序叫用管線。 活動可以取用傳遞給管線的引數。 如需詳細資訊，請參閱[管線和觸發程序](concepts-pipeline-execution-triggers.md)。 

### <a name="custom-state-passing"></a>自訂狀態傳遞
活動輸出 (包括狀態) 可供管線中的後續活動取用。 例如，在活動的 JSON 定義中，您可以使用下列語法來存取前一個活動的輸出：`@activity('NameofPreviousActivity').output.value`。 使用這項功能，您可以建立工作流程，以在其中透過活動傳遞值。

### <a name="looping-containers"></a>迴圈容器
[ForEach 活動](control-flow-for-each-activity.md)可定義管線中重複的控制流程。 此活動可逐一查看整個集合，然後以迴圈執行指定的活動。 此活動的迴圈實作與程式設計語言中的 Foreach 迴圈結構相似。 

[Until](control-flow-until-activity.md) 活動所提供的功能，與 do-until 迴圈結構在程式設計語言中提供的功能相同。 它會以迴圈的方式執行一系列活動，直到與該活動相關聯的條件評估為 `true` 為止。 您可以在 Data Factory 中針對 until 活動指定逾時的值。  

### <a name="trigger-based-flows"></a>以觸發程序為基礎的流程
可依需求或依時鐘時間觸發管線。 [管線和觸發程序](concepts-pipeline-execution-triggers.md)一文有觸發程序的詳細資訊。 

### <a name="invoking-a-pipeline-from-another-pipeline"></a>從另一個管線叫用管線
[執行管線活動](control-flow-execute-pipeline-activity.md)允許 Data Factory 管線叫用另一個管線。

### <a name="delta-flows"></a>差異流程
ETL 模式中的重要使用案例是「差異載入」，其只會載入從管線上次反覆運算後變更的資料。 現行版本的新功能 (例如[查閱活動](control-flow-lookup-activity.md)、彈性排程及控制流程) 會以自然的方式啟用此使用案例。 如需具有逐步指示的教學課程，請參閱[教學課程：累加複製](tutorial-incremental-copy-powershell.md)。

### <a name="other-control-flow-activities"></a>其他控制流程活動
以下是目前的 Data Factory 版本所支援的其他控制流程活動。 

控制活動 | 說明
---------------- | -----------
[ForEach 活動](control-flow-for-each-activity.md) | 定義管線中重複的控制流程。 此活動用來逐一查看整個集合，然後以迴圈執行指定的活動。 此活動的迴圈實作與程式設計語言中的 Foreach 迴圈結構相似。
[Web 活動](control-flow-web-activity.md) | 從 Data Factory 管線呼叫自訂 REST 端點。 您可以傳遞資料集和連結服務，以供活動取用和存取。 
[查閱活動](control-flow-lookup-activity.md) | 讀取或查閱任何外部來源中的記錄或資料表名稱值。 此輸出可供後續活動進一步參考。 
[取得中繼資料活動](control-flow-get-metadata-activity.md) | 擷取 Azure Data Factory 中任何資料的中繼資料。 
[Wait 活動](control-flow-wait-activity.md) | 管線會暫停一段指定的時間。

## <a name="deploy-ssis-packages-to-azure"></a>將 SSIS 套件部署至 Azure 
您可以使用 Azure-SSIS 將 SSIS 工作負載移到雲端、使用目前的版本建立資料處理站，以及佈建 Azure-SSIS Integration Runtime。

Azure-SSIS 整合執行階段是一個完全受管理的 Azure VM (節點) 叢集，專門用來執行您的雲端 SSIS 套件。 佈建 Azure-SSIS 整合執行階段之後，您可以使用您用於將 SSIS 套件部署到內部部署 SSIS 環境的相同工具。 

例如，您可以使用 SQL Server Data Tools 或 SQL Server Management Studio，將 SSIS 套件部署到 Azure 上的此執行階段。 如需逐步指示，請參閱[將 SQL Server Integration Services 套件部署至 Azure](tutorial-create-azure-ssis-runtime-portal.md)教學課程。 

## <a name="flexible-scheduling"></a>彈性排程
在目前的 Data Factory 版本中，您不需要定義資料集可用性排程。 您可以定義觸發程序資源，以排程時鐘排程器範例中的管線。 您也可以將參數從觸發程序傳遞至管線，以取得彈性排程和執行模型。 

在現行 Data Factory 版本中的管線沒有時間執行「視窗」。 startTime、endTime 及 isPaused 的 Data Factory V1 概念已不存在於目前的 Data Factory 版本中。 如需如何在目前的 Data Factory 版本中建置而後排程管線的詳細資訊，請參閱[管線執行和觸發程序](concepts-pipeline-execution-triggers.md)。

## <a name="support-for-more-data-stores"></a>支援更多資料存放區
相較於 V1，目前的版本支援在更多資料存放區中來回複製資料。 如需支援的資料存放區清單，請參閱下列文章：

- [版本 1 - 支援的資料存放區](v1/data-factory-data-movement-activities.md#supported-data-stores-and-formats)
- [目前的版本 - 支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)

## <a name="support-for-on-demand-spark-cluster"></a>支援隨選 Spark 叢集
目前的版本支援建立隨選 Azure HDInsight Spark 叢集。 若要建立隨選 Spark 叢集，請在隨選 HDInsight 連結服務定義中將叢集類型指定為 Spark。 然後，您可以在管線中設定 Spark 活動，以使用此連結服務。 

在執行階段，活動執行後，Data Factory 服務會為您自動建立 Spark 叢集。 如需詳細資訊，請參閱下列文章：

- [現行 Data Factory 版本中的 Spark 活動](transform-data-using-spark.md)
- [Azure HDInsight 隨選連結服務](compute-linked-services.md#azure-hdinsight-on-demand-linked-service)

## <a name="custom-activities"></a>自訂活動
在 V1 中，您可實作 (自訂) DotNet 活動程式碼，做法是建立一個 .NET 類別庫專案，其中有一個類別會實作 IDotNetActivity 介面的 Execute 方法。 因此，您需要在 .Net Framework 4.5.2 中撰寫自訂程式碼，並在以 Windows 為基礎的 Azure Batch 集區節點中執行。 

在目前版本的自訂活動中，您不需要實作 .NET 介面。 您可以直接執行命令、指令碼，以及自己的自訂程式碼 (已編譯為可執行檔)。 

如需詳細資訊，請參閱 [Data Factory 與第 1 版的自訂活動差異](transform-data-using-dotnet-custom-activity.md#compare-v2-v1)。

## <a name="sdks"></a>SDK
 目前的 Data Factory 版本提供一組豐富的 SDK，可用來撰寫、管理及監視管線。

- **.NET SDK**：目前版本中的 .NET SDK 已更新。

- **PowerShell**：目前版本中的 PowerShell Cmdlet 已更新。 現行版本 Cmdlet 的名稱中會有 **DataFactoryV2**，例如 Get-AzureRmDataFactoryV2。 

- **Python SDK**：此 SDK 是目前版本的新增項目。

- **REST API**：目前版本中的 REST API 已更新。 

在目前的版本中更新的 SDK 不具備與 V1 用戶端的回溯相容性。 

## <a name="authoring-experience"></a>撰寫體驗

| &nbsp; | V2 | V1 |
| ------ | -- | -- | 
| Azure 入口網站 | [是](quickstart-create-data-factory-portal.md) | [是](data-factory-build-your-first-pipeline-using-editor.md) |
| Azure PowerShell | [是](quickstart-create-data-factory-powershell.md) | [是](data-factory-build-your-first-pipeline-using-powershell.md) |
| .NET SDK | [是](quickstart-create-data-factory-dot-net.md) | [是](data-factory-build-your-first-pipeline-using-vs.md) |
| REST API | [是](quickstart-create-data-factory-rest-api.md) | [是](data-factory-build-your-first-pipeline-using-rest-api.md) |
| Python SDK | [是](quickstart-create-data-factory-python.md) | 否 |
| Resource Manager 範本 | [是](quickstart-create-data-factory-resource-manager-template.md) | [是](data-factory-build-your-first-pipeline-using-arm.md) | 

## <a name="roles-and-permissions"></a>角色和權限

第 1 版 Data Factory 參與者角色可用來建立及管理現行版本的 Data Factory 資源。 如需詳細資訊，請參閱 [Data Factory 參與者](../role-based-access-control/built-in-roles.md#data-factory-contributor)。

## <a name="monitoring-experience"></a>監視體驗
在目前的版本中，您可以使用 [Azure 監視器](monitor-using-azure-monitor.md)來監視資料處理站。 新的 PowerShell Cmdlet 支援[整合執行階段](monitor-integration-runtime.md)的監視。 V1 和 V2 都支援透過可從 Azure 入口網站啟動的監視應用程式進行視覺化監視。


## <a name="next-steps"></a>後續步驟
依照下列快速入門中的逐步指示操作，以了解如何建立資料處理站：[PowerShell](quickstart-create-data-factory-powershell.md)[.NET](quickstart-create-data-factory-dot-net.md)[Python](quickstart-create-data-factory-python.md)[REST API](quickstart-create-data-factory-rest-api.md)。 
