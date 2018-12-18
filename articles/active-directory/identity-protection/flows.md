---
title: 使用 Azure AD Identity Protection 時的登入體驗 | Microsoft Docs
description: 當 Identity Protection 已降低或補救使用者時，或是原則需要 Multi-Factor Authentication 時，請提供使用者經驗的概觀。
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: de5bf637-75a7-4104-b6d8-03686372a319
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: 3c1c681e2c7ccd6b5fd3eaa3639853d99cb1e0b7
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005206"
---
# <a name="sign-in-experiences-with-azure-ad-identity-protection"></a>使用 Azure AD Identity Protection 時的登入體驗
透過 Azure Active Directory Identity Protection，您可以：

* 要求使用者進行註冊，以進行 Multi-Factor Authentication
* 處理高風險的登入和遭到入侵的使用者

系統對這些問題的回應會影響使用者的登入體驗，因為藉由提供使用者名稱和密碼直接登入已不再可行。 需要有其他步驟，才能讓使用者安全返回工作。

本文會針對可能發生的所有案例，為您提供使用者的登入體驗概觀。

**Multi-Factor Authentication**

* Multi-Factor Authentication 註冊

**有風險的登入**

* 有風險的登入復原
* 已封鎖有風險的登入
* 在有風險的登入期間註冊 Multi-Factor Authentication

**有風險的使用者**

* 遭到入侵的帳戶復原
* 已封鎖遭到入侵的帳戶

## <a name="multi-factor-authentication-registration"></a>Multi-Factor Authentication 註冊
在遭到入侵的帳戶復原流程和有風險的登入流程中，最佳的使用者體驗皆是使用者可以自行復原。 如果使用者已註冊 Multi-Factor Authentication，他們便有與其帳戶相關聯的電話號碼可用來通過安全性挑戰。 從帳戶入侵復原時，不需要技術服務人員或系統管理員介入。 因此，強烈建議您讓使用者註冊 Multi-Factor Authentication。 

系統管理員可以設定一個原則，要求使用者設定其帳戶進行其他安全性驗證。 此原則最多可讓使用者略過多重要素驗證註冊 14 天。 無法設定 14 天的寬限期。

**Multi-Factor Authentication 註冊具有三個步驟：**

1. 在第一個步驟中，使用者會得到設定此帳戶進行 Multi-Factor Authentication 之需求的相關通知。 
   
    ![補救](./media/flows/140.png "補救")
2. 若要設定 Multi-Factor Authentication，您需要讓系統知道您要連線的方式。
   
    ![補救](./media/flows/141.png "補救")
3. 系統會送出一項挑戰給您，而您需要回應。
   
    ![補救](./media/flows/142.png "補救")

## <a name="risky-sign-in-recovery"></a>有風險的登入復原
當系統管理員設定登入風險的原則後，受影響的使用者會在嘗試登入時收到通知。 

**有風險的登入流程有兩個步驟：** 

1. 使用者獲知偵測到不尋常的登入，例如從新的位置、裝置或 app 登入。 
   
    ![補救](./media/flows/120.png "補救")
2. 使用者必須解決安全性挑戰以證明其身分識別。 如果使用者已註冊 Multi-Factor Authentication，他們必須回傳送至其電話號碼的安全密碼。 由於這只是有風險的登入，並不是遭到入侵的帳戶，所以使用者不必在此流程中變更密碼。 
   
    ![補救](./media/flows/121.png "補救")

## <a name="risky-sign-in-blocked"></a>已封鎖有風險的登入
系統管理員也可以選擇設定登入風險原則，以根據風險層級防止使用者登入。 若要解除封鎖，使用者必須連絡系統管理員或技術服務人員，或者嘗試從熟悉的位置或裝置登入。 藉由解決 Multi-Factor Authentication 自行復原，不是此種情況的適用選項。

![補救](./media/flows/200.png "補救")

## <a name="compromised-account-recovery"></a>遭到入侵的帳戶復原
設定使用者風險安全性原則之後，符合原則中指定的使用者風險層級 (因而假定遭到入侵) 的使用者，必須先經歷使用者入侵復原流程，才可以登入。 

**使用者入侵復原流程有三個步驟：**

1. 使用者獲知其帳戶安全性因為可疑活動或認證外洩而有風險。
   
    ![補救](./media/flows/101.png "補救")
2. 使用者必須解決安全性挑戰以證明其身分識別。 如果使用者已註冊 Multi-Factor Authentication，他們可以從損害中自行復原。 他們必須回傳送至其電話號碼的安全密碼。 
   
   ![補救](./media/flows/110.png "補救")
3. 最後，使用者會被迫變更其密碼，因為其他人可能有其帳戶的存取權。 
   這項體驗的螢幕擷取畫面如下。
   
   ![補救](./media/flows/111.png "補救")

## <a name="compromised-account-blocked"></a>已封鎖遭到入侵的帳戶
若要讓遭到使用者風險安全性原則封鎖的使用者解除封鎖，該使用者必須連絡系統管理員或技術服務人員。 藉由解決 Multi-Factor Authentication 自行復原，不是此種情況的適用選項。

![補救](./media/flows/104.png "補救")

## <a name="reset-password"></a>重設密碼
如果遭到入侵的使用者已遭封鎖而無法登入，系統管理員可以為其產生暫時密碼。 使用者必須在下次登入期間變更密碼。

![補救](./media/flows/160.png "補救")

## <a name="see-also"></a>另請參閱
* [Azure Active Directory Identity Protection](../active-directory-identityprotection.md) 

