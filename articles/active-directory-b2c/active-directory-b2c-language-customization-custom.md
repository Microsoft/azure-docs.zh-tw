---
title: Azure Active Directory B2C 中的自訂原則語言自訂 | Microsoft Docs
description: 了解如何在多語言的自訂原則中使用當地語系化內容。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/13/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 6269ac65e5db20521346d5312bcbadd0905c36e2
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37440559"
---
# <a name="language-customization-in-custom-policies"></a>自訂原則中的語言自訂

> [!NOTE]
> 這項功能處於公開預覽狀態。
> 

在自訂原則中，語言自訂的運作方式與內建原則相同。  請參閱內建[文件](https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-language-customization)，內容描述如何根據參數和瀏覽器設定來選擇語言的行為。

## <a name="enable-supported-languages"></a>啟用支援的語言
如果未指定 UI 地區設定，而使用者的瀏覽器要求其中一種語言，則會以支援的語言顯示。  

支援的語言以下列格式定義於 `<BuildingBlocks>` 中：

```XML
<BuildingBlocks>
  <Localization>
    <SupportedLanguages DefaultLanguage="en">
      <SupportedLanguage>qps-ploc</SupportedLanguage>
      <SupportedLanguage>en</SupportedLanguage>
    </SupportedLanguages>
  </Localization>
</BuildingBlocks>
```

預設語言和支援的語言的運作方式和內建原則相同。

## <a name="enable-custom-language-strings"></a>啟用自訂語言字串

建立自訂語言字串需要兩個步驟：
1. 編輯頁面的 `<ContentDefinition>`，以指定所需語言的資源識別碼
2. 在您的 `<BuildingBlocks>` 中以對應的識別碼建立 `<LocalizedResources>`

請記住，您可以同時在延伸檔案或信賴原則檔案中放入 `<ContentDefinition>` 和 `<BuildingBlock>`，這取決於您是否希望變更在所有的繼承原則中生效。

### <a name="edit-the-contentdefinition-for-the-page"></a>編輯頁面的 ContentDefinition

對於您想要當地語系化的每個頁面，您可以在 `<ContentDefinition>` 中指定每個語言代碼尋找的語言資源。

```XML
<ContentDefinition Id="api.signuporsignin">
      <LocalizedResourcesReferences>
        <LocalizedResourcesReference Language="fr" LocalizedResourcesReferenceId="fr" />
        <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="en" />
      </LocalizedResourcesReferences>
</ContentDefinition>
```

在此範例中，法文 (fr) 和英文 (en) 自訂字串會加入至統一註冊或登入頁面。  每個 `LocalizedResourcesReference` 的 `LocalizedResourcesReferenceId` 和其地區設定相同，但您可以使用任何字串做為識別碼。  對於每個語言與頁面組合，您必須建立對應的 `<LocalizedResources>`，如下所示。


### <a name="create-the-localizedresources"></a>建立 LocalizedResources

覆寫項目會包含在您的 `<BuildingBlocks>` 中，而且您在 `<ContentDefinition>` 中為每個頁面指定的每個頁面和語言組合都有一個 `<LocalizedResources>`。  每個覆寫項目會指定為一個 `<LocalizedString>`，如下列範例所示：

```XML
<BuildingBlocks>
  <Localization>
    <LocalizedResources Id="en">
      <LocalizedStrings>
        <LocalizedString ElementType="ClaimsProvider" StringId="SignInWithLogonNameExchange">Local Account Sign-in</LocalizedString>
        <LocalizedString ElementType="ClaimType" ElementId="UserId" StringId="DisplayName">Username</LocalizedString>
        <LocalizedString ElementType="ClaimType" ElementId="UserId" StringId="UserHelpText">Username used for signing in.</LocalizedString>
        <LocalizedString ElementType="ClaimType" ElementId="UserId" StringId="PatternHelpText">The username you provided is not valid.</LocalizedString>
        <LocalizedString ElementType="UxElement" StringId="button_signin">Sign In Now</LocalizedString>
        <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidPassword">Your password is incorrect.</LocalizedString>
      </LocalizedStrings>
    </LocalizedResources>
  </Localization>
</BuildingBlocks>
```

頁面上有四種類型的字串項目：

**ClaimsProvider** - 識別提供者 (Facebook、Google 及 Azure AD 等) 的標籤。**ClaimType** - 屬性和其對應說明文字或欄位驗證錯誤的標籤 **UxElement** - 頁面上預設的其他字串項目，例如按鈕、連結或文字 **ErrorMessage** - 表單驗證錯誤訊息

確認使用這些覆寫項目之頁面的 `StringId` 是否相符，否則上傳時原則驗證會將它封鎖。  