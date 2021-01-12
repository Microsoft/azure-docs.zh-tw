---
title: 專用 SQL 集區的持續整合和部署
description: Azure Synapse Analytics 中的專用 SQL 集區的企業級資料庫 DevOps 體驗，並使用 Azure Pipelines 進行持續整合和部署的內建支援。
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: cb80f2d21246a75d41fc5753e72995d409d5c6b4
ms.sourcegitcommit: aacbf77e4e40266e497b6073679642d97d110cda
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98119181"
---
# <a name="continuous-integration-and-deployment-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Azure Synapse Analytics 中專用 SQL 集區的持續整合和部署

此簡易教學課程會概述如何將您的 SQL Server Data tools (SSDT) 資料庫專案與 Azure DevOps 整合，並運用 Azure Pipelines 來設定持續整合及部署。 本教學課程是為數據倉儲建立持續整合和部署管線的第二個步驟。

## <a name="before-you-begin"></a>開始之前

- 請檢閱[原始檔控制整合教學課程](sql-data-warehouse-source-control-integration.md)

- 設定及連線至 Azure DevOps

## <a name="continuous-integration-with-visual-studio-build"></a>與 Visual Studio 組建持續整合

1. 瀏覽至 Azure Pipelines 並建立新的建置管線。

      ![新增管線](./media/sql-data-warehouse-continuous-integration-and-deployment/1-new-build-pipeline.png "新增管線")

2. 選取您的原始程式碼存放庫 (Azure Repos Git)，然後選取 .NET 桌面應用程式範本。

      ![管線設定](./media/sql-data-warehouse-continuous-integration-and-deployment/2-pipeline-setup.png "管線設定")

3. 編輯您的 YAML 檔案，以使用適當的代理程式集區。 YAML 檔案應該會看起來像這樣：

      ![YAML](./media/sql-data-warehouse-continuous-integration-and-deployment/3-yaml-file.png "YAML")

此時，您會有一個簡單的環境，您的原始檔控制存放庫主要分支的任何簽入都應該自動觸發成功的資料庫專案 Visual Studio 組建。 藉由在您的本機資料庫專案中進行變更，並將變更簽入您的主要分支，以驗證自動化的運作端。

## <a name="continuous-deployment-with-the-azure-synapse-analytics-or-database-deployment-task"></a>使用 Azure Synapse Analytics (或資料庫) 部署工作進行持續部署

1. 使用 [Azure SQL Database 部署工作](/azure/devops/pipelines/targets/azure-sqldb)加入新的工作，並填寫必要欄位，即可連線到您的目標資料倉儲。 當此工作執行時，從先前建置程序產生的 DACPAC 就會部署到目標資料倉儲。 您也可以使用 [Azure Synapse Analytics 部署](https://marketplace.visualstudio.com/items?itemName=ms-sql-dw.SQLDWDeployment)工作。

      ![部署工作](./media/sql-data-warehouse-continuous-integration-and-deployment/4-deployment-task.png "部署工作")

2. 如果您使用自我裝載的代理程式，請確定您已將環境變數設定為針對 Azure Synapse Analytics 使用正確的 SqlPackage.exe。 路徑應該會看起來像這樣：

      ![環境變數](./media/sql-data-warehouse-continuous-integration-and-deployment/5-environment-variable-preview.png "環境變數")

   C:\Program Files (x86)\Microsoft Visual Studio\2019\Preview\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\150  

   執行及驗證您的管線。 您可以在本機進行變更，並將變更簽入原始檔控制，以產生自動建置和部署。

## <a name="next-steps"></a>後續步驟

- 探索 [專用的 sql 集區 (先前的 SQL DW) 架構](massively-parallel-processing-mpp-architecture.md)
- 快速 [建立專用的 sql 集區 (先前的 SQL DW) ](create-data-warehouse-portal.md)
- [載入範例資料](./load-data-from-azure-blob-storage-using-copy.md)
- 探索[影片](sql-data-warehouse-videos.md)