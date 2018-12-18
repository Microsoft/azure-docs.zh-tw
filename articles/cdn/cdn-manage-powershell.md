---
title: 使用 PowerShell 管理 Azure CDN | Microsoft Docs
description: 了解如何使用 Azure PowerShell Cmdlet 管理 Azure CDN。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: fb6f57a5-6e26-4847-8fd9-b51fb05a79eb
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/13/2018
ms.author: magattus
ms.openlocfilehash: d6a67bef831028426dec660a1c79feb4ab9340d1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957705"
---
# <a name="manage-azure-cdn-with-powershell"></a>使用 PowerShell 管理 Azure CDN
PowerShell 提供一種最具彈性的方法，以管理您的 Azure CDN 設定檔和端點。  您可以用互動方式使用 PowerShell 或透過撰寫指令碼來自動進行管理工作。  本教學課程會示範數個您可透過 PowerShell 完成的最常見工作，以管理 Azure CDN 設定檔和端點。

## <a name="prerequisites"></a>必要條件
若要使用 PowerShell 管理 Azure CDN 設定檔和端點，您必須安裝 Azure PowerShell 模組。  若要了解如何安裝 Azure PowerShell 及使用 `Connect-AzureRmAccount` Cmdlet 連接至 Azure，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

> [!IMPORTANT]
> 您必須先使用 `Connect-AzureRmAccount` 登入，才能執行 Azure PowerShell Cmdlet。
> 
> 

## <a name="listing-the-azure-cdn-cmdlets"></a>列出 Azure CDN Cmdlet
您可以使用 `Get-Command` Cmdlet 列出所有的 Azure CDN Cmdlet。

```text
PS C:\> Get-Command -Module AzureRM.Cdn

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-AzureRmCdnCustomDomain                         2.0.0      AzureRm.Cdn
Cmdlet          Get-AzureRmCdnEndpoint                             2.0.0      AzureRm.Cdn
Cmdlet          Get-AzureRmCdnEndpointNameAvailability             2.0.0      AzureRm.Cdn
Cmdlet          Get-AzureRmCdnOrigin                               2.0.0      AzureRm.Cdn
Cmdlet          Get-AzureRmCdnProfile                              2.0.0      AzureRm.Cdn
Cmdlet          Get-AzureRmCdnProfileSsoUrl                        2.0.0      AzureRm.Cdn
Cmdlet          New-AzureRmCdnCustomDomain                         2.0.0      AzureRm.Cdn
Cmdlet          New-AzureRmCdnEndpoint                             2.0.0      AzureRm.Cdn
Cmdlet          New-AzureRmCdnProfile                              2.0.0      AzureRm.Cdn
Cmdlet          Publish-AzureRmCdnEndpointContent                  2.0.0      AzureRm.Cdn
Cmdlet          Remove-AzureRmCdnCustomDomain                      2.0.0      AzureRm.Cdn
Cmdlet          Remove-AzureRmCdnEndpoint                          2.0.0      AzureRm.Cdn
Cmdlet          Remove-AzureRmCdnProfile                           2.0.0      AzureRm.Cdn
Cmdlet          Set-AzureRmCdnEndpoint                             2.0.0      AzureRm.Cdn
Cmdlet          Set-AzureRmCdnOrigin                               2.0.0      AzureRm.Cdn
Cmdlet          Set-AzureRmCdnProfile                              2.0.0      AzureRm.Cdn
Cmdlet          Start-AzureRmCdnEndpoint                           2.0.0      AzureRm.Cdn
Cmdlet          Stop-AzureRmCdnEndpoint                            2.0.0      AzureRm.Cdn
Cmdlet          Test-AzureRmCdnCustomDomain                        2.0.0      AzureRm.Cdn
Cmdlet          Unpublish-AzureRmCdnEndpointContent                2.0.0      AzureRm.Cdn
```

## <a name="getting-help"></a>取得說明
您可以使用 `Get-Help` Cmdlet 取得這些 Cmdlet 的說明。  `Get-Help` 提供使用方式與語法，並選擇性地顯示範例。

```text
PS C:\> Get-Help Get-AzureRmCdnProfile

NAME
    Get-AzureRmCdnProfile

SYNOPSIS
    Gets an Azure CDN profile.


SYNTAX
    Get-AzureRmCdnProfile [-ProfileName <String>] [-ResourceGroupName <String>] [-InformationAction
    <ActionPreference>] [-InformationVariable <String>] [<CommonParameters>]


DESCRIPTION
    Gets an Azure CDN profile and all related information.


RELATED LINKS

REMARKS
    To see the examples, type: "get-help Get-AzureRmCdnProfile -examples".
    For more information, type: "get-help Get-AzureRmCdnProfile -detailed".
    For technical information, type: "get-help Get-AzureRmCdnProfile -full".

```

## <a name="listing-existing-azure-cdn-profiles"></a>列出現有的 Azure CDN 設定檔
不含任何參數的 `Get-AzureRmCdnProfile` Cmdlet 會擷取所有現有的 CDN 設定檔。

```powershell
Get-AzureRmCdnProfile
```

此輸出可以輸送到 Cmdlet 以便列舉。

```powershell
# Output the name of all profiles on this subscription.
Get-AzureRmCdnProfile | ForEach-Object { Write-Host $_.Name }

# Return only **Azure CDN from Verizon** profiles.
Get-AzureRmCdnProfile | Where-Object { $_.Sku.Name -eq "Standard_Verizon" }
```

您也可以指定設定檔名稱和資源群組，以傳回單一設定檔。

```powershell
Get-AzureRmCdnProfile -ProfileName CdnDemo -ResourceGroupName CdnDemoRG
```

> [!TIP]
> 可能會有多個具有相同名稱的 CDN 設定檔，只要它們位於不同的資源群組即可。  省略 `ResourceGroupName` 參數會傳回所有具有相符名稱的設定檔。
> 
> 

## <a name="listing-existing-cdn-endpoints"></a>列出現有的 CDN 端點
`Get-AzureRmCdnEndpoint` 可以擷取設定檔上的個別端點或所有端點。  

```powershell
# Get a single endpoint.
Get-AzureRmCdnEndpoint -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo

# Get all of the endpoints on a given profile. 
Get-AzureRmCdnEndpoint -ProfileName CdnDemo -ResourceGroupName CdnDemoRG

# Return all of the endpoints on all of the profiles.
Get-AzureRmCdnProfile | Get-AzureRmCdnEndpoint

# Return all of the endpoints in this subscription that are currently running.
Get-AzureRmCdnProfile | Get-AzureRmCdnEndpoint | Where-Object { $_.ResourceState -eq "Running" }
```

## <a name="creating-cdn-profiles-and-endpoints"></a>建立 CDN 設定檔和端點
`New-AzureRmCdnProfile` 和 `New-AzureRmCdnEndpoint` 用來建立 CDN 設定檔和端點。 支援以下 SKU：
- Standard_Verizon
- Premium_Verizon
- Custom_Verizon
- Standard_Akamai
- Standard_Microsoft
- Standard_ChinaCdn

```powershell
# Create a new profile
New-AzureRmCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -Sku Standard_Akamai -Location "Central US"

# Create a new endpoint
New-AzureRmCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -Location "Central US" -EndpointName cdnposhdoc -OriginName "Contoso" -OriginHostName "www.contoso.com"

# Create a new profile and endpoint (same as above) in one line
New-AzureRmCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -Sku Standard_Akamai -Location "Central US" | New-AzureRmCdnEndpoint -EndpointName cdnposhdoc -OriginName "Contoso" -OriginHostName "www.contoso.com"

```

## <a name="checking-endpoint-name-availability"></a>檢查端點名稱可用性
`Get-AzureRmCdnEndpointNameAvailability` 可傳回物件，指出端點名稱是否可用。

```powershell
# Retrieve availability
$availability = Get-AzureRmCdnEndpointNameAvailability -EndpointName "cdnposhdoc"

# If available, write a message to the console.
If($availability.NameAvailable) { Write-Host "Yes, that endpoint name is available." }
Else { Write-Host "No, that endpoint name is not available." }
```

## <a name="adding-a-custom-domain"></a>新增自訂網域
`New-AzureRmCdnCustomDomain` 會將自訂網域名稱新增至現有端點。

> [!IMPORTANT]
> 您必須以[如何將自訂網域對應至內容傳遞網路 (CDN) 端點](cdn-map-content-to-custom-domain.md)中所述方式，透過您的 DNS 提供者設定 CNAME。  您可以先測試對應，然後使用 `Test-AzureRmCdnCustomDomain`修改端點。
> 
> 

```powershell
# Get an existing endpoint
$endpoint = Get-AzureRmCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -EndpointName cdnposhdoc

# Check the mapping
$result = Test-AzureRmCdnCustomDomain -CdnEndpoint $endpoint -CustomDomainHostName "cdn.contoso.com"

# Create the custom domain on the endpoint
If($result.CustomDomainValidated){ New-AzureRmCdnCustomDomain -CustomDomainName Contoso -HostName "cdn.contoso.com" -CdnEndpoint $endpoint }
```

## <a name="modifying-an-endpoint"></a>修改端點
`Set-AzureRmCdnEndpoint` 可修改現有的端點。

```powershell
# Get an existing endpoint
$endpoint = Get-AzureRmCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -EndpointName cdnposhdoc

# Set up content compression
$endpoint.IsCompressionEnabled = $true
$endpoint.ContentTypesToCompress = "text/javascript","text/css","application/json"

# Save the changed endpoint and apply the changes
Set-AzureRmCdnEndpoint -CdnEndpoint $endpoint
```

## <a name="purgingpre-loading-cdn-assets"></a>清除/預先載入 CDN 資產
`Unpublish-AzureRmCdnEndpointContent` 會清除快取的資產，而 `Publish-AzureRmCdnEndpointContent` 會在支援的端點上預先載入資產。

```powershell
# Purge some assets.
Unpublish-AzureRmCdnEndpointContent -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo -PurgeContent "/images/kitten.png","/video/rickroll.mp4"

# Pre-load some assets.
Publish-AzureRmCdnEndpointContent -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo -LoadContent "/images/kitten.png","/video/rickroll.mp4"

# Purge everything in /images/ on all endpoints.
Get-AzureRmCdnProfile | Get-AzureRmCdnEndpoint | Unpublish-AzureRmCdnEndpointContent -PurgeContent "/images/*"
```

## <a name="startingstopping-cdn-endpoints"></a>啟動/停止 CDN 端點
`Start-AzureRmCdnEndpoint` 和 `Stop-AzureRmCdnEndpoint` 可用來啟動和停止個別端點的端點群組。

```powershell
# Stop the cdndocdemo endpoint
Stop-AzureRmCdnEndpoint -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo

# Stop all endpoints
Get-AzureRmCdnProfile | Get-AzureRmCdnEndpoint | Stop-AzureRmCdnEndpoint

# Start all endpoints
Get-AzureRmCdnProfile | Get-AzureRmCdnEndpoint | Start-AzureRmCdnEndpoint
```

## <a name="deleting-cdn-resources"></a>刪除 CDN 資源
`Remove-AzureRmCdnProfile` 和 `Remove-AzureRmCdnEndpoint` 可用來移除設定檔和端點。

```powershell
# Remove a single endpoint
Remove-AzureRmCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -EndpointName cdnposhdoc

# Remove all the endpoints on a profile and skip confirmation (-Force)
Get-AzureRmCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG | Get-AzureRmCdnEndpoint | Remove-AzureRmCdnEndpoint -Force

# Remove a single profile
Remove-AzureRmCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG
```

## <a name="next-steps"></a>後續步驟
了解如何透過 [.NET](cdn-app-dev-net.md) 或 [Node.js](cdn-app-dev-node.md) 自動化 Azure CDN。

若要了解 CDN 功能，請參閱 [CDN 概觀](cdn-overview.md)。

