---
title: 從 Azure 上的自訂語音服務移轉端點的定價層 | Microsoft Docs
description: 了解如何將「認知服務」中「自訂語音服務」端點的部署從 S0 和 S1 層移轉成 S2 層。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 07/05/2017
ms.author: panosper
ms.openlocfilehash: a9bdb257137db0063d39f028a69e2164eccbdc31
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340455"
---
# <a name="migrate-deployments-to-the-new-pricing-model"></a>將部署移轉成新的定價模式

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

從 2017 年 7 月起，「自訂語音服務」提供[新的定價模式](https://azure.microsoft.com/pricing/details/cognitive-services/custom-speech-service/)。 新模式「更容易了解」、「成本計算更簡單」，且在規模調整上「更具有彈性」。 針對規模調整，Microsoft 導入了縮放單位的概念。 每個縮放單位可以處理五個並行要求。 在舊模式中，並行要求的規模是針對 S0 層設定為 5 個並行要求，針對 S1 層設定為 12 個並行要求。 我們已開放這些限制，以便在您的使用案例需求上，提供更大的彈性。

如果您執行舊的 S0 或 S1 層，建議您將現有的部署移轉成新的 S2 層。 新的 S2 層同時涵蓋 S0 和 S1 層。 您可以在下圖中查看可用的選項：

![[選擇您的定價層] 頁面](../../../media/cognitive-services/custom-speech-service/custom-speech-pricing-tier.png)

Microsoft 會以半自動方式處理移轉。 首先，您藉由選取新定價層來觸發移轉。 接著，我們會自動移轉您的部署。

下圖顯示舊層與縮放單位的對應：

| 層 | 並行要求數 (舊模式) | 移轉 | 並行要求數 |
|----- | ----- | ---- | ---- |
| S0 |  5   |   => 含 1 個縮放單位的 **S2** |   5 |
| S1 |  12  |   => 含 3 個縮放單位的 **S2** |  15 |

若要移轉成新的層，請執行下列操作：

## <a name="step-1-check-your-existing-deployment"></a>步驟 1：檢查您現有的部署
前往[自訂語音服務入口網站](http://cris.ai)，然後檢查您現有的部署。 在我們的範例中，有兩個部署。 一個部署在 S0 層上執行，另一個部署在 S1 層上執行。 這些部署顯示在下表的 [部署選項] 資料行中：

![[部署] 頁面](../../../media/cognitive-services/custom-speech-service/custom-speech-deployments.png)

## <a name="step-2-select-your-new-pricing-tier-in-the-azure-portal"></a>步驟 2：在 Azure 入口網站中選取新的定價層
1. 開啟新的瀏覽器索引標籤，然後登入 [Azure 入口網站](http://ms.portal.azure.com/)。 

2. 在 [認知服務] 窗格的 [訂用帳戶] 清單中，選取您的自訂語音訂用帳戶。 

3. 在您訂用帳戶的窗格中，選取 [定價層]。

    ![[定價層] 連結](../../../media/cognitive-services/custom-speech-service/custom-speech-update-tier.png)

4. 在 [選擇您的定價層] 頁面上，選取 [S2 標準]。 此定價層是新的、已簡化且更具彈性的定價層。

5. 選取 [選取] 按鈕。

    ![[選擇您的定價層] 頁面](../../../media/cognitive-services/custom-speech-service/custom-speech-update-pricing.png)

## <a name="step-3-check-the-migration-status-in-the-custom-speech-service-portal"></a>步驟 3：在自訂語音服務入口網站中檢查移轉狀態
返回「自訂語音服務」入口網站，然後檢查您的部署。 (如果您的瀏覽器視窗仍保持開啟，請重新整理視窗)。 

相關部署的狀態應該會切換成「處理中」。 您還應該檢查 [部署選項] 資料行來驗證移轉。 您現在可以從該處找到有關縮放單位和記錄的資訊。 縮放單位應該會反映您先前的定價層。 記錄功能應該也已開啟，如下表所示：

![[部署選項] 資料行](../../../media/cognitive-services/custom-speech-service/custom-speech-deployments-new.png)


> [!NOTE]
> 如果您在移轉期間遇到任何問題，請與我們連絡。
>

## <a name="next-steps"></a>後續步驟
如需更多教學課程，請參閱：
* [建立自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)
* [建立自訂語言模型](cognitive-services-custom-speech-create-language-model.md)
* [建立自訂的語音轉換文字端點](cognitive-services-custom-speech-create-endpoint.md)
