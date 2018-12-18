---
title: 管理 Azure DNS 中的 DNS 區域 - PowerShell | Microsoft Docs
description: 您可以使用 Azure Powershell 管理 DNS 區域。 本文說明如何在 Azure DNS 上更新、刪除及建立 DNS 區域
services: dns
documentationcenter: na
author: vhorne
manager: timlt
ms.assetid: a67992ab-8166-4052-9b28-554c5a39e60c
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2018
ms.author: victorh
ms.openlocfilehash: 2cdf6ea0a992c996f4aeeb117c440e94b55ff8a8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961955"
---
# <a name="how-to-manage-dns-zones-using-powershell"></a>如何管理使用 PowerShell 的 DNS 區域

> [!div class="op_single_selector"]
> * [入口網站](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [Azure 傳統 CLI](dns-operations-dnszones-cli-nodejs.md)
> * [Azure CLI](dns-operations-dnszones-cli.md)

本文說明如何使用 Azure PowerShell 管理 DNS 區域。 您也可以使用跨平台 [Azure CLI](dns-operations-dnszones-cli.md) 或 Azure 入口網站來管理 DNS 區域。

本指南特別說明公用 DNS 區域。 如需在 Azure DNS 中使用 Azure PowerShell 管理私人區域的資訊，請參閱[利用 PowerShell 開始使用 Azure DNS 私人區域](private-dns-getstarted-powershell.md)。

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

[!INCLUDE [dns-powershell-setup](../../includes/dns-powershell-setup-include.md)]


## <a name="create-a-dns-zone"></a>建立 DNS 區域

使用 `New-AzureRmDnsZone` Cmdlet 建立 DNS 區域。

下列範例會在稱為 *MyResourceGroup* 的資源群組中建立稱為 *contoso.com* 的 DNS 區域：

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

下列範例示範如何使用 *project = demo* 和 *env = test* 這兩個 [Azure Resource Manager 標籤](dns-zones-records.md#tags)，建立 DNS 區域：

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

Azure DNS 現在也支援私人 DNS 區域 (目前處於公開預覽狀態)。  若要深入了解私人 DNS 區域，請參閱[使用 Azure DNS 私人網域](private-dns-overview.md)。 如需如何建立私人 DNS 區域的範例，請參閱[利用 PowerShell 開始使用 Azure DNS 私人區域](./private-dns-getstarted-powershell.md)。

## <a name="get-a-dns-zone"></a>取得 DNS 區域

若要擷取 DNS 區域，請使用 `Get-AzureRmDnsZone` Cmdlet。 此作業會傳回對應至 Azure DNS 中現有區域的 DNS 區域物件。 這個物件包含區域的相關資料 (例如記錄集的數目)，但不包含記錄集本身 (請參閱 `Get-AzureRmDnsRecordSet`)。

```powershell
Get-AzureRmDnsZone -Name contoso.com –ResourceGroupName MyAzureResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-8ec2-f4879750d201
Tags                  : {project, env}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org.,
                        ns4-01.azure-dns.info.}
NumberOfRecordSets    : 2
MaxNumberOfRecordSets : 5000
```

## <a name="list-dns-zones"></a>列出 DNS 區域

您可以從 `Get-AzureRmDnsZone` 中省略區域名稱，以列舉資源群組中的所有區域： 此作業會傳回一系列的區域物件。

```powershell
$zoneList = Get-AzureRmDnsZone -ResourceGroupName MyAzureResourceGroup
```

您可以從 `Get-AzureRmDnsZone` 中省略區域名稱和資源群組名稱，以列舉 Azure 訂用帳戶中的所有區域。

```powershell
$zoneList = Get-AzureRmDnsZone
```

## <a name="update-a-dns-zone"></a>更新 DNS 區域

您可以使用 `Set-AzureRmDnsZone`變更 DNS 區域資源。 這個 Cmdlet 不會更新區域內的任何 DNS 記錄集 (請參閱[如何管理 DNS 記錄](dns-operations-recordsets.md))。 它只用來更新區域資源本身的屬性。 這些可寫入區域屬性目前僅限於[區域資源的Azure Resource Manager「標籤」](dns-zones-records.md#tags)。

請使用下列兩種方法之一來更新 DNS 區域：

### <a name="specify-the-zone-using-the-zone-name-and-resource-group"></a>使用區域名稱和資源群組來指定區域

這種方法會以指定的值取代現有的區域標籤。

```powershell
Set-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

### <a name="specify-the-zone-using-a-zone-object"></a>使用 $zone 物件來指定區域

此方法會擷取現有的區域物件、修改標籤，然後認可變更。 如此一來，就可以保留現有的標籤。

```powershell
# Get the zone object
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup

# Remove an existing tag
$zone.Tags.Remove("project")

# Add a new tag
$zone.Tags.Add("status","approved")

# Commit changes
Set-AzureRmDnsZone -Zone $zone
```

使用 `Set-AzureRmDnsZone` 搭配 $zone 物件時，會使用 [Etag 檢查](dns-zones-records.md#etags)，以確保不會覆寫並行變更。 您可以使用選擇性的 `-Overwrite` 參數來停用這些檢查。

## <a name="delete-a-dns-zone"></a>刪除 DNS 區域

您可以使用 `Remove-AzureRmDnsZone` Cmdlet 刪除 DNS 區域。

> [!NOTE]
> 刪除 DNS 區域也會刪除該區域內的所有 DNS 記錄。 此作業無法復原。 如果 DNS 區域正在使用中，當該區域遭到刪除時，使用該區域的服務將會失敗。
>
>若要防止區域意外遭到刪除，請參閱[如何保護 DNS 區域和記錄](dns-protect-zones-recordsets.md)。


請使用下列兩種方法之一來刪除 DNS 區域：

### <a name="specify-the-zone-using-the-zone-name-and-resource-group-name"></a>使用區域名稱和資源群組名稱來指定區域

```powershell
Remove-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

### <a name="specify-the-zone-using-a-zone-object"></a>使用 $zone 物件來指定區域

您可以使用 `Get-AzureRmDnsZone` 所傳回的 `$zone` 物件，指定要刪除的區域。

```powershell
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
Remove-AzureRmDnsZone -Zone $zone
```

區域物件也可以經由管道輸送，而不是當做參數傳遞：

```powershell
Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup | Remove-AzureRmDnsZone

```

如同 `Set-AzureRmDnsZone` 一樣，使用 `$zone` 物件指定區域可啟用 Etag 檢查，以確保不會刪除並行的變更。 使用 `-Overwrite` 參數來停用這些檢查。

## <a name="confirmation-prompts"></a>確認提示

`New-AzureRmDnsZone`、`Set-AzureRmDnsZone` 和 `Remove-AzureRmDnsZone` cmdlets 皆支援確認提示。

如果 `$ConfirmPreference` PowerShell 喜好設定變數的值為 `Medium` 或更低，則 `New-AzureRmDnsZone` 和 `Set-AzureRmDnsZone` 兩者都會顯示確認提示。 由於刪除 DNS 區域可能造成重大影響，所以如果 `$ConfirmPreference` PowerShell 變數的值不是 `None`，`Remove-AzureRmDnsZone` Cmdlet 就會顯示確認提示。

由於 `$ConfirmPreference` 的預設值是 `High`，預設只有 `Remove-AzureRmDnsZone` 會顯示確認提示。

您可以使用 `-Confirm` 參數覆寫目前 `$ConfirmPreference` 設定。 如果您指定 `-Confirm` 或 `-Confirm:$True`，此 cmdlet 在執行前會提示您進行確認。 如果您指定 `-Confirm:$False`，此 cmdlet 不會提示您進行確認。

如需 `-Confirm` 和 `$ConfirmPreference` 的詳細資訊，請參閱[有關喜好設定變數](https://msdn.microsoft.com/powershell/reference/5.1/Microsoft.PowerShell.Core/about/about_Preference_Variables)。

## <a name="next-steps"></a>後續步驟

了解如何在 DNS 區域中[管理記錄集和記錄](dns-operations-recordsets.md)。
<br>
了解如何[將您的網域委派給 Azure DNS](dns-domain-delegation.md)。
<br>
檢閱 [Azure DNS PowerShell 參考文件](/powershell/module/azurerm.dns)。

