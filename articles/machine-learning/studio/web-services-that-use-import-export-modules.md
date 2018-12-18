---
title: 在 Azure Machine Learning Web 服務中使用匯入/匯出資料 | Microsoft Docs
description: 了解如何使用「匯入資料」及「匯出資料」模組來傳送和接收 Web 服務的資料。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 3a7ac351-ebd3-43a1-8c5d-18223903d08e
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/28/2017
ms.openlocfilehash: 27873930ebef75923088f8bf2170c8e6a383cfa8
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837407"
---
# <a name="deploying-azure-ml-web-services-that-use-data-import-and-data-export-modules"></a>部署使用資料匯入和資料匯出模組的 Azure ML Web 服務

當您建立預測性實驗時，通常會新增 Web 服務輸入和輸出。 當您部署實驗時，取用者可以透過輸入和輸出傳送和接收 Web 服務的資料。 對於某些應用程式，取用者的資料可能可以從資料摘要獲得，或者資料已經位於外部資料來源 (例如 Azure Blob 儲存體)。 在這些情況下，應用程式就不需要使用 Web 服務的輸入和輸出讀取和寫入資料。 而是可以改用「批次執行服務 (BES)」使用「匯入資料」模組從資料來源讀取資料，然後使用「匯出資料」模組將評分結果寫入不同的資料位置。

「匯入資料」和「匯出資料」模組可以讀取和寫入各種資料位置，例如，透過 HTTP 的 Web URL、Hive 查詢、Azure SQL Database、Azure 表格儲存體、Azure Blob 儲存體、資料摘要提供或內部部署的 SQL 資料庫。

本主題使用「範例 5︰二進位分類的訓練、測試、評估︰成人資料集」範例，並假設資料集已載入名為 censusdata 的 Azure SQL 資料表。

## <a name="create-the-training-experiment"></a>建立訓練實驗
當您開啟「範例 5︰二進位分類的訓練、測試、評估︰成人資料集」範例時，它會使用範例「成人收入普查二進位分類」資料集。 畫布中的實驗看起來類似下圖：

![實驗的初始組態。](./media/web-services-that-use-import-export-modules/initial-look-of-experiment.png)

從 Azure SQL 資料表讀取資料︰

1. 刪除資料集模組。
2. 在元件搜尋方塊中，輸入「匯入」。
3. 從結果清單中，將「匯入資料」  模組加入實驗畫布。
4. 連接「匯入資料」模組的輸出與「清除遺漏的資料」模組的輸入。
5. 在屬性窗格中，在 [資料來源]  in the  。
6. 在 [資料庫伺服器名稱]、[資料庫名稱]、[使用者名稱] 和 [密碼] 欄位中，輸入資料庫的適當資訊。
7. 在資料庫查詢欄位中，輸入下列查詢。
   
     select [age],
   
        [workclass],
        [fnlwgt],
        [education],
        [education-num],
        [marital-status],
        [occupation],
        [relationship],
        [race],
        [sex],
        [capital-gain],
        [capital-loss],
        [hours-per-week],
        [native-country],
        [income]
     from dbo.censusdata;
8. 在實驗畫布底端，按一下 [執行] 。

## <a name="create-the-predictive-experiment"></a>建立預測性實驗
接著，您要設定用來部署 Web 服務的預測性實驗。

1. 在實驗畫布底端，按一下 [設定 Web 服務]，然後選取 [預測性 Web 服務 [建議]]。
2. 從預測性實驗移除「Web 服務輸入」和「Web 服務輸出」模組。 
3. 在元件搜尋方塊中，輸入「匯出」。
4. 從結果清單中，將「匯出資料」  模組加入實驗畫布。
5. 連接「評分模型」模組的輸出與「匯出資料」模組的輸入。 
6. 在屬性窗格中，在資料目的地下拉式清單中選取 [Azure SQL Database]  。
7. 在 [資料庫伺服器名稱]、[資料庫名稱]、[伺服器使用者帳戶名稱] 和 [伺服器使用者帳戶密碼] 欄位中，輸入資料庫的適當資訊。
8. 在 [要儲存的資料行逗號分隔清單]  欄位中，輸入「評分標籤」。
9. 在 [資料表名稱] 欄位中，輸入 dbo.ScoredLabels。 如果資料表不存在，則在執行實驗或呼叫 Web 服務時會建立資料表。
10. 在 [資料表資料行的逗號分隔清單]  欄位中，輸入 ScoredLabels。

當您撰寫的應用程式呼叫最後一個 Web 服務時，您可能要在執行階段指定不同的輸入查詢或目的地資料表。 若要設定這些輸入和輸出，請使用 Web 服務參數功能設來定「匯入資料」模組的「資料來源」屬性以及「匯出資料」模式資料目的地屬性。  如需有關 Web 服務參數的詳細資訊，請參閱 Cortana Intelligence and Machine Learning Blog (Cortana 智慧與機器學習部落格) 上的 [AzureML Web Service Parameters (AzureML Web 服務參數)](https://blogs.technet.microsoft.com/machinelearning/2014/11/25/azureml-web-service-parameters/) 。

設定匯入查詢和目的地資料表的 Web 服務參數︰

1. 在「匯入資料」模組的屬性窗格中，按一下 [資料庫查詢] 欄位右上角的圖示，然後選取 [設為 Web 服務參數]。
2. 在「匯出資料」模組的屬性窗格中，按一下 [資料表名稱] 欄位右上角的圖示，然後選取 [設為 Web 服務參數]。
3. 在「匯出資料」模組屬性窗格的底部，在 [Web 服務參數] 區段中，按一下 [資料庫查詢] 並將它重新命名為「查詢」。
4. 按一下 [資料表名稱] 並將它重新命名為**資料表**。

完成之後，您的實驗看起來應該類似下圖：

![實驗的最終外觀。](./media/web-services-that-use-import-export-modules/experiment-with-import-data-added.png)

您現在可以將實驗部署為 Web 服務。

## <a name="deploy-the-web-service"></a>部署 Web 服務
您可以部署到傳統或新的 Web 服務。

### <a name="deploy-a-classic-web-service"></a>部署傳統 Web 服務
若要部署為傳統 Web 服務，並建立應用程式取用它︰

1. 在實驗畫布底端，按一下 [執行]。
2. 執行完成時，按一下 [部署 Web 服務]，然後選取 [部署 Web 服務 [傳統]]。
3. 在 Web 服務儀表板上，找出您的 API 金鑰。 複製並儲存起來供日後使用。
4. 在 [預設端點] 資料表中，按一下 [批次執行] 連結以開啟 [API 說明頁面]。
5. 在 Visual Studio 中建立 C# 主控台應用程式：[新增] > [專案] > [Visual C#] > [Windows 傳統桌面] > [主控台應用程式 (.NET Framework)]。
6. 在 [API 說明頁面] 的頁面底部找到 [範例程式碼]  區段。
7. 將 C# 範例程式碼複製並貼入 Program.cs 檔案，然後移除 Blob 儲存體的所有參考。
8. 使用先前儲存的 API 金鑰更新 *apiKey* 變數的值。
9. 找出要求宣告並更新傳入「匯入資料」和「匯出資料」模組的 Web 服務參數值。 在此情況下，您會使用原始的查詢，但定義新的資料表名稱。
   
        var request = new BatchExecutionRequest() 
        {           
            GlobalParameters = new Dictionary<string, string>() {
                { "Query", @"select [age], [workclass], [fnlwgt], [education], [education-num], [marital-status], [occupation], [relationship], [race], [sex], [capital-gain], [capital-loss], [hours-per-week], [native-country], [income] from dbo.censusdata" },
                { "Table", "dbo.ScoredTable2" },
            }
        };
10. 執行應用程式。 

執行完成時，新的資料表會加入包含評分結果的資料庫。

### <a name="deploy-a-new-web-service"></a>部署新的 Web 服務

> [!NOTE] 
> 若要部署新的 Web 服務，您必須在要部署 Web 服務的訂用帳戶中具備足夠的權限。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。 

部署為新的 Web 服務，並建立應用程式取用它︰

1. 在實驗畫布底端，按一下 [執行] 。
2. 執行完成時，按一下 [部署 Web 服務]，然後選取 [部署 Web 服務 [新]]。
3. 在 [部署實驗] 頁面上，輸入您 Web 服務的名稱並選取定價方案，然後按一下 [部署]。
4. 在 [快速入門] 頁面上，按一下 [取用]。
5. 在 [範例程式碼] 區段中，按一下 [批次]。
6. 在 Visual Studio 中建立 C# 主控台應用程式：[新增] > [專案] > [Visual C#] > [Windows 傳統桌面] > [主控台應用程式 (.NET Framework)]。
7. 將 C# 範例程式碼複製並貼入 Program.cs 檔案。
8. 使用位於 [基本取用資訊] 區段中的 [主索引鍵] 更新 *apiKey* 變數的值。
9. 找出 *scoreRequest* 宣告並更新傳入「匯入資料」和「匯出資料」模組的 Web 服務參數值。 在此情況下，您會使用原始的查詢，但定義新的資料表名稱。
   
        var scoreRequest = new
        {       
            Inputs = new Dictionary<string, StringTable>()
            {
            },
            GlobalParameters = new Dictionary<string, string>() {
                { "Query", @"select [age], [workclass], [fnlwgt], [education], [education-num], [marital-status], [occupation], [relationship], [race], [sex], [capital-gain], [capital-loss], [hours-per-week], [native-country], [income] from dbo.censusdata" },
                { "Table", "dbo.ScoredTable3" },
            }
        };
10. 執行應用程式。 

