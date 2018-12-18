---
title: 處理使用進階分析的 Azure Blob 資料 | Microsoft Docs
description: 處理 Azure Blob 儲存體中的資料。
services: machine-learning,storage
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: d8a59078-91d3-4440-b85c-430363c3f4d1
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 3daf86f59a84f8c442581160142dcf806173b626
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836597"
---
# <a name="heading"></a>處理使用進階分析的 Azure Blob 資料
本文件涵蓋探索資料以及從 Azure Blob 儲存體中儲存的資料產生功能的說明。 

## <a name="load-the-data-into-a-pandas-data-frame"></a>將資料載入至 Pandas 資料框架
若要探索和操作資料集，必須從 Blob 來源將資料集下載至本機檔案，然後將其載入 Pandas 資料框架中。 以下是此程序的遵循步驟：

1. 使用 Blob 服務，透過下列 Python 程式碼範例，從 Azure Blob 下載資料。 使用您的特定值來取代下列程式碼中的變數： 
   
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
2. 從下載的檔案中，將資料讀取至 Pandas 資料框架。
   
        #LOCALFILE is the file path    
        dataframe_blobdata = pd.read_csv(LOCALFILE)

現在您已經準備好探索資料並在此資料集上產生功能。

## <a name="blob-dataexploration"></a>資料探索
以下是數個可使用 Pandas 探索資料的範例方式：

1. 檢查資料列和資料行的數目 
   
        print 'the size of the data is: %d rows and  %d columns' % dataframe_blobdata.shape
2. 檢查資料集中的前幾個或最後幾個資料列，如下所示：
   
        dataframe_blobdata.head(10)
   
        dataframe_blobdata.tail(10)
3. 檢查使用下列程式碼範例匯入之每個資料行的資料類型
   
        for col in dataframe_blobdata.columns:
            print dataframe_blobdata[col].name, ':\t', dataframe_blobdata[col].dtype
4. 檢查資料集中資料行的基本統計資料，如下所示
   
        dataframe_blobdata.describe()
5. 查看每個資料行值的項目數，如下所示
   
        dataframe_blobdata['<column_name>'].value_counts()
6. 使用下列程式碼範例，來計算每個資料行中的遺漏值與實際項目數
   
        miss_num = dataframe_blobdata.shape[0] - dataframe_blobdata.count()
        print miss_num
7. 如果您在資料的特定資料行中有遺漏值，則可卸除它們，如下所示：
   
     dataframe_blobdata_noNA = dataframe_blobdata.dropna()   dataframe_blobdata_noNA.shape
   
   取代遺漏值的另一種方式是使用模式函式：
   
     dataframe_blobdata_mode = dataframe_blobdata.fillna({'<column_name>':dataframe_blobdata['<column_name>'].mode()[0]})        
8. 使用變動數目的分類收納組來建立長條圖，以繪製變數的分佈    
   
        dataframe_blobdata['<column_name>'].value_counts().plot(kind='bar')
   
        np.log(dataframe_blobdata['<column_name>']+1).hist(bins=50)
9. 使用散佈圖或使用內建的相互關聯函式，來查看變數之間的相互關聯
   
        #relationship between column_a and column_b using scatter plot
        plt.scatter(dataframe_blobdata['<column_a>'], dataframe_blobdata['<column_b>'])
   
        #correlation between column_a and column_b
        dataframe_blobdata[['<column_a>', '<column_b>']].corr()

## <a name="blob-featuregen"></a>功能產生
我們可以使用 Python 來產生功能，如下所示：

### <a name="blob-countfeature"></a>以指標值為基礎的功能產生
類別功能可使用如下的方式來建立：

1. 檢查類別資料行的分佈：
   
        dataframe_blobdata['<categorical_column>'].value_counts()
2. 產生每個資料行值的指標值
   
        #generate the indicator column
        dataframe_blobdata_identity = pd.get_dummies(dataframe_blobdata['<categorical_column>'], prefix='<categorical_column>_identity')
3. 將指標資料行與原始資料框架聯結在一起 
   
            #Join the dummy variables back to the original data frame
            dataframe_blobdata_with_identity = dataframe_blobdata.join(dataframe_blobdata_identity)
4. 移除原始變數本身：
   
        #Remove the original column rate_code in df1_with_dummy
        dataframe_blobdata_with_identity.drop('<categorical_column>', axis=1, inplace=True)

### <a name="blob-binningfeature"></a>分類收納功能產生
若要產生分類收納功能，我們可使用如下的方式繼續進行：

1. 新增一系列的資料行，以分類收納數值資料行
   
        bins = [0, 1, 2, 4, 10, 40]
        dataframe_blobdata_bin_id = pd.cut(dataframe_blobdata['<numeric_column>'], bins)
2. 將分類收納轉換為一系列的布林值變數
   
        dataframe_blobdata_bin_bool = pd.get_dummies(dataframe_blobdata_bin_id, prefix='<numeric_column>')
3. 最後，將虛擬變數新增回原始資料框架中
   
        dataframe_blobdata_with_bin_bool = dataframe_blobdata.join(dataframe_blobdata_bin_bool)    

## <a name="sql-featuregen"></a>將資料寫回 Azure Blob 並在 AzureMachine Learning 中取用
在您探索資料和建立必要功能後，可以上傳資料 (取樣性或功能性) 至 Azure Blob，並在 Azure Machine Learning 中透過下列步驟取用資料：請注意，您也可以在 Azure Machine Learning Studio 中建立其他功能。 

1. 將資料框架寫入本機檔案中
   
        dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
2. 將資料上傳至 Azure Blob，如下所示：
   
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
            print ("Something went wrong with uploading blob:"+BLOBNAME)
3. 現在您可以使用 Azure Machine Learning [匯入資料][import-data]模組從 Blob 讀取資料，如以下畫面所示：

![讀取器 Blob][1]

[1]: ./media/data-blob/reader_blob.png


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/

