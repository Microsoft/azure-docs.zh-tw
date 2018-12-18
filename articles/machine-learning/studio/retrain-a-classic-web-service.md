---
title: 重新訓練傳統 Web 服務 | Microsoft Docs
description: 了解如何在 Azure Machine Learning 中以程式設計方式重新定型模型，以及使用新定型的模型來更新 Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: e36e1961-9e8b-4801-80ef-46d80b140452
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.openlocfilehash: 6fc03865185b97fb1f34028239f647f97d5bd315
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836869"
---
# <a name="retrain-a-classic-web-service"></a>重新訓練傳統 Web 服務
您部署的預測性 Web 服務是預設評分端點。 預設端點會與原始定型和計分實驗同步，因此無法取代預設端點的定型模型。 若要重新訓練 Web 服務，必須在 Web 服務新增新端點。 

## <a name="prerequisites"></a>先決條件
您必須已設定訓練實驗與預測性實驗，如[以程式設計方式重新訓練機器學習服務模型](retrain-models-programmatically.md)中所示。 

> [!IMPORTANT]
> 預測性實驗必須部署為傳統 Machine Learning Web 服務。 
> 
> 

如需關於部署 Web 服務的其他資訊，請參閱[部署 Azure Machine Learning Web 服務](publish-a-machine-learning-web-service.md)。

## <a name="add-a-new-endpoint"></a>新增端點
您部署的預測性 Web 服務包含預設評分端點，它會與原始定型和評分實驗定型的模型保持同步。 若要將您的 Web 服務更新為新訓練的模型，您必須建立新的評分端點。 

在能以定型模型更新的預測性 Web 服務上，建立新的評分端點︰

> [!NOTE]
> 請確定您將端點新增至預測性 Web 服務，而不是定型 Web 服務。 如果您正確部署定型和預測性 Web 服務，您應該會看到列出兩個不同的 Web 服務。 預測性 Web 服務應是以 "[predictive exp.]" 結尾。
> 
> 

有兩種方式可在 Web 服務新增端點︰

1. 以程式設計方式
2. 使用 Microsoft Azure Web 服務入口網站

### <a name="programmatically-add-an-endpoint"></a>以程式設計方式新增端點
您可以使用此 [GitHub 儲存機制](https://github.com/raymondlaghaeian/AML_EndpointMgmt/blob/master/Program.cs)提供的範例程式碼來新增評分端點。

### <a name="use-the-microsoft-azure-web-services-portal-to-add-an-endpoint"></a>使用 Microsoft Azure Web 服務入口網站新增端點
1. 在 Machine Learning Studio 中，按一下左側的 [Web 服務]。
2. 在 Web 服務儀表板底部，按一下 [管理端點預覽]。
3. 按一下 [新增] 。
4. 輸入新端點的名稱和描述。 選取記錄層級，以及是否啟用範例資料。 如需有關記錄的詳細資訊，請參閱 [為 Machine Learning Web 服務啟用記錄](web-services-logging.md)。

## <a name="update-the-added-endpoints-trained-model"></a>更新已新增端點的定型模型
若要完成重新定型程序，您必須更新新增端點的定型模型。

如果您是使用範例程式碼新增端點，這會包含說明 URL 的位置 (靠輸出中的 HelpLocationURL  值識別)。

擷取路徑 URL：

1. 將 URL 複製並貼到瀏覽器。
2. 按一下 [更新資源] 連結。
3. 複製 PATCH 要求的 POST URL。 例如︰
   
     修補 URL：https://management.azureml.net/workspaces/00bf70534500b34rebfa1843d6/webservices/af3er32ad393852f9b30ac9a35b/endpoints/newendpoint2

您現在可以使用定型模型來更新您先前建立的評分端點。

下列範例程式碼示範如何使用 *BaseLocation*、*RelativeLocation*、*SasBlobToken* 和 PATCH URL 來更新端點。

    private async Task OverwriteModel()
    {
        var resourceLocations = new
        {
            Resources = new[]
            {
                new
                {
                    Name = "Census Model [trained model]",
                    Location = new AzureBlobDataReference()
                    {
                        BaseLocation = "https://esintussouthsus.blob.core.windows.net/",
                        RelativeLocation = "your endpoint relative location", //from the output, for example: “experimentoutput/8946abfd-79d6-4438-89a9-3e5d109183/8946abfd-79d6-4438-89a9-3e5d109183.ilearner”
                        SasBlobToken = "your endpoint SAS blob token" //from the output, for example: “?sv=2013-08-15&sr=c&sig=37lTTfngRwxCcf94%3D&st=2015-01-30T22%3A53%3A06Z&se=2015-01-31T22%3A58%3A06Z&sp=rl”
                    }
                }
            }
        };

        using (var client = new HttpClient())
        {
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

            using (var request = new HttpRequestMessage(new HttpMethod("PATCH"), endpointUrl))
            {
                request.Content = new StringContent(JsonConvert.SerializeObject(resourceLocations), System.Text.Encoding.UTF8, "application/json");
                HttpResponseMessage response = await client.SendAsync(request);

                if (!response.IsSuccessStatusCode)
                {
                    await WriteFailedResponse(response);
                }

                // Do what you want with a successful response here.
            }
        }
    }

在端點儀表板上可以看到呼叫的 *apiKey* 與 *endpointUrl*。

*Resources* 中 *Name* 參數的值，應該符合預測性實驗中已儲存之訓練模型的「資源名稱」。 取得資源名稱：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表中的 [Machine Learning] 。
3. 在 [名稱] 下，按一下您的工作區，然後按一下 [Web 服務] 。
4. 在 [名稱] 下，按一下 [普查模型 [predictive exp.]] 。
5. 按一下您新增的端點。
6. 在端點儀表板中，按一下 [更新資源] 。
7. 在 Web 服務的 [更新資源 API 文件] 頁面，您可以在 [可更新的資源] 下找到 [資源名稱]。

如果在您完成端點更新之前 SAS 權杖已到期，您必須執行 GET 以作業識別碼取得新的權杖。

程式碼執行成功後，新的端點應該會在大約 30 秒後開始使用重新定型模型。

## <a name="summary"></a>總結
使用重新定型 API 可以更新預測性 Web 服務的定型模型，運用於如下案例︰

* 定期以新的資料重新定型模型。
* 散發模型給客戶，目的是要讓他們使用自己的資料重新定型模型。

## <a name="next-steps"></a>後續步驟
[Azure Machine Learning 傳統 Web 服務的重新訓練進行疑難排解](troubleshooting-retraining-models.md)

