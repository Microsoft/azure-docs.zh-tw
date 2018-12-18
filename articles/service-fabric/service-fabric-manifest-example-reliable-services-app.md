---
title: Azure Service Fabric Reliable Services 應用程式資訊清單範例 | Microsoft Docs
description: 了解如何設定 Reliable Services Service Fabric 應用程式的應用程式和服務資訊清單設定。
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
ms.date: 06/11/2018
ms.author: ryanwi
ms.openlocfilehash: c90715608b5d35520605c504b5cebb5e7a3ec021
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47096628"
---
# <a name="reliable-services-application-and-service-manifest-examples"></a>Reliable Services 應用程式和服務資訊清單範例
以下舉例說明 Service Fabric 應用程式 (具有 ASP.NET Core Web 前端和具狀態後端) 的應用程式和服務資訊清單。 這些範例的目的是要說明有哪些設定可供使用以及要如何使用。 這些應用程式和服務資訊清單是以 [Service Fabric .NET 快速入門](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/)的資訊清單作為基礎。

下面是會顯示的功能：
|資訊清單|特性|
|---|---|
|[應用程式資訊清單](#application-manifest)| [資源管理](service-fabric-resource-governance.md)、[以本機系統管理員帳戶的身分執行服務](service-fabric-application-runas-security.md)、[對所有服務程式碼套件套用預設原則](service-fabric-application-runas-security.md#apply-a-default-policy-to-all-service-code-packages)、[建立使用者和群組主體](service-fabric-application-runas-security.md)、在服務執行個體之間共用資料套件、[覆寫服務端點](service-fabric-service-manifest-resources.md#overriding-endpoints-in-servicemanifestxml)| 
|[FrontEndService 服務資訊清單](#frontendservice-service-manifest)| [在服務啟動時執行指令碼](service-fabric-run-script-at-service-startup.md)、[定義 HTTPS 端點](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest) | 
|[BackEndService 服務資訊清單](#backendservice-service-manifest)| [宣告設定套件](service-fabric-application-and-service-manifests.md)、[宣告資料套件](service-fabric-application-and-service-manifests.md)、[設定端點](service-fabric-service-manifest-resources.md)| 

如需特定 XML 元素的詳細資訊，請參閱[應用程式資訊清單元素](#application-manifest-elements)、[VotingWeb 服務資訊清單元素](#votingweb-service-manifest-elements)和 [VotingData 服務資訊清單元素](#votingdata-service-manifest-elements)。

## <a name="application-manifest"></a>應用程式資訊清單

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="Memory" DefaultValue="4084" />
    <Parameter Name="BlockIOWeight" DefaultValue="200" />
    <Parameter Name="MaximumIOBandwidth" DefaultValue="1024" />
    <Parameter Name="MemoryReservationInMB" DefaultValue="1024" />
    <Parameter Name="MemorySwapInMB" DefaultValue="4084"/>
    <Parameter Name="Port" DefaultValue="8081" />
    <Parameter Name="Protocol" DefaultValue="tcp" />
    <Parameter Name="Type" DefaultValue="internal" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <!-- Override endpoints declared in the service manifest. -->
    <ResourceOverrides>
      <Endpoints>
        <Endpoint Name="DataEndpoint" Port="[Port]" Protocol="[Protocol]" Type="[Type]" />
      </Endpoints>
    </ResourceOverrides>

    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      
      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[Memory]"/>

      <!-- Set resource governance at the code package level. -->
      <ResourceGovernancePolicy CodePackageRef="Code" CpuPercent="10" MemoryInMB="[Memory]" BlockIOWeight="[BlockIOWeight]" 
                                MaximumIOBandwidth="[MaximumIOBandwidth]" MaximumIOps="[MaximumIOps]" MemoryReservationInMB="[MemoryReservationInMB]" 
                                MemorySwapInMB="[MemorySwapInMB]"/>

      <!-- Share the data package across multiple instances of the VotingData service-->
      <PackageSharingPolicy PackageRef="VotingDataPkg.Data"/>

      <!-- Give read rights on the "DataEndpoint" endpoint to the Customer2 account.-->
      <SecurityAccessPolicy GrantRights="Read" PrincipalRef="Customer2" ResourceRef="DataEndpoint" ResourceType="Endpoint"/>         
    </Policies>
  </ServiceManifestImport>
  
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    
    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      <!-- Run the setup entry point (defined in the imported service manifest) as the SetupAdminUser account 
      (declared in the following Principals section). -->
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
      
    </Policies>

  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <!-- Define users and groups required to run the services and access resources.  Principals are used in the Policies section(s). -->
  <Principals>
    <!-- Declare a set of groups as security principals, which can be referenced in policies. Groups are useful if there are multiple users 
    for different service entry points and they need to have certain common privileges that are available at the group level. -->
    <Groups>
      <!-- Create a group that has administrator privileges. -->
      <Group Name="LocalAdminGroup">
        <Membership>
          <SystemGroup Name="Administrators" />
        </Membership>
      </Group>
    </Groups>
    <Users>
      <!-- Declare a user and add the user to the Administrators system group. The SetupAdminUser account is used to run the 
      setup entry point of the VotingWebPkg code package (described in the preceding Policies section).-->
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
      <!-- Create a user. Local user accounts are created on the machines where the application is deployed. By default, these accounts 
      do not have the same names as those specified here. Instead, they are dynamically generated and have random passwords. -->
      <User Name="Customer1" >
        <MemberOf>
          <!-- Add the user to the local administrators group.-->
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <!-- Create a user as a local user with the specified account name and password. Local user accounts are created on the machines 
      where the application is deployed. -->
      <User Name="Customer2" AccountType="LocalUser" AccountName="Customer1" Password="MyPassword">
        <MemberOf>
          <!-- Add the user to the local administrators group.-->
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <!-- Create a user as NetworkService. -->
      <User Name="MyDefaultAccount" AccountType="NetworkService" />      
    </Users>
  </Principals>
  <!-- Policies applied at the application level. -->
  <Policies>
    <!-- Specify a default user account for all code packages that don’t have a specific RunAsPolicy defined in 
    the ServiceManifestImport section(s). -->
    <DefaultRunAsPolicy UserRef="MyDefaultAccount" />
    
  </Policies>
</ApplicationManifest>

```

## <a name="votingweb-service-manifest"></a>VotingWeb 服務資訊清單

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <!-- A privileged entry point that by default runs with the same credentials as Service Fabric (typically the NetworkService account) before 
    any other entry point. Use the setup entry point to set system environment variables, give the account running the service (NETWORK SERVICE, by default) 
    access to a certificate's private key, or perform other setup tasks. In the application manifest, you can change the security permissions to run the startup script 
    under a local system account or an administrator account.  -->
    <SetupEntryPoint>
      <ExeHost>
        <!-- The setup script to run. -->
        <Program>Setup.bat</Program>
        <!-- Pass arguments to the script when it runs.-->
        <Arguments>MyValue</Arguments>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
        <!-- Warning! Do not use console rediriction in a production application, only use it for local development and debugging. Redirects console output from the startup
        script to an output file in the application folder called "log" on the cluster node where the application is deployed and run. Also set the number of output files
        to retain and the maximum file size (in KB). -->
        <ConsoleRedirection FileRetentionCount="10" FileMaxSizeInKb="20480"/>
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- Configure a HTTPS endpoint on port 443. This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>

```

## <a name="votingdata-service-manifest"></a>VotingData 服務資訊清單

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingDataPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatefulServiceType ServiceTypeName="VotingDataType"  HasPersistedState="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingData.exe</Program>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Declares a folder, named by the Name attribute, under PackageRoot that contains a Settings.xml file. This file contains sections of user-defined, 
  key-value pair settings that the process can read back at run time. During an upgrade, if only the ConfigPackage version has changed, 
  then the running process is not restarted. Instead, a callback notifies the process that configuration settings have changed so they can be reloaded dynamically. -->
  <ConfigPackage Name="Config" Version="1.0.0" />
  
  <!-- Declares a folder, named by the Name attribute, under PackageRoot which contains static data files to be consumed by the process at run time. -->
  <DataPackage Name="Data" Version="1.0.0"/>

  <Resources>
    <Endpoints>
      <!-- Define an internal (used for intra-application communication) TCP endpoint. Since no port is specified, one is created and assigned dynamically
           to the service.-->
      <Endpoint Name="DataEndpoint" Protocol="tcp" Type="Internal" />
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

### <a name="resourceoverrides-element"></a>ResourceOverrides 元素
指定服務資訊清單資源中所宣告端點的資源覆寫。 如需詳細資訊，請參閱 [ResourceOverrides 元素](service-fabric-service-model-schema-elements.md#ResourceOverridesElementResourceOverridesTypeComplexTypeDefinedInServiceManifestImportelement)

### <a name="endpoints-element"></a>Endpoints 元素
要覆寫的端點。 如需詳細資訊，請參閱 [Endpoints 元素](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourceOverridesTypecomplexType)

### <a name="endpoint-element"></a>Endpoint 元素
服務資訊清單中所宣告要覆寫的端點。 如需詳細資訊，請參閱 [Endpoint 元素](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement)

### <a name="policies-element"></a>Policies 元素
說明要在匯入的服務資訊清單上套用的原則 (端點繫結、套件共用、執行身分和安全性存取)。 如需詳細資訊，請參閱 [Policies 元素](service-fabric-service-model-schema-elements.md#PoliciesElementServiceManifestImportPoliciesTypeComplexTypeDefinedInServiceManifestImportelement)

### <a name="servicepackageresourcegovernancepolicy-element"></a>ServicePackageResourceGovernancePolicy 元素
定義在整個服務套件層級套用的資源治理原則。 如需詳細資訊，請參閱 [ServicePackageResourceGovernancePolicy 元素](service-fabric-service-model-schema-elements.md#ServicePackageResourceGovernancePolicyElementServicePackageResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInServicePackageTypecomplexType)

### <a name="resourcegovernancepolicy-element"></a>ResourceGovernancePolicy 元素
指定 codepackage 的資源限制。 如需詳細資訊，請參閱 [ResourceGovernancePolicy 元素](service-fabric-service-model-schema-elements.md#ResourceGovernancePolicyElementResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelementDefinedInDigestedEndpointelement)

### <a name="packagesharingpolicy-element"></a>PackageSharingPolicy 元素
指出是否應該跨相同服務類型的服務執行個體來共用程式碼、設定或資料套件。 如需詳細資訊，請參閱 [PackageSharingPolicy 元素](service-fabric-service-model-schema-elements.md#PackageSharingPolicyElementPackageSharingPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexType)

### <a name="securityaccesspolicy-element"></a>SecurityAccessPolicy 元素
授與存取權限給服務資訊清單中定義之資源上的主體 (例如端點)。 一般而言，控制並限制服務對不同資源的存取很適合用來將安全性風險降到最低。 當應用程式是透過市集內一組由不同開發人員所開發之服務建置而成時，這一點特別重要。 如需詳細資訊，請參閱 [SecurityAccessPolicy 元素](service-fabric-service-model-schema-elements.md#SecurityAccessPolicyElementSecurityAccessPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInSecurityAccessPolicieselementDefinedInDigestedEndpointelement)

### <a name="runaspolicy-element"></a>RunAsPolicy 元素
指定將作為服務程式碼套件執行身分的本機使用者或本機系統帳戶。 有 Azure Active Directory 可供使用的 Windows Server 部署可支援網域帳戶。 根據預設，應用程式會在用以執行 Fabric.exe 程序的帳戶之下執行。 應用程式也可以使用其他帳戶作為執行身分，但必須在 Principals 區段中予以宣告。 如果您將 RunAs 原則套用到服務，而且服務資訊清單宣告具有 HTTP 通訊協定的端點資源，則您也必須指定 SecurityAccessPolicy，以確保配置給這些端點的連接埠都已針對用以執行服務的 RunAs 使用者帳戶，正確列入存取控制清單中。 針對 HTTPS 端點，您也必須定義 EndpointBindingPolicy 以指出要傳回給用戶端的憑證名稱。 如需詳細資訊，請參閱 [RunAsPolicy 元素](service-fabric-service-model-schema-elements.md#RunAsPolicyElementRunAsPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelement)

### <a name="defaultservices-element"></a>DefaultServices 元素
宣告服務執行個體，該執行個體會在每次應用程式針對此應用程式類型具現化時建立。 如需詳細資訊，請參閱 [DefaultServices 元素](service-fabric-service-model-schema-elements.md#DefaultServicesElementDefaultServicesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInApplicationInstanceTypecomplexType)

### <a name="service-element"></a>Service 元素
宣告要在應用程式具現化時自動建立的服務。 如需詳細資訊，請參閱 [Service 元素](service-fabric-service-model-schema-elements.md#ServiceElementanonymouscomplexTypeComplexTypeDefinedInDefaultServicesTypecomplexType)

### <a name="statefulservice-element"></a>StatefulService 元素
定義具狀態服務。 如需詳細資訊，請參閱 [StatefulService 元素](service-fabric-service-model-schema-elements.md#StatefulServiceElementStatefulServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement)

### <a name="statelessservice-element"></a>StatelessService 元素
定義無狀態服務。 如需詳細資訊，請參閱 [StatelessService 元素](service-fabric-service-model-schema-elements.md#StatelessServiceElementStatelessServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement)

### <a name="principals-element"></a>Principals 元素
說明要讓此應用程式執行服務和保護資源所需要的安全性主體 (使用者、群組)。 主體會在 Policies 區段來參考。 如需詳細資訊，請參閱 [Principals 元素](service-fabric-service-model-schema-elements.md#PrincipalsElementSecurityPrincipalsTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInEnvironmentTypecomplexType)

### <a name="groups-element"></a>Groups 元素
宣告一組群組來作為安全性主體，以供在原則中參考。 當不同的服務進入點有多個使用者，而他們需要具備可在群組層級取得的某些通用權限時，群組會很有用。 如需詳細資訊，請參閱 [Groups 元素](service-fabric-service-model-schema-elements.md#GroupsElementanonymouscomplexTypeComplexTypeDefinedInSecurityPrincipalsTypecomplexType)

### <a name="group-element"></a>Group 元素
宣告群組來作為安全性主體，以供在原則中參考。 如需詳細資訊，請參閱 [Group 元素](service-fabric-service-model-schema-elements.md#GroupElementanonymouscomplexTypeComplexTypeDefinedInGroupselement)

### <a name="membership-element"></a>Membership 元素
 如需詳細資訊，請參閱 [Membership 元素](service-fabric-service-model-schema-elements.md#MembershipElementanonymouscomplexTypeComplexTypeDefinedInGroupelement)

### <a name="systemgroup-element"></a>SystemGroup 元素
 如需詳細資訊，請參閱 [SystemGroup 元素](service-fabric-service-model-schema-elements.md#SystemGroupElementanonymouscomplexTypeComplexTypeDefinedInMembershipelement)

### <a name="users-element"></a>Users 元素
宣告一組使用者來作為安全性主體，以供在原則中參考。 如需詳細資訊，請參閱 [Users 元素](service-fabric-service-model-schema-elements.md#UsersElementanonymouscomplexTypeComplexTypeDefinedInSecurityPrincipalsTypecomplexType)

### <a name="user-element"></a>User 元素
宣告使用者來作為安全性主體，以供在原則中參考。 如需詳細資訊，請參閱 [User 元素](service-fabric-service-model-schema-elements.md#UserElementanonymouscomplexTypeComplexTypeDefinedInUserselement)

### <a name="memberof-element"></a>MemberOf 元素
使用者可以新增至任何現有的成員資格群組，因此它可以繼承該成員資格群組的所有屬性和安全性設定。 成員資格群組可用來保護不同服務或相同服務 (在不同的電腦上) 需要存取的外部資源。 如需詳細資訊，請參閱 [MemberOf 元素](service-fabric-service-model-schema-elements.md#MemberOfElementanonymouscomplexTypeComplexTypeDefinedInUserelement)

### <a name="systemgroup-element"></a>SystemGroup 元素
使用者所要新增到的系統群組。  系統群組必須在 Groups 區段中定義。 如需詳細資訊，請參閱 [SystemGroup 元素](service-fabric-service-model-schema-elements.md#SystemGroupElementanonymouscomplexTypeComplexTypeDefinedInMemberOfelement)

### <a name="group-element"></a>Group 元素
使用者所要新增到的群組。  群組必須在 Groups 區段中定義。 如需詳細資訊，請參閱 [Group 元素](service-fabric-service-model-schema-elements.md#GroupElementanonymouscomplexTypeComplexTypeDefinedInMemberOfelement)

### <a name="policies-element"></a>Policies 元素
說明要在應用程式層級套用的原則 (記錄集合、預設執行身分、健康情況和安全性存取)。 如需詳細資訊，請參閱 [Policies 元素](service-fabric-service-model-schema-elements.md#PoliciesElementApplicationPoliciesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInEnvironmentTypecomplexType)

### <a name="defaultrunaspolicy-element"></a>DefaultRunAsPolicy 元素
針對未在 ServiceManifestImport 區段定義特定 RunAsPolicy 的所有服務程式碼封裝，指定預設的使用者帳戶。 如需詳細資訊，請參閱 [DefaultRunAsPolicy 元素](service-fabric-service-model-schema-elements.md#DefaultRunAsPolicyElementanonymouscomplexTypeComplexTypeDefinedInApplicationPoliciesTypecomplexType)




## <a name="votingweb-service-manifest-elements"></a>VotingWeb 服務資訊清單元素
### <a name="servicemanifest-element"></a>ServiceManifest 元素
以宣告方式說明服務類型和版本。 它會列出可獨立升級的程式碼、組態和資料套件，這些項目合在一起即可構成服務套件以支援一或多個服務類型。 它也會指定資源、診斷設定和服務中繼資料，例如服務類型、健康情況屬性和負載平衡計量。 如需詳細資訊，請參閱 [ServiceManifest 元素](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType)

### <a name="servicetypes-element"></a>ServiceTypes 元素
宣告此資訊清單中的 CodePackage 支援哪些服務類型。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 服務類型是在資訊清單層級而非程式碼套件層級宣告。 如需詳細資訊，請參閱 [ServiceTypes 元素](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="statelessservicetype-element"></a>StatelessServiceType 元素
說明無狀態服務類型。 如需詳細資訊，請參閱 [StatelessServiceType 元素](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType)

### <a name="codepackage-element"></a>CodePackage 元素
說明可支援已定義之服務類型的程式碼套件。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 產生的處理程序預期在執行階段註冊支援的服務類型。 如果有多個程式碼套件，每當系統尋找任何一個宣告的服務類型時，它們都會啟動。 如需詳細資訊，請參閱 [CodePackage 元素](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement)

### <a name="setupentrypoint-element"></a>SetupEntryPoint 元素
以與 Service Fabric 相同的認證 (通常是 NETWORKSERVICE 帳戶) 預設執行的特殊權限進入點，其順序優先於任何其他進入點。 EntryPoint 指定的可執行檔通常是長時間執行的服務主機。 有個別設定的進入點，就不需要使用較高權限來長時間執行服務主機。 如需詳細資訊，請參閱 [SetupEntryPoint 元素](service-fabric-service-model-schema-elements.md#SetupEntryPointElementanonymouscomplexTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="exehost-element"></a>ExeHost 元素
 如需詳細資訊，請參閱 [ExeHost 元素](service-fabric-service-model-schema-elements.md#ExeHostElementExeHostEntryPointTypeComplexTypeDefinedInSetupEntryPointelement)

### <a name="program-element"></a>Program 元素
可執行檔名稱。  例如，"MySetup.bat" 或 "MyServiceHost.exe"。 如需詳細資訊，請參閱 [Program 元素](service-fabric-service-model-schema-elements.md#ProgramElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType)

### <a name="arguments-element"></a>Arguments 元素
 如需詳細資訊，請參閱 [Arguments 元素](service-fabric-service-model-schema-elements.md#ArgumentsElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType)

### <a name="workingfolder-element"></a>WorkingFolder 元素
應用程式部署所在的叢集節點上，所含程式碼套件中程序的工作目錄。 您可以指定三個值：Work (預設值)、CodePackage 或 CodeBase。 CodeBase 會指定要將工作目錄設定為 EXE 在程式碼套件中定義所在的目錄。 不論 EXE 是否定義在程式碼套件的目錄中，CodePackage 都會將工作目錄設定為程式碼套件的根目錄。 Work 會將工作目錄設定為節點上所建立的唯一資料夾。  整個應用程式執行個體的這個資料夾都會相同。 根據預設，應用程式中所有程序的工作目錄會設定為應用程式的工作資料夾。 程序可在此資料夾中寫入資料。 不建議在程式碼套件或程式碼基底中寫入資料，原因是這些資料夾可能會在不同的應用程式執行個體之間共用，並可能會遭到刪除。 如需詳細資訊，請參閱 [WorkingFolder 元素](service-fabric-service-model-schema-elements.md#WorkingFolderElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType)

### <a name="consoleredirection-element"></a>ConsoleRedirection 元素
警告！ 請勿在生產應用程式中使用主控台重新導向，您只能將它用於本機開發及偵錯。 請將啟動指令碼的主控台輸出重新導向至應用程式部署和執行所在叢集節點上「記錄」應用程式資料夾中的輸出檔案。 如需詳細資訊，請參閱 [ConsoleRedirection 元素](service-fabric-service-model-schema-elements.md#ConsoleRedirectionElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType)

### <a name="entrypoint-element"></a>EntryPoint 元素
EntryPoint 指定的可執行檔通常是長時間執行的服務主機。 有個別設定的進入點，就不需要使用較高權限來長時間執行服務主機。 EntryPoint 指定的可執行檔是在 SetupEntryPoint 成功結束之後執行。 如果曾經終止或當機，產生的程序會監視並重新啟動 (以 SetupEntryPoint再次開始)。 如需詳細資訊，請參閱 [EntryPoint 元素](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="exehost-element"></a>ExeHost 元素
 如需詳細資訊，請參閱 [ExeHost 元素](service-fabric-service-model-schema-elements.md#ExeHostElementanonymouscomplexTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType)

### <a name="configpackage-element"></a>ConfigPackage 元素
在包含 Settings.xml 檔案的 PackageRoot 底下，宣告以 Name 屬性命名的資料夾。 此檔案包含程序可以在執行階段讀回的使用者定義、成對的索引鍵/值設定等區段。 在升級期間，如果只有 ConfigPackage 版本已變更，則不會重新啟動執行中程序。 相反地，回呼會通知程序組態設定已變更，因此它們可以動態方式重新載入。 如需詳細資訊，請參閱 [ConfigPackage 元素](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement)

### <a name="resources-element"></a>Resources 元素
說明此服務所使用的資源，即可在不修改已編譯之程式碼的情況下加以宣告，並於服務已部署時加以變更。 這些資源的存取權會透過應用程式資訊清單的 Principals 和 Policies 區段來控制。 如需詳細資訊，請參閱 [Resources 元素](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="endpoints-element"></a>Endpoints 元素
定義服務的端點。 如需詳細資訊，請參閱 [Endpoints 元素](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType)

### <a name="endpoint-element"></a>Endpoint 元素
服務資訊清單中所宣告要覆寫的端點。 如需詳細資訊，請參閱 [Endpoint 元素](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement)



## <a name="votingdata-service-manifest-elements"></a>VotingData 服務資訊清單元素
### <a name="servicemanifest-element"></a>ServiceManifest 元素
以宣告方式說明服務類型和版本。 它會列出可獨立升級的程式碼、組態和資料套件，這些項目合在一起即可構成服務套件以支援一或多個服務類型。 它也會指定資源、診斷設定和服務中繼資料，例如服務類型、健康情況屬性和負載平衡計量。 如需詳細資訊，請參閱 [ServiceManifest 元素](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType)

### <a name="servicetypes-element"></a>ServiceTypes 元素
宣告此資訊清單中的 CodePackage 支援哪些服務類型。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 服務類型是在資訊清單層級而非程式碼套件層級宣告。 如需詳細資訊，請參閱 [ServiceTypes 元素](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="statefulservicetype-element"></a>StatefulServiceType 元素
說明具狀態服務類型。 如需詳細資訊，請參閱 [StatefulServiceType 元素](service-fabric-service-model-schema-elements.md#StatefulServiceTypeElementStatefulServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType)

### <a name="codepackage-element"></a>CodePackage 元素
說明可支援已定義之服務類型的程式碼套件。 當針對其中一種服務類型來具現化服務時，會藉由執行其進入點來啟動此資訊清單中宣告的所有程式碼封裝。 產生的處理程序預期在執行階段註冊支援的服務類型。 如果有多個程式碼套件，每當系統尋找任何一個宣告的服務類型時，它們都會啟動。 如需詳細資訊，請參閱 [CodePackage 元素](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement)

### <a name="entrypoint-element"></a>EntryPoint 元素
EntryPoint 指定的可執行檔通常是長時間執行的服務主機。 有個別設定的進入點，就不需要使用較高權限來長時間執行服務主機。 EntryPoint 指定的可執行檔是在 SetupEntryPoint 成功結束之後執行。 如果曾經終止或當機，產生的程序會監視並重新啟動 (以 SetupEntryPoint再次開始)。 如需詳細資訊，請參閱 [EntryPoint 元素](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType)

### <a name="exehost-element"></a>ExeHost 元素
 如需詳細資訊，請參閱 [ExeHost 元素](service-fabric-service-model-schema-elements.md#ExeHostElementanonymouscomplexTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType)

### <a name="program-element"></a>Program 元素
可執行檔名稱。  例如，"MySetup.bat" 或 "MyServiceHost.exe"。 如需詳細資訊，請參閱 [Program 元素](service-fabric-service-model-schema-elements.md#ProgramElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType)

### <a name="workingfolder-element"></a>WorkingFolder 元素
應用程式部署所在的叢集節點上，所含程式碼套件中程序的工作目錄。 您可以指定三個值：Work (預設值)、CodePackage 或 CodeBase。 CodeBase 會指定要將工作目錄設定為 EXE 在程式碼套件中定義所在的目錄。 不論 EXE 是否定義在程式碼套件的目錄中，CodePackage 都會將工作目錄設定為程式碼套件的根目錄。 Work 會將工作目錄設定為節點上所建立的唯一資料夾。  整個應用程式執行個體的這個資料夾都會相同。 根據預設，應用程式中所有程序的工作目錄會設定為應用程式的工作資料夾。 程序可在此資料夾中寫入資料。 不建議在程式碼套件或程式碼基底中寫入資料，原因是這些資料夾可能會在不同的應用程式執行個體之間共用，並可能會遭到刪除。 如需詳細資訊，請參閱 [WorkingFolder 元素](service-fabric-service-model-schema-elements.md#WorkingFolderElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType)

### <a name="configpackage-element"></a>ConfigPackage 元素
在包含 Settings.xml 檔案的 PackageRoot 底下，宣告以 Name 屬性命名的資料夾。 此檔案包含程序可以在執行階段讀回的使用者定義、成對的索引鍵/值設定等區段。 在升級期間，如果只有 ConfigPackage 版本已變更，則不會重新啟動執行中程序。 相反地，回呼會通知程序組態設定已變更，因此它們可以動態方式重新載入。 如需詳細資訊，請參閱 [ConfigPackage 元素](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement)

### <a name="datapackage-element"></a>DataPackage 元素
在 PackageRoot (包含要由程序在執行階段取用的靜態資料檔案) 底下宣告以 Name 屬性命名的資料夾。 當服務資訊清單中列出的資料套件有任何一個已進行升級，Service Fabric 就會回收主機和支援套件中指定的所有 EXE 和 DLLHOST。 如需詳細資訊，請參閱 [DataPackage 元素](service-fabric-service-model-schema-elements.md#DataPackageElementDataPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedDataPackageelement)

### <a name="resources-element"></a>Resources 元素
說明此服務所使用的資源，即可在不修改已編譯之程式碼的情況下加以宣告，並於服務已部署時加以變更。 這些資源的存取權會透過應用程式資訊清單的 Principals 和 Policies 區段來控制。 如需詳細資訊，請參閱 [Resources 元素](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType)

### <a name="endpoints-element"></a>Endpoints 元素
定義服務的端點。 如需詳細資訊，請參閱 [Endpoints 元素](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType)

### <a name="endpoint-element"></a>Endpoint 元素
服務資訊清單中所宣告要覆寫的端點。 如需詳細資訊，請參閱 [Endpoint 元素](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement)

