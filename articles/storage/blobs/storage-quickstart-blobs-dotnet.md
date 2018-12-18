---
title: 快速入門：使用 .NET 在物件儲存體中建立 Blob - Azure 儲存體
description: 在此快速入門中，您將了解如何使用適用於 .NET 的 Azure 儲存體用戶端程式庫在 Blob (物件) 儲存體中建立容器與 Blob。 接下來，您要了解如何將 Blob 下載到本機電腦，以及如何列出容器中的所有 Blob。
services: storage
author: tamram
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 08/27/2018
ms.author: tamram
ms.openlocfilehash: cde8516b93c2aac0cb74d9c9599168c8e6a70472
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45734128"
---
# <a name="quickstart-use-net-to-create-a-blob-in-object-storage"></a>快速入門：使用 .NET 在物件儲存體中建立 Blob

在此快速入門中，您將了解如何使用適用於 .NET 的 Azure 儲存體用戶端程式庫在 Blob (物件) 儲存體中建立容器與 Blob。 接下來，您要了解如何將 Blob 下載到本機電腦，以及如何列出容器中的所有 Blob。

## <a name="prerequisites"></a>必要條件

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

若要完成本快速入門，首先在 [Azure 入口網站](https://portal.azure.com/#create/Microsoft.StorageAccount-ARM)中建立 Azure 儲存體帳戶。 如需建立帳戶的協助，請參閱[建立儲存體帳戶](../common/storage-quickstart-create-account.md)。

接下來，請下載並安裝您的作業系統適用的 .NET Core 2.0。 如果執行 Windows，您可以安裝 Visual Studio 並使用 .NET Framework (視您的喜好而定)。 您也可以選擇安裝編輯器來搭配您的作業系統使用。

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

- 安裝 [.NET Core for Windows](https://www.microsoft.com/net/download/windows) 或 [.NET Framework](https://www.microsoft.com/net/download/windows) (隨附於 Visual Studio for Windows)
- 安裝 [Visual Studio for Windows](https://www.visualstudio.com/)。 如果您使用 .NET Core，可選擇性安裝 Visual Studio。  

如需選擇 .NET Core 或 .NET Framework 的相關資訊，請參閱[針對伺服器應用程式選擇 .NET Core 或 .NET Framework](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server)。

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

- 安裝 [.NET Core for Linux](https://www.microsoft.com/net/download/linux)
- 選擇性安裝 [Visual Studio Code](https://www.visualstudio.com/) 和 [C# 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp&dotnetid=963890049.1518206068)

# <a name="macostabmacos"></a>[macOS](#tab/macos)

- 安裝 [.NET Core for macOS](https://www.microsoft.com/net/download/macos).
- 選擇性安裝 [Visual Studio for Mac](https://www.visualstudio.com/vs/visual-studio-mac/)

---

## <a name="download-the-sample-application"></a>下載範例應用程式

本快速入門使用的範例應用程式是基本的主控台應用程式。 您可探索 [GitHub](https://github.com/Azure-Samples/storage-blobs-dotnet-quickstart) 上的範例應用程式。

使用 [git](https://git-scm.com/) 將應用程式的複本下載至您的開發環境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-dotnet-quickstart.git
```

此命令會將存放庫複製到本機的 git 資料夾。 若要開啟 Visual Studio 解決方案，請找出 *storage-blobs-dotnet-quickstart* 資料夾並開啟它，然後按兩下 *storage-blobs-dotnet-quickstart.sln*。 

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串

若要執行應用程式，您必須提供儲存體帳戶的連接字串。 此範例應用程式會讀取環境變數中的連接字串，並使用它來授權對 Azure 儲存體的要求。

在複製您的連接字串後，請在執行應用程式的本機電腦上，將該字串寫入至新的環境變數中。 若要設定環境變數，請開啟主控台視窗，並遵循您的作業系統所適用的指示。 將 `<yourconnectionstring>` 用實際的連接字串取代：

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

新增環境變數之後，您可能需要重新啟動任何需要讀取環境變數的執行中程式，包括主控台視窗。 例如，如果您使用 Visual Studio 做為編輯器，請在執行範例前重新啟動 Visual Studio。 

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```bash
export storageconnectionstring=<yourconnectionstring>
```

新增環境變數之後，從主控台視窗執行 `source ~/.bashrc`，讓變更生效。

# <a name="macostabmacos"></a>[macOS](#tab/macos)

編輯 .bash_profile，然後新增環境變數：

```bash
export STORAGE_CONNECTION_STRING=<yourconnectionstring>
```

新增環境變數之後，從主控台視窗執行 `source .bash_profile`，讓變更生效。

---

## <a name="run-the-sample"></a>執行範例

這個範例會在本機 **MyDocuments** 資料夾中建立測試檔案，並將它上傳到 Blob 儲存體。 這個範例會接著列出容器中的 Blob，並下載具有新名稱的檔案，您便可比較舊檔案和新檔案。 

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

如果您使用 Visual Studio 作為編輯器，請按 **F5** 來執行。 

否則，瀏覽至您的應用程式目錄，並使用 `dotnet run` 命令執行應用程式。

```
dotnet run
```

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

瀏覽至您的應用程式目錄，並使用 `dotnet run` 命令執行應用程式。

```
dotnet run
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

瀏覽至您的應用程式目錄，並使用 `dotnet run` 命令執行應用程式。

```
dotnet run
```

---

範例應用程式的輸出類似下列範例：

```
Azure Blob storage - .NET Quickstart sample

Created container 'quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f'

Temp file = C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt
Uploading to Blob storage as blob 'QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt'

Listing blobs in container.
https://storagesamples.blob.core.windows.net/quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f/QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt

Downloading blob to C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db_DOWNLOADED.txt

Press any key to delete the sample files and example container.
```

當您按 **Enter** 鍵時，應用程式會刪除儲存體容器和檔案。 在刪除之前，請檢查 **MyDocuments** 資料夾，找出這兩個檔案。 您可以開啟它們，並觀察它們是否相同。 複製主控台視窗中的 Blob URL，將它貼至瀏覽器以檢視 Blob 的內容。

確認檔案之後，請按任一鍵以完成示範並刪除測試檔案。 現在您已知道這個範例的工作內容，請開啟 Program.cs 檔案中查看程式碼。 

## <a name="understand-the-sample-code"></a>了解範例程式碼

接下來，探索範例程式碼來了解其運作方式。

### <a name="try-parsing-the-connection-string"></a>嘗試剖析連接字串

此範例進行的第一件事就是檢查環境變數是否包含可剖析的連接字串，以建立指向儲存體帳戶的 [CloudStorageAccount](/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount) 物件。 若要檢查連接字串是否有效，請使用[TryParse](/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount.tryparse) 方法。 如果 **TryParse** 成功，它會初始化 storageAccount 變數並傳回 **true**。

```csharp
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable on the machine running the application called storageconnectionstring.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell or application needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with operations against Blob storage here.
    ...
}
else
{
    // Otherwise, let the user know that they need to define the environment variable.
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add a environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
    Console.WriteLine("Press any key to exit the sample application.");
    Console.ReadLine();
}
```

### <a name="create-the-container-and-set-permissions"></a>建立容器並設定權限

接下來，此範例會建立容器並設定其權限，讓容器中的任何 blob 都是公用的。 如果 blob 是公用的，任何用戶端都可以匿名方式存取它。

若要建立容器，請先建立 [CloudBlobClient](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobclient) 物件的執行個體，其指向您儲存體帳戶中的 Blob 儲存體。 接下來，建立 [CloudBlobContainer](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer) 物件的執行個體，然後建立容器。 

在此情況下，此範例會呼叫 [CreateAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.createasync) 方法來建立容器。 容器名稱會附加 GUID 值，以確保它是唯一的。 在生產環境中，通常偏好使用 [CreateIfNotExistsAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.createifnotexistsasync) 方法來建立容器 (僅限容器不存在時)，以避免發生命名衝突。

> [!IMPORTANT]
> 容器名稱必須是小寫字母。 如需為容器和 Blob 命名的詳細資訊，請參閱[命名和參考容器、Blob 及中繼資料](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。


```csharp
// Create the CloudBlobClient that represents the Blob storage endpoint for the storage account.
CloudBlobClient cloudBlobClient = storageAccount.CreateCloudBlobClient();

// Create a container called 'quickstartblobs' and append a GUID value to it to make the name unique. 
cloudBlobContainer = cloudBlobClient.GetContainerReference("quickstartblobs" + Guid.NewGuid().ToString());
await cloudBlobContainer.CreateAsync();

// Set the permissions so the blobs are public. 
BlobContainerPermissions permissions = new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
};
await cloudBlobContainer.SetPermissionsAsync(permissions);
```

### <a name="upload-blobs-to-the-container"></a>將 Blob 上傳到容器

接下來，此範例會將本機檔案上載至區塊 blob。 此程式碼範例會藉由在前一節建立的容器上呼叫 [GetBlockBlobReference](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.getblockblobreference) 方法，以取得 **CloudBlockBlob** 物件的參考。 然後藉由呼叫 [UploadFromFileAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblockblob.uploadfromfileasync) 方法，將所選的檔案上傳到 blob。 如果 Blob 不存在，此方法會建立 Blob，若已存在，則會覆寫它。 

```csharp
// Create a file in your local MyDocuments folder to upload to a blob.
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string localFileName = "QuickStart_" + Guid.NewGuid().ToString() + ".txt";
sourceFile = Path.Combine(localPath, localFileName);
// Write text to the file.
File.WriteAllText(sourceFile, "Hello, World!");

Console.WriteLine("Temp file = {0}", sourceFile);
Console.WriteLine("Uploading to Blob storage as blob '{0}'", localFileName);

// Get a reference to the blob address, then upload the file to the blob.
// Use the value of localFileName for the blob name.
CloudBlockBlob cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(localFileName);
await cloudBlockBlob.UploadFromFileAsync(sourceFile);
```

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

此範例會使用 [ListBlobsSegmentedAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.listblobssegmentedasync) 方法列出容器中的 blob。 在此範例中，只有一個 blob 新增至容器，所以清單作業只會傳回一個 blob。

如果單一呼叫中有太多 blob 要傳回 (根據預設，超過 5000 個)，則 **ListBlobsSegmentedAsync** 方法會傳回總結果集區段和接續 Token。 若要擷取 blob 的下一個區段，您可提供前一次呼叫傳回的接續 Token 等等，直到接續 Token 是 null 為止。 Null 接續 Token 表示已擷取所有的 blob。 此範例程式碼會示範如何使用接續 Token 來達到最佳做法。

```csharp
// List the blobs in the container.
Console.WriteLine("List blobs in container.");
BlobContinuationToken blobContinuationToken = null;
do
{
    var results = await cloudBlobContainer.ListBlobsSegmentedAsync(null, blobContinuationToken);
    // Get the value of the continuation token returned by the listing call.
    blobContinuationToken = results.ContinuationToken;
    foreach (IListBlobItem item in results.Results)
    {
        Console.WriteLine(item.Uri);
    }
} while (blobContinuationToken != null); // Loop while the continuation token is not null. 

```

### <a name="download-blobs"></a>下載 Blob

接下來，此範例會使用 [DownloadToFileAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.downloadtofileasync) 方法，將先前建立的 blob 下載至本機檔案系統。 此範例程式碼會將 "_DOWNLOADED" 後置字元新增至 blob 名稱，您便可在本機檔案系統中看到這兩個檔案。

```csharp
// Download the blob to a local file, using the reference created earlier. 
// Append the string "_DOWNLOADED" before the .txt extension so that you can see both files in MyDocuments.
destinationFile = sourceFile.Replace(".txt", "_DOWNLOADED.txt");
Console.WriteLine("Downloading blob to {0}", destinationFile);
await cloudBlockBlob.DownloadToFileAsync(destinationFile, FileMode.Create);  
```

### <a name="clean-up-resources"></a>清除資源

此範例會藉由使用 [CloudBlobContainer.DeleteAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.deleteasync) 刪除整個容器，以清除它所建立的資源。 您也可以視需要刪除本機檔案。

```csharp
Console.WriteLine("Press the 'Enter' key to delete the sample files, example container, and exit the application.");
Console.ReadLine();
// Clean up resources. This includes the container and the two temp files.
Console.WriteLine("Deleting the container");
if (cloudBlobContainer != null)
{
    await cloudBlobContainer.DeleteIfExistsAsync();
}
Console.WriteLine("Deleting the source, and downloaded files");
File.Delete(sourceFile);
File.Delete(destinationFile);
```

## <a name="resources-for-developing-net-applications-with-blobs"></a>可供使用 Blob 開發 .NET 應用程式的資源

請參閱以下可供使用 Blob 儲存體進行 .NET 開發的額外資源：

### <a name="binaries-and-source-code"></a>二進位檔和原始程式碼

- 針對 Azure 儲存體最新版的 [.NET 用戶端程式庫](https://www.nuget.org/packages/WindowsAzure.Storage/)下載 NuGet 套件。 
- 在 GitHub 上檢視 [.NET 用戶端程式庫原始程式碼](https://github.com/Azure/azure-storage-net)。

### <a name="client-library-reference-and-samples"></a>用戶端程式庫參考和範例

- 如需 .NET 用戶端程式庫的詳細資訊，請參閱 [.NET API 參考](https://docs.microsoft.com/dotnet/api/overview/azure/storage)。
- 探索使用 .NET 用戶端程式庫所撰寫的 [Blob 儲存體範例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=blob)。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何使用 .NET 上傳、下載及列出 Blob。 

若要了解如何建立可將映像上傳至 Blob 儲存體的 Web 應用程式，請繼續進行[使用 Azure 儲存體在雲端上傳影像資料](storage-upload-process-images.md)。

> [!div class="nextstepaction"]
> [Blob 儲存體作業操作說明](storage-dotnet-how-to-use-blobs.md)

- 若要深入了解 .NET Core，請參閱[在 10 分鐘內開始使用 .NET](https://www.microsoft.com/net/learn/get-started/)。
- 若要探索您可以從 Visual Studio for Windows 部署的範例應用程式，請參閱[使用 Azure Blob 儲存體的 .NET 相片圖庫 Web 應用程式範例](https://azure.microsoft.com/resources/samples/storage-blobs-dotnet-webapp/)。
 