---
title: 使用 Power BI 分析 Azure Data Lake Storage Gen1 中的資料 | Microsoft Docs
description: 使用 Power BI 分析儲存在 Azure Data Lake Storage Gen1 的資料
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: 57d19d27-e135-49d9-a7ea-46c48ef4e3bd
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 5db9d18a31af4d6b407fcd9172ac80fc6f93f085
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297174"
---
# <a name="analyze-data-in-azure-data-lake-storage-gen1-by-using-power-bi"></a>使用 Power BI 分析 Azure Data Lake Storage Gen1 中的資料
在本文中，您將了解如何使用 Power BI Desktop 分析儲存在 Azure Data Lake Storage Gen1 的資料並加以視覺化。

## <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Data Lake Storage Gen1 帳戶**。 請遵循[透過 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示。 本文假設您已經建立稱為 **myadlsg1** 的 Data Lake Storage Gen1 帳戶，並將範例資料檔案 (**Drivers.txt**) 上載到其中。 此範例檔案可從 [Azure Data Lake Git 儲存機制](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/Drivers.txt)進行下載。
* **Power BI Desktop**。 您可以從 [Microsoft 下載中心](https://www.microsoft.com/en-us/download/details.aspx?id=45331)下載此項目。 

## <a name="create-a-report-in-power-bi-desktop"></a>在 Power BI Desktop 中建立報表
1. 在您的電腦上啟動 Power BI Desktop。
2. 從 [首頁] 功能區中，按一下 [取得資料]，然後按一下 [其他]。 在 [取得資料] 對話方塊中，依序按一下 [Azure]、[Azure Data Lake Store] 和 [連線]。
   
    ![連線至 Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account.png "連線至 Data Lake Storage Gen1")
3. 如果您看到連接器處於開發階段的對話方塊，請選擇繼續。
4. 在 [Azure Data Lake Store] 對話方塊中，提供 Data Lake Storage Gen1 帳戶的 URL，然後按一下 [確定]。
   
    ![Data Lake Storage Gen1 的 URL](./media/data-lake-store-power-bi/get-data-lake-store-account-url.png "Data Lake Storage Gen1 的 URL")
5. 在下一個對話方塊中，按一下 [登入] 登入 Data Lake Storage Gen1 帳戶。 您將會被重新導向至組織的登入頁面。 遵循提示登入此帳戶。
   
    ![登入 Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-signin.png "登入 Data Lake Storage Gen1")
6. 順利登入之後，請按一下 [連線] 。
   
    ![連線至 Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-connect.png "連線至 Data Lake Storage Gen1")
7. 下一個對話方塊會顯示您已上載至 Data Lake Storage Gen1 帳戶的檔案。 驗證資訊，然後按一下 [載入] 。
   
    ![從 Data Lake Storage Gen1 載入資料](./media/data-lake-store-power-bi/get-data-lake-store-account-load.png "從 Data Lake Storage Gen1 載入資料")
8. 順利將資料載入至 Power BI 之後，即會在 [欄位]  索引標籤中看到下列欄位。
   
    ![匯入的欄位](./media/data-lake-store-power-bi/imported-fields.png "匯入的欄位")
   
    不過，若要分析資料及加以視覺化，請使用下列欄位中的資料：
   
    ![所需的欄位](./media/data-lake-store-power-bi/desired-fields.png "所需的欄位")
   
    在後續步驟中，我們將更新查詢，以轉換所需格式的已匯入資料。
9. 從 [首頁] 功能區中，按一下 [編輯查詢]。
   
    ![編輯查詢](./media/data-lake-store-power-bi/edit-queries.png "編輯查詢")
10. 在 [查詢編輯器] 的 [內容] 資料行下，按一下 [二進位]。
    
    ![編輯查詢](./media/data-lake-store-power-bi/convert-query1.png "編輯查詢")
11. 您將會看到檔案圖示，其代表您已上傳的 **Drivers.txt** 檔案。 在檔案上按一下滑鼠右鍵，然後按一下 [CSV] 。    
    
    ![編輯查詢](./media/data-lake-store-power-bi/convert-query2.png "編輯查詢")
12. 您應該會看到如下的輸出。 您的資料現在為可用來建立視覺效果的格式。
    
    ![編輯查詢](./media/data-lake-store-power-bi/convert-query3.png "編輯查詢")
13. 從 [首頁] 功能區中，按一下 [關閉並套用]，然後按一下 [關閉並套用]。
    
    ![編輯查詢](./media/data-lake-store-power-bi/load-edited-query.png "編輯查詢")
14. 更新查詢之後，[欄位]  索引標籤將會顯示可用於視覺效果的新欄位。
    
    ![更新的欄位](./media/data-lake-store-power-bi/updated-query-fields.png "更新的欄位")
15. 讓我們建立圓形圖，以代表指定國家/地區的每個城市中的驅動程式。 若要這麼做，請進行下列選擇。
    
    1. 從 [視覺效果] 索引標籤中，按一下圓形圖的符號。
       
        ![建立圓形圖](./media/data-lake-store-power-bi/create-pie-chart.png "建立圓形圖")
    2. 我們將使用的資料行是 [資料行 4]\(城市名稱) 和 [資料行 7]\(國家/地區名稱)。 將這些資料行從 [欄位] 索引標籤拖曳到 [視覺效果] 索引標籤 (如下所示)。
       
        ![建立視覺效果](./media/data-lake-store-power-bi/create-visualizations.png "建立視覺效果")
    3. 圓形圖現在應該與下面類似。
       
        ![圓形圖](./media/data-lake-store-power-bi/pie-chart.png "建立視覺效果")
16. 透過從頁面層級篩選中選取特定國家/地區，您現在可以看到所選國家/地區的每個城市中的驅動程式數目。 例如，在 [視覺效果] 索引標籤的 [頁面層級篩選] 下，選取 [巴西]。
    
    ![選取國家/地區](./media/data-lake-store-power-bi/select-country.png "選取國家/地區")
17. 圓形圖會自動更新以顯示巴西城市中的驅動程式。
    
    ![國家/地區中的驅動程式](./media/data-lake-store-power-bi/driver-per-country.png "每個國家的驅動程式")
18. 從 [檔案] 功能表中，按一下 [儲存] 將視覺效果儲存為 Power BI Desktop 檔案。

## <a name="publish-report-to-power-bi-service"></a>將報表發佈到 Power BI 服務
在 Power BI Desktop 中建立視覺效果之後，即可將它發佈到 Power BI 服務，與其他人共用。 如需如何執行的指示，請參閱[從 Power BI Desktop 發佈](https://powerbi.microsoft.com/documentation/powerbi-desktop-upload-desktop-files/)。

## <a name="see-also"></a>另請參閱
* [使用 Data Lake Analytics 分析 Data Lake Storage Gen1 中的資料](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

