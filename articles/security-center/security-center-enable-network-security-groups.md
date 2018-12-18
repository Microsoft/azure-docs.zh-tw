---
title: 在 Azure 資訊安全中心啟用網路安全性群組 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「啟用網路安全性群組」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: f53ed853-ffaf-4530-a019-1906ba6f341b
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2017
ms.author: terrylan
ms.openlocfilehash: fe75781629e8d6416405cc9eec9ce14e61a00b14
ms.sourcegitcommit: ea5193f0729e85e2ddb11bb6d4516958510fd14c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36301422"
---
# <a name="enable-network-security-groups-in-azure-security-center"></a>在 Azure 資訊安全中心啟用網路安全性群組
如果尚未啟用網路安全性群組 (NSG)，Azure 資訊安全中心會建議您啟用。 NSG 包含存取控制清單 (ACL) 規則的清單，可允許或拒絕虛擬網路中 VM 執行個體的網路流量。 NSG 可與子網路或該子網路內的個別 VM 執行個體相關聯。 當 NSG 與子網路相關聯時，ACL 規則便會套用至該子網路中的所有 VM 執行個體。 此外，將 NSG 直接關聯至該 VM，即可進一步限制個別 VM 的流量。 若要深入了解，請參閱 [什麼是網路安全性群組 (NSG)？](../virtual-network/security-overview.md)

如果您尚未啟用 NSG，資訊安全中心會為您呈現兩個建議︰在子網路上啟用網路安全性群組和在虛擬機器上啟用網路安全性群組。 您選擇哪個層級、子網路或 VM 要套用 NSG。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  這不是逐步指南。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取在子網路或虛擬機器上 [啟用網路安全性群組]。
   ![啟用網路安全性群組][1]
2. 這會開啟針對子網路或虛擬機器 [設定遺漏的網路安全性群組]  刀鋒視窗，取決於您選取的建議。 選取要設定 NSG 的子網路或虛擬機器。

   ![針對子網路設定 NSG][2]

   ![針對 VM 設定 NSG][3]
3. 在 [選擇網路安全性群組] 刀鋒視窗中，選取現有的 NSG 或選取 [建立新的] 以建立一個 NSG。

   ![選擇網路安全性群組][4]

如果您建立一個 NSG，請依照[管理網路安全性群組](../virtual-network/manage-network-security-group.md)中的步驟，建立 NSG 並設定安全性規則。

## <a name="see-also"></a>另請參閱
本文說明了如何實作資訊安全中心建議的針對子網路或虛擬機器「啟用網路安全性群組」。 若要深入了解啟用 NSG，請參閱下列項目：

* [什麼是網路安全性群組 (NSG)？](../virtual-network/security-overview.md)
* [管理網路安全性群組](../virtual-network/manage-network-security-group.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-enable-nsg/enable-nsg.png
[2]:./media/security-center-enable-nsg/configure-nsg-for-subnet.png
[3]: ./media/security-center-enable-nsg/configure-nsg-for-vm.png
[4]: ./media/security-center-enable-nsg/choose-nsg.png
