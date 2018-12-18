[!INCLUDE [active-directory-b2c-portal-navigate-b2c-service](active-directory-b2c-portal-navigate-b2c-service.md)]

在設定的原則區段中，選取 [註冊或登入原則]，然後按一下 [+ 新增]。

![選取註冊或登入原則，然後按一下 [新增] 按鈕](media/active-directory-b2c-create-sign-in-sign-up-policy/add-b2c-signup-signin-policy.png)

輸入原則 [名稱]，以供您的應用程式參考。 例如，輸入 `SiUpIn`。

選取 [識別提供者]，然後勾選 [電子郵件註冊]。 (選擇性) 您也可以選取社交身分識別提供者 (如果已經設定)。 按一下 [確定]。

![以識別提供者身分來選取 [電子郵件註冊]，然後按一下 [確定] 按鈕](media/active-directory-b2c-create-sign-in-sign-up-policy/add-b2c-signup-signin-identity-providers.png)

選取 [註冊屬性]。 選擇註冊期間要向取用者收集的屬性。 例如，勾選 [國家/區域]、[顯示名稱] 和 [郵遞區號]。 按一下 [確定]。

![選取某些屬性，然後按一下 [確定] 按鈕](media/active-directory-b2c-create-sign-in-sign-up-policy/add-b2c-signup-signin-sign-up-attributes.png)

選取 [應用程式宣告]。 選擇成功註冊或登入後，您要在授權權杖中傳回給應用程式的宣告。 例如，選取 [顯示名稱]、[識別提供者]、[郵遞區號]、[使用者是新的] 和 [使用者的物件識別碼]。

![選取某些應用程式宣告，然後按一下 [確定] 按鈕](media/active-directory-b2c-create-sign-in-sign-up-policy/add-b2c-signup-signin-application-claims.png)

按一下 [建立] 以新增原則。 此原則列示為 **B2C_1_SiUpIn**。 名稱會附加 **B2C_1_** 前置詞。

選取 [B2C_1_SiUpIn] 以開啟原則。 驗證資料表中指定的設定，然後按一下 [立刻執行]。

![選取原則並加以執行](media/active-directory-b2c-create-sign-in-sign-up-policy/run-b2c-signup-signin-policy.png)

| 設定      | 值  |
| ------------ | ------ |
| **應用程式** | Contoso B2C 應用程式 |
| **選取回覆 URL** | `https://localhost:44316/` |

新的瀏覽器索引標籤隨即開啟，您可以依照設定驗證註冊或登入取用者體驗。

> [!NOTE]
> 建立和更新原則後，需要經過一分鐘才會生效。
>