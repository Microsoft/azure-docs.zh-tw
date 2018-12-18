---
title: 在 Azure Active Directory B2C 中使用年齡管制 | Microsoft Docs
description: 了解如何使用應用程式識別未成年人。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 04/29/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: a1020dfcb6c8d312001fbdb1c170987e1216c5d5
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318855"
---
# <a name="using-age-gating-in-azure-ad-b2c"></a>在 Azure AD B2C 中使用年齡管制

>[!IMPORTANT]
>這項功能處於個人預覽版狀態。  請參閱我們已發佈的[服務部落格](https://blogs.msdn.microsoft.com/azureadb2c/)以取得詳細資訊，或連絡 AADB2CPreview@microsoft.com。  請「不要」在生產目錄上使用此功能，在公開上市之前使用這些新功能可能會導致資料遺失，且可能會產生非預期的行為變更。  
>

## <a name="age-gating"></a>年齡管制
年齡管制可讓您使用 Azure AD B2C 識別應用程式中的未成年人。  您可以選擇封鎖使用者而使其無法登入應用程式，或讓他們在另行發出宣告而指出使用者的年齡群組和其家長同意狀態後返回應用程式。  

>[!NOTE]
>家長同意狀態會在名為 `consentProvidedForMinor` 的使用者屬性中受到追蹤。  您可以透過圖形 API 更新此屬性，且系統在更新 `legalAgeGroupClassification` 時將會使用此欄位。
>

## <a name="setting-up-your-directory-for-age-gating"></a>設定年齡管制的目錄
若要在使用者流程中使用年齡管制，您必須設定目錄，使其具有其他屬性。 您可以透過功能表中的 `Properties` 來執行這項作業 (您必須已註冊個人預覽版，才能使用該項目)。  
1. 在 Azure AD B2C 擴充功能中，在左側的功能表中按一下您租用戶的 [屬性]。
2. 在 [年齡管制] 區段下，按一下 [設定] 按鈕。
3. 等候作業完成，您的目錄將會進行年齡管制的設定。

## <a name="enabling-age-gating-in-your-user-flow"></a>在使用者流程中啟用年齡管制
在您的目錄設定為使用年齡管制後，您即可在預覽版的使用者流量中使用這項功能。  此功能需要進行變更，使其與現有類型的使用者流量不相容。  您可以透過下列步驟啟用年齡管制：
1. 建立預覽使用者流量。
2. 建立之後，請移至功能表中的 [屬性]。
3. 在 [年齡管制] 區段中，按切換按鈕以啟用功能。
4. 然後，您可以選擇要如何管理識別為未成年人的使用者。

## <a name="what-does-enabling-age-gating-do"></a>啟用年齡管制的作用為何？
在使用者流程中啟用年齡管制後，使用者體驗就會變更。  在註冊時，除了為使用者流程設定的使用者屬性以外，此時系統還會要求使用者提供其出生日期和居住地國家/地區。  在登入時，對於先前未曾輸入出生日期和居住地國家/地區的使用者，系統將會在使用者下一次登入時要求這項資訊。  Azure AD B2C 將藉由這兩個值來識別使用者是否為未成年人，並更新 `ageGroup` 欄位，此值可以是 `null`、`Undefined`、`Minor`、`Adult` 和 `NotAdult`。  接著，系統會使用 `ageGroup` 和 `consentProvidedForMinor` 欄位來計算 `legalAgeGroupClassification`。 

## <a name="age-gating-options"></a>年齡管制選項
您可以選擇讓 Azure AD B2C 直接封鎖未取得家長同意的未成年人；也可以允許他們進入，而讓應用程式決定要如何處理。  

### <a name="allowing-minors-without-parental-consent"></a>允許未取得家長同意的未成年人
對於允許註冊和/或登入的使用者流量，您可以選擇允許未經同意的未成年人進入您的應用程式。  未取得家長同意的未成年人可以正常登入或註冊，Azure AD B2C 會以 `legalAgeGroupClassification` 宣告發出識別碼權杖。  藉由使用此宣告，您可以選擇這些使用者所將擁有的體驗，例如收集家長同意 (並更新 `consentProvidedForMinor` 欄位) 的體驗。

### <a name="blocking-minors-without-parental-consent"></a>封鎖未取得家長同意的未成年人
對於允許註冊和/或登入的使用者流量，您可以選擇封鎖未經應用程式同意的未成年人。  有兩個選項可處理 Azure AD B2C 中遭封鎖的使用者：
* 將 JSON 傳回至應用程式 - 此選項會將回應傳回至封鎖未成年人的應用程式。
* 顯示錯誤頁面 - 使用者將會看見一個頁面，指出他們無法存取應用程式
