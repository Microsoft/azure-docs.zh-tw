---
title: 適用於 Windows 的 Azure 效能診斷 VM 擴充功能 |Microsoft Docs
description: 介紹適用於 Windows 的 Azure 效能診斷 VM 擴充功能。
services: virtual-machines-windows'
documentationcenter: ''
author: genlin
manager: cshepard
editor: na
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: 0482a882b2dea47752eb38eadbaaa72e36ae2eae
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411685"
---
# <a name="azure-performance-diagnostics-vm-extension-for-windows"></a>適用於 Windows 的 Azure 效能診斷 VM 擴充功能

Azure 效能診斷 VM 擴充功能可協助從 Windows VM 收集效能診斷資料。 此擴充功能可執行分析和提供結果與建議報告，以識別並解決虛擬機器上的效能問題。 此擴充功能會安裝名為 [PerfInsights](http://aka.ms/perfinsights) 的疑難排解工具。

> [!NOTE]
    > 如果您想要從 Azure 入口網站在您的 VM 上針對非傳統 VM 執行診斷，建議使用新體驗。 如需詳細資訊，請參閱 [Azure 虛擬機器的效能診斷](performance-diagnostics.md) 

## <a name="prerequisites"></a>必要條件

此擴充功能可安裝於 Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2 及 Windows Server 2016。 它也可以安裝於 Windows 8.1 和 Windows 10。

## <a name="extension-schema"></a>擴充功能結構描述
下列 JSON 會顯示 Azure 效能診斷 VM 擴充功能的結構描述。 此擴充功能需要儲存體帳戶的名稱與金鑰，才能儲存診斷輸出與報告。 這些都是機密值。 儲存體帳戶金鑰應該儲存在受保護的設定組態內。 Azure VM 擴充功能保護的設定資料會經過加密，只會在目標虛擬機器上解密。 請注意，**storageAccountName** 和 **storageAccountKey** 須區分大小寫。 其他必要的參數會列在下一節中。

```JSON
    {
      "name": "[concat(parameters('vmName'),'/AzurePerformanceDiagnostics')]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2015-06-15",
      "properties": {
        "publisher": "Microsoft.Azure.Performance.Diagnostics",
        "type": "AzurePerformanceDiagnostics",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "storageAccountName": "[parameters('storageAccountName')]",
            "performanceScenario": "[parameters('performanceScenario')]",
            "traceDurationInSeconds": "[parameters('traceDurationInSeconds')]",
            "perfCounterTrace": "[parameters('perfCounterTrace')]",
            "networkTrace": "[parameters('networkTrace')]",
            "xperfTrace": "[parameters('xperfTrace')]",
            "storPortTrace": "[parameters('storPortTrace')]",
            "srNumber": "[parameters('srNumber')]",
            "requestTimeUtc":  "[parameters('requestTimeUtc')]",
            "resourceId": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        },
        "protectedSettings": {
            "storageAccountKey": "[parameters('storageAccountKey')]"        
        }
      }
    }
```

### <a name="property-values"></a>屬性值

|   **名稱**   |**值 / 範例**|       **說明**      |
|--------------|-------------------|----------------------------|
|apiVersion|2015-06-15|API 的版本。
|publisher|Microsoft.Azure.Performance.Diagnostics|擴充功能的發行者命名空間。
|type|AzurePerformanceDiagnostics|VM 擴充功能的類型。
|typeHandlerVersion|1.0|擴充功能處理常式的版本。
|performanceScenario|basic|要對其擷取資料的效能案例。 有效值為：**basic**、**vmslow**、**azurefiles** 及 **custom**。
|traceDurationInSeconds|300|追蹤的持續時間 (若有選取任何追蹤選項)。
|perfCounterTrace|p|啟用效能計數器追蹤的選項。 有效值為 **p** 或空值。 如果您不想要擷取此追蹤，請將值保持空白即可。
|networkTrace|n|用於啟用網路追蹤的選項。 有效值為 **n** 或空值。 如果您不想要擷取此追蹤，請將值保持空白即可。
|xperfTrace|x|啟用 XPerf 追蹤的選項。 有效值為 **x** 或空值。 如果您不想要擷取此追蹤，請將值保持空白即可。
|storPortTrace|s|啟用 StorPort 追蹤的選項。 有效值為 **s** 或空值。 如果您不想要擷取此追蹤，請將值保持空白即可。
|srNumber|123452016365929|支援票證號碼 (若可用)。 如果您沒有此值，請保持空白。
|requestTimeUtc|2017-09-28T22:08:53.736Z|目前的日期時間 (UTC)。 如果您使用入口網站來安裝此擴充功能，就不需提供此值。
|ResourceId|/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}|VM 的唯一識別碼。
|storageAccountName|mystorageaccount|要儲存診斷記錄和結果的儲存體帳戶名稱。
|storageAccountKey|lDuVvxuZB28NNP…hAiRF3voADxLBTcc==|儲存體帳戶的金鑰。

## <a name="install-the-extension"></a>安裝擴充功能

遵循下列指示以在 Windows 虛擬機器上安裝擴充功能：

1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 選取您要安裝此擴充功能的虛擬機器。

    ![Azure 入口網站的螢幕擷取畫面 (已醒目提示 [虛擬機器])](media/performance-diagnostics-vm-extension/select-the-virtual-machine.png)
3. 選取 [擴充功能] 刀鋒視窗，然後選取 [新增]。

    ![[擴充功能] 刀鋒視窗的螢幕擷取畫面 (已醒目提示 [新增])](media/performance-diagnostics-vm-extension/select-extensions.png)
4. 選取 [Azure 效能診斷]，檢閱條款及條件，然後選取 [建立]。

    ![新資源畫面的螢幕擷取畫面 (已醒目提示 [Azure 效能診斷])](media/performance-diagnostics-vm-extension/create-azure-performance-diagnostics-extension.png)
5. 提供安裝的參數值，然後選取 [確定] 以安裝擴充功能。 如需支援案例的詳細資訊，請參閱[如何使用 PerfInsights](how-to-use-perfInsights.md#supported-troubleshooting-scenarios)。 

    ![安裝擴充功能對話方塊的螢幕擷取畫面](media/performance-diagnostics-vm-extension/install-the-extension.png)
6. 安裝成功時，您會看到指出此狀態的訊息。

    ![佈建成功訊息的螢幕擷取畫面](media/performance-diagnostics-vm-extension/provisioning-succeeded-message.png)

    > [!NOTE]
    > 此擴充功能會在佈建成功時執行。 完成基本案例需要兩分鐘或更少的時間。 對於其他案例，它會在於安裝期間所指定的持續時間內持續執行。

## <a name="remove-the-extension"></a>移除擴充功能
若要從虛擬機器移除擴充功能，請遵循下列步驟：

1. 登入 [Azure 入口網站](http://portal.azure.com)，選取您要從中移除此擴充功能的虛擬機器，然後選取 [擴充功能] 刀鋒視窗。 
2. 從清單中選取效能診斷擴充功能項目的 (**...**)，然後選取 [解除安裝]。

    ![[擴充功能] 刀鋒視窗的螢幕擷取畫面 (已醒目提示 [解除安裝])](media/performance-diagnostics-vm-extension/uninstall-the-extension.png)

    > [!NOTE]
    > 您也可以選取擴充功能項目，然後選取 [解除安裝] 選項。

## <a name="template-deployment"></a>範本部署
可以使用 Azure Resource Manager 範本部署 Azure 虛擬機器擴充功能。 上一節中詳述的 JSON 結構描述可使用於 Azure Resource Manager 範本。 這會在 Azure Resource Manager 範本部署期間執行 Azure 效能診斷 VM 擴充功能。 以下是範例範本：

````
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "yourVMName"
    },
    "location": {
      "type": "string",
      "defaultValue": "southcentralus"
    },
    "storageAccountName": {
      "type": "securestring"
      "defaultValue": "yourStorageAccount"
    },
    "storageAccountKey": {
      "type": "securestring"
      "defaultValue": "yourStorageAccountKey"
    },
    "performanceScenario": {
      "type": "string",
      "defaultValue": "basic"
    },
    "srNumber": {
      "type": "string",
      "defaultValue": ""
    },
    "traceDurationInSeconds": {
      "type": "int",
    "defaultValue": 300
    },
    "perfCounterTrace": {
      "type": "string",
      "defaultValue": "p"
    },
    "networkTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "xperfTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "storPortTrace": {
      "type": "string",
      "defaultValue": ""
    },
    "requestTimeUtc": {
      "type": "string",
      "defaultValue": "10/2/2017 11:06:00 PM"
    }       
  },
  "resources": [
    {
      "name": "[concat(parameters('vmName'),'/AzurePerformanceDiagnostics')]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2015-06-15",
      "properties": {
        "publisher": "Microsoft.Azure.Performance.Diagnostics",
        "type": "AzurePerformanceDiagnostics",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "storageAccountName": "[parameters('storageAccountName')]",
            "performanceScenario": "[parameters('performanceScenario')]",
            "traceDurationInSeconds": "[parameters('traceDurationInSeconds')]",
            "perfCounterTrace": "[parameters('perfCounterTrace')]",
            "networkTrace": "[parameters('networkTrace')]",
            "xperfTrace": "[parameters('xperfTrace')]",
            "storPortTrace": "[parameters('storPortTrace')]",
            "srNumber": "[parameters('srNumber')]",
            "requestTimeUtc":  "[parameters('requestTimeUtc')]",
            "resourceId": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        },
        "protectedSettings": {
            "storageAccountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
  ]
}
````

## <a name="powershell-deployment"></a>PowerShell 部署
`Set-AzureRmVMExtension` 命令可用來將 Azure 效能診斷 VM 擴充功能部署到現有的虛擬機器。

PowerShell

````
$PublicSettings = @{ "storageAccountName"="mystorageaccount";"performanceScenario"="basic";"traceDurationInSeconds"=300;"perfCounterTrace"="p";"networkTrace"="";"xperfTrace"="";"storPortTrace"="";"srNumber"="";"requestTimeUtc"="2017-09-28T22:08:53.736Z";"resourceId"="VMResourceId" }
$ProtectedSettings = @{"storageAccountKey"="mystoragekey" }

Set-AzureRmVMExtension -ExtensionName "AzurePerformanceDiagnostics" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.Azure.Performance.Diagnostics" `
    -ExtensionType "AzurePerformanceDiagnostics" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS
````

## <a name="information-on-the-data-captured"></a>所擷取資料的詳細資訊
PerfInsights 工具會根據所選取的案例，收集各種記錄、組態和診斷資料。 如需詳細資訊，請參閱 [PerfInsights 文件](http://aka.ms/perfinsights)。

## <a name="view-and-share-the-results"></a>檢視並共用結果

在上傳到安裝期間指定的儲存體帳戶及使用[共用存取簽章 (SAS)](../../storage/common/storage-dotnet-shared-access-signature-part-1.md) 共用 30 天的 zip 檔案中，可找到延伸模組的輸出。 這個 zip 檔案包含診斷記錄及具有結果和建議的報告。 連結至輸出 zip 檔案的 SAS 連結，可以在 **C:\Packages\Plugins\Microsoft.Azure.Performance.Diagnostics.AzurePerformanceDiagnostics\\\<version>** 資料夾底下名為 zipfilename_saslink.txt 的文字檔中找到。 任何具有此連結的人員都能夠下載該 zip 檔案。

若要協助支援工程師處理支援票證，Microsoft 可能會使用此 SAS 連結來下載診斷資料。

若要檢視報告，請將 zip 檔案解壓縮，然後開啟 **PerfInsights Report.html** 檔案。

您也可以選取擴充功能來直接從入口網站下載 zip 檔案。

![效能診斷詳細狀態的螢幕擷取畫面](media/performance-diagnostics-vm-extension/view-detailed-status.png)

> [!NOTE]
> 顯示在入口網站中的 SAS 連結有時候可能無法運作。 編碼和解碼作業期間的錯誤 URL 可能會造成此問題。 您可以改由直接從 VM 的 *_saslink.txt 檔案取得該連結。

## <a name="troubleshoot-and-support"></a>疑難排解與支援

- 即使已成功佈建擴充功能，擴充功能部署狀態 (位於通知區域中) 還是有可能會顯示為「部署進行中」。

    只要擴充功能狀態表示已成功佈建擴充功能，就可以放心地忽略這個問題。
- 您可以使用擴充功能記錄來解決安裝期間發生的一些問題。 擴充功能執行輸出會記錄至下列目錄中的檔案︰

        C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Performance.Diagnostics.AzurePerformanceDiagnostics\<version>

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
