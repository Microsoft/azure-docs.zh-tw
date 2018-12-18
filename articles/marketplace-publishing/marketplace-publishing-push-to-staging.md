---
title: 準備和測試供應項目以部署至 Azure Marketplace | Microsoft Docs
description: 將供應項目部署至 Azure Marketplace 之前，關於提供行銷內容、設定價格方案和測試供應項目的詳細指示。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 3ccd2448-895b-477e-adf6-ab655a21d2fa
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 08/17/2016
ms.author: hascipio
ms.openlocfilehash: 7db86716cdf8f9eb921c3c1813970acae7a3016b
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39714954"
---
# <a name="complete-the-offer-creation-with-marketing-content"></a>使用行銷內容完成供應項目建立程序
在發佈程序的這個步驟中，您需要在 Azure Marketplace 中提供特定的行銷內容，以及關於您的供應項目和 (或) SKU 的詳細資料。 例如，您將提供產品的描述、公司標誌、價目表、方案的詳細資料，以及其他將您的供應項目和 (或) SKU 推送至預備環境的必要資訊。 此資訊會做為 Azure 入口網站中的行銷內容。 您將在[發佈入口網站][link-pubportal]中開始進行此程序。

## <a name="step-1-provide-marketplace-marketing-content"></a>步驟 1：提供 Marketplace 行銷內容
**英文是預設值，並且是唯一支援的語言。** 請確定欄位中提供的所有資訊都是英文。 在您進入預備環境之前，所有資訊皆可隨時編輯。

1. 移至發佈入口網站：[https://publish.windowsazure.com](https://publish.windowsazure.com)。
2. 在左側功能表上，按一下 [行銷]  索引標籤。
3. 在主面板中，按一下 [英文 (美國)]  按鈕。
   
   > [!IMPORTANT]
   > 所有欄位都必須填入內容 (包含影像)，才能推送至預備環境。
   > 
   > 

### <a name="details-and-plans"></a>詳細資料和方案
1. 在 [詳細資料]  索引標籤下方，輸入供應項目標題 (最多 50 個字元)、供應項目摘要 (最多 100 個字元)、供應項目完整摘要 (最多 256 個字元)、供應項目描述 (最多 1300 個字元)、標誌
2. 在 [方案]  索引標籤下方，輸入方案標題 (最多 50 個字元)、方案摘要 (最多 100 個字元)、方案描述 (最多 2000 個字元)。
   
   > [!NOTE]
   > 您可以使用下列 HTML 標記來格式化供應項目與方案的摘要、完整摘要及描述。 允許的 HTML 標記為 h1、h2、h3、h4、h5、p、ol、ul、li、a[target|href]、strong、em、b、i。
   > 
   > 
3. 請不要在供應項目和方案描述下方輸入重複文字。
4. 請不要在方案標題和供應項目完整摘要下方輸入重複文字。
5. 請不要在方案標題和供應項目摘要下方輸入重複文字。
6. 對於含有多個方案的供應項目，請勿輸入相同的方案標題。
7. 以 PNG 格式上傳必要規格的影像 (在發佈入口網站中提及)，每種大小各一個。
8. 請確定標誌遵循下面所提及的 Azure Marketplace 標誌指導方針。
   
   ![繪圖](media/marketplace-publishing-push-to-staging/pubportal-marketingcontent-details-02.png)

**Azure Marketplace 標誌指導方針**

發佈入口網站中所上傳的所有標誌都應該遵循下列指導方針︰

* Azure 設計具有簡單的調色盤。 請保持標誌上具有最少的主要和次要色彩數目。
* Azure 入口網站的佈景主題色彩是白色與黑色。 因此請避免使用這些色彩做為您標誌的背景色彩。 請使用會讓您的標誌在 Azure 入口網站顯得突出的某些色彩。 建議您使用簡單的主要色彩。 **如果您使用透明背景，請確定標誌/文字不是白色、黑色或藍色。**
* 請不要在標誌上使用漸層背景。
* 避免在標誌上放置文字 (甚至是公司或品牌名稱)。 您標誌的外觀與風格應該是「一般」，而且應該避免漸層。
* 標誌不應該自動進行縮放。
* 小型標誌的大小應該是 40 X 40 像素
* 中型標誌的大小應該是 90 X 90 像素
* 大型標誌的大小應該是 115 X 115 像素
* 寬標誌的大小應該是 255 X 115 像素
* 主圖標誌的大小應該是 815 X 290 像素

> [!NOTE]
> 主圖標誌為選用項。 發行者可以選擇不上傳主圖標誌。 但主圖圖示一旦上傳，即無法從發佈入口網站中刪除。 屆時合作夥伴必須遵循主圖圖示的 Azure Marketplace 指導方針。
> 
> 

  ![繪圖](media/marketplace-publishing-push-to-staging/pubportal-marketingcontent-details-03.png)

**主圖標誌圖示的其他指導方針 (選擇性)**

* 主圖標誌為選用項。 發行者可以選擇不上傳主圖標誌。 **但主圖圖示一旦上傳，即無法從發佈入口網站中刪除。屆時合作夥伴必須遵循主圖圖示的 Azure Marketplace 指導方針，否則供應項目將無法核准進入生產環境。**
* 發行者顯示名稱、方案標題和供應項目完整摘要會以白色字型色彩顯示。 因此，您應避免在主圖圖示的背景使用淺色系。 主圖圖示不允許使用黑色、白色和透明背景。
* 供應項目列出之後，會以程式設計方式將發行者顯示名稱、方案標題、供應項目完整摘要和 [建立] 按鈕內嵌在主圖標誌內。 因此您在設計主圖標誌時不應輸入任何文字。 請將右邊直接留白，因為我們將會透過程式設計方式，在該處納入文字 (也就是發行者顯示名稱、方案標題、供應項目完整摘要)。 文字右側的空白空間應為 415 x 100 (從左邊開始位移 370px)。
  
  ![繪圖](media/marketplace-publishing-push-to-staging/pubportal-herobanner.png)

### <a name="links"></a>連結
在左側工具列的 [連結]  索引標籤上，輸入任何可能對客戶有幫助的資訊連結。 輸入每個連結的名稱和 URL。

![繪圖](media/marketplace-publishing-push-to-staging/pubportal-marketingcontent-link-01.png)

### <a name="sample-images-optional"></a>範例影像 (選擇性)
> [!NOTE]
> 加入範例影像是選擇性步驟。
> 雖然您可以在發佈入口網站上傳多個範例影像，但只有一個影像會顯示在 Azure 入口網站 (系統隨機選取)。 基於這個理由，建議您最多上傳一個範例影像。
> 
> 

在左側功能表的 [範例影像] 索引標籤上，按一下 [上傳新的影像] 以上傳新的影像。 如果目前已有影像而您想要取代它，請按一下 [取代影像] 。

![繪圖](media/marketplace-publishing-push-to-staging/pubportal-marketingcontent-sampleimg-01.png)

### <a name="legal"></a>法律
在 [法律聲明]  索引標籤上，提供您的使用原則/條款連結。 在大型 [使用條款]  方塊中輸入或貼上條款。 法律使用條款的字元限制為 1,000,000 個字元。

![繪圖](media/marketplace-publishing-push-to-staging/pubportal-marketingcontent-legal-01.png)

 對於「虛擬機器」供應項目，一旦供應項目/SKU於 Azure 入口網站預備完成，您就無法變更以下欄位：

* **供應項目識別碼︰**[發佈入口網站 -> 虛擬機器 -> 您的供應項目 -> VM 映像索引標籤 -> 供應項目識別碼]
* **SKU 識別碼︰**[發佈入口網站 -> 虛擬機器 -> 選取您的供應項目 -> SKU 索引標籤 -> 新增 SKU]
* **發行者命名空間︰**[發佈入口網站 -> 虛擬機器 -> 逐步解說索引標籤 -> 告知我們您的公司 (請見「步驟 2：註冊您的公司」) -> 發行者命名空間 -> 命名空間]

對於「虛擬機器」供應項目，一旦供應項目/SKU 列於 Azure Marketplace，您就無法變更以下欄位：

* **供應項目識別碼︰**[發佈入口網站 -> 虛擬機器 -> 選取您的供應項目 -> VM 映像 -> 供應項目識別碼]
* **SKU 識別碼︰**[發佈入口網站 -> 虛擬機器 -> 選取您的供應項目 -> SKU 索引標籤 -> 新增 SKU]
* **發行者命名空間︰**[發佈入口網站 -> 虛擬機器 -> 逐步解說索引標籤 -> 告知我們您的公司 (請見「步驟 2：註冊」) -> 發行者命名空間 -> 命名空間]
* **連接埠：**[發佈入口網站 -> 虛擬機器 -> 您的供應項目 -> VM 映像索引標籤 -> 開啟連接埠]
* **已列出 SKU 的價格變更**
* **已列出 SKU 的計費模式變更**
* **移除已列出 SKU 的計費區域**
* **變更已列出 SKU 的資料磁碟計數**

## <a name="step-2-set-your-prices"></a>步驟 2：設定價格
### <a name="pricing-models"></a>定價模式
| 定價模式 | 說明 |
| --- | --- |
| 基本 |購買時支付的每月分期費率，例如 $10 美元/月 |
| 耗用量 (又稱為 使用量、計量器) |每次使用付費，由供應項目發行者定義。 無法定義每一基座、每位使用者等等的超額部分，因為並沒有使用者分數的概念，也沒有比例分攤的功能。 使用量由合作夥伴每小時報告一次。 客戶依每月計費週期付款，而非預付式的月繳方案。 |
| 免費試用 |客戶在一段有限期間內可以免費使用，之後就依正常費率付費。 |
| 免費層 |方案一律免費。 |
| 方案移轉 (又稱為 轉換或升級/降級) |使用者從目前方案改用另一種可接受方案的概念，由合作夥伴定義。 |

**依供應項目類型的價格模式**

> [!IMPORTANT]
> 依供應項目類型可用的某些價格模式。 請參閱下表。
> 
> 

|  | 僅基本 | 僅耗用量 | 基本 + 耗用量 |
| --- | --- | --- | --- |
| 虛擬機器映像 |否 |是 |否 |
| 開發人員服務 |是 |yes |是 |

### <a name="21-set-your-vm-prices"></a>2.1. 設定 VM 價格
目前對於虛擬機器，我們提供下列 **3 種計費模式**

* **每小時︰** 依據發行者針對 VM 大小設定的費率，以每小時為基礎向客戶收費。 如果是 SKU 的 **每小時計費** 模式，總價格會是發行者收取的軟體成本和 Microsoft 收取的基礎結構成本的總和。 當客戶考慮購買時，總成本將顯示為每小時和每月價格以供客戶參考 (請參閱以下螢幕擷取畫面)。 **發行者會收取 80% 的應收軟體成本。** 因此，在您設定 SKU 的價格之前，請先以此作計算。
  
    ![繪圖](media/marketplace-publishing-push-to-staging/img2.1-01.png)
* **免費試用版︰** 這是「每小時」模式的另一項好處。 此模式的客戶在 VM 部署後的前 30 天不需負擔任何軟體成本 (免費)。 在 30 天過後，他們就必須依據發行者在每小時模型中設定的費率，以每小時為基礎付費。
* **自備授權 (BYOL)：** 發行者管理 VM 上所執行軟體的授權。

**重要：** 一旦供應項目/SKU 列於 Azure Marketplace，您就無法變更以下欄位。

* **已列出 SKU 的價格變更**
* **已列出 SKU 的計費模式變更**
* **移除已列出 SKU 的計費區域**
* **變更已列出 SKU 的資料磁碟計數**
* **供應項目識別碼︰**[發佈入口網站 -> 虛擬機器 -> 選取您的供應項目 -> VM 映像 -> 供應項目識別碼]
* **SKU 識別碼︰**[發佈入口網站 -> 虛擬機器 -> 選取您的供應項目 -> SKU 索引標籤 -> 新增 SKU]
* **發行者命名空間︰**[發佈入口網站 -> 虛擬機器 -> 逐步解說索引標籤 -> 告知我們您的公司 (請見「步驟 2：註冊」) -> 發行者命名空間 -> 命名空間]
* **連接埠：**[發佈入口網站 -> 虛擬機器 -> 您的供應項目 -> VM 映像索引標籤 -> 開啟連接埠]

### <a name="sell-to-countries-of-the-sku"></a>SKU 的銷往國家/地區
您必須仔細考慮要公開發行您的 SKU 的國家/地區。 某些國家/地區歸類為「Microsoft 免稅」，其他則歸類為「ISV 免稅」。

* 在「Microsoft 免稅」國家/地區，Microsoft 會收集客戶的稅款並繳納 (免除) 給政府的稅款。
* 在「ISV 免稅」國家/地區，則是由合作夥伴負責收集客戶的稅款並繳納給政府。 如果您選擇在「ISV 免稅」國家/地區銷售，您必須有計算及繳納您所選國家/地區稅款的能力。

> [!NOTE]
> 除非您已在 [發佈入口網站](https://publish.windowsazure.com)設定您的 SKU 價格，否則將無法在那些國家/地區公開發行。 以下是設定「每小時」和 BYOL SKU 價格的指引。
> 
> 

### <a name="211-how-to-setup-hourly-pricing-model-for-a-sku"></a>2.1.1 如何設定 SKU 的每小時定價模式
請依照下列步驟來設定 SKU 的「每小時」定價模式︰

1. 登入 [發佈入口網站](https://publish.windowsazure.com)。
2. 瀏覽至 [虛擬機器]  索引標籤並選取您的供應項目。
3. 從左側功能表，按一下 [SKU]  索引標籤。
4. 請確定 SKU 標示為「每小時計費模式」。 如果不是，請按一下 [編輯]  按鈕來還原計費模式。 隨即開啟一個視窗。 取消核取 [計費與授權於 Azure 外部進行 (亦即自備授權)] 核取方塊並儲存變更。
5. 如果您想要啟用 SKU 部署前 30 天的免費試用版，請針對 [是否提供免費試用版?] 問題選取 [一個月] 選項。 否則請選取 [無試用版] 選項。 現在請依照下列步驟執行。
6. 從左側功能表，按一下 [價格]  索引標籤。
7. 選取您的基本區域。
   
   ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.1_07.png)
8. 設定所有核心的價格。 **即使您的 SKU 不支援，您仍然必須提供 SKU 所有核心的價格。**
   
    ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.1_08.png)
9. 手動設定其他區域的價格，或者您可以使用「自動定價」精靈，依據基本區域設定其他區域的價格。 若要使用「自動定價」精靈，請按一下 [依據美國價格自動訂定其他市場價格] 按鈕。  視選取的區域而定，按鈕的標籤可能會不同。 由於我們在建立這份文件時選取美國，因此以下螢幕擷取畫面中的按鈕標示為 [依據美國價格自動訂定其他市場價格]。
   
   ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.1_09.png)
10. 自動定價精靈隨即開啟。 第 1 頁會顯示基本市場的選項。 選擇您的區域，然後按一下 "->" 按鈕移至下一頁。
    
    ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.1_10.png)
11. 選取核心和方案的選項將顯示在第 2 頁。 選取所需的方案，然後按一下 [->] 按鈕。 按一下 [全部切換] 按鈕以選取所有**服務方案**和**計量器**，或您可以手動選取核取方塊。 **即使您的 SKU 不支援，您仍然必須提供 SKU 所有核心的價格。** 因此，請確定已選取所有核心大小。
    
    ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.1_11.png)
12. 第 3 頁會顯示市場/區域。 按一下 [全部切換] 按鈕以選取所有區域，或手動選取區域的方塊。 按一下 [->] 按鈕移至下一頁。 **注意：**「Microsoft 免稅國家/地區」以房屋形狀的符號表示。 如需詳細資訊，請參閱此頁面的＜SKU 的銷往國家/地區＞一節。
    
    ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.1_12.png)
13. 第 4 頁會顯示匯率。 按一下 [完成] 按鈕以完成步驟。

### <a name="212-how-to-setup-byol-pricing-model-for-a-sku"></a>2.1.2 如何設定 SKU 的 BYOL 定價模式
請依照下列步驟來設定 SKU 的 BYOL 定價模式︰

1. 登入 [發佈入口網站](https://publish.windowsazure.com)。
2. 瀏覽至 [虛擬機器]  索引標籤並選取您的供應項目。
3. 從左側功能表，按一下 [SKU]  索引標籤。
4. 請確定 SKU 標示為「自備授權 SKU」。 如果不是，請按一下 [編輯] 按鈕來還原計費模式。 隨即開啟一個視窗。 核取 [計費與授權於 Azure 外部進行 (亦即自備授權)] 核取方塊並儲存變更。
   
   ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.2_04.png)
5. 從左側功能表，按一下 [價格]  索引標籤。
6. 選取您的基本區域，在 [外部授權的 (BYOL) SKU 可用性] 區段下方核取該 SKU 的核取方塊，以在該區域公開發行 SKU (請參閱以下螢幕擷取畫面)。
   
   ![繪圖](media/marketplace-publishing-push-to-staging/img2.1.2_06.png)
7. 您可以手動設定於其他區域公開發行 SKU，也可以使用「自動定價」精靈來設定。 請參閱此頁面的 **2.1.1 如何設定 SKU 的每小時定價模式** 一節，其中第 9 到 13 點說明「自動定價」精靈的使用方式。

### <a name="22-set-your-developer-service-prices"></a>2.2. 設定開發人員服務價格
方案可以是基本 + 耗用量的任意組合，其中基本是每月價格，而超額部分是每次使用付費價格 (詳細資訊請見下文)。

**範例：** Contoso 開發人員服務供應項目

| 規劃 | 價格 | 包括 | 移轉路徑 |
| --- | --- | --- | --- |
| 免費 |$0 美元/月 |基本功能。 |可以移轉至其他方案 |
| Bronze |$10 美元/月 |基本功能和 1,000 個功能 X 的配額。 |可以移轉至 Bronze Plus、Silver 和 Gold 方案 |
| Bronze Plus |免費試用期間：$0 美元/月 + $0 美元/meter01 |基本功能和 10,000 個功能 X 的配額。功能 X 配額用完時，客戶可以透過 meter01，依每次使用付費。 |可以移轉至 Silver Plus 和 Gold 方案 |
| Bronze Plus |付費期間 (也稱為 免費試用版已過期)：$10/每月 + $0.05/meter01 |基本功能和 10,000 個功能 X 的配額。功能 X 配額用完時，客戶可以透過 meter01，依每次使用付費。 |可以移轉至 Silver Plus 和 Gold 方案 |
| Silver |$0.15 美元/meter01 |客戶可以透過 meter01 (適用於 X 功能)，依每次使用付費。 |可以移轉至 Bronze 和 Gold 方案 |
| Silver Plus |$20 美元/月 + $0.15 美元/meter01 + $0.01 美元/meter02 |基本功能及 10,000 個功能 X 和 100 個功能 Y 的配額。功能 X 配額用完時，客戶可以透過 meter01，依每次使用付費。  功能 Y 配額用完時，客戶可以透過 meter02，依每次使用付費。 |可以移轉至 Bronze Plus 和 Gold 方案 |
| Gold |$1000 美元/月 |10,000 個功能 X、1,000 個功能 Y 和無限制的功能 Z 的配額。 |可以移轉至所有方案，免費除外 |

## <a name="step-3-provide-support-information"></a>步驟 3：提供支援資訊
合約詳細資料僅用於合作夥伴與 Microsoft 之間的內部通訊。 支援 URL 可供一般客戶使用。

1. 移至發佈入口網站左側的 [支援]  標題。
2. 在 [工程連絡人] 下方輸入資訊。
3. 在 [客戶支援] 下方輸入資訊。 如果您只提供電子郵件支援，請輸入虛設的電話號碼，該內容會改用您提供的電子郵件。
4. 輸入支援 URL。

## <a name="step-4-choose-azure-marketplace-categories"></a>步驟 4：選擇 Azure Marketplace 類別
[類別]  索引標籤會提供選取項目的陣列。 您的供應項目可能會落在這些選取項目中，而且您最多可選取五個類別。

## <a name="how-your-marketing-will-appear"></a>行銷活動的呈現方式
以下是如何在 [Azure Marketplace 網站](https://azure.microsoft.com/marketplace/)上和 [Azure 入口網站](https://portal.azure.com)中使用供應項目行銷資訊的詳細檢視。

### <a name="azure-marketplace-website"></a>Azure Marketplace
![繪圖](media/marketplace-publishing-push-to-staging/acom-catalog-01.png)

![繪圖](media/marketplace-publishing-push-to-staging/acom-catalog-02.png)


  *Azure.com Marketplace 網站上的供應項目清單*

![繪圖](media/marketplace-publishing-push-to-staging/acom-listing-details-01.png)


  *Azure.com Marketplace 網站上的供應項目描述詳細資料*

![繪圖](media/marketplace-publishing-push-to-staging/acom-listing-details-02.png)


  *Azure.com Marketplace 網站上的供應項目定價詳細資料*

### <a name="azure-portal"></a>Azure 入口網站
![繪圖](media/marketplace-publishing-push-to-staging/azureportal-galleryblade-01.png)


  *Azure 入口網站中的供應項目清單*

![繪圖](media/marketplace-publishing-push-to-staging/azureportal-galleryblade-02.png)


  *Azure 入口網站中的供應項目描述詳細資料*

## <a name="next-steps"></a>後續步驟
既然已載入您的 Marketplace 內容，讓我們繼續在預備環境中測試您的供應項目。 不過，您必須從下列清單選取適當的供應項目類型，因為步驟會隨著供應項目類型而有所不同。

* 
  [在預備環境中測試您的 VM 供應項目](marketplace-publishing-vm-image-test-in-staging.md)
* [在預備環境中測試您的解決方案範本供應項目](marketplace-publishing-solution-template-test-in-staging.md)

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

[img-map-acom]:media/marketplace-publishing-push-to-staging/pubportal-mapping-acom.jpg
[img-map-portal]:media/marketplace-publishing-push-to-staging/pubportal-mapping-azure-portal.jpg
[img-map-link]:media/marketplace-publishing-push-to-staging/marketing-content-guide-links.jpg
[img-map-logo]:media/marketplace-publishing-push-to-staging/marketing-content-guide-logos.jpg
[img-map-title]:media/marketplace-publishing-push-to-staging/marketing-content-guide-publisher-offer.png

[link-pubportal]:https://publish.windowsazure.com
[link-push-to-production]:marketplace-publishing-push-to-production.md
