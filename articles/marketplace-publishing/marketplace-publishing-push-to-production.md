---
title: 將您的供應項目部署至 Azure Marketplace | Microsoft Docs
description: 了解並逐步依照指示執行，將您的供應項目 (虛擬機器映像、開發人員服務、資料服務等) 部署至 Azure Marketplace。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 8f79b891-84e2-4f41-ba0d-66420e2c6b2e
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/02/2016
ms.author: hascipio
ms.openlocfilehash: 12dc81642905cd9449a1032c7ab57298e6b69ba8
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39714276"
---
# <a name="deploy-your-offer-to-the-azure-marketplace"></a>將您的供應項目部署至 Azure Marketplace
當您對供應項目感到滿意 (意即經過測試的客戶案例、行銷內容等) 並準備推出時，請在 [發佈] 索引標籤中要求 [推送至生產環境]。  

1. 發佈入口網站的 [逐步解說] 頁面中的四個步驟應是完成和綠色的狀態。 對於虛擬機器供應項目，請務必遵循下列指導方針。
   
    ![繪圖][img-pubportal-walkthru-checked]
2. 從左側清單中選取 [ **發佈** ] 索引標籤。
   
    ![繪圖][img-pubportal-menu-publish]
3. 按一下 [要求核准推送到生產環境] 按鈕。 提出要求後，核准小組會執行最後檢閱，然後您的供應項目即可在 Azure Marketplace 中公開發行。
   
    ![繪圖][img-pubportal-publish-pushproduction]

> [!IMPORTANT]
> 如果是「虛擬機器」，當您按一下 [要求核准推送到生產環境] 按鈕時，會在場景後方執行下列步驟。 您可以在發佈入口網站的 [發佈] 索引標籤下檢視每個步驟的進度。 您必須定期查看此頁面是否有需要您更正的任何失敗資訊 (直到狀態顯示「已列出」為止) 。
> 
> * 一開始，您的生產環境要求會送到驗證 VHD 的憑證小組。 不過，如果您是要更新已列出的供應項目或您的要求只有行銷變更，則會略過憑證步驟。
> * 在下一個步驟中，要求會送交驗證供應項目行銷內容的內容驗證小組。
> * 如果上述步驟都成功，便核准供應項目進入生產環境。 此時在發佈入口網站中的狀態會變成「已列出」。 不過，這個「已列出」狀態不代表程序已完成。 在可於 Azure Marketplace 公開發行供應項目之前，必須先完成下列步驟。
> * 一旦供應項目於上述步驟中經核准進入生產環境，就會開始跨所有 Azure 資料中心複寫供應項目。 複寫通常需要 24-48 小時才能完成，但依據 VHD 大小而定，最長可能需要一週的時間。 不過，如果您是要更新已列出的供應項目或您的要求只有行銷變更，則複寫會更快速。
> * 當複寫完成後，供應項目就會在 Azure Marketplace 公開發行。
> 
> 當供應項目的狀態為 [草稿] 時 (也就是永遠不會 [推送至預備環境] 或 [推送至生產環境])，您可隨時將其刪除。 在 [歷程記錄] 索引標籤中，按一下頁面底部的 [捨棄草稿] 按鈕以刪除草稿。
> 
> 

## <a name="production-checklist-for-all-virtual-machine-offers"></a>適用於所有虛擬機器供應項目的生產環境檢查清單
* 請確定您是 Microsoft Azure 認證合作夥伴
* 在 [SKU] 索引標籤下，只有當 SKU 是方案範本的一部分時，[從 Marketplace 隱藏此 SKU，因為應該永遠透過方案範本購買它] 選項才應該標示為 [是]。 否則，這個選項應該永遠標示為 [否]。
* 請記住︰您不應在 SKU 列出後變更 SKU 可見性設定。 我們不支援這項功能。
* 請確定標誌遵循以下的 Azure Marketplace 標誌指導方針。
* 供應項目和 SKU 描述不應相同。
* SKU 的標題和供應項目完整摘要不應相同。
* SKU 標題和供應項目摘要不應相同。
* 對於含多個 SKU 的供應項目，其 SKU 標題不應相同。

**Azure Marketplace 標誌指導方針**

* Azure 設計具有簡單的調色盤。 請保持標誌上具有最少的主要和次要色彩數目。
* Azure 入口網站的佈景主題色彩是白色與黑色。 因此請避免使用這些色彩做為您標誌的背景色彩。 請使用會讓您的標誌在 Azure 入口網站顯得突出的某些色彩。 建議您使用簡單的主要色彩。 如果您使用透明背景，請確定標誌/文字不是白色或黑色。
* 請不要在標誌上使用漸層背景。
* 避免在標誌上放置文字 (甚至是公司或品牌名稱)。
* 您標誌的外觀與風格應該是「一般」，而且應該避免漸層。
* 標誌不應該自動進行縮放。

**主圖標誌的其他指導方針：**

* 主圖標誌為選用項。 發行者可以選擇不上傳主圖標誌。 **但主圖圖示一旦上傳，即無法從發佈入口網站中刪除。屆時合作夥伴必須遵循主圖圖示的 Azure Marketplace 指導方針，否則供應項目將無法核准進入生產環境。**
* 發行者顯示名稱、SKU 標題和供應項目完整細摘要會以白色字型色彩顯示。 因此，您應避免在主圖圖示的背景使用淺色系。 主圖圖示不允許使用黑色、白色和透明背景。
* 供應項目列出之後，會以程式設計方式將發行者顯示名稱、SKU 標題、供應項目完整摘要和 [建立] 按鈕內嵌在主圖標誌內。 因此您在設計主圖標誌時不應輸入任何文字。 請將右邊直接留白，因為我們將會透過程式設計方式，在該處納入文字 (也就是發行者顯示名稱、SKU 標題、供應項目完整摘要)。 文字右側的空白空間應為 415 x 100 (從左邊開始位移 370px)。

## <a name="additional-production-checklist-for-already-listed-virtual-machine-offers"></a>適用於已列出的虛擬機器供應項目的其他生產環境檢查清單
* 請檢查是否已經有任何供應項目包含您公司所提供的同一供應項目。 如果有，您應在現有的供應項目加入新版 SKU，而不是新建重複的供應項目。
* 請勿變更同一個 SKU 的兩個版本之間的資料磁碟。
* Azure Marketplace 不支援變更已列出 SKU 的價格，因為它會影響現有客戶的帳單。 請務必不要變更 SKU 公開發行區域已列出的 SKU 價格。 但是您可以在現有的 SKU 中加入新的 SKU 或新區域。

## <a name="next-steps"></a>後續步驟
一旦供應項目上線，請測試客戶案例，如同在預備環境中進行測試和驗證一樣，驗證在生產環境中的所有合約和功能都運作正常。

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

[img-pubportal-walkthru-checked]:media/marketplace-publishing-push-to-production/pubportal-walkthru-checked.png
[img-pubportal-menu-publish]:media/marketplace-publishing-push-to-production/pubportal-menu-publish.png
[img-pubportal-publish-pushproduction]:media/marketplace-publishing-push-to-production/pubportal-publish-pushproduction.png
