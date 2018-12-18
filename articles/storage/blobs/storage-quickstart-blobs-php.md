---
title: Azure 快速入門 - 使用 PHP 在物件儲存體中建立 Blob | Microsoft Docs
description: 快速了解使用 PHP 在 Azure Blob 儲存體之間傳送物件
services: storage
author: roygara
ms.service: storage
ms.devlang: php
ms.topic: quickstart
ms.date: 04/09/2018
ms.author: rogarana
ms.openlocfilehash: b0a562bd37c1877651863525241d70990d38ded2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969589"
---
#  <a name="transfer-objects-tofrom-azure-blob-storage-using-php"></a>使用 PHP 在 Azure Blob 儲存體之間傳送物件
在本快速入門中，您會了解如何使用 PHP 在 Azure Blob 儲存體容器中上傳、下載及列出區塊 Blob。 

## <a name="prerequisites"></a>必要條件

若要完成本快速入門： 
* 安裝 [PHP](http://php.net/downloads.php)
* 安裝 [Azure Storage SDK for PHP](https://github.com/Azure/azure-storage-php)


如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="download-the-sample-application"></a>下載範例應用程式
本快速入門中使用的[範例應用程式](https://github.com/Azure-Samples/storage-blobs-php-quickstart.git)是基本的 PHP 應用程式。  

使用 [git](https://git-scm.com/) 將應用程式的複本下載至您的開發環境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-php-quickstart.git
```

此命令會將存放庫複製到本機的 git 資料夾。 若要開啟 PHP 範例應用程式，請尋找 storage-blobs-php-quickstart 資料夾，然後開啟 phpqs.php 檔案。  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串
在應用程式中，您必須提供儲存體帳戶名稱和帳戶金鑰，才能建立應用程式的 **BlobRestProxy** 執行個體。 建議您在執行應用程式的本機電腦上，將這些識別碼儲存在環境變數內。 請根據您的作業系統，使用以下其中一個範例來建立環境變數。 使用您的帳戶名稱和金鑰取代 **youraccountname** 和 **youraccountkey** 值。

# <a name="linux-tablinux"></a>[Linux] (#tab/linux)

```bash
export ACCOUNT_NAME=<youraccountname>
export ACCOUNT_KEY=<youraccountkey>
```

# <a name="windows-tabwindows"></a>[Windows] (#tab/windows)

```cmd
setx ACCOUNT_NAME=<youraccountname>
setx ACCOUNT_KEY=<youraccountkey>
```
---

## <a name="configure-your-environment"></a>設定您的環境
從本機 git 資料夾取得資料夾，並將它放在您的 PHP 伺服器所提供的目錄中。 然後，開啟範圍限於該相同目錄的命令提示字元並輸入：`php composer.phar install`

## <a name="run-the-sample"></a>執行範例
這個範例會在 '.' 資料夾中建立測試檔案。 範例程式會將測試檔案上傳至 Blob 儲存體、列出容器中的 Blob，並下載具有新名稱的檔案。 

執行範例。 下列輸出是執行應用程式時所傳回的輸出範例：
  
```
Uploading BlockBlob: HelloWorld.txt
These are the blobs present in the container: HelloWorld.txt: https://myexamplesacct.blob.core.windows.net/blockblobsleqvxd/HelloWorld.txt

This is the content of the blob uploaded: Hello Azure!
```
當您按所顯示的按鈕時，範例程式會刪除儲存體容器和檔案。 在繼續之前，請檢查您伺服器的資料夾，找出這兩個檔案。 您可以開啟它們，並查看它們是否相同。

您也可以使用 [Azure 儲存體總管](http://storageexplorer.com) 之類的工具來檢視 Blob 儲存體中的檔案。 Azure 儲存體總管是免費的跨平台工具，可讓您存取儲存體帳戶資訊。 

確認檔案之後，請按任一鍵以完成示範並刪除測試檔案。 現在您已知道這個範例的功用，請開啟 example.rb 檔案查看程式碼。 

## <a name="understand-the-sample-code"></a>了解範例程式碼

接下來，我們將透過範例程式碼來了解其運作方式。

### <a name="get-references-to-the-storage-objects"></a>取得儲存體物件的參考
第一件事是建立用來存取和管理 Blob 儲存體的物件參考。 這些物件是互為建置基礎，各自都為清單中的下一個物件所使用。

* 建立 Azure 儲存體 **BlobRestProxy** 物件的執行個體來設定連線認證。 
* 建立 **BlobService** 物件，它會指向儲存體帳戶中的 Blob 服務。 
* 建立 **Container** 物件，它代表您要存取的容器。 容器是用來組織 Blob，就像在電腦上用資料夾組織檔案一樣。

一旦有了 **blobClient** 容器物件，您就可以建立 **Block** Blob 物件，以指向您感興趣的特定 Blob。 您可接著執行上傳、下載和複製等作業。

> [!IMPORTANT]
> 容器名稱必須是小寫字母。 如需有關容器和 Blob 名稱的詳細資訊，請參閱[命名和參考容器、Blob 及中繼資料](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

在本節中，您可以設定 Azure 儲存體用戶端的執行個體、具現化 blob 服務物件、建立新的容器，以及設定容器上的權限，以便這些 Blob 為公用 Blob。 容器名為 **quickstartblobs**。 

```PHP
    # Setup a specific instance of an Azure::Storage::Client
    $connectionString = "DefaultEndpointsProtocol=https;AccountName=".getenv('account_name').";AccountKey=".getenv('account_key');
    
    // Create blob client.
    $blobClient = BlobRestProxy::createBlobService($connectionString);
    
    # Create the BlobService that represents the Blob service for the storage account
    $createContainerOptions = new CreateContainerOptions();
    
    $createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);
    
    // Set container metadata.
    $createContainerOptions->addMetaData("key1", "value1");
    $createContainerOptions->addMetaData("key2", "value2");

    $containerName = "blockblobs".generateRandomString();

    try    {
        // Create container.
        $blobClient->createContainer($containerName, $createContainerOptions);
```

### <a name="upload-blobs-to-the-container"></a>將 Blob 上傳到容器

Blob 儲存體支援區塊 Blob、附加 Blob 和分頁 Blob。 最常使用的是區塊 Blob，這也是本快速入門中所使用的。  

若要將檔案上傳至 Blob，請加入本機磁碟機上的目錄名稱和檔案名稱，取得檔案的完整路徑。 然後，您可以使用 **createBlockBlob()** 方法，將檔案上傳至指定的路徑。 

範例程式碼會取得本機檔案並將它上傳至 Azure。 此檔案會在程式碼中儲存為 **myfile**，而 Blob 的名稱會儲存為 **fileToUpload**。 下列範例會將檔案上傳到名為 **quickstartblobs** 的容器。

```PHP
    $myfile = fopen("HelloWorld.txt", "w") or die("Unable to open file!");
    fclose($myfile);

    # Upload file as a block blob
    echo "Uploading BlockBlob: ".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    
    $content = fopen($fileToUpload, "r");

    //Upload blob
    $blobClient->createBlockBlob($containerName, $fileToUpload, $content);
```

若要執行區塊 blob 的部分更新內容，請使用 **createblocklist()** 方法。 區塊 Blob 可以大到 4.7 TB，而且可以是 Excel 試算表到大型視訊檔案的任何一種。 分頁 Blob 主要用於備份 IaaS VM 所用的 VHD 檔案。 附加 Blob 用於記錄，例如當您想要寫入檔案，並繼續新增更多資訊時。 附加 blob 應該用於單一寫入模式。 儲存在 Blob 儲存體中的大部分物件都是區塊 Blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

您可以使用 **listBlobs()** 方法，取得容器中的檔案清單。 下列程式碼會擷取 Blob 的清單，然後透過它們執行迴圈，顯示在容器中找到的 Blob 名稱。  

```PHP
    $listBlobsOptions = new ListBlobsOptions();
    $listBlobsOptions->setPrefix("HelloWorld");
    
    echo "These are the blobs present in the container: ";
    
    do{
        $result = $blobClient->listBlobs($containerName, $listBlobsOptions);
        foreach ($result->getBlobs() as $blob)
        {
            echo $blob->getName().": ".$blob->getUrl()."<br />";
        }
        
        $listBlobsOptions->setContinuationToken($result->getContinuationToken());
    } while($result->getContinuationToken());
```

### <a name="get-the-content-of-your-blobs"></a>取得 Blob 的內容

使用 **getBlob()** 方法來取得 blob 的內容。 下列程式碼會顯示前一節中上傳的 Blob 內容。

```PHP
    $blob = $blobClient->getBlob($containerName, $fileToUpload);
    fpassthru($blob->getContentStream());
```

### <a name="clean-up-resources"></a>清除資源
如果不再需要本快速入門中上傳的 Blob，可以使用 **deleteContainer()** 刪除整個容器。 如果不再需要建立的檔案，請使用 **deleteBlob()** 方法來刪除檔案。

```PHP
    // Delete blob.
    echo "Deleting Blob".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    $blobClient->deleteBlob($_GET["containerName"], $fileToUpload);

    // Delete container.
    echo "Deleting Container".PHP_EOL;
    echo $_GET["containerName"].PHP_EOL;
    echo "<br />";
    $blobClient->deleteContainer($_GET["containerName"]);

    //Deleting local file
    echo "Deleting file".PHP_EOL;
    echo "<br />";
    unlink($fileToUpload);   
```

## <a name="resources-for-developing-php-applications-with-blobs"></a>可供使用 Blob 開發 PHP 應用程式的資源

請參閱以下可供使用 Blob 儲存體進行 PHP 開發的額外資源：

- 檢視、下載及安裝 GitHub 上適用於 Azure 儲存體的 [PHP 用戶端程式庫原始程式碼](https://github.com/Azure/azure-storage-php)。
- 探索使用 PHP 用戶端程式庫所撰寫的 [Blob 儲存體範例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=php&term=blob)。

## <a name="next-steps"></a>後續步驟
 
在此快速入門中，您已了解如何使用 PHP 在本機磁碟和 Azure Blob 儲存體之間傳輸檔案。 若要深入了解 PHP 的用法，請繼續使用我們的 PHP 開發人員中心。

> [!div class="nextstepaction"]
> [PHP 開發人員中心](https://azure.microsoft.com/develop/php/)


如需儲存體總管和 Blob 的詳細資訊，請參閱[使用儲存體總管管理 Azure Blob 儲存體資源](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
