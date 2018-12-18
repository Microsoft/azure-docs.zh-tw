---
title: Azure 雲端服務定義。WebRole 結構描述 | Microsoft Docs
ms.custom: ''
ms.date: 04/14/2015
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: 85368e4e-a0db-4c02-8dbc-8e2928fa6091
caps.latest.revision: 60
author: jpconnock
ms.author: jeconnoc
manager: timlt
ms.openlocfilehash: e548841f334705aa71ada92c43ccde207a1f6318
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39002308"
---
# <a name="azure-cloud-services-definition-webrole-schema"></a>Azure 雲端服務定義 WebRole 結構描述
Azure Web 角色是專為 IIS 7 所支援之 Web 應用程式的程式設計 (例如，ASP.NET、PHP、Windows Communication Foundation 及 FastCGI) 所自訂的角色。

服務定義檔的預設副檔名為 .csdef。

## <a name="basic-service-definition-schema-for-a-web-role"></a>Web 角色的基本服務定義結構描述  
包含 Web 角色之服務定義檔的基本格式如下所示。

```xml
<ServiceDefinition …>  
  <WebRole name="<web-role-name>" vmsize="<web-role-size>" enableNativeCodeExecution="[true|false]">  
    <Certificates>  
      <Certificate name="<certificate-name>" storeLocation="<certificate-store>" storeName="<store-name>" />  
    </Certificates>      
    <ConfigurationSettings>  
      <Setting name="<setting-name>" />  
    </ConfigurationSettings>  
    <Imports>  
      <Import moduleName="<import-module>"/>  
    </Imports>  
    <Endpoints>  
      <InputEndpoint certificate="<certificate-name>" ignoreRoleInstanceStatus="[true|false]" name="<input-endpoint-name>" protocol="[http|https|tcp|udp]" localPort="<port-number>" port="<port-number>" loadBalancerProbe="<load-balancer-probe-name>" />  
      <InternalEndpoint name="<internal-endpoint-name>" protocol="[http|tcp|udp|any]" port="<port-number>">  
         <FixedPort port="<port-number>"/>  
         <FixedPortRange min="<minium-port-number>" max="<maximum-port-number>"/>  
      </InternalEndpoint>  
     <InstanceInputEndpoint name="<instance-input-endpoint-name>" localPort="<port-number>" protocol="[udp|tcp]">  
         <AllocatePublicPortFrom>  
            <FixedPortRange min="<minium-port-number>" max="<maximum-port-number>"/>  
         </AllocatePublicPortFrom>  
      </InstanceInputEndpoint>  
    </Endpoints>  
    <LocalResources>  
      <LocalStorage name="<local-store-name>" cleanOnRoleRecycle="[true|false]" sizeInMB="<size-in-megabytes>" />  
    </LocalResources>  
    <LocalStorage name="<local-store-name>" cleanOnRoleRecycle="[true|false]" sizeInMB="<size-in-megabytes>" />  
    <Runtime executionContext="[limited|elevated]">  
      <Environment>  
         <Variable name="<variable-name>" value="<variable-value>">  
            <RoleInstanceValue xpath="<xpath-to-role-environment-settings>"/>  
          </Variable>            
      </Environment>  
      <EntryPoint>  
         <NetFxEntryPoint assemblyName="<name-of-assembly-containing-entrypoint>" targetFrameworkVersion="<.net-framework-version>"/>  
      </EntryPoint>  
    </Runtime>  
    <Sites>  
      <Site name="<web-site-name>">  
        <VirtualApplication name="<application-name>" physicalDirectory="<directory-path>"/>  
        <VirtualDirectory name="<directory-path>" physicalDirectory="<directory-path>"/>  
        <Bindings>  
          <Binding name="<binding-name>" endpointName="<endpoint-name-bound-to>" hostHeader="<url-of-the-site>"/>  
        </Bindings>  
      </Site>  
    </Sites>  
    <Startup priority="<for-internal-use-only>">  
      <Task commandLine="<command-to=execute>" executionContext="[limited|elevated]" taskType="[simple|foreground|background]">  
        <Environment>  
         <Variable name="<variable-name>" value="<variable-value>">  
            <RoleInstanceValue xpath="<xpath-to-role-environment-settings>"/>  
          </Variable>            
        </Environment>  
      </Task>  
    </Startup>  
    <Contents>  
      <Content destination="<destination-folder-name>" >  
        <SourceDirectory path="<local-source-directory>" />  
      </Content>  
    </Contents>  
  </WebRole>  
</ServiceDefinition>  
```  

## <a name="schema-elements"></a>結構描述元素  
服務定義檔包含下列元素，本主題的後續幾節會有這些元素的詳細說明：  

[WebRole](#WebRole)

[ConfigurationSettings](#ConfigurationSettings)

[設定](#Setting)

[LocalResources](#LocalResources)

[LocalStorage](#LocalStorage)

[Endpoints](#Endpoints)

[InternalEndpoint](#InternalEndpoint)

[InstanceInputEndpoint](#InstanceInputEndpoint)

[AllocatePublicPortFrom](#AllocatePublicPortFrom)

[FixedPort](#FixedPort)

[FixedPortRange](#FixedPortRange)

[Certificates](#Certificates)

[憑證](#Certificate)

[Imports](#Imports)

[匯入](#Import)

[執行階段](#Runtime)

[環境](#Environment)

[Variable](#Variable)

[RoleInstanceValue](#RoleInstanceValue)

[NetFxEntryPoint](#NetFxEntryPoint)

[Sites](#Sites)

[Site](#Site)

[VirtualApplication](#VirtualApplication)

[VirtualApplication](#VirtualApplication)

[Bindings](#Bindings)

[Binding](#Binding)

[Startup](#Startup)

[Task](#Task)

[Contents](#Contents)

[內容](#Content)

[SourceDirectory](#SourceDirectory)

##  <a name="WebRole"></a> WebRole  
`WebRole` 元素會說明專為 IIS 7 及 ASP.NET 所支援之 Web 應用程式的程式設計所自訂的角色。 服務可包含零個以上的 Web 角色。

下表說明 `WebRole` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 Web 角色的名稱。 角色的名稱必須是唯一的。|  
|enableNativeCodeExecution|布林值|選用。 預設值是 `true`；預設會啟用機器碼執行和完全信任。 將此屬性設為 `false` 會停用 Web 角色的機器碼執行，並改用 Azure 部分信任。|  
|vmsize|字串|選用。 設定此值可變更對角色所配置的虛擬機器大小。 預設值為 `Small`。 如需詳細資訊，請參閱[雲端服務的虛擬機器大小](cloud-services-sizes-specs.md)。|  

##  <a name="ConfigurationSettings"></a> ConfigurationSettings  
`ConfigurationSettings` 元素會說明 Web 角色之組態設定的集合。 此元素是 `Setting` 元素的父代。

##  <a name="Setting"></a> Setting  
`Setting` 元素會說明用以指定角色執行個體之組態設定的名稱/值組。

下表說明 `Setting` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 組態設定的唯一名稱。|  

角色的組態設定是名稱/值組，此組合會於服務定義檔中宣告並於服務組態檔中設定。

##  <a name="LocalResources"></a> LocalResources  
`LocalResources` 元素會說明 Web 角色之本機儲存資源的集合。 此元素是 `LocalStorage` 元素的父代。

##  <a name="LocalStorage"></a> LocalStorage  
`LocalStorage` 元素可識別會在執行階段為服務提供檔案系統空間的本機儲存資源。 角色可定義零個以上的本機儲存資源。

> [!NOTE]
>  `LocalStorage` 元素可顯示為 `WebRole` 元素的子系，以支援與舊版 Azure SDK 的相容性。

下表說明 `LocalStorage` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 本機存放區的唯一名稱。|  
|cleanOnRoleRecycle|布林值|選用。 指出當角色重新啟動時，是否應清除本機存放區。 預設值為 `true`。|  
|sizeInMb|int|選用。 想要配置給本機存放區的儲存體空間數量，以 MB 為單位。 若未指定，則預設配置的儲存體空間是 100 MB。 可配置的儲存體空間最小數量為 1 MB。<br /><br /> 本機資源最大的大小取決於虛擬機器大小。 如需詳細資訊，請參閱[雲端服務的虛擬機器大小](cloud-services-sizes-specs.md)。|  
  
配置給本機儲存資源之目錄的名稱會對應至提供給名稱屬性的值。

##  <a name="Endpoints"></a> Endpoints  
`Endpoints` 元素會說明角色之輸入 (外部)、內部和執行個體輸入端點的集合。 此元素是 `InputEndpoint`、`InternalEndpoint` 和 `InstanceInputEndpoint` 元素的父代。

輸入和內部端點會分開配置。 服務總共可以有 25 個輸入、內部和執行個體輸入端點，而這些端點則可配置給服務所允許的 25 個角色。 例如，如果您有 5 個角色，則可為每個角色配置 5 個輸入端點，或者，您也可以將 25 個輸入端點全都配置給單一角色，又或者，您也可以對這 25 個角色各配置 1 個輸入端點。

> [!NOTE]
>  所部署的每個角色都需要一個執行個體。 訂用帳戶的預設佈建僅限 20 個核心，因此，一個角色僅限擁有 20 個執行個體。 如果您的應用程式所需的執行個體數目超過預設佈建所提供的數目，請參閱[計費、訂用帳戶管理和配額支援](https://azure.microsoft.com/support/options/)，以深入了解如何增加配額。

##  <a name="InputEndpoint"></a> InputEndpoint  
`InputEndpoint` 元素會說明 Web 角色的外部端點。

您可以定義多個端點，包括 HTTP、HTTPS、UDP 和 TCP 端點。 您可以指定針對輸入端點選擇的任何通訊埠編號，但服務中每個角色指定的連接埠號碼必須是唯一的。 例如，如果您指定一個 Web 角色將連接埠 80 用於 HTTP 和連接埠 443 用於 HTTPS，可能會接著指定第二個 Web 角色將連接埠 8080 用於 HTTP 和連接埠 8043 用於 HTTPS。

下表說明 `InputEndpoint` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 外部端點的唯一名稱。|  
|protocol|字串|必要。 外部端點的傳輸通訊協定。 針對 Web 角色，可能的值為 `HTTP`、`HTTPS`、`UDP` 或 `TCP`。|  
|連接埠|int|必要。 外部端點的連接埠。 您可以指定所選擇的任何通訊埠編號，但服務中每個角色指定的連接埠號碼必須是唯一的。<br /><br /> 可能的值範圍介於 1 到 65535 (含) (Azure SDK 1.7 版或更高版本)。|  
|憑證|字串|HTTPS 端點的必要項。 `Certificate` 元素所定義的憑證名稱。|  
|localPort|int|選用。 指定用於端點上內部連線的通訊埠。 `localPort` 屬性會將端點上的外部連接埠對應至角色上的內部連接埠。 這對於一個角色必須與不同於對外連接埠之連接埠上的內部元件通訊的情節很有用。<br /><br /> 如果未指定，`localPort` 的值會與 `port` 屬性相同。 將 `localPort` 的值設定為 “*”，可使用執行階段 API 自動指派可探索的未配置連接埠。<br /><br /> 可能的值範圍介於 1 到 65535 (含) (Azure SDK 1.7 版或更高版本)。<br /><br /> 在使用 Azure SDK 1.3 版或更新版本時，才能使用 `localPort` 屬性。|  
|ignoreRoleInstanceStatus|布林值|選用。 當這個屬性的值設定為 `true` 時，就會忽略服務的狀態，且負載平衡器不會移除端點。 將此值設定為 `true` 適用於服務的偵錯忙碌執行個體。 預設值為 `false`。 **注意：** 即使角色不是處於就緒狀態，端點仍可以接收流量。|  
|loadBalancerProbe|字串|選用。 與輸入端點相關聯之負載平衡器探查的名稱。 如需詳細資訊，請參閱 [LoadBalancerProbe 結構描述](schema-csdef-loadbalancerprobe.md)。|  

##  <a name="InternalEndpoint"></a> InternalEndpoint  
`InternalEndpoint` 元素會說明 Web 角色的內部端點。 內部端點僅適用於服務內執行的其他角色執行個體，它不適用於服務外部的用戶端。 未包含 `Sites` 元素的 Web 角色只能有單一 HTTP、UDP 或 TCP 內部端點。

下表說明 `InternalEndpoint` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 內部端點的唯一名稱。|  
|protocol|字串|必要。 內部端點的傳輸通訊協定。 可能的值為 `HTTP`、`TCP`、`UDP` 或 `ANY`。<br /><br /> `ANY` 的值會指定允許的任何通訊協定、任何連接埠。|  
|連接埠|int|選用。 用於端點上內部負載平衡連線的通訊埠。 負載平衡的端點會使用兩個連接埠。 用於公用 IP 位址的連接埠和私用 IP 位址上使用的通訊埠。 這些通常會設為相同，但您可以選擇使用不同的通訊埠。<br /><br /> 可能的值範圍介於 1 到 65535 (含) (Azure SDK 1.7 版或更高版本)。<br /><br /> 在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Port` 屬性。|  

##  <a name="InstanceInputEndpoint"></a> InstanceInputEndpoint  
`InstanceInputEndpoint` 元素會說明 Web 角色的執行個體輸入端點。 執行個體輸入端點會與特定的角色執行個體相關聯，方法是使用負載平衡器中的連接埠轉送。 每個執行個體輸入端點都會從可能的連接埠範圍對應到特定的通訊埠。 此元素是 `AllocatePublicPortFrom` 元素的父代。

在使用 Azure SDK 1.7 版或更新版本時，才能使用 `InstanceInputEndpoint` 元素。

下表說明 `InstanceInputEndpoint` 元素的屬性。
  
| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 端點的唯一名稱。|  
|localPort|int|必要。 指定所有角色執行個體都會接聽的內部連接埠，可接收從負載平衡器轉送的連入流量。 可能的值範圍介於 1 到 65535 (含) 之間。|  
|protocol|字串|必要。 內部端點的傳輸通訊協定。 可能的值為 `udp` 或 `tcp`。 將 `tcp` 用於以 http/https 作為基礎的流量。|  
  
##  <a name="AllocatePublicPortFrom"></a> AllocatePublicPortFrom  
`AllocatePublicPortFrom` 元素會描述可供外部客戶存取每個執行個體輸入端點的公用連接埠範圍。 在租用戶部署和更新期間，會從此範圍配置公用 (VIP) 連接埠號碼，並加以指派給每個個別的角色執行個體端點。 此元素是 `FixedPortRange` 元素的父代。

在使用 Azure SDK 1.7 版或更新版本時，才能使用 `AllocatePublicPortFrom` 元素。

##  <a name="FixedPort"></a> FixedPort  
`FixedPort` 元素會指定內部端點的連接埠，從而啟用點上的負載平衡端連線。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `FixedPort` 元素。

下表說明 `FixedPort` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|連接埠|int|必要。 內部端點的連接埠。 這與將 `FixedPortRange` min 和 max 設定為相同連接埠會有相同的效果。<br /><br /> 可能的值範圍介於 1 到 65535 (含) (Azure SDK 1.7 版或更高版本)。|  

##  <a name="FixedPortRange"></a> FixedPortRange  
`FixedPortRange` 元素會指定指派給內部端點或執行個體輸入端點的連接埠範圍，並可設定端點上用於負載平衡連線的連接埠。

> [!NOTE]
>  `FixedPortRange` 元素會根據其所在的元素而有不同運作方式。 當 `FixedPortRange` 元素在 `InternalEndpoint` 元素中時，它會針對角色執行所在的所有虛擬機器，開啟 min 和 max 屬性範圍內負載平衡器上的所有連接埠。 當 `FixedPortRange` 元素在 `InstanceInputEndpoint` 元素中時，它會在執行角色的每個虛擬機器上，只開啟一個 min 和 max 屬性範圍內的連接埠。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `FixedPortRange` 元素。

下表說明 `FixedPortRange` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|Min|int|必要。 範圍內的最小連接埠。 可能的值範圍介於 1 到 65535 (含) (Azure SDK 1.7 版或更高版本)。|  
|max|字串|必要。 範圍內的最大連接埠。 可能的值範圍介於 1 到 65535 (含) (Azure SDK 1.7 版或更高版本)。|  

##  <a name="Certificates"></a> Certificates  
`Certificates` 元素會說明 Web 角色之憑證的集合。 此元素是 `Certificate` 元素的父代。 一個角色可以有任意數目的相關聯憑證。 如需使用 certificates 元素的詳細資訊，請參閱[使用憑證修改服務定義檔](cloud-services-configure-ssl-certificate-portal.md#step-2-modify-the-service-definition-and-configuration-files)。

##  <a name="Certificate"></a> Certificate  
`Certificate` 元素會說明與 Web 角色相關聯的憑證。

下表說明 `Certificate` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 此憑證的名稱，當與 HTTPS `InputEndpoint` 元素相關聯時，會用來參考它。|  
|storeLocation|字串|必要。 可在本機電腦上找到此憑證的憑證存放區位置。 可能的值為 `CurrentUser` 與 `LocalMachine`。|  
|storeName|字串|必要。 可在本機電腦上找到此憑證的憑證存放區名稱。 可能的值包括內建存放區名稱 `My`、`Root`、`CA`、`Trust`、`Disallowed`、`TrustedPeople`、`TrustedPublisher`、`AuthRoot`、`AddressBook` 或任何自訂存放區名稱。 如果指定自訂存放區名稱，則會自動建立該存放區。|  
|permissionLevel|字串|選用。 指定提供給角色處理序的存取權限。 如果您希望只有提升權限的處理序能夠存取私密金鑰，則請指定 `elevated` 權限。 `limitedOrElevated` 權限可讓所有角色處理序存取私密金鑰。 可能的值為 `limitedOrElevated` 或 `elevated`。 預設值為 `limitedOrElevated`。|  

##  <a name="Imports"></a> Imports  
`Imports` 元素會說明在客體作業系統中新增元件之 Web 角色的匯入模組集合。 此元素是 `Import` 元素的父代。 這是選用元素，一個角色只能有一個匯入區塊。 

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Imports` 元素。

##  <a name="Import"></a> Import  
`Import` 元素會指定要新增到客體作業系統的模組。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Import` 元素。

下表說明 `Import` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|moduleName|字串|必要。 要匯入之模組的名稱。 有效的匯入模組為：<br /><br /> -   RemoteAccess<br />-   RemoteForwarder<br />-   Diagnostics<br /><br /> RemoteAccess 和 RemoteForwarder 模組可讓您設定遠端桌面連線的角色執行個體。 如需詳細資訊，請參閱[啟用遠端桌面連線](cloud-services-role-enable-remote-desktop-new-portal.md)。<br /><br /> 診斷模組可讓您收集角色執行個體的診斷資料。|  

##  <a name="Runtime"></a> Runtime  
`Runtime` 元素會說明 Web 角色的環境變數設定之集合，可控制 Azure 主機處理序的執行階段環境。 此元素是 `Environment` 元素的父代。 這是選用元素，一個角色只能有一個執行階段區塊。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Runtime` 元素。

下表說明 `Runtime` 元素的屬性：  

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|executionContext|字串|選用。 指定要在其中啟動角色處理序的內容。 預設內容為 `limited`。<br /><br /> -   `limited` – 處理序會啟動，且不具有系統管理員權限。<br />-   `elevated` – 處理序會啟動，且具有系統管理員權限。|  

##  <a name="Environment"></a> Environment  
`Environment` 元素會說明 Web 角色之環境變數設定的集合。 此元素是 `Variable` 元素的父代。 一個角色可以有任意數目的環境變數集。

##  <a name="Variable"></a> Variable  
`Variable` 元素會指定要在客體作業中設定的環境變數。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Variable` 元素。

下表說明 `Variable` 元素的屬性：  

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 要設定之環境變數的名稱。|  
|value|字串|選用。 要為環境變數設定的值。 您必須包含值屬性或 `RoleInstanceValue` 元素。|  

##  <a name="RoleInstanceValue"></a> RoleInstanceValue  
`RoleInstanceValue` 元素會指定要從中擷取變數值的 xPath。

下表說明 `RoleInstanceValue` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|xpath|字串|選用。 執行個體之部署設定的位置路徑。 如需詳細資訊，請參閱[組態變數與 XPath](cloud-services-role-config-xpath.md)。<br /><br /> 您必須包含值屬性或 `RoleInstanceValue` 元素。|  

##  <a name="EntryPoint"></a> EntryPoint  
`EntryPoint` 元素會指定角色的進入點。 此元素是 `NetFxEntryPoint` 元素的父代。 這些元素可讓您指定預設 WaWorkerHost.exe 以外的應用程式來作為角色進入點。

在使用 Azure SDK 1.5 版或更新版本時，才能使用 `EntryPoint` 元素。

##  <a name="NetFxEntryPoint"></a> NetFxEntryPoint  
`NetFxEntryPoint` 元素會指定要為角色執行的程式。

> [!NOTE]
>  在使用 Azure SDK 1.5 版或更新版本時，才能使用 `NetFxEntryPoint` 元素。

下表說明 `NetFxEntryPoint` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|assemblyName|字串|必要。 包含進入點之組件的路徑和檔案名稱。 此路徑會相對於資料夾 **\\%ROLEROOT%\Approot** (請勿在 `commandLine` 中指定 **\\%ROLEROOT%\Approot**，因為系統已採用此資料夾)。 **%ROLEROOT%** 是由 Azure 維護的環境變數，而它代表的是您角色的根資料夾位置。 **\\%ROLEROOT%\Approot** 資料夾代表您角色的應用程式資料夾。<br /><br /> 若為 HWC 角色，此路徑一律會相對於 **\\%ROLEROOT%\Approot\bin** 資料夾。<br /><br /> 若為完整的 IIS 和 IIS Express Web 角色，且找不到相對於 **\\%ROLEROOT%\Approot** 資料夾的組件，則會搜尋 **\\%ROLEROOT%\Approot\bin**。<br /><br /> 完整 IIS 的這種回復行為並非建議的最佳作法，未來的版本可能會將其移除。|  
|targetFrameworkVersion|字串|必要。 用以建置組件的 .NET Framework 版本。 例如： `targetFrameworkVersion="v4.0"`。|  

##  <a name="Sites"></a> Sites  
`Sites` 元素會說明 Web 角色所裝載之網站和 Web 應用程式的集合。 此元素是 `Site` 元素的父代。 如果您未指定 `Sites` 元素，則您的 Web 角色會裝載為舊版 Web 角色，且其中只能裝載一個網站。 這是選用元素，一個角色只能有一個網站區塊。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Sites` 元素。

##  <a name="Site"></a> Site  
`Site` 元素會指定屬於 Web 角色一部分的網站或 Web 應用程式。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Site` 元素。

下表說明 `Site` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 網站或應用程式的名稱。|  
|physicalDirectory|字串|網站根目錄之內容目錄的位置。 您可以將位置指定為絕對路徑或相對於 .csdef 位置的路徑。|  

##  <a name="VirtualApplication"></a> VirtualApplication  
`VirtualApplication` 元素會定義 Internet Information Services (IIS) 7 中的應用程式是用來傳遞內容或透過通訊協定 (如 HTTP) 提供服務之檔案的群組。 當您在 IIS 7 中建立應用程式時，應用程式的路徑會變成網站 URL 的一部分。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `VirtualApplication` 元素。

下表說明 `VirtualApplication` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 指定用來識別虛擬應用程式的名稱。|  
|physicalDirectory|字串|必要。 指定開發機器上包含虛擬應用程式的路徑。 在計算模擬器中，IIS 會設定為從這個位置擷取內容。 在部署至 Azure 時，實體目錄的內容會連同服務的其餘部分一起封裝。 當服務套件部署至 Azure 時，系統會對 IIS 設定解除封裝之內容的位置。|  

##  <a name="VirtualDirectory"></a> VirtualDirectory  
`VirtualDirectory` 元素會指定目錄名稱 (也稱為路徑)，您會在 IIS 中指定此名稱，並對應至本機或遠端伺服器上的實體目錄。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `VirtualDirectory` 元素。

下表說明 `VirtualDirectory` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 指定用來識別虛擬目錄的名稱。|  
|value|physicalDirectory|必要。 指定開發機器上包含網站或虛擬目錄內容的路徑。 在計算模擬器中，IIS 會設定為從這個位置擷取內容。 在部署至 Azure 時，實體目錄的內容會連同服務的其餘部分一起封裝。 當服務套件部署至 Azure 時，系統會對 IIS 設定解除封裝之內容的位置。|  

##  <a name="Bindings"></a> Bindings  
`Bindings` 元素會說明網站之繫結的集合。 它是 `Binding` 元素的父代元素。 每個 `Site` 元素都需要此元素。 如需如何設定端點的詳細資訊，請參閱[啟用角色執行個體的通訊](cloud-services-enable-communication-role-instances.md)。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Bindings` 元素。

##  <a name="Binding"></a> Binding  
`Binding` 元素會指定要讓要求與網站或 Web 應用程式通訊所需的組態資訊。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Binding` 元素。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|name|字串|必要。 指定用來識別繫結的名稱。|  
|endpointName|字串|必要。 指定要繫結的端點名稱。|  
|hostHeader|字串|選用。 指定可讓您使用不同主機名稱在單一 IP 位址/連接埠號碼組合上裝載多個網站的主機名稱。|  

##  <a name="Startup"></a> Startup  
`Startup` 元素會說明角色啟動時所執行之工作的集合。 此元素可以是 `Variable` 元素的父代。 如需如何使用角色啟動工作的詳細資訊，請參閱[如何設定啟動工作](cloud-services-startup-tasks.md)。 這是選用元素，一個角色只能有一個啟動區塊。

下表說明 `Startup` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|優先順序|int|僅供內部使用。|  

##  <a name="Task"></a> Task  
`Task` 元素會指定角色啟動時會發生的啟動工作。 啟動工作可用來執行某些工作，以便讓角色準備好執行這類安裝軟體元件或執行其他應用程式。 工作會依其在 `Startup` 元素區塊內的出現順序來執行。

在使用 Azure SDK 1.3 版或更新版本時，才能使用 `Task` 元素。

下表說明 `Task` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|commandLine|字串|必要。 包含要執行之命令的指令碼，例如 CMD 檔案。 啟動命令和批次檔必須以 ANSI 格式儲存。 在檔案開頭設定位元組順序標記的檔案格式將不會正確地處理。|  
|executionContext|字串|指定指令碼執行所在的內容。<br /><br /> -   `limited` [預設值] – 使用和裝載處理序之角色相同的權限來執行。<br />-   `elevated` – 使用系統管理員權限來執行。|  
|taskType|字串|指定命令的執行行為。<br /><br /> -   `simple` [預設值] – 系統會等到工作結束後，才啟動任何其他工作。<br />-   `background` – 系統不會等到工作結束。<br />-   `foreground` – 與背景類似，但角色不會等到所有前景工作都結束後才重新啟動。|  

##  <a name="Contents"></a> Contents  
`Contents` 元素會說明 Web 角色之內容的集合。 此元素是 `Content` 元素的父代。

在使用 Azure SDK 1.5 版或更新版本時，才能使用 `Contents` 元素。

##  <a name="Content"></a> Content  
`Content` 元素會定義要複製到 Azure 虛擬機器之內容的來源位置，以及此內容的複製目的地路徑。

在使用 Azure SDK 1.5 版或更新版本時，才能使用 `Content` 元素。

下表說明 `Content` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|目的地|字串|必要。 Azure 虛擬機器上要用來放置內容的位置。 此位置會相對於資料夾 **%ROLEROOT%\Approot**。|  

此元素是 `SourceDirectory` 元素的父代元素。

##  <a name="SourceDirectory"></a> SourceDirectory  
`SourceDirectory` 元素會定義要從中複製內容的本機目錄。 您可以使用此元素來指定要複製到 Azure 虛擬機器的本機內容。

在使用 Azure SDK 1.5 版或更新版本時，才能使用 `SourceDirectory` 元素。

下表說明 `SourceDirectory` 元素的屬性。

| 屬性 | 類型 | 說明 |  
| --------- | ---- | ----------- |  
|path|字串|必要。 其內容將會複製到 Azure 虛擬機器之本機目錄的相對或絕對路徑。 支援在目錄路徑中展開環境變數。|  
  
## <a name="see-also"></a>另請參閱
[雲端服務 (傳統) 定義結構描述](schema-csdef-file.md)
