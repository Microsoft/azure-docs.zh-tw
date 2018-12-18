---
title: 管理 Azure 資料總管資料庫權限
description: 本文會說明 Azure 資料總管中適用於資料庫和資料表的角色型存取控制。
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
services: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: fbbc5a199116e46aac0874f3dc6d6d9aa18c60cd
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954011"
---
# <a name="manage-azure-data-explorer-database-permissions"></a>管理 Azure 資料總管資料庫權限

Azure 資料總管可讓您使用角色型存取控制模型，來控制對資料庫和資料表的存取權。 在此模型中，*主體* (使用者、群組和應用程式) 會對應至*角色*。 主體可以根據其指派的角色存取資源。

本文說明可用的角色，以及如何將主體指派給這些角色。

## <a name="roles-and-permissions"></a>角色和權限

Azure 資料總管會有下列角色：

|角色                       |權限                                                                        |
|---------------------------|-----------------------------------------------------------------------------------|
|資料庫管理             |可以在特定資料庫的範圍內執行任何動作。|
|資料庫使用者              |可以讀取資料庫中的所有資料和中繼資料。 此外，他們可以在資料庫中建立資料表 (成為該資料表的資料表管理員) 和函式。|
|資料庫檢視器            |可以讀取資料庫中的所有資料和中繼資料。|
|資料庫擷取器          |可以將資料擷取到資料庫中所有現有的資料表，但不能查詢資料。|
|資料庫不受限制檢視器|可以查詢啟用了 **RestrictedViewAccess** 原則的資料表。 無法查詢其他資料表。|
|資料庫監視器           |可以在資料庫和其子實體的內容中執行 '.show ...' 命令。|
|資料表系統管理員                |可以在特定資料表的範圍內執行任何操作。 |
|資料表擷取器             |可以在特定資料表的範圍內擷取資料，但不能查詢資料。|

## <a name="manage-permissions-in-the-azure-portal"></a>在 Azure 入口網站中管理權限

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 瀏覽至您的 Azure 資料總管叢集。

1. 在 [概觀] 區段中，選取您要管理權限的資料庫。

    ![選取資料庫](media/manage-database-permissions/select-database.png)

1. 選取 [權限]，然後 [新增]。

    ![資料庫權限](media/manage-database-permissions/database-permissions.png)

1. 在 [新增資料庫權限] 下，選擇要指派給主體的角色，然後選擇 [選取主體]。

    ![新增資料庫權限](media/manage-database-permissions/add-permission.png)

1. 查詢主體、選取它，然後 [選取]。

    ![在 Azure 入口網站中管理權限](media/manage-database-permissions/new-principals.png)

1. 選取 [ **儲存**]。

    ![在 Azure 入口網站中管理權限](media/manage-database-permissions/save-permission.png)

## <a name="manage-permissions-with-management-commands"></a>使用管理命令管理權限

1. 登入 [https://dataexplorer.azure.com](https://dataexplorer.azure.com)，然後新增您的叢集 (如果尚無法使用)。

1. 在左窗格中，選取適當的資料庫。

1. 使用 `.add` 命令將主體指派給角色：`.add database databasename rolename ('aaduser | aadgroup=user@domain.com')`。 若要將使用者加入到資料庫使用者角色，請執行下列命令，取代您的資料庫名稱和使用者。

    ```Kusto
    .add database <TestDatabase> users ('aaduser=<user@contoso.com>')
    ```

    該命令的輸出顯示現有使用者的清單，以及他們在資料庫中受指派的角色。

## <a name="next-steps"></a>後續步驟

[撰寫查詢](write-queries.md)