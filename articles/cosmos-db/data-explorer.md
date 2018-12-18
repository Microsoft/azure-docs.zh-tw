---
title: 使用 Azure Cosmos DB 總管來管理資料 | Microsoft Docs
description: Azure Cosmos DB 總管是一個獨立的 Web 介面，可讓您檢視和管理 Azure Cosmos DB 中儲存的資料。
services: cosmos-db
author: deborahc
manager: kfile
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/16/2018
ms.author: Deborah.Chen
ms.openlocfilehash: 844193a3753f76d47a76252386489c9b0fe58911
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39116948"
---
# <a name="use-azure-cosmos-db-explorer-to-manage-your-data"></a>使用 Azure Cosmos DB 總管來管理資料 

Azure Cosmos DB 總管是一個獨立的 Web 介面，可讓您檢視和管理 Azure Cosmos DB 中儲存的資料。 Azure Cosmos DB 總管相當於現有的**資料總管**索引標籤，可在您於 Azure 入口網站中建立 Azure Cosmos DB 帳戶時使用。 Azure Cosmos DB 總管優於現有資料總管的主要優點如下：

* 您可以使用全螢幕畫面來檢視資料，執行查詢、預存程序、觸發程序，並檢視其結果。  

* 您可以為不具 Azure 入口網站或訂用帳戶存取權限的其他使用者，提供暫時或永久的讀取或讀寫存取權來存取您的資料庫帳戶及其集合。  

* 您可以與不具 Azure 入口網站或訂用帳戶存取權的其他使用者共用查詢結果。  

## <a name="access-azure-cosmos-db-explorer"></a>存取 Azure Cosmos DB 總管

1. 登入 [Azure 入口網站](https://portal.azure.com/)。 

2. 從 [所有資源] 中，尋找並瀏覽至您的 Azure Cosmos DB 帳戶、選取 [金鑰]，然後複製 [主要連接字串]。  

3. 移至 https://cosmos.azure.com/、貼上連接字串，然後選取 [連線]。 藉由使用連接字串，您就能存取 Azure Cosmos DB 總管，而且完全沒有時間限制。  

   如果您想要為其他使用者提供您 Azure Cosmos DB 帳戶的暫時存取權，則可使用讀寫和讀取存取 URL 來達成此目的。 

4. 開啟 [資料總管] 刀鋒視窗，然後選取 [開啟全螢幕]。 從快顯對話方塊中，您可以檢視兩個存取 URL：**讀寫**和**讀取**。 這些 URL 可讓您與其他使用者暫時共用您的 Azure Cosmos DB 帳戶。 帳戶的存取權將在 24 小時之後到期，在那之後，您可以使用新的存取 URL 或連接字串重新連線。 

   **讀寫**：當您與其他使用者共用讀寫 URL 時，他們可以檢視和修改資料庫、集合、查詢，以及與該特定帳戶相關聯的其他資源。

   **讀取**：當您與其他使用者共用唯讀 URL 時，他們可以檢視資料庫、集合、查詢，以及與該特定帳戶相關聯的其他資源。 例如，如果您想要與不具 Azure 入口網站或您 Azure Cosmos DB 帳戶存取權的小組成員共用查詢結果，您可以為他們提供此 URL。

   選擇您想要用來開啟帳戶的存取權類型，然後按一下 [開啟]。 當您開啟總管之後，體驗會與您在 Azure 入口網站中使用 [資料總管] 索引標籤的體驗相同。   

   ![開啟 Azure Cosmos DB 總管](./media/data-explorer/open-data-explorer-with-access-url.png)

## <a name="known-issues"></a>已知問題

Azure Cosmos DB Gremlin 和資料表 API 帳戶目前尚未支援可讓您暫時共用讀寫或讀取存取權的**開啟全螢幕**體驗。 您仍然可以藉由將連接字串傳遞至 Azure Cosmos DB 總管，來檢視您的 Gremlin 和資料表 API 帳戶。 

## <a name="next-steps"></a>後續步驟
既然您已了解如何開始使用 Azure Cosmos DB 總管來管理資料，接著您可以：

* 開始使用 SQL 語法來定義[查詢](sql-api-sql-query-reference.md)，並使用預存程序、UDF、觸發程序來執行[伺服器端程式設計](programming.md)。 