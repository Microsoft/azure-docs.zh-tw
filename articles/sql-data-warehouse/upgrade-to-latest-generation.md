---
title: 升級至最新一代 Azure SQL 資料倉儲 | Microsoft Docs
description: 將「Azure SQL 資料倉儲」升級至最新一代 Azure 硬體和儲存體架構。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 08/22/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 9b2c5fa38d0ad21a374d3e5acee8534aeb9c0384
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45580543"
---
# <a name="optimize-performance-by-upgrading-sql-data-warehouse"></a>升級 SQL 資料倉儲可將效能發揮到極限
將「Azure SQL 資料倉儲」升級至最新一代 Azure 硬體和儲存體架構。

## <a name="why-upgrade"></a>為何要升級？
您現在可以在 Azure 入口網站中順暢地升級至 SQL 資料倉儲計算最佳化 Gen2 層。 如果您有計算最佳化 Gen1 層的資料倉儲，則建議您升級。 透過升級，您可以使用最新一代的 Azure 硬體和增強的儲存體架構。 您能夠利用更快的效能、更高的延展性，且無限制的單欄式儲存體。 

> [!VIDEO https://www.youtube.com/embed/9B2F0gLoyss]

## <a name="applies-to"></a>適用於
這項升級適用於計算最佳化 Gen1 層資料倉儲。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="before-you-begin"></a>開始之前
> [!NOTE]
> 如果您現有的計算最佳化 Gen1 層資料倉儲不是位於可使用計算最佳化 Gen2 層的區域，您可以透過 PowerShell [異地還原](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-restore-database-powershell#restore-from-an-azure-geographical-region)到支援的區域。
> 
>

1. 如果要升級的計算最佳化 Gen1 層資料倉儲已暫停，請[繼續執行資料倉儲](pause-and-resume-compute-portal.md)。
2. 請做好停機幾分鐘的準備。 



## <a name="start-the-upgrade"></a>開始升級

1. 在 Azure 入口網站中移至計算最佳化 Gen1 層資料倉儲，然後按一下 [升級至 Gen2]：![Upgrade_1](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_1.png)

2. 依預設，使用下列對應，根據計算最佳化 Gen1 層目前的效能層級，針對資料倉儲**選取建議的效能層級**：
    
   | 計算最佳化 Gen1 層 | 計算最佳化 Gen2 層 |
   | :----------------------: | :-------------------: |
   |      DW100 – DW1000      |        DW1000c        |
   |          DW1200          |        DW1500c        |
   |          DW1500          |        DW1500c        |
   |          DW2000          |        DW2000c        |
   |          DW3000          |        DW3000c        |
   |          DW6000          |        DW6000c        |

3. 在升級之前，請確定您的工作負載已完成執行並停止。 在您的資料倉儲重新上線成為計算最佳化 Gen2 層資料倉儲之前，會發生幾分鐘的停機時間。 按一下 [升級]。 計算最佳化 Gen2 層效能層級的價格目前在預覽版期間為半價：
    
   ![Upgrade_2](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_2.png)

4. 檢查 Azure 入口網站中的狀態來**監視您的升級**：

   ![Upgrade3](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_3.png)
   
   升級程序的第一個步驟會進行調整規模作業 (「升級 - 離線」)，期間所有工作階段都會終止，且連線將會予以捨棄。 
   
   升級程序的第二個步驟是資料移轉 (「升級 - 上線」)。 資料移轉是線上緩慢移動背景程序，會將單欄式資料緩慢地從舊的儲存體架構移至新的儲存體架構，以充分利用本機 SSD 快取。 在此期間，您的資料倉儲將會上線以進行查詢和載入。 您所有的資料無論已遷移與否，都將可供查詢。 資料移轉會依據您的資料大小、效能層級和資料行存放區的區段數目，而以不同的速率進行。 

5. **選擇性的建議：** 為了加速資料移轉背景程序，您可以用較大的 SLO 和資源類別在您會查詢的所有主要資料行存放區資料表上執行 [Alter Index rebuild](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-index)，立即強制執行資料移動。 相較於緩慢移動背景程序，這項作業會**離線**進行，因此可能需要幾個小時才能完成 (視資料表的數量和大小而定)；不過，資料移轉將會更快，接著一旦完成後，您便可使用高品質的資料列群組充分利用新的增強儲存體架構。 

下列查詢會產生必要的 Alter Index Rebuild 命令，以加速資料移轉程序：

```sql
SELECT 'ALTER INDEX [' + idx.NAME + '] ON [' 
       + Schema_name(tbl.schema_id) + '].[' 
       + Object_name(idx.object_id) + '] REBUILD ' + ( CASE 
                                                         WHEN ( 
                                                     (SELECT Count(*) 
                                                      FROM   sys.partitions 
                                                             part2 
                                                      WHERE  part2.index_id 
                                                             = idx.index_id 
                                                             AND 
                                                     idx.object_id = 
                                                     part2.object_id) 
                                                     > 1 ) THEN 
              ' PARTITION = ' 
              + Cast(part.partition_number AS NVARCHAR(256)) 
              ELSE '' 
                                                       END ) + '; SELECT ''[' + 
              idx.NAME + '] ON [' + Schema_name(tbl.schema_id) + '].[' + 
              Object_name(idx.object_id) + '] ' + ( 
              CASE 
                WHEN ( (SELECT Count(*) 
                        FROM   sys.partitions 
                               part2 
                        WHERE 
                     part2.index_id = 
                     idx.index_id 
                     AND idx.object_id 
                         = part2.object_id) > 1 ) THEN 
              ' PARTITION = ' 
              + Cast(part.partition_number AS NVARCHAR(256)) 
              + ' completed'';' 
              ELSE ' completed'';' 
                                                    END ) 
FROM   sys.indexes idx 
       INNER JOIN sys.tables tbl 
               ON idx.object_id = tbl.object_id 
       LEFT OUTER JOIN sys.partitions part 
                    ON idx.index_id = part.index_id 
                       AND idx.object_id = part.object_id 
WHERE  idx.type_desc = 'CLUSTERED COLUMNSTORE'; 
```



## <a name="next-steps"></a>後續步驟
您升級的資料倉儲已上線。 若要充分利用增強的架構，請參閱[適用於工作負載管理的資源類別](resource-classes-for-workload-management.md)。
 
