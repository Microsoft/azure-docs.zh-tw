---
title: Azure 資料表儲存體概觀 | Microsoft Docs
description: 使用 Azure 表格儲存體 (NoSQL 資料存放區) 將結構化的資料儲存在雲端。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: dotnet
ms.topic: overview
ms.date: 11/03/2017
ms.author: sngun
ms.openlocfilehash: 495fc0fff39a04dee4fc707bd0d2f6b3571cc46a
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38582059"
---
# <a name="azure-table-storage-overview"></a>Azure 資料表儲存體概觀

[!INCLUDE [storage-table-cosmos-db-tip-include](../../includes/storage-table-cosmos-db-tip-include.md)]

Azure 表格儲存體是可將結構化的 NoSQL 資料儲存在雲端中的服務，並提供具有無結構描述設計的索引鍵/屬性存放區。 由於表格儲存體並無結構描述，因此可輕易隨著應用程式發展需求改寫資料。 相較於類似資料量的傳統 SQL，對許多類型的應用程式而言，表格儲存體資料可快速存取且符合成本效益，通常可降低成本。

您可以使用表格儲存體來儲存具彈性的資料集，例如 Web 應用程式的使用者資料、通訊錄、裝置資訊，以及服務所需的其他中繼資料類型。 您可以在資料表中儲存任意數目的實體，且儲存體帳戶可包含任意數目的資料表，最高可達儲存體帳戶的容量限制。

[!INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

## <a name="next-steps"></a>後續步驟

* [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。

* [在 .NET 中開始使用 Azure 表格儲存體](table-storage-how-to-use-dotnet.md)

* 如需可用 API 的完整詳細資訊，請檢視資料表服務參考文件：

    * [Storage Client Library for .NET 參考資料](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)

    * [REST API 參考資料](http://msdn.microsoft.com/library/azure/dd179355)
