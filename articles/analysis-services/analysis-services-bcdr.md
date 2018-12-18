---
title: Azure Analysis Services 高可用性 | Microsoft Docs
description: 確保 Azure Analysis Services 的高可用性。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 42e270f26a4576014d3a08a0ab3c59808c65fc56
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37445461"
---
# <a name="analysis-services-high-availability"></a>Analysis Services 的高可用性
本文說明如何確保 Azure Analysis Services 伺服器的高可用性。 


## <a name="assuring-high-availability-during-a-service-disruption"></a>在服務中斷期間確保高可用性
雖然很罕見，但 Azure 資料中心也可能會有中斷的時候。 發生中斷時，業務可能只會中斷幾分鐘，也可能會持續幾小時。 高可用性最常透過伺服器備援來實現。 使用 Azure Analysis Services，您就可以藉由在一或多個區域建立其他的次要伺服器來實現備援。 在建立備援伺服器時，為了確保這些伺服器上的資料和中繼資料會與區域中已離線的伺服器同步，您可以︰

* 將模型部署到其他區域的備援伺服器。 此方法需要平行處理主要伺服器和備援伺服器的資料，以確保所有伺服器保持同步。

* 從主要伺服器[備份](analysis-services-backup.md)資料庫，並還原到備援伺服器上。 例如，您可以讓目標為 Azure 儲存體的夜間備份作業自動進行，並還原到其他區域的其他備援伺服器。 

不論是哪一種情況，如果您的主要伺服器發生中斷，您都必須將報告用戶端中的連接字串，變更為連線到不同區域資料中心的伺服器。 此變更作業應視為最後手段，只有在發生重大的區域資料中心中斷時才應考慮使用。 比較常見的情況是，您可能還未更新所有用戶端上的連線，裝載主要伺服器的資料中心就已從中斷狀態恢復為上線狀態。 

若要避免需要變更報告用戶端上的連接字串，您可以為主要伺服器建立伺服器[別名](analysis-services-server-alias.md)。 當主要伺服器關閉時，您可以變更別名以指向其他區域中的備援伺服器。 您可以藉由編碼主要伺服器上的端點健康情況檢查，以自動設定伺服器名稱的別名。 如果健康情況檢查失敗，相同的端點即可導向至其他區域中的備援伺服器。 

## <a name="related-information"></a>相關資訊
[備份與還原](analysis-services-backup.md)   
[管理 Azure Analysis Services](analysis-services-manage.md)   
[伺服器名稱別名](analysis-services-server-alias.md) 

