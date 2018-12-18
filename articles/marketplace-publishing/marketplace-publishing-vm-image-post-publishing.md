---
title: 管理 Azure Marketplace 上的虛擬機器映像 | Microsoft Docs
description: 如何在初始發佈後管理 Azure Marketplace 上的虛擬機器映像的詳細指南
services: Azure Marketplace
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: cc8648d4-59c2-4678-b47d-992300677537
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 08/03/2016
ms.author: hascipio;
ms.openlocfilehash: e1f90650e71345957c2d353774cb8bef62c1868b
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39715010"
---
# <a name="post-production-guide-for-virtual-machine-offers-in-the-azure-marketplace"></a>關於 Azure Marketplace 中的虛擬機器供應項目的後期製作指南
本文說明如何更新 Azure Marketplace 中已上線的虛擬機器供應項目。 它會引導您進行將一或多個新 SKU 新增至現有供應項目的程序。 同時引導您進行從 Marketplace 移除上線虛擬機器供應項目或 SKU 的程序。

供應項目/SKU 在 [Azure 入口網站](http://portal.azure.com)預備完成後，您就無法變更下列文字方塊：

* **供應項目識別碼**︰在發佈入口網站中，移至 [虛擬機器] 並選取您的供應項目。 然後按一下 [VM 映像] > [供應項目識別碼]。
* **SKU 識別碼**︰在發佈入口網站中，移至 [虛擬機器] 並選取您的供應項目。 然後按一下 [SKU] > [新增 SKU]。
* **發行者命名空間**︰在發佈入口網站中，移至 **[虛擬機器]** > **[逐步解說]** > **[告知我們您的公司]** (請見「步驟 2：註冊您的公司」) > **[發行者命名空間]** > **[命名空間** ]。

供應項目/SKU 列在 [Marketplace](http://azure.microsoft.com/marketplace) 後，您就無法變更下列文字方塊：

* **供應項目識別碼**︰在發佈入口網站中，移至 [虛擬機器] 並選取您的供應項目。 然後按一下 [VM 映像] > [供應項目識別碼]。
* **SKU 識別碼**︰在發佈入口網站中，移至 [虛擬機器] 並選取您的供應項目。 然後按一下 [SKU] > [新增 SKU]。
* **發行者命名空間**︰在發佈入口網站中，移至 **[虛擬機器]** > **[逐步解說]** > **[告知我們您的公司]** (請見「步驟 2 註冊」) > **[發行者命名空間]** > **[命名空間]** 。
* **連接埠**︰在發佈入口網站中，移至 [虛擬機器] 並選取您的供應項目。 然後按一下 [VM 映像] > [開啟連接埠]。
* **已列出 SKU 的價格變更**
* **已列出 SKU 的計費模式變更**
* **移除已列出 SKU 的計費區域**
* **變更已列出 SKU 的資料磁碟計數**

## <a name="update-the-technical-details-of-a-sku"></a>更新 SKU 的技術詳細資料
若要將新版本新增到已列出的 SKU，然後重新發佈供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [VM 映像] 索引標籤。
4. 在 [SKU] 區段中，找出您想要更新的 SKU。
5. 新增新的 SKU 版本號碼，然後按一下 **+** 按鈕。 新版本應該是 X.Y.Z 格式，其中 X、Y 和 Z 是整數。 只能以累加方式變更版本。
6. 在 [OS VHD URL]  方塊中，輸入針對作業系統 VHD 所建立的共用存取簽章 URI，然後儲存變更。

   > [!IMPORTANT]
   > 您不能增加/減少已列出的 SKU 的資料磁碟計數。 在此案例中，您需要建立新的 SKU。 如需詳細指引，請參閱[在已列出的供應項目下新增 SKU](#add-a-new-sku-under-a-listed-offer)一節。
   >
   >
7. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
8. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![VM 映像](media/marketplace-publishing-vm-image-post-publishing/img01_07.png)

## <a name="update-the-nontechnical-details-of-an-offer-or-a-sku"></a>更新供應項目或 SKU 的技術性詳細資料
您可以更新 Marketplace 中已上線的供應項目或 SKU 的非技術性 (行銷、法律、支援、類別) 詳細資料。

### <a name="update-the-offer-description-and-logos"></a>更新供應項目描述和標誌
若要更新供應項目詳細資訊並重新發佈您的供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [行銷] 索引標籤。
4. 按一下 [英文 (美國)]。
5. 按一下 [詳細資料] 索引標籤。在 [描述] 區段中，更新供應項目的 [標題]、[摘要] 和 [完整摘要]，然後儲存變更。

   > [!NOTE]
   > 當您更新 SKU 詳細資料時，請注意下列限制︰ 
   * 請不要對供應項目描述和 SKU 描述輸入重複的文字。
   * 請不要對 SKU 標題和供應項目完整摘要輸入重複的文字。 
   * 請不要對 SKU 標題和供應項目摘要輸入重複的文字。
   >
   >

    ![詳細資料](media/marketplace-publishing-vm-image-post-publishing/img02.1_05.png)
6. 在 [詳細資料] 索引標籤的 [標誌] 區段中，更新標誌。 請確定標誌遵循 [Azure Marketplace 指導方針](marketplace-publishing-push-to-staging.md#step-1-provide-marketplace-marketing-content)。

   > [!NOTE]
   > 主圖圖示是選擇性的。 您可以選擇不上傳主圖圖示。 然而，上傳主圖圖示後，就無法從發佈入口網站中刪除。 請遵循[主圖圖示指導方針](marketplace-publishing-push-to-staging.md#step-1-provide-marketplace-marketing-content)。
   >
   >
7. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
8. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![標誌](media/marketplace-publishing-vm-image-post-publishing/img02.1_08.png)

### <a name="update-the-sku-description"></a>更新 SKU 描述
若要更新 SKU 詳細資訊並重新發佈您的供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [行銷] 索引標籤。
4. 按一下 [英文 (美國)]。
5. 按一下 [方案] 。在 [SKU] 區段中，更新 SKU 的 [標題]、[摘要] 和 [描述]，然後儲存變更。

   > [!NOTE]
   > 當您更新 SKU 詳細資料時，請注意下列限制︰ 
   * 請不要對供應項目描述和 SKU 描述輸入重複的文字。 
   * 請不要對 SKU 標題和供應項目完整摘要輸入重複的文字。 
   * 請不要對 SKU 標題和供應項目摘要輸入重複的文字。
   >
   >
6. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
7. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![方案](media/marketplace-publishing-vm-image-post-publishing/img02.2_07.png)

### <a name="change-existing-links-or-add-new-links"></a>變更現有連結或新增連結
若要變更現有連結或新增連結，然後重新發佈您的供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [行銷] 索引標籤。
4. 按一下 [英文 (美國)]。
5. 按一下 [連結] 索引標籤。
6. 若要新增連結，請在 [連結] 區段中，按一下 [+ 新增連結]。 在 [新增連結] 對話方塊中，輸入連結的 [標題] 和 [URL]，然後儲存變更。 您可以輸入任何包含資訊來協助客戶的連結。
7. 若要更新或刪除現有的連結，請選取連結，然後按一下 [編輯] 按鈕或 [刪除] 按鈕。

   > [!NOTE]
   > 確保您在此區段中輸入的連結可正常運作，因為在生產環境要求處理期間會驗證這些連結。
   >
   >
8. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
9. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![連結](media/marketplace-publishing-vm-image-post-publishing/img02.3_09-01.png)

    ![新增連結](media/marketplace-publishing-vm-image-post-publishing/img02.3-2.png)

### <a name="change-an-existing-sample-image-or-add-a-new-sample-image"></a>變更現有範例映像或新增範例映像
若要變更現有範例映像或新增範例映像，然後重新發佈您的供應項目，請遵循下列步驟︰

> [!NOTE]
> 只有一個範例映像會顯示在 [Azure 入口網站](https://portal.azure.com)中。
>
>

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [行銷] 索引標籤。
4. 按一下 [英文 (美國)]。
5. 按一下 [範例映像] 索引標籤。
6. 若要新增範例映像，請在 [範例映像] 區段中，按一下 [上傳新映像] ，然後儲存變更。

   > [!NOTE]
   > 包含範例映像是選擇性作業。
   >
7. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
8. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![範例映像](media/marketplace-publishing-vm-image-post-publishing/img02.4_09.png)

### <a name="update-the-legal-content"></a>更新法律聲明內容
若要更新法律聲明內容並重新發佈您的供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [行銷] 索引標籤。
4. 按一下 [英文 (美國)]。
5. 按一下 [法律聲明] 索引標籤。在 [法律聲明] 索引標籤中，更新使用原則/條款。 在 [使用條款] 方塊中輸入或貼上原則/條款，然後儲存變更。
6. 法律使用條款的字元限制為 1 百萬個字元。
7. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
8. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![法律](media/marketplace-publishing-vm-image-post-publishing/img02.5_08.png)

### <a name="update-the-support-information"></a>更新支援資訊
若要更新支援資訊並重新發佈您的供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [支援] 索引標籤。
4. 在 [工程連絡人] 區段中，更新工程連絡人詳細資料。 這些詳細資料僅用於合作夥伴與 Microsoft 之間的內部通訊。
5. 在 [客戶支援] 區段中，更新支援連絡人詳細資料和 [支援 URL]。 這些詳細資料僅用於合作夥伴與 Microsoft 之間的內部通訊。

   > [!NOTE]
   > 如果您只想要提供電子郵件支援，請在 [客戶支援] 區段中輸入虛設的電話號碼。 在此情況下，會改用您提供的電子郵件。
   >
   >
6. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
7. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![支援](media/marketplace-publishing-vm-image-post-publishing/img02.6_07.png)

### <a name="update-the-categories"></a>更新類別
若要更新供應項目的類別區段，然後重新發佈供應項目，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [類別] 索引標籤。
4. 在 [類別]  區段中，更新供應項目的類別，然後儲存變更。 您最多可以為 Azure Marketplace 資源庫選擇五個類別。
5. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
6. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![類別](media/marketplace-publishing-vm-image-post-publishing/img02.7_06.png)

## <a name="add-a-new-sku-under-a-listed-offer"></a>在已列出的供應項目下新增 SKU
若要在上線的供應項目中新增 SKU，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [SKU] 索引標籤。然後按一下 [新增 SKU]。 
4. 在對話方塊中，輸入小寫的 [SKU 識別碼]。 如果您想要以 BYOL 計費模式發佈新的 SKU，請選取 [自備授權 (BYOL) 計費模式] 核取方塊。 否則，清除此核取方塊。 按一下核取記號以建立新的 SKU。 如果您並未選取 BYOL 計費模式，則計費模式會自動設為「每小時」。 如果您想要「每小時」計費模式的 30 天免費試用，請選取 [有免費試用可用嗎?] 的 [一個月]。 否則，請選取 [無試用]。 (只有在建立新的 SKU 時尚未選取 BYOL，[有免費試用可用嗎?] 才會出現。)

   > [!IMPORTANT]
   > 「只有」當您獲准發佈方案範本時，[從 Marketplace 隱藏此 SKU，因為應該永遠透過方案範本購買它] 才會為 [是]。 否則，這個選項應該永遠都是 [否]。
   >
   >
4. 在左側功能表中，按一下 [VM 映像] 索引標籤並找出您已建立的新 SKU。
5. 若要設定新的 SKU，請參閱[取得 VM 映像的憑證](marketplace-publishing-vm-image-creation.md#5-obtain-certification-for-your-vm-image)中的指引。
6. 若要為新的 SKU 新增行銷資料，請參閱[提供 Marketplace 行銷內容](marketplace-publishing-push-to-staging.md#step-1-provide-marketplace-marketing-content)。
7. 若要為新的 SKU 新增價格資訊，請參閱[設定價格](marketplace-publishing-push-to-staging.md#step-2-set-your-prices)。
8. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
9. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

    ![SKU](media/marketplace-publishing-vm-image-post-publishing/img03_09-01.png)

    ![新增 SKU](media/marketplace-publishing-vm-image-post-publishing/img03_09-02.png)

## <a name="change-the-data-disk-count-for-a-listed-sku"></a>變更已列出 SKU 的資料磁碟計數
您不能增加/減少已列出的 SKU 的資料磁碟計數。 在此案例中，您需要建立新的 SKU。 如需詳細指引，請參閱[在已列出的供應項目下新增 SKU](#add-a-new-sku-under-a-listed-offer)一節。

## <a name="delete-a-listed-offer-from-the-marketplace"></a>從 Marketplace 刪除已列出的供應項目
要求移除上線供應項目時，需要注意幾個層面。 若要取得支援小組的指引，以從 Marketplace 移除已列出的供應項目，請遵循下列步驟︰

1. 在[建立事件](https://support.microsoft.com/en-us/getsupport?wf=0&tenant=ClassicCommercial&oaspworkflow=start_1.0.0.0&locale=en-us&supportregion=en-us&pesid=15635&ccsid=635993707583706681)頁面上引發支援票證。

2. 選取 [管理供應項目] 作為 [問題類型]，然後選取 [修改已在生產中的供應項目和/或 SKU] 作為 [類別]。
3. 提交要求。

支援小組會引導您完成供應項目/SKU 刪除程序。

> [!NOTE]
> 當供應項目的狀態為 [草稿] \(但不是 [預備] 或 [生產]) 時，您可隨時將其刪除。 在 [歷程記錄] 索引標籤上，按一下 [捨棄草稿]。
>
>

## <a name="delete-a-listed-sku-from-the-marketplace"></a>從 Marketplace 刪除已列出的 SKU
若要從 Marketplace 刪除已列出的 SKU，請遵循下列步驟︰

1. 登入[發佈入口網站](https://publish.windowsazure.com)。

2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側窗格中，按一下 [SKU] 索引標籤。
4. 選取您要刪除的 SKU，然後按一下 [刪除] 按鈕。
5. 移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈供應項目。
6. 在 Marketplace 中重新發佈供應項目之後，會從 Marketplace 和 Azure 入口網站刪除 SKU。

## <a name="delete-the-current-version-of-a-listed-sku-from-the-marketplace"></a>從 Marketplace 刪除已列出 SKU 的目前版本
若要從 Marketplace 刪除已列出 SKU 的目前版本，請遵循下列步驟︰ 

1. 登入[發佈入口網站](https://publish.windowsazure.com)。

2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [VM 映像] 索引標籤。
4. 選取您要刪除其目前版本的 SKU，然後按一下 [刪除] 按鈕。
5. 移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈供應項目。
6. 在 Marketplace 中重新發佈供應項目之後，會從 Marketplace 和 Azure 入口網站刪除已列出 SKU 的目前版本。 SKU 會接著回復為先前的版本。

## <a name="revert-the-listing-price-to-production-values"></a>將列出價格還原成生產環境值
若要將列出價格還原成生產環境值，請遵循下列步驟：

1. 登入[發佈入口網站](https://publish.windowsazure.com)。
2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [價格] 索引標籤。
4. 選取您想要重設其價格的區域。

    ![價格區域](media/marketplace-publishing-vm-image-post-publishing/img08-04.png)
5. 對於每小時計費模式的 SKU，請重設所有核心的價格，因為它們處於所選區域的生產環境。 對於 BYOL 計費模式的 SKU，請在 [外部授權的 (BYOL) SKU 可用性] 區段中核取該 SKU 的核取方塊，以在該區域提供 SKU。

    ![定價模式](media/marketplace-publishing-vm-image-post-publishing/img08-05.png)
6. 按一下 [依據美國價格自動訂定其他市場價格] 。

   > [!NOTE]
   > 視選取的區域而定，按鈕的標籤可能會有所不同。 因為我們選取了美國，所以此按鈕會標示為 [依據美國價格自動訂定其他市場價格]。
   >
   >

    ![自動定價](media/marketplace-publishing-vm-image-post-publishing/img08-06.png)
7. 在自動定價精靈的第 1 頁，選擇基底市場並按一下**箭號**按鈕。

    ![基底市場](media/marketplace-publishing-vm-image-post-publishing/img08-07.png)
8. 在第 2 頁，選擇服務方案和計量 (核心)，然後按一下**箭號**按鈕。

    ![服務方案和計量](media/marketplace-publishing-vm-image-post-publishing/img08-08.png)
9. 在第 3 頁，按一下 [全部切換] 以選取所有區域。 或者，您可以手動選取特定區域的核取方塊。 按一下**箭號**按鈕。

    ![選擇市場](media/marketplace-publishing-vm-image-post-publishing/img08-09.png)
10. 在第 4 頁，檢閱匯率，然後按一下 [完成]。 精靈會根據您的選擇重設價格。

11. 在 [價格] 索引標籤上，按一下 [檢視摘要和變更]。
    對於 [檢視版本]，選取 [草稿]，以及針對 [比較項目]，選取 [生產]。 如果看不出價格差異，即表示價格已順利還原成生產環境值。

    ![摘要檢視和變更](media/marketplace-publishing-vm-image-post-publishing/img08-11.png)
12. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
13. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

## <a name="revert-the-billing-model-to-production-values"></a>將計費模式還原成生產環境值
若要將計費模式還原成生產環境值，請遵循下列步驟：

1. 登入[發佈入口網站](https://publish.windowsazure.com)。

2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [SKU] 索引標籤。
4. 按一下 [編輯] 按鈕以還原計費模式。 在開啟的視窗中，選取或清除 [計費與授權於 Azure 外部進行 (亦即自備授權)] 核取方塊。

    ![編輯計費](media/marketplace-publishing-vm-image-post-publishing/img09-04.png)
5. 請本文「將列出價格還原成生產環境值」中的步驟。
6. 移至 [發佈] 索引標籤，然後按一下 [推送至預備環境]。 如需在預備環境中測試供應項目的詳細指引，請參閱[針對 Marketplace 測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)。
7. 在預備環境中測試您的供應項目之後，請移至發佈入口網站中的 [發佈] 索引標籤。 按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

## <a name="revert-the-visibility-setting-of-a-listed-sku-to-the-production-value"></a>將已列出 SKU 的可見性設定還原成生產環境值
若要將已列出 SKU 的可見性設定還原成生產環境值，請遵循下列步驟：

1. 登入[發佈入口網站](https://publish.windowsazure.com)。

2. 移至 [虛擬機器] 索引標籤，然後選取您的供應項目。
3. 在左側功能表中，按一下 [SKU] 索引標籤。
4. 選取您的 SKU，將 SKU 的可見性設定還原成生產環境值。

    ![可見性](media/marketplace-publishing-vm-image-post-publishing/img10-04.png)
5. 在變更完成後，按一下 [要求核准以推送至生產環境]，以在 Marketplace 中重新發佈您的供應項目。

## <a name="see-also"></a>另請參閱
* [使用者入門：將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)
* [了解付款報告](marketplace-publishing-report-payout.md)
* [變更雲端解決方案提供者轉售商獎勵](marketplace-publishing-csp-incentive.md)
* [針對 Marketplace 中常見的發佈問題進行疑難排解](marketplace-publishing-support-common-issues.md)
* [以發佈者身分取得支援](marketplace-publishing-get-publisher-support.md)
* [建立內部部署 VM 映像](marketplace-publishing-vm-image-creation-on-premise.md)
* [在 Azure Preview 入口網站中建立執行 Windows 的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
