---
title: 使用 Power BI 來將 SQL 資料倉儲資料視覺化 | Microsoft Azure
description: 使用 Power BI 視覺化 SQL 資料倉儲資料
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 514516dcc28e99d03611c7db6ec09c7b4a48ff0e
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306658"
---
# <a name="visualize-data-with-power-bi"></a>使用 Power BI 視覺化資料
本教學課程會示範如何使用 Power BI 來連接到 SQL 資料倉儲，並建立一些基本的視覺效果。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Data-Warehouse-Sample-Data-and-PowerBI/player]
> 
> 

## <a name="prerequisites"></a>必要條件
若要逐步執行本教學課程，您需要：

* 預先載入 AdventureWorksDW 資料庫的 SQL 資料倉儲。 若要佈建資料倉儲，請參閱[建立 SQL 資料倉儲](create-data-warehouse-portal.md)並選擇載入範例資料。 如果您已經有資料倉儲但沒有範例資料，您可以[載入 WideWorldImportersDW](load-data-wideworldimportersdw.md)。

## <a name="1-connect-to-your-database"></a>1.連接到您的資料庫
若要開啟 Power BI 並連接到您的 AdventureWorksDW 資料庫：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下 [SQL 資料庫]  ，並選擇您的 AdventureWorks SQL 資料倉儲資料庫。
   
    ![尋找您的資料庫](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-find-database.png)
3. 按一下 [在 Power BI 中開啟] 按鈕。
   
    ![Power BI 按鈕](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-button.png)
4. 您現在應該會看到 SQL 資料倉儲連接頁面顯示您的資料庫網址。 按 [下一步]。
   
    ![Power BI 連接](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-connect-to-azure.png)
5. 輸入您的 Azure SQL 伺服器使用者名稱和密碼。
   
    ![Power BI 登入](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-sign-in.png)
6. 若要開啟資料庫，請在左刀鋒視窗上按一下 AdventureWorksDW 資料集。
   
    ![Power BI 開啟 AdventureWorksDW](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-open-adventureworks.png)

## <a name="2-create-a-report"></a>2.建立報表
您現在已準備好使用 Power BI 來分析 AdventureWorksDW 範例資料。 為了執行分析，AdventureWorksDW 有一個稱為 AggregateSales 的檢視。 這個檢視包含用來分析公司銷售的一些重要指標。

1. 若要根據郵遞區號建立銷售金額的對應圖，請在右手邊欄位窗格中按一下 AggregateSales 檢視以展開它。 按一下 [PostalCode] 和 [SalesAmount] 資料行來選取它們。
   
    ![Power BI 選取 AggregateSales](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-aggregatesales.png)
   
    Power BI 會自動辨識地理資料，並將它放入地圖中。
   
    ![Power BI 對應圖](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-map.png)

2. 這個步驟會建立橫條圖，顯示每個客戶收入的銷售金額。 若要建立橫條圖，請移至展開的 AggregateSales 檢視。 按一下 [SalesAmount] 欄位。 將 [客戶收入] 欄位向左拖放到座標軸。
   
    ![Power BI 選取軸](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-chooseaxis.png)
   
    橫條圖在左邊。
   
    ![Power BI 長條圖](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-bar.png)
3. 這個步驟會建立折線圖，顯示每個訂單日期的銷售金額。 若要建立折線圖，請移至展開的 AggregateSales 檢視。 按一下 [SalesAmount] 和 [OrderDate]。 在 [視覺效果] 資料行中按一下 [折線圖] 圖示；這是視覺效果下第二行中的第一個圖示。
   
    ![Power BI 選取折線圖](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-prepare-line.png)
   
    您現在有一份報告，顯示資料的三種不同視覺效果。
   
    ![Power BI 折線圖](media/sql-data-warehouse-get-started-visualize-with-power-bi/pbi-line.png)

您也可以隨時按一下 [檔案]，並選取 [儲存] 來儲存您的進度。

## <a name="using-direct-connnect"></a>使用直接連接
在 Azure SQL Database 中，SQL Data Warehouse 直接連接允許邏輯下推以及 Power BI 的分析功能。 透過直接連接，查詢會在您瀏覽資料時即時傳送回到您的 Azure SQL 資料倉儲。  此功能與 SQL 資料倉儲結合的功能，可讓您在數分鐘內針對數 TB 的資料建立動態報表。 此外，引入 [在 Power BI 中開啟] 按鈕可讓使用者直接將 Power BI 連接到其 SQL 資料倉儲，而不需從其他 Azure 部分收集資訊。

使用直接連接時：

* 在連接時指定完整的伺服器名稱。
* 確定資料庫的防火牆規則都設定為 [允許存取 Azure 服務]。
* 每個動作 (例如選取資料行或新增篩選器) 會直接查詢資料倉儲。
* 大約每隔 15 分鐘會自動重新整理一次圖格。
* 問答集不適用於 Direct Connect 資料集。
* 會自動納入結構描述變更。
* 所有直接連接查詢都將在 2 分鐘後逾時。

隨著使用體驗改善，這些限制和助益事項可能會改變。

## <a name="next-steps"></a>後續步驟
既然我們已經提供您一些時間，讓您利用範例資料進入狀況，接著請查看如何進行[開發](sql-data-warehouse-overview-develop.md)、[載入](design-elt-data-loading.md)或[移轉](sql-data-warehouse-overview-migrate.md)。 或者，看一下 [Power BI 網站](http://www.powerbi.com/)。
