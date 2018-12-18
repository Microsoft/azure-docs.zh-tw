---
title: 雲端服務角色組態 XPath 功能提要 | Microsoft Docs
description: 您可以在雲端服務角色組態中用來公開設定以做為環境變數的各種 XPath 設定。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: c51e4493-0643-4d05-bc44-06c76bcbf7d1
ms.service: cloud-services
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/19/2017
ms.author: jeconnoc
ms.openlocfilehash: 2db63be6c6997840f7409a3ca79f1845f30e4ceb
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008054"
---
# <a name="expose-role-configuration-settings-as-an-environment-variable-with-xpath"></a>利用 XPath 公開角色組態設定以做為環境變數
在雲端服務背景工作角色或 Web 角色服務定義檔中，您可以公開執行階段組態值以做為環境變數。 支援下列 XPath 值 (其會對應至 API 值)。

這些 XPath 值也可以透過 [Microsoft.WindowsAzure.ServiceRuntime](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.aspx) 程式庫來取得。 

## <a name="app-running-in-emulator"></a>在模擬器中執行的應用程式
表示應用程式正在模擬器中執行。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/Deployment/@emulated" |
| 代碼 |var x = RoleEnvironment.IsEmulated; |

## <a name="deployment-id"></a>部署 ID
擷取執行個體的部署 ID。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/Deployment/@id" |
| 代碼 |var deploymentId = RoleEnvironment.DeploymentId; |

## <a name="role-id"></a>角色 ID
擷取執行個體目前的角色 ID。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@id" |
| 代碼 |var id = RoleEnvironment.CurrentRoleInstance.Id; |

## <a name="update-domain"></a>更新網站
擷取執行個體的更新網域。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@updateDomain" |
| 代碼 |var ud = RoleEnvironment.CurrentRoleInstance.UpdateDomain; |

## <a name="fault-domain"></a>容錯網域
擷取執行個體的容錯網域。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@faultDomain" |
| 代碼 |var fd = RoleEnvironment.CurrentRoleInstance.FaultDomain; |

## <a name="role-name"></a>角色名稱
擷取執行個體的角色名稱。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@roleName" |
| 代碼 |var rname = RoleEnvironment.CurrentRoleInstance.Role.Name; |

## <a name="config-setting"></a>組態設定
擷取指定之組態設定的值。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/ConfigurationSettings/ConfigurationSetting[@name='Setting1']/@value" |
| 代碼 |var setting = RoleEnvironment.GetConfigurationSettingValue("Setting1"); |

## <a name="local-storage-path"></a>本機儲存體路徑
擷取執行個體的本機儲存體路徑。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='LocalStore1']/@path" |
| 代碼 |var localResourcePath = RoleEnvironment.GetLocalResource("LocalStore1").RootPath; |

## <a name="local-storage-size"></a>本機儲存體大小
擷取執行個體的本機儲存體大小。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='LocalStore1']/@sizeInMB" |
| 代碼 |var localResourceSizeInMB = RoleEnvironment.GetLocalResource("LocalStore1").MaximumSizeInMegabytes; |

## <a name="endpoint-protocol"></a>端點通訊協定
擷取執行個體的端點通訊協定。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/Endpoints/Endpoint[@name='Endpoint1']/@protocol" |
| 代碼 |var prot = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["Endpoint1"].Protocol; |

## <a name="endpoint-ip"></a>端點 IP
取得指定端點的 IP 位址。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/Endpoints/Endpoint[@name='Endpoint1']/@address" |
| 代碼 |var address = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["Endpoint1"].IPEndpoint.Address |

## <a name="endpoint-port"></a>端點連接埠
擷取執行個體的端點連接埠。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/Endpoints/Endpoint[@name='Endpoint1']/@port" |
| 代碼 |var port = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["Endpoint1"].IPEndpoint.Port; |

## <a name="example"></a>範例
以下的背景工作角色範例會使用名為 `TestIsEmulated` 且設定為 [@emulated xpath 值](#app-running-in-emulator)的環境變數來建立啟動工作。 

```xml
<WorkerRole name="Role1">
    <ConfigurationSettings>
      <Setting name="Setting1" />
    </ConfigurationSettings>
    <LocalResources>
      <LocalStorage name="LocalStore1" sizeInMB="1024"/>
    </LocalResources>
    <Endpoints>
      <InternalEndpoint name="Endpoint1" protocol="tcp" />
    </Endpoints>
    <Startup>
      <Task commandLine="example.cmd inputParm">
        <Environment>
          <Variable name="TestConstant" value="Constant"/>
          <Variable name="TestEmptyValue" value=""/>
          <Variable name="TestIsEmulated">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated"/>
          </Variable>
          ...
        </Environment>
      </Task>
    </Startup>
    <Runtime>
      <Environment>
        <Variable name="TestConstant" value="Constant"/>
        <Variable name="TestEmptyValue" value=""/>
        <Variable name="TestIsEmulated">
          <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated"/>
        </Variable>
        ...
      </Environment>
    </Runtime>
    ...
</WorkerRole>
```

## <a name="next-steps"></a>後續步驟
深入了解 [ServiceConfiguration.cscfg](cloud-services-model-and-package.md#serviceconfigurationcscfg) 檔案。

建立 [ServicePackage.cspkg](cloud-services-model-and-package.md#servicepackagecspkg) 封裝。

為角色啟用 [遠端桌面](cloud-services-role-enable-remote-desktop-new-portal.md) 。

