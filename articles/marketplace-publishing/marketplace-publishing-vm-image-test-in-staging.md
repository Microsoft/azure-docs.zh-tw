---
title: 測試將發佈到 Marketplace 的 VM 供應項目 | Microsoft Docs
description: 了解如何測試您的 VM 供應項目在 Azure Marketplace 的表現。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 7a41c3c6-625c-4478-b804-e124dee89040
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/01/2016
ms.author: hascipio
ms.openlocfilehash: 26f856059b381be91b9cdd1f98a11dc90813c0c5
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39715867"
---
# <a name="test-your-vm-offer-for-the-azure-marketplace-in-staging"></a>在預備環境中測試您將發佈到 Azure Marketplace 的 VM 供應項目
預備環境代表將您的 SKU 部署在私人的「沙箱」中，您可以在部署到 Marketplace 之前在沙箱中測試與驗證其功能。 SKU 會出現在預備環境中，就如同客戶已部署該項目一樣。 您的 VM 映像必須經過認證才能推送至預備環境。

## <a name="step-1-push-your-offer-to-staging"></a>步驟 1：推送供應項目至預備環境
1. 在 [發佈] 索引標籤上,按一下 [推送至預備環境]。
   
    ![繪圖](media/marketplace-publishing-vm-image-test-in-staging/vm-image-push-to-staging.png)
2. 如果發佈入口網站通知您任何錯誤，修正它們。
3. 在 [ **誰可以存取您預備的供應項目？** ] 對話方塊中，輸入您要用來在 [Azure Preview 入口網站](https://portal.azure.com)中預覽供應項目的 Azure 訂用帳戶清單。
   
   > [!NOTE]
   > 如果是「虛擬機器」和「方案」範本，請 **不要** 將 CSP、DreamSpark 或 Azure in Open 類型的訂用帳戶加入允許清單。
   > 
   > 

    > 如果是「虛擬機器」，當您按一下 [推送至預備環境] 按鈕時，會在場景後方執行下列步驟。 您可以在發佈入口網站的 [發佈] 索引標籤下檢視每個步驟的進度。 您必須定期查看此頁面是否有需要您更正的任何失敗資訊 (直到狀態顯示「已預備」為止) 。

    > - 一開始，您的預備環境要求會送到驗證 VHD 的憑證小組。 不過，如果您的要求只有行銷變更，則會略過憑證步驟。
    > - 憑證完成後，就會開始跨所有 Azure 資料中心複寫供應項目。 複寫通常需要 24-48 小時才能完成，但依據 VHD 大小而定，最長可能需要一週的時間。 不過，如果您的要求只有行銷變更，複寫會更快速。
    > - 當複寫完成後，供應項目就會在 [Azure 入口網站](http:/portal.azure.com)公開發行。 屆時在發佈入口網站中的狀態會變成「已預備」。 只有當您使用與預備供應項目所用的訂用帳戶相關的電子郵件識別碼時，才能在 [Azure 入口網站](http:/portal.azure.com) 中看到已預備的供應項目。

1. 使用上一個步驟中列出的其中一個 Azure 訂用帳戶登入 [Azure Preview 入口網站](https://portal.azure.com) 。
2. 尋找您的供應項目，並驗證您的 VM 映像點：
   
   * 請確定該行銷內容可在 MarketPlace 中正確顯示。
   * VM 映像的端對端部署
     
      ![img-map-portal](media/marketplace-publishing-push-to-staging/pubportal-mapping-azure-portal.jpg)

> [!IMPORTANT]
> 您的供應項目將留在預備環境中，直到您透過發佈入口網站 ([發佈] 索引標籤 > 按一下 [要求核准以推送至生產環境] 按鈕) 通知 Microsoft 您已準備好推送至生產環境為止。 這是所有團隊成員為您即將列出的供應項目做好萬全準備並徹底檢查的最佳時機。
> 
> 預備平台是為了讓發行者在預覽模式下測試供應項目所設計。 我們強烈建議您勿將此平台使用於商業用途。
> 
> 

## <a name="next-steps"></a>後續步驟
現在您的供應項目「預備」好了，您已經測試過它的功能和行銷內容，您可以繼續進行最終的發佈階段，即 **步驟 4**： [將您的供應項目部署至 Marketplace](marketplace-publishing-push-to-production.md)。

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

