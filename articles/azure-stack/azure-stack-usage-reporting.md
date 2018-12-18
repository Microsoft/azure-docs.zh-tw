---
title: 向 Azure 回報 Azure Stack 使用情況資料 | Microsoft Docs
description: 了解如何在 Azure Stack 中設定使用情況資料報告。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/30/2018
ms.author: sethm
ms.reviewer: alfredop
ms.openlocfilehash: cf6604730ab3bd9ee04fac36e703022f63dcb949
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49090360"
---
# <a name="report-azure-stack-usage-data-to-azure"></a>向 Azure 回報 Azure Stack 使用情況資料 

使用量資料 (也稱為取用資料) 代表所使用的資源量。 

使用取用型計費模型的 Azure Stack 多節點系統應該向 Azure 回報使用量資料以供計費。  Azure Stack 操作員應該設定其 Azure Stack 執行個體，以向 Azure 回報使用量資料。

> [!NOTE]
> 對採用依使用量付費授權模型的 Azure Stack 多節點使用者而言，使用量資料回報是必要功能。 對採用容量授權模型的客戶而言，則為選用功能 (請參閱[購買方式頁面](https://azure.microsoft.com/overview/azure-stack/how-to-buy/))。 對於 Azure Stack 開發套件使用者，Azure Stack 操作員可以回報使用量資料，並測試此功能。 不過，使用者不會因為其造成的任何使用量而需要付費。 


![計費流程](media/azure-stack-usage-reporting/billing-flow.png)

使用情況資料是從 Azure Stack 透過 Azure Bridge 傳送到 Azure。 在 Azure 中，商務系統會處理使用情況資料並產生帳單。 產生帳單之後，Azure 訂用帳戶擁有者可以從 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)檢視並下載帳單。 若要了解 Azure Stack 授權，請參閱 [Azure Stack 包裝與定價文件](https://go.microsoft.com/fwlink/?LinkId=842847&clcid=0x409) \(英文\)。

## <a name="set-up-usage-data-reporting"></a>設定使用情況資料報告

若要設定使用情況資料報告，您必須[向 Azure 註冊您的 Azure Stack 執行個體](azure-stack-register.md)。 在註冊過程中，會設定 Azure Stack 的 Azure Bridge 元件，此元件會將 Azure Stack 連線到 Azure 並傳送使用量資料。 下列情況資料會從 Azure Stack 傳送到 Azure：

- **計量識別碼**：所取用之資源的唯一識別碼。
- **數量** – 資源使用量。
- **位置**：目前 Azure Stack 資源部署所在位置。
- **資源 URI**：將回報其使用狀況之資源的完整 URI。
- **訂用帳戶識別碼**：Azure Stack 使用者的訂用帳戶識別碼，這是本機 (Azure Stack) 訂用帳戶。
- **時間**：使用情況資料的開始與結束時間。 在 Azure Stack 中取用這些資源與向商務系統回報使用情況資料的時間有一些延遲。 Azure Stack 每隔 24 小時會彙總一次使用情況資料，而向 Azure 中的商務管線回報使用情況資料則需要數小時。 因此，在午夜前短暫發生的使用情況可能會在隔天顯示在 Azure 中。

## <a name="generate-usage-data-reporting"></a>產生使用情況資料回報

1. 若要測試使用情況資料回報，請在 Azure Stack 中建立一些資源。 例如，您可以建立[儲存體帳戶](azure-stack-provision-storage-account.md)、[Windows Server VM](azure-stack-provision-vm.md) 與 Linux VM (搭配基本與標準 SKU) 以查看核心使用情況的回報方式。 不同類型資源的使用情況資料會在不同的計量下回報。

2. 讓您的資源執行幾小時。 系統大約每隔一小時會收集一次使用情況資訊。 收集之後，此資料會傳輸到 Azure 並在 Azure 商務系統中處理。 此程序可能需要最多數小時的時間。

## <a name="view-usage---csp-subscriptions"></a>檢視使用量 - CSP 訂用帳戶

如果您已使用 CSP 訂用帳戶註冊您的 Azure Stack，則可以檢視您的使用量和費用，方式和 Azure 耗用量的檢視方式相同。 Azure Stack 使用量會包含在您的發票和對帳檔案中，透過[合作夥伴中心](https://partnercenter.microsoft.com/partner/home)即可取得。 對帳檔案會每月更新。 如果您需要存取最新的 Azure Stack 使用量資訊，您可以使用合作夥伴中心 API。

   ![合作夥伴中心](media/azure-stack-usage-reporting/partner-center.png)


## <a name="view-usage--enterprise-agreement-subscriptions"></a>檢視使用量 – Enterprise 合約訂用帳戶

如果您已使用 Enterprise 合約訂用帳戶註冊您的 Azure Stack，則可以在 [EA 入口網站](https://ea.azure.com/)檢視您的使用量和費用。 Azure Stack 使用量會包含在進階下載中，Azure 使用量則會在 EA 入口網站的 [報告] 區段下。 

## <a name="view-usage--other-subscriptions"></a>檢視使用量 - 其他訂用帳戶

如果您已使用其他任何訂用帳戶類型 (例如，隨用隨付訂用帳戶) 註冊您的 Azure Stack，則可以在 Azure 帳戶中心檢視使用量和費用。 以 Azure 帳戶系統管理員身分登入 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)然後選取您用來註冊 Azure Stack 的 Azure 訂用帳戶。 您可以檢視 Azure Stack 使用情況資料，以及針對所使用之每個資源收取的費用，如下列影像所示：

   ![計費流程](media/azure-stack-usage-reporting/pricing-details.png)

針對 Azure Stack 開發套件，我們不會就使用的 Azure Stack 資源收費，因此價格顯示為 $0.00。

## <a name="which-azure-stack-deployments-are-charged"></a>哪些 Azure Stack 部署需要付費？

針對 Azure Stack 開發套件，資源的使用是免費的。 針對 Azure Stack 多節點系統，工作負載 VM、儲存體服務與應用程式服務需要付費。

## <a name="are-users-charged-for-the-infrastructure-vms"></a>使用基礎結構 VM 是否需要付費？

否。 某些 Azure Stack 資源提供者 VM 的使用量資料會回報給 Azure，但這些 VM 以及在部署期間為了啟用 Azure Stack 基礎結構所建立的 VM，都沒有費用。  

使用者只需針對在租用戶訂用帳戶下執行的 VM 付費。 必須將所有工作負載部署在租用戶訂用帳戶底下，才符合 Azure Stack 的授權條款。

## <a name="i-have-a-windows-server-license-i-want-to-use-on-azure-stack-how-do-i-do-it"></a>我有 Windows Server 授權並想要用在 Azure Stack 上，我該怎麼做？

使用現有授權可避免產生使用量計量。 如 [Azure Stack 授權指南](https://go.microsoft.com/fwlink/?LinkId=851536&clcid=0x409)的＜在 Azure Stack 使用現有軟體＞一節所述，現有的 Windows Server 授權可用在 Azure Stack 中。 客戶必須依照[適用於 Windows Server 授權的混合式權益](https://docs.microsoft.com/azure/virtual-machines/windows/hybrid-use-benefit-licensing)一文中所述的方式部署其 Windows Server 虛擬機器，才能使用其現有授權。

## <a name="which-subscription-is-charged-for-the-resources-consumed"></a>會針對哪個訂用帳戶就所取用的資源收費？
我們會對[向 Azure 註冊 Azure Stack](azure-stack-register.md) 時提供的訂用帳戶收費。

## <a name="what-types-of-subscriptions-are-supported-for-usage-data-reporting"></a>哪些類型的訂用帳戶支援使用情況資料回報？

針對 Azure Stack 多節點，支援「Enterprise 合約」(EA) 和 CSP 訂用帳戶。 針對「Azure Stack 開發套件」，「Enterprise 合約」(EA)、隨用隨付、CSP 及 MSDN 訂用帳戶支援使用量資料回報。

## <a name="does-usage-data-reporting-work-in-sovereign-clouds"></a>使用情況資料回報在主權雲端中是否能運作？

在 Azure Stack 開發套件中，使用情況資料回報需要在全球 Azure 系統中建立的訂用帳戶。 在其中一個主權雲端 (Azure Government、Azure 德國與 Azure 中國雲端) 中建立的訂用帳戶無法向 Azure 註冊，因此它們不支援使用情況資料回報。

## <a name="why-doesnt-the-usage-reported-in-azure-stack-match-the-report-generated-from-azure-account-center"></a>為何在 Azure Stack 中回報的使用情況與從 Azure 帳戶中心產生的報告不符？

在 Azure Stack 使用量 API 所回報的使用量資料與「Azure 帳戶中心」所回報的使用量資料之間一定會有延遲。 此延遲是將使用量資料從 Azure Stack 上傳到 Azure 商務系統所需的時間。 由於此延遲，在午夜前短暫發生的使用情況可能會在隔天顯示在 Azure 中。 若使用 [Azure Stack 使用情況 API](azure-stack-provider-resource-api.md)，並將結果與在 Azure 計費入口網站中回報的使用情況比較，您可以看到差異。

## <a name="next-steps"></a>後續步驟

* [提供者使用量 API](azure-stack-provider-resource-api.md)  
* [租用戶使用量 API](azure-stack-tenant-resource-usage-api.md)
* [使用狀況常見問題集](azure-stack-usage-related-faq.md)
* [以雲端服務提供者身分管理使用量和計費](azure-stack-add-manage-billing-as-a-csp.md)
