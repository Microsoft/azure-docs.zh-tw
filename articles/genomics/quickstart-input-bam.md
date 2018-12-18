---
title: 快速入門：使用 BAM 檔案輸入來提交工作流程 | Microsoft Genomics
titleSuffix: Azure
description: 此快速入門假設您已安裝 msgen 用戶端，並已透過服務成功地執行範例資料。
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: quickstart
ms.date: 12/07/2017
ms.openlocfilehash: 91bc13dbf024df20ac23ca0ff83a73ed246dce28
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45729499"
---
# <a name="submit-a-workflow-using-a-bam-file-input"></a>使用 BAM 檔案輸入提交工作流程

此快速入門示範如何在您的輸入檔案是單一的 BAM 檔案時，將工作流程提交到 Microsoft Genomics 服務。 此主題假設您已安裝並執行 `msgen` 用戶端，且熟悉如何使用 Azure 儲存體。 如果您已使用提供的範例資料成功地提交工作流程，即準備好繼續進行此快速入門。 

## <a name="set-up-upload-your-bam-file-to-azure-storage"></a>設定：將您的 BAM 檔案上傳至 Azure 儲存體
假設您有單一 BAM 檔案，reads.bam，且您已將它上傳至 Azure 中的儲存體帳戶 myaccount 作為 **https://<span></span>myaccount.blob.core<span></span>.windows<span></span>.net<span></span>/inputs/reads<span></span>.bam<span></span>**。 您具有 API URL 以及存取金鑰。 您需要 **https://<span></span>myaccount.blob.core<span></span>.windows<span></span>.net<span></span>/outputs<span></span>** 中的輸出。



## <a name="submit-your-job-to-the-msgen-client"></a>將作業提交到 `msgen` 用戶端 


以下是您必須提供給 `msgen` 用戶端的引數最低組數，為了清楚起見，已新增分行符號：

若為 Windows：

```
msgen submit ^
  --api-url-base <Genomics API URL> ^
  --access-key <Genomics access key> ^
  --process-args R=b37m1 ^
  --input-storage-account-name myaccount ^
  --input-storage-account-key <storage access key to "myaccount"> ^
  --input-storage-account-container inputs ^
  --input-blob-name-1 reads.bam ^
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
  --input-blob-name-1 reads.bam \
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
input_blob_name_1:                reads.bam
output_storage_account_name:      myaccount
output_storage_account_key:       <storage access key to "myaccount">
output_storage_account_container: outputs
```

使用這個引動過程來提交 `config.txt` 檔案：`msgen submit -f config.txt`

## <a name="next-steps"></a>後續步驟
在此文章中，您已將 BAM 檔案上傳到 Azure 儲存體，並已透過 `msgen` Python 用戶端將工作流程提交到 Microsoft Genomics 服務。 如需有關工作流程提交以及可與 Microsoft Genomics 服務搭配使用之其他命令的詳細資訊，請參閱我們的[常見問題集](frequently-asked-questions-genomics.md)。 
