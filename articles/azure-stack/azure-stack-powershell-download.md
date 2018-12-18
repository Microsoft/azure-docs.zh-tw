---
title: 從 GitHub 下載 Azure Stack 工具 | Microsoft Docs
description: 了解如何下載處理 Azure Stack 所需的工具。
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
ms.date: 09/28/2018
ms.author: mabrigg
ms.reviewer: thoroet
ms.openlocfilehash: 5b90ebc554738c89816cf88f8984ffab01c87d4d
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451308"
---
# <a name="download-azure-stack-tools-from-github"></a>從 GitHub 下載 Azure Stack 工具

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

**AzureStack-Tools** 是一個 [GitHub 存放庫](https://github.com/Azure/AzureStack-Tools)，當中裝載用來管理資源並將其部署至 Azure Stack 的 PowerShell 模組。 如果您打算建立 VPN 連線能力，您可以將這些 PowerShell 模組下載到 Azure Stack 開發套件或以 Windows 為基礎的外部用戶端。 若要取得這些工具，請複製 GitHub 存放庫或下載 **AzureStack-Tools** 資料夾，方法是執行下列指令碼：

```PowerShell
# Change directory to the root directory. 
cd \

# Download the tools archive.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
invoke-webrequest `
  https://github.com/Azure/AzureStack-Tools/archive/master.zip `
  -OutFile master.zip

# Expand the downloaded files.
expand-archive master.zip `
  -DestinationPath . `
  -Force

# Change to the tools directory.
cd AzureStack-Tools-master

```

## <a name="functionality-provided-by-the-modules"></a>模組所提供的功能

**AzureStack-Tools** 存放庫包含 PowerShell 模組，其支援下列 Azure Stack 功能：  

| 功能 | 說明 | 可以使用此模組的人員 |
| --- | --- | --- |
| [雲端功能](user/azure-stack-validate-templates.md) | 使用此模組來取得雲端的雲端功能。 例如，您可使用此模組來取得雲端功能，例如 API 版本和 Azure Resource Manager 資源。 您也可以使用此模組來取得 Azure Stack 和 Azure 雲端的 VM 擴充功能。 | 雲端操作員和使用者 |
| [適用於 Azure Stack 的 Resource Manager 原則](user/azure-stack-policy-module.md) | 使用此模組來設定 Azure 訂用帳戶或 Azure 資源群組，包含與 Azure Stack 相同的版本和服務可用性。 | 雲端操作員和使用者 |
| [註冊 Azure](azure-stack-register.md) | 搭配使用此模組與 Azure 來註冊您的開發套件執行個體。 註冊之後，您可以從 Azure 下載 Marketplace 項目，並在 Azure Stack 中使用它們。 | 雲端操作員 |
| [Azure Stack 部署](azure-stack-run-powershell-script.md) | 搭配使用此模組與 Azure Stack 虛擬硬碟 (VHD) 映像，準備要部署與重新部署的 Azure Stack 主機電腦。 | 雲端操作員|
| [連線至 Azure Stack](azure-stack-connect-powershell.md) | 使用此模組來設定對 Azure Stack 的 VPN 連線能力。 | 雲端操作員和使用者 |
| [範本驗證程式](user/azure-stack-validate-templates.md) | 使用此模組來確認現有或新範本是否可以部署到 Azure Stack。 | 雲端操作員和使用者|


## <a name="next-steps"></a>後續步驟
* [設定 Azure Stack 使用者的 PowerShell 環境](user/azure-stack-powershell-configure-user.md)   
* [透過 VPN 連線至 Azure Stack 開發套件](azure-stack-connect-azure-stack.md)  
