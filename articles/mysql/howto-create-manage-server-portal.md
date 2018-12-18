---
title: 使用 Azure 入口網站建立和管理適用於 MySQL 的 Azure 資料庫伺服器
description: 本文說明如何使用 Azure 入口網站，快速建立新的「適用於 MySQL 的 Azure 資料庫」伺服器和管理伺服器。
services: mysql
author: ajlam
ms.author: andrela
editor: jasonwhowell
manager: kfile
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 065eb708a1d80b0eac618bd9039a859db6ef1340
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35265579"
---
# <a name="create-and-manage-azure-database-for-mysql-server-using-azure-portal"></a>使用 Azure 入口網站建立和管理適用於 MySQL 的 Azure 資料庫伺服器
本主題說明如何快速建立新的適用於 MySQL 的 Azure 資料庫伺服器。 它也包含有關如何使用 Azure 入口網站來管理伺服器的資訊。 伺服器管理包括檢視伺服器詳細資料和資料庫、重設密碼、調整資源及刪除伺服器。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站
登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="create-an-azure-database-for-mysql-server"></a>建立適用於 MySQL 的 Azure 資料庫伺服器
請遵循下列步驟建立名為 "mydemoserver" 的「適用於 MySQL 的 Azure 資料庫」伺服器。

1. 按一下位於 Azure 入口網站左上角的 [建立資源] 按鈕。

2. 在 [新增] 頁面上，選取 [資料庫]，然後在 [資料庫] 頁面上選取 [適用於 MySQL 的 Azure 資料庫]。

    > 建立的「適用於 MySQL 的 Azure 資料庫」伺服器會有一組已定義的[計算和儲存體](./concepts-pricing-tiers.md)資源。 建立的資料庫位於 Azure 資源群組和「適用於 MySQL 的 Azure 資料庫」伺服器中。

   ![建立新的伺服器](./media/howto-create-manage-server-portal/create-new-server.png)

3. 在「適用於 MySQL 的 Azure 資料庫」表單中填寫下列資訊︰

    | **表單欄位** | **欄位描述** |
    |----------------|-----------------------|
    | *伺服器名稱* | mydemoserver (伺服器名稱是全域唯一的) |
    | *訂用帳戶* | mysubscription (從下拉式功能表中選取) |
    | *資源群組* | myresourcegroup (建立新的資源群組，或使用現有的資源群組) |
    | *選取來源* | 空白 (建立空白的 MySQL 伺服器) |
    | *伺服器管理員登入* | myadmin (設定管理帳戶名稱) |
    | *密碼* | 設定管理帳戶密碼 |
    | *確認密碼* | 確認管理帳戶密碼 |
    | *位置* | 東南亞 (選取 [北歐] 或 [美國西部]) |
    | *版本* | 5.7 (選擇適用於 MySQL 的 Azure 資料庫伺服器版本) |

4. 按一下 [定價層] 指定新資料庫的服務層和效能等級。 選取 [一般用途] 索引標籤。Gen 4、2 個虛擬核心、5 GB 和 7 天是**計算世代**、**虛擬核心**、**儲存體**和**備份保留期限**。 您可以讓這些滑桿保留原狀。 若要啟用異地備援儲存體中的伺服器備份，請從 [備份備援選項] 中選取 [異地備援]。

   ![建立伺服器定價層](./media/howto-create-manage-server-portal/create-server-pricing-tier.png)

5. 按一下 [建立] 以佈建伺服器。 佈建需要幾分鐘的時間。

    > 選擇 [釘選到儀表板] 選項以輕鬆追蹤部署。

## <a name="update-an-azure-database-for-mysql-server"></a>更新適用於 MySQL 的 Azure 資料庫伺服器
佈建新的伺服器之後，使用者會有數個設定現有伺服器的選項，包括重設系統管理員密碼，以及透過變更 vCore 或儲存體來相應增加或相應減少伺服器。

### <a name="change-the-administrator-user-password"></a>變更系統管理員使用者密碼
1. 從伺服器 [概觀]，按一下 [重設密碼] 來顯示密碼重設視窗。

   ![概觀](./media/howto-create-manage-server-portal/overview.png)

2. 在視窗中輸入新密碼並確認密碼，如下所示︰

   ![reset-password](./media/howto-create-manage-server-portal/reset-password.png)

3. 按一下 [確定] 儲存新密碼。

### <a name="scale-vcores-updown"></a>相應增加/減少 vCores

1. 按一下 [設定] 底下的 [定價層]。

2. 將滑桿移至所需的值來變更 [vCore] 設定。

    ![scale-compute](./media/howto-create-manage-server-portal/scale-compute.png)

3. 按一下 [確定] 儲存變更。

### <a name="scale-storage-up"></a>相應增加儲存體

1. 按一下 [設定] 底下的 [定價層]。

2. 將滑桿移至所需的值來變更 [儲存體] 設定。

    ![scale-storage](./media/howto-create-manage-server-portal/scale-storage.png)

3. 按一下 [確定] 儲存變更。

## <a name="delete-an-azure-database-for-mysql-server"></a>刪除適用於 MySQL 的 Azure 資料庫伺服器

1. 從伺服器 [概觀] 中，按一下 [刪除] 按鈕，開啟刪除確認提示。

    ![delete](./media/howto-create-manage-server-portal/delete.png)

2. 將伺服器的名稱輸入至輸入方塊中以便再次確認。

    ![confirm-delete](./media/howto-create-manage-server-portal/confirm.png)

3. 按一下 [刪除]按鈕，確認刪除伺服器。 等待「已成功刪除 MySQL 伺服器」出現在通知列中。

## <a name="list-the-azure-database-for-mysql-databases"></a>列出「適用於 MySQL 的 Azure 資料庫」資料庫
從伺服器 [概觀] 中，向下捲動直到在底部看到資料庫圖格。 資料表中會列出伺服器中的所有資料庫。

   ![顯示資料庫](./media/howto-create-manage-server-portal/show-databases.png)

## <a name="show-details-of-an-azure-database-for-mysql-server"></a>顯示「適用於 MySQL 的 Azure 資料庫」伺服器的詳細資料
按一下位於 [設定] 下的 [屬性]檢視有關伺服器的詳細資訊。

![properties](./media/howto-create-manage-server-portal/properties.png)

## <a name="next-steps"></a>後續步驟

[快速入門：使用 Azure 入口網站建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-portal.md)