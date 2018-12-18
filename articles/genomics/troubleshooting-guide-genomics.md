---
title: Microsoft Genomics 疑難排解指南
titleSuffix: Azure
description: 深入了解疑難排解策略
keywords: 疑難排解, 錯誤, 偵錯
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: article
ms.date: 07/18/2018
ms.openlocfilehash: bd946f84023345c68a01a48a4dc310b7afb68397
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45735381"
---
# <a name="troubleshooting-guide-for-microsoft-genomics"></a>Microsoft Genomics 疑難排解指南
本概觀描述解決使用 Microsoft Genomics 服務時常見問題的策略。 如需一般常見問題集，請參閱[常見問題](frequently-asked-questions-genomics.md)。 


## <a name="how-do-i-check-my-job-status"></a>如何查看我的作業狀態？
您可以從命令列呼叫 `msgen status` 來查看工作流程的狀態，如下所示。 

```
msgen status -u URL -k KEY -w ID [-f CONFIG] 
```

有三個必要引數：
* URL - API 的基底 URI
* KEY - 您 Genomics 帳戶的存取金鑰。 
* ID - 工作流程識別碼

若要尋找您的 URL 和 KEY，請前往 Azure 入口網站，然後開啟您的 Genomics 帳戶頁面。 在 [管理] 標題之下，選擇 [存取金鑰]。 您可以看到 API URL 以及存取金鑰。

您也可以包含設定檔的路徑，而非直接輸入 URL 和 KEY。 請注意，如果您在命令列和設定檔中包含這些引數，命令列引數將具有優先權。 

呼叫 `msgen status` 之後，系統將會顯示簡單易懂的訊息，描述工作流程成功或提供作業失敗的原因。 


## <a name="get-more-information-about-my-workflow-status"></a>取得工作流程狀態的詳細資訊

有關作業未能成功的原因，若要取得詳細資訊，可以瀏覽在工作流程期間產生的記錄檔。 在您的輸出容器中，您應該會看到 `[youroutputfilename].logs.zip` 資料夾。  將此資料夾解壓縮後，您會看到下列項目：

* outputFileList.txt - 工作流程期間所產生輸出檔案的清單
* standarderror.txt - 此檔案是空白的。
* standardoutput.txt - 包含工作流程的最上層記錄。 
* GATK 記錄檔 - `logs` 資料夾中的其他所有檔案

若要判斷為什麼工作流程未成功，很適合從 `standardoutput.txt` 開始，因為它包含更多工作流程的底層資訊。 

## <a name="common-issues-and-how-to-resolve-them"></a>常見問題與解決方式
本節簡短說明常見問題與它們解決方式。

### <a name="fastq-files-are-unmatched"></a>Fastq 檔案不相符
Fastq 檔案之間的差異應該只有位於範例識別碼尾端的 /1 或 /2。 如果您不小心提交不相符的 FASTQ 檔案，當您呼叫 `msgen status` 時，便可能會看到下列錯誤訊息。
* `Encountered an unmatched read`
* `Error reading a FASTQ file, make sure the input files are valid and paired correctly` 

若要解決此問題，請檢閱提交到工作流程的 fastq 檔案是否為相符的集合。 


### <a name="error-uploading-bam-file-output-blob-already-exists-and-the-overwrite-option-was-set-to-false"></a>上傳 .bam 檔案時發生錯誤。 輸出 blob 已存在，且覆寫選項已設為 False。
如果您看到下列錯誤訊息，表示輸出資料夾已含有相同名稱的輸出檔案：`Error uploading .bam file. Output blob already exists and the overwrite option was set to False`。  請刪除現有的輸出檔案，或開啟設定檔中的覆寫選項。 然後重新提交您的工作流程。

### <a name="when-to-contact-microsoft-genomics-support"></a>連絡 Microsoft Genomics 支援服務的時機
如果您看到下列錯誤訊息，表示發生內部錯誤。 

* `Error locating input files on worker machine`
* `Process management failure`

請嘗試重新提交您的工作流程。 如果持續發生作業失敗，或是有任何其他問題，請從 Azure 入口網站連絡 Microsoft Genomics 支援服務。 您可以在[此處](file-support-ticket-genomics.md)找到如何提交支援要求的其他資訊。

## <a name="next-steps"></a>後續步驟
在本文中，您已學會如何對 Microsoft Genomics 服務的常見問題進行疑難排解並嘗試解決。 如需詳細資訊和更多一般常見問題集，請參閱[常見問題](frequently-asked-questions-genomics.md)。 
