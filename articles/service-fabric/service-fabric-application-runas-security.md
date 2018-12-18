---
title: 在系統和本機安全性帳戶下執行 Azure Service Fabric 服務 | Microsoft Docs
description: 了解如何在系統和本機安全性帳戶下執行 Service Fabric 應用程式。  建立安全性主體並套用「執行身分」原則以安全地執行您的服務。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: 4242a1eb-a237-459b-afbf-1e06cfa72732
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/29/2018
ms.author: mfussell
ms.openlocfilehash: 33ca23834f35e631c6943ec22a88f4fe3dc853e1
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212393"
---
# <a name="run-a-service-as-a-local-user-account-or-local-system-account"></a>以本機使用者帳戶或本機系統帳戶身分執行服務
藉由使用 Azure Service Fabric，您便可以保護在叢集中以不同使用者帳戶執行的應用程式。 根據預設，Service Fabric 應用程式會在用以執行 Fabric.exe 程序的帳戶之下執行。 Service Fabric 也會提供在本機使用者或系統帳戶底下執行應用程式的功能。 支援的本機系統帳戶類型為 **LocalUser**、**NetworkService**、**LocalService** 和 **LocalSystem**。  如果您在 Windows 獨立叢集上執行 Service Fabric，便可以在 [Active Directory 網域帳戶](service-fabric-run-service-as-ad-user-or-group.md)或[群組受控服務帳戶](service-fabric-run-service-as-gmsa.md)底下執行服務。

在應用程式資訊清單中，您要定義**主體**一節中執行服務或安全資源所需的使用者帳戶。 您也可以定義和建立使用者群組，以便您可以一起管理一或多個使用者。 當不同的服務進入點有多個使用者，而且他們需要可在群組層級取得的常見權限時，這會很有用。  接著會在 RunAs 原則中參考使用者，其會套用至特定服務或應用程式中的所有服務。 

依預設，RunAs 原則會套用至主要進入點。  如果您需要[在系統帳戶下執行某些高特殊權限安裝程式作業](service-fabric-run-script-at-service-startup.md)，也可以將 RunAs 原則套用至安裝程式進入點，或是套用至主要和安裝程式進入點。  

> [!NOTE] 
> 如果您將 RunAs 原則套用到服務，而服務資訊清單宣告具有 HTTP 通訊協定的端點資源，您就必須指定 **SecurityAccessPolicy**。  如需詳細資訊，請參閱[為 HTTP 和 HTTPS 端點指派安全性存取原則](service-fabric-assign-policy-to-endpoint.md)。 
>

## <a name="run-a-service-as-a-local-user"></a>以本機使用者身分執行服務
您可以建立一個本機使用者，以便用來協助保護應用程式內的服務。 在應用程式資訊清單的主體區段中指定 **LocalUser** 帳戶類型時，Service Fabric 會在部署應用程式所在的電腦上建立本機使用者帳戶。 根據預設，這些帳戶的名稱不會與應用程式資訊清單中所指定的名稱相同 (例如，以下應用程式資訊清單範例中的 Customer3)。 相反地，它們會動態產生並具有隨機密碼。

在 **ServiceManifestImport** 的 **RunAsPolicy** 區段中，指定**主體**區段中的使用者帳戶來執行服務程式碼套件。  下列範例會示範如何建立本機使用者，以及將 RunAs 原則套用至主要進入點：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer3" EntryPointType="Main" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>    
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="Customer3" />
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="create-a-local-user-group"></a>建立本機使用者群組
您可以建立使用者群組，並將一或多個使用者新增至群組。 當不同的服務進入點有多個使用者，而他們需要具備可在群組層級取得的某些通用權限時，這會很有用。 下列應用程式資訊清單範例會顯示名為 LocalAdminGroup 且具有系統管理員權限的本機群組。 Customer1 和 Customer2 這兩個使用者會成為此本機群組的成員。 在 **ServiceManifestImport** 區段中，會套用 RunAs 原則以 Customer2 的身分執行 Stateful1Pkg 程式碼套件。  另一個 RunAs 原則會套用來以 Customer1 的身分執行 Web1Pkg 程式碼套件。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer2" EntryPointType="Main"/>
    </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" EntryPointType="Main"/>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="Stateful1" ServicePackageActivationMode="ExclusiveProcess">
      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
      </StatefulService>
    </Service>
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Groups>
      <Group Name="LocalAdminGroup">
        <Membership>
          <SystemGroup Name="Administrators" />
        </Membership>
      </Group>
    </Groups>
    <Users>
      <User Name="Customer1">
        <MemberOf>
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <User Name="Customer2">
        <MemberOf>
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="apply-a-default-policy-to-all-service-code-packages"></a>將預設原則套用到所有的服務程式碼封裝
您使用 **DefaultRunAsPolicy** 區段來針對未定義特定 **RunAsPolicy** 的所有程式碼套件，指定預設的使用者帳戶。 如果在應用程式所使用的服務資訊清單中指定的大部分程式碼封裝必須以相同的使用者身分執行，則應用程式可以只使用該使用者帳戶定義預設的 RunAs 原則。 下列範例指定某個程式碼套件未指定 **RunAsPolicy** 時，該程式碼套件應該以主體區段中指定的 **MyDefaultAccount** 使用者執行。  支援的帳戶類型為 LocalUser、NetworkService、LocalSystem 和 LocalService。  如果是使用本機使用者或服務，也請指定帳戶名稱和密碼。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    
  </ServiceManifestImport>
  <DefaultServices>    
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="MyDefaultAccount" AccountType="NetworkService" />      
    </Users>
  </Principals>
  <Policies>
    <DefaultRunAsPolicy UserRef="MyDefaultAccount" />
  </Policies>
</ApplicationManifest>
```

## <a name="debug-a-code-package-locally-using-console-redirection"></a>使用主控台重新導向在本機進行程式碼套件偵錯
基於偵錯的目的，查看執行服務後的主控台輸出偶爾會有幫助。 您可以在服務資訊清單中的進入點上設定主控台重新導向原則，其會將輸出寫入至檔案。 檔案輸出會寫入至部署並執行應用程式所在的叢集節點上，稱為 **log** 的應用程式資料夾中。 

> [!WARNING]
> 切勿在實際部署的應用程式中使用主控台重新導向原則，因為這可能會影響應用程式容錯移轉。 僅將此原則用於本機開發及偵錯。  
> 
> 

下列服務資訊清單範例示範如何使用 FileRetentionCount 值啟用主控台重新導向：

```xml
<CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
        <ConsoleRedirection FileRetentionCount="10"/>
      </ExeHost>
    </EntryPoint>
</CodePackage>

```

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>後續步驟
* [了解應用程式模型](service-fabric-application-model.md)
* [在服務資訊清單中指定資源](service-fabric-service-manifest-resources.md)
* [部署應用程式](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
