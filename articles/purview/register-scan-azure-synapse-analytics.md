---
title: 如何掃描 Azure Synapse Analytics
description: 本操作指南說明如何掃描 Azure Synapse Analytics 的詳細資料。
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 10/22/2020
ms.openlocfilehash: 3ba43b83166b5548dee4ea4e52c7411db48d23f5
ms.sourcegitcommit: ca215fa220b924f19f56513fc810c8c728dff420
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2021
ms.locfileid: "98567271"
---
# <a name="register-and-scan-azure-synapse-analytics"></a>註冊並掃描 Azure Synapse Analytics

本文討論如何在範疇中註冊和掃描 Azure Synapse Analytics 先前 (SQL DW) 的實例。

## <a name="supported-capabilities"></a>支援的功能

Azure Synapse Analytics (先前的 SQL DW) 支援完整和增量掃描，以捕捉中繼資料和架構。 掃描也會根據系統和自訂分類規則自動分類資料。

### <a name="known-limitations"></a>已知的限制

Azure 範疇不支援在 Azure Synapse Analytics 中掃描[視圖](/sql/relational-databases/views/views?view=azure-sqldw-latest&preserve-view=true)

## <a name="prerequisites"></a>Prerequisites

- 註冊資料來源之前，請先建立 Azure 範疇帳戶。 如需有關建立範疇帳戶的詳細資訊，請參閱 [快速入門：建立 Azure 範疇帳戶](create-catalog-portal.md)。
- 您必須是 Azure 範疇資料來源管理員
- 範疇帳戶與 Azure Synapse Analytics 之間的網路存取。
 
## <a name="setting-up-authentication-for-a-scan"></a>設定掃描驗證

有三種方式可以設定 Azure Synapse Analytics 的驗證：

- 受控識別
- SQL 驗證
- 服務主體

    > [!Note]
    > 只有 master 資料庫中的伺服器層級主體登入 (由佈建程序所建立) 或 `loginmanager` 資料庫角色成員，才能建立新登入。 授與權限之後大約需要 15 分鐘，Purview 帳戶才會有適當的權限可掃描資源。

### <a name="managed-identity-recommended"></a> (建議的受控識別)  
   
您的範疇帳戶有自己的受控識別，基本上是您在建立時的範疇名稱。 您必須遵循 [使用 Azure AD 應用程式建立 Azure AD 使用者](/azure/azure-sql/database/authentication-aad-service-principal-tutorial)的必要條件和教學課程，在 Azure Synapse Analytics (先前為 SQL DW) 的範疇中建立 Azure AD 使用者。

建立使用者並授與權限的範例 SQL 語法：

```sql
CREATE USER [PurviewManagedIdentity] FROM EXTERNAL PROVIDER
GO

EXEC sp_addrolemember 'db_owner', [PurviewManagedIdentity]
GO
```

驗證必須具有取得資料庫、架構和資料表之中繼資料的許可權。 此外也必須能夠查詢要取樣以進行分類的資料表。 建議將許可權指派給身分 `db_owner` 識別。

### <a name="service-principal"></a>服務主體

若要使用服務主體驗證進行掃描，您可以使用現有的驗證，或建立一個新的。 

> [!Note]
> 如果需要建立新的服務主體，請依照下列步驟操作：
> 1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
> 1. 從左側功能表中，選取 [Azure Active Directory]。
> 1. 選取 **應用程式註冊**。
> 1. 選取 [+新增應用程式註冊]。
> 1. 輸入 **應用程式** 的名稱 (服務主體名稱)。
> 1. 選取 [僅此組織目錄中的帳戶]。
> 1. 在 [重新導向 URI] 中選取 [Web]，並輸入您想要的任何 URL；不一定要實際或工作。
> 1. 然後，選取 [註冊]。

需要取得服務主體的應用程式識別碼和密碼：

1. 在 [Azure 入口網站](https://portal.azure.com)中瀏覽至您的服務主體
1. 從 [概觀] 複製 [應用程式 (用戶端) 識別碼] 的值，並從 [憑證和秘密] 複製 [用戶端密碼]。
1. 瀏覽至您的金鑰保存庫
1. 選取 [設定] > [秘密]
1. 選取 [+ 產生/匯入]，然後輸入您選擇的 [名稱]，以及與服務主體的 [用戶端密碼] 相同的 [值]
1. 選取 [建立] 以完成作業
1. 如果您的金鑰保存庫尚未連線至 Purview，您將需要[建立新的金鑰保存庫連線](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)
1. 最後，使用服務主體[建立新的認證](manage-credentials.md#create-a-new-credential)，以設定您的掃描 

#### <a name="granting-the-service-principal-access-to-your-azure-synapse-analytics-formerly-sql-dw"></a>將 Azure Synapse Analytics 的服務主體存取權授與 (先前為 SQL DW) 

此外，您也必須遵循 [使用 Azure AD 應用程式建立 Azure AD 使用者](https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-service-principal-tutorial)的必要條件和教學課程，在 Azure Synapse Analytics 中建立 Azure AD 使用者。 建立使用者並授與權限的範例 SQL 語法：

```sql
CREATE USER [ServicePrincipalName] FROM EXTERNAL PROVIDER
GO

EXEC sp_addrolemember 'db_owner', [ServicePrincipalName]
GO
```

> [!Note]
> Purview 將需要 **應用程式 (用戶端) 識別碼** 和 **用戶端密碼** 才能進行掃描。

### <a name="sql-authentication"></a>SQL 驗證

您可以依照 [建立登](/sql/t-sql/statements/create-login-transact-sql?view=azure-sqldw-latest&preserve-view=true#examples-1) 入中的指示，建立 Azure Synapse Analytics (先前的 SQL DW) 的登入（如果您還沒有的話）。

當選取的驗證方法為 **SQL 驗證** 時，您必須取得密碼並儲存在金鑰保存庫中：

1. 取得 SQL 登入的密碼
1. 瀏覽至您的金鑰保存庫
1. 選取 [設定] > [秘密]
1. 選取 [ **+ 產生/匯入**]，並輸入 **名稱** 和 **值** 做為 SQL 登入的 *密碼*
1. 選取 [建立] 以完成作業
1. 如果您的金鑰保存庫尚未連線至 Purview，您將需要[建立新的金鑰保存庫連線](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)
1. 最後，使用金鑰 [建立新的認證](manage-credentials.md#create-a-new-credential) 來設定您的掃描

## <a name="register-an-azure-synapse-analytics-instance-formerly-sql-dw"></a> (先前的 SQL DW 註冊 Azure Synapse Analytics 實例) 

若要在您的資料目錄中註冊新的 Azure Synapse Analytics 伺服器，請執行下列動作：

1. 瀏覽至您的 Purview 帳戶
1. 在左側導覽列中選取 [來源]
1. 選取 [註冊]
1. 在 [登錄 **來源**] 上，選取 **先前的 SQL DW Azure Synapse Analytics ()**
1. 選取 [繼續]

在 [ **註冊來源 (] Azure Synapse Analytics)** 畫面上，執行下列動作：

1. 輸入會在目錄中列出的資料來源 **名稱**。
1. 選擇您要指向所需儲存體帳戶的方式：
   1. 選取 [ **從 azure 訂用** 帳戶]，從 [ **azure 訂** 用帳戶] 下拉式方塊中選取適當的訂用帳戶，並從 [ **伺服器名稱** ] 下拉式方塊中選取適當的伺服器。
   1. 或者，您可以選取 [手動輸入]，並輸入 **伺服器名稱**。
1. **完成** 以註冊資料來源。

:::image type="content" source="media/register-scan-azure-synapse-analytics/register-sources.png" alt-text="註冊來源選項" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>後續步驟

- [瀏覽 Azure Purview 資料目錄](how-to-browse-catalog.md)
- [搜尋 Azure Purview 資料目錄](how-to-search-catalog.md)

