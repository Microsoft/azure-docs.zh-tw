---
title: Azure Cosmos DB 的 Azure 表格儲存體支援 | Microsoft Docs
description: 了解 Azure Cosmos DB 資料表 API 和 Azure 儲存體資料表如何一起運作。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: na
ms.topic: overview
ms.date: 11/15/2017
ms.author: sngun
ms.openlocfilehash: 114286b45df5f47e81bd2b990c8b50c8b7b7a482
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43185361"
---
# <a name="developing-with-azure-cosmos-db-table-api-and-azure-table-storage"></a>使用 Azure Cosmos DB 資料表 API 和 Azure 表格儲存體進行開發

Azure Cosmos DB 資料表 API 和 Azure 表格儲存體共用同一個資料表資料模型，並透過本身的 SDK 公開相同的建立、刪除、更新和查詢作業。 

[!INCLUDE [storage-table-cosmos-comparison](../../includes/storage-table-cosmos-comparison.md)]

## <a name="developing-with-the-azure-cosmos-db-table-api"></a>使用 Azure Cosmos DB 資料表 API 進行開發

此時，[Azure Cosmos DB 資料表 API](table-introduction.md) 有四個 SDK 可供開發使用： 
- [Microsoft.Azure.CosmosDB.Table](https://aka.ms/tableapinuget) .NET SDK。 此文件庫具有與公用 [Windows Azure 儲存體 SDK](https://www.nuget.org/packages/WindowsAzure.Storage) 相同的類別和方法簽章，但它也可以使用資料表 API 來連線到 Azure Cosmos DB 帳戶。 請注意，`Microsoft.Azure.CosmosDB.Table` 文件庫目前僅適用於 .NET Standard，尚不適用於 .NET Core。
- [Python SDK](table-sdk-python.md)。 新的 Azure Cosmos DB Python SDK 是唯一支援 Azure 表格儲存體 (Python) 的 SDK。 此 SDK 與 Azure 表格儲存體和 Azure Cosmos DB 資料表 API 連線。
- [Java SDK](table-sdk-java.md)。 此 Azure 儲存體 SDK 能夠使用資料表 API 連線至 Azure Cosmos DB 帳戶。
- [Node.js SDK](table-sdk-nodejs.md)。 此 Azure 儲存體 SDK 能夠使用資料表 API 連線至 Azure Cosmos DB 帳戶。

[常見問題集：使用資料表 API 開發](faq.md#table)一文提供使用資料表 API 的其他資訊。

## <a name="developing-with-azure-table-storage"></a>使用 Azure 表格儲存體進行開發

Azure 表格儲存體有下列 SDK 可用於開發：

- [WindowsAzure.Storage .NET SDK](https://www.nuget.org/packages/WindowsAzure.Storage/). 此程式庫可讓您使用表格儲存體服務。
- [Python SDK](table-sdk-python.md)。 適用於 Python 的 Azure Cosmos DB 資料表 SDK 也支援儲存體表格服務。
- [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)。 此 Azure 儲存體 SDK 會在 Java 中提供用戶端程式庫，以取用 Azure 表格儲存體。
- [Node.js SDK](table-sdk-nodejs.md)。 這個 SDK 提供 Node.js 套件和瀏覽器相容 JavaScript 用戶端程式庫，以取用儲存體資料表服務。
- [AzureRmStorageTable PowerShell 模組](https://www.powershellgallery.com/packages/AzureRmStorageTable/1.0.0.7). 這個 PowerShell 模組有 Cmdlet 可處理儲存體資料表。
- [Azure Storage Client Library for C++](https://github.com/Azure/azure-storage-cpp/)。 此程式庫可讓您針對 Azure 儲存體建置應用程式。
- [適用於 Ruby 的 Azure 儲存體資料表用戶端程式庫](https://github.com/azure/azure-storage-ruby/tree/master/table)。 這個專案提供 Ruby 套件，可讓您輕鬆地存取 Azure 儲存體資料表服務。
- [Azure 儲存體資料表 PHP 用戶端程式庫](https://github.com/Azure/azure-storage-php/tree/master/azure-storage-table)。 這個專案提供 PHP 用戶端程式庫，可讓您輕鬆地存取 Azure 儲存體資料表服務。


   





