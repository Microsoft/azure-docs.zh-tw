---
title: 使用 Java 開發 Azure 檔案服務 | Microsoft Docs
description: 了解如何開發使用 Azure 檔案服務來儲存檔案資料的 Java 應用程式和服務。
services: storage
author: wmgries
ms.service: storage
ms.devlang: Java
ms.topic: article
ms.date: 09/19/2017
ms.author: renash
ms.component: files
ms.openlocfilehash: aa63a31f7f84502a29aad6b38f454ea1080127e0
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529089"
---
# <a name="develop-for-azure-files-with-java"></a>使用 Java 開發 Azure 檔案服務
[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="about-this-tutorial"></a>關於本教學課程
本教學課程將示範基本概念，說明如何利用 Java來開發使用 Azure 檔案服務以儲存檔案資料的應用程式或服務。 在本教學課程中，我們將建立主控台應用程式，並說明如何執行 Java 和 Azure 檔案服務的基本動作：

* 建立及刪除 Azure 檔案共用
* 建立及刪除目錄
* 列舉 Azure 檔案共用的檔案和目錄
* 上傳、下載及刪除檔案

> [!Note]  
> 由於 Azure 檔案服務可透過 SMB 存取，因此便可使用標準 Java I/O 類別撰寫應用程式以存取 Azure 檔案共用。 本文將說明如何撰寫使用 Azure 儲存體 Java SDK 的應用程式，它會使用 [Azure 檔案服務 REST API](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api) 與 Azure 檔案服務通訊。

## <a name="create-a-java-application"></a>建立 Java 應用程式
若要建置範例，您將需要 Java Development Kit (JDK) 和 [Azure Storage SDK for Java](https://github.com/Azure/azure-storage-java)。 您也應該建立 Azure 儲存體帳戶。

## <a name="set-up-your-application-to-use-azure-files"></a>設定您的應用程式以使用 Azure 檔案服務
若要使用 Azure 儲存體 API，請將下列陳述式加入至您要從其存取儲存體服務的 Java 檔案頂端。

```java
// Include the following imports to use blob APIs.
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.file.*;
```

## <a name="set-up-an-azure-storage-connection-string"></a>設定 Azure 儲存體連接字串
若要使用 Azure 檔案服務，您必須連接到您的 Azure 儲存體帳戶。 第一個步驟是設定連接字串，我們將用它來連線至您的儲存體帳戶。 讓我們定義靜態變數以便進行。

```java
// Configure the connection-string with your values
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account_name;" +
    "AccountKey=your_storage_account_key";
```

> [!NOTE]
> 將 your_storage_account_name 和 your_storage_account_key 取代為您的儲存體帳戶的實際值。
> 
> 

## <a name="connecting-to-an-azure-storage-account"></a>連接到 Azure 儲存體帳戶
若要連接到您的儲存體帳戶，您必須使用 **CloudStorageAccount** 物件，將連接字串傳遞至其**剖析**方法。

```java
// Use the CloudStorageAccount object to connect to your storage account
try {
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
} catch (InvalidKeyException invalidKey) {
    // Handle the exception
}
```

**CloudStorageAccount.parse** 會擲回 InvalidKeyException，因此您必須將其放在 try/catch 區塊內。

## <a name="create-an-azure-file-share"></a>建立 Azure 檔案共用
Azure 檔案服務中的所有檔案和目錄都位於名為 [共用] 的容器中。 您的儲存體帳戶可以有帳戶容量允許數量的共用。 若要取得共用及其內容的存取權，您必須使用 Azure 檔案服務用戶端。

```java
// Create the Azure Files client.
CloudFileClient fileClient = storageAccount.createCloudFileClient();
```

使用 Azure 檔案服務用戶端時，您即可取得共用的參考。

```java
// Get a reference to the file share
CloudFileShare share = fileClient.getShareReference("sampleshare");
```

若要實際建立共用，請使用 CloudFileShare 物件的 **createIfNotExists** 方法。

```java
if (share.createIfNotExists()) {
    System.out.println("New share created");
}
```

目前，**共用**會將參考保留至名為 **sampleshare** 的共用。

## <a name="delete-an-azure-file-share"></a>刪除 Azure 檔案共用
刪除共用可以藉由在 CloudFileShare 物件呼叫 **deleteIfExists** 方法來完成。 以下是執行該作業的範例程式碼。

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    // Create the file client.
   CloudFileClient fileClient = storageAccount.createCloudFileClient();

   // Get a reference to the file share
   CloudFileShare share = fileClient.getShareReference("sampleshare");

   if (share.deleteIfExists()) {
       System.out.println("sampleshare deleted");
   }
} catch (Exception e) {
    e.printStackTrace();
}
```

## <a name="create-a-directory"></a>建立目錄
您也可以組織儲存體，方法是將檔案放在子目錄中，而不是將所有檔案都放在根目錄中。 Azure 檔案服務可讓您建立帳戶允許數量的目錄。 下列程式碼會在根目錄底下建立名為 **sampledir** 的子目錄。

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the sampledir directory
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

if (sampleDir.createIfNotExists()) {
    System.out.println("sampledir created");
} else {
    System.out.println("sampledir already exists");
}
```

## <a name="delete-a-directory"></a>刪除目錄
刪除目錄是一項直接的工作，不過請注意您無法刪除仍然包含檔案或其他目錄的目錄。

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory you want to delete
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

// Delete the directory
if ( containerDir.deleteIfExists() ) {
    System.out.println("Directory deleted");
}
```

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>列舉 Azure 檔案共用的檔案和目錄
取得共用內檔案和目錄的清單很容易，只要在 CloudFileDirectory 參考上呼叫 **listFilesAndDirectories** 即可。 方法會傳回您可以逐一查看的 ListFileItem 物件清單。 例如，下列程式碼會列出根目錄內的檔案和目錄。

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
    System.out.println(fileItem.getUri());
}
```

## <a name="upload-a-file"></a>上傳檔案
在本節中，您將學習如何從本機儲存體將檔案上傳至共用的根目錄。

上傳檔案的第一個步驟是取得檔案所在之目錄的參考。 您可以藉由呼叫共用物件的 **getRootDirectoryReference** 方法來完成。

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();
```

現在您具有共用之根目錄的參考，您可以使用下列程式碼上傳檔案。

```java
        // Define the path to a local file.
        final String filePath = "C:\\temp\\Readme.txt";
    
        CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
        cloudFile.uploadFromFile(filePath);
```

## <a name="download-a-file"></a>下載檔案
您針對 Azure 檔案服務執行的其中一個較頻繁作業是下載檔案。 在下列範例中，程式碼會下載 SampleFile.txt，並顯示其內容。

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the directory that contains the file
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

//Get a reference to the file you want to download
CloudFile file = sampleDir.getFileReference("SampleFile.txt");

//Write the contents of the file to the console.
System.out.println(file.downloadText());
```

## <a name="delete-a-file"></a>刪除檔案
另一個常見的 Azure 檔案服務作業是刪除檔案。 下列程式碼會刪除儲存在名為 **sampledir**之目錄內名為 SampleFile.txt 的檔案。

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory where the file to be deleted is in
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

String filename = "SampleFile.txt"
CloudFile file;

file = containerDir.getFileReference(filename)
if ( file.deleteIfExists() ) {
    System.out.println(filename + " was deleted");
}
```

## <a name="next-steps"></a>後續步驟
如果您想要深入了解其他 Azure 儲存體 API，請參考下列連結。

* [適用於 Java 開發人員的 Azure](/java/azure)/)
* [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)
* [Azure Storage SDK for Android](https://github.com/azure/azure-storage-android)
* [Azure 儲存體用戶端 SDK 參考](http://dl.windowsazure.com/storage/javadoc/)
* [Azure 儲存體服務 REST API](https://msdn.microsoft.com/library/azure/dd179355.aspx)
* [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage/)
* [使用 AzCopy 命令列公用程式傳輸資料](../common/storage-use-azcopy.md)
* [針對 Azure 檔案服務問題進行疑難排解 - Windows](storage-troubleshoot-windows-file-connection-problems.md)