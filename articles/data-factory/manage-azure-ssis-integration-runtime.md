---
title: 重新設定 Azure-SSIS 整合執行階段 | Microsoft Docs
description: 了解如何在佈建 Azure Data Factory 中的 Azure SSIS 整合執行階段後重新設定。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: 7d4accf7958b93addd1b6d99e2d40a447c32dba9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956586"
---
# <a name="reconfigure-the-azure-ssis-integration-runtime"></a>重新設定 Azure-SSIS 整合執行階段
本文說明如何重新設定現有的 Azure-SSIS 整合執行階段。 若要了解如何在 Azure Data Factory 建立 Azure-SSIS 整合執行階段 (IR)，請瀏覽[建立 Azure SSIS 整合執行階段](create-azure-ssis-integration-runtime.md)，。  

## <a name="data-factory-ui"></a>Data Factory UI 
您可以使用 Data Factory UI 來停止、編輯/重新設定或刪除 Azure-SSIS IR。 

1. 在 **Data Factory UI** 中，切換至 [編輯] 索引標籤。若要啟動 Data Factory UI，請按一下您資料處理站首頁上的 [製作與監視]。
2. 在左窗格中，按一下 [連線]。
3. 在右窗格中，切換至 [整合執行階段]。 
4. 您可以使用 [動作] 資料行中的按鈕來**停止**、**編輯**或**刪除**整合執行階段。 [動作] 資料行中的 [程式碼] 按鈕可讓您檢視與整合執行階段關聯的 JSON 定義。  
    
    ![Azure SSIS IR 的動作](./media/manage-azure-ssis-integration-runtime/actions-for-azure-ssis-ir.png)

### <a name="to-reconfigure-an-azure-ssis-ir"></a>重新設定 Azure-SSIS IR
1. 按一下 [動作] 資料行中的 [停止] 來停止整合執行階段。 若要重新整理清單檢視，請按一下工具列上的 [重新整理]。 在 IR 停止之後，您會看到第一個動作會讓您啟動 IR。 

    ![Azure SSIS IR 的動作 - 停止後](./media/manage-azure-ssis-integration-runtime/actions-after-ssis-ir-stopped.png)
2. 按一下 [動作] 資料行中的 [編輯] 按鈕。 在 [整合執行階段設定] 視窗中，變更設定 (例如節點大小、節點數目，或每個節點的平行執行數上限)。 
3. 若要重新啟動 IR，請按一下 [動作] 資料行中的 [啟動] 按鈕。     

## <a name="azure-powershell"></a>Azure PowerShell
佈建並啟動 Azure SSIS 整合執行階段的執行個體之後，您可以執行一連串`Stop` - `Set` - `Start` PowerShell Cmdlet 來重新設定。 例如，下列 PowerShell 指令碼會將配置給 Azure-SSIS 整合執行階段執行個體的節點數目變更為五。

### <a name="reconfigure-an-azure-ssis-ir"></a>重新設定 Azure-SSIS IR

1. 首先，使用 [Stop-AzureRmDataFactoryV2IntegrationRuntime](/powershell/module/azurerm.datafactoryv2/stop-azurermdatafactoryv2integrationruntime?view=azurermps-4.4.1) Cmdlet 停止 Azure-SSIS 整合執行階段。 此命令會釋出其所有節點並停止計費。

    ```powershell
    Stop-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName 
    ```
2. 接著，使用 [Set-AzureRmDataFactoryV2IntegrationRuntime](/powershell/module/azurerm.datafactoryv2/set-azurermdatafactoryv2integrationruntime?view=azurermps-4.4.1) Cmdlet 重新設定 Azure-SSIS IR。 下列範例命令可將 Azure-SSIS 整合執行階段相應放大為五個節點。

    ```powershell
    Set-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -NodeCount 5
    ```  
3. 然後，使用 [Start-AzureRmDataFactoryV2IntegrationRuntime](/powershell/module/azurerm.datafactoryv2/start-azurermdatafactoryv2integrationruntime?view=azurermps-4.4.1) Cmdlet 開始 Azure-SSIS 整合執行階段。 此命令會配置其所有節點以執行 SSIS 套件。   

    ```powershell
    Start-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName
    ```

### <a name="delete-an-azure-ssis-ir"></a>刪除 Azure-SSIS IR
1. 首先，列出資料處理站下所有現有的 Azure SSIS IR。

    ```powershell
    Get-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName -Status
    ```
2. 接著，停止資料處理站下所有現有的 Azure SSIS IR。

    ```powershell
    Stop-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
    ```
3. 然後，逐一移除資料處理站下所有現有的 Azure SSIS IR。

    ```powershell
    Remove-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
    ```
4. 最後，移除您的資料處理站。

    ```powershell
    Remove-AzureRmDataFactoryV2 -Name $DataFactoryName -ResourceGroupName $ResourceGroupName -Force
    ```
5. 如果您已建立新的資源群組，請移除此資源群組。

    ```powershell
    Remove-AzureRmResourceGroup -Name $ResourceGroupName -Force 
    ```

## <a name="next-steps"></a>後續步驟
如需 Azure-SSIS 執行階段的詳細資訊，請參閱下列主題： 

- [Azure-SSIS 整合執行階段](concepts-integration-runtime.md#azure-ssis-integration-runtime)。 本文提供整合執行階段的一般概念性資訊，包括 Azure-SSIS IR。 
- [教學課程：將 SSIS 套件部署至 Azure](tutorial-create-azure-ssis-runtime-portal.md)。 本文逐步說明如何建立 Azure-SSIS IR，並使用 Azure SQL Database 裝載 SSIS 目錄。 
- [如何：建立 Azure-SSIS 整合執行階段](create-azure-ssis-integration-runtime.md)。 這篇文章會詳述教學課程，並提供使用 Azure SQL Database 受控執行個體，以及將 IR 加入虛擬網路的指示。 
- [將 Azure-SSIS IR 加入虛擬網路](join-azure-ssis-integration-runtime-virtual-network.md)。 這篇文章提供將 Azure SSIS IR 加入至 Azure 虛擬網路的概念資訊。 它也提供使用 Azure 入口網站來設定虛擬網路，好讓 Azure SSIS IR 可加入虛擬網路的步驟。 
- [監視 Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime). 本文示範如何在傳回的資訊中擷取 Azure-SSIS IR 的相關資訊和狀態描述。 
 
