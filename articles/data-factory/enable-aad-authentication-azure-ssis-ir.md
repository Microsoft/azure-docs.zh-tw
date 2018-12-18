---
title: 啟用適用於 Azure-SSIS 整合執行階段的 Azure Active Directory 驗證 | Microsoft Docs
description: 本文說明如何設定 Azure-SSIS 整合執行階段，以啟用使用 Azure Active Directory 驗證的連線。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 06/21/2018
ms.author: douglasl
ms.openlocfilehash: aa06110a6f6fe668388c6aecd98c1ddeeae37edd
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576624"
---
# <a name="enable-azure-active-directory-authentication-for-the-azure-ssis-integration-runtime"></a>啟用適用於 Azure-SSIS 整合執行階段的 Azure Active Directory 驗證

本文說明如何使用 Azure Data Factory 服務識別建立 Azure-SSIS IR。 針對 Azure SSIS 整合執行階段使用受控服務識別 (MSI) 的 Azure Active Directory (Azure AD) 驗證，可讓您使用 Data Factory MSI 來建立 Azure-SSIS 整合執行階段，而不使用 SQL 驗證。

如需 Data Factory MSI 的詳細資訊，請參閱 [Azure Data Factory 服務識別](https://docs.microsoft.com/azure/data-factory/data-factory-service-identity)。

> [!NOTE]
> 如果您已建立使用 SQL 驗證的 Azure SSIS 整合執行階段，此時您無法將該 IR 重新設定為透過 PowerShell 使用 Azure AD 驗證。

## <a name="create-a-group-in-azure-ad-and-make-the-data-factory-msi-a-member-of-the-group"></a>在 Azure AD 中建立群組，並讓 Data Factory MSI 成為此群組的成員

您可以使用現有的 Azure AD 群組，或使用 Azure AD PowerShell 建立一個新群組。

1.  安裝 [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2) 模組。

2.  使用 `Connect-AzureAD` 登入，執行下列命令以建立群組，並將其儲存在變數中：

    ```powershell
    $Group = New-AzureADGroup -DisplayName "SSISIrGroup" `
                              -MailEnabled $false `
                              -SecurityEnabled $true `
                              -MailNickName "NotSet"
    ```

    輸出如下列範例所示，也會檢查變數的值：

    ```powershell
    $Group

    ObjectId DisplayName Description
    -------- ----------- -----------
    6de75f3c-8b2f-4bf4-b9f8-78cc60a18050 SSISIrGroup
    ```

3.  將 Data Factory MSI 新增至群組。 您可以依照 [Azure Data Factory 服務識別](https://docs.microsoft.com/azure/data-factory/data-factory-service-identity)來取得主體服務識別的識別碼 (例如 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc，但是不要將服務識別應用程式識別碼用於此用途)。

    ```powershell
    Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc
    ```

    您也可以之後再檢查群組成員資格。

    ```powershell
    Get-AzureAdGroupMember -ObjectId $Group.ObjectId
    ```

## <a name="enable-azure-ad-on-azure-sql-database"></a>在 Azure SQL Database 上啟用 Azure AD

Azure SQL Database 支援由 Azure AD 使用者建立資料庫。 因此，您可以將 Azure AD 使用者設定為 Active Directory 管理員，然後使用 Azure AD 使用者登入 SSMS。 接著，您可以建立 Azure AD 群組所包含的使用者，讓 IR 在能夠伺服器上建立 SQL Server Integration Services (SSIS) 目錄。

### <a name="enable-azure-ad-authentication-for-the-azure-sql-database"></a>啟用 Azure SQL Database 的 Azure AD 驗證

您可以使用下列步驟[為 SQL Database 設定 Azure AD 驗證](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)：

1.  在 Azure 入口網站中，選取左側導覽中的 [所有服務] -> [SQL 伺服器]。

2.  選取要啟用 Azure AD 驗證的 SQL Database。

3.  在刀鋒視窗的 [設定] 區段中，選取 [Active Directory 管理員]。

4.  在命令列中選取 [設定管理員]。

5.  選取要設為伺服器管理員的 Azure AD 使用者帳戶，然後選取 [選取]。

6.  在命令列中選取 [儲存]。

### <a name="create-a-contained-user-in-the-database-that-represents-the-azure-ad-group"></a>在代表 Azure AD 群組的資料庫中建立包含的使用者

在此後續步驟中，您需要 [Microsoft SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。

1.  啟動 SQL Server Management Studio。

2.  在 [連線到伺服器] 對話方塊中，在 [伺服器名稱] 欄位中輸入您的 SQL 伺服器名稱。

3.  在 [驗證] 欄位中，選取 [具 MFA 支援的 Active Directory - 通用]。 (您也可以使用其他兩種 Active Directory 驗證類型。 請參閱[設定及管理用於 SQL Database 和受控執行個體的 Azure Active Directory 驗證](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)。)

4.  在 [使用者名稱] 欄位中，輸入您設為伺服器系統管理員的 Azure AD 帳戶名稱，例如 testuser@xxxonline.com。

5.  選取 [連線]。 完成登入程序。

6.  在 [物件總管] 中，展開 [資料庫] -> [系統資料庫] 資料夾。

7.  以滑鼠右鍵按一下 **master** 資料庫，並選取 [新增查詢]。

8.  在查詢視窗中，輸入下列這一行，然後選取工具列中的 [執行]︰

    ```sql
    CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
    ```

    命令應該會順利完成，為群組建立包含的使用者。

9.  清除查詢視窗，輸入下列這一行，然後選取工具列中的 [執行]︰

    ```sql
    ALTER ROLE dbmanager ADD MEMBER [SSISIrGroup]
    ```

    命令應該會順利完成，將建立資料庫的能力授與內含的使用者。

## <a name="enable-azure-ad-on-azure-sql-database-managed-instance"></a>為 Azure SQL Database 受控執行個體啟用 Azure AD

Azure SQL Database 受控執行個體不支援由 AD 管理員以外的任何 Azure AD 使用者建立資料庫。因此，您必須將 Azure AD 群組設定為 Active Directory 管理員。您不需要建立內含的使用者。

您可以使用下列步驟，[為 SQL Database 受控執行個體伺服器設定 Azure AD 驗證](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)：

7.  在 Azure 入口網站中，選取左側導覽中的 [所有服務] -> [SQL 伺服器]。

8.  選取要啟用 Azure AD 驗證的 SQL 伺服器。

9.  在刀鋒視窗的 [設定] 區段中，選取 [Active Directory 管理員]。

10. 在命令列中選取 [設定管理員]。

11. 搜尋並選取 Azure AD 群組 (例如 SSISIrGroup)，然後選取 [選取]。

12. 在命令列中選取 [儲存]。

## <a name="provision-the-azure-ssis-ir-in-the-portal"></a>在入口網站中佈建 Azure SSIS IR

當您使用 Azure 入口網站佈建 Azure-SSIS IR 時，請在 [SQL 設定] 頁面上勾選 [對您的 ADF MSI 使用 AAD 驗證] 選項。 (下列螢幕擷取畫面顯示 Azure SQL Database 的 IR 設定。 如果是受控執行個體的 IR，[類別目錄資料庫服務層] 會無法使用，其他設定則都相同。)

如需關於如何建立 Azure-SSIS 整合執行階段的詳細資訊，請參閱[在 Azure Data Factory 中建立 Azure-SSIS 整合執行階段](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime)。

![Azure-SSIS 整合執行階段的設定](media/enable-aad-authentication-azure-ssis-ir/enable-aad-authentication.png)

## <a name="provision-the-azure-ssis-ir-with-powershell"></a>使用 PowerShell 佈建 Azure-SSIS IR

若要使用 PowerShell 佈建 Azure-SSIS IR，請執行下列作業：

1.  安裝 [Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v5.5.0-March2018) 模組。

2.  在您的指令碼中，請勿設定 *CatalogAdminCredential* 參數。 例如︰

    ```powershell
    Set-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                               -DataFactoryName $DataFactoryName `
                                               -Name $AzureSSISName `
                                               -Type Managed `
                                               -CatalogServerEndpoint $SSISDBServerEndpoint `
                                               -CatalogPricingTier $SSISDBPricingTier `
                                               -Description $AzureSSISDescription `
                                               -Edition $AzureSSISEdition `
                                               -Location $AzureSSISLocation `
                                               -NodeSize $AzureSSISNodeSize `
                                               -NodeCount $AzureSSISNodeNumber `
                                               -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                               -SetupScriptContainerSasUri $SetupScriptContainerSasUri

    Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                                 -DataFactoryName $DataFactoryName `
                                                 -Name $AzureSSISName
   ```
