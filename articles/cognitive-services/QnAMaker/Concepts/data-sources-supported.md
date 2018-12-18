---
title: 支援的資料來源 - QnA Maker
titleSuffix: Azure Cognitive Services
description: QnA Maker 會自動從半結構化的內容 (例如，儲存為網頁、PDF 檔案或 MS Word DOC 檔的常見問題集、產品手冊、指導方針、支援文件和原則) 擷取問答組合。 內容也可以從結構化 QnA 內容檔案新增至知識庫。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/25/2018
ms.author: tulasim
ms.openlocfilehash: 982bcbb9060a3f29000de2a0487b61dc58e24f6e
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855454"
---
# <a name="data-sources-for-qna-maker-content"></a>QnA Maker 內容的資料來源

QnA Maker 會自動從半結構化的內容 (例如，儲存為網頁、PDF 檔案或 MS Word DOC 檔的常見問題集、產品手冊、指導方針、支援文件和原則) 擷取問答組合。 內容也可以從結構化 QnA 內容檔案新增至知識庫。 

下表摘要說明 QnA Maker 所支援的內容類型和檔案格式。

|來源類型|內容類型| 範例|
|--|--|--|
|URL|常見問題集 (一般，含有區段或含有主題首頁)|[一般的常見問題集](https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs)、[具有連結的常見問題集](https://www.microsoft.com/software-download/faq)、[具有主題首頁的常見問題集](https://support.microsoft.com/products/windows?os=windows-10)|
|PDF/DOC|常見問題集、產品手冊、摺頁冊、白皮書、傳單原則、支援指南、結構化 QnA 等等。|[Structured QnA.doc](https://qnamakerstore.blob.core.windows.net/qnamakerdata/docs/Bot%20Service%20Sample%20FAQ.docx)、[Sample Product Manual.pdf](https://download.microsoft.com/download/2/9/B/29B20383-302C-4517-A006-B0186F04BE28/surface-pro-4-user-guide-EN.pdf)、[Sample semi-structured.doc](https://qnamakerstore.blob.core.windows.net/qnamakerdata/docs/Manage%20Azure%20Blob%20Storage.docx)、[Sample white paper.pdf](https://azure.microsoft.com/mediahandler/files/resourcefiles/azure-stack-wortmann-bring-the-power-of-the-public-cloud-into-your-data-center/Azure_Stack_Wortmann_Bring_the_Power_of_the_Public_Cloud_into_Your_Data_Center.pdf)|
|Excel|結構化 QnA 檔案 (包括 RTF、HTML 支援)|[Sample QnA FAQ.xls](https://qnamakerstore.blob.core.windows.net/qnamakerdata/docs/QnA%20Maker%20Sample%20FAQ.xlsx)|
|TXT/TSV|結構化 QnA 檔案|[Sample chit-chat.tsv](https://raw.githubusercontent.com/Microsoft/BotBuilder-PersonalityChat/master/CSharp/Datasets/Queries_Responses_Friendly_QnAMaker.tsv)|

## <a name="faq-urls"></a>常見問題集 URL

QnA Maker 可支援 3 種不同形式的常見問題集網頁：一般常見問題集網頁、具有連結的常見問題集網頁、具有主題首頁的常見問題集網頁。

### <a name="plain-faq-pages"></a>一般的常見問題集頁面

這是最常用的常見問題集頁面類型，在此處，解答會在相同的頁面中緊接在問題後面。 

以下是一般常見問題集網頁的範例：

![一般的常見問題集頁面](../media/qnamaker-concepts-datasources/plain-faq.png) 

 
### <a name="faq-pages-with-links"></a>具有連結的常見問題集網頁 

在這種常見問題集網頁中，問題會彙總在一起並連結至解答 (位於相同網頁的不同區段，或位於不同網頁)。

以下是連結位於相同網頁區段的常見問題集網頁範例：

 ![區段連結常見問題集頁面](../media/qnamaker-concepts-datasources/sectionlink-faq.png) 


### <a name="faq-pages-with-a-topics-homepage"></a>具有主題首頁的常見問題集網頁

這種常見問題集會有首頁，首頁會有主題，且每個主題都是其相關 QnA 在不同網頁的連結。 在此，QnA Maker 會搜耙所有連結頁面，以擷取對應的問題和解答。

以下是主題首頁上有不同網頁常見問題集區段連結的常見問題集網頁範例。 

 ![深層連結常見問題集頁面](../media/qnamaker-concepts-datasources/topics-faq.png) 


## <a name="pdf-doc-files"></a>PDF/DOC 檔

QnA Maker 可以處理 PDF 或 DOC 檔中的半結構化內容，並將它轉換成 QnA。 可以完好擷取的良好檔案，其內容會組織成某種結構化形式，並會以妥善定義的區段來表示。 區段可以再分成子區段或子主題。 文件若具有包含階層式標題的明確結構，擷取將會有最佳效果。

QnA Maker 可根據視覺線索 (例如字型大小、字型樣式、編號、色彩等等) 來識別檔案中的區段和子區段以及關聯性。半結構化的 PDF 或 DOC 檔可以是手冊、常見問題集、指導方針、原則、摺頁冊、傳單和其他諸多類型的檔案。 以下是這些檔案類型的一些範例。

### <a name="product-manuals"></a>產品手冊

手冊通常式隨附於產品的指引內容。 它可協助使用者設定、使用、維護產品，並對其問題進行疑難排解。 當 QnA Maker 處理手冊時，它會將標題和子標題擷取為問題，並將後續的內容擷取為解答。 請參閱[這裡](https://download.microsoft.com/download/2/9/B/29B20383-302C-4517-A006-B0186F04BE28/surface-pro-4-user-guide-EN.pdf)的範例。

以下是具有索引頁面與階層式內容的手冊範例

 ![產品手冊範例](../media/qnamaker-concepts-datasources/product-manual.png) 

> [!NOTE]
> 手冊若具有目錄及/或索引頁面，和包含階層式標題的明確結構，擷取將會有最佳效果。

### <a name="brochures-guidelines-papers-and-other-files"></a>摺頁冊、指導方針、白皮書及其他檔案

許多其他類型的文件經過處理後也能產生問答組合，但前提是這些文件要有明確的結構和版面配置。 這些文件包括：摺頁冊、指導方針、報告、白皮書、科學論文、原則、書籍等。請參閱[這裡](https://qnamakerstore.blob.core.windows.net/qnamakerdata/docs/Manage%20Azure%20Blob%20Storage.docx)的範例。

以下是沒有索引的半結構化文件範例：

 ![Azure Blob 儲存體的半結構化文件](../media/qnamaker-concepts-datasources/semi-structured-doc.png) 

### <a name="structured-qna-document"></a>結構化 QnA 文件

DOC 檔中結構化問答的格式，會採用逐行交替列出問題和解答的形式，每行一個問題，接著的下一行則是其解答，如下所示： 

```
Question1

Answer1

Question2

Answer2
```

以下是結構化 QnA Word 文件的範例：

 ![結構化 QnA 文件](../media/qnamaker-concepts-datasources/structured-qna-doc.png) 

## <a name="structured-txt-tsv-and-xls-files"></a>結構化的 TXT、TSV 和 XLS 檔案

採用結構化 .txt、.tsv 或 .xls 檔案形式的 QnA 也可上傳至 QnA Maker，以便建立或擴大知識庫。  這些檔案可以是純文字，也可以具有 RTF 或 HTML 的內容。 

| 問題  | Answer  | 中繼資料                |
|-----------|---------|-------------------------|
| 問題 1 | 解答 1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| 問題 2 | 解答 2 |      `Key:Value`           |

來源檔案中的任何其他資料行都會被忽略。

以下是具有 HTML 內容的結構化 QnA .xls 檔案範例：

 ![結構化 QnA Excel](../media/qnamaker-concepts-datasources/structured-qna-xls.png)

## <a name="structured-data-format-through-import"></a>透過匯入的結構化資料格式

匯入知識庫後，將會取代現有知識庫的內容。 匯入時必須要有包含資料來源資訊的結構化 .tsv 檔案。 這項資訊可協助 QnA Maker 將問答組分組，並將其歸於特定資料來源。

| 問題  | Answer  | 來源| 中繼資料                |
|-----------|---------|----|---------------------|
| 問題 1 | 解答 1 | Url1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| 問題 2 | 解答 2 | 編輯|    `Key:Value`       |

## <a name="editorially-add-to-knowledge-base"></a>透過編輯來新增至知識庫

如果您沒有既有的內容可填入知識庫，您也可以透過編輯將 QnA 新增至 QnA Maker 知識庫中。 在[這裡](../How-To/edit-knowledge-base.md)了解如何更新知識庫。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [設定 QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)

## <a name="see-also"></a>另請參閱 

[QnA Maker 概觀](../Overview/overview.md)
