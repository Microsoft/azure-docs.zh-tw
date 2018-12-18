---
title: Azure Active Directory Identity Protection 偵測到的弱點 | Microsoft Docs
description: Azure Active Directory Identity Protection 偵測到的弱點概觀。
services: active-directory
keywords: azure active directory identity protection, cloud discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: 92233a5b-cb34-4d28-88cc-d5d29c0f3256
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/27/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 680e52fefd8256b3ac270e8d721f27645ced49eb
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40004246"
---
# <a name="vulnerabilities-detected-by-azure-active-directory-identity-protection"></a>Azure Active Directory Identity Protection 偵測到的弱點
弱點是您的環境中攻擊者可以利用的弱點。 我們建議您處理這些弱點，以改善組織的安全性狀態，並防止攻擊者利用這些弱點。


![弱點](./media/vulnerabilities/101.png "弱點")



下列各節概述 Identity Protection 所報告的弱點。

## <a name="multi-factor-authentication-registration-not-configured"></a>未設定 Multi-Factor Authentication 註冊
此弱點可協助您控制組織中部署的 Azure Multi-Factor Authentication。 

Azure Multi-Factor Authentication 會為使用者驗證提供第二層安全性。 這有助於保護對資料與應用程式的存取，同時可以滿足使用者對簡單登入程序的需求。 它可以透過一些簡單的驗證選項 (例如電話、文字訊息，或行動應用程式通知或驗證碼，以及第三方 OATH 權杖) 來提供強大的驗證功能。

建議您要求對使用者登入進行 Multi-Factor Authentication。在 Identity Protection 提供的以風險為基礎的條件式存取原則中，Multi-Factor Authentication 扮演關鍵角色。

如需詳細資訊，請參閱[什麼是 Azure Multi-Factor Authentication？](../authentication/multi-factor-authentication.md)。

## <a name="unmanaged-cloud-apps"></a>非受控雲端應用程式
此弱點可協助您識別組織中非受控雲端應用程式。

在現代企業中，IT 部門通常不會知道組織中的使用者用於進行工作的所有雲端應用程式。 您應該不難想像為什麼系統管理員必須對未經授權存取公司資料、可能的資料外洩和其他安全性風險有所顧慮。 

建議您部署 Cloud Discovery 來探索非受控雲端應用程式，並使用 Azure Active Directory 來管理這些應用程式。

如需詳細資訊，請參閱 [Cloud Discovery](/cloud-app-security/set-up-cloud-discovery)。

## <a name="security-alerts-from-privileged-identity-management"></a>來自 Privileged Identity Management 的安全性警示
此弱點可協助您找出並解決有關您組織中特殊權限身分識別的警示。  

若要讓使用者能夠執行特殊權限作業，組織必須賦予使用者在 Azure AD、Azure 或 Office 365 資源或其他 SaaS 應用程式中的暫時或永久特殊權限存取權。 這些特殊權限的使用者會增加您組織的受攻擊面。 此弱點可協助您識別具有非必要特殊權限存取權的使用者，並採取適當的行動來減少或消除其所造成的風險。 

建議您的組織使用 Azure AD Privileged Identity Management 來管理、控制和監視特殊權限身分識別，以及其在 Azure AD 和其他 Microsoft 線上服務 (如 Office 365 或 Microsoft Intune) 中的資源存取權。

如需詳細資訊，請參閱 [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md)。 

## <a name="see-also"></a>另請參閱

[Azure Active Directory Identity Protection](../active-directory-identityprotection.md)

