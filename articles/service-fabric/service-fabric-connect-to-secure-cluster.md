---
title: 以安全的方式連線到 Azure Service Fabric 叢集 | Microsoft Docs
description: 說明如何驗證用戶端對 Service Fabric 叢集的存取，以及如何保護用戶端與叢集之間的通訊。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: 759a539e-e5e6-4055-bff5-d38804656e10
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/18/2018
ms.author: ryanwi
ms.openlocfilehash: f2a181fbae8ab1e08669021c42c5b4be08f66172
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34364806"
---
# <a name="connect-to-a-secure-cluster"></a>連線到安全的叢集

當用戶端連線到 Service Fabric 叢集節點時，用戶端可以使用憑證安全性或 Azure Active Directory (AAD) 來接受驗證及保護已建立的通訊。 此驗證可確保只有已獲授權的使用者可以存取叢集和已部署的應用程式，以及執行管理工作。  憑證或 AAD 安全性必須在叢集建立之時即事先在叢集上啟用。  如需有關叢集安全性案例的詳細資訊，請參閱 [叢集安全性](service-fabric-cluster-security.md)。 如果您要連接到使用憑證保護的叢集，請在要連接到叢集的電腦上[設定用戶端憑證](service-fabric-connect-to-secure-cluster.md#connectsecureclustersetupclientcert)。 

<a id="connectsecureclustercli"></a> 

## <a name="connect-to-a-secure-cluster-using-azure-service-fabric-cli-sfctl"></a>使用 Azure Service Fabric CLI (sfctl) 連線到安全的叢集

有幾個不同的方式可使用 Service Fabric CLI (sfctl) 來連線到安全的叢集。 使用用戶端憑證來進行驗證時，憑證詳細資料必須與部署至叢集節點的憑證相符。 如果您的憑證有「憑證授權單位」(CA)，就必須額外指定信任的 CA。

您可以使用 `sfctl cluster select` 命令來連線到叢集。

指定用戶端憑證時，可以使用兩種不同的方式，以憑證與金鑰組方式指定，或是以單一 pem 檔案方式指定。 針對受密碼保護的 `pem` 檔案，系統會自動提示您輸入密碼。

若要以 pem 檔案指定用戶端憑證，請在 `--pem` 引數中指定檔案路徑。 例如︰

```azurecli
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem
```

受密碼保護的 pem 檔案會在執行任何命令之前，先提示您輸入密碼。

為了指定憑證，金鑰組會使用 `--cert` 和 `--key` 引數來指定每個個別檔案的檔案路徑。

```azurecli
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --cert ./client.crt --key ./keyfile.key
```

有時，用來保護測試或開發叢集的憑證會讓憑證驗證失敗。 若要略過憑證驗證，請指定 `--no-verify` 選項。 例如︰

> [!WARNING]
> 連線到生產環境 Service Fabric 叢集時，請勿使用 `no-verify` 選項。

```azurecli
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --no-verify
```

此外，您可以指定受信任 CA 憑證，或個別憑證的目錄路徑。 若要指定這些路徑，請使用 `--ca` 引數。 例如︰

```azurecli
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --ca ./trusted_ca
```

連線之後，您應該能夠[執行其他 sfctl 命令](service-fabric-cli.md)來與叢集互動。

<a id="connectsecurecluster"></a>

## <a name="connect-to-a-cluster-using-powershell"></a>使用 PowerShell 來連線到叢集
在您透過 PowerShell 執行叢集上的作業之前，先建立叢集的連接。 叢集連接適用於指定 PowerShell 工作階段中的所有後續命令。

### <a name="connect-to-an-unsecure-cluster"></a>連線到不安全的叢集

若要連接至不安全的叢集，請提供叢集端點位址至 **Connect-ServiceFabricCluster** 命令︰

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 
```

### <a name="connect-to-a-secure-cluster-using-azure-active-directory"></a>使用 Azure Active Directory 連線到安全的叢集

若要連接至使用 Azure Active Directory 來授權叢集系統管理員存取權的安全叢集，請提供叢集憑證指紋，並使用 AzureActiveDirectory 旗標。  

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
-ServerCertThumbprint <Server Certificate Thumbprint> `
-AzureActiveDirectory
```

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>使用用戶端憑證連線到安全的叢集
執行下列 PowerShell 命令來連線至使用用戶端憑證授權系統管理員存取權的安全叢集。 

#### <a name="connect-using-certificate-common-name"></a>使用憑證一般名稱的連線
提供叢集憑證一般名稱，以及已授與權限來管理叢集的用戶端憑證一般名稱。 憑證詳細資料必須與叢集節點上的憑證相符。

```powershell
Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCommonName <certificate common name>  `
    -FindType FindBySubjectName `
    -FindValue <certificate common name> `
    -StoreLocation CurrentUser `
    -StoreName My 
```
*ServerCommonName* 是安裝在叢集節點上的伺服器憑證的一般名稱。 *FindValue* 是系統管理員用戶端憑證的一般名稱。 填入參數後，命令看起來如下列範例所示︰
```powershell
$ClusterName= "sf-commonnametest-scus.southcentralus.cloudapp.azure.com:19000"
$certCN = "sfrpe2eetest.southcentralus.cloudapp.azure.com"

Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCommonName $certCN  `
    -FindType FindBySubjectName `
    -FindValue $certCN `
    -StoreLocation CurrentUser `
    -StoreName My 
```

#### <a name="connect-using-certificate-thumbprint"></a>使用憑證指紋連線
提供叢集憑證指紋，以及已授與權限來管理叢集的用戶端憑證指紋。 憑證詳細資料必須與叢集節點上的憑證相符。

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `  
          -KeepAliveIntervalInSec 10 `  
          -X509Credential -ServerCertThumbprint <Certificate Thumbprint> `  
          -FindType FindByThumbprint -FindValue <Certificate Thumbprint> `  
          -StoreLocation CurrentUser -StoreName My
```

*ServerCertThumbprint* 是安裝在叢集節點上的伺服器憑證指紋。 *FindValue* 是系統管理員用戶端憑證的指紋。  填入參數後，命令看起來如下列範例所示︰

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint clustername.westus.cloudapp.azure.com:19000 `  
          -KeepAliveIntervalInSec 10 `  
          -X509Credential -ServerCertThumbprint A8136758F4AB8962AF2BF3F27921BE1DF67F4326 `  
          -FindType FindByThumbprint -FindValue 71DE04467C9ED0544D021098BCD44C71E183414E `  
          -StoreLocation CurrentUser -StoreName My 
```

### <a name="connect-to-a-secure-cluster-using-windows-active-directory"></a>使用 Windows Active Directory 連線到安全的叢集
如果您的獨立叢集是使用 AD 安全性部署，可加上 "WindowsCredential" 參數來連接到叢集。

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
          -WindowsCredential
```

<a id="connectsecureclusterfabricclient"></a>

## <a name="connect-to-a-cluster-using-the-fabricclient-apis"></a>使用 FabricClient API 來連線到叢集
Service Fabric SDK 會提供叢集管理的 [FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient) 類別。 若要使用 FabricClient API，請取得 Microsoft.ServiceFabric NuGet 封裝。

### <a name="connect-to-an-unsecure-cluster"></a>連線到不安全的叢集

若要連接至遠端不安全的叢集，建立 FabricClient 執行個體，並提供叢集位址︰

```csharp
FabricClient fabricClient = new FabricClient("clustername.westus.cloudapp.azure.com:19000");
```

針對在叢集 (例如，Reliable Service) 內執行的程式碼，建立 FabricClient 且不指定叢集位址。 FabricClient 連接到目前正在執行程式碼之節點上的本機管理閘道，避免額外的網路躍點。

```csharp
FabricClient fabricClient = new FabricClient();
```

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>使用用戶端憑證連線到安全的叢集

叢集中的節點必須具備有效的憑證，這些憑證在 SAN 中的通用名稱或 DNS 名稱會出現在於 [FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient) 上設定的 [RemoteCommonNames](https://docs.microsoft.com/dotnet/api/system.fabric.x509credentials#System_Fabric_X509Credentials_RemoteCommonNames) 屬性中。 遵循此程序，就可讓用戶端與叢集節點之間進行相互驗證。

```csharp
using System.Fabric;
using System.Security.Cryptography.X509Certificates;

string clientCertThumb = "71DE04467C9ED0544D021098BCD44C71E183414E";
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string CommonName = "www.clustername.westus.azure.com";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var xc = GetCredentials(clientCertThumb, serverCertThumb, CommonName);
var fc = new FabricClient(xc, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

static X509Credentials GetCredentials(string clientCertThumb, string serverCertThumb, string name)
{
    X509Credentials xc = new X509Credentials();
    xc.StoreLocation = StoreLocation.CurrentUser;
    xc.StoreName = "My";
    xc.FindType = X509FindType.FindByThumbprint;
    xc.FindValue = clientCertThumb;
    xc.RemoteCommonNames.Add(name);
    xc.RemoteCertThumbprints.Add(serverCertThumb);
    xc.ProtectionLevel = ProtectionLevel.EncryptAndSign;
    return xc;
}
```

### <a name="connect-to-a-secure-cluster-interactively-using-azure-active-directory"></a>使用 Azure Active Directory 以互動方式連線到安全的叢集

以下範例針對用戶端身分識別使用 Azure Active Directory，以及針對伺服器身分識別啟用伺服器憑證。

連線到叢集時，就會自動彈出對話方塊視窗，以供互動式登入。

```csharp
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);

var fc = new FabricClient(claimsCredentials, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}
```

### <a name="connect-to-a-secure-cluster-non-interactively-using-azure-active-directory"></a>使用 Azure Active Directory 以非互動方式連線到安全的叢集

以下範例依存於 Microsoft.IdentityModel.Clients.ActiveDirectory，版本：2.19.208020213。

如需 AAD 權杖取得的詳細資訊，請參閱 [Microsoft.IdentityModel.Clients.ActiveDirectory](https://msdn.microsoft.com/library/microsoft.identitymodel.clients.activedirectory.aspx)。

```csharp
string tenantId = "C15CFCEA-02C1-40DC-8466-FBD0EE0B05D2";
string clientApplicationId = "118473C2-7619-46E3-A8E4-6DA8D5F56E12";
string webApplicationId = "53E6948C-0897-4DA6-B26A-EE2A38A690B4";

string token = GetAccessToken(
    tenantId,
    webApplicationId,
    clientApplicationId,
    "urn:ietf:wg:oauth:2.0:oob");

string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);
claimsCredentials.LocalClaims = token;

var fc = new FabricClient(claimsCredentials, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

...

static string GetAccessToken(
    string tenantId,
    string resource,
    string clientId,
    string redirectUri)
{
    string authorityFormat = @"https://login.microsoftonline.com/{0}";
    string authority = string.Format(CultureInfo.InvariantCulture, authorityFormat, tenantId);
    var authContext = new AuthenticationContext(authority);

    var authResult = authContext.AcquireToken(
        resource,
        clientId,
        new UserCredential("TestAdmin@clustenametenant.onmicrosoft.com", "TestPassword"));
    return authResult.AccessToken;
}

```

### <a name="connect-to-a-secure-cluster-without-prior-metadata-knowledge-using-azure-active-directory"></a>使用 Azure Active Directory 連線到安全的叢集而不需事先知道中繼資料

以下範例使用非互動權杖取得，但可以使用同樣的方法來建立自訂的互動式權杖取得經驗。 權杖取得所需的 Azure Active Directory 中繼資料是從叢集設定來讀取。

```csharp
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);

var fc = new FabricClient(claimsCredentials, connection);

fc.ClaimsRetrieval += (o, e) =>
{
    return GetAccessToken(e.AzureActiveDirectoryMetadata);
};

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

...

static string GetAccessToken(AzureActiveDirectoryMetadata aad)
{
    var authContext = new AuthenticationContext(aad.Authority);

    var authResult = authContext.AcquireToken(
        aad.ClusterApplication,
        aad.ClientApplication,
        new UserCredential("TestAdmin@clustenametenant.onmicrosoft.com", "TestPassword"));
    return authResult.AccessToken;
}

```

<a id="connectsecureclustersfx"></a>

## <a name="connect-to-a-secure-cluster-using-service-fabric-explorer"></a>使用 Service Fabric Explorer 連線到安全的叢集
若要連線到指定之叢集的 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)，請將您的瀏覽器指向：

`http://<your-cluster-endpoint>:19080/Explorer`

Azure 入口網站的叢集基本資訊窗格中也會提供完整 URL。

如需使用瀏覽器連線到 Windows 或 OS X 上的安全叢集，您可以匯入用戶端憑證，然後瀏覽器將會提示您指定要用於連線到叢集所需的憑證。  在 Linux 機器上，則必須使用進階瀏覽器設定 (每種瀏覽器會有不同的機制) 來匯入憑證，並指向磁碟上的憑證位置。

### <a name="connect-to-a-secure-cluster-using-azure-active-directory"></a>使用 Azure Active Directory 連線到安全的叢集

若要連接至使用 AAD 保護的叢集，請將您的瀏覽器指向：

`https://<your-cluster-endpoint>:19080/Explorer`

系統將會自動提示您使用 AAD 登入。

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>使用用戶端憑證連線到安全的叢集

若要連接至使用憑證保護的叢集，請將您的瀏覽器指向：

`https://<your-cluster-endpoint>:19080/Explorer`

系統會自動提示您選取用戶端憑證。

<a id="connectsecureclustersetupclientcert"></a>
## <a name="set-up-a-client-certificate-on-the-remote-computer"></a>設定遠端電腦上的用戶端憑證
至少應使用兩個憑證保護叢集，一個是叢集和伺服器憑證，另一個用於用戶端存取。  建議您也使用額外的次要憑證和用戶端存取憑證。  若要使用憑證安全性來保護用戶端與叢集節點之間的通訊，您必須先取得並安裝用戶端憑證。 此憑證可以安裝到本機電腦或目前使用者的個人 (My) 存放區。  您也需要伺服器憑證的指紋，讓用戶端可以驗證叢集。

請執行下列 PowerShell Cmdlet 以在您存取叢集的電腦上設定用戶端憑證。

```powershell
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
        -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
        -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
```

如果是自我簽署憑證，您必須先把它匯入您電腦的「受信任的人」存放區，才能使用此憑證來連線到安全的叢集。

```powershell
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople `
-FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
-Password (ConvertTo-SecureString -String test -AsPlainText -Force)
```

## <a name="next-steps"></a>後續步驟

* [Service Fabric 叢集升級程序與您的期望](service-fabric-cluster-upgrade.md)
* [在 Visual Studio 中管理 Service Fabric 應用程式](service-fabric-manage-application-in-visual-studio.md)
* [Service Fabric 健康情況模型簡介](service-fabric-health-introduction.md)
* [應用程式安全性及 RunAs](service-fabric-application-runas-security.md)
* [開始使用 Service Fabric CLI](service-fabric-cli.md)
