---
title: Azure Active Directory B2C 中的區域可用性和資料落地 | Microsoft Docs
description: 有關 Azure Active Directory B2C 租用戶類型的主題。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/10/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 7c0348dad507b118af542c1bbc47f6ceb88ae7be
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443047"
---
# <a name="azure-active-directory-b2c-region-availability--data-residency"></a>Azure Active Directory B2C：區域可用性和資料存留處
區域可用性和資料存留處是兩個非常不同的概念，這些概念套用到 Azure AD B2C 的方式與套用到其餘 Azure 的方式不同。 本文將說明這兩個概念之間的差異，並比較它們如何套用在 Azure 與 Azure AD B2C。

## <a name="summary"></a>總結
Azure AD B2C 是**在全球正式運作**，具有**資料存留處在美國或歐洲**的選項。

## <a name="concepts"></a>概念
* 「區域可用性」係指一項服務是否可供使用。
* 「資料存留處」係指使用者資料的儲存位置。

## <a name="region-availability"></a>區域可用性
Azure AD B2C 是透過 Azure 公用雲端在全球提供使用。 

這與大多數其他 Azure 服務所依循的模型不同，該模型是將可用性與資料存留處結合。 您可以在 Azure 的[依區域提供的產品](https://azure.microsoft.com/regions/services/)頁面和 [Azure Active Directory B2C 定價計算機](https://azure.microsoft.com/pricing/details/active-directory-b2c/)中看到這個的相關範例。

## <a name="data-residency"></a>資料存留處
Azure AD B2C 會將使用者資料儲存在美國或歐洲。

資料存留處是根據[建立 Azure AD B2C 租用戶](active-directory-b2c-get-started.md)時所選取的國家/地區來決定。

![預覽租用戶的螢幕擷取畫面](./media/active-directory-b2c-reference-tenant-type/data-residency-b2c-tenant.png)

針對下列國家/地區，資料會位於美國：

> 美國、加拿大、哥斯大黎加、多明尼加共和國、薩爾瓦多、瓜地馬拉、墨西哥、巴拿馬、波多黎各和千里達及托巴哥

針對下列國家/地區，資料會位於歐洲：

> 阿爾及利亞、奧地利、亞塞拜然、巴林、白俄羅斯、比利時、保加利亞、克羅埃西亞、賽普勒斯、捷克共和國、丹麥、埃及、愛沙尼亞、芬蘭、法國、德國、希臘、匈牙利、冰島、愛爾蘭、以色列、義大利、約旦、哈薩克、肯亞、科威特、拉脫維亞、黎巴嫩、列支敦斯登、立陶宛、盧森堡、馬其頓 (FYRO)、馬爾他、蒙特內哥羅、摩洛哥、荷蘭、奈及利亞、挪威、阿曼、巴基斯坦、波蘭、葡萄牙、卡達、羅馬尼亞、俄羅斯、沙烏地阿拉伯、塞爾維亞、斯洛伐克、斯洛維尼亞、南非、西班牙、瑞典、瑞士、突尼西亞、土耳其、烏克蘭、阿拉伯聯合大公國和英國。

其餘國家/地區正在新增到清單。  目前，您仍可透過挑選上述任何國家/地區來使用 Azure AD B2C。

> 阿富汗、阿根廷、澳洲、巴西、智利、哥倫比亞、厄瓜多、香港特別行政區、印度、印尼、伊拉克、日本、南韓、馬來西亞、紐西蘭、巴拉圭、祕魯、菲律賓、新加坡、斯里蘭卡、台灣、泰國、烏拉圭和委內瑞拉。

## <a name="preview-tenant"></a>預覽租用戶
如果您已在 Azure AD B2C 的預覽期間建立 B2C 租用戶，您的 [租用戶類型] 可能會顯示為 [預覽租用戶]。 如果情況如此，您只能將租用戶用於開發和測試用途，而不能用於生產應用程式。

> [!IMPORTANT]
> 預覽 B2C 租用戶沒有任何途徑可以移轉到生產級別 B2C 租用戶。 請注意，當您刪除預覽 B2C 租用戶並使用相同的網域名稱重建生產級別 B2C 租用戶時，會發生已知的問題。 您必須使用不同的網域名稱建立生產級別 B2C 租用戶。


![預覽租用戶的螢幕擷取畫面](./media/active-directory-b2c-reference-tenant-type/preview-b2c-tenant.png)
