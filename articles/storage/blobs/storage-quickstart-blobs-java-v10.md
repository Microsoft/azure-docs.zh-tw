---
title: Azure 快速入門：使用 Java Storage SDK V10 在物件儲存體中建立 Blob | Microsoft Docs
description: 在本快速入門中，您會使用 Java Storage SDK 在物件 (Azure Blob) 儲存體中建立容器、上傳檔案、列出物件，並進行下載。
services: storage
author: roygara
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 07/02/2018
ms.author: rogarana
ms.openlocfilehash: dfd04aa0c8f314327afaefa67f1c63b1ff605e9b
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387203"
---
# <a name="quickstart-upload-download-and-list-blobs-by-using-the-java-storage-sdk-v10-preview"></a>快速入門：使用 Java Storage SDK V10 (預覽) 上傳、下載及列出 Blob

在本快速入門中，您將了解如何使用新的 Java Storage SDK 在 Azure Blob 儲存體容器中上傳、下載及列出區塊 Blob。 新的 Java SDK 會使用以 RxJava 執行非同步作業的回應式程式設計模型。 深入了解[適用於 Java VM 的 RxJava 回應式擴充功能](https://github.com/ReactiveX/RxJava)。 

## <a name="prerequisites"></a>必要條件

安裝及設定下列應用程式：

* 從命令列運作的 [Maven](http://maven.apache.org/download.cgi)，或您偏好的任何 Java 整合式開發環境
* [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="download-the-sample-application"></a>下載範例應用程式

本快速入門使用的[應用程式範例](https://github.com/Azure-Samples/storage-blobs-java-v10-quickstart)是基本的主控台應用程式。 

使用 [git](https://git-scm.com/) 將應用程式的複本下載至您的開發環境。

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-v10-quickstart.git
```

此命令會將存放庫複製到本機的 git 資料夾。

當專案匯入完成後，請開啟 **Quickstart.java** (位於 **src/main/java/quickstart**)。

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串
使用此解決方案時，您必須安全地儲存您的儲存體帳戶名稱和金鑰。 請將其儲存在執行範例的電腦本機的環境變數中。 根據您的作業系統，依照 Linux 或 Windows 範例來建立環境變數。

### <a name="linux-example"></a>Linux 範例

```
export AZURE_STORAGE_ACCOUNT="<youraccountname>"
export AZURE_STORAGE_ACCESS_KEY="<youraccountkey>"
```

### <a name="windows-example"></a>Windows 範例

```
setx AZURE_STORAGE_ACCOUNT "<youraccountname>"
setx AZURE_STORAGE_ACCESS_KEY "<youraccountkey>"
```

## <a name="run-the-sample"></a>執行範例

對於 Windows 使用者，此範例會在預設目錄 **AppData\Local\Temp** 中建立測試檔案。 然後，它會提示您執行下列步驟：

1. 輸入命令，將測試檔案上傳至 Azure Blob 儲存體。
2. 列出容器中的 Blob。
3. 以新名稱下載先前上傳的檔案，以便比較新舊檔案。 

如果您想要在命令列上使用 Maven 執行範例，請開啟殼層，然後瀏覽至複製目錄內的 **storage-blobs-java-v10-quickstart**。 然後輸入 `mvn compile exec:java`。

此範例顯示您在 Windows 上執行應用程式時的輸出。

```
Created quickstart container
Enter a command
(P)utBlob | (L)istBlobs | (G)etBlob | (D)eleteBlobs | (E)xitSample
# Enter a command :
P
Uploading the sample file into the container: https://<storageaccount>.blob.core.windows.net/quickstart
# Enter a command :
L
Listing blobs in the container: https://<storageaccount>.blob.core.windows.net/quickstart
Blob name: SampleBlob.txt
# Enter a command :
G
Get the blob: https://<storageaccount>.blob.core.windows.net/quickstart/SampleBlob.txt
The blob was downloaded to C:\Users\<useraccount>\AppData\Local\Temp\downloadedFile13097087873115855761.txt
# Enter a command :
D
Delete the blob: https://<storageaccount>.blob.core.windows.net/quickstart/SampleBlob.txt

# Enter a command :
>> Blob deleted: https://<storageaccount>.blob.core.windows.net/quickstart/SampleBlob.txt
E
Cleaning up the sample and exiting!
```

您可以控制範例，因此請輸入命令，使其執行程式碼。 輸入會區分大小寫。

您也可以使用 [Azure 儲存體總管](http://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 之類的工具來檢視 Blob 儲存體中的檔案。 Azure 儲存體總管是免費的跨平台工具，可讓您存取儲存體帳戶資訊。 

驗證檔案。 然後，選取 **E**，再選取 **Enter** 鍵，以完成示範並刪除測試檔案。 現在您已知道這個範例的功用，請開啟 **Quickstart.java** 檔案查看程式碼。 

## <a name="understand-the-sample-code"></a>了解範例程式碼

以下幾節將逐步說明範例程式碼，讓您了解其運作方式。

### <a name="get-references-to-the-storage-objects"></a>取得儲存體物件的參考

首先，您必須建立用來存取和管理 Blob 儲存體的物件參考。 這些物件互為建置基礎。 各物件分別由清單中的下一個物件所使用。

1. 建立指向儲存體帳戶的 **StorageURL** 物件執行個體。

    * [StorageURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._storage_u_r_l?view=azure-java-preview) 物件代表您的儲存體帳戶。 您可以用它來產生新的管線。 
    * 管線是一組原則，用來處理授權、記錄和重試機制的要求和回應。 如需詳細資訊，請參閱 [HTTP 管線](https://github.com/Azure/azure-storage-java/wiki/Azure-Storage-Java-V10-Overview#url-types--http-pipeline)。  
    * 藉由使用管線，建立 [ServiceURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._service_u_r_l?view=azure-java-preview) 物件的執行個體。
    * 藉由使用 **ServiceURL** 物件，建立 [ContainerURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l?view=azure-java-preview) 的執行個體。
    * **ContainerURL** 是在 Blob 容器上執行作業的必要項目。

2. 建立 **ContainerURL** 物件的執行個體，代表您要存取的容器。 容器會用來組織 Blob，就像在電腦上用資料夾來組織檔案一樣。

    * **ContainerURL** 可提供容器服務的存取點。 
    * 您可以使用 [ContainerURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l?view=azure-java-preview) 來建立 [BlobURL](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._blob_u_r_l?view=azure-java-preview) 物件的執行個體。
    * **BlobURL** 是建立 Blob 的必要項目。

3. 建立 **BlobURL** 物件的執行個體，指向您所需的特定 Blob。 

> [!IMPORTANT]
> 容器名稱必須是小寫字母。 如需關於容器和 Blob 名稱的詳細資訊，請參閱[命名和參考容器、Blob 及中繼資料](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

### <a name="create-a-container"></a>建立容器 

在本節中，您會建立 **ContainerURL** 的執行個體。 此外，您會建立新容器。 範例中的容器名為 **quickstart**。 

此範例使用 [containerURL.create](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l.create?view=azure-java-preview)，因此您可以在每次執行範例時建立新的容器。 或者，您可以事先建立容器，就不需要在程式碼中建立。

```java
// Create a ServiceURL to call the Blob service. We will also use this to construct the ContainerURL
SharedKeyCredentials creds = new SharedKeyCredentials(accountName, accountKey);
// We are using a default pipeline here, you can learn more about it at https://github.com/Azure/azure-storage-java/wiki/Azure-Storage-Java-V10-Overview
final ServiceURL serviceURL = new ServiceURL(new URL("http://" + accountName + ".blob.core.windows.net"), StorageURL.createPipeline(creds, new PipelineOptions()));

// Let's create a container using a blocking call to Azure Storage
// If container exists, we'll catch and continue
containerURL = serviceURL.createContainerURL("quickstart");

try {
    ContainersCreateResponse response = containerURL.create(null, null).blockingGet();
    System.out.println("Container Create Response was " + response.statusCode());
} catch (RestException e){
    if (e instanceof RestException && ((RestException)e).response().statusCode() != 409) {
        throw e;
    } else {
        System.out.println("quickstart container already exists, resuming...");
    }
}
```

### <a name="upload-blobs-to-the-container"></a>將 Blob 上傳到容器

Blob 儲存體支援區塊 Blob、附加 Blob 和分頁 Blob。 最常使用區塊 Blob。 我們將其用於本快速入門中。 

1. 若要將檔案上傳至 Blob，請取得目標容器中的 Blob 參考。 
2. 在取得 Blob 參考之後，您可以使用下列其中一個 API 將檔案上傳至該參考：

    * 低階 API。 範例包括 **BlockBlobURL** 執行個體中的 [BlockBlobURL.upload](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._block_blob_u_r_l.upload?view=azure-java-preview) (也稱為 PutBlob) 和 [BlockBlobURL.stageBlock](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._block_blob_u_r_l.stageblock?view=azure-java-preview#com_microsoft_azure_storage_blob__block_blob_u_r_l_stageBlock_String_Flowable_ByteBuffer__long_LeaseAccessConditions_) (也稱為 PutBLock)。 

    * 高階 API 提供於 [TransferManager 類別](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._transfer_manager?view=azure-java-preview)中。 例如，[TransferManager.uploadFileToBlockBlob](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._transfer_manager.uploadfiletoblockblob?view=azure-java-preview) 方法。 

    此作業會建立 Blob (如果尚不存在)。 若已存在，則會覆寫 Blob。

範例程式碼會建立一個用於上傳和下載的本機檔案。 它會將要上傳的檔案儲存為 **sourceFile**，並將 Blob 的 URL 儲存在 **Blob** 中。 下列範例會將檔案上傳到名為 **quickstart** 的容器。

```java
static void uploadFile(BlockBlobURL blob, File sourceFile) throws IOException {
    
    FileChannel fileChannel = FileChannel.open(sourceFile.toPath());
            
    // Uploading a file to the blobURL using the high-level methods available in TransferManager class
    // Alternatively call the Upload/StageBlock low-level methods from BlockBlobURL type
    TransferManager.uploadFileToBlockBlob(fileChannel, blob, 8*1024*1024, null)
        .subscribe(response-> {
            System.out.println("Completed upload request.");
            System.out.println(response.response().statusCode());
        });
}
```

區塊 Blob 可以是任何類型的文字或二進位檔案。 分頁 Blob 主要用於備份 IaaS VM 所用的 VHD 檔案。 「附加 Blob」可用來將資料附加到結尾，通常用於記錄。 儲存在 Blob 儲存體中的大部分物件都是區塊 Blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

您可以使用 [ContainerURL.listBlobsFlatSegment](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l.listblobsflatsegment?view=azure-java-preview) 取得容器中的物件清單。 如果在容器中要列出更多物件，此方法可藉由接續標記 (下一個標記) 一次傳回多達 5,000 個物件。 請建立 Helper 函式，並使其在先前的 **listBlobsFlatSegment** 回應中有下一個標記的情況下重複呼叫本身。

```java
static void listBlobs(ContainerURL containerURL) {
    // Each ContainerURL.listBlobsFlatSegment call return up to maxResults (maxResults=10 passed into ListBlobOptions below).
    // To list all Blobs, we are creating a helper static method called listAllBlobs, 
    // and calling it after the initial listBlobsFlatSegment call
    ListBlobsOptions options = new ListBlobsOptions(null, null, 10);

    containerURL.listBlobsFlatSegment(null, options)
        .flatMap(containersListBlobFlatSegmentResponse -> 
            listAllBlobs(containerURL, containersListBlobFlatSegmentResponse))    
                .subscribe(response-> {
                    System.out.println("Completed list blobs request.");
                    System.out.println(response.statusCode());
                });
}

private static Single <ContainersListBlobFlatSegmentResponse> listAllBlobs(ContainerURL url, ContainersListBlobFlatSegmentResponse response) {                
    // Process the blobs returned in this result segment (if the segment is empty, blobs() will be null.
    if (response.body().blobs() != null) {
        for (Blob b : response.body().blobs().blob()) {
            String output = "Blob name: " + b.name();
            if (b.snapshot() != null) {
                output += ", Snapshot: " + b.snapshot();
            }
            System.out.println(output);
        }
    }
    else {
        System.out.println("There are no more blobs to list off.");
    }
    
    // If there is not another segment, return this response as the final response.
    if (response.body().nextMarker() == null) {
        return Single.just(response);
    } else {
        /*
        IMPORTANT: ListBlobsFlatSegment returns the start of the next segment; you MUST use this to get the next
        segment (after processing the current result segment
        */
            
        String nextMarker = response.body().nextMarker();

        /*
        The presence of the marker indicates that there are more blobs to list, so we make another call to
        listBlobsFlatSegment and pass the result through this helper function.
        */
            
    return url.listBlobsFlatSegment(nextMarker, new ListBlobsOptions(null, null,1))
        .flatMap(containersListBlobFlatSegmentResponse ->
            listAllBlobs(url, containersListBlobFlatSegmentResponse));
    }
}
```

### <a name="download-blobs"></a>下載 Blob

使用 [BlobURL.download](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._blob_u_r_l.download?view=azure-java-preview) 方法，將 Blob 下載至本機磁碟。

下列程式碼會下載前一節中上傳的 Blob。 它會將 **_DOWNLOADED** 新增為 Blob 名稱的尾碼，讓您可在本機磁碟上看到這兩個檔案。 

```java
static void getBlob(BlockBlobURL blobURL, File sourceFile) {
    try {
        // Get the blob using the low-level download method in BlockBlobURL type
        // com.microsoft.rest.v2.util.FlowableUtil is a static class that contains helpers to work with Flowable
        blobURL.download(new BlobRange(0, Long.MAX_VALUE), null, false)
            .flatMapCompletable(response -> {
                AsynchronousFileChannel channel = AsynchronousFileChannel.open(Paths
                    .get(sourceFile.getPath()), StandardOpenOption.CREATE,  StandardOpenOption.WRITE);
                        return FlowableUtil.writeFile(response.body(), channel);
            }).doOnComplete(()-> System.out.println("The blob was downloaded to " + sourceFile.getAbsolutePath()))
            // To call it synchronously add .blockingAwait()
            .subscribe();
    } catch (Exception ex){
        System.out.println(ex.toString());
    }
}
```

### <a name="clean-up-resources"></a>清除資源

如果您已不需要在本快速入門中上傳的 Blob，您可以使用 [ContainerURL.delete](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob._container_u_r_l.delete?view=azure-java-preview) 刪除整個容器。 此方法也會刪除容器中的檔案。

```java
containerURL.delete(null).blockingGet();
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已了解如何使用 Java 在本機磁碟和 Azure Blob 儲存體之間傳送檔案。 

> [!div class="nextstepaction"]
> [適用於 Java 原始程式碼的 Storage SDK V10](https://github.com/Azure/azure-storage-java/tree/New-Storage-SDK-V10-Preview)
> [API 參考](https://docs.microsoft.com/java/api/overview/azure/storage/client?view=azure-java-preview)
> [深入了解 RxJava](https://github.com/ReactiveX/RxJava)
