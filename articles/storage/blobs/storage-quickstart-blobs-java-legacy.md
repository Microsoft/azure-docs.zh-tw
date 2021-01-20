---
title: 快速入門：適用於 Java 的 Azure Blob 儲存體用戶端程式庫 v8
description: 在物件 (Blob) 儲存體中建立儲存體帳戶和容器。 然後，使用 Java 適用的 Azure 儲存體用戶端程式庫 v8，將 Blob 上傳至 Azure 儲存體、下載 Blob，以及列出容器中的 Blob。
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 01/19/2021
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: b3bb8736eb7a8e24f47812fc4feecbf96628dfb6
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98599803"
---
# <a name="quickstart-manage-blobs-with-java-v8-sdk"></a>快速入門：使用 Java v8 SDK 來管理 Blob

在本快速入門中，您將了解如何使用 Java 來管理 Blob。 Blob 是可保存大量文字或二進位資料的物件，包括影像、文件、串流媒體和封存資料。 您會上傳、下載及列出 Blob 您也會建立、設定容器權限和刪除容器。

> [!NOTE]
> 本快速入門會使用舊版的 Azure Blob 儲存體用戶端程式庫。 若要開始使用最新版本，請參閱[快速入門：使用 Java v12 SDK 來管理 Blob](storage-quickstart-blobs-java.md)。

## <a name="prerequisites"></a>必要條件

- 具有有效訂用帳戶的 Azure 帳戶。 [免費建立帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- Azure 儲存體帳戶。 [建立儲存體帳戶](../common/storage-account-create.md)。
- 具有 Maven 整合的 IDE。 本指南使用具備「適用於 Java 開發人員的 Eclipse IDE」設定的 [Eclipse](https://www.eclipse.org/downloads/)。

## <a name="download-the-sample-application"></a>下載範例應用程式

[範例應用程式](https://github.com/Azure-Samples/storage-blobs-java-quickstart)是基本主控台應用程式。

使用 [git](https://git-scm.com/) 將應用程式的複本下載至您的開發環境。

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-quickstart.git
```

此命令會將存放庫複製到本機的 git 資料夾。 若要開啟專案，請啟動 Eclipse 並關閉歡迎畫面。 選取 [File] \(檔案\)，然後選取 [Open Projects from File System] \(從檔案系統中開啟專案\)。 請確定已核取 [Detect and configure project natures] \(偵測並設定專案性質\)。 選取 [Directory] \(目錄\)，然後瀏覽至您儲存複製存放庫的位置。 在複製的存放庫中選取 [blobAzureApp] 資料夾。 請確定 **blobAzureApp** 專案顯示為 Eclipse 專案，然後選取 [Finish] \(完成\)。

當專案完成匯入之後，開啟 **AzureApp.java** (位於 **src/main/java** 內的 **blobQuickstart.blobAzureApp**)，並取代 `storageConnectionString` 字串內的 `accountname` 和 `accountkey`。 然後執行應用程式。 下列各節會說明可用來完成這些工作的特定指示。

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串

在應用程式，您必須提供儲存體帳戶的連接字串。 開啟 **AzureApp.Java** 檔案。 找出 `storageConnectionString` 變數，並貼上您在上一節中複製的連接字串值。 您的 `storageConnectionString` 變數看起來應該會像下列程式碼範例：

```java
public static final String storageConnectionString =
"DefaultEndpointsProtocol=https;" +
"AccountName=<account-name>;" +
"AccountKey=<account-key>";
```

## <a name="run-the-sample"></a>執行範例

此應用程式範例會在您預設的目錄 (如果是 Windows 使用者，則是 *C:\Users\<user>\AppData\Local\Temp*) 中建立測試檔案、將其上傳至 Blob 儲存體、列出容器中的 Blob，然後以新名稱下載該檔案，以便比較新舊檔案。

在命令列上使用 Maven 執行範例。 開啟殼層，然後瀏覽至複製目錄內的 **blobAzureApp**。 然後輸入 `mvn compile exec:java`。

如果您是在 Windows 上執行應用程式，則下列範例會顯示輸出。

```
Azure Blob storage quick start sample
Creating container: quickstartcontainer
Creating a sample file at: C:\Users\<user>\AppData\Local\Temp\sampleFile514658495642546986.txt
Uploading the sample file
URI of blob is: https://myexamplesacct.blob.core.windows.net/quickstartcontainer/sampleFile514658495642546986.txt
The program has completed successfully.
Press the 'Enter' key while in the console to delete the sample files, example container, and exit the application.

Deleting the container
Deleting the source, and downloaded files
```

在繼續之前，請先檢查預設目錄 (如果是 Windows 使用者，則是 *C:\Users\<user>\AppData\Local\Temp*) 是否有範例檔案。 複製主控台視窗的 Blob URL，將它貼至瀏覽器以檢視 Blob 儲存體中的檔案內容。 如果您比較目錄中的範例檔案與 Blob 儲存體中儲存的內容，您會發現兩者相同。

  >[!NOTE]
  >您也可以使用 [Azure 儲存體總管](https://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 之類的工具來檢視 Blob 儲存體中的檔案。 Azure 儲存體總管是免費的跨平台工具，可讓您存取儲存體帳戶資訊。

確認檔案之後，請按 **Enter** 鍵以完成示範並刪除測試檔案。 現在您已知道這個範例的功用，請開啟 **AzureApp.java** 檔案查看程式碼。

## <a name="understand-the-sample-code"></a>了解範例程式碼

接下來，我們將透過範例程式碼來了解其運作方式。

### <a name="get-references-to-the-storage-objects"></a>取得儲存體物件的參考

第一件事是建立用來存取和管理 Blob 儲存體的物件參考。 這些物件是互為建置基礎，各自都為清單中的下一個物件所使用。

* 建立指向儲存體帳戶之 [CloudStorageAccount](/java/api/com.microsoft.azure.management.storage.storageaccount) 物件的執行個體。

    **CloudStorageAccount** 物件是您儲存體帳戶的表示法，它可讓您以程式設計方式來設定和存取儲存體帳戶屬性。 使用 **CloudStorageAccount** 物件，您可以建立 **CloudBlobClient** 的執行個體，這是存取 Blob 服務的必要條件。

* 建立 **CloudBlobClient** 物件的執行個體，指向您儲存體帳戶中的 [Blob 服務](/java/api/com.microsoft.azure.storage.blob.cloudblobclient)。

    **CloudBlobClient** 能為您提供 Blob 服務的存取點，可讓您以程式設計方式來設定和存取 Blob 儲存體屬性。 使用 **CloudBlobClient**，您可以建立 **CloudBlobContainer** 物件的執行個體，這是建立容器的必要條件。

* 建立 [CloudBlobContainer](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer) 物件的執行個體，代表您要存取的容器。 請使用容器來組織 Blob，就像在電腦上用資料夾組織檔案一樣。

    只要您有 **CloudBlobContainer**，就可以建立 [CloudBlockBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob) 物件的執行個體，指向您感興趣的特定 Blob 並執行上傳、下載、複製和其他作業。

> [!IMPORTANT]
> 容器名稱必須是小寫字母。 如需容器的詳細資訊，請參閱[命名和參考容器、Blob 及中繼資料](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

### <a name="create-a-container"></a>建立容器

在本節中，您要建立物件的執行個體、建立新的容器，然後設定容器上的權限，以便公開 Blob 並使用 URL 即可存取。 容器名為 **quickstartcontainer**。

因為我們需要在每次執行範例時建立新的容器，所以此範例會使用 [CreateIfNotExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists)。 在整個應用程式都使用相同容器的生產環境中，最好只呼叫一次 **CreateIfNotExists**。 或者，您可以事先建立容器，就不需要在程式碼中建立。

```java
// Parse the connection string and create a blob client to interact with Blob storage
storageAccount = CloudStorageAccount.parse(storageConnectionString);
blobClient = storageAccount.createCloudBlobClient();
container = blobClient.getContainerReference("quickstartcontainer");

// Create the container if it does not exist with public access.
System.out.println("Creating container: " + container.getName());
container.createIfNotExists(BlobContainerPublicAccessType.CONTAINER, new BlobRequestOptions(), new OperationContext());
```

### <a name="upload-blobs-to-the-container"></a>將 Blob 上傳到容器

若要將檔案上傳至區塊 Blob，請取得目標容器中的 Blob 參考。 取得 Blob 參考之後，即可使用 [CloudBlockBlob.Upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload) 將資料上傳至 Blob。 如果沒有 Blob，此作業會建立 Blob，如已存在，則予以覆寫。

範例程式碼會建立用於上傳和下載的本機檔案，將要上傳的檔案以 **blob** 中的 Blob 名稱儲存為 **source**。 下列範例會將檔案上傳到名為 **quickstartcontainer** 的容器。

```java
//Creating a sample file
sourceFile = File.createTempFile("sampleFile", ".txt");
System.out.println("Creating a sample file at: " + sourceFile.toString());
Writer output = new BufferedWriter(new FileWriter(sourceFile));
output.write("Hello Azure!");
output.close();

//Getting a blob reference
CloudBlockBlob blob = container.getBlockBlobReference(sourceFile.getName());

//Creating blob and uploading file to it
System.out.println("Uploading the sample file ");
blob.uploadFromFile(sourceFile.getAbsolutePath());
```

可與 Blob 儲存體搭配使用的 `upload` 方法有好幾種，包括 [upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload)、[uploadBlock](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadblock)、[uploadFullBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadfullblob)、[uploadStandardBlobTier](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadstandardblobtier) 和 [uploadText](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadtext)。 例如，如果您有一個字串，便可以使用 `UploadText` 方法來取代 `Upload` 方法。

區塊 Blob 可以是任何類型的文字或二進位檔案。 分頁 Blob 主要用於備份 IaaS VM 的 VHD 檔案。 附加 Blob 可用於記錄，例如，當您想要寫入檔案，並繼續新增更多資訊時。 儲存在 Blob 儲存體中的大部分物件都是區塊 Blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

您可以使用 [CloudBlobContainer.ListBlobs](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobs) 來取得容器中的檔案清單。 下列程式碼會擷取 Blob 的清單，透過它們執行迴圈，顯示找到的 Blob URI。 您可以複製命令視窗中的 URI，將它貼入至瀏覽器來檢視檔案。

```java
//Listing contents of container
for (ListBlobItem blobItem : container.listBlobs()) {
    System.out.println("URI of blob is: " + blobItem.getUri());
}
```

### <a name="download-blobs"></a>下載 Blob

使用 [CloudBlob.DownloadToFile](/java/api/com.microsoft.azure.storage.blob.cloudblob.downloadtofile) 將 Blob 下載到本機磁碟。

下列程式碼會下載上一節中上傳的 Blob，將 "_DOWNLOADED" 尾碼新增至 Blob 名稱，以便您在本機磁碟上看到這兩個檔案。

```java
// Download blob. In most cases, you would have to retrieve the reference
// to cloudBlockBlob here. However, we created that reference earlier, and
// haven't changed the blob we're interested in, so we can reuse it.
// Here we are creating a new file to download to. Alternatively you can also pass in the path as a string into downloadToFile method: blob.downloadToFile("/path/to/new/file").
downloadedFile = new File(sourceFile.getParentFile(), "downloadedFile.txt");
blob.downloadToFile(downloadedFile.getAbsolutePath());
```

### <a name="clean-up-resources"></a>清除資源

如果不再需要您所上傳的 Blob，您可以使用 [CloudBlobContainer.DeleteIfExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists) 刪除整個容器。 此方法也會刪除容器中的檔案。

```java
try {
if(container != null)
    container.deleteIfExists();
} catch (StorageException ex) {
System.out.println(String.format("Service error. Http code: %d and error code: %s", ex.getHttpStatusCode(), ex.getErrorCode()));
}

System.out.println("Deleting the source, and downloaded files");

if(downloadedFile != null)
downloadedFile.deleteOnExit();

if(sourceFile != null)
sourceFile.deleteOnExit();
```

## <a name="next-steps"></a>下一步

在本文中，您已了解如何使用 Java 在本機磁碟和 Azure Blob 儲存體之間傳送檔案。 若要深入了解 Java 的用法，請繼續使用我們的 GitHub 開放原始碼存放庫。

> [!div class="nextstepaction"]
> [Java API 參考](/java/api/overview/azure/storage?view=azure-java-legacy&preserve-view=true)
> [Java 的程式碼範例](../common/storage-samples-java.md)