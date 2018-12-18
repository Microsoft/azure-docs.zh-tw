---
title: Azure VM 的災害復原案例 | Microsoft Docs
description: 了解發生影響 Azure 虛擬機器的 Azure 服務中斷事件時該怎麼辦。
services: virtual-machines
documentationcenter: ''
author: kmouss
manager: jeconnoc
editor: ''
ms.assetid: 65272148-ff06-4bce-91f1-851d706d4d40
ms.service: virtual-machines
ms.workload: virtual-machines
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2017
ms.author: kmouss;aglick
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 70aec41c885ab81371f5318f7557b0e628ac3308
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30915410"
---
# <a name="what-to-do-in-the-event-that-an-azure-service-disruption-impacts-azure-vms"></a>發生影響 Azure VM 的 Azure 服務中斷事件時該怎麼辦
Microsoft 的同仁一向努力確保提供您需要的服務。 有時候因為不可抗力之影響，造成服務意外中斷。

Microsoft 為其服務提供服務等級協定 (SLA)，作為執行時間和連接承諾。 個別的 Azure 服務 SLA 位於 [Azure 服務等級協定](https://azure.microsoft.com/support/legal/sla/)。

Azure 已經有許多支援高可用性應用程式的內建平台功能。 如需這些服務的詳細資訊，請參閱 [Azure 應用程式的災害復原與高可用性](../resiliency/resiliency-disaster-recovery-high-availability-azure-applications.md)。

本文內含當整個地區因重大天災或大規模服務中斷而發生中斷的真實災害復原案例。 這些都是極其罕見的情況，但您還是必須對整個區域發生中斷的可能性有所準備。 如果整個區域的服務中斷，會暫時無法使用本機的備援資料複本。 如果已啟用異地複寫，會在不同的區域儲存額外三份 Azure 儲存體 Blob 和資料表。 如果發生全面性區域中斷或嚴重損壞，且主要區域無法復原時，Azure 會重新對應異地複寫區域的所有 DNS 項目。

為協助您處理這些罕見事件，我們提供以下 Azure 虛擬機器指引，以因應 Azure 虛擬機器應用程式部署所在的整個區域發生服務中斷的情況。

## <a name="option-1-initiate-a-failover-by-using-azure-site-recovery"></a>選項 1︰使用 Azure Site Recovery 起始容錯移轉
您可以為 VM 設定 Azure Site Recovery，如此一來，只要按一下花幾分鐘就能復原應用程式。 您可以複寫至所選擇的 Azure 區域，而不限於配對的區域。 您可以[複寫虛擬機器](https://aka.ms/a2a-getting-started)來開始進行。 您可以[建立復原方案](../site-recovery/site-recovery-create-recovery-plans.md)，來將應用程式的整個容錯移轉程序自動化。 您可以事先[測試容錯移轉](../site-recovery/site-recovery-test-failover-to-azure.md)，而不影響實際執行應用程式或進行中的複寫。 如果主要區域發生中斷，只要[起始容錯移轉](../site-recovery/site-recovery-failover.md)並將您的應用程式帶到目標區域即可。


## <a name="option-2-wait-for-recovery"></a>選項 2︰等待復原
在此情況下，您不需要採取任何動作。 但您要知道，我們正在努力還原服務的可用性。 您可以在 [Azure 服務健康狀態儀表板](https://azure.microsoft.com/status/)上看見目前的服務狀態。

如果發生中斷前尚未設定 Azure Site Recovery、讀取權限異地備援儲存體或異地備援儲存體，這便是您的最佳選項。 如果儲存 VM 虛擬硬碟 (VHD) 的儲存體帳戶已設定異地備援儲存體或讀取權限異地備援儲存體，您可以指望復原基本映像 VHD，並嘗試用它佈建新的 VM。 因為不保證能夠同步處理資料，所以這不是慣用的選項。 因此，不保證此選項可以運作。


> [!NOTE]
> 請注意，您完全無法控制這個程序，而且它只有在全區域服務中斷時才會發生。 因此，您也必須依賴其他的應用程式特定備份策略，以達到最高層級的可用性。 如需詳細資訊，請參閱 [災害復原的資料策略](https://docs.microsoft.com/azure/architecture/resiliency/disaster-recovery-azure-applications#data-strategies-for-disaster-recovery)一節。
>
>

## <a name="next-steps"></a>後續步驟

- 使用 Azure Site Recovery 開始[保護在 Azure 虛擬機器上執行的應用程式](https://aka.ms/a2a-getting-started)

- 若要深入了解如何實作災害復原和高可用性策略，請參閱 [Azure 應用程式的災害復原和高可用性](../resiliency/resiliency-disaster-recovery-high-availability-azure-applications.md)。

- 若要開發雲端平台功能的詳細技術知識，請參閱 [Azure 復原技術指導](../resiliency/resiliency-technical-guidance.md)。


- 如果指示不清楚，或如果您希望 Microsoft 代您執行作業，請連絡 [客戶支援](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
