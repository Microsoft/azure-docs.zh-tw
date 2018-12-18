---
title: 如何使用 iOS 的物件 (Blob) 儲存體 - Azure | Microsoft Docs
description: 使用 Azure Blob 儲存體 (物件儲存體) 在雲端中儲存非結構化資料。
services: storage
author: michaelhauss
ms.service: storage
ms.devlang: objective-c
ms.topic: article
ms.date: 03/21/2018
ms.author: michaelhauss
ms.component: blobs
ms.openlocfilehash: 35cfa36aba8042bcc753c1005c8fd8dbcdb70e79
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397667"
---
# <a name="how-to-use-blob-storage-from-ios"></a>如何使用 iOS 的 Blob 儲存體

本文示範如何使用 Microsoft Azure Blob 儲存體執行一般案例。 這些範例均以 Objective-C 撰寫，並使用 [Azure Storage Client Library for iOS](https://github.com/Azure/azure-storage-ios)。 所涵蓋的案例包括「上傳、列出、下載」及「刪除」Blob。 如需 Blob 的詳細資訊，請參閱 [後續步驟](#next-steps) 一節。 您也可以下載 [範例應用程式](https://github.com/Azure/azure-storage-ios/tree/master/BlobSample) ，以快速查看在 iOS 應用程式中使用 Azure 儲存體的方法。

## <a name="what-is-blob-storage"></a>什麼是 Blob 儲存體？

[!INCLUDE [storage-blob-concepts-include](../../../includes/storage-blob-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="import-the-azure-storage-ios-library-into-your-application"></a>將 Azure 儲存體 iOS 程式庫匯入您的應用程式中
您可以透過使用 [Azure 儲存體 CocoaPod](https://cocoapods.org/pods/AZSClient) 或匯入 **Framework** 檔，將 Azure 儲存體 iOS 程式庫匯入至您的應用程式。 CocoaPod 是建議的方式，因為它讓程式庫整合更容易，不過從架構匯入檔案較不干擾您的現有專案。

若要使用此程式庫，您需要：
- iOS 8+
- Xcode 7+

## <a name="cocoapod"></a>CocoaPod
1. 如果您還沒有這麼做，請開啟終端機視窗，並執行下列命令，在電腦上 [安裝 CocoaPods](https://guides.cocoapods.org/using/getting-started.html#toc_3)
    
    ```shell   
    sudo gem install cocoapods
    ```

2. 接下來，在專案目錄 (包含您 .xcodeproj 檔案的目錄) 中建立稱為 _Podfile_ 的檔案 (無副檔名)。 將下列內容加入至 _Podfile_ 並儲存。

    ```ruby
    platform :ios, '8.0'

    target 'TargetName' do
      pod 'AZSClient'
    end
    ```

3. 在終端機視窗中，巡覽至專案目錄並執行下列命令

    ```shell    
    pod install
    ```

4. 如果您已在 Xcode 中開啟 .xcodeproj，請將它關閉。 在您的專案目錄中開啟新建立的專案檔案 (副檔名為 .xcworkspace)。 這是您從現在開始會使用的檔案。

## <a name="framework"></a>Framework
程式庫的另一個用法，是手動建立架構︰

1. 首先，請下載或複製 [azure-storage-ios repo](https://github.com/azure/azure-storage-ios)。
2. 移至 *azure-storage-ios* -> *Lib* -> *Azure 儲存體用戶端程式庫*，然後在 Xcode 中開啟 `AZSClient.xcodeproj`。
3. 在 Xcode 左上方，將作用中配置從「Azure 儲存體用戶端程式庫」變更為「架構」。
4. 建置專案 (⌘ + B)。 這會在您的桌面上建立 `AZSClient.framework` 檔案。

您可以透過下列方式，將 Framework 檔案匯入至您的應用程式：

1. 建立新的專案，或在 Xcode 中開啟現有的專案。
2. 將 `AZSClient.framework` 拖曳到您的 Xcode 專案導覽器。
3. 選取 [必要時複製項目]，然後按一下 [完成]。
4. 按一下左側導覽列中的專案，然後按一下專案編輯器頂端的 [一般]  索引標籤。
5. 在 [連結的架構和程式庫]  區段下，按一下 [新增] 按鈕 (+)。
6. 在已提供的程式庫清單中搜尋 `libxml2.2.tbd` ，並將它新增至您的專案。

## <a name="import-the-library"></a>匯入程式庫 
```objc
// Include the following import statement to use blob APIs.
#import <AZSClient/AZSClient.h>
```

如果您使用 Swift，必須建立橋接標頭並在標頭匯入 <AZSClient/AZSClient.h>︰

1. 建立標頭檔 `Bridging-Header.h`，並加入上述匯入陳述式。
2. 移至 [建置設定] 索引標籤，搜尋[OBJECTIVE-C 橋接標頭]。
3. 按兩下 [OBJECTIVE-C 橋接標頭] 欄位，並將此路徑加入標頭檔︰`ProjectName/Bridging-Header.h`
4. 建立專案 (⌘ + B) 以確認 Xcode 已收取橋接標頭。
5. 直接在任何 Swift 檔案中開始使用程式庫，就不需要匯入陳述式。

[!INCLUDE [storage-mobile-authentication-guidance](../../../includes/storage-mobile-authentication-guidance.md)]

## <a name="asynchronous-operations"></a>非同步作業
> [!NOTE]
> 所有對服務執行要求的方法，都是非同步作業。 在程式碼範例中，您會發現這些方法具有完成處理常式。 完成處理常式內的程式碼會在要求完成 **之後** 執行。 完成處理常式後面的程式碼會在進行要求 **期間** 執行。
> 
> 

## <a name="create-a-container"></a>建立容器
儲存體 Blob 中的每個 Blob 必須位於一個容器中。 下列範例說明如何在您的儲存體帳戶中建立名為 *newcontainer*的容器 (如果還不存在)。 為您的容器選擇名稱時，請留意上面提到的命名規則。

```objc
-(void)createContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"newcontainer"];

    // Create container in your Storage account if the container doesn't already exist
    [blobContainer createContainerIfNotExistsWithCompletionHandler:^(NSError *error, BOOL exists) {
        if (error){
            NSLog(@"Error in creating container.");
        }
    }];
}
```

您可以查看 [Microsoft Azure 儲存體總管](http://storageexplorer.com) ，並驗證 *newcontainer* 位於儲存體帳戶的容器清單中，以確認運作正常。

## <a name="set-container-permissions"></a>設定容器權限
依預設會針對 [私人]  存取設定容器的權限。 不過，容器會提供幾個不同的容器存取選項：

* **私人**：只有帳戶擁有者可以讀取容器和 Blob 資料。
* **Blob**：您可以透過匿名要求讀取此容器內的 Blob 資料，但您無法使用容器資料。 用戶端無法透過匿名要求列舉容器內的 Blob。
* **容器**：可以透過匿名要求讀取容器和 Blob 資料。 用戶端可以透過匿名要求列舉容器內的 Blob，但無法列舉儲存體帳戶內的容器。

下列範例說明如何建立具有 [容器]  存取權限，而允許網際網路上的所有使用者進行公用、唯讀存取的容器：

```objc
-(void)createContainerWithPublicAccess{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Create container in your Storage account if the container doesn't already exist
    [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists){
        if (error){
            NSLog(@"Error in creating container.");
        }
    }];
}
```

## <a name="upload-a-blob-into-a-container"></a>將 Blob 上傳至容器
如 [Blob 服務概念](#blob-service-concepts) 一節中所述，Blob 儲存體提供三種不同類型的 Blob：區塊 Blob、附加 Blob 和分頁 Blob。 Azure 儲存體 iOS 程式庫支援三種類型的 Blob。 在大部分情況下，建議使用區塊 Blob 的類型。

下列範例說明如何從 NSString 上傳區塊 Blob。 如果此容器中有相同名稱的 Blob 存在，此 Blob 的內容將會被覆寫。

```objc
-(void)uploadBlobToContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists)
        {
            if (error){
                NSLog(@"Error in creating container.");
            }
            else{
                // Create a local blob object
                AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

                // Upload blob to Storage
                [blockBlob uploadFromText:@"This text will be uploaded to Blob Storage." completionHandler:^(NSError *error) {
                    if (error){
                        NSLog(@"Error in creating blob.");
                    }
                }];
            }
        }];
}
```

您可以查看 [Microsoft Azure 儲存體總管](http://storageexplorer.com)，並驗證容器 *containerpublic* 包含 Blob *sampleblob*，以確認運作正常。 在此範例中我們使用公用容器，所以您也可以移至 Blob URI 來確認應用程式運作是否正常：

    https://nameofyourstorageaccount.blob.core.windows.net/containerpublic/sampleblob

除了從 NSString 上傳區塊 Blob 以外，NSData、NSInputStream 或本機檔案也有類似的方法。

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob
下列範例說明如何列出容器中的所有 Blob。 執行此作業時，請留意下列參數：     

* **continuationToken** - 接續權杖表示列出作業應從哪裡開始。 如果未提供權杖，則會從頭列出 Blob。 可以列出任何數目的 Blob，從零到設定的最大值皆可。 即使此方法傳回零個結果，如果 `results.continuationToken` 不是 nil，則可能會有服務上的更多 Blob 未列出。
* **prefix** -您可以指定要用於 Blob 列出作業的前置詞。 只有開頭為此前置詞的 Blob 才會列出。
* **useFlatBlobListing** - 如 [命名與參考容器和 Blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata) 一節中所述，雖然 Blob 服務是一般儲存體配置，雖然您可以路徑資訊為 Blob 命名，以建立虛擬階層。 不過目前不支援非一般列出。 不久後將有此功能。 目前，此值應為 **YES**。
* **blobListingDetails** - 您可以指定列出 Blob 時所要包含的項目
  * _AZSBlobListingDetailsNone_：僅列出已認可的 Blob，且不傳回 Blob 中繼資料。
  * _AZSBlobListingDetailsSnapshots_︰列出已認可的 Blob 和 Blob 快照。
  * _AZSBlobListingDetailsMetadata_：為清單中傳回的每個 Blob 擷取 Blob 中繼資料。
  * _AZSBlobListingDetailsUncommittedBlobs_︰列出已認可和未認可的 Blob。
  * _AZSBlobListingDetailsCopy_：在清單中包含複製屬性。
  * _AZSBlobListingDetailsAll_：列出所有可用的已認可 Blob、未認可的 Blob 和快照，並傳回這些 Blob 的所有中繼資料和複製狀態。
* **maxResults** - 要為此作業傳回的結果數目上限。 使用 -1 則不會設定限制。
* **completionHandler** - 要執行的程式碼區塊與列出作業的結果。

在此範例中，將會在每次傳回接續權杖時使用 Helper 方法以遞迴方式呼叫列出 Blob 方法。

```objc
-(void)listBlobsInContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    //List all blobs in container
    [self listBlobsInContainerHelper:blobContainer continuationToken:nil prefix:nil blobListingDetails:AZSBlobListingDetailsAll maxResults:-1 completionHandler:^(NSError *error) {
        if (error != nil){
            NSLog(@"Error in creating container.");
        }
    }];
}

//List blobs helper method
-(void)listBlobsInContainerHelper:(AZSCloudBlobContainer *)container continuationToken:(AZSContinuationToken *)continuationToken prefix:(NSString *)prefix blobListingDetails:(AZSBlobListingDetails)blobListingDetails maxResults:(NSUInteger)maxResults completionHandler:(void (^)(NSError *))completionHandler
{
    [container listBlobsSegmentedWithContinuationToken:continuationToken prefix:prefix useFlatBlobListing:YES blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:^(NSError *error, AZSBlobResultSegment *results) {
        if (error)
        {
            completionHandler(error);
        }
        else
        {
            for (int i = 0; i < results.blobs.count; i++) {
                NSLog(@"%@",[(AZSCloudBlockBlob *)results.blobs[i] blobName]);
            }
            if (results.continuationToken)
            {
                [self listBlobsInContainerHelper:container continuationToken:results.continuationToken prefix:prefix blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:completionHandler];
            }
            else
            {
                completionHandler(nil);
            }
        }
    }];
}
```

## <a name="download-a-blob"></a>下載 Blob
下列範例說明如何將 Blob 下載到 NSString 物件。

```objc
-(void)downloadBlobToString{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Create a local blob object
    AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

    // Download blob
    [blockBlob downloadToTextWithCompletionHandler:^(NSError *error, NSString *text) {
        if (error) {
            NSLog(@"Error in downloading blob");
        }
        else{
            NSLog(@"%@",text);
        }
    }];
}
```

## <a name="delete-a-blob"></a>刪除 Blob
下列範例說明如何刪除 Blob。

```objc
-(void)deleteBlob{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Create a local blob object
    AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob1"];

    // Delete blob
    [blockBlob deleteWithCompletionHandler:^(NSError *error) {
        if (error) {
            NSLog(@"Error in deleting blob.");
        }
    }];
}
```

## <a name="delete-a-blob-container"></a>刪除 Blob 容器
下列範例說明如何刪除容器。

```objc
-(void)deleteContainer{
    NSError *accountCreationError;

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here" error:&accountCreationError];

    if(accountCreationError){
        NSLog(@"Error in creating account.");
    }

    // Create a blob service client object.
    AZSCloudBlobClient *blobClient = [account getBlobClient];

    // Create a local container object.
    AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

    // Delete container
    [blobContainer deleteContainerIfExistsWithCompletionHandler:^(NSError *error, BOOL success) {
        if(error){
            NSLog(@"Error in deleting container");
        }
    }];
}
```

## <a name="next-steps"></a>後續步驟
您現在已經了解如何從 iOS 中使用 Blob 儲存體，請參考下列連結以深入了解 iOS 程式庫和儲存體服務。

* [Azure Storage Client Library for iOS](https://github.com/azure/azure-storage-ios)
* [Azure 儲存體 iOS 參考文件](http://azure.github.io/azure-storage-ios/)
* [Azure 儲存體服務 REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)
* [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage)

如果您有關於此程式庫的問題，歡迎您貼文到我們的 [MSDN Azure 論壇](http://social.msdn.microsoft.com/Forums/windowsazure/home?forum=windowsazuredata)或 [Stack Overflow](http://stackoverflow.com/questions/tagged/windows-azure-storage+or+windows-azure-storage+or+azure-storage-blobs+or+azure-storage-tables+or+azure-table-storage+or+windows-azure-queues+or+azure-storage-queues+or+azure-storage-emulator+or+azure-storage-files)。
如果您有 Azure 儲存體功能方面的建議，請貼文到 [Azure 儲存體意見反應](https://feedback.azure.com/forums/217298-storage/)。

