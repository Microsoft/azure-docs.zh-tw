---
title: Azure 快速入門 - 使用 Ruby 在物件儲存體中建立 Blob | Microsoft Docs
description: 在本快速入門中，您會在物件 (Blob) 儲存體中建立儲存體帳戶和容器。 然後，使用 Ruby 的儲存體用戶端程式庫將 blob 上傳至 Azure 儲存體、下載 blob，以及列出容器中的 blob。
services: storage
author: tamram
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 04/09/2018
ms.author: seguler
ms.openlocfilehash: 88394d7da1aab52b752aee68de60e638d0c7e7f0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992757"
---
# <a name="quickstart-upload-download-and-list-blobs-using-ruby"></a>快速入門：使用 Ruby 上傳、下載及列出 Blob

在本快速入門中，您會了解如何使用 Ruby 在 Azure Blob 儲存體容器中上傳、下載及列出區塊 Blob。 

## <a name="prerequisites"></a>必要條件

若要完成本快速入門： 
* 安裝 [Ruby](https://www.ruby-lang.org/en/downloads/)
* 使用 rubygem 套件來安裝[適用於 Ruby 的 Azure 儲存體程式庫](https://docs.microsoft.com/azure/storage/blobs/storage-ruby-how-to-use-blob-storage#configure-your-application-to-access-storage)。 

```
gem install azure-storage-blob
```

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="download-the-sample-application"></a>下載範例應用程式
本快速入門中使用的[範例應用程式](https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git)是基本的 Ruby 應用程式。  

使用 [git](https://git-scm.com/) 將應用程式的複本下載至您的開發環境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git 
```

此命令會將存放庫複製到本機的 git 資料夾。 若要開啟 Ruby 範例應用程式，請尋找 storage-blobs-ruby-quickstart 資料夾，並開啟 example.rb 檔案。  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串
在應用程式中，您必須提供儲存體帳戶名稱和帳戶金鑰，才能建立應用程式的 `BlobService` 執行個體。 從 IDE 中的方案總管開啟 `example.rb` 檔案。 使用您的帳戶名稱和金鑰取代 **accountname** 和 **accountkey**。 

```ruby 
blob_client = Azure::Storage::Blob::BlobService.create(
            storage_account_name: account_name,
            storage_access_key: account_key
          )
```

## <a name="run-the-sample"></a>執行範例
這個範例會在 'Documents' 資料夾中建立測試檔案。 範例程式會將測試檔案上傳至 Blob 儲存體、列出容器中的 Blob，並下載具有新名稱的檔案。 

執行範例。 下列輸出是執行應用程式時所傳回的輸出範例：
  
```
Creating a container: quickstartblobs7b278be3-a0dd-438b-b9cc-473401f0c0e8

Temp file = C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

Uploading to Blob storage as blobQuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

List blobs in the container
         Blob name: QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

Downloading blob to C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078_DOWNLOADED.txt
```
當您按任意鍵繼續時，範例程式會刪除儲存體容器和檔案。 在繼續之前，請檢查 'Documents' 資料夾，找出這兩個檔案。 您可以開啟它們，並查看它們是否相同。

您也可以使用 [Azure 儲存體總管](http://storageexplorer.com) 之類的工具來檢視 Blob 儲存體中的檔案。 Azure 儲存體總管是免費的跨平台工具，可讓您存取儲存體帳戶資訊。 

確認檔案之後，請按任一鍵以完成示範並刪除測試檔案。 現在您已知道這個範例的功用，請開啟 example.rb 檔案查看程式碼。 

## <a name="understand-the-sample-code"></a>了解範例程式碼

接下來，我們將透過範例程式碼來了解其運作方式。

### <a name="get-references-to-the-storage-objects"></a>取得儲存體物件的參考
第一件事是建立用來存取和管理 Blob 儲存體的物件參考。 這些物件是互為建置基礎，各自都為清單中的下一個物件所使用。

* 建立 Azure 儲存體 **BlobService** 物件的執行個體來設定連線認證。 
* 建立 **Container** 物件，它代表您要存取的容器。 容器是用來組織 Blob，就像在電腦上用資料夾組織檔案一樣。

一旦有了雲端 Blob 容器，您就可以建立**區塊** Blob 物件，指向您感興趣的特定 Blob，並執行上傳、下載和複製等作業。

> [!IMPORTANT]
> 容器名稱必須是小寫字母。 如需有關容器和 Blob 名稱的詳細資訊，請參閱[命名和參考容器、Blob 及中繼資料](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

在本節中，您可以設定 Azure 儲存體用戶端的執行個體、具現化 blob 服務物件、建立新的容器，然後設定容器上的權限，以便這些 Blob 為公用 Blob。 容器名為 **quickstartblobs**。 

```ruby 
# Create a BlobService object
blob_client = Azure::Storage::Blob::BlobService.create(
    storage_account_name: account_name,
    storage_access_key: account_key
    )

# Create a container called 'quickstartblobs'.
container_name ='quickstartblobs' + SecureRandom.uuid
container = blob_client.create_container(container_name)   

# Set the permission so the blobs are public.
blob_client.set_container_acl(container_name, "container")
```

### <a name="upload-blobs-to-the-container"></a>將 Blob 上傳到容器

Blob 儲存體支援區塊 Blob、附加 Blob 和分頁 Blob。 最常使用的是區塊 Blob，這也是本快速入門中所使用的。  

若要將檔案上傳至 Blob，請加入本機磁碟機上的目錄名稱和檔案名稱，取得檔案的完整路徑。 然後，您可以使用 **create\_block\_blob()** 方法，將檔案上傳至指定的路徑。 

範例程式碼會建立用於上傳和下載的本機檔案，同時將要上傳的檔案儲存為 **file\_path\_to\_file**，並將 Blob 的名稱儲存為 **local\_file\_name**。 下列範例會將檔案上傳到名為 **quickstartblobs** 的容器。

```ruby
# Create a file in Documents to test the upload and download.
local_path=File.expand_path("~/Documents")
local_file_name ="QuickStart_" + SecureRandom.uuid + ".txt"
full_path_to_file =File.join(local_path, local_file_name)

# Write text to the file.
file = File.open(full_path_to_file,  'w')
file.write("Hello, World!")
file.close()

puts "Temp file = " + full_path_to_file
puts "\nUploading to Blob storage as blob" + local_file_name

# Upload the created file, using local_file_name for the blob name
blob_client.create_block_blob(container.name, local_file_name, full_path_to_file)
```

若要執行區塊 blob 的部分更新內容，請使用 **create\_block\_list()** 方法。 區塊 Blob 可以大到 4.7 TB，而且可以是 Excel 試算表到大型視訊檔案的任何一種。 分頁 Blob 主要用於備份 IaaS VM 所用的 VHD 檔案。 附加 Blob 用於記錄，例如當您想要寫入檔案，並繼續新增更多資訊時。 附加 blob 應該用於單一寫入模式。 儲存在 Blob 儲存體中的大部分物件都是區塊 Blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

您可以使用 **list\_blobs()** 方法，取得容器中的檔案清單。 下列程式碼會擷取 Blob 的清單，然後透過它們執行迴圈，顯示在容器中找到的 Blob 名稱。  

```ruby
# List the blobs in the container
nextMarker = nil
loop do
    blobs = blob_client.list_blobs(container_name, { marker: nextMarker })
    blobs.each do |blob|
        puts "\tBlob name #{blob.name}"
    end
    nextMarker = blobs.continuation_token
    break unless nextMarker && !nextMarker.empty?
end
```

### <a name="download-the-blobs"></a>下載 Blob

使用 **get\_blob()** 方法，將 Blob 下載至本機磁碟。 下列程式碼會下載前一節中上傳的 Blob。 "_DOWNLOADED" 會新增為 Blob 名稱的尾碼，讓您可在本機磁碟上看到這兩個檔案。 

```ruby
# Download the blob(s).
# Add '_DOWNLOADED' as prefix to '.txt' so you can see both files in Documents.
full_path_to_file2 = File.join(local_path, local_file_name.gsub('.txt', '_DOWNLOADED.txt'))

puts "\n Downloading blob to " + full_path_to_file2
blob, content = blob_client.get_blob(container_name,local_file_name)
File.open(full_path_to_file2,"wb") {|f| f.write(content)}
```

### <a name="clean-up-resources"></a>清除資源
如果不再需要本快速入門中上傳的 Blob，可以使用 **delete\_container()** 刪除整個容器。 如果不再需要建立的檔案，請使用 **delete\_blob()** 方法來刪除檔案。

```ruby
# Clean up resources. This includes the container and the temp files
blob_client.delete_container(container_name)
File.delete(full_path_to_file)
File.delete(full_path_to_file2)    
```
## <a name="resources-for-developing-ruby-applications-with-blobs"></a>可供使用 Blob 開發 Ruby 應用程式的資源

請參閱以下可供使用 Blob 儲存體進行 Ruby 開發的額外資源：

- 檢視及下載 GitHub 上適用於 Azure 儲存體的 [Ruby 用戶端程式庫原始程式碼](https://github.com/Azure/azure-storage-ruby)。
- 探索使用 Ruby 用戶端程式庫所撰寫的 [Blob 儲存體範例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=ruby&term=blob)。

## <a name="next-steps"></a>後續步驟
 
在此快速入門中，您已了解如何使用 Ruby 在本機磁碟和 Azure Blob 儲存體之間傳送檔案。 若要深入了解 Blob 儲存體的用法，請繼續閱讀 Blob 儲存體操作說明。

> [!div class="nextstepaction"]
> [Blob 儲存體作業操作說明](./storage-ruby-how-to-use-blob-storage.md)


如需儲存體總管和 Blob 的詳細資訊，請參閱[使用儲存體總管管理 Azure Blob 儲存體資源](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
