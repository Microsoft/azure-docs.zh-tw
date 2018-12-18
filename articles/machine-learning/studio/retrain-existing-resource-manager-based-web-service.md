---
title: 重新訓練現有的預測 Web 服務 | Microsoft Docs
description: 了解如何在 Azure Machine Learning 中重新定型模型，以及使用新定型的模型來更新 Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: cc4c26a2-5672-4255-a767-cfd971e46775
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/07/2017
ms.openlocfilehash: b06e3d742a0bed778dc7671128980708ba379e39
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39714888"
---
# <a name="retrain-an-existing-predictive-web-service"></a>重新定型現有的預測 Web 服務
本文件描述下列案例的重新定型程序︰

* 您有訓練實驗和預測實驗，您已部署為實際運作的 Web 服務。
* 您有新的資料想要讓預測 Web 服務用來執行其評分。

> [!NOTE]
> 若要部署新的 Web 服務，您必須在要部署 Web 服務的訂用帳戶中具備足夠的權限。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。

從您現有的 Web 服務和實驗開始，您必須請遵循下列步驟︰

1. 更新模型。
   1. 修改您的訓練實驗，允許 Web 服務輸入和輸出。
   2. 將訓練實驗部署為定型 Web 服務。
   3. 使用訓練實驗批次執行服務 (BES) 重新定型模型。
2. 使用 Azure Machine Learning PowerShell Cmdlet 來更新預測實驗。
   1. 登入您的 Azure Resource Manager 帳戶。
   2. 取得 Web 服務定義。
   3. 將 Web 服務定義匯出為 JSON。
   4. 在 JSON 中將參考更新為 ilearner blob。
   5. 將 JSON 匯入至 Web 服務定義。
   6. 使用新的 Web 服務定義更新 Web 服務。

## <a name="deploy-the-training-experiment"></a>部署訓練實驗
若要將訓練實驗部署為重新定型 Web 服務，您必須將 Web 服務輸入和輸出新增至模型。 藉由將 *Web 服務輸出* 模組連接至實驗的 *[培訓模型][train-model]* 模組，即可讓訓練實驗產生可在預測實驗中使用的定型模型。 如果您有「評估模型」模組，您也可以附加 Web 服務輸出，以取得評估結果做為輸出。

若要更新您的訓練實驗︰

1. 將「Web 服務輸入」模組連接至您的資料輸入 (例如，「清除遺漏資料」模組)。 通常，您會想要確保您的輸入資料與原始的訓練資料以相同方式處理。
2. 將 *Web 服務輸出* 模組連接至 *訓練模型* 模組的輸出。
3. 如果您有「評估模型」模組，且您想要輸出評估結果，請將「Web 服務輸出」模組連接至「評估模型」模組的輸出。

執行您的實驗。

接下來您必須將訓練實驗部署為可產生訓練模型與模型評估結果的 Web 服務。

在實驗畫布底端，按一下 [設定 Web 服務]，然後選取 [部署 Web 服務 [新式]]。 Azure Machine Learning Web Services 入口網站會開啟 [部署 Web 服務] 頁面。 輸入您的 Web 服務名稱，選擇付款方案，然後按一下 [部署] 。 您只能使用批次執行方法來建立定型模型。

## <a name="retrain-the-model-with-new-data-by-using-bes"></a>使用 BES 以新資料重新定型模型
在此範例中，我們使用 C# 建立重新訓練應用程式。 您也可以使用 Python 或 R 範例程式碼來完成這項工作。

若要呼叫重新定型 API：

1. 在 Visual Studio 中建立 C# 主控台應用程式：[新增] > [專案] > [Visual C#] > [Windows 傳統桌面] > [主控台應用程式 (.NET Framework)]。
2. 登入 Machine Learning Web 服務入口網站。
3. 按一下您要使用的 Web 服務。
4. 按一下 [取用] 。
5. 在 [取用] 頁面底部的 [範例程式碼] 區段，按一下 [批次]。
6. 複製可用於批次執行的範例 C# 程式碼，然後貼入 Program.cs 檔案。 請確定命名空間保持不變。

新增 NuGet 套件 Microsoft.AspNet.WebApi.Client，如註解中所述。 若要新增指向 Microsoft.WindowsAzure.Storage.dll 的參考，您可能需要先安裝 [Azure 儲存體服務的用戶端程式庫](https://www.nuget.org/packages/WindowsAzure.Storage)。

下列螢幕擷取畫面顯示 Azure Machine Learning Web 服務入口網站的 [取用]頁面。

![取用頁面][1]

### <a name="update-the-apikey-declaration"></a>更新 apikey 宣告
找到 **apikey** 宣告：

    const string apiKey = "abc123"; // Replace this with the API key for the web service

在 [取用] 頁面的 [基本取用資訊] 區段中，找到主索引鍵，將其複製到 **apiKey** 宣告。

### <a name="update-the-azure-storage-information"></a>更新 Azure 儲存體資訊
BES 範例程式碼會將檔案從本機磁碟機 (例如，C:\temp\CensusIpnput.csv) 上傳至 Azure 儲存體、加以處理後，再將結果寫回 Azure 儲存體。

執行實驗後，產生的工作流程應該如下：

![執行後產生的工作流程][4]

1. 登入 Azure 入口網站。
2. 在左側導覽中，按一下 [更多服務]，搜尋 [儲存體帳戶]，並加以選取。
3. 在儲存體帳戶的清單中，選取要儲存重新定型模型的帳戶。
4. 在左側導覽中，按一下 [存取金鑰]。
5. 複製並儲存 [主要存取金鑰]。
6. 在左側導覽中，按一下 [容器]。
7. 您可以使用現有容器，或建立新的容器並儲存名稱。

找到 StorageAccountName、StorageAccountKey、StorageContainerName 宣告，更新為您從入口網站儲存的值。

    const string StorageAccountName = "mystorageacct"; // Replace this with your Azure storage account name
    const string StorageAccountKey = "a_storage_account_key"; // Replace this with your Azure Storage key
    const string StorageContainerName = "mycontainer"; // Replace this with your Azure Storage container name

您也必須確保程式碼中指定的位置有輸入檔案。

### <a name="specify-the-output-location"></a>指定輸出位置
在要求承載中指定輸出位置時，必須將 RelativeLocation  中指定檔案的副檔名指定為 `ilearner`。 請參閱下列範例：

    Outputs = new Dictionary<string, AzureBlobDataReference>() {
        {
            "output1",
            new AzureBlobDataReference()
            {
                ConnectionString = storageConnectionString,
                RelativeLocation = string.Format("{0}/output1results.ilearner", StorageContainerName) /*Replace this with the location you want to use for your output file and a valid file extension (usually .csv for scoring results or .ilearner for trained models)*/
            }
        },

以下是重新定型輸出的範例︰

![重新定型輸出][6]

## <a name="evaluate-the-retraining-results"></a>評估重新定型結果
當您執行應用程式時，輸出會包含存取評估結果所需的 URL 和共用存取簽章權杖。

您可以組合 output2 輸出結果中的 BaseLocation、RelativeLocation、SasBlobToken (如之前重新訓練輸出的圖中所示)，再將完整 URL 貼入瀏覽器網址列，便能看到重新訓練模型的執行結果。

查看結果以判斷新的定型模型的執行效能是否良好並足以取代現有模型。

複製輸出結果中的 BaseLocation、RelativeLocation、SasBlobToken。

## <a name="retrain-the-web-service"></a>重新定型 Web 服務
當您重新定型新的 Web 服務時，可以更新預測性 Web 服務定義以參考新的定型模型。 Web 服務定義是 Web 服務定型模型的內部表示法，且不可直接修改。 請確定您要擷取的是預測性實驗而非訓練實驗的 Web 服務定義。

## <a name="sign-in-to-azure-resource-manager"></a>登入 Azure Resource Manager
您必須先在 PowerShell 環境中，使用 [Connect-AzureRmAccount](/powershell/module/azurerm.profile/connect-azurermaccount) Cmdlet 登入您的 Azure 帳戶。

## <a name="get-the-web-service-definition-object"></a>取得 Web 服務定義物件
接下來，呼叫 [Get-AzureRmMlWebService](https://docs.microsoft.com/powershell/module/azurerm.machinelearning/get-azurermmlwebservice) Cmdlet 取得 Web 服務定義物件。

    $wsd = Get-AzureRmMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'

若要判斷現有 Web 服務的資源群組名稱，執行不含任何參數的 Get-AzureRmMlWebService cmdlet 來顯示您訂用帳戶中的 Web 服務。 找出 Web 服務，再查看其 Web 服務識別碼。 資源群組的名稱是識別碼中的第四個元素，緊接在 resourceGroups  元素之後。 在下列範例中，資源群組名稱是 Default-MachineLearning-SouthCentralUS。

    Properties : Microsoft.Azure.Management.MachineLearning.WebServices.Models.WebServicePropertiesForGraph
    Id : /subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237
    Name : RetrainSamplePre.2016.8.17.0.3.51.237
    Location : South Central US
    Type : Microsoft.MachineLearning/webServices
    Tags : {}

或者，若要判斷現有 Web 服務的資源群組名稱，登入 Azure Machine Learning Web 服務入口網站。 選取 Web 服務。 資源群組名稱是 Web 服務 URL 的第五個元素，緊接在 resourceGroups  元素之後。 在下列範例中，資源群組名稱是 Default-MachineLearning-SouthCentralUS。

    https://services.azureml.net/subscriptions/<subcription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237


## <a name="export-the-web-service-definition-object-as-json"></a>將 Web 服務定義物件匯出為 JSON
若要將定義修改為定型模型以使用新定型的模型，您必須先使用 [Export-AzureRmMlWebService](https://docs.microsoft.com/powershell/module/azurerm.machinelearning/export-azurermmlwebservice) Cmdlet 將其匯出為 JSON 格式檔案。

    Export-AzureRmMlWebService -WebService $wsd -OutputFile "C:\temp\mlservice_export.json"

## <a name="update-the-reference-to-the-ilearner-blob"></a>將參考更新為 ilearner blob
在資產中，找出 [定型模型]，使用 ilearner blob 的 URI 更新 locationInfo 節點中的 uri 值。 URI 的產生方式為結合來自 BES 重新定型呼叫輸出的 BaseLocation 和 RelativeLocation。

     "asset3": {
        "name": "Retrain Sample [trained model]",
        "type": "Resource",
        "locationInfo": {
          "uri": "https://mltestaccount.blob.core.windows.net/azuremlassetscontainer/baca7bca650f46218633552c0bcbba0e.ilearner"
        },
        "outputPorts": {
          "Results dataset": {
            "type": "Dataset"
          }
        }
      },

## <a name="import-the-json-into-a-web-service-definition-object"></a>將 JSON 匯入至 Web 服務定義物件
您必須使用 [Import-AzureRmMlWebService](https://docs.microsoft.com/powershell/module/azurerm.machinelearning/import-azurermmlwebservice) Cmdlet 將修改過的 JSON 檔案轉換回可用來更新預測性實驗的 Web 服務定義物件。

    $wsd = Import-AzureRmMlWebService -InputFile "C:\temp\mlservice_export.json"


## <a name="update-the-web-service"></a>更新 Web 服務
最後，使用 [Update-AzureRmMlWebService](https://docs.microsoft.com/powershell/module/azurerm.machinelearning/update-azurermmlwebservice) Cmdlet 來更新預測性實驗。

    Update-AzureRmMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'

[1]: ./media/retrain-existing-arm-web-service/machine-learning-retrain-models-consume-page.png
[4]: ./media/retrain-existing-arm-web-service/machine-learning-retrain-models-programmatically-IMAGE04.png
[6]: ./media/retrain-existing-arm-web-service/machine-learning-retrain-models-programmatically-IMAGE06.png

<!-- Module References -->
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
