---
title: 從一個實驗建立多個模型 | Microsoft Docs
description: 使用 PowerShell 以相同演算法但不同的訓練資料集，建立多個機器學習服務模型和 Web 服務端點。
services: machine-learning
documentationcenter: ''
author: hning86
ms.author: haining
manager: mwinkle
editor: cgronlun
ms.assetid: 1076b8eb-5a0d-4ac5-8601-8654d9be229f
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/04/2017
ms.openlocfilehash: 17354891b50138911f36314620f0c826db4b5dac
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833660"
---
# <a name="create-many-machine-learning-models-and-web-service-endpoints-from-one-experiment-using-powershell"></a>使用 PowerShell，從一個實驗中建立許多機器學習服務模型和 Web 服務端點
以下是一個常見的機器學習服務問題︰您想要建立許多模型，這些模型具有相同的訓練工作流程，且使用相同的演算法。 但您想要以不同的訓練資料集作為輸入。 本文說明如何在 Azure Machine Learning Studio 中只使用單一實驗來大規模完成這項工作。

例如，假設您擁有一家全球性自行車出租加盟商。 您想要建立迴歸模型，根據歷史資料來預測出租需求。 您在世界各地有 1000 個租用地點，且您為每個地點收集了一個資料集。 其中包含多項重要功能，例如專屬於每個地點的日期、時間、天氣和流量。

您可以使用所有地點的所有資料集合併版本，將模型訓練一次。 但是，您的每個地點都有其獨特的環境。 較好的作法是使用每個地點的資料集，個別地訓練您的迴歸模型。 這樣一來，每個訓練的模型就會將店面大小、數量、地理位置、人口、自行車友善交通環境等因素納入考量。

這可能是最好的方法，但您不想在 Azure Machine Learning 中建立 1,000 個訓練實驗，各代表唯一的地點。 除了工作繁重，看起來也很沒效率，因為除了訓練資料集不同，每個實驗都有完全相同的元件。

所幸，您可以使用 [Azure Machine Learning 重新訓練 API](retrain-models-programmatically.md)，並利用 [Azure Machine Learning PowerShell](powershell-module.md) 將工作自動化，來達到此目的。

> [!NOTE]
> 為了加速執行範例，請將地點從 1,000 個減到 10 個。 但使用與 1,000 個地點相同的原理和程序。 不過，如果您想要從 1,000 個資料集來訓練，建議您以平行方式執行下列 PowerShell 指令碼。 相關做法已超出本文的範圍，但您可以在網際網路上找到 PowerShell 多執行緒的範例。  
> 
> 

## <a name="set-up-the-training-experiment"></a>設定訓練實驗
請使用 [Cortana Intelligence Gallery](http://gallery.cortanaintelligence.com) 中的[訓練實驗](https://gallery.cortanaintelligence.com/Experiment/Bike-Rental-Training-Experiment-1)範例。 在 [Azure Machine Learning Studio](https://studio.azureml.net) 工作區開啟此實驗。

> [!NOTE]
> 為了依照此範例進行，建議您使用標準工作區，而不是免費工作區。 您將為每個客戶建立一個端點 (總共 10 個端點)，而這需要標準工作區，因為免費工作區只能有 3 個端點。 如果您只有免費工作區，請變更指令碼，而僅允許有三個地點。
> 
> 

此實驗使用「匯入資料」  模組，從 Azure 儲存體帳戶匯入訓練資料集 *customer001.csv* 。 假設您已從所有自行車出租地點收集訓練資料集，並儲存在相同的 Blob 儲存體位置中，檔案名稱範圍從 *rentalloc001.csv* 至 *rentalloc10.csv*。

![映像](./media/create-models-and-endpoints-with-powershell/reader-module.png)

請注意，**Web 服務輸出**模組已新增至**訓練模型**模組。
當此實驗部署為 Web 服務時，與該輸出相關聯的端點會傳回格式為 .ilearner 檔案的訓練模型。

另請注意，您會設定 Web 服務參數，以定義**匯入資料**模組使用的 URL。 這可讓您使用參數來指定個別的訓練資料集，以訓練每個地點的模型。
您也可以透過其他方式完成此作業。 您可以使用 SQL 查詢和 Web 服務參數，從 SQL Azure 資料庫取得資料。 或者，您可以使用 **Web 服務輸入**模組，將資料集傳入 Web 服務。

![映像](./media/create-models-and-endpoints-with-powershell/web-service-output.png)

現在，讓我們使用預設值 *rental001.csv* 作為訓練資料集，執行此訓練實驗。 如果您檢視**評估**模組的輸出 (按一下輸出並選取 [視覺化])，您可以看到您取得適當的效能 *AUC* = 0.91。 此時，您已準備好根據此訓練實驗來部署 Web 服務。

## <a name="deploy-the-training-and-scoring-web-services"></a>部署訓練和評分 Web 服務
若要部署訓練 Web 服務，請按一下實驗畫布下方的 [設定 Web 服務] 按鈕，然後選取 [部署 Web 服務]。 將此 Web 服務命名為「自行車出租訓練」。

現在，您必須部署評分 Web 服務。
若要這樣做，請按一下畫布下方的 [設定 Web 服務]，然後選取 [預測性 Web 服務]。 這會建立評分實驗。
您必須做一些微幅調整，使其以 Web 服務的形式運作。 請從輸入資料中移除標籤資料行 "cnt"，並限制只輸出執行個體識別碼和對應的預測值。

為了省去這項工作的麻煩，您可以開啟資源庫中已備妥的[預測性實驗](https://gallery.cortanaintelligence.com/Experiment/Bike-Rental-Predicative-Experiment-1)。

若要部署 Web 服務，請執行預測性實驗，然後按一下畫布下方的 [部署 Web 服務]  按鈕。 將評分 Web 服務命名為「自行車出租評分」。

## <a name="create-10-identical-web-service-endpoints-with-powershell"></a>使用 PowerShell 建立 10 個相同的 Web 服務端點
此 Web 服務隨附一個預設端點。 但您不會使用預設端點，因為它無法更新。 您必須做的是另外建立 10 個端點，每個地點各一個端點。 您可以利用 PowerShell 來完成此作業。

首先，您應設定 PowerShell 環境：

    Import-Module .\AzureMLPS.dll
    # Assume the default configuration file exists and is properly set to point to the valid Workspace.
    $scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
    $trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'

接著，執行下列 PowerShell 命令：

    # Create 10 endpoints on the scoring web service.
    For ($i = 1; $i -le 10; $i++){
        $seq = $i.ToString().PadLeft(3, '0');
        $endpointName = 'rentalloc' + $seq;
        Write-Host ('adding endpoint ' + $endpointName + '...')
        Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
    }

現在，您已建立 10 個端點，而且全部都包含經過 *customer001.csv* 訓練的相同訓練模型。 您可以在 Azure 入口網站中加以檢視。

![映像](./media/create-models-and-endpoints-with-powershell/created-endpoints.png)

## <a name="update-the-endpoints-to-use-separate-training-datasets-using-powershell"></a>使用 PowerShell 更新端點來使用不同的訓練資料集
下一步是使用經過每個客戶的個別資料所特別訓練的模型來更新端點。 但首先您必須從**自行車出租訓練** Web 服務來產生這些模型。 讓我們回到 **自行車出租訓練** Web 服務。 您必須以 10 個不同的訓練資料集呼叫其 BES 端點 10 次，以產生 10 個不同的模型。 請使用 **InovkeAmlWebServiceBESEndpoint** PowerShell Cmdlet 來執行這項操作。

您也必須在 `$configContent` 中提供您 Blob 儲存體帳戶的認證。 也就是說，必須填寫欄位 `AccountName`、`AccountKey` 和 `RelativeLocation`。 `AccountName` 可以是您的其中一個帳戶名稱，如 **Azure 入口網站** ([儲存體] 索引標籤) 中所示。 按一下儲存體帳戶後，按下底部的 [管理存取金鑰] 按鈕，即可找到儲存體帳戶的 `AccountKey`，並複製*主要存取金鑰*。 `RelativeLocation` 是相對於您儲存體的路徑，其可儲存新模型。 例如，下列指令碼中的路徑 `hai/retrain/bike_rental/` 指向名為 `hai` 的容器，而 `/retrain/bike_rental/` 是子資料夾。 目前，您無法透過入口網站 UI 建立子資料夾，但有[數個 Azure 儲存體總管](../../storage/common/storage-explorers.md)可讓您執行這項操作。 建議您在儲存體中建立新的容器，以儲存新的訓練模型 (.iLearner 檔案)，如下所示︰從儲存體頁面中，按一下底部的 [新增] 按鈕，並將其命名為 `retrain`。 簡而言之，下列指令碼的必要變更係有關於 `AccountName`、`AccountKey` 和 `RelativeLocation` (:`"retrain/model' + $seq + '.ilearner"`)。

    # Invoke the retraining API 10 times
    # This is the default (and the only) endpoint on the training web service
    $trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
    $submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
    $apiKey = $trainingSvcEp.PrimaryKey;
    For ($i = 1; $i -le 10; $i++){
        $seq = $i.ToString().PadLeft(3, '0');
        $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
        $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
        Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
        Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
    }

> [!NOTE]
> BES 端點是這項作業唯一支援的模式。 RRS 無法用於產生訓練的模型。
> 
> 

如您所見，您並未建構 10 個不同的 BES 作業設定 json 檔案，而是以動態方式建立了設定字串。 接著，請將其傳給 **InvokeAmlWebServceBESEndpoint** Cmdlet 的 *jobConfigString* 參數。 您不需要在磁碟上保留複本。

如果一切順利，一段時間之後，您應該會在 Azure 儲存體帳戶中看到 10 個 .iLearner 檔案，從 *model001.ilearner* 至 *model010.ilearner*。 現在，您已準備好要使用 **Patch-AmlWebServiceEndpoint** PowerShell Cmdlet，根據這些模型來更新 10 個評分 Web 服務端點。 同樣地，請記住您只能修補稍早以程式設計方式建立的非預設端點。

    # Patch the 10 endpoints with respective .ilearner models
    $baseLoc = 'http://bostonmtc.blob.core.windows.net/'
    $sasToken = '<my_blob_sas_token>'
    For ($i = 1; $i -le 10; $i++){
        $seq = $i.ToString().PadLeft(3, '0');
        $endpointName = 'rentalloc' + $seq;
        $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
        Write-Host ('Patching endpoint ' + $endpointName + '...');
        Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
    }

這應該會非常快速地執行。 執行完成時，您即已成功建立 10 個預測性 Web 服務端點。 每個端點都將包含經過出租地點特有的資料集所訓練的訓練模型，全部出自於單一訓練實驗。 若要驗證這一點，您可以使用 **InvokeAmlWebServiceRRSEndpoint** Cmdlet 嘗試呼叫這些端點，並提供相同的輸入資料。 您應該會看到不同的預測結果，因為模型是以不同的訓練集所訓練的。

## <a name="full-powershell-script"></a>完整 PowerShell 指令碼
以下是完整原始程式碼的清單︰

    Import-Module .\AzureMLPS.dll
    # Assume the default configuration file exists and properly set to point to the valid workspace.
    $scoringSvc = Get-AmlWebService | where Name -eq 'Bike Rental Scoring'
    $trainingSvc = Get-AmlWebService | where Name -eq 'Bike Rental Training'

    # Create 10 endpoints on the scoring web service
    For ($i = 1; $i -le 10; $i++){
        $seq = $i.ToString().PadLeft(3, '0');
        $endpointName = 'rentalloc' + $seq;
        Write-Host ('adding endpoint ' + $endpontName + '...')
        Add-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -Description $endpointName     
    }

    # Invoke the retraining API 10 times to produce 10 regression models in .ilearner format
    $trainingSvcEp = (Get-AmlWebServiceEndpoint -WebServiceId $trainingSvc.Id)[0];
    $submitJobRequestUrl = $trainingSvcEp.ApiLocation + '/jobs?api-version=2.0';
    $apiKey = $trainingSvcEp.PrimaryKey;
    For ($i = 1; $i -le 10; $i++){
        $seq = $i.ToString().PadLeft(3, '0');
        $inputFileName = 'https://bostonmtc.blob.core.windows.net/hai/retrain/bike_rental/BikeRental' + $seq + '.csv';
        $configContent = '{ "GlobalParameters": { "URI": "' + $inputFileName + '" }, "Outputs": { "output1": { "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=<myaccount>;AccountKey=<mykey>", "RelativeLocation": "hai/retrain/bike_rental/model' + $seq + '.ilearner" } } }';
        Write-Host ('training regression model on ' + $inputFileName + ' for rental location ' + $seq + '...');
        Invoke-AmlWebServiceBESEndpoint -JobConfigString $configContent -SubmitJobRequestUrl $submitJobRequestUrl -ApiKey $apiKey
    }

    # Patch the 10 endpoints with respective .ilearner models
    $baseLoc = 'http://bostonmtc.blob.core.windows.net/'
    $sasToken = '?test'
    For ($i = 1; $i -le 10; $i++){
        $seq = $i.ToString().PadLeft(3, '0');
        $endpointName = 'rentalloc' + $seq;
        $relativeLoc = 'hai/retrain/bike_rental/model' + $seq + '.ilearner';
        Write-Host ('Patching endpoint ' + $endpointName + '...');
        Patch-AmlWebServiceEndpoint -WebServiceId $scoringSvc.Id -EndpointName $endpointName -ResourceName 'Bike Rental [trained model]' -BaseLocation $baseLoc -RelativeLocation $relativeLoc -SasBlobToken $sasToken
    }
