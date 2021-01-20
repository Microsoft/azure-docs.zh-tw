---
title: 教學課程：開始使用專用 SQL 集區來分析資料
description: 在此教學課程中，您將使用 NYC 計程車範例資料來探索 SQL 集區的分析功能。
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: tutorial
ms.date: 12/31/2020
ms.openlocfilehash: 683da659dcfa07c0a105382f4cc93d1f4dfb21b5
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98219527"
---
# <a name="analyze-data-with-dedicated-sql-pools"></a>使用專用 SQL 集區來分析資料

Azure Synapse Analytics 可為您提供使用專用 SQL 集區分析資料的功能。 在此教學課程中，您將使用紐約市計程車資料來探索專用 SQL 集區的功能。

## <a name="load-the-nyc-taxi-data-into-sqlpool1"></a>將 NYC 計程車資料載入 SQLPOOL1 中

1. 在 Synapse Studio 中，流覽至 [ **開發** 中樞]，按一下 **+** 按鈕以新增資源，然後建立新的 SQL 腳本。
1. 選取在本教學課程的 [步驟 1](./get-started-create-workspace.md) 中建立的集區 ' SQLPOOL1 ' (集區，) 腳本上方的 [連接到] 下拉式清單中。
1. 輸入下列程式碼：
    ```
    CREATE TABLE [dbo].[Trip]
    (
        [DateID] int NOT NULL,
        [MedallionID] int NOT NULL,
        [HackneyLicenseID] int NOT NULL,
        [PickupTimeID] int NOT NULL,
        [DropoffTimeID] int NOT NULL,
        [PickupGeographyID] int NULL,
        [DropoffGeographyID] int NULL,
        [PickupLatitude] float NULL,
        [PickupLongitude] float NULL,
        [PickupLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
        [DropoffLatitude] float NULL,
        [DropoffLongitude] float NULL,
        [DropoffLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
        [PassengerCount] int NULL,
        [TripDurationSeconds] int NULL,
        [TripDistanceMiles] float NULL,
        [PaymentType] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
        [FareAmount] money NULL,
        [SurchargeAmount] money NULL,
        [TaxAmount] money NULL,
        [TipAmount] money NULL,
        [TollsAmount] money NULL,
        [TotalAmount] money NULL
    )
    WITH
    (
        DISTRIBUTION = ROUND_ROBIN,
        CLUSTERED COLUMNSTORE INDEX
    );

    COPY INTO [dbo].[Trip]
    FROM 'https://nytaxiblob.blob.core.windows.net/2013/Trip2013/QID6392_20171107_05910_0.txt.gz'
    WITH
    (
        FILE_TYPE = 'CSV',
        FIELDTERMINATOR = '|',
        FIELDQUOTE = '',
        ROWTERMINATOR='0X0A',
        COMPRESSION = 'GZIP'
    )
    OPTION (LABEL = 'COPY : Load [dbo].[Trip] - Taxi dataset');
    ```
1. 按一下 [執行] 按鈕以執行腳本。
1. 此腳本會在60秒內完成。 它會將2000000資料列的 NYC 計程車資料載入到名為 dbo 的資料表中 **。旅程**。

## <a name="explore-the-nyc-taxi-data-in-the-dedicated-sql-pool"></a>探索專用 SQL 集區中的紐約市計程車資料

1. 在 Synapse Studio 中，移至 [資料] 中樞。
1. 移至 [SQLPOOL1] > [資料表]。 
1. 以滑鼠右鍵按一下 [dbo.Trip] 資料表，然後選取 [新增 SQL 指令碼] > [選取前 100 個資料列]。
1. 請等到新 SQL 指令碼建立完成並開始執行。
1. 請注意，在 SQL 指令碼頂端，[連線至] 會自動設定成名為 **SQLPOOL1** 的 SQL 集區。
1. 將 SQL 指令碼的文字取代為此程式碼並加以執行。

    ```sql
    SELECT PassengerCount,
          SUM(TripDistanceMiles) as SumTripDistance,
          AVG(TripDistanceMiles) as AvgTripDistance
    FROM  dbo.Trip
    WHERE TripDistanceMiles > 0 AND PassengerCount > 0
    GROUP BY PassengerCount
    ORDER BY PassengerCount;
    ```

    此查詢會顯示總行程距離和平均路程距離與乘客數的關聯。
1. 在 [SQL 指令碼結果] 視窗中，將 [視圖] 變更為 [圖表]，以折線圖形式查看結果的視覺效果。
    
    > [!NOTE]
    > 您可以透過資料中樞的工具提示，來識別已啟用工作區的專用 SQL 集區 (先前稱為 SQL DW)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Spark 進行分析](get-started-analyze-spark.md)