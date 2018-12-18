---
title: 在 Azure Active Directory B2C 中管理使用者存取 | Microsoft Docs
description: 了解如何使用 Azure AD B2C 在您的應用程式中識別未成年人、收集出生日期和國家/地區資料，以及取得同意使用規定的確認。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/24/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6709fb8ae328f749b367c58f95b8a9ef8da9bc65
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42146181"
---
# <a name="manage-user-access-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中管理使用者存取

本文討論如何使用 Azure Active Directory (Azure AD) B2C 管理使用者對您應用程式的存取。 應用程式的存取管理包括：

- 識別未成年人並控制其對您應用程式的使用者存取。
- 未成年人需要家長同意才可使用您的應用程式。
- 對使用者收集生日和國家/地區資料。
- 擷取使用規定協議並管制存取。

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

## <a name="control-minor-access"></a>控制未成年人的存取

應用程式和組織可決定是否封鎖未成年人，使其無法使用並非以他們為使用對象的應用程式和服務。 或者，應用程式和組織可決定接受未成年人並管理家長同意，而依照商務規則的指示和法規允許提供經許可的體驗。 

如果將某個使用者識別為未成年人，則您可以在 Azure AD B2C 中將使用者流程設定為三種選項之一：

- **將已簽署的 JWT id_token 傳回至應用程式**：使用者在目錄中註冊後，權杖會傳回至應用程式。 接著，應用程式會套用商務規則。 例如，應用程式可在家長同意程序完成後繼續執行。 若要使用此方法，您應選擇從應用程式接收 **ageGroup** 和 **consentProvidedForMinor** 宣告。

- **將未簽署的 JSON 權杖傳送至應用程式**：Azure AD B2C 會通知應用程式使用者屬未成年人，並提供使用者的家長同意狀態。 接著，應用程式會套用商務規則。 JSON 權杖不會對應用程式完成成功的驗證。 應用程式必須根據包含在 JSON 權杖中的宣告處理未驗證的使用者，而該權杖中可能包含**名稱**、**電子郵件**、**ageGroup** 和 **consentProvidedForMinor**。

- **封鎖使用者**：如果使用者是未成年人，且未提供家長同意，則 Azure AD B2C 可告知使用者已遭封鎖。 此時將不會簽發權杖，而會封鎖存取，且註冊作業期間不會建立使用者帳戶。 若要實作此通知，您可以提供適當的 HTML/CSS 內容頁面以通知使用者，並顯示適當選項。 應用程式不需要針對新的註冊執行進一步的動作。

## <a name="get-parental-consent"></a>取得家長同意

根據應用程式的法規，可能需要由經驗證為成人的使用者授與家長同意。 Azure AD B2C 不會提供相關程序來驗證個人的年齡，然後讓經驗證的成人為未成年人授與家長同意。 此程序必須由應用程式或其他服務提供者提供。

以下是收集家長同意的使用者流程範例：

1. [Azure Active Directory 圖形 API](https://msdn.microsoft.com/library/azure/ad/graph/api/api-catalog) 作業將使用者識別為未成年人，並以未簽署的 JSON 權杖形式將使用者資料傳回至應用程式。

2. 應用程式在處理 JSON 權杖後對未成年人顯示相關畫面，告知必須提供家長同意，並要求家長於線上出具同意。 

3. Azure AD B2C 會顯示登入程序，讓使用者能正常登入，並將權杖簽發給已設定為包含 **legalAgeGroupClassification = “minorWithParentalConsent”** 的應用程式。 應用程式會收集家長的電子郵件地址，並確認家長是成人。 為此，它會使用受信任的來源，例如戶政單位、授權驗證或信用卡證明。 如果驗證成功，應用程式會提示未成年人使用 Azure AD B2C 使用者流程登入。 如果拒絕同意 (例如 **legalAgeGroupClassification = “minorWithoutParentalConsent”**)，則 Azure AD B2C 會將 JSON 權杖 (不是登入資料) 傳回至應用程式以重新啟動同意程序。 使用者流程可以選擇性地自訂，讓未成年人或成人可據以按照記錄將註冊程式碼傳送至未成年人的電子郵件地址或成人的電子郵件地址，以重新取得未成年人帳戶的存取權。

4. 應用程式會為未成年人提供撤銷同意的選項。

5. 當未成年人或成人撤銷同意時，可以使用 Azure AD 圖形 API 將 **consetProvidedForMinor** 變更為**拒絕**。 或者，應用程式可以選擇刪除同意已撤銷的未成年人。 使用者流程可以選擇性地自訂，讓已驗證的未成年人 (或使用未成年人帳戶的成人) 據以撤銷同意。 Azure AD B2C 會將 **consentProvidedForMinor** 記錄為**拒絕**。

如需 **legalAgeGroupClassification**、**consentProvidedForMinor** 和 **ageGroup** 的詳細資訊，請參閱[使用者資源類型](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/user)。 如需自訂屬性的詳細資訊，請參閱[使用自訂屬性收集取用者的相關資訊](active-directory-b2c-reference-custom-attr.md)。 在使用 Azure AD 圖形 API 來處理擴充屬性時，您必須使用屬性的完整版本，例如 *extension_18b70cf9bb834edd8f38521c2583cd86_dateOfBirth*: *2011-01-01T00:00:00Z*。

## <a name="gather-date-of-birth-and-country-data"></a>收集出生日期和國家/地區資料

應用程式在註冊期間可能會仰賴 Azure AD B2C 來收集所有使用者的出生日期 (DOB) 和國家/地區資訊。 如果這樣資訊尚不存在，應用程式將可在使用者下一次驗證 (登入) 時要求提供。 使用者若未提供其 DOB 和國家/地區資訊，將無法繼續操作。 Azure AD B2C 會根據該國家/地區的法規標準，使用這項判斷個人是否被視為未成年人。 

自訂的使用者流程可收集 DOB 和國家/地區資訊，並使用 Azure AD B2C 宣告轉換來判斷 **ageGroup**，並且將結果保存在目錄中 (或直接保存 DOB 和國家/地區資訊)。

下列步驟說明用來從使用者的出生日期計算 **ageGroup** 的邏輯：

1. 嘗試在清單中依國碼 (地區碼) 尋找國家/地區。 如果找不到國家/地區，則切換回**預設值**。

2. 如果 **MinorConsent** 節點存在於國家/地區元素中：

    a. 計算使用者要被視為成人所必須具備的出生日期。 例如，如果目前的日期為 2015 年 3 月 14 日，且 **MinorConsent** 為 18，則出生日期不可晚於 2000 年 3 月 14 日。

    b. 比較出生日期下限與實際出生日期。 如果出生日期下限早於使用者的出生日期，則計算會傳回**未成年人**作為年齡群組的計算結果。

3. 如果 **MinorNoConsentRequired** 節點存在於國家 (地區) 元素中，請使用 **MinorNoConsentRequired** 中的值重複步驟 2a 和 2b。 如果出生日期下限早於使用者的出生日期，則 2b 的輸出會傳回 **MinorNoConsentRequired**。 

4. 如果兩項計算都未傳回 true，則計算會傳回**成人**。

如果某個應用程式已透過其他方法可靠地收集到 DOB 或國家/地區資料，則該應用程式可使用圖形 API 以這項資訊更新使用者記錄。 例如︰

- 如果已知使用者是成人，請以**成人**值更新目錄屬性 **ageGroup**。
- 如果已知使用者是未成年人，請以**未成年人**值更新目錄屬性 **ageGroup**，並視需要設定 **consentProvidedForMinor**。

如需關於收集 DOB 資料的詳細資訊，請參閱[在 Azure AD B2C 中使用年齡管制](basic-age-gating.md)。

## <a name="capture-terms-of-use-agreement"></a>擷取使用規定協議

當您開發應用程式時，您通常會在使用者的應用程式中擷取其接受使用規定的確認，而使用者目錄完全不會或鮮少涉入。 不過，此時也可以使用 Azure AD B2C 使用者流程來收集使用者同意使用規定的確認、在使用者未表接受時限制存取，並根據前次接受日期和最新版使用規定的日期強制接受日後對規定的變更。

**使用規定**也可包含「同意與第三方共用資料」。 根據當地法規和商務規則，您可以收集使用者同時接受這兩項條件的確認，或者，您可以允許使用者僅接受一項條件，而不接受另一項。

下列步驟說明如何管理使用規定：

1. 使用圖形 API 和擴充屬性記錄使用規定的接受和接受的日期。 您可以使用內建和自訂的使用者流程來完成此動作。 建議您建立並使用 **extension_termsOfUseConsentDateTime** 和 **extension_termsOfUseConsentVersion** 屬性。

2. 建立標示為「接受使用規定」的必要核取方塊，並在註冊期間記錄結果。 您可以使用內建和自訂的使用者流程來完成此動作。

3. Azure AD B2C 會儲存使用規定協議和使用者的同意確認。 您可以使用圖形 API 來讀取用以記錄回應的擴充屬性 (例如讀取 **termsOfUseTestUpdateDateTime**)，以查詢任何使用者的狀態。 您可以使用內建和自訂的使用者流程來完成此動作。

4. 藉由比較接受日期與最新版使用規定的日期，要求接受更新的使用規定。 您只能藉由使用自訂使用者流程來比較日期。 使用擴充屬性 **extension_termsOfUseConsentDateTime**，並比較其值與 **termsOfUseTextUpdateDateTime** 的宣告。 如果接受日期較早，則顯示自我判斷畫面強制執行新的接受。 否則會使用原則邏輯封鎖存取。

5. 藉由比較接受的版本號碼與最近接受的版本號碼，要求接受更新的使用規定。 您只能藉由使用自訂使用者流程來比較版本號碼。 使用擴充屬性 **extension_termsOfUseConsentDateTime**，並比較其值與 **extension_termsOfUseConsentVersion** 的宣告。 如果接受日期較早，則顯示自我判斷畫面強制執行新的接受。 否則會使用原則邏輯封鎖存取。

您可以在下列情況下擷取使用規定接受確認：

- 新的使用者註冊時。 此時會顯示使用規定，並儲存接受結果。
- 先前已接受最新或作用中使用規定的使用者進行登入。 此時不會顯示使用規定。
- 尚未接受最新或作用中使用規定的使用者進行登入。 此時會顯示使用規定，並儲存接受結果。
- 進行登入的使用者已接受舊版使用規定，但該使用規定現已更新為最新版本。 此時會顯示使用規定，並儲存接受結果。

下圖顯示建議的使用者流程：

![接受使用者流程](./media/manage-user-access/user-flow.png) 

下列範例說明宣告中以日期時間為基礎的使用規定同意：

```
<ClaimsTransformations>
  <ClaimsTransformation Id="GetNewUserAgreeToTermsOfUseConsentDateTime" TransformationMethod="GetCurrentDateTime">
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="extension_termsOfUseConsentDateTime" TransformationClaimType="currentDateTime" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="IsTermsOfUseConsentRequired" TransformationMethod="IsTermsOfUseConsentRequired">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="extension_termsOfUseConsentDateTime" TransformationClaimType="termsOfUseConsentDateTime" />
    </InputClaims>
    <InputParameters>
      <InputParameter Id="termsOfUseTextUpdateDateTime" DataType="dateTime" Value="2098-01-30T23:03:45" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="result" />
    </OutputClaims>
  </ClaimsTransformation>
</ClaimsTransformations>
```

下列範例說明宣告中以版本為基礎的使用規定同意：

```
<ClaimsTransformations>
  <ClaimsTransformation Id="GetEmptyTermsOfUseConsentVersionForNewUser" TransformationMethod="CreateStringClaim">
    <InputParameters>
      <InputParameter Id="value" DataType="string" Value=""/>
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="extension_termsOfUseConsentVersion" TransformationClaimType="createdClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="GetNewUserAgreeToTermsOfUseConsentVersion" TransformationMethod="CreateStringClaim">
    <InputParameters>
      <InputParameter Id="value" DataType="string" Value="V1"/>
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="extension_termsOfUseConsentVersion" TransformationClaimType="createdClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="IsTermsOfUseConsentRequiredForVersion" TransformationMethod="CompareClaimToValue">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="extension_termsOfUseConsentVersion" TransformationClaimType="inputClaim1" />
    </InputClaims>
    <InputParameters>
      <InputParameter Id="compareTo" DataType="string" Value="V1" />
      <InputParameter Id="operator" DataType="string" Value="not equal" />
      <InputParameter Id="ignoreCase" DataType="string" Value="true" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="outputClaim" />
    </OutputClaims>
  </ClaimsTransformation>
</ClaimsTransformations> 
```

## <a name="next-steps"></a>後續步驟

- 若要了解如何刪除及匯出使用者資料，請參閱[管理使用者資料](manage-user-data.md)。
