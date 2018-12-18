---
title: 使用 Microsoft Azure 儲存體資料移動程式庫傳輸資料 | Microsoft Docs
description: 使用資料移動程式庫從 Blob 和檔案內容移動或來回複製資料。 從本機檔案複製資料到 Azure 儲存體，或在儲存體帳戶內或之間複製資料。 輕鬆地將資料移轉至 Azure 儲存體。
services: storage
author: seguler
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 09/27/2017
ms.author: seguler
ms.component: common
ms.openlocfilehash: e62023f5aeca73973598430ed357e8e9800a17f8
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45732238"
---
# <a name="transfer-data-with-the-microsoft-azure-storage-data-movement-library"></a>使用 Microsoft Azure 儲存體資料移動程式庫傳輸資料

## <a name="overview"></a>概觀
Microsoft Azure 儲存體資料移動程式庫是跨平台的開放原始碼程式庫，設計用來提供 Azure 儲存體 Blob 和檔案的高效能上傳、下載及複製。 這個程式庫是支援 [AzCopy](../storage-use-azcopy.md) 的核心資料移動架構。 資料移動程式庫可提供傳統的 [.NET Azure 儲存體用戶端程式庫](../blobs/storage-dotnet-how-to-use-blobs.md)中並未提供的簡便方法。 這包括設定平行作業數目、追蹤傳輸進度、輕鬆繼續已取消的傳輸等等。  

此程式庫也會使用 .NET Core，這表示您在建置適用於 Windows、Linux 和 macOS 的 .NET 應用程式時可以使用它。 若要深入了解 .NET Core，請參閱 [.NET Core 文件 (英文)](https://dotnet.github.io/)。 這個程式庫也適用於 Windows 的傳統 .NET 架構應用程式。 

本文件將示範如何建立可在 Windows、Linux 和 macOS 上執行的 .NET Core 主控台應用程式，並執行下列案例：

- 將檔案和目錄上傳至 Blob 儲存體。
- 定義傳輸資料時的平行作業數目。
- 追蹤資料傳輸進度。
- 繼續已取消的資料傳輸。 
- 將檔案從 URL 複製到 Blob 儲存體。 
- 從 Blob 儲存體複製到 Blob 儲存體。

**您需要的項目：**

* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure 儲存體帳戶](storage-quickstart-create-account.md)

> [!NOTE]
> 本指南假設您已熟悉 [Azure 儲存體](https://azure.microsoft.com/services/storage/)。 如果不熟悉，閱讀 [Azure 儲存體簡介](storage-introduction.md)說明文件會很有幫助。 最重要的是，您需要[建立儲存體帳戶](storage-quickstart-create-account.md)才能開始使用資料移動程式庫。
> 
> 

## <a name="setup"></a>設定  

1. 瀏覽 [.NET Core 安裝指南](https://www.microsoft.com/net/core)以安裝 .NET Core。 選取環境時，請選擇命令列選項。 
2. 從命令列為專案建立目錄。 瀏覽到此目錄中，然後輸入 `dotnet new console -o <sample-project-name>` 以建立 C# 主控台專案。
3. 在 Visual Studio Code 中開啟此目錄。 您可以透過在 Windows 中於命令列輸入 `code .` 來快速完成此步驟。  
4. 從 Visual Studio Code Marketplace 安裝 [C# 擴充 (英文)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)。 重新啟動 Visual Studio Code。 
5. 這時，您應該會看到兩個提示。 其中一個是新增「建置和偵錯的必要資產。」 按一下 [是]。 另一個提示是還原無法解析的相依性。 按一下 [還原]。
6. 修改 `.vscode` 下的 `launch.json` 以將外部終端機作為主控台。 這項設定應讀取為 ` "console": "externalTerminal"`
7. Visual Studio Code 可讓您偵錯 .NET Core 應用程式。 按 `F5` 以執行應用程式，並確定您的設定運作正常。 您應該會看到「Hello World!」 印出到主控台。 

## <a name="add-data-movement-library-to-your-project"></a>將資料移動程式庫加入至專案

1. 將最新版本的資料移動程式庫加入到 `<project-name>.csproj` 檔案的 `dependencies` 區段。 撰寫本文時，此版本為 `"Microsoft.Azure.Storage.DataMovement": "0.6.2"` 
2. 您應該會看到提示顯示以還原專案。 按一下 [還原] 按鈕。 您也可以在專案目錄的根目錄中輸入命令 `dotnet restore`，來從命令列還原專案。

修改 `<project-name>.csproj`：

    <Project Sdk="Microsoft.NET.Sdk">

        <PropertyGroup>
            <OutputType>Exe</OutputType>
            <TargetFramework>netcoreapp2.0</TargetFramework>
        </PropertyGroup>
        <ItemGroup>
            <PackageReference Include="Microsoft.Azure.Storage.DataMovement" Version="0.6.2" />
            </ItemGroup>
        </Project>

## <a name="set-up-the-skeleton-of-your-application"></a>設定應用程式的基本架構
我們要做的第一件事是設定應用程式的「基本架構」程式碼。 此程式碼會提示我們輸入儲存體帳戶的名稱及帳戶金鑰，並使用該認證來建立 `CloudStorageAccount` 物件。 這個物件是用來在所有的傳輸案例中與儲存體帳戶互動。 該程式碼也會提示我們選擇要執行的傳輸作業類型。 

修改 `Program.cs`：

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Diagnostics;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.DataMovement;

namespace DMLibSample
{
    public class Program
    {
        public static void Main()
        {
            Console.WriteLine("Enter Storage account name:");           
            string accountName = Console.ReadLine();

            Console.WriteLine("\nEnter Storage account key:");           
            string accountKey = Console.ReadLine();

            string storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=" + accountName + ";AccountKey=" + accountKey;
            CloudStorageAccount account = CloudStorageAccount.Parse(storageConnectionString);

            ExecuteChoice(account);
        }

        public static void ExecuteChoice(CloudStorageAccount account)
        {
            Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
            int choice = int.Parse(Console.ReadLine());

            if(choice == 1)
            {
                TransferLocalFileToAzureBlob(account).Wait();
            }
            else if(choice == 2)
            {
                TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
            }
            else if(choice == 3)
            {
                TransferUrlToAzureBlob(account).Wait();
            }
            else if(choice == 4)
            {
                TransferAzureBlobToAzureBlob(account).Wait();
            }
        }

        public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
        { 
            
        }

        public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
        { 
            
        }

        public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
        {

        }
    }
}
```

## <a name="transfer-local-file-to-azure-blob"></a>將本機檔案傳輸至 Azure Blob
將方法 `GetSourcePath` 和 `GetBlob` 加入到 `Program.cs`：

```csharp
public static string GetSourcePath()
{
    Console.WriteLine("\nProvide path for source:");
    string sourcePath = Console.ReadLine();

    return sourcePath;
}

public static CloudBlockBlob GetBlob(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    Console.WriteLine("\nProvide name of new Blob:");
    string blobName = Console.ReadLine();
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    return blob;
}
```

修改 `TransferLocalFileToAzureBlob` 方法：

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{ 
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    await TransferManager.UploadAsync(localFilePath, blob);
    Console.WriteLine("\nTransfer operation complete.");
    ExecuteChoice(account);
}
```

此程式碼會提示我們輸入本機檔案的路徑、新的或現有容器的名稱，和新的 Blob 的名稱。 `TransferManager.UploadAsync` 方法會利用此資訊執行上傳。 

按 `F5` 以執行應用程式。 您可以使用 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)檢視儲存體帳戶，確認是否有上傳。

## <a name="set-number-of-parallel-operations"></a>設定平行作業的數目
資料移動程式庫提供的一項絕佳功能是設定平行作業的數目，以增加資料傳輸輸送量。 根據預設，資料移動程式庫會將平行作業的數目設為 8 * 您電腦的核心數目。 

請留意，在低頻寬環境執行大量的平行作業可能會拖垮網路連線，並造成作業無法完成。 您必須針對此設定進行實驗，以根據您的可用網路頻寬決定最佳的運作方式。 

讓我們加入一些程式碼，以便設定平行作業數目。 另外，也加入可用來計算傳輸完成所需時間的程式碼。

將 `SetNumberOfParallelOperations` 方法加入到 `Program.cs`：

```csharp
public static void SetNumberOfParallelOperations()
{
    Console.WriteLine("\nHow many parallel operations would you like to use?");
    string parallelOperations = Console.ReadLine();
    TransferManager.Configurations.ParallelOperations = int.Parse(parallelOperations);
}
```

修改 `ExecuteChoice` 方法以使用 `SetNumberOfParallelOperations`：

```csharp
public static void ExecuteChoice(CloudStorageAccount account)
{
    Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
    int choice = int.Parse(Console.ReadLine());

    SetNumberOfParallelOperations();

    if(choice == 1)
    {
        TransferLocalFileToAzureBlob(account).Wait();
    }
    else if(choice == 2)
    {
        TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
    }
    else if(choice == 3)
    {
        TransferUrlToAzureBlob(account).Wait();
    }
    else if(choice == 4)
    {
        TransferAzureBlobToAzureBlob(account).Wait();
    }
}
```

修改 `TransferLocalFileToAzureBlob` 方法以使用計時器：

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{ 
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="track-transfer-progress"></a>追蹤傳輸進度
能知道資料完成傳輸需要多少時間，是一件很不錯的事。 不過，能夠在傳輸作業期間看到傳輸的進度會更好。 為了達成此案例，我們需要建立 `TransferContext` 物件。 `TransferContext` 物件有兩種形式：`SingleTransferContext` 和 `DirectoryTransferContext`。 前者用於傳輸單一檔案 (也就是我們現在要做的)，而後者用於傳輸檔案的目錄 (我們將在稍後進行)。

將方法 `GetSingleTransferContext` 和 `GetDirectoryTransferContext` 加入到 `Program.cs`： 

```csharp
public static SingleTransferContext GetSingleTransferContext(TransferCheckpoint checkpoint)
{
    SingleTransferContext context = new SingleTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });
    
    return context;
}

public static DirectoryTransferContext GetDirectoryTransferContext(TransferCheckpoint checkpoint)
{
    DirectoryTransferContext context = new DirectoryTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });
    
    return context;
}
```

修改 `TransferLocalFileToAzureBlob` 方法以使用 `GetSingleTransferContext`：

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{ 
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    Console.WriteLine("\nTransfer started...\n");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob, null, context);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="resume-a-canceled-transfer"></a>繼續已取消的傳輸
資料移動程式庫提供的另一項方便功能，是能夠繼續已取消的傳輸。 讓我們加入一些程式碼，使我們能夠輸入 `c` 來暫時取消傳輸，然後在 3 秒之後繼續傳輸。

修改 `TransferLocalFileToAzureBlob`：

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{ 
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account); 
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint); 
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.UploadAsync(localFilePath, blob, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);  
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadAsync(localFilePath, blob, null, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

目前為止，`checkpoint` 的值一律是設為 `null`。 現在，如果我們取消傳輸，我們會擷取傳輸的最後一個檢查點，然後在新的傳輸內容中使用這個新的檢查點。 

## <a name="transfer-local-directory-to-azure-blob-directory"></a>將本機目錄傳輸到 Azure Blob 目錄
如果資料移動程式庫一次只能傳輸一個檔案，不免令人有些失望。 所幸，事實並非如此。 資料移動程式庫可以傳輸檔案目錄及其子目錄的所有內容。 讓我們加入一些程式碼來這麼做。

首先，將方法 `GetBlobDirectory` 加入 `Program.cs`：

```csharp
public static CloudBlobDirectory GetBlobDirectory(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container. This can be a new or existing Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    CloudBlobDirectory blobDirectory = container.GetDirectoryReference("");

    return blobDirectory;
}
```

然後，修改 `TransferLocalDirectoryToAzureBlobDirectory`：

```csharp
public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
{ 
    string localDirectoryPath = GetSourcePath();
    CloudBlobDirectory blobDirectory = GetBlobDirectory(account); 
    TransferCheckpoint checkpoint = null;
    DirectoryTransferContext context = GetDirectoryTransferContext(checkpoint); 
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    UploadDirectoryOptions options = new UploadDirectoryOptions()
    {
        Recursive = true
    };

    try
    {
        task = TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);  
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetDirectoryTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

這個方法與上傳單一檔案的方法有一些差異。 我們現在使用 `TransferManager.UploadDirectoryAsync` 和稍早建立的 `getDirectoryTransferContext` 方法。 此外，我們現在會提供 `options` 值給上傳作業，這可讓我們指出要在上傳中包含的子目錄。 

## <a name="copy-file-from-url-to-azure-blob"></a>將檔案從 URL 複製到 Azure Blob
現在，讓我們加入程式碼，以從 URL 將檔案複製到 Azure Blob。 

修改 `TransferUrlToAzureBlob`：

```csharp
public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
{
    Uri uri = new Uri(GetSourcePath());
    CloudBlockBlob blob = GetBlob(account); 
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint); 
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);  
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

這項功能的一個重要使用案例，是當您必須從另一個雲端服務 (例如 AWS) 將資料移動到 Azure 時。 只要您有可以存取資源的 URL，您就可以使用 `TransferManager.CopyAsync` 方法，輕鬆地將資源移動至 Azure Blob。 這個方法也引入新的布林值參數。 將此參數設定為 `true`，表示我們要執行非同步伺服器端複製。 將此參數設定為 `false` 表示同步複製，這代表資源會先下載到本機電腦，然後再上傳至 Azure Blob。 不過，同步複製目前僅適用於從某個 Azure 儲存體資源複製到另一個 Azure 儲存體。 

## <a name="transfer-azure-blob-to-azure-blob"></a>將 Azure Blob 傳輸至 Azure Blob
資料移動程式庫提供的另一項獨特功能，是可從某個 Azure 儲存體資源複製到另一個。 

修改 `TransferAzureBlobToAzureBlob`：

```csharp
public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
{
    CloudBlockBlob sourceBlob = GetBlob(account);
    CloudBlockBlob destinationBlob = GetBlob(account); 
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint); 
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(sourceBlob, destinationBlob, true, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);  
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(sourceBlob, destinationBlob, false, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

在此範例中，我們將 `TransferManager.CopyAsync` 中的布林值參數設為 `false`，指出我們要執行同步複製。 這表示資源會先下載到本機電腦，然後再上傳至 Azure Blob。 同步複製選項是確保複製作業有一致速度的絕佳方式。 相較之下，非同步伺服器端複製取決於伺服器的可用網路頻寬，速度可能會有所變動。 不過，相較於非同步複製，同步複製可能會產生額外的輸出成本。 建議的方法是在與您來源儲存體帳戶位於同一區域的 Azure VM 中使用同步複製，以避免產生輸出成本。

## <a name="conclusion"></a>結論
現在已經完成資料移動應用程式。 [您可在 GitHub 上取得完整的程式碼範例](https://github.com/azure-samples/storage-dotnet-data-movement-library-app)。 

## <a name="next-steps"></a>後續步驟
在此快速入門中，我們已建立可在 Windows、Linux 和 macOS 上執行並與 Azure 儲存體互動的應用程式。 此快速入門的重點在 Blob 儲存體。 不過，同樣的知識可套用至檔案儲存體。 若要深入了解，請參閱 [Azure 儲存體資料移動程式庫參考文件 (英文)](https://azure.github.io/azure-storage-net-data-movement)。

[!INCLUDE [storage-try-azure-tools-blobs](../../../includes/storage-try-azure-tools-blobs.md)]




