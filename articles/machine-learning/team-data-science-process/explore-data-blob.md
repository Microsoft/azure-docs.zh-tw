---
title: 使用 Pandas 瀏覽 Azure blob 儲存體中的資料 | Microsoft Docs
description: 如何使用 Pandas 瀏覽儲存在 Azure blob 容器的資料。
services: machine-learning,storage
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: feaa9e54-01e0-48c8-a917-1eba0f9d9ec7
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: deguhath
ms.openlocfilehash: c727df985f3285f5def9bfdc249ee27b4d748a01
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837043"
---
# <a name="explore-data-in-azure-blob-storage-with-pandas"></a>使用 Pandas 瀏覽 Azure blob 儲存體中的資料
本文件涵蓋如何使用 [Pandas](http://pandas.pydata.org/) Python 封裝瀏覽儲存在 Azure blob 容器的資料。

下列 **功能表** 所連結的主題會說明如何從各種不同的儲存體環境使用工具來瀏覽資料。 此工作是 [資料科學程序]()中的一個步驟。

[!INCLUDE [cap-explore-data-selector](../../../includes/cap-explore-data-selector.md)]

## <a name="prerequisites"></a>先決條件
本文假設您已經：

* 建立 Azure 儲存體帳戶。 如需指示，請參閱[建立 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md#create-a-storage-account)
* 將您的資料儲存在 Azure blob 儲存體帳戶。 如需指示，請參閱 [從 Azure 儲存體來回移動資料](../../storage/common/storage-moving-data.md)

## <a name="load-the-data-into-a-pandas-dataframe"></a>將資料載入 Pandas 資料框架
若要探索和操作資料集，必須先從 Blob 來源將資料集下載至本機檔案，然後將其載入 Pandas 資料框架。 以下是此程序的遵循步驟：

1. 使用 blob 服務，透過下列 Python 程式碼範例，從 Azure blob 下載資料。 使用您的特定值來取代下列程式碼中的變數： 
   
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

## <a name="blob-dataexploration"></a>使用 Pandas 的資料探索範例
以下是數個可使用 Pandas 探索資料的範例方式：

1. 檢查 **資料列和資料行的數目** 
   
        print 'the size of the data is: %d rows and  %d columns' % dataframe_blobdata.shape
2. **檢查**資料集中的前幾個或最後幾個**資料列**：
   
        dataframe_blobdata.head(10)
   
        dataframe_blobdata.tail(10)
3. 檢查使用下列程式碼範例匯入之每個資料行的 **資料類型**
   
        for col in dataframe_blobdata.columns:
            print dataframe_blobdata[col].name, ':\t', dataframe_blobdata[col].dtype
4. 檢查資料集中資料行的 **基本統計資料** ，如下所示
   
        dataframe_blobdata.describe()
5. 查看每個資料行值的項目數，如下所示
   
        dataframe_blobdata['<column_name>'].value_counts()
6. **計算遺漏值** 與實際項目數
   
        miss_num = dataframe_blobdata.shape[0] - dataframe_blobdata.count()
        print miss_num
7. 如果您在資料的特定資料行中有 **遺漏值** ，則可卸除它們，如下所示：
   
     dataframe_blobdata_noNA = dataframe_blobdata.dropna()   dataframe_blobdata_noNA.shape
   
   取代遺漏值的另一種方式是使用模式函式：
   
     dataframe_blobdata_mode = dataframe_blobdata.fillna({'<column_name>':dataframe_blobdata['<column_name>'].mode()[0]})        
8. 使用變動數目的分類收納組來建立 **長條圖** ，以繪製變數的分佈    
   
        dataframe_blobdata['<column_name>'].value_counts().plot(kind='bar')
   
        np.log(dataframe_blobdata['<column_name>']+1).hist(bins=50)
9. 使用散佈圖或使用內建的相互關聯函式，來查看變數之間的 **相互關聯**
   
        #relationship between column_a and column_b using scatter plot
        plt.scatter(dataframe_blobdata['<column_a>'], dataframe_blobdata['<column_b>'])
   
        #correlation between column_a and column_b
        dataframe_blobdata[['<column_a>', '<column_b>']].corr()

