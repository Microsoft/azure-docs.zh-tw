---
title: 複製資料工具 - Azure Data Factory | Microsoft Docs
description: 提供有關 Azure Data Factory UI 中複製資料工具的資訊
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/18/2018
ms.author: yexu
ms.openlocfilehash: 8941ec26cef5e3dc2f17faf0d7eb843b76f8926f
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "36331832"
---
# <a name="copy-data-tool-in-azure-data-factory"></a>Azure Data Factory 中的複製資料工具
Azure Data Factory「複製資料」工具把將資料內嵌到 Data Lake 的程序 (通常是端對端資料整合案例中的第一步) 簡化並最佳化。  它可節省時間，尤其是當您第一次使用 Azure Data Factory 從資料來源內嵌資料的時候。 使用此工具的一些優點包括：

- 使用 Azure Data Factory「複製資料」工具時，您不需要了解已連結的服務、資料集、管線、活動及觸發程序的 Data Factory 定義。 
- 在將資料載入至 Data Lake 方面，「複製資料」工具的流程是相當直覺式的。 此工具會自動建立所有必要的 Data Factory 資源，以將資料從所選來源資料存放區複製到所選目的地/接收資料存放區。 
- 「複製資料」工具可協助您驗證撰寫時所內嵌的資料，這有助於避免在開始時於本身發生任何潛在錯誤。
- 如果您需要實作複雜的商務邏輯來將資料載入至 Data Lake，您仍然可以使用 Data Factory UI 中的個別活動撰寫功能來編輯「複製資料」工具所建立的 Data Factory 資源。 

下表提供有關何時使用「複製資料」工具與何時使用 Data Factory UI 中個別活動撰寫功能的比較指引： 

| 複製資料工具 | 個別活動 (複製活動) 撰寫功能 |
| -------------- | -------------------------------------- |
| 您想要在無須了解 Azure Data Factory 實體 (已連結的服務、資料集、管線等) 的情況下，輕鬆建置資料載入工作。 | 您想要實作複雜且彈性的邏輯來將資料載入 Data Lake 中。 |
| 您想要將大量資料構件快速載入至 Data Lake 中。 | 您想要將「複製」活動與後續用於清理或處理資料的活動加以鏈結。 |

若要啟動「複製資料」工具，請按一下您資料處理站首頁上的 [複製資料] 圖格。

![開始使用頁面 -「複製資料」工具連結](./media/copy-data-tool/get-started-page.png)


## <a name="intuitive-flow-for-loading-data-into-a-data-lake"></a>將資料載入至 Data Lake 的直覺式流程
此工具讓您只需幾分鐘，即可輕鬆地將資料從各種來源移至目的地：  

1. 設定**來源**的設定。
2. 設定**目的地**的設定。 
3. 設定複製作業的**進階設定**，例如資料行對應、效能設定及容錯設定。 
4. 指定資料載入工作的**排程**。 
5. 檢閱所要建立之 Data Factory 實體的**摘要**。 
6. 視需要**編輯**管線，以更新複製活動的設定。 

 此工具從一開始設計時就已將巨量資料納入構想，可支援多樣化的資料和物件類型。 您可以使用它來移動數百個資料夾、檔案或資料表。 此工具支援自動資料預覽、結構描述擷取及自動對應，也支援資料篩選。

![複製資料工具](./media/copy-data-tool/copy-data-tool.png)

## <a name="automatic-data-preview"></a>自動資料預覽
您可以預覽來自所選來源資料存放區的部分資料，這可讓您驗證所要複製的資料。 此外，如果來源資料位於文字檔中，「複製資料」工具就會剖析該文字檔，以自動偵測資料列和資料行的分隔符號，以及結構描述。

![檔案設定](./media/copy-data-tool/file-format-settings.png)

偵測後：

![偵測到的檔案設定和預覽](./media/copy-data-tool/after-detection.png)

## <a name="schema-capture-and-automatic-mapping"></a>結構描述擷取和自動對應
在許多情況下，資料來源的結構描述可能會與資料目的地的結構描述不同。 在此案例中，您需要將來源結構描述的資料行對應到目的地結構描述的資料行。

當您在來源與目的地存放區之間對應資料行時，「複製資料」工具會監視並學習您的行為。 在您從來源資料存放區選取一個或一些資料行，並將其對應至目的地結構描述之後，「複製資料」工具就會開始分析您從兩邊選取之資料行配對的模式。 然後，它會將相同的模式套用至其餘資料行。 因此，在按幾下滑鼠之後，您就會看到所有資料行都已以您想要的方式對應至目的地。  如果您不滿意「複製資料」工具所提供的資料行對應選擇，您可以忽略它，然後繼續手動對應資料行。 在此同時，「複製資料」工具會不斷地學習並更新模式，並在最終達到您想要達成的正確資料行對應模式。 

> [!NOTE]
> 將資料從 SQL Server 或 Azure SQL Database 複製到「Azure SQL 資料倉儲」時，如果目的地存放區中沒有資料表，「複製資料」工具支援使用來源結構描述來自動建立資料表。 

## <a name="filter-data"></a>篩選資料
您可以篩選來源資料，只選取需要複製到接收資料存放區的資料。 篩選可減少要複製到接收資料存放區的資料，因此可增強複製作業的輸送量。 「複製資料」工具提供一種彈性方式，可藉由使用 SQL 查詢語言來篩選關聯式資料庫中的資料，或篩選 Azure Blob 資料夾中的檔案。 

### <a name="filter-data-in-a-database"></a>篩選資料庫中的資料
以下螢幕擷取畫面顯示一個用來篩選資料的 SQL 查詢。

![篩選資料庫中的資料](./media/copy-data-tool/filter-data-in-database.png)

### <a name="filter-data-in-an-azure-blob-folder"></a>篩選 Azure Blob 資料夾中的資料
您可以在資料夾路徑中使用變數，以從資料夾中複製資料。 支援的變數包括︰**{year}****{month}****{day}****{hour}** 及 **{minute}**。 例如︰inputfolder/{year}/{month}/{day}。 

假設您的輸入資料夾格式如下︰ 

```
2016/03/01/01
2016/03/01/02
2016/03/01/03
...
```

按一下 [檔案或資料夾] 的 [瀏覽] 按鈕、瀏覽至其中一個資料夾 (例如 2016->03->01->02)，然後按一下 [選擇]。 您應該會在文字方塊中看到 2016/03/01/02。 

接著，將 **2016** 取代為 **{year}**、**03** 取代為 **{month}**、**01** 取代為 **{day}**，以及 **02** 取代為 **{hour}**，然後按 **Tab** 鍵。 您應該會看到選取這四個變數之格式的下拉式清單︰

![篩選檔案或資料夾](./media/copy-data-tool/filter-file-or-folder.png)

「複製資料」工具會在建立管線時，產生含有運算式、函式及可用來代表 {year}、{month}、{day}、{hour} 及 {minute} 之系統變數的參數。 如需詳細資訊，請參閱[如何讀取或寫入分割的資料](how-to-read-write-partitioned-data.md)一文。

## <a name="scheduling-options"></a>排程選項
您可以執行複製作業一次，或按照排程 (每小時、每日等) 執行。 這些選項可用於各個不同環境的連接器，包括內部部署、雲端及本機桌面。 

一次性複製作業只能進行一次從來源到目的地的資料移動。 它適用於任何規模和任何支援格式的資料。 排程複製可讓您依照指定的週期來複製資料。 在設定排程複製時，您可以使用豐富的設定 (如重試、逾時和警示)。

![排程選項](./media/copy-data-tool/scheduling-options.png)


## <a name="next-steps"></a>後續步驟
請試試下列使用「複製資料」工具的教學課程：

- [快速入門：使用複製資料工具來建立資料處理站](quickstart-create-data-factory-copy-data-tool.md)
- [教學課程：使用複製資料工具在 Azure 中複製資料](tutorial-copy-data-tool.md) 
- [教學課程：使用複製資料工具將內部部署資料複製到 Azure](tutorial-hybrid-copy-data-tool.md)
