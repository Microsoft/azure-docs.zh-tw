---
title: Azure Cosmos DB 帳戶的 MongoDB 連接字串 | Microsoft Docs
description: 了解如何使用 MongoDB 連接字串，將 MongoDB 應用程式連線至 Azure Cosmos DB 帳戶。
keywords: mongodb 連接字串
services: cosmos-db
author: slyons
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.devlang: na
ms.topic: conceptual
ms.date: 12/19/2017
ms.author: sclyon
ms.openlocfilehash: ad8d6fe36c289c4c9e37689e1c7d755dc3bf9048
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038082"
---
# <a name="connect-a-mongodb-application-to-azure-cosmos-db"></a>將 MongoDB 應用程式連接到 Azure Cosmos DB
了解如何使用 MongoDB 連接字串，將 MongoDB 應用程式連線至 Azure Cosmos DB 帳戶。 您可以使用 Azure Cosmos DB 資料庫作為 MongoDB 應用程式的資料存放區。 

本教學課程提供兩種方式來擷取連接字串資訊︰

- [快速啟動方法](#QuickstartConnection)，用來搭配 .NET、Node.js、MongoDB 殼層、Java 和 Python 驅動程式
- [自訂連接字串方法](#GetCustomConnection)，用來搭配其他驅動程式

## <a name="prerequisites"></a>必要條件

- 一個 Azure 帳戶。 如果您沒有 Azure 帳戶，可以立即建立一個[免費的 Azure 帳戶](https://azure.microsoft.com/free/)。 
- Azure Cosmos DB 帳戶。 如需相關指示，請參閱[使用 .NET 和 Azure 入口網站建置 MongoDB API Web 應用程式](create-mongodb-dotnet.md)。

## <a id="QuickstartConnection"></a>使用快速入門取得 MongoDB 連接字串
1. 在網際網路瀏覽器中，登入 [Azure 入口網站](https://portal.azure.com)。
2. 在 [Azure Cosmos DB] 刀鋒視窗中，選取 MongoDB 帳戶的 API。 
3. 在帳戶刀鋒視窗的左窗格中，按一下 [快速入門]。 
4. 選擇您的平台 (**.NET**、**Node.js**、**MongoDB 殼層**、**Java**、**Python**)。 如果您沒有看到您的驅動程式或工具被列出，別擔心，我們會持續加入更多連線程式碼片段。 請在下面加入您想要看到何種內容的意見。 若要了解如何製作您自己的連線，請閱讀[取得帳戶的連接字串資訊](#GetCustomConnection)。
5. 將程式碼片段複製和貼上您的 MongoDB 應用程式。

    ![快速入門刀鋒視窗](./media/connect-mongodb-account/QuickStartBlade.png)

## <a id="GetCustomConnection"></a> 取得 MongoDB 連接字串以自訂
1. 在網際網路瀏覽器中，登入 [Azure 入口網站](https://portal.azure.com)。
2. 在 [Azure Cosmos DB] 刀鋒視窗中，選取 MongoDB 帳戶的 API。 
3. 在帳戶刀鋒視窗的左窗格中，按一下 [連接字串]。 
4. [連接字串] 刀鋒視窗隨即開啟。 其中包含使用 MongoDB 適用的驅動程式連線至帳戶所需的所有資訊，包括預先建構的連接字串。

    ![連接字串刀鋒視窗](./media/connect-mongodb-account/ConnectionStringBlade.png)

## <a name="connection-string-requirements"></a>連接字串需求
> [!Important]
> Azure Cosmos DB 有嚴格的安全性需求和標準。 Azure Cosmos DB 帳戶需要驗證和透過 *SSL* 的安全通訊。 
>
>

Azure Cosmos DB 支援標準 MongoDB 連接字串 URI 格式，有幾個特定需求︰Azure Cosmos DB 帳戶需要驗證和透過 SSL 的安全通訊。 所以，連接字串格式為：

    mongodb://username:password@host:port/[database]?ssl=true

在先前顯示的 [連接字串] 刀鋒視窗中可取得此字串的值：

* 使用者名稱 (必要)：Azure Cosmos DB 帳戶名稱。
* 密碼 (必要)：Azure Cosmos DB 帳戶密碼。
* 主機 (必要)：Azure Cosmos DB 帳戶的 FQDN。
* 連接埠 (必要)：10255。
* 資料庫 (選用)：連線所使用的資料庫。 如果未提供資料庫，則預設資料庫是 "test"。
* ssl=true (必要)

例如，請考慮 [連接字串] 刀鋒視窗中顯示的帳戶。 有效的連接字串為：

    mongodb://contoso123:0Fc3IolnL12312asdfawejunASDF@asdfYXX2t8a97kghVcUzcDv98hawelufhawefafnoQRGwNj2nMPL1Y9qsIr9Srdw==@contoso123.documents.azure.com:10255/mydatabase?ssl=true

## <a name="next-steps"></a>後續步驟
* 了解如何[使用 Studio 3T (MongoChef)](mongodb-mongochef.md) 搭配 Azure Cosmos DB API for MongoDB 帳戶。
* 檢視[範例](mongodb-samples.md)，以瀏覽 MongoDB 適用的 Azure Cosmos DB API。
