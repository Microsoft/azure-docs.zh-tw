---
title: 調整彈性集區資源 - Azure SQL Database | Microsoft Docs
description: 此頁面將針對 Azure SQL Database 中的彈性集區說明調整資源。
services: sql-database
ms.service: sql-database
subservice: elastic-pool
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: carlrab
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: 2b304ac26f9a18b0e98cb4c42de3ca386637d864
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47164714"
---
# <a name="scale-elastic-pool-resources-in-azure-sql-database"></a>在 Azure SQL Database 中調整彈性集區

本文說明如何在 Azure SQL Database 中調整彈性集區和集區資料庫可用的計算和儲存資源。 

## <a name="vcore-based-purchasing-model-change-elastic-pool-storage-size"></a>以虛擬核心為基礎的購買模型：變更彈性集區儲存體大小

- 可以將儲存體佈建到大小上限： 
 - 針對標準儲存體，以 10 GB 為增量單位增加或減少大小
 - 針對進階儲存體，以 250 GB 為增量單位增加或減少大小
- 增加或減少其大小上限，即可佈建彈性集區的儲存體。
- 彈性集區儲存體的價格為儲存體數量乘以服務層的儲存體單價。 如需有關額外儲存體的價格詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情況下，您可能需要壓縮資料庫來回收未使用的空間。 如需詳細資訊，請參閱[管理 Azure SQL Database 中的檔案空間](sql-database-file-space-management.md)。

## <a name="vcore-based-purchasing-model-change-elastic-pool-compute-resources-vcores"></a>以虛擬核心為基礎的購買模型：變更彈性集區計算資源 (虛擬核心)

您可以使用 [Azure 入口網站](sql-database-elastic-pool-scale.md#azure-portal-manage-elastic-pools-and-pooled-databases)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqlelasticpool)、[Azure CLI](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) 或 [REST API](/rest/api/sql/elasticpools/update)，根據資源需求來增加或減少彈性集區的計算大小。

- 重新調整集區虛擬核心時，會短暫中斷資料庫連線。 在為單一資料庫 (而非在集區中) 重新調整 DTU 時，也會發生相同的行為。 如需重新調整作業期間，資料庫中斷連線的持續時間和影響的詳細資訊，請參閱[重新調整單一資料庫的 DTU](#single-database-change-storage-size)。 
- 重新調整集區虛擬核心的持續時間可能取決於集區中所有資料庫使用的儲存空間總量。 一般情況下，重新調整延遲時間平均為每 100 GB 在 90 分鐘以內。 舉例來說，如果集區中所有資料庫使用的總空間為 200 GB，則重新調整集區的預期延遲時間為 3 小時內。 在標準層或基本層內的某些情況下，不論使用的空間量多寡，重新調整延遲時間可能會少於五分鐘。
- 一般而言，變更每個資料庫之虛擬核心下限或每個資料庫的虛擬核心上限的持續時間會在五分鐘內。
- 縮減集區虛擬核心的大小時，集區使用的空間必須小於目標服務層和集區虛擬核心允許的大小上限。

## <a name="dtu-based-purchasing-model-change-elastic-pool-storage-size"></a>以 DTU 為基礎的購買模型：變更彈性集區儲存體大小

- 彈性集區的 eDTU 價格包含一定數量不額外收費的儲存體。 佈建超過內含量的額外儲存體會產生額外費用，以 250 GB 為單位最多增加到大小上限 1 TB，超過 1 TB 則以 256 GB 為單位增加。 如需了解內含儲存體數量和大小上限，請參閱[彈性集區：儲存體大小與計算大小](sql-database-dtu-resource-limits-elastic-pools.md#elastic-pool-storage-sizes-and-compute-sizes)。
- 可藉由使用 [Azure 入口網站](sql-database-elastic-pool-scale.md#azure-portal-manage-elastic-pools-and-pooled-databases)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqlelasticpool)、[Azure CLI](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) 或 [REST API](/rest/api/sql/elasticpools/update) 增加其大小上限，以佈建彈性集區的額外儲存體。
- 彈性集區之額外儲存體的價格為額外儲存體數量乘以服務層的額外儲存體單價。 如需有關額外儲存體的價格詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情況下，您可能需要壓縮資料庫來回收未使用的空間。 如需詳細資訊，請參閱[管理 Azure SQL Database 中的檔案空間](sql-database-file-space-management.md)。

## <a name="dtu-based-purchasing-model-change-elastic-pool-compute-resources-edtus"></a>以 DTU 為基礎的購買模型：變更彈性集區計算資源 (eDTU)

您可以使用 [Azure 入口網站](sql-database-elastic-pool-scale.md#azure-portal-manage-elastic-pools-and-pooled-databases)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqlelasticpool)、[Azure CLI](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) 或 [REST API](/rest/api/sql/elasticpools/update)，根據資源需求來增加或減少彈性集區可用的資源。

- 重新調整集區 eDTU 時，會短暫中斷資料庫連線。 在為單一資料庫 (而非在集區中) 重新調整 DTU 時，也會發生相同的行為。 如需重新調整作業期間，資料庫中斷連線的持續時間和影響的詳細資訊，請參閱[重新調整單一資料庫的 DTU](#single-database-change-storage-size)。 
- 重新調整集區 eDTU 的持續時間可能取決於集區中所有資料庫使用的儲存空間總量。 一般情況下，重新調整延遲時間平均為每 100 GB 在 90 分鐘以內。 舉例來說，如果集區中所有資料庫使用的總空間為 200 GB，則重新調整集區的預期延遲時間為 3 小時內。 在標準層或基本層內的某些情況下，不論使用的空間量多寡，重新調整延遲時間可能會少於五分鐘。
- 一般而言，變更每個資料庫之 eDTU 下限或每個資料庫的 eDTU 上限的持續時間會在五分鐘內。
- 縮減集區 eDTU 的大小時，集區使用的空間必須小於目標服務層和集區 eDTU 允許的大小上限。
- 調整集區 eDTU 時，如果發生下列情況，會產生額外的儲存體成本：(1) 目標集區可支援集區的儲存體大小上限，而且 (2) 儲存體大小上限超過目標集區的內含儲存體數量。 例如，如果大小上限為 100 GB 的 100 eDTU 標準集區縮減為 50 eDTU 標準集區，則會產生額外的儲存體成本，因為目標集區可支援 100 GB 的大小上限，而且其內含儲存體數量只有 50 GB。 因此，額外的儲存體數量為 100 GB – 50 GB = 50 GB。 如需額外儲存體的價格詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。 如果實際的使用空間量小於內含的儲存體數量，則可將資料庫大小上限降低至內含量，以避免造成額外成本。 

## <a name="next-steps"></a>後續步驟

如需整體資源限制，請參閱[SQL Database 以虛擬核心為基礎的資源限制 - 彈性集區](sql-database-vcore-resource-limits-elastic-pools.md)和[SQL Database 以 DTU 為基礎的資源限制 - 彈性集區](sql-database-dtu-resource-limits-elastic-pools.md)。
