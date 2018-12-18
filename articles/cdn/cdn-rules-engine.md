---
title: 使用 Azure CDN 規則引擎覆寫 HTTP 行為 | Microsoft Docs
description: 規則引擎可讓您自訂 Azure CDN 處理 HTTP 要求的方式，例如封鎖傳遞特定類型的內容、定義快取原則及修改 HTTP 標頭。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 625a912b-91f2-485d-8991-128cc194ee71
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/11/2018
ms.author: magattus
ms.openlocfilehash: 2ac43b472758f3403bc87bf3d64321eb97109f53
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092382"
---
# <a name="override-http-behavior-using-the-azure-cdn-rules-engine"></a>使用 Azure CDN 規則引擎覆寫 HTTP 行為
[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>概觀
Azure CDN 規則引擎可讓您自訂 HTTP 要求的處理方式。 例如，封鎖特定內容類型的傳遞、定義快取原則，或修改 HTTP 標頭。 本教學課程會示範如何建立用以變更 CDN 資產之快取行為的規則。 如需規則引擎語法的詳細資訊，請參閱 [Azure CDN 規則引擎參考](cdn-rules-engine-reference.md)。

## <a name="access"></a>Access
若要存取規則引擎，您必須先從 [CDN 設定檔] 頁面頂端選取 [管理]，以存取 Azure CDN 管理頁面。 視您的端點是否已針對動態網站加速 (DSA) 最佳化，然後存取具有您的端點類型適用之規則集的規則引擎：

- 已針對一般 Web 傳遞或其他非 DSA 最佳化進行最佳化的端點： 
    
    選取 [HTTP 大型] 索引標籤，然後選取 [規則引擎]。

    ![HTTP 的規則引擎](./media/cdn-rules-engine/cdn-http-rules-engine.png)

- 已針對 DSA 最佳化的端點： 
    
    選取 [ADN] 索引標籤，然後選取 [規則引擎]。 
    
    ADN 是 Verizon 用來指定 DSA 內容的詞彙。 您設定檔中未針對 DSA 最佳化的所有端點都會忽略您在此處建立的任何規則。 

    ![DSA 的規則引擎](./media/cdn-rules-engine/cdn-dsa-rules-engine.png)

## <a name="tutorial"></a>教學課程
1. 從 [CDN 設定檔] 頁面選取 [管理]。
   
    ![CDN 設定檔 [管理] 按鈕](./media/cdn-rules-engine/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。

2. 選取 [HTTP 大型] 索引標籤，然後選取 [規則引擎]。
   
    新規則的選項隨即顯示。
   
    ![CDN 新規則選項](./media/cdn-rules-engine/cdn-new-rule.png)
   
   > [!IMPORTANT]
   > 多項規則的列出順序會影響規則的處理方式。 後一項規則可能會覆寫前一項規則所指定的動作。
   > 

3. 在 [名稱/描述]  文字方塊中輸入名稱。

4. 識別要套用此規則的要求類型。 使用預設比對條件 [一律]。 
   
   ![CDN 規則比對條件](./media/cdn-rules-engine/cdn-request-type.png)
   
   > [!NOTE]
   > 下拉式清單中提供多個比對條件。 如需目前所選比對條件的相關資訊，請選取其左邊的藍色資訊圖示。
   > 
   >  如需詳細的條件運算式清單，請參閱[規則引擎條件運算式](cdn-rules-engine-reference-match-conditions.md)。
   >  
   > 如需詳細的比對條件清單，請參閱[規則引擎比對條件](cdn-rules-engine-reference-match-conditions.md)。
   > 
   > 

5. 若要新增功能，請選取 [功能] 旁的 **+** 按鈕。  在左側下拉式清單中，選取 [強制執行內部最大壽命]。  在出現的文字方塊中，輸入 **300**。 請勿變更其餘的預設值。
   
   ![CDN 規則功能](./media/cdn-rules-engine/cdn-new-feature.png)
   
   > [!NOTE]
   > 下拉式清單中提供多項功能。 如需目前所選功能的相關資訊，請選取其左邊的藍色資訊圖示。 
   >
   > 在 [強制執行內部最大壽命] 中，資產的 `Cache-Control` 和 `Expires` 標頭會遭到覆寫，以控制 CDN 邊緣節點何時要從原始來源重新整理資產。 在此範例中，CDN 邊緣節點會先快取資產 300 秒 (或 5 分鐘)，再從其原始來源重新整理資產。
   > 
   > 如需功能的詳細清單，請參閱[規則引擎功能](cdn-rules-engine-reference-features.md)。
   > 
   > 

6. 選擇 [新增] 以儲存新規則。  新規則現在正在等待核准。 核准之後，狀態會從 [暫止 XML] 變更為 [使用中 XML]。
   
   > [!IMPORTANT]
   > 規則變更可能需要最多 10 分鐘才能傳遍 Azure CDN。
   > 
   > 

## <a name="see-also"></a>另請參閱
* [Azure CDN 概觀](cdn-overview.md)
* [規則引擎參考](cdn-rules-engine-reference.md)
* [規則引擎比對條件](cdn-rules-engine-reference-match-conditions.md)
* [規則引擎條件運算式](cdn-rules-engine-reference-conditional-expressions.md)
* [規則引擎功能](cdn-rules-engine-reference-features.md)
* [Azure Fridays: Azure CDN's powerful new Premium Features (影片：Azure 星期五：Azure CDN 強大的新高階功能)](https://azure.microsoft.com/documentation/videos/azure-cdns-powerful-new-premium-features/)