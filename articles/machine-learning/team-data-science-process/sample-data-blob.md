---
title: Azure Blob 儲存體中的取樣資料 | Microsoft Docs
description: Azure blob 儲存體中的取樣資料
services: machine-learning,storage
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: e8d9ad2c-86c5-43d6-80b8-d355b5c0dccf
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: ffb92df0e0727d02985ec26f61739cdc759d0f93
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837662"
---
# <a name="heading"></a>Azure blob 儲存體中的取樣資料
本文件說明為儲存於 Azure blob 儲存體中的資料進行取樣的方法，您可以利用程式設計方式加以下載，然後使用以 Python 撰寫的程序進行取樣。

以下**功能表**所連結的主題會說明如何從各種不同儲存體環境進行資料取樣。 

[!INCLUDE [cap-sample-data-selector](../../../includes/cap-sample-data-selector.md)]

**為何要對您的資料進行取樣？**
如果您規劃分析的資料集很龐大，通常最好是對資料進行向下取樣，將資料縮減為更小但具代表性且更容易管理的大小。 這有助於資料了解、探索和功能工程。 它在 Cortana 分析程序中扮演的角色是能夠快速建立資料處理函式與機器學習服務模型的原型。

這個取樣工作是 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/)中的一個步驟。

## <a name="download-and-down-sample-data"></a>下載和降低取樣資料
1. 從下列 Python 程式碼範例中，使用 Blob 服務，從 Azure Blob 儲存體下載資料。 
   
        from azure.storage.blob import BlobService
        import tables
   
        STORAGEACCOUNTNAME= <storage_account_name>
        STORAGEACCOUNTKEY= <storage_account_key>
        LOCALFILENAME= <local_file_name>        
        CONTAINERNAME= <container_name>
        BLOBNAME= <blob_name>
   
        #download from blob
        t1=time.time()
        blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
        blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
        t2=time.time()
        print(("It takes %s seconds to download "+blobname) % (t2 - t1))

2. 從上述下載的檔案中將資料讀取至 Pandas 資料框架。
   
        import pandas as pd
   
        #directly ready from file on disk
        dataframe_blobdata = pd.read_csv(LOCALFILE)

3. 使用 `numpy` 的 `random.choice` 以進行降低取樣資料，如下所示：
   
        # A 1 percent sample
        sample_ratio = 0.01 
        sample_size = np.round(dataframe_blobdata.shape[0] * sample_ratio)
        sample_rows = np.random.choice(dataframe_blobdata.index.values, sample_size)
        dataframe_blobdata_sample = dataframe_blobdata.ix[sample_rows]

現在您可以使用上述具有 1% 樣本的資料框架，進行進一步探索和功能產生。

## <a name="heading"></a>將資料上傳並將其讀入 Azure Machine Learning
您可以使用下列程式碼範例，對資料進行向下取樣，並直接在 Azure Machine Learning 中使用它：

1. 將資料框架寫入本機檔案
   
        dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)

2. 使用下列程式碼範例，將本機檔案上傳至 Azure Blob：
   
        from azure.storage.blob import BlobService
        import tables
   
        STORAGEACCOUNTNAME= <storage_account_name>
        LOCALFILENAME= <local_file_name>
        STORAGEACCOUNTKEY= <storage_account_key>
        CONTAINERNAME= <container_name>
        BLOBNAME= <blob_name>
   
        output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
        localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory
   
        try:
   
        #perform upload
        output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)
   
        except:            
            print ("Something went wrong with uploading to the blob:"+ BLOBNAME)

3. 使用 Azure Machine Learning [匯入資料](https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/) 從 Azure Blob 讀取資料，如下圖所示：

![讀取器 Blob](./media/sample-data-blob/reader_blob.png)

