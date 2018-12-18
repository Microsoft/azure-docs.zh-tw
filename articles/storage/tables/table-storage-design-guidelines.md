---
title: Azure 儲存體資料表設計的指引 | Microsoft Docs
description: 設計您的 Azure 表格服務以有效率地支援讀取作業。
services: storage
author: SnehaGunda
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.author: sngun
ms.component: tables
ms.openlocfilehash: ef6d257aee532d4b6325bd3d2f619fd00824e06f
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525413"
---
# <a name="guidelines-for-table-design"></a>資料表設計指導方針

設計可搭配使用 Azure 儲存體表格服務的資料表，與設計關聯式資料庫的考量相當不同。 本文將說明設計表格服務解決方案的指引，讓讀取和寫入作業更有效率。

## <a name="design-your-table-service-solution-to-be-read-efficient"></a>將表格服務解決方案設計為易於讀取

* ***設計大量讀取應用程式中的查詢。*** 當您在設計資料表時，請先思考您將執行的查詢 (尤其是較無法容續延遲的查詢)，然後再思考更新實體的方法。 透過這樣的方式，通常可造就一個有效率且高效能的解決方案。  
* ***在查詢中同時指定 PartitionKey 和 RowKey。*** *點查詢* 都是最有效率的表格服務查詢。  
* ***考慮儲存重複的實體副本。*** 資料表儲存體的成本較低，因此請考慮多次儲存相同的實體 (使用不同索引鍵)，以啟用更有效率的查詢。  
* ***考慮將資料反正規化。*** 資料表儲存體十分便宜，所以建議您考慮將資料反正規化。 例如儲存摘要實體，可讓彙總資料的查詢只需存取單一實體。  
* ***使用複合索引鍵值。*** 您只有 **PartitionKey** 和 **RowKey** 這兩個索引鍵。 例如，使用複合索引鍵值啟用替代的實體索引鍵式存取路徑。  
* ***使用查詢預測。*** 使用僅選取您所需欄位的查詢，即可減少透過網路傳輸的資料量。  

## <a name="design-your-table-service-solution-to-be-write-efficient"></a>將表格服務解決方案設計為易於寫入  

* ***不要建立熱分割。*** 選擇可讓您在任何時間點，將要求分散到多個分割上的索引鍵。  
* ***避免流量尖峰。*** 將流量平均分散到合理的時段，以避免產生流量尖峰。
* ***不一定要為每個實體的類型建立個別的資料表。*** 需要跨實體類型執行不可部分完成的交易時，您可以在相同資料表的相同分割中儲存多個實體類型。
* ***考慮必須達到的最大輸送量。*** 您必須留意資料表服務的延展性目標，並確保您的設計不會超過目標。  

當您閱讀本指南時，您會看到將這些原則全都納入實作的範例。 

## <a name="next-steps"></a>後續步驟

- [資料表設計模式](table-storage-design-patterns.md)
- [查詢的設計](table-storage-design-for-query.md)
- [加密資料表資料](table-storage-design-encrypt-data.md)
- [資料修改的設計](table-storage-design-for-modification.md)