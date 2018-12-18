---
title: 在 Azure Active Directory B2C 中針對建立租用戶進行疑難排解 | Microsoft Docs
description: 關於建立 Azure Active Directory 或 Azure Active Directory B2C 租用戶的問題與解決方法。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/06/2016
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 009f7ac2f7e614b7e07623e41888973f1a2b254d
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37440971"
---
# <a name="troubleshoot-creating-an-azure-active-directory-or-azure-active-directory-b2c-tenant"></a>針對建立 Azure Active Directory 或 Azure Active Directory B2C 租用戶進行疑難排解 

## <a name="create-an-azure-ad-tenant"></a>建立 Azure AD 租用戶
如果您第一次嘗試時無法建立 Azure Active Directory (Azure AD) 租用戶，請再試一次。 若問題持續發生，請連絡 Azure 支援中心。

## <a name="create-an-azure-ad-b2c-tenant"></a>建立 Azure AD B2C 租用戶
如果您在[建立 Azure Active Directory B2C (Azure AD B2C) 租用戶](active-directory-b2c-get-started.md)時遇到問題，請嘗試下列選項：

* 若您的租用戶清單未顯示 Azure AD B2C 租用戶，請再試一次建立租用戶。
* 如果 Azure AD B2C 租用戶未顯示在您的租用戶清單中，而且您看到下列錯誤訊息，請刪除租用戶，然後再一次建立：

    「無法完成建立 B2C 租用戶 'contosob2c'。 如需詳細指引，請造訪此[連結](http://go.microsoft.com/fwlink/?LinkID=624192&clcid=0x409)。」
* 當您刪除現有的 Azure AD B2C 租用戶並使用相同的網域名稱加以重建時，會發生已知的問題。 當您建立新的 Azure AD B2C 租用戶時，您必須使用不同的網域名稱。
* 若這些解決方法無效，請連絡 Azure 支援中心。 如需詳細資訊，請參閱[提出 Azure AD B2C 的支援要求](active-directory-b2c-support.md)。

