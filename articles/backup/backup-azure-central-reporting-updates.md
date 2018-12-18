---
title: 更新 Azure 備份中心報告內容套件
description: 有關 Power BI 中 Azure 備份內容套件更新的資訊
services: backup
documentationcenter: ''
author: adigan
manager: shivamg
editor: ''
ms.assetid: 59df5bec-d959-457d-8731-7b20f7f1013e
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: adigan;
ms.openlocfilehash: 07774234849d96b9a44678b68ca7b13b6b4830f9
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39266709"
---
# <a name="updating-azure-backup-central-reporting-content-pack"></a>更新 Azure 備份中心報告內容套件 

[Azure 備份內容套件](https://docs.microsoft.com/azure/backup/backup-azure-configure-reports#view-reports-in-power-bi)可用來集中檢視備份的相關報告。 內容套件會定期更新以新增更多功能和修正錯誤。 本文會引導您完成用來更新內容套件，以及延遲此更新和過往所發佈更新的步驟。

## <a name="how-to-get-updates-to-the-content-pack"></a>如何取得內容套件的更新

### <a name="to-get-the-updated-content-pack"></a>取得更新的內容套件
如果您未對內容套件進行過任何變更，則系統會自動更新內容套件。 若內容套件已變更，您除了會在 Power BI 中收到通知，也會收到同樣內容的電子郵件通知。 您可以選擇在方便時取得更新的內容套件。 

### <a name="to-delay-the-update"></a>延遲更新
最佳做法是將內容套件匯入到[自訂工作區](https://youtu.be/26zyOtyHPJM?t=1m57s)。 您現在可以編輯報告。
如上所述，如果內容套件已變更，您會在 Power BI 中看到通知。 您可以選擇在稍後取得此內容套件。 

## <a name="coming-soon"></a>敬請期待
   
Azure 備份內容套件正在更新，以便在目前的 MAB 和 Azure VM 備份支援外，再支援其他工作負載，例如在 IaaS VM 備份中支援作為 SQL，以及支援作為 SC DPM。 這表示您很快就能夠在一個集中位置檢視和分析所有備份資料。 [您也可以自訂報告](https://youtu.be/26zyOtyHPJM)以符合組織需求。

為了讓所有工作負載的報告更有意義，我們正在變更 Azure 備份內容套件所預先設定的一組報告。 您可以在此一窺即將推出的一組報告：

### <a name="summary"></a>總結
   
![總結](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-Summary.png)

### <a name="billing"></a>計費

![計費](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-Billing.png)

### <a name="compliance"></a>法規遵循

![法規遵循](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-Compliance.png)

### <a name="storage"></a>儲存體

![儲存體](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-Storage.png)

### <a name="backup-items"></a>備份項目
![BackupItems](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-BackupItem.png)

### <a name="alerts"></a>警示

![警示](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-Alerts.png)

### <a name="jobs"></a>工作

![工作](.\media\backup-azure-central-reporting\AzBackup-Central-Reporting-Jobs.png)
    

## <a name="next-steps"></a>後續步驟：

* [跨組織共用報告](https://youtu.be/26zyOtyHPJM)
* [Azure 備份常見問題集](backup-azure-backup-faq.md)
