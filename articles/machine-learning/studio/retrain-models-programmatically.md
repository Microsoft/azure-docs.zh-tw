---
title: 以程式設計方式重新訓練機器學習服務模型 | Microsoft Docs
description: 了解如何在 Azure Machine Learning 中以程式設計方式重新定型模型，以及使用新定型的模型來更新 Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 7ae4f977-e6bf-4d04-9dde-28a66ce7b664
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.openlocfilehash: b2090b39991363ee2a5b2e12945d97dc0fa9f2b2
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835499"
---
# <a name="retrain-machine-learning-models-programmatically"></a>以程式設計方式重新定型機器學習服務模型
在此逐步解說中，您將學習如何以程式設計方式使用 C# 和 Machine Learning 批次執行服務來重新訓練 Azure Machine Learning Web 服務。

重新訓練模型之後，下列逐步解說會說明如何更新預測性 Web 服務中的模型︰

* 如果您是在 Machine Learning Web 服務入口網站中部署傳統 Web 服務，請參閱[重新訓練傳統 Web 服務](retrain-a-classic-web-service.md)。 
* 如果您是部署新式 Web 服務，請參閱[使用 Machine Learning Management PowerShell Cmdlet 重新訓練新的 Web 服務](retrain-new-web-service-using-powershell.md)。

如需重新訓練程序的概觀，請參閱[重新訓練 Machine Learning 模型](retrain-machine-learning-model.md)。

如果您要開始使用現有新的 Azure Resource Manager 型 Web 服務，請參閱 [Retrain an existing Predictive web service (重新訓練現有預測性 Web 服務)](retrain-existing-resource-manager-based-web-service.md)。

## <a name="create-a-training-experiment"></a>建立訓練實驗
針對這個範例，您將使用Microsoft Azure Machine Learning 範例當中的「範例 5：定型、測試、評估二進位分類：成人資料集」。 

建立實驗：

1. 登入 Microsoft Azure Machine Learning Studio。 
2. 按一下儀表板右下角的 [新增] 。
3. 從 Microsoft 範例中，選取 範例 5。
4. 若要重新命名此實驗，在頂端的實驗畫布上，選取實驗名稱 [範例 5：定型、測試、評估二進位分類：成人資料集]。
5. 輸入「普查模型」。
6. 在實驗畫布底端，按一下 [執行] 。
7. 按一下 [設定 Web 服務]，然後選取 [重新訓練 Web 服務]。 

以下顯示初始實驗。
   
   ![初始實驗。][2]


## <a name="create-a-predictive-experiment-and-publish-as-a-web-service"></a>建立預測性實驗並發佈為 Web 服務
接下來，您要建立預測性實驗。

1. 在實驗畫布底端，按一下 [設定 Web 服務]，然後選取 [預測性 Web 服務]。 這樣會將模型儲存為訓練模型，並新增 Web 服務輸入與輸出模組。 
2. 按一下 **[執行]**。 
3. 實驗完成執行之後，按一下 [部署 Web 服務 [傳統]] 或 [部署 Web 服務[新式]]。

> [!NOTE] 
> 若要部署新的 Web 服務，您必須在要部署 Web 服務的訂用帳戶中具備足夠的權限。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。 

## <a name="deploy-the-training-experiment-as-a-training-web-service"></a>將訓練實驗部署為訓練 Web 服務
若要重新訓練已訓練的模型，您必須將您建立的訓練實驗部署為重新訓練 Web 服務。 這個 Web 服務需要將「Web 服務輸出」模組連接到[定型模型][train-model]模組，才能產生新的定型模組。

1. 若要回到訓練實驗，按一下左窗格中的 [實驗] 圖示，然後按一下名為 [普查模型] 的實驗。  
2. 在 [搜尋實驗項目] 方塊中，輸入「Web 服務」。 
3. 將 [Web 服務輸入] 模組拖曳到實驗畫布，然後將其連接到 [清除遺漏資料] 模組。  這可確保您的訓練資料與原始的訓練資料以相同方式處理。
4. 將兩個 [Web 服務輸出] 模組拖曳到實驗畫布。 將 [訓練模組] 模組的輸出連接到其中一個，將 [評估模型] 模組的輸出連接到另一個。 [訓練模型] 的 Web 服務輸出將會提供新的已訓練模型。 連接到**評估模型**的輸出會傳回該模組的輸出，即執行結果。
5. 按一下 **[執行]**。 

接下來您必須將訓練實驗部署為可產生訓練模型與模型評估結果的 Web 服務。 若要這麼做，您的下一組動作取決於您使用傳統 Web 服務或新式 Web 服務。  

**傳統 Web 服務**

在實驗畫布底端，按一下 [設定 Web 服務]，然後選取 [部署 Web 服務 [傳統]]。 顯示的 Web 服務 [儀表板]  中也包含批次執行的 API 金鑰與 API 說明頁面。 只有批次執行方法能夠用來建立定型模型。

**新式 Web 服務**

在實驗畫布底端，按一下 [設定 Web 服務]，然後選取 [部署 Web 服務 [新式]]。 Web 服務的 Azure Machine Learning Web 服務入口網站會開啟 [部署 Web 服務] 頁面。 輸入您 Web 服務的名稱，選擇付款方案，然後按一下 [部署]。 只有批次執行方法能夠用來建立定型模型

不論傳統或新式，在實驗完成執行後，產生的工作流程應該看起來像這樣：

![執行後產生的工作流程。][4]



## <a name="retrain-the-model-with-new-data-using-bes"></a>使用 BES 以新資料重新定型模型
在此範例中，您使用 C# 建立重新訓練應用程式。 您也可以使用 Python 或 R 範例程式碼來完成這項工作。

呼叫重新定型 API：

1. 在 Visual Studio 中建立 C# 主控台應用程式：[新增] > [專案] > [Visual C#] > [Windows 傳統桌面] > [主控台應用程式 (.NET Framework)]。
2. 登入 Machine Learning Web 服務入口網站。
3. 如果您使用傳統 Web 服務，按一下 [傳統 Web 服務]。
   1. 按一下您要使用的 Web 服務。
   2. 按一下預設端點。
   3. 按一下 [取用] 。
   4. 在 [取用] 頁面底部的 [範例程式碼] 區段，按一下 [批次]。
   5. 繼續執行此程序的步驟 5。
4. 如果您是使用新式 Web 服務，請按一下 [Web 服務]。
   1. 按一下您要使用的 Web 服務。
   2. 按一下 [取用] 。
   3. 在 [取用] 頁面底部的 [範例程式碼] 區段，按一下 [批次]。
5. 複製可用於批次執行的範例 C# 程式碼，然後貼入 Program.cs 檔案；請確定命名空間保持不變。

新增 Nuget 套件 Microsoft.AspNet.WebApi.Client，如註解中所述。 若要新增指向 Microsoft.WindowsAzure.Storage.dll 的參考，您可能需要先安裝 Microsoft Azure 儲存體服務的用戶端程式庫。 如需詳細資訊，請參閱 [Windows 儲存體服務](https://www.nuget.org/packages/WindowsAzure.Storage)。

### <a name="update-the-apikey-declaration"></a>更新 apikey 宣告
找到 **apikey** 宣告。

    const string apiKey = "abc123"; // Replace this with the API key for the web service

在 [取用] 頁面的 [基本取用資訊] 區段中，找到主索引鍵，將其複製到 **apiKey** 宣告。

### <a name="update-the-azure-storage-information"></a>更新 Azure 儲存體資訊
BES 範例程式碼會將檔案從本機磁碟機 (例如 C:\temp\CensusIpnput.csv) 上傳至 Azure 儲存體、加以處理後，再將結果寫回 Azure 儲存體。  

若要完成此工作，您必須從 Azure 傳統入口網站擷取儲存體帳戶的儲存體帳戶名稱、金鑰和容器資訊，然後更新程式碼裡的對應值。 

1. 登入 Azure 傳統入口網站。
2. 在左側導覽中，按一下 [儲存體] 。
3. 在儲存體帳戶的清單中，選取要儲存重新定型模型的帳戶。
4. 按一下此頁面底部的 [管理存取金鑰] 。
5. 複製並儲存 **主要存取金鑰** ，然後關閉對話方塊。 
6. 按一下頁面頂端的 [容器] 。
7. 您可以使用現有容器，或建立新的容器並儲存名稱。

找到 *StorageAccountName*、*StorageAccountKey*、*StorageContainerName* 宣告，更新為您從 Azure 入口網站儲存的值。

    const string StorageAccountName = "mystorageacct"; // Replace this with your Azure Storage Account name
    const string StorageAccountKey = "a_storage_account_key"; // Replace this with your Azure Storage Key
    const string StorageContainerName = "mycontainer"; // Replace this with your Azure Storage Container name

您也必須確保程式碼中指定的位置有輸入檔案。 

### <a name="specify-the-output-location"></a>指定輸出位置
在要求承載中指定輸出位置時，必須將 RelativeLocation  中指定檔案的副檔名指定為 .ilearner。 

請參閱下列範例：

    Outputs = new Dictionary<string, AzureBlobDataReference>() {
        {
            "output1",
            new AzureBlobDataReference()
            {
                ConnectionString = storageConnectionString,
                RelativeLocation = string.Format("{0}/output1results.ilearner", StorageContainerName) /*Replace this with the location you would like to use for your output file, and valid file extension (usually .csv for scoring results, or .ilearner for trained models)*/
            }
        },

> [!NOTE]
> 輸出位置的名稱可能不同於此逐步解說中的名稱，取決於您新增 Web 服務輸出模組的順序。 因為您設定這個訓練實驗有兩個輸出，結果會包含這兩者的儲存位置資訊。  
> 
> 

![重新定型輸出][6]

圖 4：重新定型輸出。

## <a name="evaluate-the-retraining-results"></a>評估重新定型結果
當您執行應用程式時，輸出會包含存取評估結果所需的 URL 和 SAS 權杖。

您可以組合 *output2* 輸出結果中的 *BaseLocation*、*RelativeLocation*、*SasBlobToken* (如之前重新訓練輸出的圖中所示)，再將完整 URL 貼入瀏覽器網址列，便能看到重新訓練模型的執行結果。  

查看結果以判斷新的定型模型的執行效能是否良好並足以取代現有模型。

複製輸出結果中的 *BaseLocation*、*RelativeLocation*、*SasBlobToken*，您在重新訓練程序中將用到它們。

## <a name="next-steps"></a>後續步驟
如果您透過按一下 [部署 Web 服務 [傳統]] 部署預測性 Web 服務，請參閱[重新訓練傳統 Web 服務](retrain-a-classic-web-service.md)。

如果您透過按一下 [部署 Web 服務 [新式]] 部署預測性 Web 服務，請參閱[使用 Machine Learning Management PowerShell Cmdlet 重新訓練新的 Web 服務](retrain-new-web-service-using-powershell.md)。

<!-- Retrain a New web service using the Machine Learning Management REST API -->


[1]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE01.png
[2]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE02.png
[3]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE03.png
[4]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE04.png
[5]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE05.png
[6]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE06.png
[7]: ./media/retrain-models-programmatically/machine-learning-retrain-models-programmatically-IMAGE07.png


<!-- Module References -->
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
