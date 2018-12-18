---
title: 使用 PowerShell 在 Azure 中設定 Application Insights | Microsoft Docs
description: 自動設定 Azure 診斷以透過管道傳送至 Application Insights。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 4ac803a8-f424-4c0c-b18f-4b9c189a64a5
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 11/17/2015
ms.author: mbullwin
ms.openlocfilehash: 5e3a577f8c4759dec2d85a431ae31b61b2d582e0
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633150"
---
# <a name="using-powershell-to-set-up-application-insights-for-an-azure-web-app"></a>使用 PowerShell 為 Azure Web 應用程式設定 Application Insights
[Microsoft Azure](https://azure.com) 可以[設定為傳送 Azure 診斷](app-insights-azure-diagnostics.md)至 [Azure Application Insights](app-insights-overview.md)。 診斷與 Azure 雲端服務和 Azure VM 相關。 可輔助您從應用程式使用 Application Insights SDK 傳送的遙測資料。 在 Azure 中自動建立新資源的程序中，您可以使用 PowerShell 設定診斷。

## <a name="azure-template"></a>Azure 範本
如果 Web 應用程式位於 Azure 內，而且您使用 Azure Resource Manager 範本建立資源，您可以透過將以下內容新增到資源節點來設定 Application Insights：

    {
      resources: [
        /* Create Application Insights resource */
        {
          "apiVersion": "2015-05-01",
          "type": "microsoft.insights/components",
          "name": "nameOfAIAppResource",
          "location": "centralus",
          "kind": "web",
          "properties": { "ApplicationId": "nameOfAIAppResource" },
          "dependsOn": [
            "[concat('Microsoft.Web/sites/', myWebAppName)]"
          ]
        }
       ]
     } 

* `nameOfAIAppResource` - Application Insights 資源的名稱
* `myWebAppName` - Web 應用程式的識別碼

## <a name="enable-diagnostics-extension-as-part-of-deploying-a-cloud-service"></a>啟用診斷延伸模組做為部署雲端服務的一部分
`New-AzureDeployment` Cmdlet 具有 `ExtensionConfiguration` 參數，其採用診斷組態的陣列。 使用 `New-AzureServiceDiagnosticsExtensionConfig` Cmdlet 建立可以診斷組態。 例如︰

```ps

    $service_package = "CloudService.cspkg"
    $service_config = "ServiceConfiguration.Cloud.cscfg"
    $diagnostics_storagename = "myservicediagnostics"
    $webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
    $workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

    $primary_storagekey = (Get-AzureStorageKey `
     -StorageAccountName "$diagnostics_storagename").Primary
    $storage_context = New-AzureStorageContext `
       -StorageAccountName $diagnostics_storagename `
       -StorageAccountKey $primary_storagekey

    $webrole_diagconfig = `
     New-AzureServiceDiagnosticsExtensionConfig `
      -Role "WebRole" -Storage_context $storageContext `
      -DiagnosticsConfigurationPath $webrole_diagconfigpath
    $workerrole_diagconfig = `
     New-AzureServiceDiagnosticsExtensionConfig `
      -Role "WorkerRole" `
      -StorageContext $storage_context `
      -DiagnosticsConfigurationPath $workerrole_diagconfigpath

    New-AzureDeployment `
      -ServiceName $service_name `
      -Slot Production `
      -Package $service_package `
      -Configuration $service_config `
      -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig)

``` 

## <a name="enable-diagnostics-extension-on-an-existing-cloud-service"></a>在現有的雲端服務上啟用診斷延伸模組
在現有的服務上，使用 `Set-AzureServiceDiagnosticsExtension`。

```ps

    $service_name = "MyService"
    $diagnostics_storagename = "myservicediagnostics"
    $webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
    $workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"
    $primary_storagekey = (Get-AzureStorageKey `
         -StorageAccountName "$diagnostics_storagename").Primary
    $storage_context = New-AzureStorageContext `
        -StorageAccountName $diagnostics_storagename `
        -StorageAccountKey $primary_storagekey

    Set-AzureServiceDiagnosticsExtension `
        -StorageContext $storage_context `
        -DiagnosticsConfigurationPath $webrole_diagconfigpath `
        -ServiceName $service_name `
        -Slot Production `
        -Role "WebRole" 
    Set-AzureServiceDiagnosticsExtension `
        -StorageContext $storage_context `
        -DiagnosticsConfigurationPath $workerrole_diagconfigpath `
        -ServiceName $service_name `
        -Slot Production `
        -Role "WorkerRole"
```

## <a name="get-current-diagnostics-extension-configuration"></a>取得目前的診斷延伸模組組態
```ps

    Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```


## <a name="remove-diagnostics-extension"></a>移除診斷延伸模組
```ps

    Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

如果您在未使用 Role 參數的情況下使用 `Set-AzureServiceDiagnosticsExtension` 或 `New-AzureServiceDiagnosticsExtensionConfig` 啟用診斷延伸模組，則您可以在未使用 Role 參數的情況下使用 `Remove-AzureServiceDiagnosticsExtension` 移除延伸模組。 如果啟用延伸模組時使用了 Role 參數，則移除延伸模組時也必須使用該參數。

若要從每個個別的角色移除診斷延伸模組：

```ps

    Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"
```


## <a name="see-also"></a>另請參閱
* [使用 Application Insights 監視 Azure 雲端服務應用程式](app-insights-cloudservices.md)
* [將 Azure 診斷傳送至 Application Insights](app-insights-azure-diagnostics.md)
* [自動化設定警示](app-insights-powershell-alerts.md)

