---
title: Azure Front Door Service - PowerShell 參考 | Microsoft Docs
description: 本參考詳述支援 Azure Front Door Service 的不同 PowerShell Cmdlet
services: frontdoor
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/23/2018
ms.author: sharadag
Module Name: AzureRM.FrontDoor
online version: https://docs.microsoft.com/powershell/module/azurerm.frontdoor/remove-azurermfrontdoor
schema: 2.0.0
ms.openlocfilehash: ddc6a40521b951c8083ac185368621ea792b1c85
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47046471"
---
# Remove-AzureRmFrontDoor
移除 Front Door 負載平衡器

## 語法

### ByFieldsParameterSet (預設)
```
Remove-AzureRmFrontDoor -ResourceGroupName <String> -Name <String> [-PassThru]
 [-DefaultProfile <IAzureContextContainer>] [-WhatIf] [-Confirm] [<CommonParameters>]
```

### ByObjectParameterSet
```
Remove-AzureRmFrontDoor -InputObject <PSFrontDoor> [-PassThru] [-DefaultProfile <IAzureContextContainer>]
 [-WhatIf] [-Confirm] [<CommonParameters>]
```

### ResourceIdParameterSet
```
Remove-AzureRmFrontDoor -ResourceId <String> [-PassThru] [-DefaultProfile <IAzureContextContainer>] [-WhatIf]
 [-Confirm] [<CommonParameters>]
```

## 描述
**Remove-AzureRmFrontDoor** Cmdlet 會移除目前訂用帳戶下的 Front Door 負載平衡器

## 範例

### 範例 1：移除目前訂用帳戶下 "rg1" 資源群組中的 "frontdoor1"。
```powershell
PS C:\> Remove-AzureRmFrontDoor -Name "frontdoor1" -ResourceGroupName "rg1"
```

移除目前訂用帳戶下 "rg1" 資源群組中的 "frontdoor1"。

### 範例 2：移除目前訂用帳戶下 "rg1" 資源群組中所有的 FrontDoors。
```powershell
PS C:\> Get-AzureRmFrontDoor -ResourceGroupName "rg1" | Remove-AzureRmFrontDoor
```

移除目前訂用帳戶下 "rg1" 資源群組中所有的 FrontDoors。

### 範例 3：移除目前訂用帳戶下所有的 FrontDoors。
```powershell
PS C:\> Get-AzureRmFrontDoor | Remove-AzureRmFrontDoor
```

移除目前訂用帳戶下所有的 FrontDoors。

### 範例 4：移除目前訂用帳戶下名為 "frontdoor1" 的所有 FrontDoors。
```powershell
PS C:\> Remove-AzureRmFrontDoor -Name "frontdoor1" | Remove-AzureRmFrontDoor
```

移除目前訂用帳戶下名為 "frontdoor1" 的所有 FrontDoors。

## 參數

### -DefaultProfile
用於和 Azure 通訊的認證、帳戶、租用戶及訂用帳戶。

```yaml
Type: Microsoft.Azure.Commands.Common.Authentication.Abstractions.IAzureContextContainer
Parameter Sets: (All)
Aliases: AzureRmContext, AzureCredential

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -InputObject
要刪除的 Front Door 物件。

```yaml
Type: Microsoft.Azure.Commands.FrontDoor.Models.PSFrontDoor
Parameter Sets: ByObjectParameterSet
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: True (ByValue)
Accept wildcard characters: False
```

### -Name
要刪除的 Front Door 名稱。

```yaml
Type: System.String
Parameter Sets: ByFieldsParameterSet
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -PassThru
傳回物件 (若有指定)。

```yaml
Type: System.Management.Automation.SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -ResourceGroupName
Front Door 所屬的資源群組。

```yaml
Type: System.String
Parameter Sets: ByFieldsParameterSet
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -ResourceId
要刪除的 Front Door 資源識別碼

```yaml
Type: System.String
Parameter Sets: ResourceIdParameterSet
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -Confirm
在執行 Cmdlet 前提示您確認。

```yaml
Type: System.Management.Automation.SwitchParameter
Parameter Sets: (All)
Aliases: cf

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -WhatIf
顯示執行 Cmdlet 後會發生的情況。
Cmdlet 並不會執行。

```yaml
Type: System.Management.Automation.SwitchParameter
Parameter Sets: (All)
Aliases: wi

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### CommonParameters
這個 Cmdlet 支援一般參數：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。 如需詳細資訊，請參閱 about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216)。
