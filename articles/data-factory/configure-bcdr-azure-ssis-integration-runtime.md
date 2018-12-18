---
title: 設定 Azure-SSIS Integration Runtime 以進行 SQL Database 容錯移轉 | Microsoft Docs
description: 本文說明如何為 SSISDB 資料庫設定具有 Azure SQL Database 異地複寫和容錯移轉的 Azure-SSIS Integration Runtime
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 08/14/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: 2012ccf4d9fd3e62ba248f29f922f868077e4061
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42141287"
---
# <a name="configure-the-azure-ssis-integration-runtime-with-azure-sql-database-geo-replication-and-failover"></a>設定具有 Azure SQL Database 異地複寫和容錯移轉的 Azure-SSIS Integration Runtime

本文說明如何為 SSISDB 資料庫設定具有 Azure SQL Database 異地複寫的 Azure-SSIS Integration Runtime。 發生容錯移轉時，您可以確保 Azure-SSIS IR 仍會使用次要資料庫來運作。

如需 SQL Database 異地複寫和容錯移轉的詳細資訊，請參閱[概觀：主動式異地複寫和自動容錯移轉群組](../sql-database/sql-database-geo-replication-overview.md)。

## <a name="scenario-1---azure-ssis-ir-is-pointing-to-read-write-listener-endpoint"></a>案例 1 - Azure-SSIS IR 指向讀寫接聽程式端點

### <a name="conditions"></a>條件

下列條件成立時，本節便適用：

- Azure-SSIS IR 指向容錯移轉群組的讀寫接聽程式端點。

  AND

- SQL Database 伺服器不是使用虛擬網路服務端點規則來設定的。

### <a name="solution"></a>解決方法

發生容錯移轉時，Azure-SSIS IR 不會察覺。 Azure-SSIS IR 會自動連線到容錯移轉群組的新主要伺服器。

## <a name="scenario-2---azure-ssis-ir-is-pointing-to-primary-server-endpoint"></a>案例 2 - Azure-SSIS IR 指向主要伺服器端點

### <a name="conditions"></a>條件

下列其中一個條件成立時，本節便適用：

- Azure-SSIS IR 指向容錯移轉群組的主要伺服器端點。 發生容錯移轉時，此端點會變更。

  或

- Azure SQL Database 伺服器是使用虛擬網路服務端點規則來設定的。

  或

- 資料庫伺服器是使用虛擬網路所設定的 SQL Database 受控執行個體。

### <a name="solution"></a>解決方法

發生容錯移轉時，您必須執行下列動作：

1. 停止 Azure-SSIS IR。

2. 重新設定 IR，使其指向新的主要端點和新區域中的虛擬網路。

3. 重新啟動 IR。

下列各節會更詳細地說明這些步驟。

### <a name="prerequisites"></a>必要條件

- 請確定您已針對 Azure SQL Database 伺服器啟用災害復原，以免服器在同一時間發生中斷。 如需詳細資訊，請參閱[使用 Azure SQL Database 的商務持續性概觀](../sql-database/sql-database-business-continuity.md)。

- 如果您在目前的區域中使用虛擬網路，那麼您需要在新區域中使用另一個虛擬網路來與 Azure SSIS 整合執行階段連線。 如需詳細資訊，請參閱[將 Azure-SSIS 整合執行階段加入虛擬網路](join-azure-ssis-integration-runtime-virtual-network.md)。

- 如果您使用自訂安裝，可能需要為儲存自訂安裝指令碼和相關檔案的 blob 容器準備另一個 SAS URI，如此一來，發生中斷時仍可繼續存取此 blob 容器。 如需詳細資訊，請參閱[在 Azure-SSIS 整合執行階段上設定自訂安裝](how-to-configure-azure-ssis-ir-custom-setup.md)。

### <a name="steps"></a>步驟

請遵循下列步驟來停止 Azure SSIS 整合執行階段，並將整合執行階段切換到新區域，然後重新啟動該整合執行階段。

1. 停止原始區域中的整合執行階段。

2. 在 PowerShell 中呼叫下列命令來使用新的設定更新整合執行階段。

    ```powershell
    Set-AzureRmDataFactoryV2IntegrationRuntime -Location "new region" `
                    -CatalogServerEndpoint "Azure SQL Database server endpoint" `
                    -CatalogAdminCredential "Azure SQL Database server admin credentials" `
                    -VNetId "new VNet" `
                    -Subnet "new subnet" `
                    -SetupScriptContainerSasUri "new custom setup SAS URI"
    ```

    如需此 PowerShell 命令的詳細資訊，請參閱[在 Azure Data Factory 中建立 Azure-SSIS 整合執行階段](create-azure-ssis-integration-runtime.md)

3. 重新啟動整合執行階段。

## <a name="next-steps"></a>後續步驟

請考量 Azure-SSIS 整合執行階段的下列其他組態選項：

- [設定 Azure-SSIS Integration Runtime 以獲得高效能](configure-azure-ssis-integration-runtime-performance.md)

- [自訂 Azure-SSIS 整合執行階段的安裝](how-to-configure-azure-ssis-ir-custom-setup.md)

- [佈建 Azure-SSIS 整合執行階段的企業版](how-to-configure-azure-ssis-ir-enterprise-edition.md)
