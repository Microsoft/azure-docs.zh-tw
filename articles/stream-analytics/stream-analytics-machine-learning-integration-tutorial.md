---
title: Azure 串流分析與 Azure Machine Learning 整合
description: 本文說明如何使用使用者定義函式，快速設定與 Azure Machine Learning 整合的簡單 Azure 串流分析作業。
services: stream-analytics
author: jasonwhowell
ms.author: jasonh
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/16/2018
ms.openlocfilehash: 63648dfe02a0b5ed00d0a7206a6aabbe200f94c4
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31527907"
---
# <a name="performing-sentiment-analysis-by-using-azure-stream-analytics-and-azure-machine-learning"></a>使用 Azure 串流分析和 Azure Machine Learning 執行情感分析
本文說明如何快速設定簡單的 Azure 串流分析工作與 Azure Machine Learning 整合。 您要使用 Cortana 智慧資源庫的機器學習服務情感分析模型，來分析串流文字資料並即時判斷情感分數。 使用 Cortana Intelligence Suite 可讓您完成這項工作，而不需擔心建立情感分析模型的複雜性。

您可以將本文所學套用到以下這類案例：

* 分析串流 Twitter 資料的即時情感。
* 分析客戶與支援人員對談的記錄。
* 評估論壇、部落格和影片的註解。 
* 許多其他即時、預測評分的案例。

在真實世界的案例中，您會直接從 Twitter 資料流取得資料。 我們為了簡化教學課程撰寫這個部分，讓串流分析作業從 Azure Blob 儲存體中的 CSV 檔案取得推文。 您可以建立您自己的 CSV 檔案，或使用範例 CSV 檔案，如下圖所示：

![CSV 檔案的範例推文](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-figure-2.png)  

您建立的串流分析工作會將情感分析模型做為使用者定義的函數 (UDF) 套用到 Blob 存放區中的範例文字資料。 輸出 (情感分析的結果) 會以不同的 CSV 檔案寫入相同的 Blob 存放區。 

下圖示範這項設定。 請注意，如需更真實的案例，可將 Blob 儲存體更換為 Azure 事件中樞輸入內的串流 Twitter 資料。 此外，也可針對彙總情緒建置 [Microsoft Power BI](https://powerbi.microsoft.com/) 即時視覺效果。    

![串流分析機器學習服務整合概觀](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-figure-1.png)  

## <a name="prerequisites"></a>先決條件
開始之前，請確定您具有下列項目：

* 有效的 Azure 訂用帳戶。
* 內附資料的 CSV 檔。 您可以從 [GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/sampleinput.csv) 下載之前顯示的檔案，或者建立您自己的檔案。 本文中，假設您使用 GitHub 的檔案。

總的來說，若要完成本文示範的工作，您要執行下列作業︰

1. 建立 Azure 儲存體帳戶和 Blob 儲存體容器，並將 CSV 格式的輸入檔案上傳至容器。
3. 將 Cortana 智慧資源庫的情感分析模型加入 Azure Machine Learning 工作區，然後在 Machine Learning 工作區中將此模型部署為 Web 服務。
5. 建立以函數形式呼叫此 Web 服務的串流分析工作，以判斷所輸入文字的情感。
6. 啟動串流分析工作並查看輸出。

## <a name="create-a-storage-container-and-upload-the-csv-input-file"></a>建立儲存體容器並上傳 CSV 輸入檔
在此步驟中，您可以使用任何 CSV 檔案，例如從 GitHub 取得的檔案。

1. 在 Azure 入口網站中，按一下 [建立資源] > [儲存體] > [儲存體帳戶]。

2. 提供名稱 (在範例中為 `samldemo`)。 名稱只能使用小寫字母和數字，而且在整個 Azure 中必須是唯一的。 

3. 指定現有的資源群組，並指定位置。 對於位置，我們建議您在本教學課程中建立的所有資源都使用相同的位置。

    ![提供儲存體帳戶詳細資料](./media/stream-analytics-machine-learning-integration-tutorial/create-sa1.png)

4. 在 Azure 入口網站中，選取儲存體帳戶。 在 [儲存體帳戶] 刀鋒視窗中，按一下 [容器]，然後按一下 [+&nbsp;容器] 建立 Blob 儲存體。

    ![建立 Blob 容器](./media/stream-analytics-machine-learning-integration-tutorial/create-sa2.png)

5. 提供容器的名稱 (在範例中為 `azuresamldemoblob`)，並確認 [存取類型] 設為 [Blob]。 完成後，按一下 [ **確定**]。

    ![指定 Blob 容器詳細資料](./media/stream-analytics-machine-learning-integration-tutorial/create-sa3.png)

6. 在 [容器] 刀鋒視窗中，選取新的容器，這會開啟該容器的刀鋒視窗。

7. 按一下 [上傳] 。

    ![容器的 [上傳] 按鈕](./media/stream-analytics-machine-learning-integration-tutorial/create-sa-upload-button.png)

8. 在 [上傳 Blob] 刀鋒視窗中，上傳您稍早下載的 **sampleinput.csv** 檔案。 對於 [Blob 類型]，選取 [區塊 Blob]，將區塊大小設為 4 MB，這對本教學課程已經足夠。

9. 按一下刀鋒視窗底部的 [上傳] 按鈕。

## <a name="add-the-sentiment-analytics-model-from-the-cortana-intelligence-gallery"></a>新增 Cortana 智慧資源庫中的情緒分析模型

現在，Blob 中已有範例資料，您可以啟用 Cortana 智慧資源庫中的情感分析模型。

1. 移至 Cortana 智慧資源庫中的[預測情感分析模型](https://gallery.cortanaintelligence.com/Experiment/Predictive-Mini-Twitter-sentiment-analysis-Experiment-1) \(英文\) 頁面。  

2. 按一下 [在 Studio 中開啟]。  
   
   ![串流分析機器學習服務, 開啟 Machine Learning Studio](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-open-ml-studio.png)  

3. 登入以移至工作區。 選取位置。

4. 按一下頁面底部的 [執行]  。 程序會執行大約一分鐘的時間。

   ![在 Machine Learning Studio 中執行實驗](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-run-experiment.png)  

5. 程序成功執行之後，請選取頁面底部的 [部署 Web 服務]。

   ![在 Machine Learning Studio 中將實驗部署為 Web 服務](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-deploy-web-service.png)  

6. 若要驗證情感分析模型是否已準備好使用，請按一下 [測試] 按鈕。 提供文字輸入，例如「我喜歡 Microsoft」。 

   ![在 Machine Learning Studio 中測試實驗](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-test.png)  

    如果測試成功，您會看到類似以下範例的結果：

   ![Machine Learning Studio 中的測試結果](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-test-results.png)  

7. 在 [應用程式] 欄中，按一下 [Excel 2010 或舊版活頁簿] 連結以下載 Excel 活頁簿。 活頁簿包含您稍後設定串流分析作業所需的 API 金鑰和 URL。

    ![串流分析機器學習服務, 快速概覽](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-quick-glance.png)  


## <a name="create-a-stream-analytics-job-that-uses-the-machine-learning-model"></a>建立使用機器學習服務模型的串流分析工作

您現在可以建立從 Blob 儲存體的 CSV 檔案中讀取範例推文的串流分析工作。 

### <a name="create-the-job"></a>建立工作

1. 移至 [Azure 入口網站](https://portal.azure.com)。  

2. 按一下 [建立資源] > [物聯網] > [串流分析作業]。 

3. 命名工作 `azure-sa-ml-demo`、指定訂用帳戶、指定現有的資源群組或建立一個新的資源群組，並選取工作的位置。

   ![指定新串流分析工作的設定](./media/stream-analytics-machine-learning-integration-tutorial/create-job-1.png)
   

### <a name="configure-the-job-input"></a>設定工作輸入
工作會從您稍早上傳到 Blob 儲存體的 CSV 檔案取得輸入。

1. 作業建立之後，在作業刀鋒視窗的 [作業拓撲] 下，按一下 [輸入] 選項。    

2. 在 [輸入] 刀鋒視窗中，按一下 [新增資料流輸入] >[Blob 儲存體]。

3. 使用下列值填寫 [Blob 儲存體] 刀鋒視窗：

   
   |欄位  |值  |
   |---------|---------|
   |**輸入別名** | 使用名稱 `datainput`，並選取 [從您的訂用帳戶選取 Blob 儲存體]       |
   |**儲存體帳戶**  |  選取您稍早建立的儲存體帳戶。  |
   |**容器**  | 選取您稍早建立的容器 (`azuresamldemoblob`)        |
   |**事件序列化格式**  |  選取 [CSV]       |

   ![新工作輸入的設定](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-create-sa-input-new-portal.png)

4. 按一下 [檔案] 。

### <a name="configure-the-job-output"></a>設定工作輸出
工作會將結果傳送至取得輸入的相同 Blob 儲存體。 

1. 在作業刀鋒視窗的 [作業拓撲] 下，按一下 [輸出] 選項。  

2. 在 [輸出] 刀鋒視窗中，按一下 [新增] >[Blob 儲存體]，然後使用別名 `datamloutput` 新增輸出。 

3. 使用下列值填寫 [Blob 儲存體] 刀鋒視窗：

   |欄位  |值  |
   |---------|---------|
   |**輸出別名** | 使用名稱 `datamloutput`，並選取 [從您的訂用帳戶選取 Blob 儲存體]       |
   |**儲存體帳戶**  |  選取您稍早建立的儲存體帳戶。  |
   |**容器**  | 選取您稍早建立的容器 (`azuresamldemoblob`)        |
   |**事件序列化格式**  |  選取 [CSV]       |

   ![新工作輸出的設定](./media/stream-analytics-machine-learning-integration-tutorial/create-output2.png) 

4. 按一下 [檔案] 。   


### <a name="add-the-machine-learning-function"></a>新增機器學習服務函數 
稍早您將機器學習模型發佈至 Web 服務。 在此案例中，串流分析作業執行時，會將來自輸入的每則範例推文傳送至 Web 服務進行情感分析。 Machine Learning Web 服務會傳回情感 (`positive`、`neutral` 或 `negative`)，以及推文屬於正向的機率。 

在教學課程的本節中，您要在串流分析工作中定義函數。 叫用函數，可以將推文傳送至 Web 服務並得到回應。 

1. 請確定您有稍早下載的 Web 服務 URL 和 API 金鑰 (在 Excel 活頁簿中)。

2. 瀏覽至您的作業刀鋒視窗 > [函式] > [+ 新增] > [AzureML]

3. 使用下列值填寫 [Azure Machine Learning 函式] 刀鋒視窗：

   |欄位  |值  |
   |---------|---------|
   | **函式別名** | 使用名稱 `sentiment` 並選取 [手動提供 Azure Machine Learning 函式設定]，讓您能夠選擇輸入 URL 和金鑰。      |
   | **URL**| 貼上 Web 服務 URL。|
   |**金鑰** | 貼上 API 金鑰。 |
  
   ![將機器學習服務函數新增至串流分析工作的設定](./media/stream-analytics-machine-learning-integration-tutorial/add-function.png)  
    
4. 按一下 [檔案] 。

### <a name="create-a-query-to-transform-the-data"></a>建立查詢來轉換資料

串流分析使用宣告式、以 SQL 為基礎的查詢來檢查輸入並處理它。 在本節中，您要建立從輸入讀取每則推文的查詢，然後叫用機器學習服務函數來執行情感分析。 查詢接著會將結果傳送至您定義的輸出 (Blob 儲存體)。

1. 回到工作概觀刀鋒視窗。

2.  在 [工作拓撲] 下，按一下 [查詢] 方塊。

3. 輸入下列查詢：

    ```
    WITH sentiment AS (  
    SELECT text, sentiment(text) as result 
    FROM datainput  
    )  

    SELECT text, result.[Score]  
    INTO datamloutput
    FROM sentiment  
    ```    

    查詢會叫用您稍早建立的函數 (`sentiment`) 以便對輸入中的每則推文執行情感分析。 

4. 按一下 [儲存]  以儲存查詢。


## <a name="start-the-stream-analytics-job-and-check-the-output"></a>啟動串流分析工作並查看輸出

您現在可以啟動串流分析工作。

### <a name="start-the-job"></a>啟動工作
1. 回到工作概觀刀鋒視窗。

2. 按一下刀鋒視窗頂端的 [啟動]。

3. 在 [啟動工作] 中，選取 [自訂]，然後選取您將 CSV 檔上傳至 Blob 儲存體的前一天。 完成後，按一下 [啟動]。  


### <a name="check-the-output"></a>查看輸出
1. 讓工作執行幾分鐘，直到您在 [監視] 方塊中看到活動。 

2. 如果您有一般用來檢查 Blob 儲存體內容的工具，請使用該工具來檢查 `azuresamldemoblob` 容器。 或者，請在 Azure 入口網站中執行下列步驟：

    1. 在入口網站中，尋找 `samldemo` 儲存體帳戶，並在該帳戶內，尋找 `azuresamldemoblob` 容器。 您會在容器中看到兩個檔案：包含範例推文的檔案，和串流分析工作所產生的 CSV 檔案。
    2. 以滑鼠右鍵按一下所產生的檔案，然後選取 [下載]。 

   ![從 Blob 儲存體下載 CSV 工作輸出](./media/stream-analytics-machine-learning-integration-tutorial/download-output-csv-file.png)  

3. 開啟產生的 CSV 檔案。 您會看到類似下列範例的畫面：  
   
   ![串流分析機器學習服務, CSV 檢視](./media/stream-analytics-machine-learning-integration-tutorial/stream-analytics-machine-learning-integration-tutorial-csv-view.png)  


### <a name="view-metrics"></a>檢視計量
您也能檢視與 Azure Machine Learning 函數相關的度量。 下列與函數相關的計量資訊會顯示在工作刀鋒視窗的 [監視] 方塊中：

* **函數要求** 指出傳送至 Machine Learning Web 服務的要求數目。  
* **函數事件** 指出要求中的事件數目。 根據預設，每個對 Machine Learning Web 服務的要求最多可包含 1,000 個事件。  


## <a name="next-steps"></a>後續步驟

* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [整合 REST API 與 Machine Learning](stream-analytics-how-to-configure-azure-machine-learning-endpoints-in-stream-analytics.md)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)



