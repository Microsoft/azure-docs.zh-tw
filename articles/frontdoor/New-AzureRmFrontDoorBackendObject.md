---
title: Azure Front Door Service - PowerShell 參考 | Microsoft Docs
description: 此參考詳述針對 Azure Front Door Service 支援的不同 PowerShell Cmdlet
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
online version: https://docs.microsoft.com/powershell/module/azurerm.frontdoor/new-azurermfrontdoorbackendobject
schema: 2.0.0
ms.openlocfilehash: 6ec904f581258d8254e7cb69a0ed7ee858a41154
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045826"
---
# New-AzureRmFrontDoorBackendObject
建立 PSBackend 物件

## 語法

```
New-AzureRmFrontDoorBackendObject -Address <String> [-HttpPort <Int32>] [-HttpsPort <Int32>]
 [-Priority <Int32>] [-Weight <Int32>] [-EnabledState <PSEnabledState>] [-BackendHostHeader <String>]
 [-DefaultProfile <IAzureContextContainer>] [<CommonParameters>]
```

## 描述
建立 PSBackend 物件以用於 Front Door 建立

## 範例

### 範例 1
```powershell
PS C:\>New-AzureRmFrontDoorBackendObject -Address "contoso1.azurewebsites.net"


Address           : contoso1.azurewebsites.net
HttpPort          : 80
HttpsPort         : 443
Priority          : 1
Weight            : 50
BackendHostHeader :
EnabledState      : Enabled
```

建立 PSBackend 物件以用於 Front Door 建立

## 參數

### -Address
後端的位置 (IP 位址或 FQDN)

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -BackendHostHeader
用來作為傳送至後端之主機標頭的值。 預設值為後端位址。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -DefaultProfile
用於與 Azure 通訊的認證、帳戶、租用戶與訂用帳戶。

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

### -EnabledState
是否要啟用這個後端。 預設值為 Enabled

```yaml
Type: Microsoft.Azure.Commands.FrontDoor.Models.PSEnabledState
Parameter Sets: (All)
Aliases:
Accepted values: Enabled, Disabled

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -HttpPort
HTTP TCP 連接埠號碼。
必須介於 1 到 65535 之間。
預設值為 80。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -HttpsPort
HTTPS TCP 連接埠號碼。
必須介於 1 到 65535 之間。
預設值為 443

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -Priority
用於負載平衡的優先順序。
必須介於 1 到 5 之間。
預設值為 1

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -Weight
這個端點的加權，適用於負載平衡用途。
必須介於 1 到 1000 之間。
預設值為 50

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### CommonParameters
這個 Cmdlet 支援一般參數：-Debug、-ErrorAction、-ErrorVariable、-InformationAction、-InformationVariable、-OutVariable、-OutBuffer、-PipelineVariable、-Verbose、-WarningAction 和 -WarningVariable。 如需詳細資訊，請參閱 about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216)。
