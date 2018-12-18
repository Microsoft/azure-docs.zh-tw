---
title: Azure Data Factory - 範例
description: 提供 Azure Data Factory 服務隨附範例的詳細資料。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: c0538b90-2695-4c4c-a6c8-82f59111f4ab
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 417425a6982ca92e486ccfb578131ebc898a0466
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046083"
---
# <a name="azure-data-factory---samples"></a>Azure Data Factory - 範例
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [Data Factory 的 PowerShell 範例](../samples-powershell.md)和 [Azure 程式碼範例資源庫中的程式碼範例](https://azure.microsoft.com/resources/samples/?service=data-factory)。


## <a name="samples-on-github"></a>GitHub 上的範例
[GitHub Azure-DataFactory 儲存機制](https://github.com/azure/azure-datafactory) 包含數個範例，可協助您快速運用 Azure Data Factory 服務，或修改指令碼並用在您自己的應用程式中。 Samples\JSON 資料夾包含常見案例的 JSON 程式碼片段。

| 範例 | 說明 |
|:--- |:--- |
| [ADF 逐步解說](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/ADFWalkthrough) |此範例提供一個端對端逐步解說，說明如何使用 Azure Data Factory 來處理記錄檔，以將來自記錄檔的資料轉換成深入解析。 <br/><br/>在此逐步解說中，Data Factory 管線會收集範例記錄檔、處理這些記錄檔並以參考資料充實記錄檔資料，然後轉換資料以評估最近展開之行銷活動的成效。 |
| [JSON 範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/JSON) |此範例提供常見案例的 JSON 範例。 |
| [Http 資料下載程式範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/HttpDataDownloaderSample) |此範例展示如何使用自訂的 .NET 活動，從 HTTP 端點將資料下載到「Azure Blob 儲存體」。 |
| [跨 AppDomain .NET 活動範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/CrossAppDomainDotNetActivitySample) |此範例可讓您撰寫不受 ADF 啟動器所使用之組件版本 (例如 WindowsAzure.Storage v4.3.0、Newtonsoft.Json v6.0.x 等) 限制的自訂 .NET 活動。 |
| [執行 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample) |此範例包含可用來叫用 RScript.exe 的 Data Factory 自訂活動。 此範例只能與您自己的 (非隨選) 且已安裝 R 的 HDInsight 叢集搭配運作。 |
| [在 HDInsight Hadoop 叢集上叫用 Spark 作業](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/Spark) |此範例示範如何使用 MapReduce 活動來叫用 Spark 程式。 Spark 程式只是將資料從一個 Azure Blob 容器複製到另一個。 |
| [使用 Azure Machine Learning 批次評分活動進行的 Twitter 分析](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/TwitterAnalysisSample-AzureMLBatchScoringActivity) |此範例示範如何使用 AzureMLBatchScoringActivity 來叫用執行 Twitter 情緒分析、評分、預測等的 Azure Machine Learning 模型。 |
| [使用自訂活動進行的 Twitter 分析](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/TwitterAnalysisSample-CustomC%23Activity) |此範例示範如何使用自訂的 .NET 活動來叫用執行 Twitter 情緒分析、評分、預測等的 Azure Machine Learning 模型。 |
| [Azure Machine Learning 的參數化管線](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/ParameterizedPipelinesForAzureML/) |此範例提供一個端對端 C# 程式碼來部署 N 條管線，為的是以不同的區域參數來評分和重新訓練每條管線，其中區域清單是來自此檔案隨附的 parameters.txt 檔案。 |
| [Azure 串流分析作業的參考資料重新整理](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/ReferenceDataRefreshForASAJobs) |此範例示範如何使用 Azure Data Factory 搭配「Azure 串流分析」來以參考資料執行查詢，並在排程上設定參考資料重新整理。 |
| [搭配內部部署 Hortonworks Hadoop 的混合式管線](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/HybridPipelineWithOnPremisesHortonworksHadoop) |此範例使用內部部署 Hadoop 叢集做為在 Data Factory 中執行作業的運算目標，就像您會在雲端新增其他運算目標 (例如以 HDInsight 為基礎的 Hadoop 叢集) 一樣。 |
| [JSON 轉換工具](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/JSONConversionTool) |此工具可讓您將 2015-07-01-preview 之前的 JSON 版本轉換成最新版本或 2015-07-01-preview (預設)。 |
| [U-SQL 範例輸入檔](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/U-SQL%20Sample%20Input%20File) |此檔案是 U-SQL 活動所使用的範例檔案。 |
| [刪除 blob 檔案](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/DeleteBlobFileFolderCustomActivity) | 本範例將示範 C# 檔案，該檔案可以作為 ADF 自訂 .net 活動的一部分，以在複製檔案之後從來源 Azure Blob 位置刪除檔案。|

## <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本
您可以在 GitHub 上找到下列適用於 Data Factory 的 Azure Resource Manager 範本。

| 範本 | 說明 |
| --- | --- |
| [從 Azure Blob 儲存體複製到 Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-blob-to-sql-copy) |部署此範本會建立 Azure Data Factory，其中有管線可將資料從指定的 Azure Blob 儲存體複製到 Azure SQL Database。 |
| [從 Salesforce 複製到 Azure Blob 儲存體](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-salesforce-to-blob-copy) |部署此範本會建立 Azure Data Factory，其中有管線可將資料從指定的 Salesforce 帳戶複製到 Azure Blob 儲存體。 |
| [在 Azure HDInsight 叢集上執行 Hive 指令碼來轉換資料](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-hive-transformation) |部署此範本會建立 Azure Data Factory，其中有管線可透過在 Azure HDInsight Hadoop 叢集上執行範例 Hive 指令碼來轉換資料。 |

## <a name="samples-in-azure-portal"></a>Azure 入口網站中的範例
您可以使用您 Data Factory 首頁上的 [範例管線] 磚，將範例管線及其相關實體 (資料集和連結的服務) 部署到您的 Data Factory 中。

1. 建立 Data Factory 或開啟現有的 Data Factory。 請參閱[使用 Data Factory 將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)，以取得建立 Data Factory 的步驟。
2. 在 Data Factory 的 [DATA FACTORY] 刀鋒視窗中，按一下 [範例管線] 磚。

    ![範例管線圖格](./media/data-factory-samples/SamplePipelinesTile.png)
3. 在 [範例管線] 刀鋒視窗中，按一下您想要部署的「範例」。

    ![範例管線刀鋒視窗](./media/data-factory-samples/SampleTile.png)
4. 指定範例的組態設定。 例如，您的 Azure 儲存體帳戶名稱和帳戶金鑰、Azure SQL 伺服器名稱、資料庫、使用者 ID 和密碼等。

    ![範例刀鋒視窗](./media/data-factory-samples/SampleBlade.png)
5. 完成指定組態設定之後，請按一下 [建立]  ，以建立/部署範例管線，以及管線所使用的連結服務/資料表。
6. 您會在之前於 [範例管線]  刀鋒視窗上按下的範例磚上，看到部署的狀態。

    ![部署狀態](./media/data-factory-samples/DeploymentStatus.png)
7. 當您在範例磚上看到 [部署成功] 訊息時，請關閉 [範例管線] 刀鋒視窗。  
8. 在 [DATA FACTORY]  刀鋒視窗上，您會看到連結的服務、資料集及管線已新增到您的 Data Factory。  

    ![Data Factory 刀鋒視窗](./media/data-factory-samples/DataFactoryBladeAfter.png)

## <a name="samples-in-visual-studio"></a>Visual Studio 中的範例
### <a name="prerequisites"></a>先決條件
您必須已在電腦上安裝下列項目：

* Visual Studio 2013 或 Visual Studio 2015
* 下載 Azure SDK for Visual Studio 2013 或 Visual Studio 2015。 瀏覽至 [Azure 下載頁面](https://azure.microsoft.com/downloads/)，然後按一下 [.NET] 區段中的 [VS 2013] 或 [VS 2015]。
* 下載適用於 Visual Studio 的最新 Azure Data Factory 外掛程式：[VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) 或 [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005)。 如果您使用的是 Visual Studio 2013，您也可以執行下列步驟來更新外掛程式：在功能表上，按一下 [工具] -> [擴充功能和更新] -> [線上] -> [Visual Studio 組件庫] -> [Microsoft Azure Data Factory Tools for Visual Studio] -> [更新]。

### <a name="use-data-factory-templates"></a>使用 Data Factory 範本
1. 在功能表上按一下 [檔案]，指向 [新增]，然後按一下 [專案]。
2. 在 [新增專案] 對話方塊中，執行下列步驟：

   1. 在 [範本] 底下選取 [DataFactory]。
   2. 在右窗格中選取 [Data Factory 範本]  。
   3. 輸入專案的 [名稱]  。
   4. 選取專案的 [位置]  。
   5. 按一下 [確定]。

      ![[新增專案] 對話方塊](./media/data-factory-samples/vs-new-project-adf-templates.png)
3. 在 [Data Factory 範本] 對話方塊中，從 [使用案例範本] 區段選取範例範本，然後按 [下一步]。 以下步驟將引導您完成 [客戶分析]  範本的使用。 其他範例的步驟均相去不遠。

    ![Data Factory 範本對話方塊](./media/data-factory-samples/vs-data-factory-templates-dialog.png)
4. 在 [Data Factory 組態] 對話方塊中，於 [Data Factory 基本] 頁面上按 [下一步]。
5. 在 [設定 Data Factory] 頁面上，執行下列步驟：
   1. 選取 [建立新的 Data Factory]。 您也可以選取 [使用現有的 Data Factory] 。
   2. 輸入 Data Factory 的 [名稱]  。
   3. 選取您要在其中建立 Data Factory 的 [Azure 訂用帳戶]  。
   4. 選取 Data Factory 的 [資源群組]  。
   5. 針對 [區域]，選取 [美國西部]、[美國東部] 或 [北歐]。
   6. 按 [下一步] 。
6. 在 [設定資料存放區] 頁面中，指定現有的 [Azure SQL Database] 和 [Azure 儲存體帳戶]，或建立資料庫/儲存體，然後按 [下一步]。
7. 在 [設定計算] 頁面中，選取預設值，然後按 [下一步]。
8. 在 [摘要] 頁面中，檢閱所有設定，然後按 [下一步]。
9. 在 [部署狀態] 頁面中，等候部署完成，然後按一下 [完成]。
10. 在 [方案總管] 中，以滑鼠右鍵按一下專案，再按一下 [發佈] 。
11. 如果您看到 [登入您的 Microsoft 帳戶] 對話方塊，請輸入具有 Azure 訂用帳戶的帳戶認證，然後按一下 [登入]。
12. 您應該會看到下列對話方塊：

    ![發佈對話方塊](./media/data-factory-build-your-first-pipeline-using-vs/publish.png)
13. 在 [設定 Data Factory] 頁面中，執行下列步驟：

    1. 確認 [使用現有的 Data Factory]  選項。
    2. 選取您在使用範本時已選取的 **Data Factory**。
    3. 按 [下一步]，切換至 [發佈項目] 頁面。 (如果 [下一步] 按鈕已停用，請按 **TAB** 來移出 [名稱] 欄位)。
14. 在 [發佈項目] 頁面上，確認所有 Data Factory 實體都已選取，並按 [下一步] 切換至 [摘要] 頁面。     
15. 檢閱摘要，然後按 [下一步] 開始部署程序，並檢視 [部署狀態]。
16. 在 [部署狀態]  頁面上，您應該會看到部署程序的狀態。 部署完成後按一下 [完成]。

如需有關使用 Visual Studio 來撰寫 Data Factory 實體並將其發行至 Azure 的詳細資料，請參閱 [建置您的第一個 Data Factory (Visual Studio)](data-factory-build-your-first-pipeline-using-vs.md) 。          
