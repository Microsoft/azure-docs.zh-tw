---
title: 雲端服務 (傳統) 角色設定 XPath 功能提要 |Microsoft Docs
description: 您可以在雲端服務角色組態中用來公開設定以做為環境變數的各種 XPath 設定。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 397bc6845dc8d2d8bc44c00c27f6c12037651337
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98741378"
---
# <a name="expose-role-configuration-settings-as-an-environment-variable-with-xpath"></a>利用 XPath 公開角色組態設定以做為環境變數

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

在雲端服務背景工作角色或 Web 角色服務定義檔中，您可以公開執行階段組態值以做為環境變數。 支援下列 XPath 值 (其會對應至 API 值)。

這些 XPath 值也可以透過 [Microsoft.WindowsAzure.ServiceRuntime](/previous-versions/azure/reference/ee773173(v=azure.100)) 程式庫來取得。 

## <a name="app-running-in-emulator"></a>在模擬器中執行的應用程式
表示應用程式正在模擬器中執行。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/Deployment/@emulated" |
| 程式碼 |var x = RoleEnvironment.IsEmulated; |

## <a name="deployment-id"></a>部署 ID
擷取執行個體的部署 ID。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/Deployment/@id" |
| 程式碼 |var deploymentId = RoleEnvironment.DeploymentId; |

## <a name="role-id"></a>角色 ID
擷取執行個體目前的角色 ID。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@id" |
| 程式碼 |var id = RoleEnvironment.CurrentRoleInstance.Id; |

## <a name="update-domain"></a>更新網站
擷取執行個體的更新網域。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@updateDomain" |
| 程式碼 |var ud = RoleEnvironment.CurrentRoleInstance.UpdateDomain; |

## <a name="fault-domain"></a>容錯網域
擷取執行個體的容錯網域。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@faultDomain" |
| 程式碼 |var fd = RoleEnvironment.CurrentRoleInstance.FaultDomain; |

## <a name="role-name"></a>角色名稱
擷取執行個體的角色名稱。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/@roleName" |
| 程式碼 |var rname = RoleEnvironment.CurrentRoleInstance.Role.Name; |

## <a name="config-setting"></a>組態設定
擷取指定之組態設定的值。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/ConfigurationSettings/ConfigurationSetting[@name='Setting1']/@value" |
| 程式碼 |var setting = RoleEnvironment.GetConfigurationSettingValue("Setting1"); |

## <a name="local-storage-path"></a>本機儲存體路徑
擷取執行個體的本機儲存體路徑。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='LocalStore1']/@path" |
| 程式碼 |var localResourcePath = RoleEnvironment.GetLocalResource("LocalStore1").RootPath; |

## <a name="local-storage-size"></a>本機儲存體大小
擷取執行個體的本機儲存體大小。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='LocalStore1']/@sizeInMB" |
| 程式碼 |var localResourceSizeInMB = RoleEnvironment.GetLocalResource("LocalStore1").MaximumSizeInMegabytes; |

## <a name="endpoint-protocol"></a>端點通訊協定
擷取執行個體的端點通訊協定。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/Endpoints/Endpoint[@name='Endpoint1']/@protocol" |
| 程式碼 |var prot = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["Endpoint1"].Protocol; |

## <a name="endpoint-ip"></a>端點 IP
取得指定端點的 IP 位址。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/Endpoints/Endpoint[@name='Endpoint1']/@address" |
| 程式碼 |var address = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["Endpoint1"].IPEndpoint.Address |

## <a name="endpoint-port"></a>端點連接埠
擷取執行個體的端點連接埠。

| 類型 | 範例 |
| --- | --- |
| XPath |xpath="/RoleEnvironment/CurrentInstance/Endpoints/Endpoint[@name='Endpoint1']/@port" |
| 程式碼 |var port = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["Endpoint1"].IPEndpoint.Port; |

## <a name="example"></a>範例
以下是背景工作角色的範例，它會建立啟動工作，並將名為 `TestIsEmulated` 的環境變數設定為[ @emulated xpath 值](#app-running-in-emulator)。 

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




