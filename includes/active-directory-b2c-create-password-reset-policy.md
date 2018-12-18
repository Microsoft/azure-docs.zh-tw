若要在您的應用程式上啟用更細緻的密碼重設，您可使用**密碼重設**原則。 請注意，[這裡](../articles/active-directory-b2c/active-directory-b2c-reference-sspr.md)包含租用戶密碼重設的所有選項。 此原則描述客戶在密碼重設期間將經歷的體驗，以及成功完成時，應用程式將收到的權杖內容。

[!INCLUDE [active-directory-b2c-portal-navigate-b2c-service](active-directory-b2c-portal-navigate-b2c-service.md)]

在設定的原則區段中，選取 [密碼重設原則]，然後按一下 [+ 新增]。

![選取註冊或登入原則，然後按一下 [新增] 按鈕](media/active-directory-b2c-create-password-reset-policy/add-b2c-password-reset-policy.png)

輸入原則 [名稱]，以供您的應用程式參考。 例如，輸入 `SSPR`。

選取 [識別提供者]，並勾選 [使用電子郵件地址重設密碼]。 按一下 [確定]。

![以電子郵件地址作為識別提供者來選取重設密碼，然後按一下 [確定] 按鈕](media/active-directory-b2c-create-password-reset-policy/add-b2c-password-reset-identity-providers.png)

選取 [應用程式宣告]。 選擇在成功進行密碼重設之後，您要在授權權杖中傳回給應用程式的宣告。 例如，選取 [使用者的物件識別碼]。

![選取某些應用程式宣告，然後按一下 [確定] 按鈕](media/active-directory-b2c-create-password-reset-policy/add-b2c-password-reset-application-claims.png)

按一下 [建立] 以新增原則。 此原則列示為 **B2C_1_SSPR**。 名稱會附加 **B2C_1_** 前置詞。

選取 **B2C_1_SSPR** 以開啟原則。 驗證資料表中指定的設定，然後按一下 [立刻執行]。

![選取原則並加以執行](media/active-directory-b2c-create-password-reset-policy/run-b2c-password-reset-policy.png)

| 設定      | 值  |
| ------------ | ------ |
| **應用程式** | Contoso B2C 應用程式 |
| **選取回覆 URL** | `https://localhost:44316/` |

新的瀏覽器索引標籤隨即開啟，您可以在應用程式中驗證密碼重設取用者體驗。

> [!NOTE]
> 建立和更新原則後，需要經過一分鐘才會生效。
>
