---
title: 重設失敗的 Azure ExpressRoute 線路︰PowerShell | Microsoft Docs
description: 本文可協助您重設處於失敗狀態的 ExpressRoute 電路。
documentationcenter: na
services: expressroute
author: anzaman
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/28/2017
ms.author: anzaman;cherylmc
ms.openlocfilehash: 423bc1d6409e5b7fe02339a05d0775f4ff42de49
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31590678"
---
# <a name="reset-a-failed-expressroute-circuit"></a>重設失敗的 ExpressRoute 電路

當 ExpressRoute 電路上的作業未成功完成時，電路可能會進入 [失敗] 狀態。 本文可協助您重設失敗的 Azure ExpressRoute 電路。

## <a name="reset-a-circuit"></a>重設線路

1. 安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需詳細資訊，請參閱[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。

2. 以提高的權限開啟 PowerShell 主控台並連線至您的帳戶。 使用下列範例來協助您連接：

  ```powershell
  Connect-AzureRmAccount
  ```
3. 如果您有多個 Azure 訂用帳戶，請檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```
4. 指定您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
  ```
5. 執行下列命令來重設處於失敗狀態的電路：

  ```powershell
  $ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

  Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
  ```

電路現在應該狀況良好。 如果電路仍處於失敗狀態，請使用 [Microsoft 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)開啟支援票證。

## <a name="next-steps"></a>後續步驟

如果仍會發生問題，請向 [Microsoft 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) 開啟支援票證。
