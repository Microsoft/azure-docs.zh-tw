---
title: 變更 Azure 訂用帳戶供應項目 | Microsoft Docs
description: 了解如何使用 Azure 帳戶中心來變更 Azure 訂閱以及改用其他供應項目
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing,top-support-issue
ms.assetid: aae227b3-6d64-4550-a5b6-d359f53f0a59
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/11/2018
ms.author: cwatson
ms.openlocfilehash: 9cb415a84a61212b8c8e3840bdc85429b5d5308c
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47391869"
---
# <a name="change-your-azure-pay-as-you-go-subscription-to-a-different-offer"></a>將您的 Azure 預付型方案訂閱變更為其他供應項目

做為[隨用隨付](https://azure.microsoft.com/offers/ms-azr-0003p/)客戶，您可以在[帳戶中心](https://account.windowsazure.com/Subscriptions)中將 Azure 訂用帳戶切換到其他供應項目。 例如，您可以使用這項功能，利用 [Visual Studio 訂閱者的每月信用額度](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)。 

**只想要從免費試用版升級？** 請參閱[升級為預付型方案](billing-upgrade-azure-subscription.md)。

## <a name="whats-supported"></a>支援的項目：

| 從 | 至 |
| --- | --- |
| Pay-As-You-Go |[隨用隨付開發/測試](https://azure.microsoft.com/offers/ms-azr-0023p/) |
| Pay-As-You-Go |[Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/) |
| Pay-As-You-Go |[Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/) |
| Pay-As-You-Go |[MSDN 平台](https://azure.microsoft.com/offers/ms-azr-0062p/) |
| Pay-As-You-Go |[Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) |
| Pay-As-You-Go |[Visual Studio Enterprise (Bizspark)](https://azure.microsoft.com/offers/ms-azr-0064p/) |

> [!NOTE]
> 如需其他供應項目變更，請[連絡支援中心](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
>
>

## <a name="switch-subscription-offer"></a>切換訂用帳戶供應項目

> [!VIDEO https://channel9.msdn.com/Series/Microsoft-Azure-Tutorials/Switch-to-a-different-Azure-offer/player]
>
>

1. 登入 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 選取隨用隨付訂用帳戶。
1. 按一下 [切換到其他供應項目] 。 如果您使用隨用隨付方案並已支付第一期帳單，才能使用此按鈕。

   ![請注意頁面右側的 [切換供應項目] 按鈕](./media/billing-how-to-switch-azure-offer/switchbutton.png)
1. 從您的訂用帳戶可切換的供應項目清單中**選取所需的供應項目**。 這份清單會根據您帳戶相關聯的會員資格而有所不同。 如果沒有清單，請檢查[您可切換的可用供應項目清單](#whats-supported)，並確定您有正確的會員資格。 

   ![選取您要切換的目標供應項目](./media/billing-how-to-switch-azure-offer/selectoffer.png)
1. 根據您所切換的供應項目，您可能會看到有關切換所造成之影響的附註。 在您繼續進行之前，請先仔細瀏覽這份清單並依照指示操作。

   ![檢閱注意事項](./media/billing-how-to-switch-azure-offer/thingstonote.png)
1. 您可以重新命名訂用帳戶。 根據預設，我們會將它設定為新的供應項目名稱。 按一下 [切換供應項目]  以完成程序。

   ![按一下綠色按鈕](./media/billing-how-to-switch-azure-offer/confirmpage.png)
1. 成功！ 您的訂用帳戶現在已經切換到新的供應項目。

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="what-is-an-azure-offer"></a>什麼是 Azure 供應項目？

Azure 供應項目是您擁有之 Azure 訂用帳戶的「類型」。 例如，[隨用隨付](https://azure.microsoft.com/offers/ms-azr-0003p/)、[Azure in Open](https://azure.microsoft.com/offers/ms-azr-0111p/) 和 [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) 皆為 Azure 供應項目。 每個供應項目有不同的[條款](https://azure.microsoft.com/support/legal/offer-details/)，有些則有特殊優點。 您可以在帳戶中心的 [訂用帳戶] 頁面中找到訂用帳戶的供應項目。 按一下供應項目名稱以取得更多詳細資料。

   ![按一下帳戶中心的 [供應項目] 連結以取得更多詳細資料](./media/billing-how-to-switch-azure-offer/offerlink.png)

### <a name="why-dont-i-see-the-button"></a>為什麼看不到該按鈕？

在下列情況下，您可能看不到 [切換到其他供應項目] 按鈕：

* 您目前使用的是[隨用隨付](https://azure.microsoft.com/offers/ms-azr-0003p/)。 目前只有預付型方案訂閱才能轉換成其他供應項目。
  * 如果您目前使用[免費試用版](https://azure.microsoft.com/free/)，請了解如何[升級至隨用隨付](billing-upgrade-azure-subscription.md)。
  * 若要從不同的訂用帳戶切換供應項目，[請聯絡支援中心](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
* 您仍然在第一個計費週期中；必須等到第一個計費週期完成後，才能切換供應項目。

### <a name="why-do-i-see-there-are-no-offers-available-in-your-region-or-country-at-this-time"></a>為什麼我會看到「目前您的國家/地區沒有提供此供應項目」錯誤訊息？

* 您可能不符合任何改用供應項目的資格。 請查看[您可以切換的可用供應項目清單](#whats-supported)，並確定您已透過 Visual Studio 或 Bizspark 啟動適當的權益。
* 部分供應項目可能無法在所有國家/地區中使用。

### <a name="what-does-switching-azure-offers-do-to-my-service-and-billing"></a>切換 Azure 供應項目對我的服務和帳單有什麼好處？

這裡說明當您在「帳戶中心」中切換 Azure 供應項目時的詳細資料。

#### <a name="no-service-downtime"></a>沒有服務停機時間

與訂用帳戶相關聯的任何使用者均可如常使用服務。 不過，您所切換的供應項目可能會有所限制。 例如，部分供應項目禁止生產使用，因此您需要將生產資源移至其他訂用帳戶。

#### <a name="quota-increases-are-reset"></a>會重設配額增加

當您切換供應項目時，會重設任何[超過預設限制的限制或配額增加](../azure-supportability/resource-manager-core-quotas-request.md)。 沒有任何服務停機時間，即使您的資源超過預設的限制亦然。 例如，您在訂用帳戶上使用 200 個核心，然後切換供應項目會將核心配額重設回預設的 20 個核心。 使用 200 個核心的 VM 不會受到影響，而且會繼續執行。 不過，如果您不提出另一個配額增加的要求，就無法佈建更多的核心。

#### <a name="billing"></a>計費

在您切換當天，系統會為所有未付費用產生發票。 接下來，您的訂用帳戶將以新的供應項目價格條款計費。 您的訂用帳戶計費週年會變更為您變更供應項目的日期。 供應項目變更之前的使用量與計費資料將不會保留，因此我們建議您在切換之前先行下載這些資料。

### <a name="can-i-migrate-from-pay-as-you-go-to-cloud-solution-providerhttpspartnermicrosoftcomsolutionscloud-reseller-overview-csp-or-enterprise-agreementhttpsazuremicrosoftcompricingenterprise-agreement-ea"></a>我可以從隨用隨付移轉為[雲端解決方案提供者](https://partner.microsoft.com/Solutions/cloud-reseller-overview) (CSP) 或 [Enterprise 合約](https://azure.microsoft.com/pricing/enterprise-agreement/) (EA) 嗎？

* 若要移轉為 CSP，請參閱 [Azure 預付型方案訂用帳戶移轉至 CSP](https://docs.microsoft.com/azure/cloud-solution-provider/migration/migration-from-payg-to-csp)。
* 若要移轉至 EA，可請您的註冊系統管理員將您的帳戶新增至 EA。 請遵循邀請電子郵件中的指示，將訂用帳戶移至 EA 註冊底下。 若要深入了解，請參閱 EA 入口網站中的[與現有帳戶相關聯](https://ea.azure.com/helpdocs/associateExistingAccount)。

### <a name="can-i-migrate-data-and-services-to-a-new-subscription"></a>是否可以將資料與服務移轉至新的訂用帳戶？

* 您可以直接將資源移轉至新的訂用帳戶，請參閱[將資源移動到新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。
* 若要將 Azure 訂用帳戶的擁有權和其中的所有內容轉送給其他人，請參閱[轉送 Azure 訂用帳戶的擁有權](billing-subscription-transfer.md)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
