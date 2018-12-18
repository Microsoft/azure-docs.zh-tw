---
title: 使用 C++ 開發 Azure 檔案服務 | Microsoft Docs
description: 了解如何開發 C++ 應用程式和服務，以使用 Azure 檔案服務來儲存檔案資料。
services: storage
author: renashahmsft
ms.service: storage
ms.topic: article
ms.date: 09/19/2017
ms.author: renashahmsft
ms.component: files
ms.openlocfilehash: 4543784c1de85cf86f3dfa912a33641541e0373a
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529695"
---
# <a name="develop-for-azure-files-with-c"></a>使用 C++ 開發 Azure 檔案服務
[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

[!INCLUDE [storage-try-azure-tools-files](../../../includes/storage-try-azure-tools-files.md)]

## <a name="about-this-tutorial"></a>關於本教學課程
在本教學課程中，您將學習如何執行 Azure 檔案服務的基本作業。 透過以 C++ 撰寫的範例，您將學習如何建立共用和目錄、上傳、列出及刪除檔案。 如果您是 Azure 檔案服務的新手，閱讀下列各節中的概念對於了解範例會很有幫助。

* 建立及刪除 Azure 檔案共用
* 建立及刪除目錄
* 列舉 Azure 檔案共用的檔案和目錄
* 上傳、下載及刪除檔案
* 設定 Azure 檔案共用的配額 (大小上限)
* 為使用共用上所定義之共用存取原則的檔案建立共用存取簽章 (SAS 金鑰)。

> [!Note]  
> 由於 Azure 檔案服務可透過 SMB 存取，因此您可以使用標準 C++ I/O 類別和函式撰寫簡單的應用程式，以存取 Azure 檔案共用。 本文將說明如何撰寫使用 Azure 儲存體 C++ SDK 的應用程式，其會使用 [File REST API](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api) 與 Azure 檔案服務通訊。

## <a name="create-a-c-application"></a>建立 C++ 應用程式
若要建置範例，您必須安裝適用於 C++ 的 Azure 儲存體用戶端程式庫 2.4.0。 您也應該建立 Azure 儲存體帳戶。

若要安裝適用於 C++ 的 Azure 儲存體用戶端 2.4.0，您可以使用下列其中一個方法：

* **Linux：** 遵循 [Azure Storage Client Library for C++ 讀我檔案](https://github.com/Azure/azure-storage-cpp/blob/master/README.md) 頁面中提供的指示進行。
* **Windows：** 在 Visual Studio 中，按一下 [工具]&gt;[NuGet 套件管理員]&gt;[套件管理員主控台]。 在 [NuGet 套件管理員主控台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console) 中輸入下列命令，然後按下 **Enter**。
  
```
Install-Package wastorage
```

## <a name="set-up-your-application-to-use-azure-files"></a>設定您的應用程式以使用 Azure 檔案服務
在您要用來管理 Azure 檔案服務的 C++ 來源檔案頂端，新增下列 include 陳述式：

```cpp
#include <was/storage_account.h>
#include <was/file.h>
```

## <a name="set-up-an-azure-storage-connection-string"></a>設定 Azure 儲存體連接字串
若要使用檔案儲存體，您必須連接到您的 Azure 儲存體帳戶。 第一個步驟是設定連接字串，我們將用它來連線至您的儲存體帳戶。 讓我們定義靜態變數以便進行。

```cpp
// Define the connection-string with your values.
const utility::string_t 
storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

## <a name="connecting-to-an-azure-storage-account"></a>連接到 Azure 儲存體帳戶
您可以使用 **cloud_storage_account** 類別來代表儲存體帳戶資訊。 若要從儲存體連接字串擷取儲存體帳戶資訊，您可以使用 **parse** 方法。

```cpp
// Retrieve storage account from connection string.    
azure::storage::cloud_storage_account storage_account = 
  azure::storage::cloud_storage_account::parse(storage_connection_string);
```

## <a name="create-an-azure-file-share"></a>建立 Azure 檔案共用
Azure 檔案共用中的所有檔案和目錄都位於名為 **Share** 的容器中。 您的儲存體帳戶可以有帳戶容量允許數量的共用。 若要取得共用及其內容的存取權，您必須使用 Azure 檔案服務用戶端。

```cpp
// Create the Azure Files client.
azure::storage::cloud_file_client file_client = 
  storage_account.create_cloud_file_client();
```

使用 Azure 檔案服務用戶端時，您即可取得共用的參考。

```cpp
// Get a reference to the file share
azure::storage::cloud_file_share share = 
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
```

若要建立共用，請使用 **cloud_file_share** 物件的 **create_if_not_exists** 方法。

```cpp
if (share.create_if_not_exists()) {    
    std::wcout << U("New share created") << std::endl;    
}
```

目前，**共用**會將參考保留至名為 **my-sample-share** 的共用。

## <a name="delete-an-azure-file-share"></a>刪除 Azure 檔案共用
刪除共用可以藉由在 cloud_file_share 物件上呼叫 **delete_if_exists** 方法來完成。 以下是執行該作業的範例程式碼。

```cpp
// Get a reference to the share.
azure::storage::cloud_file_share share = 
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

// delete the share if exists
share.delete_share_if_exists();
```

## <a name="create-a-directory"></a>建立目錄
您可以組織儲存體，方法是將檔案放在子目錄中，而不是將所有檔案都放在根目錄中。 Azure 檔案服務可讓您建立帳戶允許數量的目錄。 下列程式碼會在根目錄底下建立名為 **my-sample-directory** 的目錄，以及名為 **my-sample-subdirectory** 的子目錄。

```cpp
// Retrieve a reference to a directory
azure::storage::cloud_file_directory directory = share.get_directory_reference(_XPLATSTR("my-sample-directory"));

// Return value is true if the share did not exist and was successfully created.
directory.create_if_not_exists();

// Create a subdirectory.
azure::storage::cloud_file_directory subdirectory = 
  directory.get_subdirectory_reference(_XPLATSTR("my-sample-subdirectory"));
subdirectory.create_if_not_exists();
```

## <a name="delete-a-directory"></a>刪除目錄
刪除目錄是一項簡單的工作，不過請注意您無法刪除仍然包含檔案或其他目錄的目錄。

```cpp
// Get a reference to the share.
azure::storage::cloud_file_share share = 
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

// Get a reference to the directory.
azure::storage::cloud_file_directory directory = 
  share.get_directory_reference(_XPLATSTR("my-sample-directory"));

// Get a reference to the subdirectory you want to delete.
azure::storage::cloud_file_directory sub_directory =
  directory.get_subdirectory_reference(_XPLATSTR("my-sample-subdirectory"));

// Delete the subdirectory and the sample directory.
sub_directory.delete_directory_if_exists();

directory.delete_directory_if_exists();
```

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>列舉 Azure 檔案共用的檔案和目錄
取得共用內檔案和目錄的清單很容易，只要在 **cloud_file_directory** 參考上呼叫 **list_files_and_directories** 即可。 若要存取傳回的 **list_file_and_directory_item** 中豐富的屬性和方法，您必須呼叫 **list_file_and_directory_item.as_file** 方法來取得 **cloud_file** 物件，或呼叫 **list_file_and_directory_item.as_directory** 方法來取得 **cloud_file_directory** 物件。

下列程式碼示範如何擷取和輸出共用之根目錄中每個項目的 URI。

```cpp
//Get a reference to the root directory for the share.
azure::storage::cloud_file_directory root_dir = 
  share.get_root_directory_reference();

// Output URI of each item.
azure::storage::list_file_and_diretory_result_iterator end_of_results;

for (auto it = directory.list_files_and_directories(); it != end_of_results; ++it)
{
    if(it->is_directory())
    {
        ucout << "Directory: " << it->as_directory().uri().primary_uri().to_string() << std::endl;
    }
    else if (it->is_file())
    {
        ucout << "File: " << it->as_file().uri().primary_uri().to_string() << std::endl;
    }        
}
```

## <a name="upload-a-file"></a>上傳檔案
Azure 檔案共用至少包含根目錄，檔案可以放置其中。 在本節中，您將學習如何從本機儲存體將檔案上傳至共用的根目錄。

上傳檔案的第一個步驟是取得檔案所在之目錄的參考。 您可以藉由呼叫共用物件的 **get_root_directory_reference** 方法來完成此作業。

```cpp
//Get a reference to the root directory for the share.
azure::storage::cloud_file_directory root_dir = share.get_root_directory_reference();
```

現在您具有共用之根目錄的參考，您可以對其上傳檔案。 此範例會從檔案、文字和串流進行上傳。

```cpp
// Upload a file from a stream.
concurrency::streams::istream input_stream = 
  concurrency::streams::file_stream<uint8_t>::open_istream(_XPLATSTR("DataFile.txt")).get();

azure::storage::cloud_file file1 = 
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-1"));
file1.upload_from_stream(input_stream);

// Upload some files from text.
azure::storage::cloud_file file2 = 
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-2"));
file2.upload_text(_XPLATSTR("more text"));

// Upload a file from a file.
azure::storage::cloud_file file4 = 
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));
file4.upload_from_file(_XPLATSTR("DataFile.txt"));    
```

## <a name="download-a-file"></a>下載檔案
若要下載檔案，請先擷取檔案參考，然後呼叫 **download_to_stream** 方法將檔案內容傳輸到您可接著保留在本機檔案的串流物件。 或者，您可以使用 **download_to_file** 方法，將檔案內容下載到本機檔案。 您可以使用 **download_text** 方法，將檔案內容當成文字字串下載。

下列範例使用 **download_to_stream** 和 **download_text** 方法，示範如何下載前幾節中所建立的檔案。

```cpp
// Download as text
azure::storage::cloud_file text_file = 
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-2"));
utility::string_t text = text_file.download_text();
ucout << "File Text: " << text << std::endl;

// Download as a stream.
azure::storage::cloud_file stream_file = 
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));

concurrency::streams::container_buffer<std::vector<uint8_t>> buffer;
concurrency::streams::ostream output_stream(buffer);
stream_file.download_to_stream(output_stream);
std::ofstream outfile("DownloadFile.txt", std::ofstream::binary);
std::vector<unsigned char>& data = buffer.collection();
outfile.write((char *)&data[0], buffer.size());
outfile.close();
```

## <a name="delete-a-file"></a>刪除檔案
另一個常見的 Azure 檔案服務作業是刪除檔案。 下列程式碼會刪除名為 my-sample-file-3 且儲存在根目錄底下的檔案。

```cpp
// Get a reference to the root directory for the share.    
azure::storage::cloud_file_share share = 
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

azure::storage::cloud_file_directory root_dir = 
  share.get_root_directory_reference();

azure::storage::cloud_file file = 
  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));

file.delete_file_if_exists();
```

## <a name="set-the-quota-maximum-size-for-an-azure-file-share"></a>設定 Azure 檔案共用的配額 (大小上限)
您可以設定檔案共用的配額 (或大小上限)，以 GB 為單位。 您也可以檢查有多少資料目前儲存在共用上。

藉由設定共用的配額，您可以限制儲存在共用上的檔案大小總計。 如果共用上的檔案大小總計超過共用上設定的配額，則用戶端將無法增加現有檔案的大小或建立新的檔案 (除非這些檔案為空白)。

下列範例示範如何檢查共用的目前使用狀況，以及如何設定共用的配額。

```cpp
// Parse the connection string for the storage account.
azure::storage::cloud_storage_account storage_account = 
  azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the file client.
azure::storage::cloud_file_client file_client = 
  storage_account.create_cloud_file_client();

// Get a reference to the share.
azure::storage::cloud_file_share share = 
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
if (share.exists())
{
    std::cout << "Current share usage: " << share.download_share_usage() << "/" << share.properties().quota();

    //This line sets the quota to 2560GB
    share.resize(2560);

    std::cout << "Quota increased: " << share.download_share_usage() << "/" << share.properties().quota();

}
```

## <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>產生檔案或檔案共用的共用存取簽章
您可以為檔案共用或個別檔案產生共用存取簽章 (SAS)。 您也可以在檔案共用上建立共用存取原則，以管理共用存取簽章。 建議您建立共用存取原則，因為如果必須洩漏 SAS，它提供了一種撤銷 SAS 的方式。

下列範例會在共用上建立共用存取原則，然後使用該原則為共用中檔案上的 SAS 提供條件約束。

```cpp
// Parse the connection string for the storage account.
azure::storage::cloud_storage_account storage_account = 
  azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the file client and get a reference to the share
azure::storage::cloud_file_client file_client = 
  storage_account.create_cloud_file_client();

azure::storage::cloud_file_share share = 
  file_client.get_share_reference(_XPLATSTR("my-sample-share"));

if (share.exists())
{
    // Create and assign a policy
    utility::string_t policy_name = _XPLATSTR("sampleSharePolicy");

    azure::storage::file_shared_access_policy sharedPolicy = 
      azure::storage::file_shared_access_policy();

    //set permissions to expire in 90 minutes
    sharedPolicy.set_expiry(utility::datetime::utc_now() + 
       utility::datetime::from_minutes(90));

    //give read and write permissions
    sharedPolicy.set_permissions(azure::storage::file_shared_access_policy::permissions::write | azure::storage::file_shared_access_policy::permissions::read);

    //set permissions for the share
    azure::storage::file_share_permissions permissions;    

    //retrieve the current list of shared access policies
    azure::storage::shared_access_policies<azure::storage::file_shared_access_policy> policies;

    //add the new shared policy
    policies.insert(std::make_pair(policy_name, sharedPolicy));

    //save the updated policy list
    permissions.set_policies(policies);
    share.upload_permissions(permissions);

    //Retrieve the root directory and file references
    azure::storage::cloud_file_directory root_dir = 
        share.get_root_directory_reference();
    azure::storage::cloud_file file = 
      root_dir.get_file_reference(_XPLATSTR("my-sample-file-1"));

    // Generate a SAS for a file in the share 
    //  and associate this access policy with it.        
    utility::string_t sas_token = file.get_shared_access_signature(sharedPolicy);

    // Create a new CloudFile object from the SAS, and write some text to the file.        
    azure::storage::cloud_file file_with_sas(azure::storage::storage_credentials(sas_token).transform_uri(file.uri().primary_uri()));
    utility::string_t text = _XPLATSTR("My sample content");        
    file_with_sas.upload_text(text);        

    //Download and print URL with SAS.
    utility::string_t downloaded_text = file_with_sas.download_text();        
    ucout << downloaded_text << std::endl;        
    ucout << azure::storage::storage_credentials(sas_token).transform_uri(file.uri().primary_uri()).to_string() << std::endl;

}
```
## <a name="next-steps"></a>後續步驟
如需深入了解 Azure 儲存體，請探索這些資源：

* [Storage Client Library for C++](https://github.com/Azure/azure-storage-cpp)
* [C++ 的 Azure 儲存體檔案服務範例] (https://github.com/Azure-Samples/storage-file-cpp-getting-started)
* [Azure 儲存體總管](http://go.microsoft.com/fwlink/?LinkID=822673&clcid=0x409)
* [Azure 儲存體文件](https://azure.microsoft.com/documentation/services/storage/)