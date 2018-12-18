---
author: yashesvi
ms.author: yashar
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 09-05-2018
ms.openlocfilehash: 065ac0855fc47b23b434287f9f4406bd641f01ae
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47454558"
---
# <a name="prepay-for-virtual-machines-with-azure-reserved-vm-instances"></a>預付具有 Azure 保留 VM 執行個體的虛擬機器

預付虛擬機器，並使用 Azure 保留的虛擬機器 (VM) 執行個體節省成本。 如需詳細資訊，請參閱 [Azure 保留的 VM 執行個體供應項目](https://azure.microsoft.com/pricing/reserved-vm-instances/)。

您可以在 [Azure 入口網站](https://portal.azure.com)購買保留 VM 執行個體。 若要購買執行個體：

- 您必須至少為一個企業或"隨用隨付"型方案訂用帳戶的「擁有者」角色。
- 針對企業訂用帳戶，必須在 [EA 入口網站](https://ea.azure.com)中啟用保留項目購買。
- 針對雲端解決方案提供者 (CSP) 程式，只有系統管理員代表或銷售代表可以購買保留項目。

## <a name="determine-the-right-vm-size-before-you-buy"></a>購買之前，請先決定正確的 VM 大小

使用量資料中的 [計量子類別] 和 [產品] 欄位不會分辨 VM 大小有無使用進階儲存體。 如果您使用這些欄位來判斷要用於保留的 VM 大小，您可能會購買不正確的大小，而且無法獲得您所預期的保留折扣。 當您購買保留時，使用下列其中一個方法來判斷正確的 VM 大小：

- 請參閱使用量檔案或使用量 API 中的 AdditionalInfo 欄位，判斷正確的 VM 大小。 請勿使用 [計量子類別] 或 [產品] 欄位的值。 這些欄位不會區分 VM 的 S 和非 S 版本。
- 使用 PowerShell、Azure Resource Manager，或從 Azure 入口網站的 VM 詳細資料，取得正確的 VM 大小資訊。

保留 VM 執行個體適用於大多數 VM 大小，但有些例外：

- 保留折扣不適用於下列 VM：
  - 傳統 VM 和雲端服務
  - 限制的 vCPU 大小
  - VM 系列：A 系列、Av2 系列或 G 系列
  - 在預覽狀態的 VM：處於預覽狀態的任何 VM 系列或大小
- 雲端：保留項目不適用於 Azure 美國政府、德國或中國區域中的購買。
- 配額不足：保留項目如果範圍限定在單一訂用帳戶，必須訂用帳戶中有可用的 vCPU 配額，才能新增 RI。 例如，如果目標訂用帳戶的配額限制為 10 個 D 系列的 vCPU，您便無法購買一個適用於 11 個 Standard_D1 執行個體的保留項目。 保留的配額檢查包含已在訂用帳戶中部署的 VM。 例如，如果訂用帳戶的配額為 10 個 D 系列的 vCPU，且已部署 2 個 standard_D1 執行個體，則您可以在此訂用帳戶中購買一個適用於 10 個 standard_D1 執行個體的保留項目。 
- 容量限制：在極少數的情況下，Azure 會限制購買 VM 大小子集的新保留，因為區域中的低容量。

## <a name="buy-a-reserved-vm-instance"></a>購買保留 VM 執行個體

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務] > [保留]。
3. 選取 [新增] 以購買新的保留項目。
4. 填寫必要欄位。 符合您選取之屬性的執行中 VM 執行個體符合取得保留項目折扣的資格。 取得折扣的 VM 執行個體實際數目取決於選取的範圍和數量。

    | 欄位      | 說明|
    |:------------|:--------------|
    |名稱        |此保留項目的名稱。| 
    |訂用帳戶|用來支付保留項目的訂用帳戶。 保留項目的預付費用會透過訂用帳戶的付款方式收取。 訂用帳戶類型必須是 Enterprise 合約 (供應項目號碼：MS-AZR-0017P) 或預付型方案 (供應項目號碼：MS-AZR-0003P)。 針對企業訂用帳戶，費用會從註冊的承諾用量金額餘額扣除或作為超額部分收費。 針對預付型方案訂用帳戶，費用是透過訂用帳戶的信用卡或發票付款方式收取。|    
    |影響範圍       |保留項目範圍可以涵蓋一個訂用帳戶或多個訂用帳戶 (共用範圍)。 如果您選取： <ul><li>單一訂用帳戶 - 保留項目折扣會套用至此訂用帳戶中的 VM。 </li><li>共用 - 保留項目折扣會套用至計費內容內任何訂用帳戶中執行的 VM。 針對企業客戶，共用範圍是註冊，並包含註冊中的所有訂用帳戶 (開發/測試訂用帳戶除外)。 針對預付型方案客戶，共用範圍是帳戶系統管理員所建立的所有預付型方案訂用帳戶。</li></ul>|
    |區域    |保留項目所涵蓋的 Azure 區域。|    
    |VM 大小     |VM 執行個體的大小。|
    |已針對下列項目最佳化     |VM 執行個體大小彈性會將保留項目折扣套用至同一個 [VM 大小群組](https://aka.ms/RIVMGroups)中的其他 VM。 容量優先順序會讓資料中心容量供您的部署優先使用。 這可讓您更加確信您能夠在需要時啟動 VM 執行個體。 保留項目範圍是單一訂用帳戶時，才可使用容量優先順序。 |
    |詞彙        |一年或三年。|
    |數量    |保留項目內所購買的執行個體數目。 數量是可以取得帳單折扣之執行中 VM 執行個體的數目。 例如，如果您在美國東部執行 10 個 Standard_D2 VM，那麼您會指定數量為 10，以最大化所有執行中機器的效益。 |
5. 當您選取 [計算成本] 時，可以檢視保留項目的成本。

    ![提交保留項目購買前的螢幕擷取畫面](./media/virtual-machines-buy-compute-reservations/virtualmachines-reservedvminstance-purchase.png)

6. 選取 [購買]。
7. 選取 [檢視此保留] 以查看您的購買狀態。

    ![提交保留項目購買後的螢幕擷取畫面](./media/virtual-machines-buy-compute-reservations/virtualmachines-reservedvmInstance-submit.png)

## <a name="cancellations-and-exchanges"></a>取消和交換

如果您需要取消保留，可能會有 12% 的提前解約金。 退款以您購買價格或目前保留價格的最低價格為準。 每年的退款金額上限為 50,000 美元。 您收到的退款是按比例計算的餘額扣除 12% 提前解約金的金額。 若要要求取消，請移至 Azure 入口網站中的保留，並選取 [退款] 以建立支援要求。

如果您需要將 [保留的 VM 執行個體] 保留變更為另一個區域、VM 大小群組或期限，您可以將它與等值或更高價值的另一個保留交換。 新保留的期限開始日期不會延續自交換的保留。 1 或 3 年的期限會從您建立新的保留時起算。 若要要求交換，請移至 Azure 入口網站中的保留，並選取 [交換] 以建立支援要求。

## <a name="next-steps"></a>後續步驟

保留項目折扣會自動套用到符合保留項目範圍和屬性之執行中虛擬機器的數目。 您可以透過 [Azure 入口網站](https://portal.azure.com)、PowerShell、CLI 或 API 來更新保留項目的範圍。

若要了解如何管理保留項目，請參閱[管理 Azure 保留項目](../articles/billing/billing-manage-reserved-vm-instance.md)。

若要深入了解 Azure 保留項目，請參閱下列文章：

- [什麼是 Azure 保留項目？](../articles/billing/billing-save-compute-costs-reservations.md)
- [管理 Azure 中的保留](../articles/billing/billing-manage-reserved-vm-instance.md)
- [了解保留項目折扣的套用方式](../articles/billing/billing-understand-vm-reservation-charges.md)
- [了解預付型方案訂用帳戶的保留項目使用量](../articles/billing/billing-understand-reserved-instance-usage.md)
- [了解 Enterprise 註冊的保留項目使用量](../articles/billing/billing-understand-reserved-instance-usage-ea.md)
- [Windows 軟體的成本不包括在保留項目內](../articles/billing/billing-reserved-instance-windows-software-costs.md)
- [合作夥伴中心雲端解決方案提供者 (CSP) 計畫中的 Azure 保留項目](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
