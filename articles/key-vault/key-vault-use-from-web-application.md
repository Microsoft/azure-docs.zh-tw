---
title: 從 Web 應用程式使用 Azure 金鑰保存庫的教學課程 | Microsoft Docs
description: 使用本教學課程來幫助您了解如何從 Web 應用程式使用 Azure 金鑰保存庫。
services: key-vault
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 9b7d065e-1979-4397-8298-eeba3aec4792
ms.service: key-vault
ms.workload: identity
ms.topic: tutorial
ms.date: 10/09/2018
ms.author: barclayn
ms.openlocfilehash: b66c9912ba0b6508c2beb786d2327efa779c6645
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49079458"
---
# <a name="tutorial-use-azure-key-vault-from-a-web-application"></a>教學課程：從 Web 應用程式使用 Azure Key Vault

使用此教學課程來幫助您了解如何從 Azure 中的 Web 應用程式使用 Azure 金鑰保存庫。 其中示範從 Azure Key Vault 存取祕密以便在 Web 應用程式中使用的程序。 本教學課程接著會以此程序為基礎，並且使用憑證，而不是用戶端祕密。 本教學課程是針對 Web 開發人員所設計，這些開發人員必須了解在 Azure 上建立 Web 應用程式的基本概念。

在本教學課程中，您了解如何： 

> [!div class="checklist"]
> * 將應用程式設定新增至 web.config 檔案
> * 新增方法以取得存取權杖
> * 在應用程式啟動時擷取權杖
> * 使用憑證進行驗證

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須具備下列項目：

* Azure 金鑰保存庫中密碼的 URI
* 已在 Azure Active Directory 註冊，且有權存取您金鑰保存庫之 Web 應用程式的用戶端識別碼和用戶端密碼
* Web 應用程式。 本教學課程說明在 Azure 中將 ASP.NET MVC 應用程式部署為 Web 應用程式的步驟。

完成[開始使用 Azure Key Vault](key-vault-get-started.md)中的步驟來取得祕密、用戶端識別碼、用戶端祕密的 URI，並註冊應用程式。 Web 應用程式會存取保存庫，而且必須在 Azure Active Directory 中註冊。 它還必須具有 Key Vault 的存取權限。 如果並非如此，請回到開始使用教學課程中的註冊應用程式，並重複列出的步驟。 如需有關建立 Azure Web Apps 的詳細資訊，請參閱 [Web Apps 概觀](../app-service/app-service-web-overview.md)。

此範例取決於手動佈建 Azure Active Directory 身分識別。 但您應該改用 [Azure 資源的受控識別](../active-directory/managed-identities-azure-resources/overview.md)，它會自動佈建 Azure AD 身分識別。 如需詳細資訊，請參閱 [GitHub](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet/) 上的範例以及相關的 [App Service 和 Functions 教學課程](https://docs.microsoft.com/azure/app-service/app-service-managed-service-identity)。 您也可以查看 Key Vault 特定的[設定 Azure Web 應用程式以從 Key Vault 讀取祕密的教學課程](tutorial-web-application-keyvault.md)。

## <a id="packages"></a>新增 NuGet 套件

有二個 Web 應用程式必須已安裝的封裝。

* Active Directory 驗證程式庫 - 具有與 Azure Active Directory 互動及管理使用者身分識別的方法
* Azure 金鑰保存庫資源庫 - 具有與 Azure 金鑰保存庫互動的方法

您可以使用 Package Manager Console 的 Install-Package 命令安裝這二個封裝。

```powershell
Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory 
Install-Package Microsoft.Azure.KeyVault
```

## <a id="webconfig"></a>修改 web.config

有三個必須加入至 web.config 檔案的應用程式設定，如下所示。 我們將在 Azure 入口網站中新增實際值來額外增加一層安全性。

```xml
    <!-- ClientId and ClientSecret refer to the web application registration with Azure Active Directory -->
    <add key="ClientId" value="clientid" />
    <add key="ClientSecret" value="clientsecret" />

    <!-- SecretUri is the URI for the secret in Azure Key Vault -->
    <add key="SecretUri" value="secreturi" />
    <!-- If you aren't hosting your app as an Azure Web App, then you should use the actual ClientId, Client Secret, and Secret URI values -->
```



## <a id="gettoken"></a>新增方法以取得存取權杖

若要使用 Key Vault API，您需要存取權杖。 金鑰保存庫用戶端會處理金鑰保存庫 API 的呼叫，但是您必須提供具有取得存取權杖的函式。 以下範例是從 Azure Active Directory 取得存取權杖的程式碼。 此程式碼可以放置在應用程式的任何位置。 我想要新增 Utils 或 EncryptionHelper 類別。  

```cs
//add these using statements
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Threading.Tasks;
using System.Web.Configuration;

//this is an optional property to hold the secret after it is retrieved
public static string EncryptSecret { get; set; }

//the method that will be provided to the KeyVaultClient
public static async Task<string> GetToken(string authority, string resource, string scope)
{
    var authContext = new AuthenticationContext(authority);
    ClientCredential clientCred = new ClientCredential(WebConfigurationManager.AppSettings["ClientId"],
                WebConfigurationManager.AppSettings["ClientSecret"]);
    AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

    if (result == null)
        throw new InvalidOperationException("Failed to obtain the JWT token");

    return result.AccessToken;
}
// Using Client ID and Client Secret is a way to authenticate an Azure AD application.
// Using it in your web application allows for a separation of duties and more control over your key management. 
// However, it does rely on putting the Client Secret in your configuration settings.
// For some people, this can be as risky as putting the secret in your configuration settings.
```

## <a id="appstart"></a>在應用程式啟動時擷取密碼

現在我們需要呼叫金鑰保存庫 API，並擷取密碼的程式碼。 下列程式碼可以放置在任何位置，只要在您需要使用它之前呼叫即可。 我已將程式碼放入 Global.asax 的應用程式啟動事件中，因此它在啟動時會執行一次，並讓祕密可在應用程式中使用。

```cs
//add these using statements
using Microsoft.Azure.KeyVault;
using System.Web.Configuration;

// I put my GetToken method in a Utils class. Change for wherever you placed your method.
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(Utils.GetToken));
var sec = await kv.GetSecretAsync(WebConfigurationManager.AppSettings["SecretUri"]);

//I put a variable in a Utils class to hold the secret for general application use.
Utils.EncryptSecret = sec.Value;
```

## <a id="portalsettings"></a>在 Azure 入口網站中新增應用程式設定

在 Azure Web 應用程式中，您現在可以在 Azure 入口網站中為 AppSettings 新增實際值。 執行此步驟，實際值將不會存在於 web.config 中，但會透過您有個別存取控制功能的入口網站受到保護。 這些值會被您在 web.config 中輸入的值取代。請確定名稱都相同。

![Azure 入口網站中顯示的應用程式設定][1]

## <a name="authenticate-with-a-certificate-instead-of-a-client-secret"></a>使用憑證 (而非用戶端祕密) 進行驗證

您現在了解如何使用用戶端識別碼和憑證來驗證 Azure AD 應用程式，讓我們開始使用用戶端識別碼和憑證。 若要在 Azure Web 應用程式中使用憑證，請使用下列步驟：

1. 取得或建立憑證
2. 將憑證與 Azure AD 應用程式產生關聯
3. 將程式碼新增至 Web 應用程式以使用憑證
4. 將憑證新增至 Web 應用程式

### <a name="get-or-create-a-certificate"></a>取得或建立憑證

 我們將為本教學課程製作測試憑證。 以下指令碼可建立自我簽署憑證。 變更您要建立憑證檔案的目錄位置。  對於憑證的開始和結束日期，您可以使用目前日期加上一年。

```powershell
#Create self-signed certificate and export pfx and cer files 
$PfxFilePath = 'KVWebApp.pfx'
$CerFilePath = 'KVWebApp.cer'
$DNSName = 'MyComputer.Contoso.com'
$Password = 'MyPassword"'

$StoreLocation = 'CurrentUser' #be aware that LocalMachine requires elevated privileges
$CertBeginDate = Get-Date
$CertExpiryDate = $CertBeginDate.AddYears(1)

$SecStringPw = ConvertTo-SecureString -String $Password -Force -AsPlainText 
$Cert = New-SelfSignedCertificate -DnsName $DNSName -CertStoreLocation "cert:\$StoreLocation\My" -NotBefore $CertBeginDate -NotAfter $CertExpiryDate -KeySpec Signature
Export-PfxCertificate -cert $Cert -FilePath $PFXFilePath -Password $SecStringPw 
Export-Certificate -cert $Cert -FilePath $CerFilePath 
```

記下 .pfx 的結束日期和密碼 (在此範例中為：2019 年 5 月 15 日和 MyPassword)。 您在下面的指令碼中會需要這些資訊。 
### <a name="associate-the-certificate-with-an-azure-ad-application"></a>將憑證與 Azure AD 應用程式產生關聯

既然您已經有憑證，現在就必須讓憑證與 Azure AD 應用程式產生關聯。 關聯可以透過 PowerShell 來完成。 請執行下列命令，將憑證關聯至 Azure AD 應用程式：

```powershell
$x509 = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$x509.Import("C:\data\KVWebApp.cer")
$credValue = [System.Convert]::ToBase64String($x509.GetRawCertData())


$adapp = New-AzureRmADApplication -DisplayName "KVWebApp" -HomePage "http://kvwebapp" -IdentifierUris "http://kvwebapp" -CertValue $credValue -StartDate $x509.NotBefore -EndDate $x509.NotAfter


$sp = New-AzureRmADServicePrincipal -ApplicationId $adapp.ApplicationId


Set-AzureRmKeyVaultAccessPolicy -VaultName 'contosokv' -ServicePrincipalName "http://kvwebapp" -PermissionsToSecrets get,list,set,delete,backup,restore,recover,purge -ResourceGroupName 'contosorg'

# get the thumbprint to use in your app settings
$x509.Thumbprint
```

執行這些命令之後，您就可以在 Azure AD 中看到應用程式。 當您搜尋應用程式註冊時，請務必在搜尋對話方塊中選取 [我的應用程式]，而不是 [所有應用程式]。 

### <a name="add-code-to-your-web-app-to-use-the-certificate"></a>將程式碼新增至 Web 應用程式以使用憑證

現在，我們將把程式碼新增到您的 Web 應用程式以存取憑證，並使用它來進行驗證。 

首先，使用程式碼存取憑證。請注意，StoreLocation 是 CurrentUser，而不是 LocalMachine。 另外，由於我們使用測試憑證，因此我們必須提供 "false" 憑證給 Find 方法。

```cs
//Add this using statement
using System.Security.Cryptography.X509Certificates;  

public static class CertificateHelper
{
    public static X509Certificate2 FindCertificateByThumbprint(string findValue)
    {
        X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
        try
        {
            store.Open(OpenFlags.ReadOnly);
            X509Certificate2Collection col = store.Certificates.Find(X509FindType.FindByThumbprint,
                findValue, false); // Don't validate certs, since the test root isn't installed.
            if (col == null || col.Count == 0)
                return null;
            return col[0];
        }
        finally
        {
            store.Close();
        }
    }
}
```



接著是使用 CertificateHelper 並建立 ClientAssertionCertificate 以便進行驗證。

```cs
public static ClientAssertionCertificate AssertionCert { get; set; }

public static void GetCert()
{
    var clientAssertionCertPfx = CertificateHelper.FindCertificateByThumbprint(WebConfigurationManager.AppSettings["thumbprint"]);
    AssertionCert = new ClientAssertionCertificate(WebConfigurationManager.AppSettings["clientid"], clientAssertionCertPfx);
}
```

以下是取得存取 Token 的新程式碼。 此程式碼會取代先前範例中的 GetToken 方法。 為方便起見，我已將其改名。 為了方便使用，我已將所有此類程式碼放入我的 Web 應用程式專案的公用程式類別。

```cs
public static async Task<string> GetAccessToken(string authority, string resource, string scope)
{
    var context = new AuthenticationContext(authority, TokenCache.DefaultShared);
    var result = await context.AcquireTokenAsync(resource, AssertionCert);
    return result.AccessToken;
}
```



最後的程式碼變更位在 Application_Start 方法中。 首先，我們需要呼叫 GetCert() 方法以載入 ClientAssertionCertificate。 接著，在建立新的 KeyVaultClient 時，變更我們所提供的回呼方法。 此程式碼會取代先前範例中的程式碼。

```cs
Utils.GetCert();
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(Utils.GetAccessToken));
```

### <a name="add-a-certificate-to-your-web-app-through-the-azure-portal"></a>透過 Azure 入口網站將憑證新增到您的 Web 應用程式

將憑證新增到您的 Web 應用程式很簡單，只需兩個步驟。 首先，請移至 Azure 入口網站並瀏覽至您的 Web 應用程式。 在 Web 應用程式的 [設定] 上，按一下 [SSL 設定] 的項目。 開啟時，請上傳您在前一個範例中建立的憑證 (KVWebApp.pfx)。 請確定您記得 pfx 的密碼。

![在 Azure 入口網站中將憑證新增到 Web 應用程式][2]

您需要做的最後一件事，就是將應用程式設定新增至名為WEBSITE\_LOAD\_CERTIFICATES 與值為 * 的 Web 應用程式。 這個步驟可確保載入所有憑證。 如果您只想載入已上傳的憑證，則可輸入其憑證指紋的逗號分隔清單。


## <a name="clean-up-resources"></a>清除資源
若不再需要，請刪除您用於本教學課程的應用程式服務、金鑰保存庫及 Azure AD 應用程式。  


## <a id="next"></a>接續步驟
> [!div class="nextstepaction"]
>[Azure Key Vault 管理 API 參考](/dotnet/api/overview/azure/keyvault/management)。


<!--Image references-->
[1]: ./media/key-vault-use-from-web-application/PortalAppSettings.png
[2]: ./media/key-vault-use-from-web-application/PortalAddCertificate.png
 