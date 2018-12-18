---
title: Azure Cloud Shell 功能 | Microsoft Docs
description: Azure Cloud Shell 中 Bash 的功能概觀
services: Azure
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/13/2018
ms.author: juluk
ms.openlocfilehash: 1321645d97e7f6ff2faed1e61ddb608afcb7b413
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037967"
---
# <a name="features--tools-for-azure-cloud-shell"></a>Azure Cloud Shell 的功能和工具

[!INCLUDE [features-introblock](../../includes/cloud-shell-features-introblock.md)]

Azure Cloud Shell 會在 `Ubuntu 16.04 LTS` 上執行。

## <a name="features"></a>特性

### <a name="secure-automatic-authentication"></a>安全的自動驗證

Cloud Shell 會安全地自動驗證 Azure CLI 2.0 和 Azure PowerShell 的帳戶存取。

### <a name="home-persistence-across-sessions"></a>工作階段之間的 $Home 持續性

若要在工作階段之間保存檔案，Cloud Shell 會在第一次啟動時逐步引導您連結 Azure 檔案共用。
完成後，Cloud Shell 會自動連結儲存體 (掛接為 `$Home\clouddrive`)，供所有未來的工作階段使用。
此外，在 Azure 檔案共用中，您的 `$Home` 目錄會保存為 .img。
在工作階段之間不會保存 `$Home` 外面的檔案和電腦狀態。 將祕密儲存為 SSH 金鑰時，請使用最佳做法。 服務例如 [Azure Key Vault 具有設定的教學課程](https://docs.microsoft.com/azure/key-vault/key-vault-manage-with-cli2#prerequisites)。

[深入了解在 Cloud Shell 中保存檔案。](persisting-shell-storage.md)

### <a name="azure-drive-azure"></a>Azure 磁碟機 (Azure:)

Cloud Shell (預覽) 中的 PowerShell 可讓您在 Azure 磁碟機 (`Azure:`) 中開始。
Azure 磁碟機可讓您輕鬆探索和導覽與檔案系統導覽類似的 Azure 資源 (例如計算、網路、儲存體等等)。
無論您使用哪個磁碟機，都可以繼續使用熟悉的 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure) 來管理這些資源。
對 Azure 資源進行的任何變更 (在 Azure 入口網站中直接進行，或透過 Azure PowerShell Cmdlet) 都會反映在 Azure 磁碟機中。  您可以執行 `dir -Force` 以重新整理您的資源。

![](media/features-powershell/azure-drive.png)

### <a name="deep-integration-with-open-source-tooling"></a>與開放原始碼工具深度整合

Cloud Shell 包含預先設定的驗證，可用於開放原始碼工具，例如 Terraform、Ansible 和 Chef InSpec。 從範例逐步解說開始試試看。

## <a name="tools"></a>工具

|類別   |Name   |
|---|---|
|Linux 工具            |Bash<br> zsh<br> sh<br> tmux<br> dig<br>               |
|Azure 工具            |[Azure CLI 2.0](https://github.com/Azure/azure-cli) \(英文\) 和 [1.0](https://github.com/Azure/azure-xplat-cli) \(英文\)<br> [AzCopy](https://docs.microsoft.com/azure/storage/storage-use-azcopy)<br> [Service Fabric CLI](https://docs.microsoft.com/azure/service-fabric/service-fabric-cli) |
|文字編輯器           |vim<br> nano<br> emacs       |
|原始檔控制         |git                    |
|建置工具            |make<br> maven<br> npm<br> pip         |
|容器             |[Docker CLI](https://github.com/docker/cli) \(英文\)/[Docker Machine](https://github.com/docker/machine) \(英文\)<br> [Kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) \(英文\)<br> [Helm](https://github.com/kubernetes/helm)<br> [DC/OS CLI](https://github.com/dcos/dcos-cli) \(英文\)         |
|資料庫              |MySQL 用戶端<br> PostgreSql 用戶端<br> [sqlcmd 公用程式](https://docs.microsoft.com/sql/tools/sqlcmd-utility) \(英文\)<br> [mssql-scripter](https://github.com/Microsoft/sql-xplat-cli) |
|其他                  |iPython 用戶端<br> [Cloud Foundry CLI](https://github.com/cloudfoundry/cli) \(英文\)<br> [Terraform](https://www.terraform.io/docs/providers/azurerm/)<br> [Ansible](https://www.ansible.com/microsoft-azure)<br> [Chef InSpec](https://www.chef.io/inspec/)| 

## <a name="language-support"></a>語言支援

|語言   |版本   |
|---|---|
|.NET Core  |2.0.0       |
|Go         |1.9        |
|Java       |1.8        |
|Node.js    |8.9.4      |
|PowerShell |[6.1.0-preview.4](https://github.com/PowerShell/powershell/releases)       |
|Python     |2.7 和 3.5 (預設)|

## <a name="next-steps"></a>後續步驟
[Cloud Shell 中 Bash 的快速入門](quickstart.md) <br>
[Cloud Shell 中的 PowerShell (預覽) 快速入門](quickstart-powershell.md) <br>
[了解 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/) <br>
[了解 Azure PowerShell](https://docs.microsoft.com/powershell/azure/) <br>
