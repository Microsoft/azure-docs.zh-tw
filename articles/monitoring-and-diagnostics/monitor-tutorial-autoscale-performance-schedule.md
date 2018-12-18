---
title: 根據效能資料或排程自動調整 Azure 資源規模
description: 使用計量資料與排程為應用程式服務方案建立自動調整規模設定
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: tutorial
ms.date: 12/11/2017
ms.author: ancav
ms.custom: mvc
ms.component: autoscale
ms.openlocfilehash: b63e1fa316e9ebeaa564731b8bb0bc3ed5ba9036
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35264056"
---
# <a name="create-an-autoscale-setting-for--azure-resources-based-on-performance-data-or-a-schedule"></a>根據效能資料或排程建立自動調整 Azure 資源規模設定

自動調整規模設定可讓您根據預先設定的條件新增/移除服務的執行個體。 這些設定可透過入口網站來建立。 此方法提供瀏覽器型使用者介面，可用於建立及設定自動調整規模設定。 

在本教學課程中，您將： 
> [!div class="checklist"]
> * 建立 Web 應用程式與 App Service 方案
> * 設定自動調整規模規則，以依據 Web 應用程式接收的要求數目相應縮小和相應放大
> * 觸發相應放大動作，觀察執行個體數目的增加
> * 觸發相應縮小動作，觀察執行個體的數目的減少
> * 清除資源

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-web-app-and-app-service-plan"></a>建立 Web 應用程式與 App Service 方案
1. 按一下左邊瀏覽窗格中的 [建立資源] 選項。
2. 搜尋並選取 [Web 應用程式] 項目，然後按一下 [建立]。
3. 選取如 *MyTestScaleWebApp*的應用程式名稱。 建立新資源群組 *myResourceGroup' 並將它放到您想要的資源群組中。

在數分鐘內，系統應該就會佈建您的資源。 在此教學課程剩餘部分使用 Web 應用程式與對應的 App Service 方案。

   ![在入口網站中建立新的應用程式服務](./media/monitor-tutorial-autoscale-performance-schedule/Web-App-Create.png)

## <a name="navigate-to-autoscale-settings"></a>瀏覽到自動調整規模設定
1. 從左邊的瀏覽窗格，選取 [監視器] 選項。 一旦頁面載入，請選取 [自動調整規模] 索引標籤。
2. 您訂用帳戶中支援自動調整規模的資源清單會列在這裡。 找出稍早在教學課程中建立的 App Service 方案，然後按一下它。

    ![瀏覽到自動調整規模設定](./media/monitor-tutorial-autoscale-performance-schedule/monitor-blade-autoscale.png)

3. 在自動調整規模設定上，按一下 [啟用自動調整規模] 按鈕。

以下數個步驟可協助您填寫自動調整規模畫面中的資料，如下圖所示：

   ![儲存自動調整規模設定](./media/monitor-tutorial-autoscale-performance-schedule/Autoscale-Setting-Save.png)

 ## <a name="configure-default-profile"></a>設定預設設定檔
1. 為動調整規模設定提供**名稱**。
2. 在預設設定檔中，確定 [縮放模式] 設定為 [調整為特定執行個體計數]。
3. 將執行個體計數設定為 **1**。 此設定可確保當沒有其他設定檔為使用中或作用中時，預設設定檔會將執行個體計數還原為 1。

  ![瀏覽到自動調整規模設定](./media/monitor-tutorial-autoscale-performance-schedule/autoscale-setting-profile.png)


## <a name="create-recurrance-profile"></a>建立循環設定檔

1. 按一下預設設定檔下的 [新增調整規模條件]。

2. 編輯此設定檔的 [名稱]，並將它指定為「星期一到星期五設定檔」。

3. 確定 [縮放模式] 設定為 [依據計量調整規模]。

4. 針對 [執行個體限制]，請將 [最小值] 設定為 '1'，將 [最大值] 設定為 '2'，並將 [預設值] 設定為 '1'。 此設定可確保此設定檔不會自動將服務的規模調整為少於 1 個執行個體或多於 2 個執行個體。 若設定檔沒有足夠的資料可做出決策，它會使用預設執行個體數目 (在此範例中為 1)。

5. 針對 [排程]，選取 [重複特定日期]。

6. 將設定檔設定為在星期一到星期五的 09:00 (太平洋時間) 到 18:00 (太平洋時間) 重複。 此設定可確保此設定檔只會在星期一到星期五的上午 9 點到下午 6 點為作用中並有效。 在所有其他時間，「預設」設定檔是自動調整規模設定所使用的設定檔。

## <a name="create-a-scale-out-rule"></a>建立相應放大規則

1. 在「星期一到星期五設定檔」中。

2. 按一下 [新增規則] 連結。

3. 將 [計量來源] 設定為 [其他資源]。 將 [資源類型] 設定為 [應用程式服務]，並將 [資源] 設定為稍早在此教學課程中建立的 Web 應用程式。

4. 將 [時間彙總] 設定為 [總計]，將 [計量名稱] 設定為 [要求]，並將 [時間粒度統計資料] 設定為 [加總]。

5. 將 [運算子] 設定為 [大於]，將 [閾值] 設定為 '10'，並將 [持續時間] 設定為 '5' 分鐘。

6. 在 [作業] 選取 [將計數增加]，在 [執行個體計數] 選取 '1'，並在 [緩和] 選取 '5' 分鐘。

7. 按一下 [新增]  按鈕。

此規則可確保若您的 Web 應用程式在 5 分鐘內收到超過 10 個要求，系統會在您的 App Service 方案中加入另一個執行個體，以管理負載。

   ![建立相應放大規則](./media/monitor-tutorial-autoscale-performance-schedule/Scale-Out-Rule.png)

## <a name="create-a-scale-in-rule"></a>建立相應縮小規則
除了相應放大規則之外，建議您也建立相應縮小規則。 擁有這兩種規則可確保您的資源不會過度佈建。 過度佈建表示執行的執行個體數目超過處理目前負載所需的數目。 

1. 在「星期一到星期五設定檔」中。

2. 按一下 [新增規則] 連結。

3. 將 [計量來源] 設定為 [其他資源]。 將 [資源類型] 設定為 [應用程式服務]，並將 [資源] 設定為稍早在此教學課程中建立的 Web 應用程式。

4. 將 [時間彙總] 設定為 [總計]，將 [計量名稱] 設定為 [要求]，並將 [時間粒度統計資料] 設定為 [平均]。

5. 將 [運算子] 設定為 [小於]，將 [閾值] 設定為 '5'，並將 [持續時間] 設定為 '5' 分鐘。

6. 在 [作業] 選取 [將計數減少]，在 [執行個體計數] 選取 '1'，並在 [緩和] 選取 '5' 分鐘。

7. 按一下 [新增]  按鈕。

    ![建立相應縮小規則](./media/monitor-tutorial-autoscale-performance-schedule/Scale-In-Rule.png)

8. **儲存**自動調整規模設定。

    ![儲存自動調整規模設定](./media/monitor-tutorial-autoscale-performance-schedule/Autoscale-Setting-Save.png)

## <a name="trigger-scale-out-action"></a>觸發相應放大動作
若要觸發剛建立之自動調整規模設定中的相應放大條件，Web 應用程式必須在 5 分鐘內收到超過 10 個要求。

1. 開啟瀏覽器視窗，並瀏覽到稍早在此教學課程中建立的 Web 應用程式。 您可以在 Azure 入口網站中瀏覽到您的 Web 應用程式資源並按一下 [概觀] 索引標籤中的 [瀏覽] 按鈕，以尋找您的 Web 應用程式 URL。

2. 快速連續重新載入該網頁 10 次以上。

3. 從左邊的瀏覽窗格，選取 [監視器] 選項。 一旦頁面載入，請選取 [自動調整規模] 索引標籤。

4. 從清單中，選取此教學課程中使用的 App Service 方案。

5. 在自動調整規模設定上，按一下 [執行歷程記錄] 索引標籤。

6. 您會看到一個圖表，該圖表反映 App Service 方案隨時間經過的執行個體計數。

7. 在數分鐘內，執行個體計數應該會從 1 提高為 2。

8. 在該圖表下，您會看到由此自動調整規模設定採取之每個調整規模動作的活動記錄項目。

## <a name="trigger-scale-in-action"></a>觸發相應縮小動作
若 Web 應用程式在 10 分鐘內收到的要求數目少於 5 個，則會觸發自動調整規模設定中的相應縮小條件。 

1. 確定沒有要求被送到您的 Web 應用程式。

2. 載入 Azure 入口網站。

3. 從左邊的瀏覽窗格，選取 [監視器] 選項。 一旦頁面載入，請選取 [自動調整規模] 索引標籤。

4. 從清單中，選取此教學課程中使用的 App Service 方案。

5. 在自動調整規模設定上，按一下 [執行歷程記錄] 索引標籤。

6. 您會看到一個圖表，該圖表反映 App Service 方案隨時間經過的執行個體計數。

7. 在數分鐘內，執行個體計數應該會從 2 降低為 1。 此程序至少需要 100 分鐘才能完成。  

8. 在該圖表下，所顯示的是由此自動調整規模設定採取之每個調整規模動作的對應活動記錄項目集合。

    ![檢視相應縮小動作](./media/monitor-tutorial-autoscale-performance-schedule/Scale-In-Chart.png)

## <a name="clean-up-resources"></a>清除資源

1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取在此教學課程中建立的 Web 應用程式。

2. 在您的資源頁面上，按一下 [刪除]在文字方塊中輸入**是**以確認刪除，然後按一下 [刪除]。

3. 接著，選取 App Service 方案資源並按一下 [刪除]。

4. 在文字方塊中輸入**是**以確認刪除，然後按一下 [刪除 ]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您：  
> [!div class="checklist"]
> * 建立 Web 應用程式與 App Service 方案
> * 設定自動調整規模規則，以依據 Web 應用程式接收的要求數目相應縮小和相應放大
> * 觸發相應放大動作，觀察執行個體數目的增加
> * 觸發相應縮小動作，觀察執行個體數目的減少
> * 清除資源


若要深入了解自動調整規模設定，請繼續閱讀[自動調整規模概觀](monitoring-overview-autoscale.md)。

> [!div class="nextstepaction"]
> [封存監視資料](monitor-tutorial-archive-monitoring-data.md)
