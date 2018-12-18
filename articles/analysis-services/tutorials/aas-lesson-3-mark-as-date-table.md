---
title: Azure Analysis Services 教學課程第 3 課：標記為日期資料表 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中標記日期資料表。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 6627cf74154e66a84f34d5d5bc1d78ed3e0c38a2
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443594"
---
# <a name="mark-as-date-table"></a>標示為日期資料表

在「第 2 課：取得資料」中，您已匯入名為 DimDate 的維度資料表。 雖然此資料表在您的模型中名為 DimDate，但可以也稱為「日期資料表」，因為它包含日期和時間資料。  
  
每當您使用 DAX 時間智慧函式時 (像是稍後您建立量值時)，您必須指定一些屬性，其中包括「日期資料表」和該資料表中的唯一識別碼「日期資料行」。
  
在這堂課中，您會將 DimDate 資料表標記為「日期資料表」，並將 Date 資料行 (在 Date 資料表中) 標記為「日期資料行」 (唯一識別碼)。  

在標記日期資料表和日期資料行之前，最好稍微整理一下，讓您的模型更容易懂。 請注意 DimDate 資料表中名為 **FullDateAlternateKey** 的資料行。 對於資料表內含的每個日曆年度中的每一天，此資料行都包含一個資料列。 您在量值公式和報告中常會用到此資料行。 但是，FullDateAlternateKey 並不是此資料行的理想識別項。 您會將它重新命名為 **Date**，使其更容易識別和納入公式中。 可能的話，最好重新命名物件，例如資料表和資料行，以更容易在 SSDT 和用戶端報告應用程式 (例如 Power BI 和 Excel) 中識別。 
  
這堂課的預估完成時間：**3 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 2 課︰取得資料](../tutorials/aas-lesson-2-get-data.md)。 

### <a name="to-rename-the-fulldatealternatekey-column"></a>重新命名 FullDateAlternateKey 資料行

1.  在模型設計師中，按一下 [DimDate] 資料表。

2.  按兩下 [FullDateAlternateKey] 資料行的標頭，然後將它重新命名為 **Date**。

  
### <a name="to-set-mark-as-date-table"></a>設定標記為日期資料表  
  
1.  選取 [Date] 資料行，然後在 [屬性] 視窗的 [資料類型] 下方，確定已選取 [日期]。  
  
2.  依序按一下 [資料表] 功能表、[日期]和 [標記為日期資料表]。  
  
3.  在 [標記為日期資料表] 對話方塊中，在 [日期] 清單方塊中選取 [Date] 資料行當作唯一識別碼。 依預設通常會選取它。 按一下 [確定]。 

    ![aas-lesson3-date-table](../tutorials/media/aas-lesson3-date-table.png)
  

## <a name="whats-next"></a>後續步驟
[第 4 課：建立關聯性](../tutorials/aas-lesson-4-create-relationships.md)。
  
