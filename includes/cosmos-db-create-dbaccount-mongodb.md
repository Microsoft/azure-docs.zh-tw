---
title: 包含檔案
description: 包含檔案
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 04/13/2018
ms.author: sngun
ms.custom: include file
ms.openlocfilehash: 690c5f07a2b0f7a88e16f0b0bbbaa9ca78e37317
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38728981"
---
1. 在新的視窗中，登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在左側功能表中，依序按一下 [建立資源]、[資料庫]，然後在 [Azure Cosmos DB] 下按一下 [建立]。
   
   ![Azure 入口網站的螢幕擷取畫面，其中反白顯示 [其他服務] 和 Azure Cosmos DB](./media/cosmos-db-create-dbaccount-mongodb/create-nosql-db-databases-json-tutorial-1.png)

3. 在 [新增帳戶] 刀鋒視窗中指定 [MongoDB] 作為 API，並填入所需的 Azure Cosmos DB 帳戶組態。
 
    * [識別碼] 必須是您要用來識別 Azure Cosmos DB 帳戶的唯一名稱。 它只能包含小寫字母、數字及 '-' 字元，且長度必須是 3 到 50 個字元。
    * [訂用帳戶] 是您的 Azure 訂用帳戶。 系統會自動加以填入。
    * [資源群組] 是您 Azure Cosmos DB 帳戶的資源群組名稱。 選取 [新建]，然後為您的帳戶輸入新的資源群組名稱。 為求簡化，您可以使用和識別碼相同的名稱。
    * [位置] 是您的 Azure Cosmos DB 帳戶所在的地理位置。 請選擇最靠近使用者的位置。

    接著，按一下 [建立]。

    ![Azure Cosmos DB 的新帳戶頁面](./media/cosmos-db-create-dbaccount-mongodb/azure-cosmos-db-create-new-account.png)

4. 建立帳戶需要幾分鐘的時間。 等候入口網站顯示 [恭喜!您的 Azure Cosmos DB 帳戶及 MongoDB API 已就緒] 頁面。

    ![Azure 入口網站的 [通知] 窗格](./media/cosmos-db-create-dbaccount-mongodb/azure-cosmos-db-account-created.png)
