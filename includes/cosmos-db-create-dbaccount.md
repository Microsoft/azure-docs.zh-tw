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
ms.openlocfilehash: e07de7ca55a5725fdaf32ee300e8068a1d3d8fce
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47021062"
---
1. 在新的瀏覽器視窗中，登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下 [建立資源] > [資料庫] > [Azure Cosmos DB]。
   
   ![Azure 入口網站資料庫窗格](./media/cosmos-db-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png)

3. 在 [新增帳戶] 頁面中，輸入新的 Azure Cosmos DB 帳戶的設定。 
 
    設定|值|說明
    ---|---|---
    ID|*輸入唯一名稱*|輸入唯一名稱來識別此 Azure Cosmos DB 帳戶。 因為 documents.azure.com 會附加到您所提供的識別碼以建立 URI，請使用可供辨識的唯一識別碼。<br><br>識別碼只能包含小寫字母、數字及連字號 (-) 字元，且必須包含 3 到 50 個字元。
    API|SQL|API 會決定要建立的帳戶類型。 Azure Cosmos DB 提供五個 API：SQL (文件資料庫)、Gremlin (圖形資料庫)、MongoDB (文件資料庫)、Azure 資料表與 Cassandra。 每個 API 目前都要求您建立個別帳戶。 <br><br>選取 [SQL]，因為在本快速入門中，您會使用 SQL 語法建立可查詢的文件資料庫且可使用 SQL API 存取。<br><br>[進一步了解 SQL API](../articles/cosmos-db/documentdb-introduction.md)|
    訂用帳戶|*您的訂用帳戶*|選取您要用於此 Azure Cosmos DB 帳戶的 Azure 訂用帳戶。 
    資源群組|新建<br><br>*然後輸入上面識別碼中所提供的同一個唯一名稱*|選取 [新建]，然後為您的帳戶輸入新的資源群組名稱。 為求簡化，您可以使用和識別碼相同的名稱。 
    位置|*選取最接近使用者的區域*|選取用來主控 Azure Cosmos DB 帳戶的地理位置。 使用最接近使用者的位置，以便他們能以最快速度存取資料。
    啟用異地備援| 保留空白 | 這會在第二個 (配對) 區域建立資料庫的複本。 將此項保留空白。  
    釘選到儀表板 | 選取 | 選取此方塊，以便將新的資料庫帳戶新增至入口網站儀表板以方便存取。

    接著，按一下 [建立]。

    ![Azure Cosmos DB 的新帳戶頁面](./media/cosmos-db-create-dbaccount/azure-cosmos-db-create-new-account.png)

4. 建立帳戶需要幾分鐘的時間。 等候入口網站顯示 [恭喜!**已建立您的 Azure Cosmos DB 帳戶]** 頁面。

    ![Azure 入口網站的 [通知] 窗格](./media/cosmos-db-create-dbaccount/azure-cosmos-db-account-created.png)

