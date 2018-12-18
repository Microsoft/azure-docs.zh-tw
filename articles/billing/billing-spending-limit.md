---
title: 了解 Azure 消費限制 | Microsoft Docs
description: 說明 Azure 消費限制的運作方式及如何移除此限制
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/15/2018
ms.author: cwatson
ms.openlocfilehash: 614102d65407485d31963afa9185400938a7d95b
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423080"
---
# <a name="understand-azure-spending-limit-and-how-to-remove-it"></a>了解 Azure 消費限制及如何移除此限制

Azure 中消費限制的目的是避免您的花費超過點數額度。 根據預設，系統會針對所有註冊試用版或包含多月點數之供應項目的新客戶開啟消費限制。 消費限制是 $0。 此限制無法變更。 隨用隨付訂用帳戶和承諾方案之類的訂用帳戶類型無法使用消費限制。 請參閱[完整的 Azure 供應項目及消費限制可用性清單](https://azure.microsoft.com/support/legal/offer-details/)。

## <a name="what-happens-when-i-reach-the-spending-limit"></a>達到消費限制時會發生什麼情況？

若您用完訂用帳戶隨附的每月額度，則您部署的服務在該計費期間的剩餘時間內將被停用。 

舉例來說，若您花光訂用帳戶隨附的所有點數，您部署的雲端服務便會從生產環境中移除，且 Azure 虛擬機器會停止運作並解除配置。 您只能以唯讀方式存取儲存體帳戶和資料庫中的資料。

下一個計費期間開始時，若您的訂用帳戶供應項目包含多月點數，系統便會自動重新啟用您的訂用帳戶。 接著，您便可以重新部署「雲端服務」，並完整存取您的儲存體帳戶和資料庫。

您達到訂用帳戶的消費限制時，我們會傳送電子郵件通知給您。 登入[帳戶中心](https://account.windowsazure.com/Subscriptions)之後，您會看到所有訂用帳戶達到消費限制的通知。

如果您使用免費試用版且達到消費限制，可以[升級為預付型方案](billing-upgrade-azure-subscription.md)來移除消費限制並自動重新啟用訂用帳戶。

<a id="remove"></a>

## <a name="remove-the-spending-limit-in-account-center"></a>在帳戶中心裡移除消費限制

只要您的訂用帳戶所關聯的付款方式有效，您就可以隨時移除消費限制。 針對有多月點數的供應項目，您也可以在下一個計費期間開始時重新啟用消費限制。

若要移除您的消費限制，請依照下列步驟操作：

1. 登入[帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 選取一個訂用帳戶。 如果訂用帳戶因為達到消費限制而遭到停用，請按一下此通知：[訂用帳戶已達到消費限制，為防止產生費用已被停用。] 否則，請按一下 [訂用帳戶狀態] 區域中的 [移除消費限制]。
1. 選取適合您的選項。

![選取用於移除消費限制的選項](./media/billing-spending-limit/remove-spending-limit.PNG)

|選項|效果|
|-------|-----|
|無限期移除消費限制|移除消費限制，不在下一個計費週期開始時自動開啟。|
|移除目前計費週期的消費限制|移除消費限制，讓它在下一個計費週期開始時自動開啟。|

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="why-would-i-want-to-remove-the-spending-limit"></a>為什麼我可能會想要移除消費限制？

消費限制會使您無法部署或使用某些協力廠商和 Microsoft 的服務。 以下是您應該移除訂用帳戶消費限制的情況。

* 您打算部署第一方映像 (例如 Oracle) 和服務 (例如 Azure DevOps Services)。 此情況會讓您幾乎立即超過消費限制，而導致訂用帳戶遭停用。
* 您有不可中斷的服務。
* 您的服務及資源有您不想遺失的設定 (例如虛擬 IP 位址)。 這些設定在服務和資源解除配置時遺失。

### <a name="how-do-i-turn-on-the-spending-limit-after-removing-it"></a>如何在移除消費限制之後再將其開啟？

唯有無限期移除消費限制時，才能使用此功能。 請將它變更為在下一個計費週期開始時自動開啟。

1. 登入[帳戶中心](https://account.windowsazure.com/Subscriptions)。
1. 按一下黃色橫幅以變更消費限制選項。
1. 選擇 [在下一個計費週期 \<計費週期開始日期\> 啟用消費限制]

### <a name="how-do-i-set-a-custom-spending-limit"></a>如何設定自訂消費限制？

無法使用自訂消費限制。

### <a name="does-the-spending-limit-prevent-all-charges-from-azure"></a>消費限制會阻擋 Azure 中產生的所有費用嗎？

[某些在 Azure Marketplace 中發佈的外部服務](billing-understand-your-azure-marketplace-charges.md)無法透過訂用帳戶點數來使用，並會產生另外的費用，即使您已設定消費限制亦然。 範例包括 Visual Studio 授權、Azure Active Directory Premium、支援方案，以及大部分的第三方品牌服務。 當您佈建新的外部服務時，系統會顯示警告，讓您知道服務會另外收費︰

![Marketplace 購買警告](./media/billing-understand-your-azure-marketplace-charges/marketplace-warning.PNG)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
