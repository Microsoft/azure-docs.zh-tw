---
title: 在 Azure 搜尋服務中匯入資料 | Microsoft Docs
description: 了解如何將資料上傳至 Azure 搜尋服務中的索引
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 01/05/2018
ms.author: heidist
ms.openlocfilehash: b518d86d5cb08f85beecd4afe30efc9e4f70e1f7
ms.sourcegitcommit: e45b2aa85063d33853560ec4bc867f230c1c18ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43371463"
---
# <a name="indexing-in-azure-search"></a>在 Azure 搜尋服務中編製索引
> [!div class="op_single_selector"]
> * [概觀](search-what-is-data-import.md)
> * [.NET](search-import-data-dotnet.md)
> * [REST](search-import-data-rest-api.md)
> 
> 

在 Azure 搜尋服務中，對您的內容執行的查詢會載入到[搜尋索引](search-what-is-an-index.md)中。 本文會檢查將內容載入到索引中的兩種基本方法：以程式設計方式將資料「推送」到索引中，或指向位於支援的資料來源並且要在資料中「提取」的 [Azure 搜尋索引子](search-indexer-overview.md)。

## <a name="pushing-data-to-an-index"></a>將資料推送到索引
推送模型是最具彈性的方法，用來以程式設計方式將資料傳送至 Azure 搜尋。 首先，它沒有資料來源類型的限制。 任何由 JSON 文件組成的資料集都可以推送到 Azure 搜尋服務索引，並假設資料集中的每份文件都有索引結構描述中定義之欄位的對應欄位。 其次，它沒有執行頻率的限制。 您可以視需要經常將變更推送到索引。 若應用程式需要極低的延遲 (例如，如果您需要讓搜尋作業與動態庫存資料庫保持同步)，推送模式是您的唯一選擇。

這種方法比提取模型更有彈性，因為您可以個別或批次上傳文件 (每個批次最多 1000 個或 16 MB，視何者先到達)。 推送模型也可讓您將文件上傳至 Azure 搜尋服務，無論資料是存放在哪裡。

### <a name="how-to-push-data-to-an-azure-search-index"></a>如何將資料推送到 Azure 搜尋服務索引

您可以使用下列 API，將單一或多份文件載入索引中：

+ [新增、更新或刪除文件 (REST API)](https://docs.microsoft.com/rest/api/searchservice/AddUpdate-or-Delete-Documents)
+ [indexAction class](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexaction?view=azure-dotnet) 或 [indexBatch class](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexbatch?view=azure-dotnet) 

目前沒有工具可支援透過入口網站推送資料。

如需每個方法的簡介，請參閱[使用 REST 匯入資料](search-import-data-rest-api.md)或[使用 .NET 匯入資料](search-import-data-dotnet.md)。


## <a name="pulling-data-into-an-index"></a>將資料提取到索引中
提取模型會將支援的資料來源編目，並自動將資料上傳到您的索引。 在 Azure 搜尋服務中，這項功能是透過「索引子」來實作，目前適用於下列平台：

+ [Blob 儲存體](search-howto-indexing-azure-blob-storage.md)
+ [資料表儲存體](search-howto-indexing-azure-tables.md)
+ [Azure Cosmos DB](http://aka.ms/documentdb-search-indexer)
+ [Azure VM 上的 Azure SQL Database 和 SQL Server](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

索引子可將索引連接到資料來源 (通常是資料表、檢視或對等結構)，並將來源欄位對應至索引中的對等欄位。 在執行期間，資料列集會自動轉換為 JSON 並載入指定的索引。 所有索引子都支援排程，以便您指定資料重新整理的頻率。 如果資料來源支援索引子，則大部分的索引子都會提供變更追蹤。 除了辨識新文件，索引子還會追蹤現有文件的變更和刪除，讓您不必主動管理索引中的資料。 


### <a name="how-to-pull-data-into-an-azure-search-index"></a>如何將資料提取到 Azure 搜尋服務索引

索引子功能會在 [Azure 入口網站](search-import-data-portal.md)、[REST API](/rest/api/searchservice/Indexer-operations) 以及 [.NET SDK](/dotnet/api/microsoft.azure.search.indexersoperationsextensions)中出現。 

使用入口網站的優點是 Azure 搜尋服務通常可藉由讀取來源資料集的中繼資料，為您產生預設的索引結構描述。 直到處理索引後，您才可以修改產生的索引，而後只允許不需要重新編製索引的結構描述編輯。 如果您想要進行的變更會直接影響結構描述，您必須重建索引。 

## <a name="verify-data-import-with-search-explorer"></a>透過搜尋總管確認資料匯入

對文件上傳執行初步檢查的快速方法是使用入口網站中的 [搜尋總管]。 此總管可讓您查詢索引，而不需撰寫任何程式碼。 搜尋體驗是以預設設定為基礎，例如[簡單語法](/rest/api/searchservice/simple-query-syntax-in-azure-search)和預設 [searchMode 查詢參數](/rest/api/searchservice/search-documents)。 結果會以 JSON 格式傳回，以便您檢查整份文件。

> [!TIP]
> 許多 [Azure 搜尋服務程式碼範例](https://github.com/Azure-Samples/?utf8=%E2%9C%93&query=search)包含內嵌或隨手可得的資料集，並提供簡單的開始使用方式。 入口網站也會提供範例索引子和資料來源 (由稱為 "realestate-us-sample" 的小型不動產資料集所組成)。 當您在範例資料來源上執行預先設定的索引子時，系統會建立索引並隨著文件載入，而後即可在 [搜尋總管] 中或由您撰寫的程式碼查詢索引。

## <a name="see-also"></a>另請參閱

+ [索引子概觀](search-indexer-overview.md)
+ [入口網站逐步解說：建立、載入、查詢索引](search-get-started-portal.md)
