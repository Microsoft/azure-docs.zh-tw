---
title: 在 Linux 上設定 Azure Service Fabric 應用程式的憑證 | Microsoft Docs
description: 在 Linux 叢集上為您的應用程式設定用於 Service Fabric 執行階段的憑證
services: service-fabric
documentationcenter: NA
author: JimacoMS2
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/26/2018
ms.author: v-jamebr
ms.openlocfilehash: aa84dc4cb23a0fa2fa854e1f3d6da1234ec00bd6
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386632"
---
# <a name="certificates-and-security-on-linux-clusters"></a>Linux 叢集上的憑證和安全性

本文提供在 Linux 叢集上設定 X.509 憑證的相關資訊。

## <a name="location-and-format-of-x509-certificates-on-linux-nodes"></a>Linux 節點上 X.509 憑證的位置與格式

對於 Service Fabric，X.509 憑證通常會出現在 Linux 叢集節點上的 */var/lib/sfcerts* 目錄中。 叢集憑證、用戶端憑證等憑證都是如此。在某些情況下，您可以為憑證指定 *var/lib/sfcerts* 資料夾以外的位置。 例如，針對使用 Service Fabric Java SDK 的 Reliable Services，您可以透過某些應用程式專用憑證的組態套件 (Settings.xml) 指定不同的位置。 若要深入了解，請參閱[組態套件 (Settings.xml) 中參考的憑證](#certificates-referenced-in-the-configuration-package-settingsxml)。

針對 Linux 叢集，Service Fabric 的憑證通常會以包含憑證和私密金鑰的 .pem 檔案形式提供，或以包含憑證的 .crt 檔案和包含私密金鑰的 .key 檔案個別提供。 所有檔案均應為 PEM 格式。 

如果您使用 [Resource Manager 範本](./service-fabric-cluster-creation-create-template.md)或 [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.servicefabric/?view=latest#service_fabric) 命令從 Azure Key Vault 安裝憑證，憑證將會以正確的格式安裝在每個節點的 */var/lib/sfcerts* 目錄中。 如果您透過其他方法安裝憑證，您必須確定憑證正確地安裝在叢集節點上。

## <a name="certificates-referenced-in-the-application-manifest"></a>應用程式資訊清單中參考的憑證

應用程式資訊清單中指定的憑證 (例如，透過 [**SecretsCertificate**](https://docs.microsoft.com/azure/service-fabric/service-fabric-service-model-schema-elements#secretscertificate-element) 或 [**EndpointCertificate**](https://docs.microsoft.com/azure/service-fabric/service-fabric-service-model-schema-elements#endpointcertificate-element) 元素指定) 必須存在於 */var/lib/sfcerts* 目錄中。 用來在應用程式資訊清單中指定憑證的元素不會採用路徑屬性，因此憑證必須存在於預設目錄中。 這些元素會採用選用的 **X509StoreName** 屬性。 預設值為 "My"，會指向 Linux 節點上的 */var/lib/sfcerts* 目錄。 任何其他值都不會定義於 Linux 叢集上。 對於在 Linux 叢集上執行的應用程式，建議您省略 **X509StoreName** 屬性。 

## <a name="certificates-referenced-in-the-configuration-package-settingsxml"></a>組態套件 (Settings.xml) 中參考的憑證

對於某些服務，您可以在 [ConfigPackage](./service-fabric-application-and-service-manifests.md) (預設為 Settings.xml) 中設定 X.509 憑證。 例如，當您為使用 Service Fabric .NET Core 或 Java SDK 建置的 Reliable Services 服務宣告用來保護 RPC 通道的憑證時，則可以進行前述設定。 有兩種方式可參考組態套件中的憑證。 .NET Core 與 Java SDK 的支援有所不同。

### <a name="using-x509-securitycredentialstype"></a>使用 X509 SecurityCredentialsType

透過 .NET 或 Java SDK，您可以指定 **SecurityCredentialsType** 的 **X509**。 此類型對應於 `SecurityCredentials` ([.NET](https://msdn.microsoft.com/library/system.fabric.securitycredentials.aspx)/[Java](https://docs.microsoft.com/java/api/system.fabric._security_credentials)) 的 `X509Credentials` ([.NET](https://msdn.microsoft.com/library/system.fabric.x509credentials.aspx)/[Java](https://docs.microsoft.com/java/api/system.fabric._x509_credentials)) 類型。

**X509** 參考可將憑證定位於憑證存放區中。 下列 XML 顯示用來指定憑證位置的參數：

```xml
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateStoreName" Value="My" />
```

對於在 Linux 上執行的服務，**LocalMachine**/**My** 會指向憑證的預設位置，即 */var/lib/sfcerts* 目錄。 對於 Linux，**CertificateStoreLocation** 和 **CertificateStoreName** 的任何其他組合皆不會定義。 

請一律將 **CertificateStoreLocation** 參數指定為 **LocalMachine**。 您不需要指定 **CertificateStoreName** 參數，因為該參數預設為 "My"。 使用 **X509** 參考時，憑證檔案必須位於叢集節點的 */var/lib/sfcerts* 目錄中。  

下列 XML 將根據此形式顯示 **TransportSettings** 區段：

```xml
<Section Name="HelloWorldStatefulTransportSettings">
    <Parameter Name="MaxMessageSize" Value="10000000" />
    <Parameter Name="SecurityCredentialsType" Value="X509" />
    <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
    <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
    <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
    <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
    <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
    <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
</Section>
```

### <a name="using-x5092-securitycredentialstype"></a>使用 X509_2 SecurityCredentialsType

透過 Java SDK，您可以指定 **SecurityCredentialsType** 的 **X509_2**。 此類型對應於 `SecurityCredentials` ([Java](https://docs.microsoft.com/java/api/system.fabric._security_credentials)) 的 `X509Credentials2` ([Java](https://docs.microsoft.com/java/api/system.fabric._x509_credentials2)) 類型。 

使用 **X509_2** 參考時，您會指定路徑參數，因此可將憑證定位於 */var/lib/sfcerts* 以外的目錄中。  下列 XML 顯示用來指定憑證位置的參數： 

```xml
     <Parameter Name="SecurityCredentialsType" Value="X509_2" />
     <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
```

下列 XML 將根據此形式顯示 **TransportSettings** 區段。

```xml
<!--Section name should always end with "TransportSettings".-->
<!--Here we are using a prefix "HelloWorldStateless".-->
<Section Name="HelloWorldStatelessTransportSettings">
    <Parameter Name="MaxMessageSize" Value="10000000" />
    <Parameter Name="SecurityCredentialsType" Value="X509_2" />
    <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
    <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
    <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
</Section>
```

> [!NOTE]
> 在上述 XML 中，憑證會指定為 .crt 檔案。 這表示在相同位置中也有一個包含私密金鑰的 .key 檔案。

## <a name="configure-a-reliable-services-app-to-run-on-linux-clusters"></a>將 Reliable Services 應用程式設定為在 Linux 叢集上執行

Service Fabric SDK 可讓您與 Service Fabric 執行階段 API 通訊以使用平台。 當您在安全 Linux 叢集上執行任何使用此功能的應用程式時，都必須設定應用程式，使其具有可用來對 Service Fabric 執行階段進行驗證的憑證。 應用程式若包含使用 .NET Core 或 Java SDK 撰寫的 Service Fabric Reliable Service 服務，則必須進行此設定。 

若要設定應用程式，請在 [憑證] 標記下新增 [**SecretsCertificate**](https://docs.microsoft.com/azure/service-fabric/service-fabric-service-model-schema-elements#secretscertificate-element) 元素；該標記位於 *ApplicationManifest.xml* 檔案中的 **ApplicationManifest** 標記下方。 下列 XML 顯示依憑證的指紋參考的憑證： 

```xml
   <Certificates>
       <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="0A00AA0AAAA0AAA00A000000A0AA00A0AAAA00" />
   </Certificates>   
```

您可以參考叢集憑證或您在每個叢集節點上安裝的憑證。 在 Linux 上，憑證檔案必須存在於 */var/lib/sfcerts* 目錄中。 若要深入了解，請參閱 [Linux 節點上 X.509 憑證的位置與格式](#location-and-format-of-x509-certificates-on-linux-nodes)。



