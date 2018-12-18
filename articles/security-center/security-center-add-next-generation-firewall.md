---
title: 在 Azure 資訊安全中心新增新一代防火牆 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「新增新一代防火牆」與「僅透過 NGFW 路由傳送流量」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 48b99015-4db8-4ce8-85e4-b544c0fa203e
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2017
ms.author: terrylan
ms.openlocfilehash: 25601b01489b95de0e86b314b4b934d3cffd38c2
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "36330735"
---
# <a name="add-a-next-generation-firewall-in-azure-security-center"></a>在 Azure 資訊安全中心新增新一代防火牆
Azure 資訊安全中心可能會建議您新增由 Microsoft 合作夥伴提供的新一代防火牆 (NGFW)，以提升您的安全防護。 本文件逐步解說如何進行這項操作的範例。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  這不是逐步指南。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取 [新增新一代防火牆]。
   ![][1]
2. 在 [新增新一代防火牆]  刀鋒視窗中，選取一個端點。
   ![選取端點][2]
3. 第二個 [新增新一代防火牆]  刀鋒視窗隨即開啟。 您可以選擇使用現有的解決方案 (如果有的話)，或是建立新的解決方案。 此範例中沒有任何可用的現有解決方案，因此我們將建立一個 NGFW。
   ![新建新一代防火牆][3]
4. 若要建立 NGFW，請從整合式合作夥伴的清單中選取一個解決方案。 在此範例中，我們會選取 [檢查點]。
   ![選取新一代防火牆解決方案][4]
5. [檢查點]  刀鋒視窗隨即開啟，為您提供合作夥伴解決方案的相關資訊。 選取資訊刀鋒視窗中的 [建立]  。
   ![防火牆資訊刀鋒視窗][5]
6. [建立虛擬機器]  刀鋒視窗就會開啟。 您可在此輸入啟動將執行 NGFW 的虛擬機器 (VM) 所需的資訊。 依照下列步驟執行，並提供所需的 NGFW 資訊。 選取 [確定] 以套用。
   ![建立虛擬機器以執行 NGFW][6]

## <a name="route-traffic-through-ngfw-only"></a>僅透過 NGFW 路由傳送流量
返回 [建議]  刀鋒視窗。 在您透過資訊安全中心加入 NGFW 後會產生新的項目，稱為「僅透過 NGFW 路由傳送流量」 。 只有當您透過資訊安全中心安裝了 NGFW，才會建立這項建議。 如果您有網際網路面向的端點，資訊安全中心會建議您設定網路安全性群組規則，強制透過 NGFW 將輸入流量傳送到 VM。

1. 在 [建議] 刀鋒視窗中，選取 [僅透過 NGFW 路由傳送流量]。
   ![僅透過 NGFW 路由傳送流量][7]
2. 這樣會開啟 [僅透過 NGFW 路由傳送流量] 刀鋒視窗，其中列出您可以路由傳送流量的 VM。 從清單中選取 VM。
   ![選取 VM][8]
3. 所選 VM 的刀鋒視窗隨即開啟，其中顯示相關的輸入規則。 說明提供後續可能步驟的詳細資訊。 選取 [編輯輸入規則]  繼續編輯輸入規則。 預期的情況是與 NGFW 連結的網際網路面向端點其 [來源] 不設定為 [任何]。 若要深入了解輸入規則的內容，請參閱[安全性規則](../virtual-network/security-overview.md#security-rules)。
   ![設定規則以限制存取][9]
   ![編輯輸入規則][10]

## <a name="see-also"></a>另請參閱
本文件說明如何實作資訊安全中心建議的「新增新一代防火牆」。 若要了解 NGFW 與檢查點合作夥伴解決方案的詳細資訊，請參閱：

* [新一代防火牆](https://en.wikipedia.org/wiki/Next-Generation_Firewall)
* [檢查點 vSEC](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/checkpoint.vsec)

如要深入了解資訊安全中心，請參閱下列主題：

* [設定 Azure 資訊安全中心的安全性原則](security-center-policies.md) -- 了解如何設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-add-next-gen-firewall/add-next-gen-firewall.png
[2]: ./media/security-center-add-next-gen-firewall/select-an-endpoint.png
[3]: ./media/security-center-add-next-gen-firewall/create-new-next-gen-firewall.png
[4]: ./media/security-center-add-next-gen-firewall/select-next-gen-firewall.png
[5]: ./media/security-center-add-next-gen-firewall/firewall-solution-info-blade.png
[6]: ./media/security-center-add-next-gen-firewall/create-virtual-machine.png
[7]: ./media/security-center-add-next-gen-firewall/route-traffic-through-ngfw.png
[8]: ./media/security-center-add-next-gen-firewall/select-vm.png
[9]: ./media/security-center-add-next-gen-firewall/configure-rules-to-limit-access.png
[10]: ./media/security-center-add-next-gen-firewall/edit-inbound-rule.png
