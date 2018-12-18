---
title: 使用 U-SQL 資料庫專案開發 Azure Data Lake 的 U-SQL 資料庫
description: 了解如何使用 Azure Data Lake Tools for Visual Studio 開發 U-SQL 資料庫。
services: data-lake-analytics
author: yanancai
ms.author: yanacai
ms.reviewer: jasonwhowell
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: conceptual
ms.workload: big-data
ms.date: 07/03/2018
ms.openlocfilehash: 98dc33cc961c5c90fc3f6737a1cc470d4a753fd0
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048752"
---
# <a name="use-a-u-sql-database-project-to-develop-a-u-sql-database-for-azure-data-lake"></a>使用 U-SQL 資料庫專案開發 Azure Data Lake 的 U-SQL 資料庫

U-SQL 資料庫會提供結構化的檢視，以便在資料表中檢視非結構化的資料和受控的結構化資料。 也會提供一般的中繼資料目錄系統來組織結構化資料和自訂程式碼。 資料庫是將這些相關物件編排在一起的概念。

深入了解 [U-SQL 資料庫和資料定義語言 (DDL)](https://msdn.microsoft.com/azure/data-lake-analytics/u-sql/data-definition-language-ddl-statements-u-sql)。 

U-SQL 資料庫專案是 Visual Studio 中的專案類型，能協助開發人員快速而輕鬆地開發、管理及部署 U-SQL 資料庫。

## <a name="create-a-u-sql-database-project"></a>建立 U-SQL 資料庫專案

Azure Data Lake Tools for Visual Studio 在 2.3.3000.0 之後的版本中，新增了稱為 U-SQL 資料庫專案的專案範本。 若要建立 U-SQL 專案，請選取 [檔案] > [新增] > [專案]。 您可以在 [Azure Data Lake] > [U-SQL 節點] 下找到 U-SQL 資料庫專案。

![Data Lake Tools for Visual Studio--建立 U-SQL 資料庫專案](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-create-usql-database-project-creation.png) 

## <a name="develop-u-sql-database-objects-by-using-a-database-project"></a>使用資料庫專案開發 U-SQL 資料庫物件

以滑鼠右鍵按一下 U-SQL 資料庫專案。 選取 [新增] > [新增項目]。 您可以在 [新增項目] 精靈中找到所有新的可支援物件類型。 

至於非組件物件 (例如資料表值函式)，新的 U-SQL 指令碼會在新項目新增後建立。 您可以開始在編輯器中開發該物件的 DDL 陳述式。

至於組件物件，此工具提供了方便使用的 UI 編輯器，可協助您註冊組件及部署 DLL 檔案和其他檔案。 以下步驟會說明如何將組件物件定義新增到 U-SQL 資料庫專案：

1.  新增 C# 專案的參考，其中包含 U-SQL 資料庫專案的 UDO/UDAG/UDF。

    ![Data Lake Tools for Visual Studio--新增 U-SQL 資料庫專案參考](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-project-reference.png) 

    ![Data Lake Tools for Visual Studio--新增 U-SQL 資料庫專案參考](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-project-reference-wizard.png)

2.  在組件設計檢視中，從 [從參考建立組件] 下拉式功能表中選擇參考組件。

    ![Data Lake Tools for Visual Studio--從參考建立組件](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-create-assembly-from-reference.png)

3.  新增 [受控相依性] 和 [其他檔案] (若有的話)。 在新增其他檔案時，工具會使用相對路徑來確保能在本機電腦和稍後的組建電腦上找到組件。 

@_DeployTempDirectory 是預先定義的變數，能將工具指向建置輸出資料夾。 在建置輸出資料夾中，每個組件都具有以組件名稱命名的子資料夾。 所有 Dll 和其他檔案都在該子資料夾中。 
 
## <a name="build-a-u-sql-database-project"></a>建置 U-SQL 資料庫專案

U-SQL 資料庫專案的建置輸出，是一個名稱加上 `.usqldbpack` 尾碼的 U-SQL 資料庫部署套件。 `.usqldbpack` 套件是一個 ZIP 檔案，其中的 **DDL** 資料夾含有單一 U-SQL 指令碼中的所有 DDL 陳述式，而 **Temp** 資料夾則含有組件的所有 DLL 和額外檔案。

深入了解[如何使用 MSBuild 命令列和 Visual Studio Team Service 建置工作來建置 U-SQL 資料庫專案](data-lake-analytics-cicd-overview.md)。

## <a name="deploy-a-u-sql-database"></a>部署 U-SQL 資料庫

您可以使用 Visual Studio 或部署 SDK，將 .usqldbpack 套件部署到本機帳戶或 Azure Data Lake Analytics 帳戶。 

### <a name="deploy-a-u-sql-database-in-visual-studio"></a>在 Visual Studio 中部署 U-SQL 資料庫

您可以在 Visual Studio 中透過 U-SQL 資料庫專案或 .usqldbpack 套件部署 U-SQL 資料庫。

#### <a name="deploy-through-a-u-sql-database-project"></a>透過 U-SQL 資料庫專案部署

1.  以滑鼠右鍵按一下 U-SQL 資料庫專案，然後選取 [部署]。
2.  在 [部署 U-SQL 資料庫精靈] 中，選擇要部署資料庫的目的地 [ADLA 帳戶]。 本機帳戶和 ADLA 帳戶兩者均可支援。
3.  [資料庫來源] 會自動填入，並指向專案建置輸出資料夾中的 .usqldbpack 套件。
4.  在 [資料庫名稱] 中輸入名稱，以建立資料庫。 如果目標 Azure Data Lake Analytics 帳戶已有相同名稱的資料庫，在資料庫專案中定義的所有物件均會建立，但不會重新建立資料庫。
5.  若要部署 U-SQL 資料庫，請按一下 [提交]。 所有資源 (組件和其他檔案) 隨即會上傳，而且包含所有 DDL 陳述式的 U-SQL 作業也會提交。

    ![Data Lake Tools for Visual Studio--部署 U-SQL 資料庫專案](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-project.png)

    ![Data Lake Tools for Visual Studio--部署 U-SQL 資料庫專案精靈](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-project-wizard.png)

#### <a name="deploy-through-a-u-sql-database-deployment-package"></a>透過 U-SQL 資料庫部署套件部署

1.  開啟 [伺服器總管]。 然後展開要部署資料庫的目的地 **Azure Data Lake Analytics 帳戶**。
2.  以滑鼠右鍵按一下 **U-SQL 資料庫**，然後選擇 [部署資料庫]。
3.  將 [資料庫來源] 設定為 U-SQL 資料庫部署套件 (.usqldbpack file) 路徑。
4.  輸入**資料庫名稱**，以建立資料庫。 如果目標 Azure Data Lake Analytics 帳戶已有相同名稱的資料庫，在資料庫專案中定義的所有物件均會建立，但不會重新建立資料庫。

    ![Data Lake Tools for Visual Studio--部署 U-SQL 資料庫套件](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-package.png)

    ![Data Lake Tools for Visual Studio--部署 U-SQL 資料庫套件精靈](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-package-wizard.png)
  
### <a name="deploy-u-sql-database-by-using-the-sdk"></a>使用 SDK 部署 U-SQL 資料庫

`PackageDeploymentTool.exe` 提供程式設計和命令列介面，協助您部署 U-SQL 資料庫。 SDK 包含在 [U-SQL SDK Nuget 套件](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/)中，位於 `build/runtime/PackageDeploymentTool.exe`。

[深入了解 SDK，以及如何為 U-SQL 資料庫部署設定 CI/CD 管線](data-lake-analytics-cicd-overview.md#deploy-u-sql-database-through-visual-studio-team-service)。

## <a name="reference-a-u-sql-database-project"></a>參考 U-SQL 資料庫專案

U-SQL 專案可以參考 U-SQL 資料庫專案。 參考會影響兩種工作負載：

- 專案建置：在編譯 U-SQL 指令碼之前設定參考資料庫環境。 
- 以 (Local-project) 帳戶進行本機執行：參考資料庫環境會在 U-SQL 指令碼執行之前部署到 (Local-project) 帳戶。 [在這裡可深入了解本機執行及 (Local-machine) 和 (Local-project) 帳戶之間的差異](data-lake-analytics-data-lake-tools-local-run.md)。

### <a name="how-to-add-a-u-sql-database-reference"></a>如何新增 U-SQL 資料庫參考

1. 在 [方案總管] 中以滑鼠右鍵按一下 U-SQL 專案，然後選擇 [新增 U-SQL 資料庫參考...]。

    ![Data Lake Tools for Visual Studio -- 新增資料庫專案參考](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-database-project-reference.png)

2. 從目前的解決方案中的 U-SQL 資料庫專案或 U-SQL 資料庫套件檔案中設定資料庫參考。
3. 提供資料庫的名稱。

    ![Data Lake Tools for Visual Studio 新增資料庫專案參考精靈](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-database-project-reference-wizard.png)

## <a name="next-steps"></a>後續步驟

- [如何設定 Azure Data Lake Analytics 的 CI/CD 管線](data-lake-analytics-cicd-overview.md)
- [如何測試 Azure Data Lake Analytics 程式碼](data-lake-analytics-cicd-test.md)
- [在本機電腦上執行 U-SQL 指令碼](data-lake-analytics-data-lake-tools-local-run.md)
