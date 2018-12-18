---
title: 限制和界限 - QnA Maker
titleSuffix: Azure Cognitive Services
description: QnA Maker 的完整限制清單。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 76024266a8ae3eb03e910e140184a07ae363e881
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49384937"
---
# <a name="qna-maker-limits"></a>QnA Maker 限制
QnA Maker 的完整限制清單。

## <a name="knowledge-bases"></a>知識庫

* 以 [Azure 搜尋服務層限制](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity)為基礎的知識庫數目上限

|**Azure 搜尋服務層** | **免費** | **基本** |**S1** | **S2**| **S3** |**S3 HD**|
|---|---|---|---|---|---|----|
|允許的已發佈知識庫數目上限 (索引數上限 -- 1 (保留以供測試)|2|14|49|199|199|2999|

## <a name="extraction-limits"></a>擷取限制
* 可擷取的檔案數目上限與檔案大小上限：請參閱 [QnAMaker 定價](https://azure.microsoft.com/pricing/details/cognitive-services/qna-maker/)
* 可以從常見問題集 HTML 頁面搜耙，以便擷取 QnAs 的深層連結數目上限：20

## <a name="metadata-limits"></a>中繼資料限制
* 每個知識庫的中繼資料欄位數目上限 (以 [Azure 搜尋服務層限制](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity)為基礎)

|**Azure 搜尋服務層** | **免費** | **基本** |**S1** | **S2**| **S3** |**S3 HD**|
|---|---|---|---|---|---|----|
|每項 QnA Maker 服務的中繼資料欄位數上限 (橫跨所有的知識庫)|1000|100*|1000|1000|1000|1000|

## <a name="knowledge-base-content-limits"></a>知識庫的內容限制
知識庫內容的整體限制：
* 解答文字的長度：250000
* 問題文字的長度：1000
* 中繼資料索引鍵/值文字的長度：100
* 中繼資料名稱的支援字元：英文字母、數字和 _  
* 中繼資料值的支援字元：: 和 | 以外的所有字元 
* 檔案名稱的長度：200
* 支援的檔案格式：".tsv"、".pdf"、".txt"、".docx"、".xlsx"。
* 替代問題數目上限：100
* 問題解答組的數目上限：取決於所選的 [Azure 搜尋服務層](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity#document-limits) 

## <a name="create-knowledge-base-call-limits"></a>建立知識庫呼叫限制：
這些代表每個建立知識庫動作的限制；也就是，按一下 [建立知識庫] 或呼叫 CreateKnowledgeBase API。
* 每個解答的替代問題數目上限：100
* URL 數目上限︰10
* 檔案數目上限︰10

## <a name="update-knowledge-base-call-limits"></a>更新知識庫呼叫限制
這些代表每個更新動作的限制；也就是，按一下 [儲存及訓練] 或呼叫 UpdateKnowledgeBase API。
* 每個來源名稱的長度：300
* 新增或刪除的替代問題數目上限：100
* 新增或刪除的中繼資料欄位數目上限：10
* 可重新整理的 URL 數目上限：5
