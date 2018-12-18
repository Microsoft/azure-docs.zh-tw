---
title: 快速入門：使用多個輸入來提交工作流程 | Microsoft Genomics
titleSuffix: Azure
description: 此快速入門假設您已安裝 msgen 用戶端，並已透過服務成功地執行範例資料。
services: genomics
author: grhuynh
manager: cgronlund
ms.author: grhuynh
ms.topic: quickstart
ms.date: 02/05/2018
ms.openlocfilehash: 1007d81a73ce9f183f997354188e534274b2fe95
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730357"
---
# <a name="submit-a-workflow-using-multiple-inputs-from-the-same-sample"></a>使用相同範例中的多個輸入來提交工作流程

此快速入門示範如何在您的輸入檔案是**來自相同範例**的多個 FASTQ 或 BAM 檔案時，將工作流程提交到 Microsoft Genomics 服務。 例如，如果您已在排序器上的多個通道中執行**相同範例**，則排序器可能會為每個通道輸出一組 FASTQ 檔案。 比起在校對和識別變體之前串連 FASTQ 檔案，您可以直接將這些輸入全部提交到 `msgen` 用戶端。 `msgen` 用戶端的輸出會是**一組**檔案，包括 .bam、.bai 和 .vcf 檔案。 

不過，請記住，您**無法**在相同的提交中將 FASTQ 和 BAM 檔案混合。 此外，您**無法**提交多個個體的 FASTQ 或 BAM 檔案。 

此文章假設您已安裝並執行 `msgen` 用戶端，且熟悉如何使用 Azure 儲存體。 如果您已使用提供的範例資料成功地提交工作流程，即準備好繼續進行此快速入門。 


## <a name="multiple-bam-files"></a>多個 BAM 檔案

### <a name="upload-your-input-files-to-azure-storage"></a>將您的輸入檔案上傳至 Azure 儲存體
假設您以多個 BAM 檔案作為輸入，reads.bam、additional_reads.bam 和 yet_more_reads.bam，且您已它們上傳至 Azure 中的儲存體帳戶 myaccount。 您具有 API URL 以及存取金鑰。 您需要 **https://<span></span>myaccount.blob.core<span></span>.windows<span></span>.net<span></span>/outputs<span></span>** 中的輸出。


### <a name="submit-your-job-to-the-msgen-client"></a>將作業提交到 `msgen` 用戶端 

您可以提交多個 BAM 檔案，方法是將其所有的名稱都傳遞給 --input-blob-name-1 引數。 請注意，所有檔案都必須來自相同的範例，但其順序並不重要。 下一節會詳細說明在 Windows 中、Unix 中及使用設定檔從命令列執行的範例提交。 為了清楚起見，已新增分行符號：


若為 Windows：

```
msgen submit ^
  --api-url-base <Genomics API URL> ^
  --access-key <Genomics access key> ^
  --process-args R=b37m1 ^
  --input-storage-account-name myaccount ^
  --input-storage-account-key <storage access key to "myaccount"> ^
  --input-storage-account-container inputs ^
  --input-blob-name-1 reads.bam additional_reads.bam yet_more_reads.bam ^
  --output-storage-account-name myaccount ^
  --output-storage-account-key <storage access key to "myaccount"> ^
  --output-storage-account-container outputs
```


若為 Unix

```
msgen submit \
  --api-url-base <Genomics API URL> \
  --access-key <Genomics access key> \
  --process-args R=b37m1 \
  --input-storage-account-name myaccount \
  --input-storage-account-key <storage access key to "myaccount"> \
  --input-storage-account-container inputs \
  --input-blob-name-1 reads.bam additional_reads.bam yet_more_reads.bam \
  --output-storage-account-name myaccount \
  --output-storage-account-key <storage access key to "myaccount"> \
  --output-storage-account-container outputs
```


如果您偏好使用設定檔，以下是它可能包含的內容：

``` config.txt
api_url_base:                     <Genomics API URL>
access_key:                       <Genomics access key>
process_args:                     R=b37m1
input_storage_account_name:       myaccount
input_storage_account_key:        <storage access key to "myaccount">
input_storage_account_container:  inputs
input_blob_name_1:                reads.bam additional_reads.bam yet_more_reads.bam
output_storage_account_name:      myaccount
output_storage_account_key:       <storage access key to "myaccount">
output_storage_account_container: outputs
```

使用這個引動過程來提交 `config.txt` 檔案：`msgen submit -f config.txt`


## <a name="multiple-paired-fastq-files"></a>多個配對的 FASTQ 檔案

### <a name="upload-your-input-files-to-azure-storage"></a>將您的輸入檔案上傳至 Azure 儲存體
假設您以多個配對的 FASTQ 檔案作為輸入，reads_1.fq.gz 與 reads_2.fq.gz、additional_reads_1.fq.gz 與 additional_reads_2.fq.gz 以及 yet_more_reads_1.fq.gz 與 yet_more_reads_2.fq.gz。 您已將它們上傳至 Azure 中的儲存體帳戶 myaccount，且您有 API URL 和存取金鑰。 您需要 **https://<span></span>myaccount.blob.core<span></span>.windows<span></span>.net<span></span>/outputs<span></span>** 中的輸出。


### <a name="submit-your-job-to-the-msgen-client"></a>將作業提交到 `msgen` 用戶端 

配對的 FASTQ 檔案不只必須來自相同的範例，還需要一起進行處理。  作為引數傳遞至 --input-blob-name-1 和 --input-blob-name-2 時，檔案名稱的順序很重要。 

下一節會詳細說明在 Windows 中、Unix 中及使用設定檔從命令列執行的範例提交。 為了清楚起見，已新增分行符號：


若為 Windows：

```
msgen submit ^
  --api-url-base <Genomics API URL> ^
  --access-key <Genomics access key> ^
  --process-args R=b37m1 ^
  --input-storage-account-name myaccount ^
  --input-storage-account-key <storage access key to "myaccount"> ^
  --input-storage-account-container inputs ^
  --input-blob-name-1 reads_1.fastq.gz additional_reads_1.fastq.gz yet_more_reads_1.fastq.gz ^
  --input-blob-name-2 reads_2.fastq.gz additional_reads_2.fastq.gz yet_more_reads_2.fastq.gz ^
  --output-storage-account-name myaccount ^
  --output-storage-account-key <storage access key to "myaccount"> ^
  --output-storage-account-container outputs
```


若為 Unix：

```
msgen submit \
  --api-url-base <Genomics API URL> \
  --access-key <Genomics access key> \
  --process-args R=b37m1 \
  --input-storage-account-name myaccount \
  --input-storage-account-key <storage access key to "myaccount"> \
  --input-storage-account-container inputs \
  --input-blob-name-1 reads_1.fastq.gz additional_reads_1.fastq.gz yet_more_reads_1.fastq.gz \
  --input-blob-name-2 reads_2.fastq.gz additional_reads_2.fastq.gz yet_more_reads_2.fastq.gz \
  --output-storage-account-name myaccount \
  --output-storage-account-key <storage access key to "myaccount"> \
  --output-storage-account-container outputs
```


如果您偏好使用設定檔，以下是它可能包含的內容：

```
api_url_base:                     <Genomics API URL>
access_key:                       <Genomics access key>
process_args:                     R=b37m1
input_storage_account_name:       myaccount
input_storage_account_key:        <storage access key to "myaccount">
input_storage_account_container:  inputs
input_blob_name_1:                reads_1.fq.gz additional_reads_1.fastq.gz yet_more_reads_1.fastq.gz
input_blob_name_2:                reads_2.fq.gz additional_reads_2.fastq.gz yet_more_reads_2.fastq.gz
output_storage_account_name:      myaccount
output_storage_account_key:       <storage access key to "myaccount">
output_storage_account_container: outputs
```

使用這個引動過程來提交 `config.txt` 檔案：`msgen submit -f config.txt`

## <a name="next-steps"></a>後續步驟
在此文章中，您已將多個 BAM 檔案或配對的 FASTQ 檔案上傳到 Azure 儲存體，並已透過 `msgen` python 用戶端將工作流程提交到 Microsoft Genomics 服務。 如需有關工作流程提交以及可與 Microsoft Genomics 服務搭配使用之其他命令的詳細資訊，請參閱[常見問題集](frequently-asked-questions-genomics.md)。 