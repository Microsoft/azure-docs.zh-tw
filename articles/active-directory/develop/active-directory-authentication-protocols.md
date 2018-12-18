---
title: Azure Active Directory 驗證通訊協定 | Microsoft Docs
description: Azure Active Directory (AD) 所支援驗證通訊協定的概觀
documentationcenter: dev-center-name
author: CelesteDG
services: active-directory
manager: mtillman
editor: ''
ms.assetid: 7a838ae2-c24c-4304-b6c0-e77fb888e6c0
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/27/2017
ms.author: celested
ms.custom: aaddev
ms.reviewer: hirsin
ms.openlocfilehash: 04cdba261d67e20762fd4bb4835568f763124fef
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578468"
---
# <a name="azure-active-directory-authentication-protocols"></a>Azure Active Directory 驗證通訊協定
Azure Active Directory (Azure AD) 支援數個最常用的驗證和授權通訊協定。 本節中的主題描述 Azure AD 中所支援的通訊協定和其實作。 這些主題包括支援的宣告類型回顧、同盟中繼資料的使用簡介、詳細的 OAuth 2.0  和 SAML 2.0 通訊協定參考文件，以及疑難排解小節。

## <a name="authentication-protocols-articles-and-reference"></a>驗證通訊協定的文件和參考資料
* [Azure AD 中簽署金鑰變換的相關重要資訊](active-directory-signing-key-rollover.md) – 了解 Azure AD 的簽署金鑰變換模式、對自動更新金鑰進行的變更，以及如何更新最常見應用程式案例的討論。
* [支援的權杖和宣告類型](v1-id-and-access-tokens.md) - 了解 Azure AD 所發出之權杖中的宣告。
* [同盟中繼資料](azure-ad-federation-metadata.md) - 了解如何尋找並解譯 Azure AD 所產生的中繼資料文件。
* [Azure AD 中的 OAuth 2.0](v1-protocols-oauth-code.md) - 了解 Azure AD 中的 OAuth 2.0 實作。
* [OpenID Connect 1.0](v1-protocols-openid-connect-code.md) - 了解如何使用 OAuth 2.0 (授權通訊協定) 進行驗證。
* [使用用戶端認證的服務對服務呼叫](v1-oauth2-client-creds-grant-flow.md) - 了解如何使用 OAuth 2.0 用戶端認證授與流程來進行服務對服務呼叫。
* [使用代理者流程的服務對服務呼叫](v1-oauth2-on-behalf-of-flow.md) - 了解如何使用 OAuth 2.0 代理者流程來進行服務對服務呼叫。
* [SAML 通訊協定參照](active-directory-saml-protocol-reference.md) - 了解 Azure AD 的單一登入和單一登出 SAML 設定檔。

## <a name="see-also"></a>另請參閱
[Azure Active Directory 開發人員指南](azure-ad-developers-guide.md)

[Active Directory 程式碼範例](sample-v1-code.md)
