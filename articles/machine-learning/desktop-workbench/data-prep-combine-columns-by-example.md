---
title: 使用 Azure Machine Learning Workbench 進行合併資料行實例化轉換
description: 「合併資料行實例化」轉換的參考文件
services: machine-learning
author: ranvijaykumar
ms.author: ranku
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc, reference
ms.topic: article
ms.date: 09/14/2017
ROBOTS: NOINDEX
ms.openlocfilehash: c952882a0a12e288eaf4f5f1074a9a2e876a21d6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986229"
---
# <a name="combine-columns-by-example-transformation"></a>合併資料行實例化轉換

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


此轉換可讓使用者藉由合併多個資料行的值以新增資料行。 使用者可以指定分隔符號，或提供組合值的範例以執行這項轉換。 當使用者提供合併範例時，轉換由**衍生資料行實例化**轉換中使用的相同**實例化**引擎處理。

## <a name="how-to-perform-this-transformation"></a>如何執行此轉換

若要執行此轉換，請遵循下列步驟：
1. 選取兩個以上的資料行以合併成一個資料行。 
2. 從 [轉換] 功能表選取 [合併資料行實例化]。 或者，以滑鼠右鍵按一下任一選取資料行的標頭，並從操作功能表選取 [合併資料行實例化]。 「轉換編輯器」隨即開啟，並在最右邊選取的資料行旁邊新增一個新資料行。 新的資料行包含由預設分隔符號分隔的組合值。 可以根據資料行標頭中的核取方塊來識別選取的資料行。 可以使用核取方塊來新增和移除選擇的資料行。
3. 您可以更新新建資料行中的組合值。 會使用更新的值作為了解轉換的範例。
4. 按一下 [確定] 以接受轉換。

### <a name="transform-editor-advanced-mode"></a>轉換編輯器：進階模式

「進階模式」針對合併資料行提供更豐富的體驗。 

在 [合併資料行依據] 下選取 [分隔符號]，可讓使用者在 [分隔符號] 文字方塊中指定字串。 從 [分隔符號] 文字方塊中選出，以預覽資料方格中的結果。 按 [確定] 以認可轉換。

在 [合併資料行依據] 下選取 [範例]，可讓使用者提供組合值的範例。 若要升級資料列作為範例，請按兩下格線中的資料列。 在文字方塊中，對應升級的資料列輸入預期的輸出。 從 [分隔符號] 文字方塊中選出，以預覽資料方格中的結果。 按 [確定] 以認可轉換。 

使用者可按一下「轉換編輯器」中的連結，以在**基本模式**和**進階模式**之間切換。

### <a name="transform-editor-send-feedback"></a>轉換編輯器：傳送意見反應

按一下 [傳送意見反應] 連結會開啟 [意見反應] 對話方塊，其中會顯示具有使用者所提供範例的意見方塊。 使用者應該檢閱意見方塊的內容並提供更多詳細資料以協助我們了解問題。 若使用者不想要與 Microsoft 分享資料，使用者在按下 [傳送意見反應] 按鈕之前應該先刪除預先填寫的範例資料。 

### <a name="editing-existing-transformation"></a>編輯現有的轉換

使用者可選取「轉換步驟」中的 [編輯] 選項，以編輯現有的**合併資料行實例化**轉換。 按一下 [編輯]，會在**基本模式**中開啟「轉換編輯器」。 使用者可以按一下標頭中的連結，以進入**進階模式**。 此處會顯示建立轉換期間提供的所有範例。

## <a name="example-using-separators"></a>使用分隔符號的範例

在此範例中，使用逗號後加一個空格作為分隔符號，以合併 *Street*、*City*、*State*，以及 *ZIP* 資料行。

|Street|City|State|ZIP|欄|
|:----|:----|:----|:----|:----|
|16011 N.E. 36th Way|REDMOND|WA|98052|16011 N.E. 36th Way, REDMOND, WA, 98052|
|16021 N.E. 36th Way|REDMOND|WA|98052|16021 N.E. 36th Way, REDMOND, WA, 98052|
|16031 N.E. 36th Way|REDMOND|WA|98052|16031 N.E. 36th Way, REDMOND, WA, 98052|
|16041 N.E. 36th Way|REDMOND|WA|98052|16041 N.E. 36th Way, REDMOND, WA, 98052|
|16051 N.E. 36th Way|REDMOND|WA|98052|16051 N.E. 36th Way, REDMOND, WA, 98052|
|16061 N.E. 36th Way|REDMOND|WA|98052|16061 N.E. 36th Way, REDMOND, WA, 98052|
|3460 157th Avenue NE|REDMOND|WA|98052|3460 157th Avenue NE, REDMOND, WA, 98052|
|3350 157th Ave N.E.|REDMOND|WA|98052|3350 157th Ave N.E., REDMOND, WA, 98052|
|3240 157th Avenue N.E.|REDMOND|WA|98052|3240 157th Avenue N.E., REDMOND, WA, 98052|

## <a name="example-using-by-example"></a>使用實例化的範例

已提供**粗體**值作為範例。

|日期|月|Year|小時|分鐘|秒|合併資料行|
|:----|:----|:----|:----|:----|:----|:----|
|13|Oct|2016|15|01|23|**13-Oct-2016 15:01:23 PDT**|
|16|Oct|2016|16|22|33|16-Oct-2016 15:01:33 PDT|
|17|Oct|2016|12|43|12|17-Oct-2016 15:01:12 PDT|
|12|Nov|2016|14|22|44|12-Nov-2016 15:01:44 PDT|
|23|Nov|2016|01|52|45|23-Nov-2016 15:01:45 PDT|
|16|Jan|2017|22|34|56|16-Jan-2016 15:01:56 PDT|
|23|Mar|2017|01|55|25|23-Mar-2016 15:01:25 PDT|
|16|Apr|2017|11|34|36|16-Apr-2016 15:01:36 PDT|

