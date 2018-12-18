---
title: 混合式雲端身分識別設計的身分識別需求 Azure | Microsoft Docs
description: 識別公司的商務需求，以引導您定義混合式身分識別設計的需求。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: de690978-84ef-41ad-9dfe-785722d343a1
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/27/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 756d55fe535d424bf796e24968cfdc9c82721c1c
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46307037"
---
# <a name="determine-identity-requirements-for-your-hybrid-identity-solution"></a>判斷混合式身分識別解決方案的身分識別需求
設計混合式身分識別解決方案的第一個步驟，是判斷將運用此解決方案的企業組織有何需求。  混合式身分識別一開始是支援的角色 (它可提供驗證而支援所有其他雲端解決方案)，接著會提供新奇有趣的功能，為使用者釋出新的工作負載。  您想要為使用者採用的這些工作負載或服務，會指定混合式身分識別設計的需求。  這些服務和工作負載在內部部署和雲端中都需要運用混合式身分識別。  

我們必須通盤審視企業的各個主要層面，以了解其目前的需求，以及公司對未來的規劃。 如果您不清楚混合式身分識別設計的長期策略，您的解決方案在未來有可能無法隨著企業的成長和變化而進行調整。   下圖中的範例說明混合式身分識別架構和要為使用者釋出的工作負載。 此範例只是為了說明所有可透過健全的整合式身分識別策略釋出及提供的新可能性。 

屬於混合式身分識別架構的部分元件 ![](./media/plan-hybrid-identity-design-considerations/hybrid-identity-architechture.png)

## <a name="determine-business-needs"></a>判斷商務需求
每家公司都會有不同的需求，即使這些公司屬於相同產業，實際的商務需求仍可能有所不同。 您仍然可以採用業界的最佳作法，但最終引導您定義混合式身分識別設計需求的，仍將是公司的商務需求。 

請確實回答下列問題以識別您的商務需求：

* 您的公司想要減少 IT 營運成本嗎？
* 您的公司想要保護雲端資產 (SaaS 應用程式、基礎結構) 嗎？
* 您的公司想要將 IT 現代化嗎？
  * 您的使用者是行動性和要求較高的 IT 人員，因而期望您的 DMZ 允許以不同類型的流量存取不同的資源嗎？
  * 您的公司是否有需要發佈給這些現代使用者、但並不容易重寫的舊型應用程式？
  * 您的公司是否需要在能夠全盤掌控的情況下完成這些工作？
* 您的公司是否想要在內部部署中導入新工具，以運用 Microsoft Azure 安全性的專業功能，保護使用者的身分識別及降低風險？
* 您的公司是否嘗試要在內部部署中擺脫可怕的「外部」帳戶，並將其移至雲端，使其不再是內部部署環境內的潛在威脅？

## <a name="analyze-on-premises-identity-infrastructure"></a>分析內部部署身分識別基礎結構
現在您已對公司的商務需求有所了解，您必須評估內部部署身分識別基礎結構。 在定義將目前的身分識別解決方案整合到雲端身分識別管理系統的技術需求時，這項評估是很重要的。 請確實回答下列問題：

* 您的公司在內部部署中使用何種驗證和授權解決方案？ 
* 您的公司目前有任何內部部署同步處理服務嗎？
* 您的貴公司是否使用任何協力廠商身分識別提供者 (IdP)？

您也需要知道公司可能有的雲端服務。 執行評估以了解您的環境中目前與 SaaS、IaaS 或 PaaS 模型的整合，是非常重要的。 請務必在這個評估期間回答下列問題︰

* 貴公司是否與任何雲端服務提供者整合？
* 如果是，請問使用了哪些服務？
* 這項整合目前已投入使用，或只是一項試驗？

> [!NOTE]
> Cloud Discovery 會針對超過 16,000 個雲端應用程式的 Microsoft Cloud App Security 雲端應用程式目錄 (根據 70 個以上的風險因素排名及評分)，分析您的流量記錄，讓您能持續明瞭雲端使用情況、Shadow IT 以及 Shadow IT 為您的組織所帶來的風險。若要開始使用，請參閱[設定 Cloud Discovery](/cloud-app-security/set-up-cloud-discovery)。
> 
> 

## <a name="evaluate-identity-integration-requirements"></a>評估身分識別整合需求
接下來，您必須評估身分識別整合需求。 要定義使用者執行驗證的方式、組織存在於雲端中的型態、組織提供授權的方式以及將會有何種使用者經驗的技術需求，這項評估是很重要的。 請確實回答下列問題：

* 您的組織是否會使用同盟和 (或) 標準驗證？
* 同盟是必要條件嗎？  原因如下：
  * Kerberos 型 SSO
  * 您的公司有使用 SAML 或類似同盟功能的內部部署應用程式 (內建或協力廠商)。
  * 透過智慧卡的 MFA。 RSA SecurID 等等。
  * 可解決下列問題的用戶端存取規則：
    1. 我是否可根據用戶端的 IP 位址封鎖所有對 Office 365 的外部存取？
    2. 我是否可封鎖所有對 Office 365 的外部存取 (Exchange ActiveSync 除外)？
    3. 我是否可封鎖所有對 Office 365 的外部存取 (瀏覽器型應用程式 (OWA、SPO) 除外)？
    4. 我是否可針對指定的 AD 群組成員封鎖所有對 Office 365 的外部存取？
* 安全性/稽核考量
* 對同盟驗證既有的投資
* 我們的組織在雲端中的網域將使用何種名稱？
* 組織是否有自訂網域嗎？
  1. 該網域是否為公用的？可透過 DNS 輕易驗證嗎？
  2. 如果不是，您是否有公用網域可用來在 AD 中註冊替代 UPN？
* 雲端中顯示的使用者識別碼是否一致？ 
* 組織是否有需要與雲端服務整合的應用程式？
* 組織是否有多個網域？全都會使用標準或同盟驗證嗎？

## <a name="evaluate-applications-that-run-in-your-environment"></a>評估在您的環境中執行的應用程式
現在您已對內部部署和雲端基礎結構有所了解，您必須評估在這些環境中執行的應用程式。 當您定義如何將這些應用程式整合到雲端身分識別管理系統的技術需求時，這項評估是很重要的。 請確實回答下列問題：

* 我們的應用程式將在何處運作？
* 使用者會存取內部部署應用程式嗎？  還是在雲端？ 或是兩者都有？
* 是否有將現有的應用程式工作負載移至雲端的計劃？
* 是否計劃要開發將存在於內部部署或雲端中，但會使用雲端驗證的新應用程式？

## <a name="evaluate-user-requirements"></a>評估使用者需求
您也必須評估使用者需求。 要定義在使用者轉換至雲端時予以登入並提供協助所需的步驟，這項評估是很重要的。 請確實回答下列問題：

* 使用者會存取應用程式內部部署嗎？
* 使用者會存取雲端中的應用程式嗎？
* 使用者通常會以何種方式登入其內部部署環境？
* 使用者將如何登入雲端？

> [!NOTE]
> 請確定會記下每個答案，並了解答案背後的原理。 [判斷事件回應需求](plan-hybrid-identity-design-considerations-incident-response-requirements.md) 將說明可用的選項，以及每個選項的優缺點。  回答這些問題之後，您就能選取最適合業務需求的選項。
> 
> 

## <a name="next-steps"></a>後續步驟
[判斷目錄同步處理需求](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

