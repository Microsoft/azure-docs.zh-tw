---
title: 了解 Azure Reservations 折扣 | Microsoft Docs
description: 了解保留折扣套用至執行中 SQL Database 的方式。
services: billing
documentationcenter: ''
author: yashesvi
manager: yashar
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2018
ms.author: yashar
ms.openlocfilehash: 923337291d297832e03892b505f4efa8d3bb3f36
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43300709"
---
# <a name="understand-how-an-azure-reservation-discount-is-applied-to-sql-databases"></a>了解 Azure 保留折扣如何套用至 SQL Database

購買 Azure SQL Database 保留容量之後，保留折扣就會自動套用至符合保留屬性和數量的 SQL Database。 保留會涵蓋 SQL Database 的計算成本。 您必須依標準費率支付軟體、儲存體和網路的費用。 您可以利用 [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) 來涵蓋 SQL Database 的授權成本。

若要了解保留的虛擬機器執行個體，請參閱[了解 Azure 保留的 VM 執行個體折扣](billing-understand-vm-reservation-charges.md)。

## <a name="reservation-discount-applied-to-sql-databases"></a>套用至 SQL Database 的保留折扣

 SQL Database 保留容量折扣會套用至執行中的 SQL Database (以小時計算)。 您所購買的保留會與對應到執行中 SQL Database 所發出的計算使用量進行比對。 對於未執行滿一個小時的 SQL Database，此保留會自動套用至其他與保留屬性相符的 SQL Database。 此折扣可以同時套用至各個執行中的 SQL Database。 如果沒有既符合保留屬性又執行滿一個小時的 SQL Database，您就無法獲得該小時的完整保留折扣權益。

下列範例說明根據所購買的核心數目，以及在您執行時，SQL Database 保留容量折扣的套用方式。

- 案例 1：您為 8 核心的 SQL Database 購買 SQL Database 保留容量。 您所執行的 16 核心 SQL Database 符合保留的其餘屬性。 您需要就 8 核心 SQL Database 計算使用量的預付型方案價格支付費用。 您會就一小時的 8 核心 SQL Database 計算使用量獲得保留折扣。

針對下列其餘範例，假設您所購買的 SQL Database 保留容量適用於 16 核心的 SQL Database，而且其餘保留屬性符合執行中的 SQL Database。

- 案例 2：您執行了兩個各有 8 核心的 SQL Database 達一小時。 16 核心的保留折扣會套用至這兩個 8 核心 SQL Database 的計算使用量。
- 案例 3：您執行了一個 16 核心的 SQL Database，時間從下午 1 點到下午 1:30。 您執行了另一個 16 核心的 SQL Database，時間從下午 1:30 到下午 2 點。 兩者都在保留折扣的涵蓋範圍內。
- 案例 4：您執行了一個 16 核心的 SQL Database，時間從下午 1 點到下午 1:45。 您執行了另一個 16 核心的 SQL Database，時間從下午 1:30 到下午 2 點。 您需要就 15 分鐘的重疊時間支付預付型方案價格的費用。 保留折扣會套用至其餘時間的計算使用量。

若要在計費使用量報告中了解及檢視 Azure Reservations 的應用，請參閱[了解 Azure 保留使用量](https://go.microsoft.com/fwlink/?linkid=862757)。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure Reservations，請參閱下列文章：

- [什麼是 Azure Reservations？](billing-save-compute-costs-reservations.md)
- [預付具有 Azure 保留 VM 執行個體的虛擬機器](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [以 Azure SQL Database 保留容量預先支付 SQL 資料庫計算資源的費用](../sql-database/sql-database-reserved-capacity.md)
- [管理 Azure 保留項目](billing-manage-reserved-vm-instance.md)
- [了解預付型方案訂用帳戶的保留項目使用量](billing-understand-reserved-instance-usage.md)
- [了解 Enterprise 註冊的保留項目使用量](billing-understand-reserved-instance-usage-ea.md)
- [了解 CSP 訂用帳戶的保留使用量](https://docs.microsoft.com/partner-center/azure-reservations)


## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
