---
title: 建構資料表設計工具的篩選字串 | Microsoft Docs
description: 建構資料表設計工具的篩選字串
services: visual-studio-online
author: ghogen
manager: douge
assetId: a1a10ea1-687a-4ee1-a952-6b24c2fe1a22
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 11/18/2016
ms.author: ghogen
ms.openlocfilehash: 7390e8f463d14f2e74e05263e970665a0199029a
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31794274"
---
# <a name="constructing-filter-strings-for-the-table-designer"></a>建構資料表設計工具的篩選字串
## <a name="overview"></a>概觀
若要在 Visual Studio [資料表設計工具] 中顯示的 Azure 資料表中篩選資料，您可以建構篩選字串並在篩選欄位中輸入它。 篩選條件字串語法由 WCF Data Services 定義，類似於 SQL WHERE 子句，但會透過 HTTP 要求傳送至表格服務。 [資料表設計工具]  會為您處理適當的編碼，因此，若要篩選所需的屬性值，您只需要在篩選欄位中輸入屬性名稱、比較運算子、準則值和 (選擇性) 布林運算子。 您不需要像透過 [儲存體服務 REST API 參考](http://go.microsoft.com/fwlink/p/?LinkId=400447)建構 URL 來查詢資料表一樣包含 $filter 查詢選項。

WCF Data Services 以 [開放式資料通訊協定](http://go.microsoft.com/fwlink/p/?LinkId=214805) (OData) 為基礎。 如需篩選系統查詢選項 (**$filter**) 的詳細資訊，請參閱 [OData URI 轉換規格](http://go.microsoft.com/fwlink/p/?LinkId=214806)。

## <a name="comparison-operators"></a>比較運算子
所有屬性類型都支援下列邏輯運算子：

| 邏輯運算子 | 說明 | 篩選字串範例 |
| --- | --- | --- |
| eq |等於 |City eq 'Redmond' |
| gt |大於 |Price gt 20 |
| ge |大於或等於 |Price ge 10 |
| lt |小於 |Price lt 20 |
| le |小於或等於 |Price le 100 |
| ne |不等於 |City ne 'London' |
| 和 |和 |Price le 200 and Price gt 3.5 |
| 或 |或 |Price le 3.5 or Price gt 200 |
| 否 |否 |not isAvailable |

建構篩選字串時，下列規則很重要：

* 使用邏輯運算子來比較屬性與值。 請注意，無法比較屬性與動態值。運算式的一端必須是常數。
* 篩選字串的所有部分都區分大小寫。
* 常數和屬性必須是相同的資料類型，篩選才能傳回有效的結果。 如需支援的屬性類型的詳細資訊，請參閱 [了解表格服務資料模型](http://go.microsoft.com/fwlink/p/?LinkId=400448)。

## <a name="filtering-on-string-properties"></a>篩選字串屬性
當您篩選字串屬性時，請用單引號括住字串常數。

下列範例會篩選 **PartitionKey** 和 **RowKey** 屬性。額外的非索引鍵屬性也可以加入至篩選字串：

    PartitionKey eq 'Partition1' and RowKey eq '00001'

您可以用括號括住每個篩選運算式 (雖然並非必要)：

    (PartitionKey eq 'Partition1') and (RowKey eq '00001')

請注意，表格服務不支援萬用字元查詢，在 [資料表設計工具] 中也不支援。 不過，您可以在所需的前置詞上使用比較運算子，以執行前置詞比對。 下列範例會傳回 LastName 屬性開頭為字母 'A' 的實體：

    LastName ge 'A' and LastName lt 'B'

## <a name="filtering-on-numeric-properties"></a>篩選數值屬性
若要篩選整數或浮點數，指定數值且不加引號。

這個範例會傳回與 Age 屬性的值大於 30 的所有實體：

    Age gt 30

這個範例會傳回與 AmountDue 屬性的值小於或等於 100.25 的所有實體：

    AmountDue le 100.25

## <a name="filtering-on-boolean-properties"></a>篩選布林值屬性
若要篩選布林值，請指定 **true** 或 **false** 且不加引號。

下列範例會傳回 IsActive 屬性設定為 **true**的所有實體：

    IsActive eq true

您也可以不加邏輯運算子來撰寫這個篩選運算式。 在下列範例中，表格服務也會傳回 IsActive 為 **true**的所有實體：

    IsActive

若要傳回 IsActive 為 false 的所有實體，您可以使用 not 運算子：

    not IsActive

## <a name="filtering-on-datetime-properties"></a>篩選 DateTime 屬性
若要 DateTime 值，請指定 **datetime** 關鍵字，後面加上以單引號括住的日期/時間常數。 日期/時間常數必須使用 UTC 組合格式，如 [格式化 DateTime 屬性值](http://go.microsoft.com/fwlink/p/?LinkId=400449)所述。

下列範例會傳回 CustomerSince 屬性等於 2008 年 7 月 10 日的實體：

    CustomerSince eq datetime'2008-07-10T00:00:00Z'
