---
title: 什麼是雲端服務模型和封裝 | Microsoft Docs
description: 說明 Azure 中的雲端服務模型 (.csdef、.cscfg) 和封裝 (.cspkg)
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 4ce2feb5-0437-496c-98da-1fb6dcb7f59e
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/05/2017
ms.author: jeconnoc
ms.openlocfilehash: 90d810916599db50249a3e2ec677046c5af42a09
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39005844"
---
# <a name="what-is-the-cloud-service-model-and-how-do-i-package-it"></a>什麼是雲端服務模型？如何封裝？
雲端服務是從三個元件建立的，也就是服務定義 (.csdef)、服務組態 (.cscfg) 和服務封裝 (.cspkg)。 **ServiceDefinition.csdef** 和 **ServiceConfig.cscfg** 這兩個檔案是以 XML 為基礎，描述雲端服務的結構及其設定方式，統稱為模型。 **ServicePackage.cspkg** 是從 **ServiceDefinition.csdef** 產生的 zip 檔案，此外，包含所有必要的二進位型相依性。 Azure 會從 **ServicePackage.cspkg** 和 **ServiceConfig.cscfg** 建立雲端服務。

一旦雲端服務在 Azure 中執行之後，您就可以透過 **ServiceConfig.cscfg** 檔案重新設定它，但您無法改變定義。

## <a name="what-would-you-like-to-know-more-about"></a>您想要深入了解什麼？
* 我想要深入了解 [ServiceDefinition.csdef](#csdef) 和 [ServiceConfig.cscfg](#cscfg) 檔案。
* 我已經了解，請給我 [一些範例](#next-steps) ，讓我可以設定。
* 我想要建立 [ServicePackage.cspkg](#cspkg)。
* 我打算使用 Visual Studio，而我想要...
  * [建立雲端服務][vs_create]
  * [重新設定現有的雲端服務][vs_reconfigure]
  * [部署雲端服務專案][vs_deploy]
  * [從遠端桌面進入雲端服務執行個體][remotedesktop]

<a name="csdef"></a>

## <a name="servicedefinitioncsdef"></a>ServiceDefinition.csdef
**ServiceDefinition.csdef** 檔案會指定 Azure 所使用的設定來設定雲端服務。 [Azure 服務定義結構描述 (.csdef 檔)](https://msdn.microsoft.com/library/azure/ee758711.aspx) 會為服務定義檔提供允許的格式。 以下範例顯示可以針對 Web 和背景工作角色定義的設定：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="MyServiceName" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WebRole name="WebRole1" vmsize="Medium">
    <Sites>
      <Site name="Web">
        <Bindings>
          <Binding name="HttpIn" endpointName="HttpIn" />
        </Bindings>
      </Site>
    </Sites>
    <Endpoints>
      <InputEndpoint name="HttpIn" protocol="http" port="80" />
      <InternalEndpoint name="InternalHttpIn" protocol="http" />
    </Endpoints>
    <Certificates>
      <Certificate name="Certificate1" storeLocation="LocalMachine" storeName="My" />
    </Certificates>
    <Imports>
      <Import moduleName="Connect" />
      <Import moduleName="Diagnostics" />
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <LocalResources>
      <LocalStorage name="localStoreOne" sizeInMB="10" />
      <LocalStorage name="localStoreTwo" sizeInMB="10" cleanOnRoleRecycle="false" />
    </LocalResources>
    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" />
    </Startup>
  </WebRole>

  <WorkerRole name="WorkerRole1">
    <ConfigurationSettings>
      <Setting name="DiagnosticsConnectionString" />
    </ConfigurationSettings>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="tcp" port="10000" />
      <InternalEndpoint name="Endpoint2" protocol="tcp" />
    </Endpoints>
  </WorkerRole>
</ServiceDefinition>
```

您可以參考[服務定義結構描述](https://msdn.microsoft.com/library/azure/ee758711.aspx)，進一步了解此處所使用的 XML 結構描述，不過，以下是某些元素的簡短說明：

**Sites**  
包含 IIS7 中所裝載的網站或 Web 應用程式的定義。

**InputEndpoints**  
包含用來連絡雲端服務的端點的定義。

**InternalEndpoints**  
包含角色執行個體用來彼此通訊的端點的定義。

**ConfigurationSettings**  
包含特定角色功能的設定定義。

**Certificates**  
包含角色所需的憑證的定義。 上述程式碼範例顯示用於設定 Azure Connect 的憑證。

**LocalResources**  
包含本機儲存資源的定義。 本機儲存資源是執行中角色執行個體所在之虛擬機器的檔案系統上的保留目錄。

**Imports**  
包含匯入的模組的定義。 上述程式碼範例顯示遠端桌面連線與 Azure Connect 的模組。

**Startup**  
包含角色啟動時執行的工作。 這些工作是在 .cmd 或可執行檔中定義。

<a name="cscfg"></a>

## <a name="serviceconfigurationcscfg"></a>ServiceConfiguration.cscfg
雲端服務設定的組態取決於 **ServiceConfiguration.cscfg** 檔案中的值。 您可以指定您想要在此檔案中為每個角色部署的執行個體的數目。 您在服務定義檔中定義的組態設定的值會加入至服務組態檔。 與雲端服務相關聯的任何管理憑證的指紋也會加入至檔案。 [Azure 服務組態結構描述 (.cscfg 檔)](https://msdn.microsoft.com/library/azure/ee758710.aspx) 為服務組態檔提供允許的格式。

服務組態檔沒有與應用程式封裝在一起，但是做為個別的檔案上傳至 Azure，並用來設定雲端服務。 您可以上傳新的服務組態檔，無需重新部署您的雲端服務。 雲端服務執行時，可以變更雲端服務的組態值。 以下範例顯示可以針對 Web 和背景工作角色定義的組態設定：

```xml
<?xml version="1.0"?>
<ServiceConfiguration serviceName="MyServiceName" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration">
  <Role name="WebRole1">
    <Instances count="2" />
    <ConfigurationSettings>
      <Setting name="SettingName" value="SettingValue" />
    </ConfigurationSettings>

    <Certificates>
      <Certificate name="CertificateName" thumbprint="CertThumbprint" thumbprintAlgorithm="sha1" />
      <Certificate name="Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption"
         thumbprint="CertThumbprint" thumbprintAlgorithm="sha1" />
    </Certificates>
  </Role>
</ServiceConfiguration>
```

您可以參考 [服務組態結構描述](https://msdn.microsoft.com/library/azure/ee758710.aspx) ，進一步了解此處所使用的 XML 結構描述，不過，以下是元素的簡短說明：

**Instances**  
設定執行中角色執行個體的數目。 為防止您的雲端服務在升級期間可能變成無法使用，建議您部署多個 Web 對向角色執行個體。 部署多個執行個體的做法符合 [Azure 計算服務等級協定 (SLA)](http://azure.microsoft.com/support/legal/sla/)中的指導方針，當您為服務部署兩個或更多個角色執行個體時，此等級協定可保證網際網路對向角色有 99.95% 的外部連線能力。

**ConfigurationSettings**  
設定執行中角色執行個體的設定。 `<Setting>` 元素的名稱必須符合服務定義檔中的設定定義。

**Certificates**  
設定服務所使用的憑證。 上述程式碼範例顯示如何定義 RemoteAccess 模組的憑證。 *thumbprint* 屬性的值必須設定為要使用的憑證的指紋。

<p/>

> [!NOTE]
> 您可以使用文字編輯器，將憑證指紋新增至組態檔。 或者，在 Visual Studio 中，也可以在角色 [屬性] 頁面的 [憑證] 索引標籤上新增值。
> 
> 

## <a name="defining-ports-for-role-instances"></a>定義角色執行個體的連接埠
Azure 對於 Web 角色，僅允許一個進入點。 這表示所有流量都是透過一個 IP 位址發生。 您可以設定您的網站共用連接埠，方法是設定主機標頭，將要求導向到正確的位置。 您也可以設定您的應用程式接聽 IP 位址的公認連接埠。

以下範例顯示具有網站及 Web 應用程式的 Web 角色的組態。 網站會設定為連接埠 80 上的預設進入位置，而 Web 應用程式則會設定為從稱為 "mail.mysite.cloudapp.net" 的替代主機標頭接收要求。

```xml
<WebRole>
  <ConfigurationSettings>
    <Setting name="DiagnosticsConnectionString" />
  </ConfigurationSettings>
  <Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
    <InputEndpoint name="Https" protocol="https" port="443" certificate="SSL"/>
    <InputEndpoint name="NetTcp" protocol="tcp" port="808" certificate="SSL"/>
  </Endpoints>
  <LocalResources>
    <LocalStorage name="Sites" cleanOnRoleRecycle="true" sizeInMB="100" />
  </LocalResources>
  <Site name="Mysite" packageDir="Sites\Mysite">
    <Bindings>
      <Binding name="http" endpointName="HttpIn" />
      <Binding name="https" endpointName="Https" />
      <Binding name="tcp" endpointName="NetTcp" />
    </Bindings>
  </Site>
  <Site name="MailSite" packageDir="MailSite">
    <Bindings>
      <Binding name="mail" endpointName="HttpIn" hostheader="mail.mysite.cloudapp.net" />
    </Bindings>
    <VirtualDirectory name="artifacts" />
    <VirtualApplication name="storageproxy">
      <VirtualDirectory name="packages" packageDir="Sites\storageProxy\packages"/>
    </VirtualApplication>
  </Site>
</WebRole>
```


## <a name="changing-the-configuration-of-a-role"></a>變更角色的組態
您可以在雲端服務於 Azure 中執行的同時，更新該雲端服務的組態，而不讓服務離線。 若要變更組態資訊，您可以上傳新的組態檔，或就地編輯現有的組態檔，並將其套用到執行中的服務。 您可以針對服務的組態進行下列變更：

* **變更組態設定的值**  
  ：當組態設定變更時，角色執行個體可以選擇在執行個體上線時套用變更，或是正常回收執行個體，並在執行個體離線時套用變更。
* **變更角色執行個體的服務拓撲**  
  ：拓撲變更不會影響執行中的執行個體，但是要移除的執行個體除外。 其餘所有執行個體通常不需要進行回收，不過，您可以選擇回收角色執行個體，以回應拓撲變更。
* **變更憑證指紋**  
  ：當角色執行個體離線時，您可以只更新憑證。 如果在角色執行個體上線時加入、刪除或變更憑證，Azure 會讓執行個體正常離線以更新憑證，並在變更完成後讓它再次上線。

### <a name="handling-configuration-changes-with-service-runtime-events"></a>使用服務執行階段事件處理組態變更
[Azure 執行階段程式庫](https://msdn.microsoft.com/library/azure/mt419365.aspx)包含 [Microsoft.WindowsAzure.ServiceRuntime](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.aspx) 命名空間，其中提供從角色來與 Azure 環境互動的類別。 [RoleEnvironment](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx) 類別會定義組態變更前後會引發的下列事件：

* **[Changing](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.changing.aspx) 事件**  
  這會在組態變更套用至指定的角色執行個體之前發生，讓您有機會記下角色執行個體 (如有需要)。
* **[Changed](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.changed.aspx) 事件**  
  ：在組態變更套用至指定的角色執行個體之後發生。

> [!NOTE]
> 由於憑證變更時，一律會讓角色執行個體離線，因此不會引發 RoleEnvironment.Changing 或 RoleEnvironment.Changed 事件。
> 
> 

<a name="cspkg"></a>

## <a name="servicepackagecspkg"></a>ServicePackage.cspkg
若要將應用程式部署為 Azure 中的雲端服務，您必須先使用適當的格式封裝應用程式。 您可以使用 **CSPack** 命令列工具 (隨 [Azure SDK](https://azure.microsoft.com/downloads/)安裝) 做為 Visual Studio 的替代方案，以建立封裝檔案。

**CSPack** 會使用服務定義檔和服務組態檔的內容，定義封裝的內容。 **CSPack** 會產生應用程式封裝檔案 (.cspkg)，您可以使用 [Azure 入口網站](cloud-services-how-to-create-deploy-portal.md#create-and-deploy)，將其上傳至 Azure。 根據預設，套件的名稱為 `[ServiceDefinitionFileName].cspkg`，但是您可以使用 **CSPack** 的 `/out` 選項指定不同的名稱。

**CSPack** 位於  
`C:\Program Files\Microsoft SDKs\Azure\.NET SDK\[sdk-version]\bin\`

> [!NOTE]
> CSPack.exe (在 Windows 上) 可以透過執行隨 SDK 一起安裝的 **Microsoft Azure 命令提示字元** 捷徑來使用。  
> 
> 執行 CSPack.exe 程式本身，以查看所有可能的參數和命令的相關文件。
> 
> 

<p />

> [!TIP]
> 在本機於 **Microsoft Azure 計算模擬器**中執行雲端服務，並使用 **/copyonly** 選項。 此選項會將應用程式的二進位檔案複製到目錄配置中，在計算模擬器中就可以從那裡執行它們。
> 
> 

### <a name="example-command-to-package-a-cloud-service"></a>封裝雲端服務的範例命令
下列範例會建立應用程式封裝，其中包含 Web 角色的資訊。 此命令會指定要使用的服務定義檔、可以找到二進位檔案所在的目錄，以及封裝檔案的名稱。

```cmd
cspack [DirectoryName]\[ServiceDefinition]
       /role:[RoleName];[RoleBinariesDirectory]
       /sites:[RoleName];[VirtualPath];[PhysicalPath]
       /out:[OutputFileName]
```

如果應用程式同時包含 Web 角色和背景工作角色，則會使用下列命令：

```cmd
cspack [DirectoryName]\[ServiceDefinition]
       /out:[OutputFileName]
       /role:[RoleName];[RoleBinariesDirectory]
       /sites:[RoleName];[VirtualPath];[PhysicalPath]
       /role:[RoleName];[RoleBinariesDirectory];[RoleAssemblyName]
```

其中的變數定義如下：

| 變數 | 值 |
| --- | --- |
| \[DirectoryName\] |專案根目錄底下的子目錄，其中包含 Azure 專案的 .csdef 檔案。 |
| \[ServiceDefinition\] |服務定義檔的名稱。 根據預設，此檔案的名稱為 ServiceDefinition.csdef。 |
| \[OutputFileName\] |所產生的封裝檔案的名稱。 一般而言，這是設定為應用程式的名稱。 如果沒有指定檔案名稱，就會將應用程式封裝建立為 \[ApplicationName\].cspkg。 |
| \[RoleName\] |服務定義檔中所定義的角色名稱。 |
| \[RoleBinariesDirectory] |角色的二進位檔案的位置。 |
| \[VirtualPath\] |在服務定義的 Sites 區段中定義的每個虛擬路徑的實體目錄。 |
| \[PhysicalPath\] |在服務定義的 site 節點中定義的每個虛擬路徑內容的實體目錄。 |
| \[RoleAssemblyName\] |角色的二進位檔案的名稱。 |

## <a name="next-steps"></a>後續步驟
我打算建立雲端服務封裝，而且我想要...

* [設定雲端服務執行個體的遠端桌面][remotedesktop]
* [部署雲端服務專案][deploy]

我打算使用 Visual Studio，而我想要...

* [建立新的雲端服務][vs_create]
* [重新設定現有的雲端服務][vs_reconfigure]
* [部署雲端服務專案][vs_deploy]
* [設定雲端服務執行個體的遠端桌面][vs_remote]

[deploy]: cloud-services-how-to-create-deploy-portal.md
[remotedesktop]: cloud-services-role-enable-remote-desktop-new-portal.md
[vs_remote]: cloud-services-role-enable-remote-desktop-visual-studio.md
[vs_deploy]: ../vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio.md
[vs_reconfigure]: ../vs-azure-tools-configure-roles-for-cloud-service.md
[vs_create]: ../vs-azure-tools-azure-project-create.md
