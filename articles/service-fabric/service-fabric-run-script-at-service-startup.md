---
title: 在 Azure Service Fabric 服務啟動時執行指令碼 | Microsoft Docs
description: 了解如何設定 Service Fabric 服務安裝程式進入點的原則，並在服務啟動期間執行指令碼。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/21/2018
ms.author: mfussell
ms.openlocfilehash: 3fe22d8bb52fa5f45ce5f1cdc7b860d1ce295a71
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34210489"
---
# <a name="run-a-service-startup-script-as-a-local-user-or-system-account"></a>以本機使用者或系統帳戶身分執行服務啟動指令碼
在 Service Fabric 服務可執行檔啟動之前，可能需要執行一些設定或安裝工作。  例如，設定環境變數。 您可以在服務可執行檔於服務的服務資訊清單中啟動之前，指定要執行的指令碼。 藉由設定服務安裝程式進入點的 RunAs 原則，您可以變更安裝程式的可執行檔要在哪一個帳戶下執行。  個別的安裝程式進入點可讓您在短時間內執行高權限設定，因此，服務主機的可執行檔不需要長時間使用高權限來執行。

安裝程式進入點 ([服務資訊清單](service-fabric-application-and-service-manifests.md)中的 **SetupEntryPoint**) 是預設以與 Service Fabric 相同的認證執行的特殊權限進入點 (通常為 *NetworkService* 帳戶)，優先於其他任何進入點。 **EntryPoint** 指定的可執行檔通常是長時間執行的服務主機。 **EntryPoint** 可執行檔會在 **SetupEntryPoint** 可執行檔成功結束之後執行。 產生的程序會受到監視，萬一終止或當機，則會同樣從 **SetupEntryPoint** 開始來重新啟動。 

## <a name="configure-the-service-setup-entry-point"></a>設定服務安裝程式進入點
以下是適用於無狀態服務的簡單服務資訊清單範例，其會在服務 **SetupEntryPoint** 中指定安裝指令碼 *MySetup.bat*。  **引數**可用來在指令碼執行時，將引數傳遞至其中。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="MyStatelessServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Description>An example service manifest.</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="MyStatelessServiceType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>MySetup.bat</Program>
        <Arguments>MyValue</Arguments>
        <WorkingFolder>Work</WorkingFolder>        
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>MyStatelessService.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```
## <a name="configure-the-policy-for-a-service-setup-entry-point"></a>設定服務安裝程式進入點的原則
依預設，服務安裝程式進入點的可執行檔會在與 Service Fabric 相同的認證 (通常為 *NetworkService* 帳戶) 下執行。  在應用程式資訊清單中，您可以變更安全性權限，在本機系統帳戶或系統管理員帳戶下執行啟動指令碼。

### <a name="configure-the-policy-by-using-a-local-system-account"></a>使用本機系統帳戶設定原則
下列應用程式資訊清單範例示範如何將服務安裝程式進入點設定為在使用者系統管理員帳戶 (SetupAdminUser) 下執行。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="MyStatelessService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyStatelessServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="MyStatelessService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyStatelessServiceType" InstanceCount="[MyStatelessService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

首先，以使用者名稱 (例如 SetupAdminUser) 建立 **Principals** 區段。 SetupAdminUser 使用者帳戶是 Administrators 系統群組的成員。

接著在 **ServiceManifestImport** 區段下方設定原則，以將此主體套用至 **SetupEntryPoint**。 此原則會告知 Service Fabric，當 **MySetup.bat** 檔案執行時，應該以 SetupAdminUser (具備系統管理員權限) 身分執行。 由於您「尚未」將原則套用至主要進入點，因此，**MyServiceHost.exe** 中的程式碼會在系統 **NetworkService** 帳戶下執行。 這是用來執行所有服務進入點的預設帳戶。

### <a name="configure-the-policy-by-using-local-system-accounts"></a>使用本機系統帳戶設定原則
通常最好是使用本機系統帳戶 (而不是系統管理員帳戶) 執行啟動指令碼。 使用 Administrators 群組成員的身分執行 RunAs 原則通常無法正常運作，因為電腦預設會啟用使用者存取控制 (UAC)。 在此情況下，建議以 LocalSystem 身分 (而不是以新增到 Administrators 群組的本機使用者身分) 執行 SetupEntryPoint。 下列範例示範如何設定 SetupEntryPoint 以 LocalSystem 身分執行：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="MyStatelessService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyStatelessServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
         <RunAsPolicy CodePackageRef="Code" UserRef="SetupLocalSystem" EntryPointType="Setup" />
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="MyStatelessService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyStatelessServiceType" InstanceCount="[MyStatelessService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
      <Users>
         <User Name="SetupLocalSystem" AccountType="LocalSystem" />
      </Users>
   </Principals>
</ApplicationManifest>
```

> [!NOTE]
> 對於 Linux 叢集，若要以 **root** 身分執行服務或安裝程式進入點，您可以將 **AccountType** 指定為 **LocalSystem**。

## <a name="run-a-script-from-the-setup-entry-point"></a>從安裝程式進入點執行指令碼
現在將啟動指令碼新增至專案，並以系統管理員權限執行。 

在 Visual Studio 中，以滑鼠右鍵按一下服務專案，並新增名為 *MySetup.bat* 的新檔案。

接下來，確定 *MySetup.bat* 檔案已包含於服務套件中。 預設不會包含該檔案。 選取該檔案、以滑鼠右鍵按一下操作功能表，然後選擇 [屬性] 。 在 [屬性] 對話方塊中，確定已將 [複製到輸出目錄] 設為 [有更新時才複製]。 請參閱下列螢幕擷取畫面。

![Visual Studio CopyToOutput for SetupEntryPoint 批次檔][image1]

現在編輯 *MySetup.bat* 檔案，然後新增下列命令來設定系統環境變數並輸出文字檔：

```
REM Set a system environment variable. This requires administrator privilege
setx -m TestVariable %*
echo System TestVariable set to > out.txt
echo %TestVariable% >> out.txt

REM To delete this system variable us
REM REG delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v TestVariable /f
```

接下來，建置解決方案並部署至本機開發叢集。 啟動服務之後 (如 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 中所示)，您就可以透過兩種方式來查看 MySetup.bat 檔案成功。 開啟 PowerShell 命令提示字元並輸入：

```
PS C:\ [Environment]::GetEnvironmentVariable("TestVariable","Machine")
MyValue
```

接著，記下已在 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 中部署並啟動服務的節點名稱。 例如，節點 2。 接下來，瀏覽至應用程式執行個體工作資料夾，尋出 out.txt 檔案，其中顯示 **TestVariable**的值。 例如，如果此服務已部署至節點 2，則您可以移至 **MyApplicationType** 的這個路徑：

```
C:\SfDevCluster\Data\_App\Node.2\MyApplicationType_App\work\out.txt
```

## <a name="run-powershell-commands-from-a-setup-entry-point"></a>從安裝程式進入點執行 PowerShell 命令
若要從 **SetupEntryPoint** 點執行 PowerShell，您可以在指向 PowerShell 檔案的批次檔中執行 **PowerShell.exe**。 首先，將 PowerShell 檔案新增至服務專案，例如 **MySetup.ps1**。 請記得設定 [有更新時才複製]  屬性，讓這個檔案也會包含於服務封裝內。 下列範例顯示的範例批次檔可啟動名為 MySetup.ps1 的 PowerShell 檔案，以設定名為 **TestVariable** 的系統環境變數。

MySetup.bat 可啟動 PowerShell 檔案：

```
powershell.exe -ExecutionPolicy Bypass -Command ".\MySetup.ps1"
```

在 PowerShell 檔案中，加入以下項目來設定系統環境變數：

```
[Environment]::SetEnvironmentVariable("TestVariable", "MyValue", "Machine")
[Environment]::GetEnvironmentVariable("TestVariable","Machine") > out.txt
```

> [!NOTE]
> 當批次檔執行時，預設會在稱為 **work** 的應用程式資料夾中查看檔案。 在此情況下，當 MySetup.bat 執行時，我們希望它會在相同資料夾 (也就是應用程式的 **code package** 資料夾) 中尋找 MySetup.ps1 檔。 若要變更此資料夾，請設定工作資料夾：
> 
> 

```xml
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    </ExeHost>
</SetupEntryPoint>
```

## <a name="debug-a-startup-script-locally-using-console-redirection"></a>使用主控台重新導向在本機進行啟動指令碼偵錯
基於偵錯的目的，查看執行安裝指令碼後的主控台輸出偶爾會有幫助。 您可以在服務資訊清單中的安裝程式進入點上設定主控台重新導向原則，其會將輸出寫入至檔案。 檔案輸出會寫入至部署並執行應用程式所在的叢集節點上，稱為 **log** 的應用程式資料夾中。 

> [!WARNING]
> 切勿在實際部署的應用程式中使用主控台重新導向原則，因為這可能會影響應用程式容錯移轉。 僅將此原則用於本機開發及偵錯。  
> 
> 

下列服務資訊清單範例示範如何使用 FileRetentionCount 值設定主控台重新導向：

```xml
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="10"/>
    </ExeHost>
</SetupEntryPoint>
```

如果您現在變更 MySetup.ps1 檔案來撰寫 **Echo** 命令，這將會寫入至輸出檔案以進行偵錯：

```
Echo "Test console redirection which writes to the application log folder on the node that the application is deployed to"
```

> [!WARNING]
> 當您為指令碼偵錯之後，請立即移除這個主控台重新導向原則。



<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>後續步驟
* [了解應用程式及服務安全性](service-fabric-application-and-service-security.md)
* [了解應用程式模型](service-fabric-application-model.md)
* [在服務資訊清單中指定資源](service-fabric-service-manifest-resources.md)
* [部署應用程式](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
