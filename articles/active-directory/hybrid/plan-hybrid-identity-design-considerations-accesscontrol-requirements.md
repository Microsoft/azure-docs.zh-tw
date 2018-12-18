---
title: 混合式身分識別設計存取控制需求 Azure | Microsoft Docs
description: 說明身分識別的要件，以及針對混合式環境中的使用者識別資源的存取需求。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: e3b3b984-0d15-4654-93be-a396324b9f5e
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 8a40acda189507c1589373a48166ed1668e30650
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308327"
---
# <a name="determine-access-control-requirements-for-your-hybrid-identity-solution"></a>判斷混合式身分識別解決方案的存取控制需求
當組織正在設計他們的混合式身分識別解決方案時，也可以利用這個機會，針對他們正規劃來讓使用者使用的資源檢閱存取需求。 您可以橫跨下列這四種身分識別要件來存取資料：

* 系統管理
* 驗證
* Authorization
* 稽核

後續小節將更詳細說明驗證與授權，系統管理和稽核則是混合式身分識別生命週期的一部分。 如需這些功能的詳細資訊，請參閱 [判斷混合式身分識別管理工作](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md) 。

> [!NOTE]
> 如需每一個要件的詳細資訊，請參閱 [身分識別的四個要件 - 混合式 IT 使用期間的身分識別管理](http://social.technet.microsoft.com/wiki/contents/articles/15530.the-four-pillars-of-identity-identity-management-in-the-age-of-hybrid-it.aspx) 。
> 
> 

## <a name="authentication-and-authorization"></a>驗證和授權
有各種適用於驗證和授權的不同案例，這些案例將具備特殊需求，必須透過公司即將採用的混合式身分識別解決方案來滿足。 涉及企業對企業 (B2B) 通訊的案例會對 IT 系統管理員造成額外的挑戰，因為他們需要確定組織所使用的驗證和授權方法可以和他們的商務合作夥伴進行通訊。 在設計驗證和授權需求的過程中，請確定會回答下列問題：

* 您的組織只會驗證和授權位於其身分識別管理系統中的使用者嗎？
  * 是否有任何適用於 B2B 案例的方案？
  * 如果有，您已經知道將使用哪些通訊協定 (SAML、OAuth、Kerberos、權杖或憑證) 來連線這兩個企業嗎？
* 您即將採用的混合式身分識別解決方案支援這些通訊協定嗎？

另一個要考量的重點是使用者和合作夥伴將使用的驗證儲存機制將位於何處以及要使用哪個系統管理模型。 請考量下列兩個核心選項：

* 集中式︰在這個模型中，使用者的認證、原則和管理可以集中於內部部署或雲端中。
* 混合式︰在這個模型中，使用者的認證、原則和管理會集中於內部部署和雲端複寫中。

組織將採用哪一個模型，將根據其商務需求而不同，您需要回答下列問題來識別身分識別管理系統所在的位置以及要使用的系統管理模式：

* 您的組織目前具有內部部署的身分識別管理嗎？
  * 如果是，他們打算如何保留它？
  * 您的組織是否有任何法規或規範需求必須遵循，以指出身分識別管理系統應位於的位置？
* 您的組織會針對位於內部部署或雲端中的 app 使用單一登入嗎？
  * 如果是，採用混合式身分識別模型會影響此程序嗎？

## <a name="access-control"></a>存取控制
儘管驗證和授權是核心元素，可透過使用者的驗證來啟用對公司資料的存取權，但是控制這些使用者將具備的存取層級，以及系統管理員在其管理的資源上具備的存取層級，也同樣重要。 您的混合式身分識別解決方案必須能夠對資源、委派和角色型存取控制提供更細微的存取權。 請確保您知道下列存取控制相關問題的答案：

* 貴公司擁有多位已提高權限來管理您身分識別系統的使用者嗎？
  * 如果是，每位使用者都需要同一個存取層級嗎？
* 貴公司需要將存取權委派給使用者來管理特定資源嗎？
  * 如果是，這種情況發生的頻率為何？
* 貴公司需要整合內部部署與雲端資源之間的存取控制功能嗎？
* 貴公司需要根據相同情況來限制資源的存取嗎？
* 貴公司是否有任何應用程式需要某些資源的自訂控制存取權？
  * 如果是，這些 app 位於何處 (內部部署或雲端中)？
  * 如果是，這些目標資源位於何處 (內部部署或雲端中)？

> [!NOTE]
> 請確定會記下每個答案，並了解答案背後的原理。 [定義資料保護策略](plan-hybrid-identity-design-considerations-data-protection-strategy.md) 將介紹可用選項，以及每個選項的優點/缺點。  回答這些問題，您就能選取最適合您業務需求的選項。
> 
> 

## <a name="next-steps"></a>後續步驟
[判斷事件因應需求](plan-hybrid-identity-design-considerations-incident-response-requirements.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

