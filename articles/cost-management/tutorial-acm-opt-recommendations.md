---
title: 教學課程 - 使用最佳化建議降低 Azure 成本 | Microsoft Docs
description: 本教學課程可在您針對最佳化建議採取行動時協助您降低 Azure 成本。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/21/2018
ms.topic: conceptual
ms.service: cost-management
manager: dougeby
ms.custom: ''
ms.openlocfilehash: 4d9e47d6da45eaba19cbe089de3fdf053c36046a
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47030672"
---
# <a name="tutorial-optimize-costs-from-recommendations"></a>教學課程：透過建議最佳化成本

Azure 成本管理可搭配 Azure Advisor，提供成本最佳化建議。 Azure Advisor 可協助您透過識別閒置及使用量過低的資源來最佳化及改善效率。 本教學課程會引導您完成範例，識別使用量過低的 Azure 資源，並讓您採取動作以降低成本。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 檢視成本最佳化建議以檢視潛在的使用效率不足
> * 針對建議採取行動，將虛擬機器的大小調整至更符合成本效益的選項
> * 驗證動作，確保虛擬機器的大小已成功調整

## <a name="prerequisites"></a>必要條件
「建議」適用於所有 [Enterprise 合約 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) 客戶。 您必須至少具備一或多個下列範圍的讀取存取，才能檢視成本資料。

- 訂用帳戶
- 資源群組

您必須擁有使用中的虛擬機器，且其至少需要活動 14 天。

## <a name="sign-in-to-azure"></a>登入 Azure
在 [https://portal.azure.com](https://portal.azure.com/) 登入 Azure 入口網站。

## <a name="view-cost-optimization-recommendations"></a>檢視成本最佳化建議

在 Azure 入口網站中，按一下服務清單中的 [成本管理與帳單]。 然後，在 [成本管理] 下方的清單中，選取 [Advisor 建議]。 隨即顯示 Advisor 成本建議。

![Advisor 建議](./media/tutorial-acm-opt-recommendations/advisor-recommendations.png)

建議清單會識別使用效率不足的情況，或是顯示能協助您節省額外成本的購買建議。 合計 [潛在每年節省成本] 會顯示關機或解除配置您所有符合建議規則的 VM 時，所能節省的總金額。 若您不想要將它們關機，建議您考慮將其大小調整至較不昂貴的 VM SKU。

[影響] 類別與 [潛在每年節省成本] 旨在協助您識別具有潛力，可以盡量節省成本的建議。 影響程度較大的建議包括[購買保留的虛擬機器執行個體以節省超出預付型方案成本](../advisor/advisor-cost-recommendations.md#buy-reserved-virtual-machine-instances-to-save-money-over-pay-as-you-go-costs)，以及[透過調整大小或關閉使用量過低的執行個體來最佳化虛擬機器支出](../advisor/advisor-cost-recommendations.md#optimize-virtual-machine-spend-by-resizing-or-shutting-down-underutilized-instances)。 影響程度中等的建議包括[移除未佈建的 ExpressRoute 線路來降低成本](../advisor/advisor-cost-recommendations.md#reduce-costs-by-eliminating-unprovisioned-expressroute-circuits)，以及[刪除或重新設定閒置虛擬網路閘道來降低成本](../advisor/advisor-cost-recommendations.md#reduce-costs-by-deleting-or-reconfiguring-idle-virtual-network-gateways)。

## <a name="act-on-a-recommendation"></a>針對建議採取行動

Azure Advisor 可監視過去 14 天的虛擬機器使用量，並找出使用量過低的虛擬機器。 CPU 使用量小於 (含) 5% 且網路使用量小於 (含) 7 MB 長達 4 天 (含) 以上的虛擬機器，將會視為低使用量虛擬機器。

5% 以下 (含) 的 CPU 使用量設定為預設，但您可以調整設定。 如需調整設定的詳細資訊，請參閱[低使用量虛擬機器建議](../advisor/advisor-get-started.md#configure-the-average-cpu-utilization-rule-for-the-low-usage-virtual-machine-recommendation)的[設定平均 CPU 使用量規則](../advisor/advisor-get-started.md#configure-the-average-cpu-utilization-rule-for-the-low-usage-virtual-machine-recommendation)一文。

雖然根據設計，某些案例可能會導致低使用量，但您通常可以藉由將虛擬機器的大小變更為較不昂貴的大小，來節省費用。 若您選擇調整大小動作，則您實際節省的成本可能會有所不同。 讓我們逐步完成調整虛擬機器大小的範例。

在建議清單中，按一下 [調整大小或關閉使用量過低的虛擬機器] 建議。 在虛擬機器候選項目清單中，選擇要調整大小的虛擬機器，然後按一下虛擬機器。 隨即顯示虛擬機器的詳細資料，讓您可以確認使用量計量。 [潛在每年節省成本] 值是關機或移除 VM 時您所能節省的金額。 調整 VM 的大小可能可以節省成本，但是您將無法節省 [潛在每年節省成本] 的完整金額。

![建議的詳細資料](./media/tutorial-acm-opt-recommendations/recommendation-details.png)

在 VM 詳細資料中，檢查虛擬機器的使用量，確認其為適合的調整大小候選項目。

![VM 詳細資料](./media/tutorial-acm-opt-recommendations/vm-details.png)

請注意目前虛擬機器的大小。 在確認虛擬機器的大小應該進行調整之後，請關閉 VM 詳細資料，讓您可以查看虛擬機器清單。

在要關機或調整大小的候選項目清單中，選取 [調整虛擬機器大小]。
![調整虛擬機器大小](./media/tutorial-acm-opt-recommendations/resize-vm.png)

接下來，您會看到一份可用的調整大小選項清單。 選擇可為您帶來最佳效能，並且可讓案例最符合成本效益的選項。 在下列範例中，選擇的選項會將 **DS14\_V2** 大小調整至 **DS13\_V2**。 下列建議可每月節省美金 $551.30 元，或每年節省美金 $6,615.60 元。

![選擇大小](./media/tutorial-acm-opt-recommendations/choose-size.png)

在您選擇適合的大小後，按一下 [選取] 以開始調整大小動作。

調整大小需要重新開機正在使用中的執行中虛擬機器。 若虛擬機器位於生產環境，我們建議您在上班時間結束之後再執行調整大小作業。 排程重新開機可減少因暫時無法使用而導致的中斷。

## <a name="verify-the-action"></a>驗證動作

當 VM 成功完成調整大小之後，隨即顯示 Azure 通知。

![調整大小完畢通知](./media/tutorial-acm-opt-recommendations/resized-notification.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 檢視成本最佳化建議以檢視潛在的使用效率不足
> * 針對建議採取行動，將虛擬機器大小調整至更符合成本效益的選項
> * 驗證動作，確保虛擬機器的大小已成功調整

若您尚未閱讀成本管理最佳做法一文，則它可提供高層級指導及協助管理成本時需要考慮的準則。

> [!div class="nextstepaction"]
> [成本管理最佳做法](cost-mgt-best-practices.md)
