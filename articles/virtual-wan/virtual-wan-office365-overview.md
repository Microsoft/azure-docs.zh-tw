---
title: Azure 虛擬 WAN 中的 Office 365 控制平面
description: 了解虛擬 WAN 中的 Office 365 控制平面。
author: cherylmc
ms.service: virtual-wan
services: virtual-wan
ms.topic: article
ms.date: 9/24/2018
ms.author: cherylmc
ms.openlocfilehash: 815b91295540e93f8f0ffbc002fcf02ce1c13365
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992669"
---
# <a name="office-365-control-plane-in-virtual-wan"></a>虛擬 WAN 中的 Office 365 控制平面

使用所選 SDWAN 裝置的虛擬 WAN 客戶可以為 Azure 入口網站設定受信任流量的 O365 網際網路分組原則。 這可讓：
- O365 流量進入接近使用者的 Microsoft 網路以提供最佳的使用者體驗。
- 避免流量回程和 Hair Pining，因此節省 WAN 成本。
- 隨著 O365 連線原則一起提供。

## <a name="faqs"></a>常見問題集
### <a name="what-is-the-customer-benefit"></a>什麼是客戶權益？
在虛擬 WAN 中使用這項功能，客戶現在可以針對直接網際網路分組指定他們信任的 Office 365 流量類別。 這個受信任 O365 流量將會略過 Proxy，直接從使用者位置路由傳送到最接近的 Microsoft POP。 這可避免流量回程和 Hair Pining，因此提供最佳的使用者經驗和節省 WAN 成本。 

### <a name="what-are-the-office-365-traffic-categories"></a>Office 365 流量類別有哪些？
Office 365 端點代表網路位址和子網路。 端點可能是 URL、IP 位址或 IP 範圍。 URL 可以是 FQDN (如 *account.office.net*) 或萬用字元 URL (如 **.office365.com*)。 端點根據其重要性分成三個類別 -「最佳化」、「允許」和「預設」。 端點類別的更多詳細資訊在[這裡](https://docs.microsoft.com/office365/enterprise/office-365-network-connectivity-principles#BKMK_Categories)。

### <a name="which-office-365-traffic-category-is-recommended-by-microsoft-for-direct-internet-breakout"></a>對直接網際網路分組，Microsoft 推薦使用哪個 Office 365 流量類別？
「最佳化」類別是最重要的網路端點，它需要略過 SSL 中斷並檢查其他網路安全性裝置。 它應該具有靠近使用者的直接網際網路輸出。 這些端點代表對網路效能、延遲和可用性最為敏感的 Office 365 案例。 這個類別包含一小組索引碼 URL (按照 ~ 10 的順序) 以及一組專供核心 Office 365 工作負載使用的已定義 IP 子網路，這些工作負載包括 Exchange Online、SharePoint Online、商務用 Skype Online 和 Microsoft Teams。 

「允許」類別也建議供直接網際網路輸出使用。 不過「允許」網路流量可以容忍某些網路延遲。 「最佳化」和「允許」類別中的端點都裝載在 Microsoft 資料中心，而且當做 Office 365 的一部分加以管理。 「預設」類別可以導向到預設網際網路輸出位置，而且它不需要直接網際網路輸出或略過 SSL，也不需要檢查裝置。

### <a name="how-do-i-set-my-o365-policies-via-virtual-wan"></a>如何透過虛擬 WAN 設定 O365 原則？
您可以透過 [虛擬 WAN] -> [設定] -> [組態] 索引標籤來啟用原則。您可以在這裡為直接網際網路分組指定慣用的 O365 流量類別。

![設定虛擬 WAN 中的 Office 365 控制平面](media/virtual-wan-office365-overview/configure-office365-control-plane.png)

### <a name="how-does-this-work"></a>運作方式為何？

1.  O365 流量進入接近使用者的 Microsoft 網路以提供最佳的體驗。
2.  路由原則由 SDWAN 使用。 它接著會為受信任類別略過安全性 Proxy，並為這些類別執行本機直接分組。
3.  避免流量回程和 Hair Pining 以節省 WAN 成本。

### <a name="which-partner-devices-support-this-via-virtual-wan"></a>哪些夥伴裝置透過虛擬 WAN 支援此功能？
目前而言，Citrix 服務透過虛擬 WAN 支援這些原則。

### <a name="what-happens-to-the-remaining-categories-of-untrusted-o365-traffic"></a>其餘 (未信任的) O365 流量類別會發生什麼事？
剩餘的 O365 流量會遵循客戶預設的網際網路流量路徑。

### <a name="what-if-i-have-already-specified-my-o365-policies-via-my-sdwan-provider"></a>如果我已經透過 SDWAN 提供者指定 O365 原則呢？
如果您透過 SDWAN UX 和 Azure 虛擬 WAN 指定原則原則，則虛擬 WAN 中設定的原則有較高的優先順序。

## <a name="next-steps"></a>後續步驟
- 深入了解[虛擬 WAN](virtual-wan-about.md)。