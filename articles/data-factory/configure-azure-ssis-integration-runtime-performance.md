---
title: 設定 Azure-SSIS Integration Runtime 效能 | Microsoft Docs
description: 了解如何設定 Azure-SSIS Integration Runtime 的屬性，以獲得高效能
services: data-factory
ms.date: 01/10/2018
ms.topic: conceptual
ms.service: data-factory
ms.workload: data-services
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: 2592c81947f48c10891fe920647612d5c30af64f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989070"
---
# <a name="configure-the-azure-ssis-integration-runtime-for-high-performance"></a>設定 Azure-SSIS Integration Runtime 以獲得高效能

本文說明如何設定 Azure-SSIS Integration Runtime (IR) 以獲得高效能。 Azure-SSIS IR 可讓您在 Azure 中部署及執行 SQL Server Integration Services (SSIS) 套件。 如需 Azure-SSIS IR 的詳細資訊，請參閱[整合執行階段](concepts-integration-runtime.md#azure-ssis-integration-runtime)一文。 如需在 Azure 上部署及執行 SSIS 套件的相關資訊，請參閱[將 SQL Server Integration Services 工作負載隨即轉移至雲端](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview)。

> [!IMPORTANT]
> 本文包含 SSIS 開發小組成員所完成之內部測試的效能結果與觀察。 您的結果可能會有所不同。 在您完成組態設定之前執行您自己的測試，這會同時影響成本和效能。

## <a name="properties-to-configure"></a>要設定的屬性

下列設定指令碼部分會顯示當您建立 Azure-SSIS Integration Runtime 時，您可以設定的屬性。 如需完整的 PowerShell 指令碼與描述，請參閱[將 SQL Server Integration Services 套件部署至 Azure](tutorial-deploy-ssis-packages-azure-powershell.md)。

```powershell
$SubscriptionName = "<Azure subscription name>"
$ResourceGroupName = "<Azure resource group name>"
# Data factory name. Must be globally unique
$DataFactoryName = "<Data factory name>" 
$DataFactoryLocation = "EastUS" 

# Azure-SSIS integration runtime information. This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "<Specify a name for your Azure-SSIS IR>"
$AzureSSISDescription = "<Specify description for your Azure-SSIS IR"
# Only EastUS, NorthEurope, and WestEurope are supported.
$AzureSSISLocation = "EastUS" 
# Only Standard_A4_v2, Standard_A8_v2, Standard_D1_v2, Standard_D2_v2, Standard_D3_v2, Standard_D4_v2 are supported
$AzureSSISNodeSize = "Standard_D3_v2"
# Only 1-10 nodes are supported.
$AzureSSISNodeNumber = 2 
# For a Standard_D1_v2 node, 1-4 parallel executions per node are supported. For other nodes, it's 1-8.
$AzureSSISMaxParallelExecutionsPerNode = 2 

# SSISDB info
$SSISDBServerEndpoint = "<Azure SQL server name>.database.windows.net"
$SSISDBServerAdminUserName = "<Azure SQL server - user name>"
$SSISDBServerAdminPassword = "<Azure SQL server - user password>"
# Remove the SSISDBPricingTier variable if you are using Azure SQL Database Managed Instance
# This parameter applies only to Azure SQL Database. For the basic pricing tier, specify "Basic", not "B". For standard tiers, specify "S0", "S1", "S2", 'S3", etc.
$SSISDBPricingTier = "<pricing tier of your Azure SQL server. Examples: Basic, S0, S1, S2, S3, etc.>"
```

## <a name="azuressislocation"></a>AzureSSISLocation
**AzureSSISLocation** 是整合執行階段背景工作節點的位置。 背景工作節點會維護 Azure SQL Database 上 SSIS 目錄資料庫 (SSISDB) 的持續連線。 將 **AzureSSISLocation** 設定為與裝載 SSISDB 之 SQL Database 伺服器相同的位置，這可讓整合執行階段盡可能有效率地運作。

## <a name="azuressisnodesize"></a>AzureSSISNodeSize
Data Factory (包括 Azure-SSIS IR) 支援下列選項：
-   Standard\_A4\_v2
-   Standard\_A8\_v2
-   Standard\_D1\_v2
-   Standard\_D2\_v2
-   Standard\_D3\_v2
-   Standard\_D4\_v2.

在由 SSIS 工程團隊進行的非官方內部測試中，D 系列似乎比 A 系列更適合 SSIS 套件執行。

-   D 系列的效能/價格比高於 A 系列。
-   相同價格下，D 系列的輸送量高於 A 系列。

### <a name="configure-for-execution-speed"></a>針對執行速度進行設定
如果您不需要執行許多套件，且希望套件可以快速執行，請使用下列圖表中的資訊，以選擇適合您案例的虛擬機器類型。

此資料表示單一背景工作節點上的單一套件執行。 套件會從 Azure Blob 儲存體載入包含姓氏和名字資料行的千萬筆記錄、產生全名資料行，並將全名長度超過 20 個字元的記錄寫入至 Azure Blob 儲存體。

![SSIS Integration Runtime 套件執行速度](media/configure-azure-ssis-integration-runtime-performance/ssisir-execution-speed.png)

### <a name="configure-for-overall-throughput"></a>針對整體輸送量進行設定

如果您需要執行許多套件，且很重視整體輸送量，請使用下列圖表中的資訊，以選擇適合您案例的虛擬機器類型。

![SSIS Integration Runtime 最大整體輸送量](media/configure-azure-ssis-integration-runtime-performance/ssisir-overall-throughput.png)

## <a name="azuressisnodenumber"></a>AzureSSISNodeNumber

**AzureSSISNodeNumber** 會調整整合執行階段的延展性。 整合執行階段的輸送量是與 **AzureSSISNodeNumber** 成比例。 一開始先將 **AzureSSISNodeNumber** 設定為較小的值、監視整合執行階段的輸送量，然後根據您的案例調整值。 若要重新設定背景工作節點計數，請參閱[管理 Azure-SSIS Integration Runtime](manage-azure-ssis-integration-runtime.md)。

## <a name="azuressismaxparallelexecutionspernode"></a>AzureSSISMaxParallelExecutionsPerNode

當您已經使用功能強大的背景工作節點來執行套件時，增加 **AzureSSISMaxParallelExecutionsPerNode** 可能會增加整合執行階段的整體輸送量。 針對 Standard_D1_v2 節點，可支援每個節點有 1-4 個平行執行。 針對所有其他類型的節點，可支援每個節點有 1-8 個平行執行。
您可以根據套件的成本和背景工作角色節點的下列設定，評估適當的值。 如需詳細資訊，請參閱[一般用途的虛擬機器大小](../virtual-machines/windows/sizes-general.md)。

| 大小             | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | 最大暫存儲存體輸送量：IOPS / 讀取 MBps / 寫入 MBps | 最大資料磁碟 / 輸送量︰IOPS | 最大 NIC/預期的網路效能 (Mbps) |
|------------------|------|-------------|------------------------|------------------------------------------------------------|-----------------------------------|------------------------------------------------|
| Standard\_D1\_v2 | 1    | 3.5         | 50                     | 3000 / 46 / 23                                             | 2 / 2x500                         | 2 / 750                                        |
| Standard\_D2\_v2 | 2    | 7           | 100                    | 6000 / 93 / 46                                             | 4 / 4x500                         | 2 / 1500                                       |
| Standard\_D3\_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 8 / 8x500                         | 4 / 3000                                       |
| Standard\_D4\_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 16 / 16x500                       | 8 / 6000                                       |
| Standard\_A4\_v2 | 4    | 8           | 40                     | 4000 / 80 / 40                                             | 8 / 8x500                         | 4 / 1000                                       |
| Standard\_A8\_v2 | 8    | 16          | 80                     | 8000 / 160 / 80                                            | 16 / 16x500                       | 8 / 2000                                       |

以下是為 **AzureSSISMaxParallelExecutionsPerNode** 屬性設定正確值的指導方針： 

1. 一開始請將它設定為較小的值。
2. 少量增加，以檢查整體輸送量是否已獲得改善。
3. 當整體輸送量達到最大值時，停止增加此值。

## <a name="ssisdbpricingtier"></a>SSISDBPricingTier

**SSISDBPricingTier** 是 Azure SQL Database 上 SSIS 目錄資料庫 (SSISDB) 的定價層。 此設定會影響 IR 執行個體中的背景工作數目上限、將套件執行排入佇列的速度，以及載入執行記錄的速度。

-   如果您不在意將套件執行排入佇列的速度和載入執行記錄的速度，則可以選擇最低資料庫定價層。 使用基本定價的 Azure SQL Database 支援一個整合執行階段執行個體中 8 個背景工作。

-   如果背景工作計數超過 8 個或核心計數超過 50 個，則請選擇比「基本」更強大的資料庫。 否則，資料庫會變成整合執行階段執行個體的瓶頸，而且會嚴重影響整體效能。

您也可以根據 Azure 入口網站上提供的[資料庫交易單位](../sql-database/sql-database-what-is-a-dtu.md) (DTU) 使用量資訊，調整資料庫定價層。

## <a name="design-for-high-performance"></a>為高效能而設計
設計一個在 Azure 上執行的 SSIS 套件，不同於設計一個內部部署執行的套件。 相對於將多個獨立工作結合在相同的套件中，Azure-SSIS IR 中將它們分成數個套件，以求更有效率地執行。 針對每個套件建立套件執行，這樣一來就不用等待彼此完成。 這種方法受益於 Azure-SSIS Integration Runtime 的延展性，並可改善整體輸送量。

## <a name="next-steps"></a>後續步驟
深入了解 Azure-SSIS Integration Runtime。 請參閱 [ Azure-SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime)。
