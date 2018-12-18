---
title: 管理 Azure 保留 | Microsoft Docs
description: 了解如何變更訂用帳戶範圍和管理 Azure 保留的存取權。
services: billing
documentationcenter: ''
author: yashesvi
manager: yashesvi
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2018
ms.author: cwatson
ms.openlocfilehash: 0b19bb0d77bb600258596ce369713464641a7d2f
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423233"
---
# <a name="manage-reservations-for-azure-resources"></a>管理 Azure 資源的保留

購買 Azure 保留之後，您可能需要將保留套用至不同的訂用帳戶、變更管理保留的人員或變更保留範圍。 您也可以將保留分割成兩個保留，以將您購買的部分執行個體套用至另一個訂用帳戶。

如果您已購買 Azure 保留的 VM 執行個體，您可以變更保留的最佳化設定。 保留折扣可以套用至相同系列的 VM，或是您可以為特定 VM 大小保留資料中心容量。

## <a name="change-the-scope-for-a-reservation"></a>變更保留範圍

 保留折扣適用於符合保留且在保留範圍內執行的虛擬機器、SQL 資料庫、Azure Cosmos DB 或其他資源。 保留的範圍可以是單一訂用帳戶或您的計費內容中的所有訂用帳戶。 如果您將範圍設定為單一訂用帳戶，此保留會對應至所選訂用帳戶中執行的資源。 如果您將範圍設定為共用，Azure 會將保留對應至計費內容中所有訂用帳戶上執行的資源。 計費內容取決於購買此保留所用的訂用帳戶。

更新保留範圍：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務] > [保留]。
3. 選取保留。
4. 選取 [設定] > [組態]。
5. 變更範圍。 

如果您從共用變更為單一範圍，您只能選取擁有者是您的訂用帳戶。 只能選取與保留屬於相同計費內容的訂用帳戶。

該範圍僅適用於預付型方案供應項目的 MS-AZR-0003P、企業供應項目的 MS-AZR-0017P 或 CSP 訂用帳戶類型。 對於企業合約，開發/測試訂用帳戶沒有取得保留折扣的資格。

## <a name="add-or-change-users-who-can-manage-a-reservation"></a>新增或變更可以管理保留的使用者

您可以透過將人員新增到保留上的角色，來委派保留的管理。 依預設，購買保留的人和帳戶管理員就是保留上的「擁有者」角色。

您可以從取得保留折扣的訂用帳戶中，獨立管理保留的存取。 當您將管理保留的權限給其他人時，並不會給他們管理訂用帳戶的權限。 而當您將保留範圍內的訂用帳戶管理權限給其他人時，並不會給他們管理保留的權限。

委派保留的存取管理：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務]  >  [保留] 來列出您具有存取權的保留。
3. 選取您想要將存取權委派給其他使用者的保留。
4. 選取 [存取控制 (IAM)]。
5. 選取 [新增] > [角色] > [擁有者]。 或是，如果您想要給予有限的存取權，可選取不同角色。
6. 輸入要新增為擁有者之使用者的電子郵件地址。
7. 選取使用者，然後選取 [儲存]。

## <a name="split-a-single-reservation-into-two-reservations"></a>將單一保留分割成兩個保留

 購買保留內的多個資源執行個體之後，建議您將該保留中的執行個體指派至不同的訂用帳戶。 依預設，所有執行個體都具有一個範圍 - 單一訂用帳戶或共用。 例如，您購買了 10 個保留執行個體，並指定範圍為訂用帳戶 A。您現在要將 7 個保留的範圍變更為訂用帳戶 A，而剩下 3 個變更為訂用帳戶 B。分割保留可讓您分散執行個體，以進行更細微的管理。 您可以選擇共用範圍來簡化訂用帳戶的配置。 但是，基於成本管理或預算考量，您可以配置一些數量給特定的訂用帳戶。

 您可以透過 PowerShell、CLI 或 API 來將保留分成兩個保留。

### <a name="split-a-reservation-by-using-powershell"></a>使用 PowerShell 分割保留

1. 執行下列命令以取得保留訂單識別碼：

    ```powershell
    # Get the reservation orders you have access to
    Get-AzureRmReservationOrder
    ```

2. 取得保留的詳細資料：

    ```powershell
    Get-AzureRmReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId b8be062a-fb0a-46c1-808a-5a844714965a
    ```

3. 將保留分割成兩個，並分散執行個體：

    ```powershell
    # Split the reservation. The sum of the reservations, the quantity, must equal the total number of instances in the reservation that you're splitting.
    Split-AzureRmReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId b8be062a-fb0a-46c1-808a-5a844714965a -Quantity 3,2
    ```
4. 您可以執行下列命令來更新範圍：

    ```powershell
    Update-AzureRmReservation -ReservationOrderId a08160d4-ce6b-4295-bf52-b90a5d4c96a0 -ReservationId 5257501b-d3e8-449d-a1ab-4879b1863aca -AppliedScopeType Single -AppliedScope /subscriptions/15bb3be0-76d5-491c-8078-61fe3468d414
    ```

## <a name="cancellations-and-exchanges"></a>取消和交換

視保留類型而定，您可能可以取消或交換保留。 如需詳細資訊，請參閱下列主題中的取消和交換小節：

- [預付具有 Azure 保留 VM 執行個體的虛擬機器](..//virtual-machines/windows/prepay-reserved-vm-instances.md#cancellations-and-exchanges)
- [從 Azure 保留預付 SUSE 軟體方案](../virtual-machines/linux/prepay-suse-software-charges.md#cancellation-and-exchanges-not-allowed)
- [以 Azure SQL Database 保留容量預先支付 SQL 資料庫計算資源的費用](../sql-database/sql-database-reserved-capacity.md#cancellations-and-exchanges)

## <a name="change-optimize-setting-for-reserved-vm-instances"></a>變更保留 VM 執行個體的最佳化設定

 當您購買保留的 VM 執行個體時，您可以選擇執行個體大小的彈性或容量優先順序。 執行個體大小彈性會將保留項目折扣套用至同一個 [VM 大小群組](https://aka.ms/RIVMGroups)中的其他 VM。 容量優先順序會讓資料中心容量供您的部署優先使用。 此選項可讓您更加確信您能夠在需要時啟動 VM 執行個體。

根據預設，當保留範圍為共用時，執行個體大小彈性就會啟用。 資料中心容量不會針對 VM 部署設定優先權。

對於單一範圍的保留，您可以為了容量優先順序 (而不是為了 VM 執行個體大小彈性) 而將保留最佳化。

若要更新保留的最佳化設定：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務]  >  [保留]。
3. 選取保留。
4. 選取 [設定] > [組態]。
5. 變更**最佳化設定**。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 保留項目，請參閱下列文章：

- [什麼是 Azure 保留項目？](billing-save-compute-costs-reservations.md)
- [預付具有 Azure 保留 VM 執行個體的虛擬機器](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [以 Azure SQL Database 保留容量預先支付 SQL 資料庫計算資源的費用](../sql-database/sql-database-reserved-capacity.md)
- [以 Azure Cosmos DB 保留容量預先支付 Azure Cosmos DB 資源的費用](../cosmos-db/cosmos-db-reserved-capacity.md)
- [從 Azure 保留預付 SUSE 軟體方案](../virtual-machines/linux/prepay-suse-software-charges.md)
- [了解 VM 保留折扣的套用方式](billing-understand-vm-reservation-charges.md)
- [了解如何套用 SUSE Linux Enterprise 軟體方案折扣](../billing/billing-understand-suse-reservation-charges.md)
- [了解其他保留折扣的套用方式](billing-understand-reservation-charges.md)
- [了解隨用隨付方案訂用帳戶的保留項目使用量](billing-understand-reserved-instance-usage.md)
- [了解 Enterprise 註冊的保留項目使用量](billing-understand-reserved-instance-usage-ea.md)
- [Windows 軟體成本不包含在 Reservations 內](billing-reserved-instance-windows-software-costs.md)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
