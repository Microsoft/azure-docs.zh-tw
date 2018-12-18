---
title: 在 Azure Active Directory B2C 中將您自己的屬性新增至自訂原則 | Microsoft Docs
description: 逐步解說如何使用擴充屬性 (Property) 和自訂屬性 (Attribute)，並將其包含於使用者介面中。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/04/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 5513e0ff434862ea7eee42cb94ff2a0f67f6d390
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338739"
---
# <a name="azure-active-directory-b2c-use-custom-attributes-in-a-custom-profile-edit-policy"></a>Azure Active Directory B2C︰在自訂設定檔編輯原則中使用自訂屬性

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

在本文中，您將在 Azure Active Directory (Azure AD) B2C 目錄中建立自訂屬性。 您將使用這個新屬性，作為設定檔編輯使用者旅程圖中的自訂宣告。

## <a name="prerequisites"></a>必要條件

依照 [Azure Active Directory B2C：開始使用自訂原則](active-directory-b2c-get-started-custom.md)一文中的步驟執行。

## <a name="use-custom-attributes-to-collect-information-about-your-customers-in-azure-ad-b2c-by-using-custom-policies"></a>使用自訂屬性，利用自訂原則來收集您的客戶在 Azure AD B2C 中的相關資訊
您的 Azure AD B2C 目錄隨附一組內建屬性。 範例包括**名字**、**姓氏**、**區/鄉/鎮/市**、**郵遞區號**及 **userPrincipalName**。 您通常必須建立自有屬性，例如下列範例：
* 面對客戶的應用程式需要保有 **LoyaltyNumber** 之類的屬性。
* 識別提供者擁有必須儲存的唯一使用者識別碼，例如 **uniqueUserGUID**。
* 自訂使用者旅程圖需要保有使用者的狀態，例如 **migrationStatus**。

Azure AD B2C 會擴充每個使用者帳戶所儲存的屬性組合。 您也可以使用 [Azure AD Graph API](active-directory-b2c-devquickstarts-graph-dotnet.md)讀取和寫入這些屬性。

擴充屬性會擴充目錄中的使用者物件結構描述。 「擴充屬性」、「自訂屬性」及「自訂宣告」等術語在本文內容中係指相同的動作。 名稱會根據內容 (例如應用程式、物件或原則) 而有所不同。

擴充屬性只能在應用程式物件上註冊，即使它們可能包含使用者的資料也一樣。 屬性會附加至應用程式。 應用程式物件必須具備寫入權限才能註冊擴充屬性。 任何單一物件均可跨所有類型和所有應用程式寫入一百個擴充屬性。 擴充屬性會新增到目標目錄類型，而且會在 Azure AD B2C 目錄租用戶中立即變成可供存取。
如果刪除應用程式，則那些擴充屬性以及其中包含的所有使用者資料也會全部移除。 如果應用程式刪除某個擴充屬性，就會從目標目錄物件上移除該擴充屬性，並刪除值。

擴充屬性只存在於租用戶中已註冊應用程式的內容裡。 該應用程式的物件識別碼必須包含在使用該識別碼的 **TechnicalProfile** 中。

>[!NOTE]
>Azure AD B2C 目錄通常包含名為 `b2c-extensions-app` 的 Web 應用程式。 此應用程式主要是供 B2C 內建原則用於透過 Azure 入口網站建立的自訂宣告。 我們僅建議進階使用者使用此應用程式來為 B2C 自訂原則註冊擴充屬性。  
指示包含於本文的＜後續步驟＞一節中。


## <a name="create-a-new-application-to-store-the-extension-properties"></a>建立新的應用程式來儲存擴充屬性

1. 開啟瀏覽工作階段並瀏覽至 [Azure 入口網站](https://portal.azure.com)。 使用您所要設定 B2C 目錄的系統管理認證來登入。
2. 在左方的導覽功能表中，選取 [Azure Active Directory]。 您可能需要選取 [更多服務] 才能找到它。
3. 選取 [應用程式註冊]。 選取 [新增應用程式註冊]。
4. 提供下列項目：
    * Web 應用程式的名稱：**WebApp-GraphAPI-DirectoryExtensions**。
    * 應用程式類型︰**Web 應用程式/API**。
    * 登入 URL：**https://{租用戶名稱}.onmicrosoft.com/WebApp-GraphAPI-DirectoryExtensions**。
5. 選取 [建立] 。
6. 選取新建立的 Web 應用程式。
7. 選取 [設定] > [必要權限]。
8. 選取 API [Windows Azure Active Directory]。
9. 在 [應用程式權限] 的 [讀取及寫入目錄資料] 輸入核取記號。 然後選取 [儲存]。
10. 選擇 [授與權限]，然後確認 [是]。
11. 將下列識別碼複製到剪貼簿並加以儲存：
    * **應用程式識別碼**。 範例： `103ee0e6-f92d-4183-b576-8c3739027780`.
    * **物件識別碼**。 範例： `80d8296a-da0a-49ee-b6ab-fd232aa45201`.



## <a name="modify-your-custom-policy-to-add-the-applicationobjectid"></a>修改您的自訂原則以新增 **ApplicationObjectId**

當您依照 [Azure Active Directory B2C：開始使用自訂原則](active-directory-b2c-get-started-custom.md)中的步驟執行時，便已經下載並修改名為 **TrustFrameworkBase.xml**、**TrustFrameworkExtensions.xml**、**SignUpOrSignin.xml**、**ProfileEdit.xml** 及 **PasswordReset.xml** 的[範例檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip) \(英文\)。 在此步驟中，您可以對那些檔案進行更多修改。

* 開啟 TrustFrameworkBase.xml 檔案，並新增 `Metadata` 區段，如下列範例所示。 插入您先前為 `ApplicationObjectId` 值記錄的物件識別碼，以及為 `ClientId` 值記錄的應用程式識別碼： 

    ```xml
    <ClaimsProviders>
        <ClaimsProvider>
          <DisplayName>Azure Active Directory</DisplayName>
            <TechnicalProfile Id="AAD-Common">
          <DisplayName>Azure Active Directory</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
              
          <!-- Provide objectId and appId before using extension properties. -->
          <Metadata>
            <Item Key="ApplicationObjectId">insert objectId here</Item>
            <Item Key="ClientId">insert appId here</Item>
          </Metadata>
          <!-- End of changes -->
              
          <CryptographicKeys>
            <Key Id="issuer_secret" StorageReferenceId="TokenSigningKeyContainer" />
          </CryptographicKeys>
          <IncludeInSso>false</IncludeInSso>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
        </ClaimsProvider>
    </ClaimsProviders>
    ```

> [!NOTE]
> 當 **TechnicalProfile** 第一次寫入至新建立的擴充屬性時，您可能會遇到一次性錯誤。 擴充屬性會在第一次使用時建立。  

## <a name="use-the-new-extension-property-or-custom-attribute-in-a-user-journey"></a>在使用者旅程圖中使用新的擴充屬性或自訂屬性

1. 開啟 ProfileEdit.xml 檔案。
2. 新增自訂宣告 `loyaltyId`。 藉由將自訂宣告納入 `<RelyingParty>` 元素中，便能將它包含於應用程式的權杖中。
    
    ```xml
    <RelyingParty>
      <DefaultUserJourney ReferenceId="ProfileEdit" />
      <TechnicalProfile Id="PolicyProfile">
        <DisplayName>PolicyProfile</DisplayName>
        <Protocol Name="OpenIdConnect" />
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
          <OutputClaim ClaimTypeReferenceId="city" />

          <!-- Provide the custom claim identifier -->
          <OutputClaim ClaimTypeReferenceId="extension_loyaltyId" />
          <!-- End of changes -->
        </OutputClaims>
        <SubjectNamingInfo ClaimType="sub" />
      </TechnicalProfile>
    </RelyingParty>
    ```

3. 開啟 TrustFrameworkExtensions.xml 檔案，並將 `<ClaimsSchema>` 元素及其子項目新增到 `BuildingBlocks` 元素：

    ```xml
    <BuildingBlocks>
      <ClaimsSchema> 
        <ClaimType Id="extension_loyaltyId"> 
          <DisplayName>Loyalty Identification Tag</DisplayName> 
          <DataType>string</DataType> 
          <UserHelpText>Your loyalty number from your membership card</UserHelpText> 
          <UserInputType>TextBox</UserInputType> 
        </ClaimType> 
      </ClaimsSchema>
    </BuildingBlocks>
    ```

4. 將相同的 `ClaimType` 定義新增到 TrustFrameworkBase.xml 中。 您不需要同時在基底和擴充檔案中新增 `ClaimType` 定義。 不過，後續步驟會將 `extension_loyaltyId` 新增至基底檔案中的 **TechnicalProfiles**。 因此，原則驗證程式會在沒有它的情況下拒絕上傳基底檔案。 針對 **TrustFrameworkBase.xml** 檔案中名為 **ProfileEdit** 的使用者旅程圖，這在追蹤其執行情形時可能很有用。 在您的編輯器中搜尋具有相同名稱的使用者旅程圖。 留意到協調流程步驟 5 會叫用 **TechnicalProfileReferenceID="SelfAsserted-ProfileUpdate**。 搜尋並檢查此 **TechnicalProfile** 以熟悉流程。

5. 開啟 **TrustFrameworkBase.xml** 檔案，並在 **TechnicalProfile SelfAsserted-ProfileUpdate** 中新增 `loyaltyId` 作為輸入和輸出宣告：

    ```xml
    <TechnicalProfile Id="SelfAsserted-ProfileUpdate">
      <DisplayName>User ID signup</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ContentDefinitionReferenceId">api.selfasserted.profileupdate</Item>
      </Metadata>
      <IncludeInSso>false</IncludeInSso>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="alternativeSecurityId" />
        <InputClaim ClaimTypeReferenceId="userPrincipalName" />
        <InputClaim ClaimTypeReferenceId="givenName" />
            <InputClaim ClaimTypeReferenceId="surname" />

        <!-- Add the loyalty identifier -->
        <InputClaim ClaimTypeReferenceId="extension_loyaltyId"/>
        <!-- End of changes -->
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="executed-SelfAsserted-Input" DefaultValue="true" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        
        <!-- Add the loyalty identifier -->
        <OutputClaim ClaimTypeReferenceId="extension_loyaltyId"/>
        <!-- End of changes -->

      </OutputClaims>
      <ValidationTechnicalProfiles>
        <ValidationTechnicalProfile ReferenceId="AAD-UserWriteProfileUsingObjectId" />
      </ValidationTechnicalProfiles>
    </TechnicalProfile>
    ```

6. 在 **TrustFrameworkBase.xml** 檔案中，將 `loyaltyId` 宣告新增至 **TechnicalProfile AAD-UserWriteProfileUsingObjectId**。 這個新增項目會在目錄中針對目前的使用者，將宣告的值保存於擴充屬性中：

    ```xml
    <TechnicalProfile Id="AAD-UserWriteProfileUsingObjectId">
      <Metadata>
        <Item Key="Operation">Write</Item>
        <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">false</Item>
        <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
      </Metadata>
      <IncludeInSso>false</IncludeInSso>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" Required="true" />
      </InputClaims>
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="objectId" />
        <PersistedClaim ClaimTypeReferenceId="givenName" />
        <PersistedClaim ClaimTypeReferenceId="surname" />

        <!-- Add the loyalty identifier -->
        <PersistedClaim ClaimTypeReferenceId="extension_loyaltyId" />
        <!-- End of changes -->

      </PersistedClaims>
      <IncludeTechnicalProfile ReferenceId="AAD-Common" />
    </TechnicalProfile>
    ```

7. 在 **TrustFrameworkBase.xml** 檔案中，將 `loyaltyId` 宣告新增至 **TechnicalProfile AAD-UserReadUsingObjectId**，以在每次使用者登入時讀取擴充屬性的值。 到目前為止，我們僅在本機帳戶的流程中變更 **TechnicalProfiles**。 如果您想要將新屬性新增至社交或同盟帳戶的流程中，就必須變更一組不同的 **TechnicalProfiles**。 請參閱＜後續步驟＞一節。

    ```xml
    <TechnicalProfile Id="AAD-UserReadUsingObjectId">
      <Metadata>
        <Item Key="Operation">Read</Item>
        <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
      </Metadata>
      <IncludeInSso>false</IncludeInSso>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" Required="true" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="otherMails" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />

        <!-- Add the loyalty identifier -->
        <OutputClaim ClaimTypeReferenceId="extension_loyaltyId" />
        <!-- End of changes -->

      </OutputClaims>
      <IncludeTechnicalProfile ReferenceId="AAD-Common" />
    </TechnicalProfile>
    ```

## <a name="test-the-custom-policy"></a>測試自訂原則

1. 開啟 [Azure AD B2C] 刀鋒視窗，然後瀏覽至 [識別體驗架構] > [自訂原則]。
1. 選取您上傳的自訂原則。 選取 [立即執行]。
1. 使用電子郵件地址來註冊。

傳回應用程式的識別碼權杖會以自訂宣告的形式包含新的擴充屬性，並會在前面加上 **extension_loyaltyId**。 請參閱下列範例：

```json
{
  "exp": 1493585187,
  "nbf": 1493581587,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "sub": "a58e7c6c-7535-4074-93da-b0023fbaf3ac",
  "aud": "4e87c1dd-e5f5-4ac8-8368-bc6a98751b8b",
  "acr": "b2c_1a_trustframeworkprofileedit",
  "nonce": "defaultNonce",
  "iat": 1493581587,
  "auth_time": 1493581587,
  "extension_loyaltyId": "abc",
  "city": "Redmond"
}
```

## <a name="next-steps"></a>後續步驟

1. 藉由變更下列 **TechnicalProfiles**，將新宣告新增至流程以登入社交帳戶。 社交和同盟帳戶會使用這兩個 **TechnicalProfiles** 來登入。 它們使用 **alternativeSecurityId** 作為使用者物件的定位器，來寫入和讀取使用者資料。

  ```xml
    <TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">

    <TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
  ```

2. 在內建和自訂原則之間使用相同的擴充屬性。 當您透過入口網站體驗新增擴充 (或自訂) 屬性時，系統會使用存在於每個 B2C 租用戶中的 **b2c-extensions-app** 來註冊那些屬性。 採取下列步驟，在自訂原則中使用擴充屬性：

  a. 在 portal.azure.com 中的 B2C 租用戶內，瀏覽至 [Azure Active Directory]，然後選取 [應用程式註冊]。  
  b. 尋找您的 **b2c-extensions-app** 並加以選取。  
  c. 在 [基本資訊] 之下，輸入**應用程式識別碼**和**物件識別碼**。  
  d. 將它們包含在您的 **AAD-Common** TechnicalProfile 中繼資料內：  

  ```xml
      <ClaimsProviders>
        <ClaimsProvider>
              <DisplayName>Azure Active Directory</DisplayName>
            <TechnicalProfile Id="AAD-Common">
                <DisplayName>Azure Active Directory</DisplayName>
                <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
                <!-- Provide objectId and appId before using extension properties. -->
                <Metadata>
                  <Item Key="ApplicationObjectId">insert objectId here</Item> <!-- This is the "Object ID" from the "b2c-extensions-app"-->
                  <Item Key="ClientId">insert appId here</Item> <!--This is the "Application ID" from the "b2c-extensions-app"-->
                </Metadata>
  ```

3. 與入口網站體驗保持一致性。 先使用入口網站 UI 建立這些屬性，然後在自訂原則中使用它們。 當您在入口網站中建立屬性 **ActivationStatus** 時，必須以下列方式參考它：

  ```
  extension_ActivationStatus in the custom policy.
  extension_<app-guid>_ActivationStatus via Graph API.
  ```


## <a name="reference"></a>參考

如需擴充屬性的詳細資訊，請參閱[目錄結構描述擴充 | 圖形 API 概念](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions) \(機器翻譯\) 一文。

> [!NOTE]
> * **TechnicalProfile** 是一個元素類型或函式，其會定義端點的名稱、中繼資料及通訊協定。 **TechnicalProfile** 會詳述識別體驗架構執行的宣告交換。 當這個函式在協調流程步驟中或從另一個 **TechnicalProfile** 被呼叫時，呼叫端會提供 **InputClaims** 和 **OutputClaims** 作為參數。  
> * 圖形 API 中的擴充屬性會使用 `extension_ApplicationObjectID_attributename` 慣例來命名。  
> * 自訂原則會以 **extension_attributename** 的名稱參考擴充屬性。 此參考在 XML 中會省略 **ApplicationObjectId**。
