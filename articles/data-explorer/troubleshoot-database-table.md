---
title: 無法在 Azure 資料總管中建立或刪除資料庫或資料表
description: 此文章說明在 Azure 資料總管中建立及刪除資料庫與資料表的疑難排解步驟。
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
services: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 6ec81d6154f15d1c49428b50f0e65eed8edcedad
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986503"
---
# <a name="troubleshoot-failure-to-create-or-delete-a-database-or-table-in-azure-data-explorer"></a>疑難排解：無法在 Azure 資料總管中建立或刪除資料庫或資料表

在 Azure 資料總管中，您經常使用資料庫和資料表。 此文章提供適用於可能出現之問題的疑難排解步驟。

## <a name="creating-a-database"></a>建立資料庫

1. 確定您有足夠的權限。 若要建立資料庫，您必須是 Azure 訂用帳戶中「參與者」或「擁有者」角色的成員。 如有必要，請與訂用帳戶系統管理員合作，以便他們可以將您新增到適當的角色。

1. 確定資料庫名稱沒有任何名稱驗證錯誤。 名稱必須是英數字元且最大長度為 260 個字元。

1. 確定資料庫保留期和快取值均位於允許的範圍內。 保留期必須介於 1 到 36500 天 (100 年) 之間。 快取必須介於 1 到保留期的最大值組合之間。

## <a name="deleting-or-renaming-a-database"></a>刪除或重新命名資料庫

確定您有足夠的權限。 若要刪除或重新命名資料庫，您必須是 Azure 訂用帳戶中「參與者」或「擁有者」角色的成員。 如有必要，請與訂用帳戶系統管理員合作，以便他們可以將您新增到適當的角色。

## <a name="creating-a-table"></a>建立資料表

1. 確定您有足夠的權限。 若要建立資料表，您必須是資料庫中「資料庫管理員」或「資料庫使用者」角色的成員，或是 Azure 訂用帳戶的「參與者」或「擁有者」角色。 如有必要，請與訂用帳戶或叢集系統管理員合作，以便他們可以將您新增到適當的角色。

    如需有關權限的詳細資訊，請參閱[管理資料庫權限](manage-database-permissions.md)。

1. 確定具有相同名稱的資料表尚未存在。 如果存在，則您可以：建立具有不同名稱的資料表；將現有的資料表重新命名 (需要「資料表管理員」角色)；或卸除現有資料表 (需要「資料庫管理員」角色)。 使用下列命令。

    ```Kusto
    .drop table <TableName>

   .rename table <OldTableName> to <NewTableName>
    ```

## <a name="deleting-or-renaming-a-table"></a>刪除或重新命名資料表

確定您有足夠的權限。 若要刪除或重新命名資料表，您必須是資料庫中「資料庫管理員」或「資料表管理員」角色的成員。 如有必要，請與訂用帳戶或叢集系統管理員合作，以便他們可以將您新增到適當的角色。

如需有關權限的詳細資訊，請參閱[管理資料庫權限](manage-database-permissions.md)。

## <a name="general-guidance"></a>一般指導方針

1. 查看 [Azure 服務健康情況儀表板](https://azure.microsoft.com/status/>)。 在您要嘗試使用資料庫或資料表的區域中尋找 Azure 資料總管的狀態。

    如果狀態不是**良好** (綠色核取記號)，請在狀態改善之後再試一次。

1. 若您仍然需要協助來解決問題，請在 [Azure 入口網站](https://portal.azure.com)中開啟支援要求。

## <a name="next-steps"></a>後續步驟

[檢查叢集健康情況](check-cluster-health.md)