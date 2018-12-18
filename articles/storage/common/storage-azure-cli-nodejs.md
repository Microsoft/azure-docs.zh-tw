---
title: 使用 Azure 傳統 CLI 搭配 Azure 儲存體 | Microsoft Docs
description: 了解如何搭配 Azure 儲存體使用 Azure 傳統命令列介面 (CLI) 來建立及管理儲存體帳戶，以及使用 Azure Blob 與檔案。
services: storage
author: seguler
ms.service: storage
ms.topic: article
ms.date: 01/30/2017
ms.author: seguler
ms.component: common
ms.openlocfilehash: e563c7000b600bed917f42d8ffb87df883564ef8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979323"
---
# <a name="using-the-azure-classic-cli-with-azure-storage"></a>搭配 Azure 儲存體使用 Azure 傳統 CLI

## <a name="overview"></a>概觀

Azure 傳統 CLI 為您提供一組開放式原始碼的跨平台命令集合，供您運用在 Azure 平台上。 它提供許多與 [Azure 入口網站](https://portal.azure.com) 相同的功能，以及豐富的資料存取功能。

在此指南中，我們將探討如何使用 [Azure 傳統 CLI](../../cli-install-nodejs.md) 來搭配 Azure 儲存體執行各種開發與管理工作。 建議您在使用此指南之前，先下載並安裝或升級至最新的傳統 CLI。

此指南假設您已了解 Azure 儲存體的基本概念。 此指南提供許多指令碼示範如何搭配 Azure 儲存體使用 Azure CLI。 在執行每個指令碼之前，請務必先根據您的設定更新指令碼變數。

> [!NOTE]
> 此指南提供傳統儲存體帳戶的 Azure 傳統 CLI 命令與指令碼範例。 如需適用於 Resource Manager 儲存體帳戶的 Azure 傳統 CLI 命令，請參閱 [使用適用於 Mac、Linux 與 Windows 的 Azure CLI 搭配 Azure 資源管理](../../virtual-machines/azure-cli-arm-commands.md#azure-storage-commands-to-manage-your-storage-objects)。
>
>

[!INCLUDE [storage-cli-versions](../../../includes/storage-cli-versions.md)]

## <a name="get-started-with-azure-storage-and-the-azure-classic-cli-in-5-minutes"></a>在 5 分鐘內開始使用 Azure 儲存體和 Azure 傳統 CLI
此指南使用 Ubuntu 作為範例，但其他作業系統平台也同樣能夠執行。

**Azure 新手：** 取得 Microsoft Azure 訂用帳戶和與該訂用帳戶相關聯的 Microsoft 帳戶。 如需 Azure 購買選項的資訊，請參閱[免費試用](https://azure.microsoft.com/pricing/free-trial/)、[購買選項](https://azure.microsoft.com/pricing/purchase-options/)和[會員供應項目](https://azure.microsoft.com/pricing/member-offers/) (適用於 MSDN、Microsoft 合作夥伴網路、BizSpark 和其他 Microsoft 方案的成員)。

如需 Azure 訂用帳戶的詳細資訊，請參閱 [在 Azure Active Directory (Azure AD) 中指派系統管理員角色](https://docs.microsoft.com/azure/active-directory/active-directory-assign-admin-roles-azure-portal) 。

**建立 Microsoft Azure 訂用帳戶和帳戶之後：**

1. 依照 [安裝 Azure 傳統 CLI](../../cli-install-nodejs.md)中的指示下載並安裝 Azure 傳統 CLI。
2. 安裝好傳統 CLI 之後，您就能從命令列介面 (Bash、終端機、命令提示字元) 中使用 azure 命令存取傳統 CLI 命令。 輸入 _azure_ 命令，您應該會看見下列輸出。

    ![Azure 命令輸出](./media/storage-azure-cli/azure_command.png)   
3. 在命令列介面中，輸入 `azure storage` 以列出所有 Azure 儲存體命令，對傳統 CLI 所提供的功能進行初步了解。 您可以輸入命令名稱搭配 **-h** 參數 (例如，`azure storage share create -h`)，查看命令語法的詳細資料。
4. 現在，我們將為您提供一個簡單的指令碼，其中顯示用來存取「Azure 儲存體」的基本傳統 CLI 命令。 指令碼會先要求您為您的儲存體帳戶和金鑰設定兩個變數。 然後，指令碼將在這個新的儲存體帳戶中建立新容器，並將現有的映像檔案 (Blob) 上傳至該容器。 指令碼列出該容器中的所有 Blob 之後，會將映像檔案下載至本機電腦上的目的地目錄。

    ```azurecli
    #!/bin/bash
    # A simple Azure storage example

    export AZURE_STORAGE_ACCOUNT=<storage_account_name>
    export AZURE_STORAGE_ACCESS_KEY=<storage_account_key>

    export container_name=<container_name>
    export blob_name=<blob_name>
    export image_to_upload=<image_to_upload>
    export destination_folder=<destination_folder>

    echo "Creating the container..."
    azure storage container create $container_name

    echo "Uploading the image..."
    azure storage blob upload $image_to_upload $container_name $blob_name

    echo "Listing the blobs..."
    azure storage blob list $container_name

    echo "Downloading the image..."
    azure storage blob download $container_name $blob_name $destination_folder

    echo "Done"
    ```

5. 在您的本機電腦上，開啟您慣用的文字編輯器 (例如 vim)。 在文字編輯器中輸入上述指令碼。
6. 現在，您需要根據您的組態設定更新指令碼變數。

   * **<storage_account_name>** 使用指令碼中的指定名稱，或是為儲存體帳戶輸入新名稱。 **重要事項：** 儲存體帳戶的名稱在 Azure 中必須是唯一的名稱。 而且必須是小寫字母！
   * **<storage_account_key>** 儲存體帳戶的存取金鑰。
   * **<container_name>** 使用指令碼中的指定名稱，或是為容器輸入新名稱。
   * **<image_to_upload>** 輸入位於本機電腦上的圖片路徑，例如 "~/images/HelloWorld.png"。
   * **<destination_folder>** 輸入用以儲存從「Azure 儲存體」下載之檔案的本機目錄路徑，例如 "~/downloadImages"。
7. 在 vim 中更新必要變數之後，請按下按鍵組合 `ESC`、`:`、`wq!` 來儲存指令碼。
8. 若要執行這個指令碼，只要在 bash 主控台中輸入指令碼檔案名稱即可。 在這個指令碼執行之後，您應該會有一個本機目的地資料夾，包含下載的映像檔案。 以下螢幕擷取畫面顯示範例輸出︰

在指令碼執行之後，您應該有包含下載的映像檔案的本機目的資料夾。

## <a name="manage-storage-accounts-with-the-azure-classic-cli"></a>使用 Azure 傳統 CLI 管理儲存體帳戶
### <a name="connect-to-your-azure-subscription"></a>連接到 Azure 訂用帳戶
雖然大多數儲存體命令在沒有 Azure 訂用帳戶的情況下也能運作，但是仍建議您從傳統 CLI 連接到您的訂用帳戶。

### <a name="create-a-new-storage-account"></a>建立新的儲存體帳戶
若要使用 Azure 儲存體，您將需要儲存體帳戶。 設定電腦以連接至您的訂用帳戶之後，您可以建立新的 Azure 儲存體帳戶。

```azurecli
azure storage account create <account_name>
```

儲存體帳戶的名稱必須介於 3 到 24 個字元的長度，而且只能使用數字和小寫字母。

### <a name="set-a-default-azure-storage-account-in-environment-variables"></a>在環境變數中設定預設 Azure 儲存體帳戶
您可以在訂用帳戶中有多個儲存體帳戶。 您可以選擇其中一個儲存體帳戶並且在環境變數中進行設定，讓同一個工作階段中的所有儲存體命令都能使用。 這可讓您執行傳統 CLI 儲存體命令，而不需明確指定儲存體帳戶和金鑰。

```azurecli
export AZURE_STORAGE_ACCOUNT=<account_name>
export AZURE_STORAGE_ACCESS_KEY=<key>
```

另一種設定預設儲存體帳戶的方式是使用連接字串。 首先透過下列命令取得連接字串：

```azurecli
azure storage account connectionstring show <account_name>
```

然後複製輸出連接字串，並將它設定為環境變數：

```azurecli
export AZURE_STORAGE_CONNECTION_STRING=<connection_string>
```

## <a name="create-and-manage-blobs"></a>建立和管理 Blob
Azure Blob 儲存體是一項儲存大量非結構化資料的服務 (例如文字或二進位資料)，全球任何地方都可透過 HTTP 或 HTTPS 來存取這些資料。 本節假設您已熟悉 Azure Blob 儲存體的概念。 如需詳細資訊，請參閱[使用 .NET 開始使用 Azure Blob 儲存體](../blobs/storage-dotnet-how-to-use-blobs.md)和 [Blob 服務概念](http://msdn.microsoft.com/library/azure/dd179376.aspx)。

### <a name="create-a-container"></a>建立容器
Azure 儲存體中的每個 Blob 必須位於一個容器中。 您可以使用 `azure storage container create` 命令建立私用容器：

```azurecli
azure storage container create mycontainer
```

> [!NOTE]
> 匿名讀取權限有三個層級：**Off**、**Blob** 和 **Container**。 若要防止匿名存取 Blob，請將 Permission 參數設定為 **Off**。 新容器預設為私人，且只能由帳戶擁有者存取。 若要允許 Blob 資源的匿名公開讀取權限，但不允許容器中繼資料或容器中 Blob 清單的匿名公開讀取權限，請將 Permission 參數設定為 **Blob**。 若要允許 Blob 資源、容器中繼資料或容器中 Blob 清單的完整公開讀取權限，請將 Permission 參數設定為 **Container**。 如需詳細資訊，請參閱 [管理對容器與 Blob 的匿名讀取權限](../blobs/storage-manage-access-to-resources.md)。
>
>

### <a name="upload-a-blob-into-a-container"></a>將 Blob 上傳至容器
Azure Blob 儲存體支援區塊 Blob 和頁面 Blob。 如需詳細資訊，請參閱 [了解區塊 Blob、附加 Blob 和分頁 Blob](http://msdn.microsoft.com/library/azure/ee691964.aspx)。

若要將 Blob 上傳至容器，您可以使用 `azure storage blob upload`。 根據預設，此命令會將本機檔案上傳至區塊 Blob。 若要指定 Blob 的類型，您可以使用 `--blobtype` 參數。

```azurecli
azure storage blob upload '~/images/HelloWorld.png' mycontainer myBlockBlob
```

### <a name="download-blobs-from-a-container"></a>從容器下載 Blob
下列範例示範如何從容器下載 Blob。

```azurecli
azure storage blob download mycontainer myBlockBlob '~/downloadImages/downloaded.png'
```

### <a name="copy-blobs"></a>複製 Blob
您可以在儲存體帳戶內或在不同儲存體帳戶和區域之間，以非同步方式複製 Blob。

下列範例示範如何從一個儲存體帳戶複製 Blob 到另一個儲存體帳戶。 在此範例中，我們建立 Blob 為公開，可匿名存取的容器。

```azurecli
azure storage container create mycontainer2 -a <accountName2> -k <accountKey2> -p Blob

azure storage blob upload '~/Images/HelloWorld.png' mycontainer2 myBlockBlob2 -a <accountName2> -k <accountKey2>

azure storage blob copy start 'https://<accountname2>.blob.core.windows.net/mycontainer2/myBlockBlob2' mycontainer
```

此範例會執行非同步複製。 您可以執行 `azure storage blob copy show` 作業監視每個複製作業的狀態。

請注意，針對複製作業提供的來源 URL 必須是可公開存取，或包含 SAS (共用存取簽章) 權杖。

### <a name="delete-a-blob"></a>刪除 Blob
若要刪除 Blob，請使用下列命令：

```azurecli
azure storage blob delete mycontainer myBlockBlob2
```

## <a name="create-and-manage-file-shares"></a>建立和管理檔案共用
Azure 檔案服務為使用標準 SMB 通訊協定的應用程式提供共用儲存體。 Microsoft Azure 虛擬機器和雲端服務，以及內部部署應用程式，可以透過掛接共用，共用檔案資料。 您可以透過傳統 CLI 管理檔案共用與檔案資料。 如需 Azure 檔案服務的詳細資訊，請參閱 [Azure 檔案服務簡介](../files/storage-files-introduction.md)。

### <a name="create-a-file-share"></a>建立檔案共用
Azure 檔案共用是 Azure 中的 SMB 檔案共用。 所有目錄和檔案都必須在檔案共用中建立。 帳戶可包含無限制數目的共用，而共用可儲存無限制數目的檔案，最多可達儲存體帳戶的容量限制。 下列範例會建立名為 **myshare** 的檔案共用。

```azurecli
azure storage share create myshare
```

### <a name="create-a-directory"></a>建立目錄
目錄會提供 Azure 檔案共用選擇性的階層式結構。 下列範例會在檔案共用中建立名為 **myDir** 的目錄。

```azurecli
azure storage directory create myshare myDir
```

請注意，目錄路徑可包含多個層級，例如 **a/b**。 不過，您必須確定所有父目錄都存在。 例如，如果路徑是 **a/b**，您必須先建立目錄 **a**，然後再建立目錄 **b**。

### <a name="upload-a-local-file-to-directory"></a>上傳本機檔案至目錄
下列範例將從 **~/temp/samplefile.txt** 上傳檔案至 **myDir** 目錄。 編輯檔案路徑，以指向本機機器上的有效檔案：

```azurecli
azure storage file upload '~/temp/samplefile.txt' myshare myDir
```

請注意，共用中的檔案大小可能高達 1 TB。

### <a name="list-the-files-in-the-share-root-or-directory"></a>列出共用根或目錄中的檔案
您可以使用下列命令，列出共用根或目錄中的檔案和子目錄：

```azurecli
azure storage file list myshare myDir
```

請注意，列出作業不一定會顯示目錄名稱。 如果省略，則命令會列出共用根目錄的內容。

### <a name="copy-files"></a>複製檔案
從傳統 CLI 0.9.8 版開始，您可以將檔案複製到另一個檔案、將檔案複製到 Blob 或將 Blob 複製到檔案。 下列示範如何使用 CLI 命令執行這些複製作業。 將檔案複製到新的目錄：

```azurecli
azure storage file copy start --source-share srcshare --source-path srcdir/hello.txt --dest-share destshare
    --dest-path destdir/hellocopy.txt --connection-string $srcConnectionString --dest-connection-string $destConnectionString
```

將 Blob 複製到檔案目錄：

```azurecli
azure storage file copy start --source-container srcctn --source-blob hello2.txt --dest-share hello
    --dest-path hellodir/hello2copy.txt --connection-string $srcConnectionString --dest-connection-string $destConnectionString
```

## <a name="next-steps"></a>後續步驟

您可以在下列網頁找到可與「儲存體」資源搭配運作的傳統 CLI 命令參考資料：

* [Resource Manager 模式中的 Azure 傳統 CLI 命令](../../virtual-machines/azure-cli-arm-commands.md#azure-storage-commands-to-manage-your-storage-objects)
* [Azure 服務管理模式中的 Azure 傳統 CLI 命令](../../cli-install-nodejs.md)

您或許也會想要試試使用最新版的 [Azure CLI](../storage-azure-cli.md) 來搭配 Resource Manager 部署模型。
