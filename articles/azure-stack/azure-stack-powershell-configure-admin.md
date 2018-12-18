---
title: 設定 Azure Stack PowerShell 環境 |Microsoft 文件
description: 了解如何設定 Azure Stack PowerShell 環境。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: article
ms.date: 06/22/2018
ms.author: mabrigg
ms.reviewer: thoroet
ms.openlocfilehash: 74a5a9408a78dd0da12fb3f8ed721774030cc438
ms.sourcegitcommit: 6eb14a2c7ffb1afa4d502f5162f7283d4aceb9e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36749856"
---
# <a name="configure-the-azure-stack-powershell-environment"></a>設定 Azure Stack PowerShell 環境

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以將 Azure Stack 設定成使用 PowerShell 來管理資源，例如建立供應項目、方案、配額及警示。 本主題將協助您設定操作員環境。

## <a name="prerequisites"></a>先決條件

從[開發套件](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop)，或從 Windows 型外部用戶端 (如果您是[透過 VPN 連線](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn))，執行下列先決條件。 

 - 安裝 [Azure Stack 相容的 Azure PowerShell 模組](azure-stack-powershell-install.md)。  
 - 下載[處理 Azure Stack 所需的工具](azure-stack-powershell-download.md)。  

## <a name="configure-the-operator-environment-and-sign-in-to-azure-stack"></a>設定操作員環境並登入 Azure Stack

使用 PowerShell 來設定 Azure Stack 操作員環境。 執行下列其中一個指令碼：請使用您自己的環境設定來取代 Azure AD tenantName、GraphAudience 端點及 ArmEndpoint 值。

````PowerShell  
    # For Azure Stack development kit, this value is set to https://adminmanagement.local.azurestack.external.
    # To get this value for Azure Stack integrated systems, contact your service provider.
    $ArmEndpoint = "<Admin Resource Manager endpoint for your environment>"

    # Register an AzureRM environment that targets your Azure Stack instance
    Add-AzureRMEnvironment `
        -Name "AzureStackAdmin" -ArmEndpoint $ArmEndpoint

    # After signing in to your environment, Azure Stack cmdlets
    # can be easily targeted at your Azure Stack instance.
    Add-AzureRmAccount -EnvironmentName "AzureStackAdmin"
````

## <a name="test-the-connectivity"></a>測試連線

一切都已準備就緒，接下來請使用 PowerShell 在 Azure Stack 中建立資源。 例如，您可以建立應用程式的資源群組，並新增虛擬機器。 若要建立名為 **MyResourceGroup** 的資源群組，請使用下列命令。

```powershell
New-AzureRmResourceGroup -Name "MyResourceGroup" -Location "Local"
```

## <a name="next-steps"></a>後續步驟

 - [開發適用於 Azure Stack 的範本](user/azure-stack-develop-templates.md)
 - [使用 PowerShell 部署範本](user/azure-stack-deploy-template-powershell.md)
