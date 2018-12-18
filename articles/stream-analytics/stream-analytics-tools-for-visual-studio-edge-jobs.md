---
title: 適用於 Visual Studio 的 Azure 串流分析工具中的 Edge 作業
description: 本文說明如何使用適用於 Visual Studio 的串流分析工具來撰寫、偵錯及建立串流分析 Edge 作業。
services: stream-analytics
author: su-jie
ms.author: sujie
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/13/2018
ms.openlocfilehash: 5dc90a1334b525c02be3eae2985900ab07cf2e05
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696243"
---
# <a name="develop-stream-analytics-edge-jobs-using-visual-studio-tools"></a>使用 Visual Studio 工具來開發串流分析 Edge 作業

在本教學課程中，您可以了解如何使用 Visual Studio 的串流分析工具來撰寫、偵錯及建立串流分析 Edge 作業。 建立和測試作業之後，您可以前往 Azure 入口網站將其部署到您的裝置。 

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要下列必要條件：

* 安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)、[Visual Studio 2015](https://www.visualstudio.com/vs/older-downloads/) 或 [Visual Studio 2013 Update 4](https://www.microsoft.com/download/details.aspx?id=45326)。 支援 Enterprise (Ultimate/Premium)、Professional 和 Community 版本。 不支援 Express 版本。  

* 依照[安裝指示](stream-analytics-tools-for-visual-studio-edge-jobs.md)，安裝 Visual Studio 適用的串流分析工具。
 
## <a name="create-a-stream-analytics-edge-project"></a>建立串流分析 Edge 專案 

在 Visual Studio 中，選取 [檔案] > [新增] > [專案]。 巡覽至左側 [範本] 清單 > 展開 [Azure 串流分析] > [串流分析 Edge] > [Azure 串流分析 Edge 應用程式]。 為您的專案提供名稱、位置及解決方案名稱，然後選取 [確定]。

![新增 Edge 專案](./media/stream-analytics-tools-for-visual-studio-edge-jobs/new-edge-project.png)

建立專案之後，請巡覽至 [方案總管] 以檢視資料夾階層。

![方案總管檢視](./media/stream-analytics-tools-for-visual-studio-edge-jobs/edge-project-in-solution-explorer.png)

 
## <a name="choose-the-correct-subscription"></a>選擇正確的訂用帳戶

1. 在 Visual Studio 的 [檢視] 功能表中，選取 [伺服器總管]。  

2. 以滑鼠右鍵按一下 [Azure] > 選取 [連線到 Microsoft Azure 訂用帳戶] > 然後以您的 Azure 帳戶登入。

## <a name="define-inputs"></a>定義輸入

1. 從 [方案總管] 中，展開 [輸入] 節點，您應該會看到名為 **EdgeInput.json** 的輸入。 按兩下以檢視其設定。  

2. 請確定 [來源類型] 設定為 [資料流] > [來源] 設定為 [Edge 中樞] > [事件序列化格式] 設定為 [Json] > 且 [編碼] 設定為 [UTF8]。 (選擇性) 您可以將 [輸入別名] 重新命名，在此範例中，我們將它保持原狀。 如果您將輸入別名重新命名，請使用您定義查詢時指定的名稱。 選取 [儲存]  以儲存設定。  
   ![輸入組態](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-input-configuration.png)
 


## <a name="define-outputs"></a>定義輸出

1. 在 [方案總管] 中，展開 [輸出] 節點，您應該會看到名為 **EdgeOutput.json** 的輸出。 按兩下以檢視其設定。  

2. 請確定 [接收] 設定為 [Edge 中樞] > [事件序列化格式] 設定為 [Json] > [編碼] 設定為 [UTF8] > 且 [格式] 設定為 [陣列]。 (選擇性) 您可以將 [輸出別名] 重新命名，在此範例中，我們將它保持原狀。 如果您將輸出別名重新命名，請使用您定義查詢時指定的名稱。 選取 [儲存]  以儲存設定。 
   ![輸出組態](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-output-configuration.png)
 
## <a name="define-the-transformation-query"></a>定義轉換查詢

部署在 Edge 環境中的串流分析作業支援大部分的[串流分析查詢語言參考](https://msdn.microsoft.com/azure/stream-analytics/reference/stream-analytics-query-language-reference?f=255&MSPPError=-2147217396)，不過 Edge 作業尚未支援下列作業： 


|**類別**  | **命令**  |
|---------|---------|
|地理空間運算子 |<ul><li>CreatePoint</li><li>CreatePolygon</li><li>CreateLineString</li><li>ST_DISTANCE</li><li>ST_WITHIN</li><li>ST_OVERLAPS</li><li>ST_INTERSECTS</li></ul> |
|其他運算子 | <ul><li>PARTITION BY</li><li>TIMESTAMP BY OVER</li><li>DISTINCT</li><li>COUNT 運算子中的運算式參數</li><li>DATE 和 TIME 函式中的微秒</li><li>JavaScript UDA (對於在雲端上部署的作業而言，這項功能仍處於預覽狀態)</li></ul>   |

當您在入口網站建立 Edge 作業時，如果您未使用支援的運算子，編譯器將會自動警告您。

在您的 Visual Studio 中，請在查詢編輯器中定義下列轉換查詢 (**script.asaql 檔案**)

```sql
SELECT * INTO EdgeOutput
FROM EdgeInput 
```

## <a name="test-the-job-locally"></a>在本機測試作業

若要在本機測試查詢，您應該上傳範例資料。 您可以從 [GitHub 存放庫](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/Registration.json)下載註冊資料並將它儲存到本機電腦，以取得範例資料。 

1. 若要上傳範例資料，請以滑鼠右鍵按一下 [EdgeInput.json] 檔案 > 選擇 [新增本機輸入]  

2. 在快顯視窗中，從本機路徑 [瀏覽] 範例資料 > 選取 [儲存]。
   ![本機輸入組態](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-local-input-configuration.png)
 
3. 系統會自動在您的 inputs 資料夾中新增名為 **local_EdgeInput.json** 的檔案。  
4. 您可以在本機執行該檔案或提交至 Azure。 若要測試查詢 > 選取 [在本機執行]。  
   ![執行選項](./media/stream-analytics-tools-for-visual-studio-edge-jobs/run-options.png)
 
5. 命令提示字元視窗會顯示作業的狀態。 作業執行成功時，它會在您的專案資料夾路徑 "Visual Studio 2015\Projects\MyASAEdgejob\MyASAEdgejob\ASALocalRun\2018-02-23-11-31-42" 中建立一個類似 "2018-02-23-11-31-42" 的資料夾。 巡覽至資料夾路徑以檢視本機資料夾中的結果：

   您也可以登入 Azure 入口網站，並確認已建立作業。 

   ![結果資料夾](./media/stream-analytics-tools-for-visual-studio-edge-jobs/result-folder.png)

## <a name="submit-the-job-to-azure"></a>將作業提交至 Azure

1. 將作業提交至 Azure 之前，您必須連線到 Azure 訂用帳戶。 開啟 [伺服器總管] > 以滑鼠右鍵按一下 [Azure] > [連線到 Microsoft Azure 訂用帳戶] > 登入您的 Azure 訂用帳戶。  

2. 若要將作業提交至 Azure，請巡覽至查詢編輯器 > 選取 [提交至 Azure]。  

3. 快顯視窗隨即開啟，您可以在其中選擇更新現有的 Edge 作業或是新建新的 Edge 作業。 當您更新現有的作業時，它會取代所有的作業組態，在此案例中，您將發佈新的作業。 選取 [建立新的 Azure 串流分析作業] > 為您的作業輸入名稱 (例如，MyASAEdgeJob) > 選擇所需的 [訂用帳戶]、[資源群組] 和 [位置] > 選取 [提交]。

   ![提交至 Azure](./media/stream-analytics-tools-for-visual-studio-edge-jobs/submit-to-azure.png)
 
   現在已建立串流分析 Edge 作業，您可以參考[在 IoT Edge 上執行作業教學課程](stream-analytics-edge.md)，以了解如何將它部署到您的裝置。 

## <a name="manage-the-job"></a>管理作業 

您可以在 [伺服器總管] 中檢視作業的狀態和作業圖表。 從 [伺服器總管] > [串流分析] > 展開訂用帳戶以及您部署 Edge 作業的資源群組 > 您可以檢視狀態為 [已建立] 的 MyASAEdgejob。 展開作業節點並按兩下來開啟作業檢視。

![伺服器總管選項](./media/stream-analytics-tools-for-visual-studio-edge-jobs/server-explorer-options.png)
 
作業檢視視窗會為您提供諸如重新整理作業、刪除作業、從 Azure 入口網站開啟作業等作業。

![作業圖表和其他選項](./media/stream-analytics-tools-for-visual-studio-edge-jobs/job-diagram-and-other-options.png) 

## <a name="next-steps"></a>後續步驟

* [Azure Iot Edge 的詳細資訊](../iot-edge/about-iot-edge.md)
* [ASA IoT Edge 教學課程](../iot-edge/tutorial-deploy-stream-analytics.md)
* [使用這個調查將意見反應傳送給小組](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR2czagZ-i_9Cg6NhAZlH9ypUMjNEM0RDVU9CVTBQWDdYTlk0UDNTTFdUTC4u) 
