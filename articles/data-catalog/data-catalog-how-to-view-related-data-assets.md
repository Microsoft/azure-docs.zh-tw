---
title: 如何檢視 Azure 資料目錄中的相關資料資產
description: 本文說明如何檢視「Azure 資料目錄」中所選資料資產的相關資料資產。
services: data-catalog
author: steelanddata
ms.author: maroche
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: d680cc69d27681883014a414255ad0ea4d022cd4
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052811"
---
# <a name="how-to-view-related-data-assets-in-azure-data-catalog"></a>如何檢視 Azure 資料目錄中的相關資料資產？
「Azure 資料目錄」可讓您檢視與所選資料資產相關的資料資產，並檢視它們之間的關係。 

## <a name="supported-data-sources"></a>支援的資料來源 
當您從下列資料來源註冊資料資產時，「Azure 資料目錄」會自動註冊與所選資料資產之間聯結關聯性相關的中繼資料。 

- SQL Server
- 連接字串
- MySQL
- Oracle

> [!NOTE]
> 若要讓資料目錄匯入兩個資料資產之間的關聯性，您必須同時註冊兩個資產。 如果您已個別新增其中一個，將它再新增一次，然後新增另一個資料資產，接著匯入它們之間的關聯性。

## <a name="view-related-data-assets"></a>檢視相關的資料資產
若要檢視與所選資料集相關的資料資產，請使用 [關聯性] 索引標籤，如下圖所示： 

![Azure 資料目錄 - 檢視相關的資料資產](media\data-catalog-how-to-view-related-data-assets\relationships-tab.png)

在此範例中，所選取的 **ProductSubcategory** 資料資產有兩個關聯性： 

- [Product] 資料表的 [ProductSubcategoryID] 資料行與所選 [ProductSubcategory] 資料表的 [ProductSubcategoryID] 資料行之間具有外部索引鍵關聯性。 
- [ProductSubCategory] 資料表的 [ProductCategoryID] 資料行與所選 [ProductCategory] 資料表的 [ProductCategoryID] 資料行之間具有外部索引鍵關聯性。

> [!NOTE]
> 請注意關聯性樹狀檢視中的箭頭方向。  

若要查看更多詳細資料 (例如資料行的完整名稱)，請將滑鼠游標移至其上方，您就會看到類似下圖的快顯訊息： 

![Azure 資料目錄 - 關聯性快顯訊息](media\data-catalog-how-to-view-related-data-assets\relationship-popup.png)

若要包含已註冊之資產間的關聯性，請重新註冊這些資產。

## <a name="next-steps"></a>後續步驟
- [如何管理資料資產](data-catalog-how-to-manage.md)