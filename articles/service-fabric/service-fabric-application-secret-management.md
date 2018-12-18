---
title: 管理 Azure Service Fabric 應用程式密碼 | Microsoft Docs
description: 瞭解如何保護在 Service Fabric 應用程式中的密碼值。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 94a67e45-7094-4fbd-9c88-51f4fc3c523a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/21/2018
ms.author: vturecek
ms.openlocfilehash: 85eb1cd40986bd6fb83c80a274046bbae3756b7e
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36295448"
---
# <a name="manage-secrets-in-service-fabric-applications"></a>管理 Service Fabric 應用程式中的祕密
本指南將逐步引導您完成管理 Service Fabric 應用程式中密碼的步驟。 密碼可以是任何機密資訊，例如儲存體連接字串、密碼或其他不會以純文字處理的值。

[Azure Key Vault][key-vault-get-started] 在此是當做憑證的安全儲存位置，以及讓憑證安裝在 Azure 中的 Service Fabric 叢集上的方法。 如果您沒有要部署至 Azure，您不需要使用金鑰保存庫管理 Service Fabric 應用程式中的密碼。 不過，應用程式中的密碼「使用」  是由平台驗證，讓應用程式可部署至裝載在任何位置的叢集。 

## <a name="obtain-a-data-encipherment-certificate"></a>取得資料加密憑證
資料編密憑證只會用於服務的 Settings.xml 中組態值的加密與解密，並無法用來驗證或簽署密碼文字。 憑證必須符合下列要求：

* 憑證必須包含私密金鑰。
* 憑證必須是為了進行金鑰交換而建立，且可匯出成個人資訊交換檔 (.pfx)。
* 憑證的金鑰使用法必須包含資料編密 (10)，而且不應該包含伺服器驗證或用戶端驗證。 
  
  例如，當使用 PowerShell 建立自我簽署的憑證時，`KeyUsage` 旗標必須設定為 `DataEncipherment`：
  
  ```powershell
  New-SelfSignedCertificate -Type DocumentEncryptionCert -KeyUsage DataEncipherment -Subject mydataenciphermentcert -Provider 'Microsoft Enhanced Cryptographic Provider v1.0'
  ```

## <a name="install-the-certificate-in-your-cluster"></a>在叢集中安裝憑證
此憑證必須安裝在叢集中的每個節點上。 它會用在執行階段來解密服務的 Settings.xml 中儲存的值。 請參閱[如何使用 Azure Resource Manager][service-fabric-cluster-creation-via-arm] 建立叢集的安裝指示。 

## <a name="encrypt-application-secrets"></a>加密應用程式密碼
在部署應用程式時，以憑證來加密密碼的值，並將其插入服務的 Settings.xml 組態檔。 Service Fabric SDK 有內建密碼加密和解密函式。 可以在建置階段加密密碼值，然後在服務代碼中以程式設計方式解密和讀取。 

下列 PowerShell 命令會用來加密密碼。 此命令只會將值加密；並**不會**簽署密碼文字。 您必須使用安裝在叢集中相同的編密憑證，以產生密碼值的加密文字︰

```powershell
Invoke-ServiceFabricEncryptText -CertStore -CertThumbprint "<thumbprint>" -Text "mysecret" -StoreLocation CurrentUser -StoreName My
```

產生的 Base-64 編碼字串同時包含密碼的加密文字，以及用來對其加密的憑證相關資訊。  當 `IsEncrypted` 屬性設為 `true` 時，Base-64 編碼的字串可插入到服務的 Settings.xml 組態檔中的參數內：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MySettings">
    <Parameter Name="MySecret" IsEncrypted="true" Value="I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=" />
  </Section>
</Settings>
```

### <a name="inject-application-secrets-into-application-instances"></a>將應用程式密碼插入應用程式執行個體內
在理想情況下，部署至不同的環境應儘可能自動化。 這可以藉由在建置環境中執行密碼加密，並在建立應用程式執行個體時提供加密的密碼做為參數來實現。

#### <a name="use-overridable-parameters-in-settingsxml"></a>在 Settings.xml 中使用可覆寫參數
Settings.xml 組態檔允許可以在應用程式建立時提供的可覆寫參數。 使用 `MustOverride` 屬性而非提供參數的值︰

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MySettings">
    <Parameter Name="MySecret" IsEncrypted="true" Value="" MustOverride="true" />
  </Section>
</Settings>
```

若要覆寫 Settings.xml 中的值，請宣告 ApplicationManifest.xml 中服務的覆寫參數︰

```xml
<ApplicationManifest ... >
  <Parameters>
    <Parameter Name="MySecret" DefaultValue="" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides>
      <ConfigOverride Name="Config">
        <Settings>
          <Section Name="MySettings">
            <Parameter Name="MySecret" Value="[MySecret]" IsEncrypted="true" />
          </Section>
        </Settings>
      </ConfigOverride>
    </ConfigOverrides>
  </ServiceManifestImport>
 ```

現在可以在建立應用程式執行個體時，將值指定為「應用程式參數」  。 可以使用 PowerShell 編寫指令碼 (或以 C# 撰寫) 來建立應用程式執行個體，使其在建置流程中很容易整合。

若使用 PowerShell，則參數會提供給 `New-ServiceFabricApplication` 命令當做 [雜湊表](https://technet.microsoft.com/library/ee692803.aspx)：

```powershell
PS C:\Users\vturecek> New-ServiceFabricApplication -ApplicationName fabric:/MyApp -ApplicationTypeName MyAppType -ApplicationTypeVersion 1.0.0 -ApplicationParameter @{"MySecret" = "I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM="}
```

若使用 C#，則應用程式參數在 `ApplicationDescription` 中會指定為 `NameValueCollection`：

```csharp
FabricClient fabricClient = new FabricClient();

NameValueCollection applicationParameters = new NameValueCollection();
applicationParameters["MySecret"] = "I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=";

ApplicationDescription applicationDescription = new ApplicationDescription(
    applicationName: new Uri("fabric:/MyApp"),
    applicationTypeName: "MyAppType",
    applicationTypeVersion: "1.0.0",
    applicationParameters: applicationParameters)
);

await fabricClient.ApplicationManager.CreateApplicationAsync(applicationDescription);
```

## <a name="decrypt-secrets-from-service-code"></a>解密來自服務代碼的密碼
您可以藉由將密碼編碼的編密憑證進行解密，從 Settings.xml 讀取加密的值。 Service Fabric 中的服務在「網路服務」下依預設是在 Windows 上執行，而且若沒有額外設定，則沒有安裝在節點上憑證的存取權。

使用資料編密憑證時，您必須確定「網路服務」或服務在其下執行的任何使用者帳戶，可以存取憑證的私密金鑰。 若您如此設定，Service Fabric 會自動處理授與服務的存取權。 可以藉由定義使用者及憑證的安全性原則，在 ApplicationManifest.xml 中完成此組態。 在下列範例中，已授與「網路服務」帳戶由其憑證指紋所定義的憑證讀取權限︰

```xml
<ApplicationManifest … >
    <Principals>
        <Users>
            <User Name="Service1" AccountType="NetworkService" />
        </Users>
    </Principals>
  <Policies>
    <SecurityAccessPolicies>
      <SecurityAccessPolicy GrantRights=”Read” PrincipalRef="Service1" ResourceRef="MyCert" ResourceType="Certificate"/>
    </SecurityAccessPolicies>
  </Policies>
  <Certificates>
    <SecretsCertificate Name="MyCert" X509FindType="FindByThumbprint" X509FindValue="[YourCertThumbrint]"/>
  </Certificates>
</ApplicationManifest>
```

> [!NOTE]
> 當從 Windows 上的憑證存放區嵌入式管理單元中複製憑證指紋時，在憑證指紋字串的開頭會放置不可見的字元。 嘗試按憑證指紋尋找憑證時，這個不可見的字元可能會導致錯誤，因此請務必刪除這個額外的字元。
> 
> 

### <a name="use-application-secrets-in-service-code"></a>在服務代碼中使用應用程式密碼
存取來自組態套件中 Settings.xml 組態值的 API，可輕鬆解密將 `IsEncrypted` 屬性設為 `true` 的值。 由於加密的文字包含用於加密的憑證相關資訊，因此您不需要手動尋找憑證。 只需要在執行服務的節點上安裝憑證。 只要呼叫 `DecryptValue()` 方法來擷取原始的密碼值︰

```csharp
ConfigurationPackage configPackage = this.Context.CodePackageActivationContext.GetConfigurationPackageObject("Config");
SecureString mySecretValue = configPackage.Settings.Sections["MySettings"].Parameters["MySecret"].DecryptValue()
```

## <a name="next-steps"></a>後續步驟
深入了解[應用程式及服務安全性](service-fabric-application-and-service-security.md)

<!-- Links -->
[key-vault-get-started]:../key-vault/key-vault-get-started.md
[config-package]: service-fabric-application-and-service-manifests.md
[service-fabric-cluster-creation-via-arm]: service-fabric-cluster-creation-via-arm.md

<!-- Images -->
[overview]:./media/service-fabric-application-secret-management/overview.png
