---
title: 使用 Azure 資料庫移轉服務來監視移轉活動 | Microsoft Docs
description: 深入了解如何使用 Azure 資料庫移轉服務來監視移轉活動。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 08/27/2018
ms.openlocfilehash: 78ad7a503cb2c99b9dac19a5500a01c8f7b7bfc3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045393"
---
# <a name="monitor-migration-activity"></a>監視移轉活動
在本文中，您會學習如何在資料庫層級與資料表層級監視移轉的進度。

## <a name="monitor-at-the-database-level"></a>在資料庫層級監視
若要在資料庫層級監視活動，請檢視資料庫層級刀鋒視窗：

![資料庫層級刀鋒視窗](media\how-to-monitor-migration-activity\dms-database-level-blade.png)

> [!NOTE]
> 選取資料庫超連結，會顯示資料表清單及其移轉進度。

下表列出了資料庫層級刀鋒視窗上的欄位，並說明每個欄位相關聯的各種狀態值。

<table id='overview' class='overview'>
  <thead>
    <tr>
      <th class="x-hidden-focus"><strong>欄位名稱</strong></th>
      <th><strong>欄位子狀態</strong></th>
      <th><strong>說明</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3" class="ActivityStatus"><strong>活動狀態</strong></td>
      <td>執行中</td>
      <td>移轉活動執行中。</td>
    </tr>
    <tr>
      <td>Succeeded</td>
      <td>移轉活動成功，沒有問題。</td>
    </tr>
    <tr>
      <td>發生錯誤</td>
      <td>移轉失敗。 選取移轉詳細資料底下的「查看錯誤詳細資料」連結，以取得完整的錯誤訊息。</td>
    </tr>
    <tr>
      <td rowspan="4" class="Status"><strong>狀態</strong></td>
      <td>正在初始化</td>
      <td>DMS 正在設定移轉管線。</td>
    </tr>
    <tr>
      <td>執行中</td>
      <td>DMS 管線正在執行中，並且正在執行移轉。</td>
    </tr>
    <tr>
      <td>完成</td>
      <td>移轉已完成。</td>
    </tr>
    <tr>
      <td>Failed</td>
      <td>移轉失敗。 按一下移轉詳細資料以查看移轉錯誤。</td>
    </tr>
    <tr>
      <td rowspan="5" class="migration-details"><strong>移轉詳細資料</strong></td>
      <td>正在初始化移轉管線</td>
      <td>DMS 正在設定移轉管線。</td>
    </tr>
    <tr>
      <td>完整資料載入進行中</td>
      <td>DMS 正在執行初始載入。</td>
    </tr>
    <tr>
      <td>準備好進行完全移轉</td>
      <td>初始載入完成之後，DMS 會將資料庫標示為準備好進行完全移轉。 使用者應該檢查資料是否跟上持續同步。</td>
    </tr>
    <tr>
      <td>已套用所有變更</td>
      <td>初始載入和持續同步已完成。 資料庫成功完全移轉之後，也會發生此狀態。</td>
    </tr>
    <tr>
      <td>查看錯誤詳細資料</td>
      <td>按一下連結以顯示錯誤詳細資料。</td>
    </tr>
    <tr>
      <td rowspan="1" class="duration"><strong>Duration</strong></td>
      <td>N/A</td>
      <td>從移轉活動初始化到移轉完成或移轉失敗的總時間。</td>
    </tr>
     </tbody>
</table>

## <a name="monitor-at-table-level--quick-summary"></a>在資料表層級監視 - 快速摘要
若要在資料表層級監視活動，請檢視資料表層級刀鋒視窗。 刀鋒視窗的上半部會顯示在完整載入和累加式更新中，所遷移的資料列詳細數目。 

刀鋒視窗的下半部會列出資料表，並顯示移轉進度的快速摘要。

![資料表層級刀鋒視窗 - 快速摘要](media\how-to-monitor-migration-activity\dms-table-level-blade-summary.png)

下表描述了資料表層級詳細資料中所顯示的欄位。

| 欄位名稱        | 說明       |
| ------------- | ------------- |
| **完整載入已完成**      | 已完成完整資料載入的資料表數目。 |
| **完整載入已排入佇列**      | 已排入佇列以進行完整載入的資料表數目。      |
| **完整載入正在載入** | 失敗的資料表數目。      |
| **累加式更新**      | 套用到目標的資料列中，異動資料擷取 (CDC) 更新的數目。 |
| **累加式插入**      | 套用到目標的資料列中，CDC 插入的數目。      |
| **累加式刪除** | 套用到目標的資料列中，CDC 刪除的數目。      |
| **暫止的變更**      | 資料列中仍在等待要套用到目標的 CDC 數目。 |
| **已套用的變更**      | 套用到目標的資料列中，CDC 更新、插入及刪除的總數。      |
| **處於錯誤狀態的資料表** | 在移轉期間處於「錯誤」狀態的資料表數目。 在某些範例中，當系統在目標中識別到重複項目，或者資料不相容而無法載入目標資料表時，資料表會進入錯誤狀態。      |

## <a name="monitor-at-table-level--detailed-summary"></a>在資料表層級監視 - 詳細摘要
有兩個索引標籤，顯示完整載入和累加式資料同步的移轉進度。
    
![完整載入索引標籤](media\how-to-monitor-migration-activity\dms-full-load-tab.png)

![累加式資料同步索引標籤](media\how-to-monitor-migration-activity\dms-incremental-data-sync-tab.png)

下表描述了資料表層級移轉進度中所顯示的欄位。

| 欄位名稱        | 說明       |
| ------------- | ------------- |
| **狀態 - 同步處理**      | 持續同步正在執行中。 |
| **插入**      | 套用到目標的資料列中，CDC 插入的數目。      |
| 更新 | 套用到目標的資料列中，CDC 更新的數目。      |
| **刪除**      | 套用到目標的資料列中，CDC 刪除的數目。 |
| **套用項目總計**      | 套用到目標的資料列中，CDC 更新、插入及刪除的總數。 |
| **資料錯誤** | 此資料表中發生的資料錯誤數目。 錯誤的一些範例是「511：無法建立大小為 %d 的資料列，這個大小大於可允許的資料列大小上限 %d，8114：將資料類型 %ls 轉換為 %ls 時發生錯誤。」  客戶應該從 Azure 目標中的 attms_apply_exceptions 資料表查詢，以查看錯誤詳細資料。    |

> [!NOTE]
> 當資料庫完全移轉或重新啟動移轉時，插入、更新、刪除及套用項目總計的 CDC 值可能會減少。

## <a name="next-steps"></a>後續步驟
- 在 Microsoft [資料庫移轉指南](https://datamigration.microsoft.com/) \(英文\) 中檢閱移轉指引。