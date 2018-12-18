---
title: 使用 PowerShell 在 Azure 雲端服務中啟用診斷 | Microsoft Docs
description: 了解如何使用 PowerShell 啟用雲端服務的診斷
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 66e08754-8639-4022-ae18-4237749ba17d
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 09/06/2016
ms.author: jeconnoc
ms.openlocfilehash: 4c4b5491bba072ba22ec20e164b7315691877a22
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037988"
---
# <a name="enable-diagnostics-in-azure-cloud-services-using-powershell"></a>使用 PowerShell 在 Azure 雲端服務中啟用診斷
您可以使用 Azure 診斷延伸模組，從雲端服務收集診斷資料 (例如應用程式記錄檔、效能計數器等)。 本文描述如何使用 PowerShell 啟用雲端服務的 Azure 診斷延伸模組。  如需這篇文章所需要的必要條件，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。

## <a name="enable-diagnostics-extension-as-part-of-deploying-a-cloud-service"></a>啟用診斷延伸模組做為部署雲端服務的一部分
此方法適用於可以啟用診斷延伸模組做為雲端服務佈署一部分的連續整合類型案例。 當您建立新的雲端服務部署時，可以啟用診斷擴充，方法是將 *ExtensionConfiguration* 參數傳入 [New-AzureDeployment](/powershell/module/servicemanagement/azure/new-azuredeployment?view=azuresmps-3.7.0) Cmdlet。 *ExtensionConfiguration* 參數接受以 [New-AzureServiceDiagnosticsExtensionConfig](/powershell/module/servicemanagement/azure/new-azureservicediagnosticsextensionconfig?view=azuresmps-3.7.0) Cmdlet 建立的診斷組態陣列。

下列範例示範如何為某個雲端服務 (其中的 WebRole 和 WorkerRole 各自擁有不同的診斷組態) 啟用診斷。

```powershell
$service_name = "MyService"
$service_package = "CloudService.cspkg"
$service_config = "ServiceConfiguration.Cloud.cscfg"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml"
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath

New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig)
```

如果診斷組態檔以某個儲存體帳戶名稱指定 `StorageAccount` 元素，則 `New-AzureServiceDiagnosticsExtensionConfig` Cmdlet 將會自動使用該儲存體帳戶。 但該儲存體帳戶所屬的訂用帳戶，必須與雲端服務部署時所屬的訂用帳戶相同，這個方法才有作用。

從 Azure SDK 2.6 開始，由 MSBuild 發佈目標輸出所產生的擴充設定檔，會包含以服務組態檔 (.cscfg) 中所指定的診斷組態字串為基礎的儲存體帳戶名稱。 下列指令碼示範在部署雲端服務時，如何從發佈目標輸出來剖析擴充設定檔，以及如何設定每個角色的診斷擴充。

```powershell
$service_name = "MyService"
$service_package = "C:\build\output\CloudService.cspkg"
$service_config = "C:\build\output\ServiceConfiguration.Cloud.cscfg"

#Find the Extensions path based on service configuration file
$extensionsSearchPath = Join-Path -Path (Split-Path -Parent $service_config) -ChildPath "Extensions"

$diagnosticsExtensions = Get-ChildItem -Path $extensionsSearchPath -Filter "PaaSDiagnostics.*.PubConfig.xml"
$diagnosticsConfigurations = @()
foreach ($extPath in $diagnosticsExtensions)
{
    #Find the RoleName based on file naming convention PaaSDiagnostics.<RoleName>.PubConfig.xml
    $roleName = ""
    $roles = $extPath -split ".",0,"simplematch"
    if ($roles -is [system.array] -and $roles.Length -gt 1)
    {
        $roleName = $roles[1]
        $x = 2
        while ($x -le $roles.Length)
            {
               if ($roles[$x] -ne "PubConfig")
                {
                    $roleName = $roleName + "." + $roles[$x]
                }
                else
                {
                    break
                }
                $x++
            }
        $fullExtPath = Join-Path -path $extensionsSearchPath -ChildPath $extPath
        $diagnosticsconfig = New-AzureServiceDiagnosticsExtensionConfig -Role $roleName -DiagnosticsConfigurationPath $fullExtPath
        $diagnosticsConfigurations += $diagnosticsconfig
    }
}
New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration $diagnosticsConfigurations
```

Visual Studio Online 使用類似的方法，自動部署具有診斷延伸模組的雲端服務。 請參閱 [Publish-AzureCloudDeployment.ps1](https://github.com/Microsoft/vso-agent-tasks/blob/master/Tasks/AzureCloudPowerShellDeployment/Publish-AzureCloudDeployment.ps1) 來取得完整的範例。

如果您未在診斷組態中指定 `StorageAccount`，則需要將 *StorageAccountName* 參數傳入 Cmdlet。 如果已指定 *StorageAccountName* 參數，則 Cmdlet 一定會使用在此參數中指定的儲存體帳戶，而非在診斷組態檔中指定的儲存體帳戶。

如果診斷儲存體帳戶和雲端服務分別屬於不同的訂用帳戶，您必須明確地將 *StorageAccountName* 和 *StorageAccountKey* 參數傳入 Cmdlet。 當診斷儲存體帳戶位於同一個訂用帳戶中時，就不需要使用 *StorageAccountKey* 參數，因為 Cmdlet 會在啟用診斷擴充功能時自動查詢並設定金鑰值。 不過，如果診斷儲存體帳戶位於不同的訂用帳戶中，Cmdlet 可能就無法自動取得金鑰，您必須透過 *StorageAccountKey* 參數來明確指定金鑰。

```powershell
$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
```

## <a name="enable-diagnostics-extension-on-an-existing-cloud-service"></a>在現有的雲端服務上啟用診斷延伸模組
您可以使用 [Set-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure/set-azureservicediagnosticsextension?view=azuresmps-3.7.0) Cmdlet，在執行中的雲端服務上啟用或更新診斷組態。

[!INCLUDE [cloud-services-wad-warning](../../includes/cloud-services-wad-warning.md)]

```powershell
$service_name = "MyService"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml"
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath

Set-AzureServiceDiagnosticsExtension -DiagnosticsConfiguration @($webrole_diagconfig,$workerrole_diagconfig) -ServiceName $service_name
```

## <a name="get-current-diagnostics-extension-configuration"></a>取得目前的診斷延伸模組組態
使用 [Get AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure/get-azureservicediagnosticsextension?view=azuresmps-3.7.0) Cmdlet 取得雲端服務目前的診斷組態。

```powershell
Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

## <a name="remove-diagnostics-extension"></a>移除診斷延伸模組
若要在雲端服務上關閉診斷，您可以使用 [Remove-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure/remove-azureservicediagnosticsextension?view=azuresmps-3.7.0) Cmdlet。

```powershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

如果您在未使用 Role 參數的情況下使用 Set-AzureServiceDiagnosticsExtension 或 New-AzureServiceDiagnosticsExtensionConfig 啟用診斷擴充，則您可以在未使用 Role 參數的情況下使用 Remove-AzureServiceDiagnosticsExtension 來移除擴充。 如果啟用延伸模組時使用了 *Role* 參數，則移除延伸模組時也必須使用該參數。

若要從每個個別的角色移除診斷延伸模組：

```powershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"
```

## <a name="next-steps"></a>後續步驟
* 如需使用 Azure 診斷和其他技術疑難排解問題的詳細指引，請參閱 [在 Azure 雲端服務和虛擬機器中啟用診斷](cloud-services-dotnet-diagnostics.md)。
* [診斷組態結構描述](https://msdn.microsoft.com/library/azure/dn782207.aspx) 說明診斷延伸模組的各種 XML 組態選項。
* 若要了解如何啟用虛擬機器的診斷延伸模組，請參閱 [使用 Azure 資源管理員範本建立具有監控和診斷功能的 Windows 虛擬機器](../virtual-machines/windows/extensions-diagnostics-template.md)
