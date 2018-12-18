
1. 在 MainPage.xaml.cs 專案檔中，新增下列 **using** 陳述式：
   
        using System.Linq;        
        using Windows.Security.Credentials;
2. 使用下列程式碼來取代 **AuthenticateAsync** 方法：
   
        private async System.Threading.Tasks.Task<bool> AuthenticateAsync()
        {
            string message;
            bool success = false;
   
            // This sample uses the Facebook provider.
            var provider = MobileServiceAuthenticationProvider.Facebook;
   
            // Use the PasswordVault to securely store and access credentials.
            PasswordVault vault = new PasswordVault();
            PasswordCredential credential = null;
   
            try
            {
                // Try to get an existing credential from the vault.
                credential = vault.FindAllByResource(provider.ToString()).FirstOrDefault();
            }
            catch (Exception)
            {
                // When there is no matching resource an error occurs, which we ignore.
            }
   
            if (credential != null)
            {
                // Create a user from the stored credentials.
                user = new MobileServiceUser(credential.UserName);
                credential.RetrievePassword();
                user.MobileServiceAuthenticationToken = credential.Password;
   
                // Set the user from the stored credentials.
                App.MobileService.CurrentUser = user;
   
                // Consider adding a check to determine if the token is 
                // expired, as shown in this post: http://aka.ms/jww5vp.
   
                success = true;
                message = string.Format("Cached credentials for user - {0}", user.UserId);
            }
            else
            {
                try
                {
                    // Sign in with the identity provider.
                    user = await App.MobileService
                        .LoginAsync(provider, "{url_scheme_of_your_app}");
   
                    // Create and store the user credentials.
                    credential = new PasswordCredential(provider.ToString(),
                        user.UserId, user.MobileServiceAuthenticationToken);
                    vault.Add(credential);
   
                    success = true;
                    message = string.Format("You are now signed in - {0}", user.UserId);
                }
                catch (MobileServiceInvalidOperationException)
                {
                    message = "You must sign in. Sign-In Required";
                }
            }
   
            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
   
            return success;
        }
   
    在這個版本的 **AuthenticateAsync** 中，應用程式會嘗試使用已儲存於 **PasswordVault** 中的認證來存取服務。 如果沒有儲存任何認證，也會執行一般登入。
   
   > [!NOTE]
   > 快取權杖可能會過期，且權杖也可能會在應用程式使用期間經驗證之後到期。 若要瞭解如何判斷權杖是否過期，請參閱 [檢查是否有過期的驗證權杖](http://aka.ms/jww5vp)(英文)。 如需處理與權杖到期相關之授權錯誤的方案，請參閱下列文章： [在 Azure 行動服務管理的 SDK 中快取和處理到期的權杖](http://blogs.msdn.com/b/carlosfigueira/archive/2014/03/13/caching-and-handling-expired-tokens-in-azure-mobile-services-managed-sdk.aspx)(英文)。 
   > 
   > 
3. 重新啟動應用程式兩次。
   
    請注意，第一次啟動時，需要再次使用該提供者登入。 不過，在第二次重新啟動時，可以使用快取的認證，並略過登入。 

