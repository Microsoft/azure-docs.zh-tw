---
title: 搭配開放原始碼 NoSQL 資料庫使用 Azure Cosmos DB 多重主機
description: ''
services: cosmos-db
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: ea3a16cf6b5aa4e46ad401e59aacb4d936a7429a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963895"
---
# <a name="using-azure-cosmos-db-multi-master-with-open-source-nosql-databases"></a>搭配開放原始碼 NoSQL 資料庫使用 Azure Cosmos DB 多重主機

Azure Cosmos DB 多重主機支援是原生的伺服器端實作，可供 Cosmos DB 支援的所有開放原始碼 NoSQL 供應項目使用。 所有 Cosmos DB 支援的 SDK 也都可以存取它。

Azure Cosmos DB 是全球第一個為這些開放原始碼 NoSQL 資料庫提供多重主機支援的服務。

|模型  |支援  |
|---------|---------|
|MongoDB  | 主動-主動  |
|圖形  | 主動-主動 |
|Cassandra  | 主動-主動   |

## <a name="use-mongodb-clients-with-multi-master"></a>搭配多重主機使用 MongoDB 用戶端

為 MongoDB API 帳戶啟用多重主機功能的方式與為其他 Azure Cosmos DB API 啟用此功能的方式相同。 為 MongoDB API 帳戶啟用多重主機之後，每個用戶端應用程式執行個體都可選取其慣用的讀取和寫入區域。 從 MongoDB 驅動程式的觀點來看，慣用區域對用戶端來說會顯示為主要複本集。 如此一來，分散式資料庫的每個區域便都可作為主要複本集。 Azure Cosmos DB 多重主機可讓您大幅降低全域分散式 MongoDB 應用程式的寫入延遲。 

### <a name="set-the-primary-region"></a>設定主要區域

每個 MongoDB 用戶端執行個體都可藉由將 `@<preferred_primary_region_name>` 附加至 MongoDB 用戶端驅動程式所接受的「應用程式名稱」欄位，來選取主要區域。 大多數驅動程式都接受在連接字串中採用此做法，例如：

`mongodb://fabrikam:KEY@fabrikam.documents.azure.com:10255/?ssl=true&replicaSet=globaldb&appname=@West US`

連線之後，可能發生下列其中一種情況：

* 如果慣用區域名稱可供作為寫入區域，則 Azure Cosmos DB 會選取它作為主要區域。

* 如果未提供區域名稱，則 Azure Cosmos DB 會選取預設區域作為主要區域。

* 如果已提供慣用區域名稱，但無法作為資料庫帳戶的寫入區域，則 Azure Cosmos DB 會選取可供作為主要區域的最鄰近寫入區域。

某些驅動程式 (例如 NodeJS 驅動程式) 會一律先向已指定初始連接字串的主機發出寫入。 針對這類驅動程式，若要確保將寫入導向到慣用區域，除了應用程式名稱之外，您還應該將連接字串內的 DNS 名稱修改成包含區域名稱。 指定區域名稱時，請務必不要包含任何空格。 例如︰

`mongodb://fabrikam:KEY@fabrikam-westus.documents.azure.com:10255/?ssl=true&replicaSet=globaldb&appname=@West US`

### <a name="conflict-resolution-mode"></a>衝突解決模式

Azure Cosmos DB MongoDB API 帳戶的衝突解決模式一律是「最後寫入為準」，其中會使用接受寫入的區域伺服器時間戳記。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解 Azure Cosmos DB MongoDB API 帳戶的多重主機支援。 接著，請參閱下列資源：

* [如何為 Azure Cosmos DB 帳戶啟用多重主機功能](enable-multi-master.md)

* [了解 Azure Cosmos DB 中的衝突解決方法](multi-master-conflict-resolution.md)
