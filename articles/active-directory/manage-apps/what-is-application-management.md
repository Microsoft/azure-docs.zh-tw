---
title: 使用 Azure Active Directory 管理應用程式 | Microsoft Docs
description: 此文章章說明整合 Azure Active Directory 與您的內部部署、雲端和 SaaS 應用程式的優點。
services: active-directory
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.topic: overview
ms.workload: identity
ms.date: 09/11/2018
ms.author: barbkess
ms.reviewer: arvinh
ms.openlocfilehash: 820c6a5cb0bf5fb2d98d27c171c1276866082700
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44716176"
---
# <a name="application-management-with-azure-active-directory"></a>搭配 Azure Active Directory 的應用程式管理

Azure Active Directory (Azure AD) 能針對雲端和內部部署應用程式提供安全且順暢的存取。 使用者僅需登入一次，便可以存取 Office 365 及其他來自 Microsoft 的商務應用程式、數千個軟體即服務 (SaaS) 應用程式、內部部署應用程式，以及企業營運 (LOB) 應用程式。 透過將使用者佈建自動化來減少系統管理成本。 使用多重要素驗證及條件式存取原則來提供安全的應用程式存取。

![透過 Azure AD 同盟的應用程式](media/migrate-adfs-apps-to-azure/migrate2.png)

## <a name="why-manage-applications-with-a-cloud-solution"></a>為何要以雲端解決方案來管理應用程式？

組織通常會有使用者完成工作所需的數百個應用程式。 使用者會從許多裝置及位置存取這些應用程式。 每天都有許多應用程式被新增、開發及淘汰。 由於有這麼多的應用程式及存取點，使用雲端式解決方案來管理所有應用程式的使用者存取就變得更加重要。

## <a name="manage-risk-with-conditional-access-policies"></a>使用條件式存取原則來管理風險
結合 Azure AD 單一登入 (SSO) 及條件式存取，將能針對應用程式的存取提供高層級的安全性。 安全性功能包括雲端規模的身分識別保護、以風險為基礎的存取控制、原生的多重要素驗證，以及條件式存取原則。 這些功能可提供以應用程式為基礎的細微控制原則，或是針對需要較高安全性層級的群組提供。

## <a name="improve-productivity-with-single-sign-on"></a>使用單一登入改善生產力
在應用程式和 Office 365 上啟用單一登入 (SSO)，能透過減少或消除登入提示，來為現有的使用者提供更優異的登入體驗。 透過免除看見多次提示，或是管理多組密碼的需求，使用者的作業環境將會變得更為一致，且較不容易令人分心。 商務群組可透過自助服務及動態成員資格來管理及核准存取權。 透過僅允許公司中的適當人員來管理應用程式的存取權，便能改善身分識別系統的安全性。

SSO 能改善安全性。 「在沒有單一登入的情況下」，系統管理員便需要逐一針對個別應用程式建立並更新使用者帳戶，這需要花費許多時間。 此外，使用者必須記住多個認證來存取其應用程式。 如此一來，使用者會傾向將密碼寫下，或使用其他密碼管理解決方案，而這會帶來資料安全性上的風險。 

## <a name="address-governance-and-compliance"></a>處理治理和合規性
透過 Azure AD，您可以透過能運用安全性事件與事件監視 (SIEM) 工具的報告，來監視應用程式登入。 您可以從入口網站或 API 存取這些報告。 以程式設計方式對存取您應用程式的人員進行稽核，並透過存取權檢閱來移除非使用中使用者的存取權。

## <a name="manage-costs"></a>管理成本
透過移轉至 Azure AD，您可以降低成本並免除自行管理內部部署基礎結構的麻煩。 Azure AD 也能針對應用程式提供自助存取，這能同時為系統管理員和使用者節省寶貴時間。 單一登入能免除使用應用程式特定的密碼，這能省下因重設應用程式密碼，以及擷取密碼時喪失生產力所導致的相關成本。

