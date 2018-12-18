---
title: 使用 Robomongo 連絡 Azure Cosmos DB | Microsoft Docs
description: 了解如何使用 Robomongo 搭配 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶
keywords: robomongo
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: na
ms.topic: conceptual
ms.date: 05/23/2017
ms.author: sngun
ms.openlocfilehash: b6d64d7d7b30d4175fb8c8bf6c98127465427d4e
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34795017"
---
# <a name="use-robomongo-with-an-azure-cosmos-db-api-for-mongodb-account"></a>使用 Robomongo 搭配 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶
若要使用 Robomongo 連線到 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶，您必須︰

* 下載並安裝 [Robomongo](https://robomongo.org/)
* 具備您的 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶[連接字串](connect-mongodb-account.md)資訊

## <a name="connect-using-robomongo"></a>使用 Robomongo 來連接
若要將 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶新增至 Robomongo MongoDB 連線，請執行下列步驟。

1. 使用[這裡](connect-mongodb-account.md)的指示來擷取 Azure Cosmos DB：適用於 MongoDB 的 API 帳戶的連線資訊。

    ![[連接字串] 刀鋒視窗的螢幕擷取畫面](./media/mongodb-robomongo/connectionstringblade.png)
2. 執行 *Robomongo.exe*

3. 按一下 [檔案] 下的 [連接] 按鈕來管理您的連接。 然後，按一下 [MongoDB 連接] 視窗中的 [建立]，將會開啟 [連接設定] 視窗。

4. 在 [連接設定] 視窗中，選擇名稱。 然後，從步驟 1 的連接資訊中，找出 [主機] 和 [連接埠]，分別輸入至 [位址] 和 [連接埠]。

    ![Robomongo 管理連線的螢幕擷取畫面](./media/mongodb-robomongo/manageconnections.png)
5. 在 [驗證] 索引標籤上，按一下 [執行驗證]。 然後，輸入您的 [資料庫] \(預設值是 *Admin*)、[使用者名稱] 和 [密碼]。
在步驟 1 的連接資訊中，可以找到 [使用者名稱] 和 [密碼]。

    ![Robomongo 驗證索引標籤的螢幕擷取畫面](./media/mongodb-robomongo/authentication.png)
6. 在 [SSL] 索引標籤上，勾選 [使用 SSL 通訊協定]，然後將 [驗證方法] 變更為 [自我簽署憑證]。

    ![Robomongo SSL 索引標籤的螢幕擷取畫面](./media/mongodb-robomongo/SSL.png)
7. 最後，按一下 [測試] 以確認您能夠連接，然後按一下 [儲存]。

## <a name="next-steps"></a>後續步驟
* 瀏覽 Azure Cosmos DB：適用於 MongoDB 的 API [範例](mongodb-samples.md)。
