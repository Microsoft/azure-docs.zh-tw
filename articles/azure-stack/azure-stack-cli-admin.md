---
title: 為 Azure Stack 使用者啟用 Azure CLI | Microsoft Docs
description: 了解如何使用跨平台命令列介面 (CLI) 在 Azure Stack 上管理及部署資源
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: f576079c-5384-4c23-b5a4-9ae165d1e3c3
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/11/2018
ms.author: mabrigg
ms.openlocfilehash: 09c551ea7196ae20a60a5dd34c1cda889ff5df46
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47091044"
---
# <a name="enable-azure-cli-for-azure-stack-users"></a>為 Azure Stack 使用者啟用 Azure CLI

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以對 Azure Stack 的使用者提供 CA 根憑證，使其可以在其開發電腦上使用 Azure CLI。 使用者必須有該憑證才能透過 CLI 管理資源。

* **Azure Stack CA 根憑證** - 如果使用者是從位於 Azure Stack 開發套件外的工作站使用 CLI，就需要此根憑證。  

* **虛擬機器別名端點** - 此端點提供一個在部署 VM 時，以單一參數參考映像發行者、供應項目、SKU 及版本的別名 (例如 "UbuntuLTS" 或 "Win2012Datacenter")。  

下列各節會說明如何取得這些值。

## <a name="export-the-azure-stack-ca-root-certificate"></a>匯出 Azure Stack CA 根憑證

您可以在開發套件中，以及在開發套件環境內執行的租用戶虛擬機器中找到 Azure Stack CA 根憑證。 若要以 PEM 格式匯出 Azure Stack 根憑證，請登入您的開發套件或租用戶虛擬機器，然後執行下列指令碼：

```powershell
$label = "AzureStackSelfSignedRootCert"
Write-Host "Getting certificate from the current user trusted store with subject CN=$label"
$root = Get-ChildItem Cert:\CurrentUser\Root | Where-Object Subject -eq "CN=$label" | select -First 1
if (-not $root)
{
    Log-Error "Certificate with subject CN=$label not found"
    return
}

Write-Host "Exporting certificate"
Export-Certificate -Type CERT -FilePath root.cer -Cert $root

Write-Host "Converting certificate to PEM format"
certutil -encode root.cer root.pem
```

## <a name="set-up-the-virtual-machine-aliases-endpoint"></a>設定虛擬機器別名端點

Azure Stack 操作員應該設定裝載虛擬機器別名檔案的可公開存取端點。 虛擬機器別名檔案是為映像提供通用名稱的 JSON 檔案。 當 VM 部署為 Azure CLI 參數時，隨後就會指定該名稱。  

在將項目新增到別名檔案之前，請確定您是[從 Azure Marketplace 下載映像](azure-stack-download-azure-marketplace-item.md)，或[已發行自己的自訂映像](azure-stack-add-vm-image.md)。 如果您發行自訂映像，請記下您在發行時所指定的發行者、供應項目、SKU 及版本資訊。 如果映像來自市集，則您可以使用 ```Get-AzureVMImage``` Cmdlet 來檢視資訊。  

有一個包含許多常見映像別名的[別名檔案範例](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)可供使用。 您可使用該範例作為起點。 將此檔案裝載在 CLI 用戶端能夠存取的空間中。 其中一個做法是將檔案裝載在 Blob 儲存體帳戶中，然後與您的使用者分享該 URL：

1. 從 GitHub 下載[範例檔案](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)。
2. 在 Azure Stack 中建立新的儲存體帳戶。 完成之後，建立新的 Blob 容器。 將存取原則設定為「公用」。  
3. 將 JSON 檔案上傳到新的容器。 完成之後，您便可以選取 fblob 名稱，然後從 Blob 屬性中選取 URL 來檢視 Blob 的 URL。

## <a name="next-steps"></a>後續步驟

- [使用 Azure CLI 部署範本](azure-stack-deploy-template-command-line.md)

- [使用 PowerShell 連線](azure-stack-connect-powershell.md)

- [管理使用者權限](azure-stack-manage-permissions.md)
