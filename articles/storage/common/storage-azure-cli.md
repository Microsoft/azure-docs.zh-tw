---
title: 使用 Azure CLI 搭配 Azure 儲存體 | Microsoft Docs
description: 了解如何搭配 Azure 儲存體使用 Azure 命令列介面 (Azure CLI) 來建立和管理儲存體帳戶，以及使用 Azure Blob 和檔案。
services: storage
author: roygara
ms.service: storage
ms.devlang: azurecli
ms.topic: article
ms.date: 06/02/2017
ms.author: rogarana
ms.component: common
ms.openlocfilehash: cd2399e25889cdc9c885b76e002e47415c0629e5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984367"
---
# <a name="using-the-azure-cli-with-azure-storage"></a>使用 Azure CLI 搭配 Azure 儲存體

開放原始碼、跨平台的 Azure CLI 提供一組命令，供您使用 Azure 平台。 它提供許多與 [Azure 入口網站](https://portal.azure.com)相同的功能，包括豐富的資料存取功能。

在此指南中，我們會說明如何使用 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2) 來執行數種工作，以便使用您 Azure 儲存體帳戶中的資源。 建議您在使用此指南之前，先下載並安裝或升級到最新版的 CLI。

本指南中的範例假設在 Ubuntu 上使用 Bash 殼層，但其他平台應以類似方式執行。 

[!INCLUDE [storage-cli-versions](../../../includes/storage-cli-versions.md)]

## <a name="prerequisites"></a>必要條件
本指南假設您已了解 Azure 儲存體的基本概念。 而且假設您可以滿足針對 Azure 和儲存體服務所指定的帳戶建立需求。

### <a name="accounts"></a>帳戶
* **Azure 帳戶**：如果您沒有 Azure 訂用帳戶，請[建立免費的 Azure 帳戶](https://azure.microsoft.com/free/)。
* **儲存體帳戶**：請參閱[關於 Azure 儲存體帳戶](storage-create-storage-account.md)中的[建立儲存體帳戶](storage-quickstart-create-account.md)。

### <a name="install-the-azure-cli"></a>安裝 Azure CLI

依照[安裝 Azure CLI](/cli/azure/install-az-cli2) 中的指示下載並安裝 Azure CLI。

> [!TIP]
> 如果您有安裝問題，請參閱本文的[安裝疑難排解](/cli/azure/install-az-cli2#installation-troubleshooting)一節，以及 GitHub 上的[安裝疑難排解](https://github.com/Azure/azure-cli/blob/master/doc/install_troubleshooting.md)指南。
>

## <a name="working-with-the-cli"></a>使用 CLI

安裝 CLI 之後，您即可在命令列介面 (Bash、終端機、命令提示字元) 中使用 `az` 命令，存取 Azure CLI 命令。 輸入 `az` 命令，以查看基底命令的完整清單 (下列輸出範例已截斷)︰

```
     /\
    /  \    _____   _ _ __ ___
   / /\ \  |_  / | | | \'__/ _ \
  / ____ \  / /| |_| | | |  __/
 /_/    \_\/___|\__,_|_|  \___|


Welcome to the cool new Azure CLI!

Here are the base commands:

    account          : Manage subscriptions.
    acr              : Manage Azure container registries.
    acs              : Manage Azure Container Services.
    ad               : Synchronize on-premises directories and manage Azure Active Directory
                       resources.
    ...
```

在命令列介面中，執行 `az storage --help` 命令以列出 `storage` 命令子群組。 子群組的描述提供 Azure CLI 針對使用儲存體資源所提供的功能概觀。

```
Group
    az storage: Durable, highly available, and massively scalable cloud storage.

Subgroups:
    account  : Manage storage accounts.
    blob     : Object storage for unstructured data.
    container: Manage blob storage containers.
    cors     : Manage Storage service Cross-Origin Resource Sharing (CORS).
    directory: Manage file storage directories.
    entity   : Manage table storage entities.
    file     : File shares that use the standard SMB 3.0 protocol.
    logging  : Manage Storage service logging information.
    message  : Manage queue storage messages.
    metrics  : Manage Storage service metrics.
    queue    : Use queues to effectively scale applications according to traffic.
    share    : Manage file shares.
    table    : NoSQL key-value storage using semi-structured datasets.
```

## <a name="connect-the-cli-to-your-azure-subscription"></a>將 CLI 連接至您的 Azure 訂用帳戶

若要使用 Azure 訂用帳戶中的資源，您必須先使用 `az login` 登入 Azure 帳戶。 登入的方法有數種：

* **互動式登入**：`az login`
* **以使用者名稱和密碼登入**：`az login -u johndoe@contoso.com -p VerySecret`
  * 這不適用於 Microsoft 帳戶或使用 Multi-Factor Authentication 的帳戶。
* **登入服務主體**：`az login --service-principal -u http://azure-cli-2016-08-05-14-31-15 -p VerySecret --tenant contoso.onmicrosoft.com`

## <a name="azure-cli-sample-script"></a>Azure CLI 範例指令碼

接下來，我們將使用可發出一些基本 Azure CLI 命令的小型殼層指令碼，來與 Azure 儲存體資源互動。 指令碼會先在儲存體帳戶中建立新的容器，然後將現有的檔案 (以 blob 形式) 上傳至該容器。 它會接著列出容器中的所有 blob，最後，將此檔案下載至您指定的本機電腦上的目的地。

```bash
#!/bin/bash
# A simple Azure Storage example script

export AZURE_STORAGE_ACCOUNT=<storage_account_name>
export AZURE_STORAGE_KEY=<storage_account_key>

export container_name=<container_name>
export blob_name=<blob_name>
export file_to_upload=<file_to_upload>
export destination_file=<destination_file>

echo "Creating the container..."
az storage container create --name $container_name

echo "Uploading the file..."
az storage blob upload --container-name $container_name --file $file_to_upload --name $blob_name

echo "Listing the blobs..."
az storage blob list --container-name $container_name --output table

echo "Downloading the file..."
az storage blob download --container-name $container_name --name $blob_name --file $destination_file --output table

echo "Done"
```

**設定及執行指令碼**

1. 開啟您喜愛的文字編輯器，然後複製上述指令碼並貼入編輯器中。

2. 接下來，更新指令碼的變數以反映您的組態設定。 依照指定取代下列值：

   * **\<storage_account_name\>**：您儲存體帳戶的名稱。
   * **\<storage_account_key\>**：儲存體帳戶的主要或次要存取金鑰。
   * **\<container_name\>**：要建立的新容器名稱，例如 "azure-cli-sample-container"。
   * **\<blob_name\>**：容器中目的地 blob 的名稱。
   * **\<file_to_upload\>**：本機電腦上小檔案的路徑，例如 "~/images/HelloWorld.png"。
   * **\<destination_file\>**：目的地檔案路徑，例如 "~/downloadedImage.png"。

3. 在您更新必要變數之後，請儲存指令碼並結束編輯器。 後續步驟假設您已將指令碼命名為 **my_storage_sample.sh**。

4. 如有必要，請將指令碼標示為可執行檔︰`chmod +x my_storage_sample.sh`

5. 執行指令碼。 例如，在 Bash 中：`./my_storage_sample.sh`

您應該會看到類似下列的輸出，而您在指令碼中指定的 **\<destination_file\>** 應出現在您的本機電腦上。

```
Creating the container...
{
  "created": true
}
Uploading the file...
Percent complete: %100.0
Listing the blobs...
Name       Blob Type      Length  Content Type              Last Modified
---------  -----------  --------  ------------------------  -------------------------
README.md  BlockBlob        6700  application/octet-stream  2017-05-12T20:54:59+00:00
Downloading the file...
Name
---------
README.md
Done
```

> [!TIP]
> 上述輸出採用**資料表**格式。 您可以在 CLI 命令中指定 `--output` 引數，或使用 `az configure` 進行全域設定，以指定要使用哪一種輸出格式。
>

## <a name="manage-storage-accounts"></a>管理儲存體帳戶

### <a name="create-a-new-storage-account"></a>建立新的儲存體帳戶
若要使用 Azure 儲存體，您需要儲存體帳戶。 設定電腦以[連接至您的訂用帳戶](#connect-to-your-azure-subscription)之後，您可以建立新的 Azure 儲存體帳戶。

```azurecli
az storage account create \
    --location <location> \
    --name <account_name> \
    --resource-group <resource_group> \
    --sku <account_sku>
```

* `--location` [必要]：位置。 例如，「美國西部」。
* `--name` [必要]：儲存體帳戶名稱。 名稱長度必須為 3-24 個字元，而且僅使用小寫英數字元。
* `--resource-group` [必要]：資源群組的名稱。
* `--sku` [必要]：儲存體帳戶 SKU。 允許的值：
  * `Premium_LRS`
  * `Standard_GRS`
  * `Standard_LRS`
  * `Standard_RAGRS`
  * `Standard_ZRS`

### <a name="set-default-azure-storage-account-environment-variables"></a>設定預設 Azure 儲存體帳戶環境變數

您可以在 Azure 訂用帳戶中有多個儲存體帳戶。 若要選取其中一個來用於所有後續的儲存體命令，您可以設定下列環境變數︰

首先，使用 [az storage account keys list](/cli/azure/storage/account/keys#list) 命令顯示您的儲存體帳戶金鑰：

```azurecli-interactive
az storage account keys list \
    --account-name <account_name> \
    --resource-group <resource_group> \
    --output table
```

現在您有了金鑰，您可以將其和帳戶名稱定義為環境變數：

```azurecli
export AZURE_STORAGE_ACCOUNT=<account_name>
export AZURE_STORAGE_KEY=<key>
```

另一種設定預設儲存體帳戶的方法是使用連接字串。 首先，透過 `show-connection-string` 命令取得連接字串︰

```azurecli
az storage account show-connection-string \
    --name <account_name> \
    --resource-group <resource_group>
```

然後複製輸出連接字串，並將它設定為 `AZURE_STORAGE_CONNECTION_STRING` 環境變數 (您可能需要用引號括住連接字串)：

```azurecli
export AZURE_STORAGE_CONNECTION_STRING="<connection_string>"
```

> [!NOTE]
> 本文下列各節中的所有範例都假設您已設定 `AZURE_STORAGE_ACCOUNT` 和 `AZURE_STORAGE_KEY` 環境變數。

## <a name="create-and-manage-blobs"></a>建立和管理 Blob
Azure Blob 儲存體是一項儲存大量非結構化資料的服務 (例如文字或二進位資料)，全球任何地方都可透過 HTTP 或 HTTPS 來存取這些資料。 本節假設您已熟悉 Azure Blob 儲存體的概念。 如需詳細資訊，請參閱[使用 .NET 開始使用 Azure Blob 儲存體](../blobs/storage-dotnet-how-to-use-blobs.md)和 [Blob 服務概念](/rest/api/storageservices/blob-service-concepts)。

### <a name="create-a-container"></a>建立容器
Azure 儲存體中的每個 Blob 必須位於一個容器中。 您可以使用 `az storage container create` 命令來建立容器：

```azurecli
az storage container create --name <container_name>
```

您可以指定選擇性 `--public-access` 引數，以針對新容器設定讀取權限的三個層級之一︰

* `off` (預設值)︰容器資料為帳戶擁有者私有。
* `blob`：Blob 的公用讀取權限。
* `container`︰整個容器的公用讀取和清單權限。

如需詳細資訊，請參閱 [管理對容器與 Blob 的匿名讀取權限](../blobs/storage-manage-access-to-resources.md)。

### <a name="upload-a-blob-to-a-container"></a>將 Blob 上傳至容器
Azure Blob 儲存體支援區塊、附加和分頁 Blob。 使用 `blob upload` 命令將 blob 上傳至容器：

```azurecli
az storage blob upload \
    --file <local_file_path> \
    --container-name <container_name> \
    --name <blob_name>
```

如果您想直接上傳到儲存體帳戶中容器內的資料夾，請將 `--name <blob_name>` 取代為 `--name <folder/blob_name>`。

 根據預設，`blob upload` 命令會將 *.vhd 檔案上傳至分頁 blob 或者區塊 blob。 若要在上傳 blob 時指定另一種類型，您可以使用 `--type` 引數 - 允許的值為 `append`、`block` 和 `page`。

 如需不同 Blob 類型的詳細資訊，請參閱[了解區塊 Blob、附加 Blob 及分頁 Blob](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs)。


### <a name="download-a-blob-from-a-container"></a>從容器下載 Blob
此範例示範如何從容器下載 Blob：

```azurecli
az storage blob download \
    --container-name mycontainer \
    --name myblob.png \
    --file ~/mydownloadedblob.png
```

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

使用 [az storage blob list](/cli/azure/storage/blob#az_storage_blob_list) 命令列出容器中的 blob。

```azurecli
az storage blob list \
    --container-name mycontainer \
    --output table
```

### <a name="copy-blobs"></a>複製 Blob
您可以在儲存體帳戶內或在不同儲存體帳戶和區域之間，以非同步方式複製 Blob。

下列範例示範如何從一個儲存體帳戶複製 Blob 到另一個儲存體帳戶。 我們會先在來源儲存體帳戶中建立容器，指定對其 Blob 的公用讀取存取權。 接著，我們會將檔案上傳至容器，最後，將 Blob 從該容器複製到目的地儲存體帳戶中的容器。

```azurecli
# Create container in source account
az storage container create \
    --account-name sourceaccountname \
    --account-key sourceaccountkey \
    --name sourcecontainer \
    --public-access blob

# Upload blob to container in source account
az storage blob upload \
    --account-name sourceaccountname \
    --account-key sourceaccountkey \
    --container-name sourcecontainer \
    --file ~/Pictures/sourcefile.png \
    --name sourcefile.png

# Copy blob from source account to destination account (destcontainer must exist)
az storage blob copy start \
    --account-name destaccountname \
    --account-key destaccountkey \
    --destination-blob destfile.png \
    --destination-container destcontainer \
    --source-uri https://sourceaccountname.blob.core.windows.net/sourcecontainer/sourcefile.png
```

在上面的範例中，目的地容器必須已存在於目的地儲存體帳戶中，複製作業才能成功。 此外，在 `--source-uri` 引數中指定的來源 Blob 必須包含共用存取簽章 (SAS) 權杖，或者可公開存取，如此範例所示。

### <a name="delete-a-blob"></a>刪除 Blob
若要刪除 Blob，請使用 `blob delete` 命令：

```azurecli
az storage blob delete --container-name <container_name> --name <blob_name>
```

## <a name="create-and-manage-file-shares"></a>建立和管理檔案共用
針對使用伺服器訊息區 (SMB) 通訊協定的應用程式，Azure 檔案服務可提供適用的共用儲存體。 Microsoft Azure 虛擬機器和雲端服務，以及內部部署應用程式，可以透過掛接共用，共用檔案資料。 您可以透過 Azure CLI 管理檔案共用和檔案資料。 如需 Azure 檔案服務的詳細資訊，請參閱 [Azure 檔案服務簡介](../files/storage-files-introduction.md)。

### <a name="create-a-file-share"></a>建立檔案共用
Azure 檔案共用是 Azure 中的 SMB 檔案共用。 所有目錄和檔案都必須在檔案共用中建立。 帳戶可包含無限制數目的共用，而共用可儲存無限制數目的檔案，最多可達儲存體帳戶的容量限制。 下列範例會建立名為 **myshare** 的檔案共用。

```azurecli
az storage share create --name myshare
```

### <a name="create-a-directory"></a>建立目錄
目錄會提供 Azure 檔案共用的階層式結構。 下列範例會在檔案共用中建立名為 **myDir** 的目錄。

```azurecli
az storage directory create --name myDir --share-name myshare
```

目錄路徑可包含多個層級，例如 **dir1/dir2**。 不過，您必須先確定所有父目錄都存在，才能建立子目錄。 例如，如果路徑是 **dir1/dir2**，您必須先建立目錄 **dir1**，然後再建立目錄 **dir2**。

### <a name="upload-a-local-file-to-a-share"></a>將本機檔案上傳至共用
下列範例會將檔案從 **~/temp/samplefile.txt** 上傳至 **myshare** 檔案共用的根目錄。 `--source` 引數會指定要上傳的現有本機檔案。

```azurecli
az storage file upload --share-name myshare --source ~/temp/samplefile.txt
```

建立目錄時，您可以指定共用中的目錄路徑，以將檔案上傳至共用中的現有目錄︰

```azurecli
az storage file upload --share-name myshare/myDir --source ~/temp/samplefile.txt
```

共用中的檔案大小可能高達 1 TB。

### <a name="list-the-files-in-a-share"></a>列出共用中的檔案
您可以使用 `az storage file list` 命令列出共用中的檔案和目錄︰

```azurecli
# List the files in the root of a share
az storage file list --share-name myshare --output table

# List the files in a directory within a share
az storage file list --share-name myshare/myDir --output table

# List the files in a path within a share
az storage file list --share-name myshare --path myDir/mySubDir/MySubDir2 --output table
```

### <a name="copy-files"></a>複製檔案      
您可以將檔案複製到另一個檔案、將檔案複製到 Blob 或將 Blob 複製到檔案。 例如，若要將檔案複製到不同共用中的目錄︰        
        
```azurecli
az storage file copy start \
--source-share share1 --source-path dir1/file.txt \
--destination-share share2 --destination-path dir2/file.txt     
```

## <a name="create-share-snapshot"></a>建立共用快照集
您可以使用 `az storage share snapshot` 命令來建立共用快照集：

```cli
az storage share snapshot -n <share name>
```

範例輸出
```json
{
  "metadata": {},
  "name": "<share name>",
  "properties": {
    "etag": "\"0x8D50B7F9A8D7F30\"",
    "lastModified": "2017-10-04T23:28:22+00:00",
    "quota": null
  },
  "snapshot": "2017-10-04T23:28:35.0000000Z"
}
```

### <a name="list-share-snapshots"></a>列出共用快照集

您可以使用 `az storage share list --include-snapshots` 列出特定共用的共用快照集

```cli
az storage share list --include-snapshots
```

**範例輸出**
```json
[
  {
    "metadata": null,
    "name": "sharesnapshotdefs",
    "properties": {
      "etag": "\"0x8D50B5F4005C975\"",
      "lastModified": "2017-10-04T19:36:46+00:00",
      "quota": 5120
    },
    "snapshot": "2017-10-04T19:44:13.0000000Z"
  },
  {
    "metadata": null,
    "name": "sharesnapshotdefs",
    "properties": {
      "etag": "\"0x8D50B5F4005C975\"",
      "lastModified": "2017-10-04T19:36:46+00:00",
      "quota": 5120
    },
    "snapshot": "2017-10-04T19:45:18.0000000Z"
  },
  {
    "metadata": null,
    "name": "sharesnapshotdefs",
    "properties": {
      "etag": "\"0x8D50B5F4005C975\"",
      "lastModified": "2017-10-04T19:36:46+00:00",
      "quota": 5120
    },
    "snapshot": null
  }
]
```

### <a name="browse-share-snapshots"></a>瀏覽共用快照集
您也可以使用 `az storage file list` 瀏覽至特定的共用快照集，以檢視其內容。 您必須指定共用名稱 `--share-name <snare name>` 與時間戳記 `--snapshot '2017-10-04T19:45:18.0000000Z'`

```azurecli-interactive
az storage file list --share-name sharesnapshotdefs --snapshot '2017-10-04T19:45:18.0000000Z' -otable
```

**範例輸出**
```
Name            Content Length    Type    Last Modified
--------------  ----------------  ------  ---------------
HelloWorldDir/                    dir
IMG_0966.JPG    533568            file
IMG_1105.JPG    717711            file
IMG_1341.JPG    608459            file
IMG_1405.JPG    652156            file
IMG_1611.JPG    442671            file
IMG_1634.JPG    1495999           file
IMG_1635.JPG    974058            file

```
### <a name="restore-from-share-snapshots"></a>從共用快照集還原

您可以使用 `az storage file download` 命令，從共用快照集複製或下載檔案，藉此還原檔案

```azurecli-interactive
az storage file download --path IMG_0966.JPG --share-name sharesnapshotdefs --snapshot '2017-10-04T19:45:18.0000000Z'
```
**範例輸出**
```
{
  "content": null,
  "metadata": {},
  "name": "IMG_0966.JPG",
  "properties": {
    "contentLength": 533568,
    "contentRange": "bytes 0-533567/533568",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentType": "application/octet-stream"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "etag": "\"0x8D50B5F49F7ACDF\"",
    "lastModified": "2017-10-04T19:37:03+00:00",
    "serverEncrypted": true
  }
}
```
## <a name="delete-share-snapshot"></a>刪除共用快照集
您可以使用 `az storage share delete` 命令，並提供具有共用快照集時間戳記的 `--snapshot` 參數來刪除共用快照集：

```cli
az storage share delete -n <share name> --snapshot '2017-10-04T23:28:35.0000000Z' 
```

範例輸出
```json
{
  "deleted": true
}
```

## <a name="next-steps"></a>後續步驟
以下有一些額外的資源，可供深入了解如何使用 Azure CLI。 

* [開始使用 Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)
* [Azure CLI 命令參考](/cli/azure)
* [GitHub 上的 Azure CLI](https://github.com/Azure/azure-cli)
