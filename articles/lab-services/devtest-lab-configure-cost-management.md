---
title: 檢視 Azure DevTest Labs 中的每月估計實驗室成本趨勢 | Microsoft Docs
description: 了解 Azure DevTest Labs 每月估計成本趨勢圖表。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 1f46fdc5-d917-46e3-a1ea-f6dd41212ba4
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2018
ms.author: spelluru
ms.openlocfilehash: 13535dae82ef2c8896dad7d6221553d15e4e6a95
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38573806"
---
# <a name="view-the-monthly-estimated-lab-cost-trend-in-azure-devtest-labs"></a>檢視 Azure DevTest Labs 中的每月估計實驗室成本趨勢
研測實驗室的成本管理功能可協助您追蹤實驗室成本。 本文會示範如何使用「每月估計成本趨勢」  圖表，來檢視目前日曆月份的到目前為止的估計成本，以及目前日曆月份的預計月底成本。 本文也會說明如何設定費用目標和閾值，並在觸達時觸發 DevTest Labs 向您報告結果，從而更輕鬆管理實驗室成本。

## <a name="viewing-the-monthly-estimated-cost-trend-chart"></a>檢視每月估計成本趨勢圖表
若要檢視「每月估計成本趨勢」圖表，請遵循下列步驟︰ 

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 視需要選取 [所有服務]，然後從清單中選取 [DevTest Labs]。 (您的實驗室可能已顯示在 [所有資源] 下方的儀表板上)。
1. 從實驗室清單中，選取所需的實驗室。  
1. 在實驗室的 [概觀] 區域中選取 [組態和原則]。   
1. 在 [成本追蹤] 下的左側，選取 [成本趨勢]。

   下列螢幕擷取畫面顯示成本圖表範例。 
   
    ![成本圖表](./media/devtest-lab-configure-cost-management/graph.png)

**估計成本** 值是到目前行事曆月份為止累計的預估成本。 **預計成本** 是目前行事曆月份整個月的估計成本，是以前五天的實驗室成本來計算。

成本金額會無條件進位到下一個整數。 例如︰ 

* 5.01 會無條件進位到 6 
* 5.50 會無條件進位到 6
* 5.99 會無條件進位到 6

如圖表上方所述，您在圖表中看到的成本預設是使用[預付型方案](https://azure.microsoft.com/offers/ms-azr-0003p/)供應項目費率的估計成本。 您也可以[管理實驗室的成本目標](#managing-cost-targets-for-your-lab)，來設定在圖表中顯示您自己的費用目標。

此外，以下「未」  併入成本計算︰

* 目前不支援 CSP 和 Dreamspark 訂用帳戶，因為 Azure DevTest Labs 使用 [Azure 計費 API](../billing/billing-usage-rate-card-overview.md) 來計算實驗室成本，而 Azure 計費 API 並不支援 CSP 或 Dreamspark 訂用帳戶。
* 您的供應項目費率。 目前，您無法使用您與 Microsoft 或 Microsoft 合作夥伴協議的供應項目費率 (顯示於您的訂用帳戶下方)。 只會使用預付型方案費率。
* 您的稅率
* 您的折扣
* 您的帳單貨幣。 目前，實驗室成本只會以美元貨幣顯示。

## <a name="managing-cost-targets-for-your-lab"></a>管理您實驗室的成本目標
DevTest Labs 可讓您更輕鬆地管理您實驗室中的成本，方法是設定費用目標，以便您可在「每月估計成本趨勢」圖表中進行檢視。 當觸達指定的目標目標或閾值上限時，DevTest Labs 也可以傳送通知給您。 

1. 在實驗室的 [概觀] 窗格中，選取 [設定與原則]。
1. 在 [成本追蹤] 下的左側，選取 [成本趨勢]。

    ![[管理目標] 按鈕](./media/devtest-lab-configure-cost-management/cost-trend.png)

1. 在 [成本趨勢] 窗格中，選取 [管理目標]。

    ![[管理目標] 按鈕](./media/devtest-lab-configure-cost-management/cost-trend-manage-target.png)

1. 在 [管理目標] 窗格中，指定您所需的費用目標和閾值。 您也可以設定要在成本趨勢圖表上報告每個選取的閾值，還是透過 webhook 通知來報告每個選取的閾值。

    ![[管理目標] 窗格](./media/devtest-lab-configure-cost-management/cost-trend-manage-target-pane.png)

   - 選擇您需要追蹤成本目標的時間週期。
      - **每月**：會追蹤每個月的成本目標。
      - **固定**：會追蹤您在 [開始日期] 和 [結束日期] 欄位中指定之日期範圍的成本目標。 一般而言，這可能會對應至您的專案排定要執行的時間長度。
   - 指定**目標成本**。 例如，這可能是在您定義的時間週期內，計劃要花費多少在這個實驗室。
   - 選取要啟用還是停用您需要報告的任何閾值 – 以 25% 增量計算 – 最高為您指定之**目標成本**的 125%。
      - **通知**：當符合此閾值時，您指定的 webhook URL 就會通知您。
      - **在圖表上繪製**：當符合此閾值時，會在您可以檢視的成本趨勢圖表上繪製結果，如[檢視每月估計成本趨勢圖表](#viewing-the-monthly-estimated-cost-trend-chart)中所述。
   - 如果您選擇在符合閾值時 [通知]，就必須指定 webhook URL。 在成本整合區域中，選取 [按一下這裡可新增整合]。

      在 [設定通知] 窗格中輸入 Webhook URL，然後選取 [確定]。

       ![[設定通知] 窗格](./media/devtest-lab-configure-cost-management/configure-notification.png)

      - 如果您指定 [通知]，就必須定義 webhook URL。
      - 同樣地，如果您定義 webhook URL，就必須將 [成本閾值] 窗格中的 [通知] 設定為 [開啟]。
      - 在這裡輸入 webhook 之前，您必須先建立 webhook。  

      如需 webhook 的詳細資訊，請參閱[建立 webhook 或 API Azure 函式](../azure-functions/functions-create-a-web-hook-or-api-function.md)。 
 

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>相關部落格文章
* [讓 DevTest Labs 成本不失控的另外兩項須知](https://blogs.msdn.microsoft.com/devtestlab/2016/06/21/keep-your-cost-on-track/)
* [為何要設定成本臨界值？](https://blogs.msdn.microsoft.com/devtestlab/2016/04/11/why-cost-thresholds/)

## <a name="next-steps"></a>後續步驟
以下是接下來要嘗試的一些事項：

* [定義實驗室原則](devtest-lab-set-lab-policy.md) - 了解如何設定用來管理您實驗室及其 VM 使用方式的各種原則。 
* [建立自訂映像](devtest-lab-create-template.md) - 當您建立 VM 時，您要指定一個基本映像，它可以是自訂映像或 Marketplace 映像。 本文會示範如何從 VHD 檔案建立自訂的映像。
* [設定 Marketplace 映像](devtest-lab-configure-marketplace-images.md) - DevTest Labs 支援根據 Azure Marketplace 映像建立 VM。 本文會示範在實驗室中建立 VM 時，如何指定可以使用哪些 Azure Marketplace 映像 (如果有的話)。
* [在實驗室中建立 VM](devtest-lab-add-vm.md) - 示範如何從基本映像 (自訂或 Marketplace) 建立 VM，以及如何使用 VM 中的構件。

