---
title: Azure Data Factory 複製精靈 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 複製精靈，將資料從支援的資料來源複製到接收。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 0974eb40-db98-4149-a50d-48db46817076
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 6330854f4ee9b2db1597d916c815bb6646da6a29
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37052744"
---
# <a name="azure-data-factory-copy-wizard"></a>Azure Data Factory 複製精靈
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 

Azure Data Factory 複製精靈簡化內嵌資料的程序，這通常是端對端資料整合案例中的第一個步驟。 逐步執行 Azure Data Factory 複製精靈時，您不需要了解任何用於連結服務、資料集和管線的 JSON 定義。 精靈會自動建立管線，將資料從選取的資料來源複製到選取的目的地。 此外，複製精靈可協助您在撰寫時驗證內嵌的資料。 這樣可節省時間，尤其是當您第一次從資料來源內嵌資料時更是如此。 若要啟動複製精靈，請在 Data Factory 首頁按一下 [複製資料]  圖格。

![複製精靈](./media/data-factory-copy-wizard/copy-data-wizard.png)

## <a name="designed-for-big-data"></a>針對巨量資料設計
此精靈可讓您在數分鐘內輕鬆地將資料從各種來源移至目的地。 逐步執行精靈之後，即會自動建立具有複製活動的管線以及相依的 Data Factory 實體 (連結的服務和資料集)。 不需任何額外的步驟，即可建立管線。   

![選取資料來源](./media/data-factory-copy-wizard/select-data-source-page.png)

> [!NOTE]
> 如需取得建立範例管線以便將資料從 Azure Blob 複製到 Azure SQL Database 資料表的逐步指示，請參閱[複製精靈教學課程](data-factory-copy-data-wizard-tutorial.md)。
>
>

精靈從一開始就針對巨量資料而設計，並且支援多樣化資料和物件類型。 您可以撰寫 Data Factory 管線，移動數百個資料夾、檔案或資料表。 精靈支援自動資料預覽、結構描述擷取和對應，以及資料篩選。

## <a name="automatic-data-preview"></a>自動資料預覽
您可以預覽一部分選取的資料來源以驗證資料是否是您要複製的資料。 此外，如果來源資料位於文字檔案中，複製精靈會自動剖析該文字檔，以了解資料列和資料行的分隔符號，以及結構描述。

![檔案格式設定](./media/data-factory-copy-wizard/file-format-settings.png)

## <a name="schema-capture-and-mapping"></a>結構描述擷取和對應
在某些情況下，輸入資料的結構描述可能不符合輸出資料的結構描述。 在此案例中，您需要將來源結構描述的資料行對應到目的地結構描述的資料行。

> [!TIP]
> 從 SQL Server 或 Azure SQL Database 中將資料複製到 Azure SQL 資料倉儲時，如果目的地存放區中沒有資料表，Data Factory 支援使用來源的結構描述來自動建立資料表。 深入了解[使用 Azure Data Factory 從 Azure SQL 資料倉儲來回移動資料](./data-factory-azure-sql-data-warehouse-connector.md)。
>

請使用下拉式清單選取來源結構描述的資料行來對應至目的地結構描述中的資料行。 複製精靈會嘗試了解資料行對應的模式， 並且對其餘的資料行套用相同的模式，所以您不需要個別選取每個資料行就能完成結構描述對應。 如果想要，還是可以使用下拉式清單逐一對應資料行來覆寫這些對應。 您對應越多資料行，模式就會變得越精確。 複製精靈會持續更新模式，直到達到您想要達成的資料行對應的正確模式。     

![結構描述對應](./media/data-factory-copy-wizard/schema-mapping.png)

## <a name="filtering-data"></a>篩選資料
您可以篩選來源資料，只選取需要複製到接收資料存放區的資料。 篩選可減少要複製到接收資料存放區的資料，因此可增強複製作業的輸送量。 它提供一種彈性方式，藉由使用 SQL 查詢語言來篩選關聯式資料庫中的資料，或使用 [Data Factory 函式與變數](data-factory-functions-variables.md)來篩選 Azure Blob 資料夾中的檔案。   

### <a name="filtering-of-data-in-a-database"></a>篩選資料庫中的資料
下列螢幕擷取畫面顯示使用 `Text.Format` 函式和 `WindowStart` 變數的 SQL 查詢。

![驗證運算式](./media/data-factory-copy-wizard/validate-expressions.png)

### <a name="filtering-of-data-in-an-azure-blob-folder"></a>篩選 Azure Blob 資料夾中的資料
您可以在資料夾路徑中使用變數，以複製在執行階段根據[系統變數](data-factory-functions-variables.md#data-factory-system-variables)決定之資料夾內的資料。 支援的變數包括︰**{year}**、**{month}**、**{day}**、**{hour}**、**{minute}** 及 **{custom}**。 例如︰inputfolder/{year}/{month}/{day}。

假設您的輸入資料夾格式如下︰

    2016/03/01/01
    2016/03/01/02
    2016/03/01/03
    ...

按一下 [檔案或資料夾] 的 [瀏覽] 按鈕、瀏覽至其中一個資料夾 (例如 2016->03->01->02)，然後按一下 [選擇]。 您應該會在文字方塊中看到 `2016/03/01/02`。 現在，將 **2016** 取代為 **{year}**、**03** 取代為 **{month}**、**01** 取代為 **{day}**，以及 **02** 取代為 **{hour}**，然後按 **Tab** 鍵。 您應該會看到選取這四個變數之格式的下拉式清單︰

![使用系統變數](./media/data-factory-copy-wizard/blob-standard-variables-in-folder-path.png)   

如下列螢幕擷取畫面所示，您也可以使用 **custom** 變數和任何[支援的格式字串](https://msdn.microsoft.com/library/8kb3ddd4.aspx)。 若要選取具有該結構的資料夾，請先使用 [瀏覽] 按鈕。 然後將值取代為 **{custom}**，並按 **Tab** 鍵來查看可輸入格式字串的文字方塊。     

![使用自訂變數](./media/data-factory-copy-wizard/blob-custom-variables-in-folder-path.png)

## <a name="scheduling-options"></a>排程選項
您可以執行複製作業一次，或按照排程 (每小時、每日等) 執行。 這兩個選項均適用於跨環境的廣闊連接器，包括內部部署、雲端和本機桌面複本。

一次性複製作業只能進行一次從來源到目的地的資料移動。 它適用於任何規模和任何支援格式的資料。 排程複製可讓您依照指定的週期複製資料。 在設定排程複製時，您可以使用豐富的設定 (如重試、逾時和警示)。

![排程屬性](./media/data-factory-copy-wizard/scheduling-properties.png)

## <a name="next-steps"></a>後續步驟
如需使用 Data Factory 複製精靈建立含複製活動之管線的快速逐步解說，請參閱 [教學課程：使用 Data Factory 複製精靈建立具有複製活動的管線](data-factory-copy-data-wizard-tutorial.md)。
