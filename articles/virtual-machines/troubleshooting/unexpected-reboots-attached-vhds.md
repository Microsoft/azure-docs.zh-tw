---
title: 在 Azure VM 上對包含連結 VHD 的 VM 非預期重新開機進行疑難排解 | Microsoft Docs
description: 如何對 VM 的非預期重新開機進行疑難排解。
keywords: ssh 連線被拒, ssh 錯誤, azure ssh, ssh 連線失敗
services: virtual-machines
author: genlin
manager: cshepard
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.service: virtual-machines
ms.topic: article
ms.date: 05/01/2018
ms.author: genli
ms.openlocfilehash: cffc6166ab7d0864646421b35fbecafdd80eb27e
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411800"
---
# <a name="troubleshoot-unexpected-reboots-of-vms-with-attached-vhds"></a>針對包含連結 VHD 的 VM 非預期重新開機進行疑難排解

當 Azure 虛擬機器 (VM) 有大量的連結 VHD 位於同一個儲存體帳戶中時，您可能會超出個別儲存體帳戶的延展性目標，導致 VM 非預期地重新開機。 檢查儲存體帳戶的每分鐘計量 (**TotalRequests**/**TotalIngress**/**TotalEgress**)，查看是否有超出儲存體帳戶延展性目標的流量暴增情況。 如需如何判斷儲存體帳戶是否發生節流作業的協助，請參閱[計量顯示 PercentThrottlingError 增加](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#metrics-show-an-increase-in-PercentThrottlingError)。

一般來說，VHD 每次來自虛擬機器的個別流量輸入或輸出，都會在底層的分頁 Blob 上轉換為 **Get Page** 或 **Put Page** 作業。 因此，您可以在環境中使用預估的 IOPS，依據應用程式的特定行為調節單一儲存體帳戶中應該具備的 VHD 數量。 Microsoft 建議單一儲存體帳戶中的磁碟少於 40 個。 如需關於儲存體帳戶延展性目標的詳細資訊，尤其是您所使用之儲存體帳戶類型的要求率總計和頻寬總計，請參閱 [Azure 儲存體延展性和效能目標](../../storage/common/storage-scalability-targets.md)。

如果您超過儲存體帳戶的延展性目標，請將 VHD 放在多個的儲存體帳戶中，以減少每個個別帳戶中的活動。
