---
title: 佈建 Azure-SSIS 整合執行階段的企業版 | Microsoft Docs
description: 本文說明 Azure-SSIS 整合執行階段企業版的功能及其佈建方式
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/13/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: 73997345895bc54f54db1d66c0c6c24c24153dd2
ms.sourcegitcommit: 301855e018cfa1984198e045872539f04ce0e707
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36267840"
---
# <a name="provision-enterprise-edition-for-the-azure-ssis-integration-runtime"></a>佈建 Azure-SSIS 整合執行階段的企業版

Azure-SSIS 整合執行階段的企業版可讓您使用下列進階功能：
-   異動資料擷取 (CDC) 元件
-   Oracle、Teradata 和 SAP BW 連接器
-   SQL Server Analysis Services (SSAS) 和 Azure Analysis Services (AAS) 連接器和轉換
-   模糊群組和模糊查閱轉換
-   詞彙擷取和詞彙查閱轉換

其中部分功能需要您安裝其他元件來自訂 Azure-SSIS IR。 如需如何安裝其他元件的詳細資訊，請參閱 [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)。

## <a name="enterprise-features"></a>企業功能

| **企業功能** | **說明** |
|---|---|
| CDC 元件 | CDC 來源、控制工作和分隔器轉換會在 Azure-SSIS IR 企業版上預先安裝。 若要連線到 Oracle，您還需要在另一部電腦上安裝 CDC 設計工具和服務。 |
| Oracle 連接器 | Oracle 連線管理員、來源和目的地會在 Azure-SSIS IR 企業版上預先安裝。 您也需要在 Azure-SSIS IR 上安裝 Oracle Call Interface (OCI) 驅動程式，並於必要時設定 Oracle Transport Network Substrate (TNS)。 如需詳細資訊，請參閱 [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)。 |
| Teradata 連接器 | 您需要在 Azure-SSIS IR 企業版上安裝 Teradata 連線管理員、來源和目的地，以及 Teradata Parallel Transporter (TPT) API 和 Teradata ODBC 驅動程式。 如需詳細資訊，請參閱 [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)。 |
| SAP BW 連接器 | SAP BW 連線管理員、來源和目的地會在 Azure-SSIS IR 企業版上預先安裝。 您也需要在 Azure-SSIS IR 上安裝 SAP BW 驅動程式。 這些連接器支援 SAP BW 7.0 或更早版本。 若要連線至更新版本的 SAP BW 或其他 SAP 產品，您可以在 Azure-SSIS IR 上從第三方 ISV 購買並安裝 SAP 連接器。 如需如何安裝其他元件的詳細資訊，請參閱 [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)。 |
| Analysis Services 元件               | 資料採礦模型定型目的地、維度處理目的地和分割區處理目的地以及資料採礦查詢轉換，會在 Azure-SSIS IR 企業版上預先安裝。 這些元件都支援 SQL Server Analysis Services (SSAS)，但是只有分割區處理目的地支援 Azure Analysis Services (AAS)。 若要連線到 SSAS，您也必須[在 SSISDB 中設定 Windows 驗證認證](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth)。 除了這些元件以外，Analysis Services 執行 DDL 工作、Analysis Services 處理工作和資料採礦查詢工作，也會在 Azure-SSIS IR 標準/企業版上預先安裝。 |
| 模糊群組和模糊查閱轉換  | 模糊群組和模糊查閱轉換會在 Azure-SSIS IR 企業版上預先安裝。 這些元件都支援 SQL Server 和 Azure SQL Database 以儲存參考資料。 |
| 詞彙擷取和詞彙查閱轉換 | 詞彙擷取和詞彙查閱轉換會在 Azure-SSIS IR 企業版上預先安裝。 這些元件都支援 SQL Server 和 Azure SQL Database 以儲存參考資料。 |

## <a name="instructions"></a>範例的指示

1.  下載並安裝 [Azure PowerShell (5.4 版或更新版本)](https://github.com/Azure/azure-powershell/releases/tag/v5.5.0-March2018)。

2.  當您使用 PowerShell 佈建或重新設定 Azure-SSIS IR 時，在啟動 Azure-SSIS IR 之前，請使用 **Enterprise** 作為 **Edition** 參數的值來執行 `Set-AzureRmDataFactoryV2IntegrationRuntime`。 以下是範例指令碼：

    ```powershell
    $MyAzureSsisIrEdition = "Enterprise"

    Set-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $MyDataFactoryName
                                               -Name $MyAzureSsisIrName
                                               -ResourceGroupName $MyResourceGroupName
                                               -Edition $MyAzureSsisIrEdition

    Start-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $MyDataFactoryName
                                                 -Name $MyAzureSsisIrName
                                                 -ResourceGroupName $MyResourceGroupName
    ```

## <a name="next-steps"></a>後續步驟

-   [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)

-   [如何開發 Azure-SSIS 整合執行階段的付費或授權自訂元件](how-to-develop-azure-ssis-ir-licensed-components.md)
