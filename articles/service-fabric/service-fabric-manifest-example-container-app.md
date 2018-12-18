---
title: Azure Service Fabric 容器應用程式資訊清單範例 | Microsoft Docs
description: 了解如何設定多容器 Service Fabric 應用程式的應用程式和服務資訊清單設定。
services: service-fabric
documentationcenter: na
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: xml
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/08/2018
ms.author: ryanwi
ms.openlocfilehash: 6f538fa821e546d12c5a2bdb9585cc85871241fa
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094147"
---
# <a name="multi-container-application-and-service-manifest-examples"></a>多容器應用程式和服務資訊清單範例
以下舉例說明多容器 Service Fabric 應用程式的應用程式和服務資訊清單。 這些範例的目的是要說明有哪些設定可供使用以及要如何使用。 這些應用程式和服務資訊清單是以 [Windows Server 2016 容器範例](https://github.com/Azure-Samples/service-fabric-containers/tree/master/Windows)的資訊清單作為基礎。

下面是會顯示的功能：
|資訊清單|特性|
|---|---|
|[應用程式資訊清單](#application-manifest)| [覆寫環境變數](service-fabric-get-started-containers.md#configure-and-set-environment-variables)、[設定容器連接埠對主機的對應](service-fabric-get-started-containers.md#configure-container-port-to-host-port-mapping-and-container-to-container-discovery)、[設定容器登錄驗證](service-fabric-get-started-containers.md#configure-container-registry-authentication)、[資源管理](service-fabric-resource-governance.md)、[設定隔離模式](service-fabric-get-started-containers.md#configure-isolation-mode)、[指定 OS 組建專屬的容器映像](service-fabric-get-started-containers.md#specify-os-build-specific-container-images)| 
|[FrontEndService 服務資訊清單](#frontendservice-service-manifest)| [設定環境變數](service-fabric-get-started-containers.md#configure-and-set-environment-variables)、[設定端點](service-fabric-get-started-containers.md#configure-communication)、將命令傳遞至容器、[將憑證匯入容器中](service-fabric-securing-containers.md)| 
|[BackEndService 服務資訊清單](#backendservice-service-manifest)|[設定環境變數](service-fabric-get-started-containers.md#configure-and-set-environment-variables)、[設定端點](service-fabric-get-started-containers.md#configure-communication)、[設定磁碟區驅動程式](service-fabric-containers-volume-logging-drivers.md)| 

如需特定 XML 元素的詳細資訊，請參閱[應用程式資訊清單元素](#application-manifest-elements)、[FrontEndService 服務資訊清單元素](#frontendservice-service-manifest-elements)和 [BackEndService 服務資訊清單元素](#backendservice-service-manifest-elements)。

## <a name="application-manifest"></a>應用程式資訊清單

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="Container.ApplicationType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="BackEndService_InstanceCount" DefaultValue="-1" />
    <Parameter Name="FrontEndService_InstanceCount" DefaultValue="-1" />
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="BlockIOWeight" DefaultValue="200" />
    <Parameter Name="MaximumIOBandwidth" DefaultValue="1024" />
    <Parameter Name="MemoryReservationInMB" DefaultValue="1024" />
    <Parameter Name="MemorySwapInMB" DefaultValue="4084"/>
    <Parameter Name="MaximumIOps" DefaultValue="20"/>
    <Parameter Name="MemoryFront" DefaultValue="4084" />
    <Parameter Name="MemoryBack" DefaultValue="2048" />
    <Parameter Name="CertThumbprint" DefaultValue=""/>
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="BackEndServicePkg" ServiceManifestVersion="1.0.0" />    
    
    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[MemoryFront]"/>

      <!-- Set resource governance at the code package level. -->
      <ResourceGovernancePolicy CodePackageRef="Code" CpuPercent="10" MemoryInMB="[MemoryFront]" BlockIOWeight="[BlockIOWeight]" MaximumIOBandwidth="[MaximumIOBandwidth]" MaximumIOps="[MaximumIOps]" MemoryReservationInMB="[MemoryReservationInMB]" MemorySwapInMB="[MemorySwapInMB]"/>
      
      <!-- Policies for activating container hosts. -->
      <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
        
        <!-- Credentials for the repository hosting the container image.-->
        <RepositoryCredentials AccountName="sfsamples" Password="ENCRYPTED-PASSWORD" PasswordEncrypted="true"/>
        
        <!-- This binds the port the container is listening on (8905 in this sample) to an endpoint resource named "BackEndServiceTypeEndpoint", which is defined in the service manifest.  -->
        <PortBinding ContainerPort="8905" EndpointRef="BackEndServiceTypeEndpoint"/>
        
        <!-- Configure the Azure Files volume plugin.  Bind the source folder on the host VM or a remote share to the destination folder within the running container. -->
        <Volume Source="azfiles" Destination="c:\VolumeTest\Data" Driver="sfazurefile">
          <!-- Driver options to be passed to driver. The Azure Files volume plugin supports the following driver options:
            shareName (the Azure Files file share that provides the volume for the container), storageAccountName (the Azure storage account
            that contains the Azure Files file share), storageAccountKey (Access key for the Azure storage account that contains the Azure Files file share).
            These three driver options are required. -->
          <DriverOption Name="shareName" Value="" />
          <DriverOption Name="storageAccountName" Value="MY-STORAGE-ACCOUNT-NAME" />
          <DriverOption Name="storageAccountKey" Value="MY-STORAGE-ACCOUNT-KEY" />
        </Volume>
        
        <!-- Windows Server containers may not be compatible across different versions of the OS.  You can specify multiple OS images per container and tag 
        them with the build versions of the OS. Get the build version of the OS by running "winver" at a Windows command prompt. -->
        <ImageOverrides>
          <!-- If the underlying OS is build version 16299 (Windows Server version 1709), Service Fabric picks the container image tagged with Os="16299". -->
          <Image Name="sfsamples.azurecr.io/sfsamples/servicefabricbackendservice_1709" Os="16299" />
          
          <!-- An untagged container image is assumed to work across all versions of the OS and overrides the image specified in the service manifest. -->
          <Image Name="sfsamples.azurecr.io/sfsamples/servicefabricbackendservice_default" />          
        </ImageOverrides>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>

  <!-- Policies to be applied to the imported service manifest. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="FrontEndServicePkg" ServiceManifestVersion="1.0.0" />
    
    <!-- This enables you to provide different values for environment variables when creating a FrontEndService
         Theses environment variables are declared in the FrontEndServiceType service manifest-->
    <EnvironmentOverrides CodePackageRef="Code">
      <EnvironmentVariable Name="BackendServiceName" Value="Container.Application/BackEndService"/>
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
      <EnvironmentVariable Name="IsContainer" Value="true"/>
    </EnvironmentOverrides>
    
    <!-- This policy maps the  port of the container (80) to the endpoint declared in the service, 
         FrontEndServiceTypeEndpoint which is exposed as port 80 on the host-->    
    <Policies>

      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[MemoryBack]"/>

      <!-- Policies for activating container hosts. -->
      <ContainerHostPolicies CodePackageRef="Code" Isolation="process">

        <!-- Credentials for the repository hosting the container image.-->
        <RepositoryCredentials AccountName="sfsamples" Password="ENCRYPTED-PASSWORD" PasswordEncrypted="true"/>

        <!-- Binds an endpoint resource (declared in the service manifest) to the exposed container port. -->
        <PortBinding ContainerPort="80" EndpointRef="FrontEndServiceTypeEndpoint"/>

        <!-- Import a certificate into the container.  The certificate must be installed in the LocalMachine store of all the cluster nodes.
          When the application starts, the runtime reads the certificate and generates a PFX file and password (on Windows) or a PEM file (on Linux).
          The PFX file and password are accessible in the container using the Certificates_ServicePackageName_CodePackageName_CertName_PFX and 
          Certificates_ServicePackageName_CodePackageName_CertName_Password environment variables. The PEM file is accessible in the container using the 
          Certificates_ServicePackageName_CodePackageName_CertName_PEM and Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey environment variables.-->
        <CertificateRef Name="MyCert1" X509StoreName="My" X509FindValue="[CertThumbprint]" />

        <!-- If the certificate is already in PFX or PEM form, you can create a data package inside your application and reference that certificate here. -->
        <CertificateRef Name="MyCert2" DataPackageRef="Data" DataPackageVersion="1.0.0" RelativePath="MyCert2.PFX" Password="ENCRYPTED-PASSWORD" IsPasswordEncrypted="true"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
        
    <Service Name="FrontEndService" >
      <StatelessService ServiceTypeName="FrontEndServiceType" InstanceCount="[FrontEndService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
        <Service Name="BackEndService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="BackEndServiceType" InstanceCount="[BackEndService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="frontendservice-service-manifest"></a>FrontEndService 服務資訊清單

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="FrontEndServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="FrontEndServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ContainerHost>
        <!--The repo and image on https://hub.docker.com or Azure Container Registry. -->
        <ImageName>sfsamples.azurecr.io/sfsamples/servicefabricfrontendservice:v1</ImageName>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container or exe.  These variables are overridden in the application manifest. -->
    <EnvironmentVariables>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="IsContainer" Value=""/>
    </EnvironmentVariables>
  </CodePackage>

  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />
  
  <!-- Data package is the contents of the Data directory under PackageRoot that contains an 
       independently-updateable and versioned static data that's consumed by the process at runtime. -->
  <DataPackage Name="Data" Version="1.0.0"/>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. For a guest executable is used to register with the NamingService at its REST endpoint
           with http scheme and port 80 -->
      <Endpoint Name="FrontEndServiceTypeEndpoint" UriScheme="http" Port="80"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="backendservice-service-manifest"></a>BackEndService 服務資訊清單

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="BackEndServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="BackEndServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ContainerHost>
        <!--The repo and image on https://hub.docker.com or Azure Container Registry. -->
        <ImageName>sfsamples.azurecr.io/sfsamples/servicefabricbackendservice:v1</ImageName>
        
        <!-- Pass comma delimited commands to your container. -->
        <Commands> dotnet, myproc.dll, 5 </Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container. These variables are overridden in the application manifest. -->
    <EnvironmentVariables>
      <EnvironmentVariable Name="IsContainer" Value="true"/>
    </EnvironmentVariables>
  </CodePackage>

  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the host port on which to 
           listen. For a guest executable is used to register with the NamingService at its REST endpoint
           with http scheme. In this case since no port is specified, one is created and assigned dynamically
           to the service. This dynamically assigned host port is mapped to the container port (8905 in this sample),
            which was specified in the application manifest.-->
      <Endpoint Name="BackEndServiceTypeEndpoint" UriScheme="http" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="application-manifest-elements"></a>應用程式資訊清單元素
### <a name="applicationmanifest-element"></a>ApplicationManifest 元素
以宣告方式說明應用程式類型和版本。 系統會參考組成服務的一個或多個服務資訊清單，來撰寫應用程式類型。 組成服務的組態設定可使用參數化的應用程式設定來加以覆寫。 預設服務、服務範本、主體、原則、診斷設定和憑證也可在應用程式層級來宣告。 如需詳細資訊，請參閱 [ApplicationManifest 元素](service-fabric-service-model-schema-elements.md#ApplicationManifestElementApplicationManifestTypeComplexType)

### <a name="parameters-element"></a>Parameters 元素
宣告會在此應用程式資訊清單中使用的參數。 這些參數的值可在具現化應用程式時予以提供，而且可用來覆寫應用程式或服務的組態設定。 如需詳細資訊，請參閱 [Parameters 元素](service-fabric-service-model-schema-elements.md#ParametersElementanonymouscomplexTypeComplexTypeDefinedInApplicationManifestTypecomplexType)

### <a name="parameter-element"></a>Parameter 元素
要在此資訊清單中使用的應用程式參數。 應用程式具現化期間可以變更參數值，或者，如果未提供值，則會使用預設值。 如需詳細資訊，請參閱 [Parameter 元素](service-fabric-service-model-schema-elements.md#ParameterElementanonymouscomplexTypeComplexTypeDefinedInParameterselement)

### <a name="servicemanifestimport-element"></a>ServiceManifestImport 元素
匯入服務開發人員所建立的服務資訊清單。 您必須針對應用程式中的每個組成服務匯入服務資訊清單。 針對服務資訊清單，則可以宣告組態覆寫和原則。 如需詳細資訊，請參閱 [ServiceManifestImport 元素](service-fabric-service-model-schema-elements.md#ServiceManifestImportElementanonymouscomplexTypeComplexTypeDefinedInApplicationManifestTypecomplexType)

### <a name="servicemanifestref-element"></a>ServiceManifestRef 元素
依參考匯入服務資訊清單。 服務資訊清單檔案 (ServiceManifest.xml) 目前必須存在於組建套件中。 如需詳細資訊，請參閱 [ServiceManifestRef 元素](service-fabric-service-model-schema-elements.md#ServiceManifestRefElementServiceManifestRefTypeComplexTypeDefinedInServiceManifestImportelement)

### <a name="policies-element"></a>Policies 元素
說明要在匯入的服務資訊清單上套用的原則 (端點繫結、套件共用、執行身分和安全性存取)。 如需詳細資訊，請參閱 [Policies 元素](service-fabric-service-model-schema-elements.md#PoliciesElementServiceManifestImportPoliciesTypeComplexTypeDefinedInServiceManifestImportelement)

### <a name="servicepackageresourcegovernancepolicy-element"></a>ServicePackageResourceGovernancePolicy 元素
定義在整個服務套件層級套用的資源治理原則。 如需詳細資訊，請參閱 [ServicePackageResourceGovernancePolicy 元素](service-fabric-service-model-schema-elements.md#ServicePackageResourceGovernancePolicyElementServicePackageResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInServicePackageTypecomplexType)

### <a name="resourcegovernancepolicy-element"></a>ResourceGovernancePolicy 元素
指定程式碼套件的資源限制。 如需詳細資訊，請參閱 [ResourceGovernancePolicy 元素](service-fabric-service-model-schema-elements.md#ResourceGovernancePolicyElementResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelementDefinedInDigestedEndpointelement)

### <a name="containerhostpolicies-element"></a>ContainerHostPolicies 元素
指定用於啟動容器主機的原則。 如需詳細資訊，請參閱 [ContainerHostPolicies 元素](service-fabric-service-model-schema-elements.md#ContainerHostPoliciesElementContainerHostPoliciesTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelement)

### <a name="repositorycredentials-element"></a>RepositoryCredentials 元素
要從中提取映像之容器映像存放庫的認證。 如需詳細資訊，請參閱 [RepositoryCredentials 元素](service-fabric-service-model-schema-elements.md#RepositoryCredentialsElementRepositoryCredentialsTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType)

### <a name="portbinding-element"></a>PortBinding 元素
指定哪些端點資源要繫結至公開的容器連接埠。 如需詳細資訊，請參閱 [PortBinding 元素](service-fabric-service-model-schema-elements.md#PortBindingElementPortBindingTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType)

### <a name="volume-element"></a>Volume 元素
指定要繫結至容器的磁碟區。 如需詳細資訊，請參閱 [Volume 元素](service-fabric-service-model-schema-elements.md#VolumeElementContainerVolumeTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType)

### <a name="driveroption-element"></a>DriverOption 元素
要傳遞至驅動程式的驅動程式選項。 如需詳細資訊，請參閱 [DriverOption 元素](service-fabric-service-model-schema-elements.md#DriverOptionElementDriverOptionTypeComplexTypeDefinedInContainerLoggingDriverTypecomplexTypeDefinedInContainerVolumeTypecomplexType)

### <a name="imageoverrides-element"></a>ImageOverrides 元素
Windows Server 容器在不同的 OS 版本之間可能不相容。  每個容器可以指定多個 OS 映像，並以 OS 的組建版本標記。 在 Windows 命令提示字元執行 "winver"，即可取得 OS 的組建版本。 如果基礎 OS 的組建版本為 16299 (Windows Server 1709 版)，Service Fabric 會挑選以 Os="16299" 標記的容器映像。 假設未標記的容器映像可跨所有 OS 版本運作，並覆寫服務資訊清單中指定的映像。 如需詳細資訊，請參閱 [ImageOverrides 元素](service-fabric-service-model-schema-elements.md#ImageOverridesElementImageOverridesTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType)

### <a name="image-element"></a>Image 元素
對應至要啟動之 OS 組建版本號碼的容器映像。 如果未指定 Os 屬性，則會假設容器映像可跨所有 OS 版本運作，並覆寫服務資訊清單中指定的映像。 如需詳細資訊，請參閱 [Image 元素](service-fabric-service-model-schema-elements.md#ImageElementImageTypeComplexTypeDefinedInImageOverridesTypecomplexType)

### <a name="environmentoverrides-element"></a>EnvironmentOverrides 元素
 如需詳細資訊，請參閱 [EnvironmentOverrides 元素](service-fabric-service-model-schema-elements.md#EnvironmentOverridesElementEnvironmentOverridesTypeComplexTypeDefinedInServiceManifestImportelement)

### <a name="environmentvariable-element"></a>EnvironmentVariable 元素
環境變數。 如需詳細資訊，請參閱 [EnvironmentVariable 元素](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexTypeDefinedInEnvironmentVariablesTypecomplexType)

### <a name="certificateref-element"></a>CertificateRef 元素
指定要對容器環境公開之 X509 憑證的相關資訊。 此憑證必須安裝在所有叢集節點的 LocalMachine 存放區中。
應用程式啟動時，執行階段會讀取憑證，並產生 PFX 檔案和密碼 (Windows 上) 或 PEM 檔案 (Linux 上)。
PFX 檔案和密碼可透過 Certificates_ServicePackageName_CodePackageName_CertName_PFX 和 Certificates_ServicePackageName_CodePackageName_CertName_Password 環境變數從容器中存取。 PEM 檔案可透過 Certificates_ServicePackageName_CodePackageName_CertName_PEM 和 Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey 環境變數從容器中存取。 如需詳細資訊，請參閱 [CertificateRef 元素](service-fabric-service-model-schema-elements.md#CertificateRefElementContainerCertificateTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType)

### <a name="defaultservices-element"></a>DefaultServices 元素
宣告服務執行個體，該執行個體會在每次應用程式針對此應用程式類型具現化時建立。 如需詳細資訊，請參閱 [DefaultServices 元素](service-fabric-service-model-schema-elements.md#DefaultServicesElementDefaultServicesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInApplicationInstanceTypecomplexType)

### <a name="service-element"></a>Service 元素
宣告要在應用程式具現化時自動建立的服務。 如需詳細資訊，請參閱 [Service 元素](service-fabric-service-model-schema-elements.md#ServiceElementanonymouscomplexTypeComplexTypeDefinedInDefaultServicesTypecomplexType)

### <a name="statelessservice-element"></a>StatelessService 元素
定義無狀態服務。 如需詳細資訊，請參閱 [StatelessService 元素](service-fabric-service-model-schema-elements.md#StatelessServiceElementStatelessServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement)


## <a name="frontendservice-service-manifest-elements"></a>FrontEndService 服務資訊清單元素
### <a name="servicemanifest-element"></a>ServiceManifest 元素
以宣告方式說明服務類型和版本。 它會列出可獨立升級的程式碼、組態和資料套件，這些項目合在一起即可構成服務套件以支援一或多個服務類型。 它也會指定資源、診斷設定和服務中繼資料，例如服務類型、健康情況屬性和負載平衡計量。 如需詳細資訊，請參閱 [ServiceManifest 元素](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType)

### <a name="servicetypes-element"></a>ServiceTypes 元素
宣告此資訊清單中的 CodePackage 支援哪些服務類型。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 服務類型是在資訊清單層級而非程式碼套件層級宣告。 如需詳細資訊，請參閱 [ServiceTypes 元素](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="statelessservicetype-element"></a>StatelessServiceType 元素
說明無狀態服務類型。 如需詳細資訊，請參閱 [StatelessServiceType 元素](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType)

### <a name="codepackage-element"></a>CodePackage 元素
說明可支援已定義之服務類型的程式碼套件。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 產生的處理程序預期在執行階段註冊支援的服務類型。 如果有多個程式碼套件，每當系統尋找任何一個宣告的服務類型時，它們都會啟動。 如需詳細資訊，請參閱 [CodePackage 元素](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement)

### <a name="entrypoint-element"></a>EntryPoint 元素
EntryPoint 指定的可執行檔通常是長時間執行的服務主機。 有個別設定的進入點，就不需要使用較高權限來長時間執行服務主機。 EntryPoint 指定的可執行檔是在 SetupEntryPoint 成功結束之後執行。 如果曾經終止或當機，產生的程序會監視並重新啟動 (以 SetupEntryPoint再次開始)。 如需詳細資訊，請參閱 [EntryPoint 元素](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="containerhost-element"></a>ContainerHost 元素
 如需詳細資訊，請參閱 [ContainerHost 元素](service-fabric-service-model-schema-elements.md#ContainerHostElementContainerHostEntryPointTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType)

### <a name="imagename-element"></a>ImageName 元素
https://hub.docker.com 或 Azure Container Registry 上的存放庫和映像。 如需詳細資訊，請參閱 [ImageName 元素](service-fabric-service-model-schema-elements.md#ImageNameElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType)

### <a name="environmentvariables-element"></a>EnvironmentVariables 元素
將環境變數傳遞到您的容器或 exe。  如需詳細資訊，請參閱 [EnvironmentVariables 元素](service-fabric-service-model-schema-elements.md#EnvironmentVariablesElementEnvironmentVariablesTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="environmentvariable-element"></a>EnvironmentVariable 元素
環境變數。 如需詳細資訊，請參閱 [EnvironmentVariable 元素](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexTypeDefinedInEnvironmentVariablesTypecomplexType)

### <a name="configpackage-element"></a>ConfigPackage 元素
宣告 Name 屬性所命名的資料夾，其中包含 Settings.xml 檔案。 此檔案包含程序可以在執行階段讀回的使用者定義、成對的索引鍵/值設定等區段。 在升級期間，如果只有 ConfigPackage 版本已變更，則不會重新啟動執行中程序。 相反地，回呼會通知程序組態設定已變更，因此它們可以動態方式重新載入。 如需詳細資訊，請參閱 [ConfigPackage 元素](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement)

### <a name="datapackage-element"></a>DataPackage 元素
宣告 Name 屬性所命名的資料夾，其中包含靜態資料檔案。 當服務資訊清單中列出的資料套件有任何一個已進行升級，Service Fabric 就會回收主機和支援套件中指定的所有 EXE 和 DLLHOST。 如需詳細資訊，請參閱 [DataPackage 元素](service-fabric-service-model-schema-elements.md#DataPackageElementDataPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedDataPackageelement)

### <a name="resources-element"></a>Resources 元素
說明此服務所使用的資源，即可在不修改已編譯之程式碼的情況下加以宣告，並於服務已部署時加以變更。 這些資源的存取權會透過應用程式資訊清單的 Principals 和 Policies 區段來控制。 如需詳細資訊，請參閱 [Resources 元素](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="endpoints-element"></a>Endpoints 元素
定義服務的端點。 如需詳細資訊，請參閱 [Endpoints 元素](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType)

### <a name="endpoint-element"></a>Endpoint 元素
如需詳細資訊，請參閱 [Endpoint 元素](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement)


## <a name="backendservice-service-manifest-elements"></a>BackEndService 服務資訊清單元素
### <a name="servicemanifest-element"></a>ServiceManifest 元素
以宣告方式說明服務類型和版本。 它會列出可獨立升級的程式碼、組態和資料套件，這些項目合在一起即可構成服務套件以支援一或多個服務類型。 它也會指定資源、診斷設定和服務中繼資料，例如服務類型、健康情況屬性和負載平衡計量。 如需詳細資訊，請參閱 [ServiceManifest 元素](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType)

### <a name="servicetypes-element"></a>ServiceTypes 元素
宣告此資訊清單中的 CodePackage 支援哪些服務類型。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 服務類型是在資訊清單層級而非程式碼套件層級宣告。 如需詳細資訊，請參閱 [ServiceTypes 元素](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="statelessservicetype-element"></a>StatelessServiceType 元素
說明無狀態服務類型。 如需詳細資訊，請參閱 [StatelessServiceType 元素](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType)

### <a name="codepackage-element"></a>CodePackage 元素
說明可支援已定義之服務類型的程式碼套件。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 產生的處理程序預期在執行階段註冊支援的服務類型。 如果有多個程式碼套件，每當系統尋找任何一個宣告的服務類型時，它們都會啟動。 如需詳細資訊，請參閱 [CodePackage 元素](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement)

### <a name="entrypoint-element"></a>EntryPoint 元素
EntryPoint 指定的可執行檔通常是長時間執行的服務主機。 有個別設定的進入點，就不需要使用較高權限來長時間執行服務主機。 EntryPoint 指定的可執行檔是在 SetupEntryPoint 成功結束之後執行。 如果曾經終止或當機，產生的程序會監視並重新啟動 (以 SetupEntryPoint再次開始)。 如需詳細資訊，請參閱 [EntryPoint 元素](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="containerhost-element"></a>ContainerHost 元素
如需詳細資訊，請參閱 [ContainerHost 元素](service-fabric-service-model-schema-elements.md#ContainerHostElementContainerHostEntryPointTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType)

### <a name="imagename-element"></a>ImageName 元素
https://hub.docker.com 或 Azure Container Registry 上的存放庫和映像。 如需詳細資訊，請參閱 [ImageName 元素](service-fabric-service-model-schema-elements.md#ImageNameElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType)

### <a name="commands-element"></a>Commands 元素
將逗號分隔命令清單傳遞到容器。 如需詳細資訊，請參閱 [Commands 元素](service-fabric-service-model-schema-elements.md#CommandsElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType)

### <a name="environmentvariables-element"></a>EnvironmentVariables 元素
將環境變數傳遞到您的容器或 exe。  如需詳細資訊，請參閱 [EnvironmentVariables 元素](service-fabric-service-model-schema-elements.md#EnvironmentVariablesElementEnvironmentVariablesTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="environmentvariable-element"></a>EnvironmentVariable 元素
環境變數。 如需詳細資訊，請參閱 [EnvironmentVariable 元素](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexTypeDefinedInEnvironmentVariablesTypecomplexType)

### <a name="configpackage-element"></a>ConfigPackage 元素
宣告 Name 屬性所命名的資料夾，其中包含 Settings.xml 檔案。 此檔案包含程序可以在執行階段讀回的使用者定義、成對的索引鍵/值設定等區段。 在升級期間，如果只有 ConfigPackage 版本已變更，則不會重新啟動執行中程序。 相反地，回呼會通知程序組態設定已變更，因此它們可以動態方式重新載入。 如需詳細資訊，請參閱 [ConfigPackage 元素](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement)

### <a name="resources-element"></a>Resources 元素
說明此服務所使用的資源，即可在不修改已編譯之程式碼的情況下加以宣告，並於服務已部署時加以變更。 這些資源的存取權會透過應用程式資訊清單的 Principals 和 Policies 區段來控制。 如需詳細資訊，請參閱 [Resources 元素](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="endpoints-element"></a>Endpoints 元素
定義服務的端點。 如需詳細資訊，請參閱 [Endpoints 元素](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType)

### <a name="endpoint-element"></a>Endpoint 元素
 如需詳細資訊，請參閱 [Endpoint 元素](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement)

