---
title: PowerShell 指令碼：使用 Azure Data Factory 在雲端中複製資料 | Microsoft Docs
description: 此 PowerShell 指令碼會將資料從 Azure Blob 儲存體中的一個位置複製到相同 Blob 儲存體中的其他位置。
services: data-factory
author: linda33wj
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/12/2017
ms.author: jingwang
ms.openlocfilehash: 1fbf09968c6dc3dfc60b27656691504a363a5a6b
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43127032"
---
# <a name="use-powershell-to-create-a-data-factory-pipeline-to-copy-data-in-the-cloud"></a>使用 PowerShell 建立資料處理站管線，以在雲端中複製資料

這個範例 PowerShell 指令碼會在 Azure Data Factory 中建立管線，以將資料從 Azure Blob 儲存體中的一個位置複製到另一個位置。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>必要條件
* **Azure 儲存體帳戶**。 您需要使用 Blob 儲存體作為**來源**和**接收**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md)來建立帳戶。 
* 在 Blob 儲存體中建立一個 **Blob 容器**，在容器中建立一個輸入**資料夾**，然後上傳一些檔案到資料夾中。 您可以使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)之類的工具連線到 Azure Blob 儲存體，建立 Blob 容器，上傳輸入檔，以及驗證輸出檔。

## <a name="sample-script"></a>範例指令碼

> [!IMPORTANT]
> 這個指令碼會建立 JSON 檔案，該檔案定義硬碟上 c:\ 資料夾中的 Data Factory 實體 (已連結的服務、資料集和管線)。

[!code-powershell[main](../../../powershell_scripts/data-factory/copy-from-azure-blob-to-blob/copy-from-azure-blob-to-blob.ps1 "Copy from Blob Storage -> Blob Storage")]


## <a name="clean-up-deployment"></a>清除部署

執行範例指令碼之後，您可以使用下列命令以移除資源群組及與其相關聯的所有資源：

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $resourceGroupName
```
若要從資源群組移除資料處理站，請執行下列命令： 

```powershell
Remove-AzureRmDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令： 

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [Set-AzureRmDataFactoryV2](/powershell/module/azurerm.datafactoryv2/set-azurermdatafactoryv2) | 建立資料處理站。 |
| [Set-AzureRmDataFactoryV2LinkedService](/powershell/module/azurerm.datafactoryv2/Set-azurermdatafactoryv2linkedservice) | 在資料處理站中建立已連結的服務。 已連結的服務會將資料存放區或計算連結到資料處理站。 |
| [Set-AzureRmDataFactoryV2Dataset](/powershell/module/azurerm.datafactoryv2/Set-azurermdatafactoryv2dataset) | 在資料處理站中建立資料集。 資料集代表管線中活動的輸入/輸出。 | 
| [Set-AzureRmDataFactoryV2Pipeline](/powershell/module/azurerm.datafactoryv2/Set-azurermdatafactoryv2pipeline) | 在資料處理站中建立管線。 管線包含一或多個執行特定作業的活動。 在此管線中，複製活動會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 |
| [Invoke-AzureRmDataFactoryV2Pipeline](/powershell/module/azurerm.datafactoryv2/Invoke-azurermdatafactoryv2pipeline) | 建立管線的執行。 也就是說，執行管線。 |
| [Get-AzureRmDataFactoryV2ActivityRun](/powershell/module/azurerm.datafactoryv2/get-azurermdatafactoryv2activityrun) | 在管線中取得活動的執行 (活動執行) 的相關詳細資料。 
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |
|||

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/)。

您可以在 [Azure Data Factory PowerShell 範例](../samples-powershell.md)中，找到其他 Azure Data Factory PowerShell 指令碼範例。
