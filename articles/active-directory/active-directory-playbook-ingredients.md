---
title: Azure Active Directory PoC 腳本組成要素 | Microsoft Docs
description: 探索並快速實作身分識別和存取管理案例
services: active-directory
keywords: azure active directory, 腳本, 概念證明, PoC
documentationcenter: ''
author: dstefanMSFT
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2017
ms.author: dstefan
ms.openlocfilehash: ff4a8601b619837d835ec25c26b1f7e69b46ae85
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26731425"
---
# <a name="azure-active-directory-proof-of-concept-playbook-ingredients"></a>Azure Active Directory 概念證明腳本組成要素 

## <a name="theme"></a>佈景主題
Azure AD 可在企業內提供多個領域的身分識別和存取解決方案。 我們將這些案例分為下列領域︰ 

* [許多應用程式、一個身分識別](active-directory-playbook-implementation.md#theme---lots-of-apps-one-identity) 
* [增加您的安全性](active-directory-playbook-implementation.md#theme---increase-your-security) 
* [使用自助式縮放比例](active-directory-playbook-implementation.md#theme---scale-with-self-service) 

定義佈景主題來制定概念證明 (Proof of Concept, PoC) 可協助您將心力集中在與業務目標配合，因為您一開始之所以會對概念證明感興趣，原因往往是業務目標。 

## <a name="environment"></a>環境

請務必確定要用來提供 PoC 之環境的詳細資料。 理想狀況下，您可以在 PoC 完成後以此環境為基礎來進行建置。 目標環境很重要，因此請在盡量呈現真實環境和條件約束的額外負荷或其他考量之間取得適當平衡。 PoC 的典型環境如下︰
* **生產︰** 各種案例會實作在實際環境中，並已部署 Microsoft Cloud 服務 (生產 AD、Office 365、Azure AD 租用戶/SSO 解決方案)。 
* **使用者接受度測試 (UAT)/開發環境︰** 您擁有測試基礎結構 (平行 AD，並可能有 Azure AD 租用戶/SSO 解決方案) 與類似生產環境的測試資料。 測試環境一般可供企業中的多個專案共用。

本指南中的大部分案例皆具有附加特質。 因此，您可以將這些案例部署在生產租用戶中，而不會影響 PoC 外的使用者。 在整份文件中，我們會指出哪些案例會對整個租用戶造成影響。 在這些情況中，您可能要考慮使用非生產環境。 


## <a name="target-users"></a>目標使用者

請務必確定將會練習這些案例的目標使用者群，尤其是當環境為生產或測試用環境時。 PoC 的目標使用者分類如下︰
* **試驗使用者︰** 環境的實際使用者，他們會以進行日常工作職責時所用的帳戶來使用解決方案
* **測試使用者︰** 在環境中建立的測試帳戶 

本指南中的大部分案例均可供試驗使用者進行練習。 在整份文件中，我們會視需要指出目標使用者的考量事項。


[!INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]