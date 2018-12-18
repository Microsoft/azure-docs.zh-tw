---
title: 升級 Azure IoT 中樞 | Microsoft Docs
description: 變更「IoT 中樞」的定價與級別層，以取得更多傳訊和裝置管理功能。
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/02/2018
ms.author: kgremban
ms.openlocfilehash: 1f60b7d30c073c49d5e0a7d35e7263c2181ed744
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37903059"
---
# <a name="how-to-upgrade-your-iot-hub"></a>如何升級 IoT 中樞

隨著您的 IoT 解決方案日漸成長，「Azure IoT 中樞」已準備好協助您擴大規模。 「Azure IoT 中樞」提供基本 (B) 和標準 (S) 兩個層級，可配合想要使用不同功能的使用者需求。 在每個層級內都有三個大小 (1、2 及 3)，用來決定每天可傳送的訊息數。 

當您有更多裝置且需要更多功能時，有三種方式可調整 IoT 中樞以符合您的需求：

* 在 IoT 中樞內增加單位。 例如，在 B1 IoT 中樞內每增加一個單位可允許每天額外 400,000 則訊息。 
* 變更 IoT 中樞的大小。 例如，從 B1 層移轉至 B2 層以增加每個單位每天可支援的訊息數量。
* 升級至較高層級。 例如，從 B1 層升級至 S1 層時，傳訊容量不變，但會具有標準層所包含的進階功能。

這些變更全都可以在不中斷現有作業的情況下進行。

如果您想要降級 IoT 中樞，可以移除單位並縮減 IoT 中樞大小。 不過，您無法降級至較低的層級。 例如，您可以從 S2 層移至 S1 層，但無法從 S2 層移至 B1 層。 

這些範例是用來協助您了解如何隨著解決方案的變更，以調整您的 IoT 中樞。 如需有關每個層級功能的特定資訊，您應該一律參考 [Azure IoT 中樞價格](https://azure.microsoft.com/pricing/details/iot-hub/)。 

## <a name="upgrade-your-existing-iot-hub"></a>升級現有的 IoT 中樞 

1. 登入 [Azure 入口網站](https://portal.azure.com/)，然後瀏覽至 IoT 中樞。 
2. 選取 [定價與級別]。 

   ![定價與級別](./media/iot-hub-upgrade/pricing-scale.png)

3. 若要變更中樞的層級，請選取 [定價與級別層]。 選取新的層級，然後按一下 [選取]。

   ![定價與級別](./media/iot-hub-upgrade/select-tier.png)

4. 若要變更您中樞內的單位數，請在 [IoT 中樞單位] 底下輸入新的值。 
5. 選取 [儲存] 來儲存變更。 

您的 IoT 中樞現已調整完成，而設定則未變更。 請注意，基本層 IoT 中樞的分割區限制為 8 個。 當您從基本層移轉到標準層時，這項限制保持不變。

## <a name="next-steps"></a>後續步驟

取得有關[如何選擇適合的 IoT 中樞層](iot-hub-scaling.md)的更多詳細資料。 

