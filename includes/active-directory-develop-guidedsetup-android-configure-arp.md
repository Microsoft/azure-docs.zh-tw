## <a name="add-the-applications-registration-information-to-your-app"></a>將應用程式的註冊資訊新增到您的應用程式

在這個步驟中，您需要將用戶端識別碼新增到您的專案。

1.  開啟 `MainActivity` (在 `app` > `java` > *`{host}.{namespace}`* 底下)
2.  將開頭為 `final static String CLIENT_ID` 的那一行取代為：
```java
final static String CLIENT_ID = "[Enter the application Id here]";
```
3. 開啟：`app` > `manifests` > `AndroidManifest.xml`
4. 將下列活動新增至 `manifest\application` 節點。 這會註冊 `BrowserTabActivity`，以允許 OS 在完成驗證之後繼續執行您的應用程式：

```xml
<!--Intent filter to capture System Browser calling back to our app after Sign In-->
<activity
    android:name="com.microsoft.identity.client.BrowserTabActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <!--Add in your scheme/host from registered redirect URI-->
        <!--By default, the scheme should be similar to 'msal[appId]' -->
        <data android:scheme="msal[Enter the application Id here]"
            android:host="auth" />
    </intent-filter>
</activity>
```

