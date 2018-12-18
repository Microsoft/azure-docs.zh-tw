---
title: 使用 Azure 備份來取代您的磁帶基礎結構
description: 了解 Azure 備份如何提供可讓您在 Azure 中備份和還原資料的磁帶式語意
services: backup
author: trinadhk
manager: vijayts
ms.service: backup
ms.topic: conceptual
ms.date: 1/10/2017
ms.author: saurse
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ff30dd0e4c7cadabddbeddc38c28a773db68d8ff
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606489"
---
# <a name="move-your-long-term-storage-from-tape-to-the-azure-cloud"></a>將您的長期儲存空間從磁帶移至 Azure 雲端
使用 Azure 備份和 System Center Data Protection Manager 的客戶可以進行下列操作：

* 依照最適合組織需求的排程備份資料。
* 長期保留備份資料
* 讓 Azure (而非磁帶) 成為長期保留需求的一部分。

此文章說明客戶如何啟用備份和保留原則。 使用磁帶處理長期保留需求的客戶，透過此功能的可用性現在擁有功能強大且可行的替代方案。 最新版的 Azure 備份已啟用此功能 (可從 [這裡](http://aka.ms/azurebackup_agent)取得)。 System Center DPM 客戶至少必須更新為 DPM 2012 R2 UR5，才能搭配使用 DPM 與 Azure 備份服務。

## <a name="what-is-the-backup-schedule"></a>什麼是備份排程？
備份排程會指出備份作業的頻率。 例如，下列畫面中的設定指出備份將於每日下午 6 點和午夜執行。

![每日排程](./media/backup-azure-backup-cloud-as-tape/dailybackupschedule.png)

客戶也可以排程每週備份。 例如，下列畫面中的設定指出備份將於每隔兩週的星期日和星期三上午 9:30 和 1:00 執行。

![每週排程](./media/backup-azure-backup-cloud-as-tape/weeklybackupschedule.png)

## <a name="what-is-the-retention-policy"></a>保留原則為何？
保留原則會指定備份必須儲存的期間。 除了僅針對所有備份點指定「一般原則」之外，客戶可以指定在進行備份時根據不同的保留原則。 例如，每日取得的備份點 (可充當作業復原點) 可保留 90 天。 在每一季結尾所取得的備份點 (供稽核之用) 可能需要保留較長的時間。

![保留原則](./media/backup-azure-backup-cloud-as-tape/retentionpolicy.png)

此原則中指定的「保留點」總數是 90 (每日保留點) + 40 (每季一點，連續 10 年) = 130。

## <a name="example--putting-both-together"></a>範例 – 將兩者綜合比較
![範例畫面](./media/backup-azure-backup-cloud-as-tape/samplescreen.png)

1. **每日的保留原則**：每日所進行的備份會儲存 7 天。
2. **每週的保留原則**：每天午夜和星期六下午 6 點所進行的備份會保留 4 週
3. **每月的保留原則**：每月最後一個星期六午夜和下午 6 點所進行的備份會保留 12 個月
4. **每年的保留原則**：每年三月最後一個星期六午夜所進行的備份會保留 10 年

上圖中計算「保留點」總數 (客戶可從該保留點還原資料) 的方式如下：

* 7 天中每天 2 個保留點 = 14 個復原點
* 4 週中每週 2 個保留點 = 8 個復原點
* 12 個月中每月 2 個保留點 = 24 個復原點
* 10 年中每年 1 個保留點 = 10 個復原點

因此復原點的總數是 56。

> [!NOTE]
> Azure 備份對於復原點數目沒有限制。
>
>

## <a name="advanced-configuration"></a>進階組態
按一下上方畫面中的 [修改]  ，客戶在指定保留排程時擁有更進一步的彈性。

![修改](./media/backup-azure-backup-cloud-as-tape/modify.png)

## <a name="next-steps"></a>後續步驟
如需有關 Azure Backup 的詳細資訊，請參閱：

* [Azure 備份的簡介](backup-introduction-to-azure-backup.md)
* [試用 Azure 備份](backup-try-azure-backup-in-10-mins.md)
