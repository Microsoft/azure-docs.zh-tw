---
title: 在 Azure (傳統) 中的 Windows VM 上將 Key Vault 與 SQL Server 整合 | Microsoft Docs
description: 了解如何自動化 SQL Server 加密的組態，以用於 Azure 金鑰保存庫。 本主題說明如何將「Azure 金鑰保存庫整合」與在傳統部署模型中建立的 SQL Server 虛擬機器搭配使用。
services: virtual-machines-windows
documentationcenter: ''
author: rothja
manager: craigg
editor: ''
tags: azure-service-management
ms.assetid: ab8d41a7-1971-4032-ab71-eb435c455dc1
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 02/17/2017
ms.author: jroth
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 203b3f79e5cca93557b3aa69c5774570c9e57022
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38719530"
---
# <a name="configure-azure-key-vault-integration-for-sql-server-on-azure-virtual-machines-classic"></a>在 Azure 虛擬機器上設定 SQL Server 的 Azure Key Vault 整合 (傳統)
> [!div class="op_single_selector"]
> * [Resource Manager](../sql/virtual-machines-windows-ps-sql-keyvault.md)
> * [傳統](../classic/ps-sql-keyvault.md)
> 
> 

## <a name="overview"></a>概觀
有多個 SQL Server 加密功能，例如[透明資料加密 (TDE)](https://msdn.microsoft.com/library/bb934049.aspx)、[資料行層級加密 (CLE)](https://msdn.microsoft.com/library/ms173744.aspx) 和[備份加密](https://msdn.microsoft.com/library/dn449489.aspx)。 這些形式的加密需要您管理和儲存用來加密的密碼編譯金鑰。 Azure 金鑰保存庫 (AKV) 服務是設計來改善這些金鑰在安全且高度可用位置的安全性和管理。 [SQL Server 連接器](http://www.microsoft.com/download/details.aspx?id=45344) 讓 SQL Server 可以從 Azure 金鑰保存庫使用這些金鑰。

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。

如果您使用內部部署機器來執行 SQL Server，有 [您可以遵循以從內部部署 SQL Server 機器存取 Azure 金鑰保存庫的步驟](https://msdn.microsoft.com/library/dn198405.aspx)。 但是對於 Azure VM 中的 SQL server，您可以使用 *Azure 金鑰保存庫整合* 功能來節省時間。 使用一些 Azure PowerShell Cmdlet 來啟用這項功能，您可以自動化 SQL VM 存取您的金鑰保存庫所需的組態。

啟用這項功能時，它會自動安裝 SQL Server 連接器、設定 EKM 提供者來存取 Azure 金鑰保存庫，並建立認證讓您存取您的保存庫。 如果您看到先前提及的內部部署文件中的步驟，您可以發現這項功能會自動執行步驟 2 和 3。 您唯一仍然需要手動進行的是建立金鑰保存庫和金鑰。 從那裡開始，會自動化 SQL VM 的整個設定。 這項功能完成此設定之後，您可以執行 T-SQL 陳述式以開始如往常一般加密您的資料庫或備份。

[!INCLUDE [AKV Integration Prepare](../../../../includes/virtual-machines-sql-server-akv-prepare.md)]

## <a name="configure-akv-integration"></a>設定 AKV 整合
使用 PowerShell 設定 Azure 金鑰保存庫整合。 以下章節提供必要參數的概觀，以及範例 PowerShell 指令碼。

### <a name="install-the-sql-server-iaas-extension"></a>安裝 SQL Server IaaS 擴充功能
首先， [安裝 SQL Server IaaS 擴充功能](../classic/sql-server-agent-extension.md)。

### <a name="understand-the-input-parameters"></a>了解輸入參數
下表列出在下一節中執行 PowerShell 指令碼所需的參數。

| 參數 | 說明 | 範例 |
| --- | --- | --- |
| **$akvURL** |**金鑰保存庫 URL** |"https://contosokeyvault.vault.azure.net/" |
| **$spName** |**服務主體名稱** |"fde2b411-33d5-4e11-af04eb07b669ccf2" |
| **$spSecret** |**服務主體密碼** |"9VTJSQwzlFepD8XODnzy8n2V01Jd8dAjwm/azF1XDKM=" |
| **$credName** |**認證名稱**：AKV 整合會在 SQL Server 內建立認證，允許 VM 具有金鑰保存庫的存取權。 選擇此認證的名稱。 |"mycred1" |
| **$vmName** |**虛擬機器名稱**：先前建立之 SQL VM 的名稱。 |"myvmname" |
| **$serviceName** |**服務名稱**：與 SQL VM 相關聯的雲端服務名稱。 |"mycloudservicename" |

### <a name="enable-akv-integration-with-powershell"></a>使用 PowerShell 啟用 AKV 整合
**New-AzureVMSqlServerKeyVaultCredentialConfig** Cmdlet 會建立 Azure 金鑰保存庫整合功能的組態物件。 **Set-AzureVMSqlServerExtension** 會以 **KeyVaultCredentialSettings** 參數設定此整合。 下列步驟說明如何使用這些命令。

1. 在 Azure PowerShell 中，先以您的特定值設定輸入參數，如本本主題前面幾節所述。 下列指令碼是範例。
   
        $akvURL = "https://contosokeyvault.vault.azure.net/"
        $spName = "fde2b411-33d5-4e11-af04eb07b669ccf2"
        $spSecret = "9VTJSQwzlFepD8XODnzy8n2V01Jd8dAjwm/azF1XDKM="
        $credName = "mycred1"
        $vmName = "myvmname"
        $serviceName = "mycloudservicename"
2. 然後使用下列指令碼來設定和啟用 AKV 整合。
   
        $secureakv =  $spSecret | ConvertTo-SecureString -AsPlainText -Force
        $akvs = New-AzureVMSqlServerKeyVaultCredentialConfig -Enable -CredentialName $credname -AzureKeyVaultUrl $akvURL -ServicePrincipalName $spName -ServicePrincipalSecret $secureakv
        Get-AzureVM -ServiceName $serviceName -Name $vmName | Set-AzureVMSqlServerExtension -KeyVaultCredentialSettings $akvs | Update-AzureVM

SQL IaaS 代理程式延伸會使用這個新的組態更新 SQL VM。

[!INCLUDE [AKV Integration Next Steps](../../../../includes/virtual-machines-sql-server-akv-next-steps.md)]

