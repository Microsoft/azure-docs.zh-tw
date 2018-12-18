---
title: 使用 Azure Machine Learning 分析資料 | Microsoft Docs
description: 使用 Azure Machine Learning，根據 Azure 資料倉儲中儲存的資料建置預測性機器學習模型。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 4324b1ac343a0e2b77c21d7834beffae08403953
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43247521"
---
# <a name="analyze-data-with-azure-machine-learning"></a>使用 Azure Machine Learning 分析資料
> [!div class="op_single_selector"]
> * [Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md)
> * [Azure Machine Learning](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

本教學課程會使用 Azure Machine Learning，根據 Azure 資料倉儲中儲存的資料建置預測性機器學習模型。 具體來說，這會為 Adventure Work 建置鎖定目標的行銷活動，預測客戶是否可能購買自行車。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Integrating-Azure-Machine-Learning-with-Azure-SQL-Data-Warehouse/player]
> 
> 

## <a name="prerequisites"></a>必要條件
若要逐步執行本教學課程，您需要：

* 預先載入 AdventureWorksDW 範例資料的 SQL 資料倉儲。 若要進行佈建，請參閱[建立 SQL 資料倉儲][Create a SQL Data Warehouse]並選擇載入範例資料。 如果您已經有資料倉儲但沒有範例資料，您可以[手動載入範例資料][load sample data manually]。

## <a name="1-get-the-data"></a>1.取得資料
此資料位於 AdventureWorksDW 資料庫的 dbo.vTargetMail 檢視中。 如何讀取此資料：

1. 登入 [Azure Machine Learning Studio][Azure Machine Learning studio] 並按一下我的實驗。
2. 按一下 [+ 新增] 並選取 [空白實驗]。
3. 輸入您的實驗名稱：目標行銷。
4. 將 [讀取器]  模組從模組窗格拖曳到畫布上。
5. 在 [屬性] 窗格中指定 SQL 資料倉儲資料庫的詳細資料。
6. 指定資料庫 **查詢** 以利讀取感興趣的資料。

```sql
SELECT [CustomerKey]
  ,[GeographyKey]
  ,[CustomerAlternateKey]
  ,[MaritalStatus]
  ,[Gender]
  ,cast ([YearlyIncome] as int) as SalaryYear
  ,[TotalChildren]
  ,[NumberChildrenAtHome]
  ,[EnglishEducation]
  ,[EnglishOccupation]
  ,[HouseOwnerFlag]
  ,[NumberCarsOwned]
  ,[CommuteDistance]
  ,[Region]
  ,[Age]
  ,[BikeBuyer]
FROM [dbo].[vTargetMail]
```

按一下實驗畫布下方的 [執行]  ，以執行實驗。
![執行實驗][1]

實驗成功執行完畢之後，按一下讀取器模組底部的輸出連接埠，然後選取 [視覺化]  以查看匯入的資料。
![檢視匯入的資料][3]

## <a name="2-clean-the-data"></a>2.清除資料
若要清除資料，請卸除與模型無關的某些資料行。 作法：

1. 將 [專案資料行]  模組拖曳到畫布上。
2. 按一下 [屬性] 窗格中的 [啟動資料行選取器]  來指定您想要卸除的資料行。
   ![][4]
3. 排除兩個資料行：CustomerAlternateKey 和 GeographyKey。
   ![移除不必要的資料行][5]

## <a name="3-build-the-model"></a>3.建立模型
我們將會以 80-20 的比例分割資料：80% 訓練機器學習模型以及 20% 測試模型。 我們將使用「二元」演算法處理這個二進位分類問題。

1. 將 [分割]  模組拖曳至畫布。
2. 在 [屬性] 窗格中第一個輸出資料集的 [比例] 資料列輸入 0.8。
   ![將資料分成訓練集和測試集][6]
3. 將 [二元促進式決策樹]  模組拖曳至畫布。
4. 將 [培訓模型]  模組拖曳至畫布並指定輸入。 然後，按一下 [屬性] 窗格中的 [啟動資料行選取器]  。
   * 第一個輸入：ML 演算法。
   * 第二個輸入：用來將演算法定型的資料。
     ![連接培訓模型模組][7]
5. 選取 **BikeBuyer** 資料行做為要預測的資料行。
   ![選取要預測的資料行][8]

## <a name="4-score-the-model"></a>4.評分模型
現在，我們將測試模型如何在測試資料上執行。 我們將會比較我們選擇的演算法和不同的演算法，以查看何者的執行效果比較好。

1. 將 [分數模型]  拖曳至畫布。
    第一個輸入：定型模型，第二個輸入：測試資料 ![評分模型][9]
2. 將 [二元貝氏點機器]  拖曳至實驗畫布。 我們將會比較這個演算法和二元促進式決策樹的執行效果。
3. 複製定型模型和分數模型等模組並貼在畫布上。
4. 將 [評估模型]  模組拖曳至畫布以比較兩個演算法。
5. **執行** 實驗。
   ![執行實驗][10]
6. 按一下 [評估模型] 模組底部的輸出連接埠，然後按一下 [視覺化]。
   ![將評估結果視覺化][11]

提供的度量資訊包括 ROC 曲線、正確性-召回圖表和升力曲線。 查看這些度量，我們可以看到第一個模型的執行效果優於第二個。 若要查看第一個模型的預測內容，請按一下 [分數模型] 的輸出連接埠，再按一下 [視覺化]。
![將評分結果視覺化][12]

您會看到另外兩個資料行加入至您的測試資料集。

* 評分機率：客戶成為自行車購買者的可能性。
* 評分標籤：由模型完成的分類 – 是自行車購買者 (1) 或不是 (0)。 標籤的機率臨界值設定為 50%，而且可以調整。

比較 BikeBuyer (實際) 資料行和評分標籤 (預測)，您可以看到模型的執行效果。 在接下來的步驟中，您可以使用此模型預測新的客戶，並將其發佈為 web 服務，或將結果寫回 SQL 資料倉儲。

## <a name="next-steps"></a>後續步驟
若要深入了解如何建置預測性機器學習模型，請參閱 [Azure 上的機器學習服務簡介][Introduction to Machine Learning on Azure]。

<!--Image references-->
[1]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img1_reader.png
[2]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img2_visualize.png
[3]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img3_readerdata.png
[4]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img4_projectcolumns.png
[5]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img5_columnselector.png
[6]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img6_split.png
[7]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img7_train.png
[8]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img8_traincolumnselector.png
[9]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img9_score.png
[10]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img10_evaluate.png
[11]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img11_evalresults.png
[12]: media/sql-data-warehouse-get-started-analyze-with-azure-machine-learning/img12_scoreresults.png


<!--Article references-->
[Azure Machine Learning studio]:https://studio.azureml.net/
[Introduction to Machine Learning on Azure]:https://azure.microsoft.com/documentation/articles/machine-learning-what-is-machine-learning/
[load sample data manually]: sql-data-warehouse-load-sample-databases.md
[Create a SQL Data Warehouse]: sql-data-warehouse-get-started-provision.md
