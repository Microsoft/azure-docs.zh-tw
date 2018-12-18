---
title: 在 Azure Stack 上使用 MySQL 配接器 RP 所提供的資料庫 | Microsoft Docs
description: 如何建立及管理使用 MySQL 配接器資源提供者佈建的 MySQL 資料庫
services: azure-stack
documentationCenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/26/2018
ms.author: jeffgilb
ms.reviewer: jeffgo
ms.openlocfilehash: 77dca29b0c60726f0a072dd662aba0d12730502a
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39413683"
---
# <a name="create-mysql-databases"></a>建立 MySQL 資料庫

您可以在使用者入口網站中建立及管理自助式資料庫。 Azure Stack 使用者需要供應項目中包含 MySQL 資料庫服務的訂用帳戶。

## <a name="test-your-deployment-by-creating-a-mysql-database"></a>建立 MySQL 資料庫來測試部署

1. 登入 Azure Stack 使用者入口網站。
2. 選取 [+ 新增] > [資料 + 儲存體] > [MySQL 資料庫] > [新增]。
3. 在 [建立 MySQL 資料庫] 底下，輸入 [資料庫名稱]，然後視您環境的需求設定其他設定。

    ![建立測試 MySQL 資料庫](./media/azure-stack-mysql-rp-deploy/mysql-create-db.png)

4. 在 [建立資料庫] 底下，選取 [SKU]。 在 [選取 MySQL SKU] 底下，選取資料庫的 SKU。

    ![選取 MySQL SKU](./media/azure-stack-mysql-rp-deploy/mysql-select-sku.png)

    >[!Note]
    >將主控伺服器新增至 Azure Stack 時，系統會指派一個 SKU 給它們。 並會在 SKU 的主控伺服器集區中建立資料庫。

5. 在 [登入] 底下，選取 [設定必要設定]。
6. 在 [選取登入] 底下，選擇現有的登入，或選取 [+ 建立新的登入] 來設定新的登入。  輸入 [資料庫登入] 名稱和 [密碼]，然後選取 [確定]。

    ![建立新的資料庫登入](./media/azure-stack-mysql-rp-deploy/create-new-login.png)

    >[!NOTE]
    >MySQL 5.7 中的資料庫登入名稱長度不可超過 32 個字元。 在較舊的版本中則不可超過 16 個字元。

7. 選取 [建立] 以完成資料庫設定。

在部署資料庫之後，記下 [基本資訊] 底下的 [連接字串]。 您可以在任何需要存取 MySQL 資料庫的應用程式中使用這個字串。

![取得 MySQL 資料庫的連接字串](./media/azure-stack-mysql-rp-deploy/mysql-db-created.png)

## <a name="update-the-administrative-password"></a>更新系統管理密碼

您可以在 MySQL 伺服器執行個體上變更密碼來修改密碼。

1. 選取 [系統管理資源] > [MySQL 主控伺服器]。 選取主控伺服器。
2. 在 [設定] 底下，選取 [密碼]。
3. 在 [密碼] 底下，輸入新密碼，然後選取 [儲存]。

![更新管理員密碼](./media/azure-stack-mysql-rp-deploy/mysql-update-password.png)

## <a name="next-steps"></a>後續步驟

[更新 MySQL 資源提供者](azure-stack-mysql-resource-provider-update.md)
