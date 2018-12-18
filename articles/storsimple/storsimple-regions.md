---
title: StorSimple 區域可用性 | Microsoft Docs
description: 說明提供各種 StorSimple 裝置型號的 Azure 區域。
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/16/2017
ms.author: alkohli
ms.openlocfilehash: d47109d541a3df93d9234e27e53d1538f6bc4c6e
ms.sourcegitcommit: c7215d71e1cdeab731dd923a9b6b6643cee6eb04
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/17/2017
ms.locfileid: "24879053"
---
# <a name="available-regions-for-your-storsimple"></a>您的 StorSimple 的可用區域

## <a name="overview"></a>概觀

Azure 資料中心在全球多個地區運行，以滿足客戶對資料位置相關之效能、需求和喜好設定的要求。 Azure 地理位置是包含至少一個 Azure 區域的已定義世界區域。 Azure 區域是包含一或多個資料中心之地理位置內的區域。

選擇 Azure 區域非常重要，而區域的選擇會受到資料落地與主權、服務可用性、效能、成本和備援等因素影響。 如需有關如何選擇區域的詳細資訊，請移至[哪個 Azure 區域適合我？](https://azure.microsoft.com/overview/datacenters/how-to-choose/)

針對 StorSimple 解決方案，區域選擇特別取決於下列因素：

- 可使用 StorSimple 裝置管理員服務的區域。
- 可使用 StorSimple 實體、雲端或虛擬裝置的國家/地區。
- 儲存 StorSimple 資料之儲存體帳戶為了取得最佳效能所在之區域。

本教學課程說明 StorSimple 裝置管理員服務、內部部署實體和雲端裝置的區域可用性。 本文所含資訊僅適用於 StorSimple 8000 和 1200 系列裝置。

## <a name="region-availability-for-storsimple-device-manager-service"></a>StorSimple 裝置管理員服務的區域可用性

目前在 12 個公用區域與 2 個 Azure Government 區域中支援 StorSimple 裝置管理員服務。

當您首次建立 StorSimple 裝置管理員服務時，會定義區域或位置。 一般來說，會選擇最接近裝置部署所在之地理區域的位置。 但是，裝置和服務也可以部署在不同的位置。

以下是 StorSimple 裝置管理員服務適用於 Azure 公用雲端，而且可以部署之區域的清單。

![storsimple-device-manager-service-regions](./media/storsimple-region/storsimple-device-manager-service-regions.png)

針對 Azure Government 雲端，StorSimple 裝置管理員服務在美國愛荷華州政府和美國維吉尼亞州政府資料中心內提供。

## <a name="region-availability-for-data-stored-in-storsimple"></a>儲存在 StorSimple 中之資料的區域可用性

StorSimple 資料實際儲存在 Azure 儲存體帳戶中，且這些帳戶在所有 Azure 區域都可使用。 當您建立 Azure 儲存體帳戶時，會選擇儲存體帳戶的主要位置，而這會決定資料所在的區域。

當您第一次建立 StorSimple 裝置管理員服務，並將儲存體帳戶與它關聯時，您的 StorSimple 裝置管理員服務和 Azure 儲存體可位於兩個不同位置。 如此一來，您需要分別建立 StorSimple 裝置管理員和 Azure 儲存體帳戶。

一般來說，您應該為您的儲存體帳戶選擇最近的服務區域。 不過，最接近的 Microsoft Azure 區域可能實際上不是具有最低延遲的區域。 延遲決定了網路服務效能，因此也決定了解決方案的效能。 因此，如果您選擇不同區域中的儲存體帳戶，了解您的服務和與您的儲存體帳戶相關聯的區域之間的延遲是非常重要的。

如果您是使用 StorSimple 雲端設備，則建議服務和相關聯的儲存體帳戶位於相同的區域。 若儲存體帳戶位於不同區域，可能導致效能不佳。

## <a name="availability-of-storsimple-device"></a>StorSimple 裝置的可用性

依型號而定，StorSimple 裝置可以在不同的地理位置或國家/地區使用。

### <a name="storsimple-physical-device-models-81008600"></a>StorSimple 實體裝置 (型號 8100/8600)

如果使用的是 StorSimple 8100 或 8600 實體裝置，則該裝置可以在下列國家/地區使用。

| #  | 國家 (地區)        | #  | 國家 (地區)     | #  | 國家 (地區)      | #  | 國家 (地區)              |
|----|----------------|----|-------------|----|--------------|----|----------------------|
| 1  | 澳大利亞      | 16 | 香港   | 31 | 紐西蘭  | 46 | 南非         |
| 2  | 奧地利        | 17 | 匈牙利     | 32 | 奈及利亞      | 47 | 南韓          |
| 3  | 巴林        | 18 | 冰島     | 33 | 挪威       | 48 | 西班牙                |
| 4  | 比利時        | 19 | 印度       | 34 | 祕魯         | 49 | 斯里蘭卡            |
| 5  | 巴西         | 20 | 印尼   | 35 | 菲律賓  | 50 | 瑞典               |
| 6  | 加拿大         | 21 | 愛爾蘭     | 36 | 波蘭       | 51 | 瑞士          |
| 7  | 智利          | 22 | 以色列      | 37 | 葡萄牙     | 52 | 台灣               |
| 8  | 哥倫比亞       | 23 | 義大利       | 38 | 波多黎各  | 53 | 泰國             |
| 9  | 捷克共和國 | 24 | 日本       | 39 | 卡達        | 54 | 土耳其               |
| 10 | 丹麥        | 25 | 肯亞       | 40 | 羅馬尼亞      | 55 | 烏克蘭              |
| 11 | 埃及          | 26 | 科威特      | 41 | 俄羅斯       | 56 | 阿拉伯聯合大公國 |
| 12 | 芬蘭        | 27 | 澳門       | 42 | 沙烏地阿拉伯 | 57 | 英國       |
| 13 | 法國         | 28 | 馬來西亞    | 43 | 新加坡    | 58 | 美國        |
| 14 | 德國        | 29 | 墨西哥      | 44 | 斯洛伐克     | 59 | 越南              |
| 15 | 希臘         | 30 | 荷蘭 | 45 | 斯洛維尼亞     | 60 | 克羅埃西亞              |

加入更多國家/地區時，此清單會變更。 如需最新的地理位置清單，請移至[產品條款](https://www.microsoft.com/en-us/Licensing/product-licensing)中的＜儲存體陣列條款附錄＞。

Microsoft 可以在前述清單中的地理位置寄送實體硬體，並提供 StorSimple 的硬體備用零件更換。

> [!IMPORTANT]
> 請勿將 StorSimple 實體裝置放在不支援 StorSimple 的區域。 Microsoft 將無法將任何更換零件出貨到不支援 StorSimple 的國家/地區。

### <a name="storsimple-cloud-appliance-models-80108020"></a>StorSimple 雲端設備 (型號 8010/8020)

如果使用的是 StorSimple 雲端設備 8010 或 8020，則該裝置在支援基礎 VM 的所有區域都受到支援並可使用。 8010 使用 _Standard_A3_ VM，所有 Azure 區域都支援該 VM。

8020 使用進階儲存體和 _Standard_DS3_ VM 建立雲端設備。 在支援進階儲存體和 _Standard_DS3_ Azure VM 的 Azure 區域中，8020 受到支援。 使用[這份清單](https://azure.microsoft.com/regions/services/)，查看您的區域是否提供「虛擬機器 > DS 系列」和「儲存體 > 磁碟儲存體」。

### <a name="storsimple-virtual-array-model-1200"></a>StorSimple Virtual Array (型號 1200)

如果使用的是 1200 系列 StorSimple Virtual Array，則所有 Azure 區域都支援虛擬磁碟映像。

## <a name="next-steps"></a>後續步驟

* 深入了解[各種 StorSimple 型號的定價](https://azure.microsoft.com/pricing/calculator/#storsimple2)。
* 深入了解 [管理 StorSimple 儲存體帳戶](storsimple-8000-manage-storage-accounts.md)。
* 深入了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。
