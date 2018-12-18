
先前範例所示範的標準登入，在每次應用程式啟動時，皆需要用戶端連絡身分識別提供者和後端 Azure 服務。 此方法效率低，而且如果有許多客戶嘗試同時啟動應用程式，則可能會發生與使用量相關的問題。 更好的方法就是快取 Azure 服務傳回的授權權杖，然後嘗試在使用提供者形式登入前先使用此方法。

> [!NOTE]
> 無論您使用用戶端管理或服務管理驗證，皆可以快取後端 Azure 服務發行的權杖。 本教學課程使用服務受控驗證。
>
>

1. 開啟 ToDoActivity.java 檔案，並新增下列 import 陳述式：

    ```java
    import android.content.Context;
    import android.content.SharedPreferences;
    import android.content.SharedPreferences.Editor;
    ```

2. 在 `ToDoActivity` 類別中新增下列成員：

    ```java
    public static final String SHAREDPREFFILE = "temp";
    public static final String USERIDPREF = "uid";
    public static final String TOKENPREF = "tkn";
    ```

3. 在 ToDoActivity.java 檔案中，新增下列 `cacheUserToken` 方法的定義。

    ```java
    private void cacheUserToken(MobileServiceUser user)
    {
        SharedPreferences prefs = getSharedPreferences(SHAREDPREFFILE, Context.MODE_PRIVATE);
        Editor editor = prefs.edit();
        editor.putString(USERIDPREF, user.getUserId());
        editor.putString(TOKENPREF, user.getAuthenticationToken());
        editor.commit();
    }
    ```

    此方法將使用者識別碼和權杖儲存在標示為私人的慣用檔案中。 這可以保護快取的存取，如此一來裝置上的其他應用程式將無法存取權杖。 應用程式的喜好設定已沙箱化。 不過，如果有人取得裝置的存取權，他們有可能也可以透過其他方法取得權杖快取的存取權。

   > [!NOTE]
   > 如果使用權杖存取的資料十分敏感，而且有人可能可以取得裝置存取權，那麼您可以使用加密來進一步保護權杖。 不過，完整的安全解決方案已超過本教學課程範圍，而且也須視您的安全性需求而定。
   >
   >

4. 在 ToDoActivity.java 檔案中，新增下列 `loadUserTokenCache` 方法的定義。

    ```java
    private boolean loadUserTokenCache(MobileServiceClient client)
    {
        SharedPreferences prefs = getSharedPreferences(SHAREDPREFFILE, Context.MODE_PRIVATE);
        String userId = prefs.getString(USERIDPREF, null);
        if (userId == null)
            return false;
        String token = prefs.getString(TOKENPREF, null);
        if (token == null)
            return false;

        MobileServiceUser user = new MobileServiceUser(userId);
        user.setAuthenticationToken(token);
        client.setCurrentUser(user);

        return true;
    }
    ```

5. 在 *ToDoActivity.java* 檔案中，將 `authenticate` 和 `onActivityResult` 方法以使用權杖快取的下列各項取代。 如果您想要使用 Google 以外的帳戶，請變更登入提供者。

    ```java
    private void authenticate() {
        // We first try to load a token cache if one exists.
        if (loadUserTokenCache(mClient))
        {
            createTable();
        }
        // If we failed to load a token cache, sign in and create a token cache
        else
        {
            // Sign in using the Google provider.
            mClient.login(MobileServiceAuthenticationProvider.Google, "{url_scheme_of_your_app}", GOOGLE_LOGIN_REQUEST_CODE);
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        // When request completes
        if (resultCode == RESULT_OK) {
            // Check the request code matches the one we send in the sign-in request
            if (requestCode == GOOGLE_LOGIN_REQUEST_CODE) {
                MobileServiceActivityResult result = mClient.onActivityResult(data);
                if (result.isLoggedIn()) {
                    // sign-in succeeded
                    createAndShowDialog(String.format("You are now signed in - %1$2s", mClient.getCurrentUser().getUserId()), "Success");
                    cacheUserToken(mClient.getCurrentUser());
                    createTable();
                } else {
                    // sign-in failed, check the error message
                    String errorMessage = result.getErrorMessage();
                    createAndShowDialog(errorMessage, "Error");
                }
            }
        }
    }
    ```

6. 使用有效的帳戶建置應用程式，並且測試驗證。 至少執行 2 次此動作。 第一次執行時，您應該會收到提示，要求您登入並建立權杖快取。 之後，每次執行會嘗試載入權杖快取來進行驗證。 您應該不需要登入。
