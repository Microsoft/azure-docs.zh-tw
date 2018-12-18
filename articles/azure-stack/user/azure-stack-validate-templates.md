---
title: 使用範本驗證工具來檢查 Azure Stack 的範本 | Microsoft Docs
description: 檢查要部署到 Azure Stack 的範本
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: d9e6aee1-4cba-4df5-b5a3-6f38da9627a3
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: sethm
ms.reviewer: jeffgo
ms.openlocfilehash: 73a0766baee8da782f0192fbc17fb2898a8360ac
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "41946409"
---
# <a name="check-your-templates-for-azure-stack-with-the-template-validation-tool"></a>使用範本驗證工具來檢查 Azure Stack 的範本

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用範本驗證工具，檢查您的 Azure Resource Manager [範本](azure-stack-arm-templates.md)是否準備好部署至 Azure Stack。 範本驗證工具是 Azure Stack 工具的一部分。 使用[從 GitHub 下載工具](azure-stack-powershell-download.md)文章中所述的步驟下載 Azure Stack 工具。

## <a name="overview"></a>概觀

若要驗證範本，您必須先建置雲端功能檔案，然後執行驗證工具。 您可使用 Azure Stack 工具中的下列 PowerShell 模組：

- 在 **CloudCapabilities** 資料夾中：<br>         AzureRM.CloudCapabilities.psm1 會建立雲端功能 JSON 檔案，該檔案代表 Azure Stack 雲端中的服務與版本。
- 在 **TemplateValidator** 資料夾中：<br>
AzureRM.TemplateValidator.psm1 使用雲端功能 JSON 檔案來測試將在 Azure Stack 中部署的範本。

## <a name="build-the-cloud-capabilities-file"></a>建置雲端功能檔案

使用範本驗證程式之前，先執行 AzureRM.CloudCapabilities PowerShell 模組來建立 JSON 檔案。

>[!NOTE]
>如果您更新您的整合式系統，或新增任何新的服務或虛擬擴充功能，則應該再次執行此模組。

1. 確定您能順利連線到 Azure Stack。 這些步驟可從 Azure Stack 開發套件主機執行，或者您可以使用 [VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) 從您的工作站連線。
2. 匯入 AzureRM.CloudCapabilities PowerShell 模組：

    ```PowerShell
    Import-Module .\CloudCapabilities\AzureRM.CloudCapabilities.psm1
    ```

3. 使用 Get-CloudCapabilities Cmdlet 來擷取服務版本並建立雲端功能 JSON 檔案。 如未指定 **-OutputPath**，目前的目錄中會建立 AzureCloudCapabilities.Json 檔案。 使用您的實際位置：

    ```PowerShell
    Get-AzureRMCloudCapability -Location <your location> -Verbose
    ```

## <a name="validate-templates"></a>驗證範本

在這些步驟中，使用 AzureRM.TemplateValidator PowerShell 模組來驗證範本。 您可以使用您自己的範本，或驗證 [Azure Stack 快速入門範本](https://github.com/Azure/AzureStack-QuickStart-Templates)。

1. 匯入 AzureRM.TemplateValidator.psm1 PowerShell 模組：

    ```PowerShell
    cd "c:\AzureStack-Tools-master\TemplateValidator"
    Import-Module .\AzureRM.TemplateValidator.psm1
    ```

2. 執行範本驗證程式：

    ```PowerShell
    Test-AzureRMTemplate -TemplatePath <path to template.json or template folder> `
    -CapabilitiesPath <path to cloudcapabilities.json> `
    -Verbose
    ```

範本驗證警告或錯誤都會記錄到 PowerShell 主控台和來源目錄中的 HTML 檔案。 下列螢幕擷取畫面顯示驗證報告範例：

![範本驗證報告](./media/azure-stack-validate-templates/image1.png)

### <a name="parameters"></a>參數

範本驗證程式支援下列參數。

| 參數 | 說明 | 必要 |
| ----- | -----| ----- |
| TemplatePath | 指定要在其中遞迴尋找 Azure Resource Manager 範本的路徑 | 是 | 
| TemplatePattern | 指定要比對的範本檔案名稱。 | 否 |
| CapabilitiesPath | 指定雲端功能 JSON 檔案的路徑 | 是 | 
| IncludeComputeCapabilities | 包括 IaaS 資源 (例如 VM 大小與 VM 擴充功能) 的評估 | 否 |
| IncludeStorageCapabilities | 包括儲存體資源 (例如 SKU 類型) 的評估 | 否 |
| 報告 | 指定所產生之 HTML 報告的名稱 | 否 |
| 詳細資訊 | 將錯誤與警告記錄到主控台 | 否|

### <a name="examples"></a>範例

這個範例會驗證已下載到本機儲存體的所有 [Azure Stack 快速入門範本](https://github.com/Azure/AzureStack-QuickStart-Templates)。 此範例也會針對 Azure Stack 開發套件功能，驗證虛擬機器大小和擴充功能。

```PowerShell
test-AzureRMTemplate -TemplatePath C:\AzureStack-Quickstart-Templates `
-CapabilitiesPath .\TemplateValidator\AzureStackCloudCapabilities_with_AddOns_20170627.json `
-TemplatePattern MyStandardTemplateName.json`
-IncludeComputeCapabilities`
-Report TemplateReport.html
```

## <a name="next-steps"></a>後續步驟

- [將範本部署到 Azure Stack](azure-stack-arm-templates.md)
- [開發適用於 Azure Stack 的範本](azure-stack-develop-templates.md)
