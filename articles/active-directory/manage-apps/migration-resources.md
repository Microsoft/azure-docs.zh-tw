---
title: 將應用程式移轉至 Azure Active Directory 的資源 | Microsoft Docs
description: 協助您將應用程式存取權限和驗證移轉至 Azure Active Directory (Azure AD) 的資源。
services: active-directory
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 09/19/2018
ms.author: barbkess
ms.reviewer: baselden
ms.openlocfilehash: 750bf1905a3ca352e181dcd5b7fcecdfc8d04f76
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465498"
---
# <a name="resources-for-migrating-applications-to-azure-active-directory"></a>將應用程式移轉至 Azure Active Directory 的資源

協助您將應用程式存取權限和驗證移轉至 Azure Active Directory (Azure AD) 的資源。 

| 資源  | 說明  |
|:-----------|:-------------|
|[將您的應用程式移轉至 Azure AD](https://aka.ms/migrateapps/whitepaper) \(英文\) | 此白皮書提供移轉的優點，並以下列四個清楚描述的階段，說明如何規劃移轉：探索、分類、移轉和持續管理。 您將會透過逐步引導，了解如何思考流程，並將您的專案細分成便於取用的片段。 此文件全篇都有在過程中可協助您的重要資源連結。 |
|[解決方案指南：從 Active Directory 同盟服務 (AD FS) 將應用程式移轉到 Azure AD](https://aka.ms/migrateapps/adfssolutionguide) \(英文\) | 此解決方案指南將以比移轉白皮書中更概略的說明方式，引導您完成相同的規劃四階段，並執行應用程式移轉專案。 在本指南中，您將了解如何將那些階段，套用至從 Active Directory 同盟服務 (AD FS) 移動應用程式到 Azure AD 的特定目標。|
| [工具：Active Directory 同盟服務移轉整備指令碼](https://aka.ms/migrateapps/adfstools) \(英文\) | 這是您可以在內部部署 Active Directory 同盟服務 (AD FS) 伺服器上執行的指令碼，可用來判斷要移轉至 Azure AD 之應用程式的整備程度。|
| [部署方案：從 AD FS 移轉至密碼雜湊同步處理](https://aka.ms/ADFSTOPHSDPDownload) \(英文\) | 使用密碼雜湊同步處理時，可以將使用者密碼的雜湊從內部部署 Active Directory 同步至 Azure AD。 這可讓 Azure AD 驗證使用者，而不需要與內部部署 Active Directory 互動。| 
| [部署方案：從 AD FS 移轉至傳遞驗證](https://aka.ms/ADFSTOPTADPDownload) \(英文\)|Azure AD 傳遞驗證可協助使用者以相同的密碼，同時登入內部部署和雲端式應用程式。 此功能可為使用者提供更好的體驗，因為可以少記一個密碼。 因為當使用者只需要記住一個密碼時，就比較不會忘記如何登入，所以也能降低 IT 技術服務人員的成本。 當使用者使用 Azure AD 登入時，此功能會向您的內部部署 Active Directory 直接驗證使用者的密碼。|
| [部署方案：使用 Azure AD 啟用單一登入至 SaaS 應用程式](https://aka.ms/SSODPDownload) \(英文\) | 單一登入 (SSO) 可協助您只使用單一使用者帳戶登入一次，然後就能夠存取所有進行工作所需的應用程式和資源。 例如，在使用者登入之後，使用者可以從 Microsoft Office 移至 SalesForce、Box，而不需要二次驗證 (例如輸入密碼)。 
| [部署方案：使用應用程式 Proxy 將應用程式延伸到 Azure AD](https://aka.ms/AppProxyDPDownload) \(英文\)| 傳統上，提供從員工的膝上型電腦和其他裝置存取內部部署應用程式的權限會涉及虛擬私人網路 (VPN) 或周邊網路 (DMZ)。 這些解決方案不僅複雜且難以確保安全，而且設定及管理成本也很高。 Azure AD 應用程式 Proxy 可讓存取內部部署應用程式變得更輕鬆。 |
| [部署方案](../fundamentals/active-directory-deployment-plans.md) | 尋找更多部署方案以部署功能，例如多重要素驗證、條件式存取、使用者佈建、無縫 SSO、自助式密碼重設等其他功能！ |


