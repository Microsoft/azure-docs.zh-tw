---
title: 快速入門：使用 Node.js 上傳、下載及列出 Blob - Azure 儲存體
description: 在物件 (Blob) 儲存體中建立儲存體帳戶和容器。 然後，使用 Node.js 的儲存體用戶端程式庫將 blob 上傳至 Azure 儲存體、下載 blob，以及列出容器中的 blob。
services: storage
author: craigshoemaker
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 09/20/2018
ms.author: cshoe
ms.openlocfilehash: 1c62dbd6856ec7bf2663f0b70a47357b52528899
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040807"
---
# <a name="quickstart-upload-download-and-list-blobs-using-nodejs"></a>快速入門：使用 Node.js 上傳、下載及列出 Blob

在本快速入門中，您會了解如何使用 Node.js 以透過 Azure Blob 儲存體上傳、下載及列出 Blob 和管理容器。

若要完成本快速入門，您需要 [Azure 訂用帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="download-the-sample-application"></a>下載範例應用程式

本快速入門中的[應用程式範例](https://github.com/Azure-Samples/storage-blobs-node-quickstart.git)是簡單的 Node.js 主控台應用程式。 若要開始，請使用下列命令將存放庫複製到您的機器：

```bash
git clone https://github.com/Azure-Samples/storage-blobs-node-quickstart.git
```

若要開啟應用程式，請尋找 storage-blobs-node-quickstart 資料夾，然後在您慣用的程式碼編輯環境中加以開啟。

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>設定儲存體連接字串

在執行應用程式之前，您必須先提供儲存體帳戶的連接字串。 存放庫範例包含名為 .env.example 的檔案。 您可以藉由移除 .example 副檔名來將這個檔案重新命名，從而產生名為 .env 的檔案。 在 .env 檔案內，於 AZURE_STORAGE_CONNECTION_STRING 索引鍵後面新增您的連接字串值。

## <a name="install-required-packages"></a>安裝必要的套件

在應用程式目錄中，執行 npm install 來安裝應用程式的所需套件。

```bash
npm install
```

## <a name="run-the-sample"></a>執行範例
相依性已安裝好，因此您可以發出下列命令來來執行範例：

```bash
npm start
```

指令碼的輸出會如下所示：

```bash
Containers:
 - container-one
 - container-two
Container "demo" is created
Blob "quickstart.txt" is uploaded
Local file "./readme.md" is uploaded
Blobs in "demo" container:
 - quickstart.txt
 - readme.md
Blob downloaded blob content: "hello Blob SDK"
Blob "quickstart.txt" is deleted
Container "demo" is deleted
Done
```

請注意，如果您在本快速入門中使用新的儲存體帳戶，則可能看不到 "Containers" 標籤底下所列的容器名稱。

## <a name="understanding-the-code"></a>了解程式碼
第一個運算式用來載入環境變數的值。

```javascript
if (process.env.NODE_ENV !== 'production') {
    require('dotenv').load();
}
```

dotenv 模組在本機執行應用程式進行偵錯會載入環境變數。 變數值會定義於名為 .env 的檔案中並載入目前的執行環境中。 在生產環境中，伺服器組態會提供這些值，這就是為何只有當指令碼未在「生產」環境下執行時，此程式碼才會執行。

```javascript
const path = require('path');
const storage = require('azure-storage');
```

這些模組的目的如下： 

名為 .env 的檔案會載入目前的執行環境中
- path 是必要項目，有此項目才能判斷要上傳至 Blob 儲存體之檔案的絕對檔案路徑
- azure-storage 是 Node.js 的 [Azure 儲存體 SDK](https://docs.microsoft.com/javascript/api/azure-storage) 模組

接下來，blobService 變數會初始化為 Azure Blob 服務的新執行個體。

```javascript
const blobService = storage.createBlobService();
```

在下列實作中，每個 blobService 函式都會包裝在 Promise 中，以允許存取 JavaScript 的 async 函式和 await 運算子來簡化 [Azure 儲存體 API](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest) 的回呼性質。 當每個函式都有成功的回應傳回時，promise 會使用相關資料以及動作特有的訊息來解析。

### <a name="list-containers"></a>列出容器

ListContainers 函式會呼叫 [listContainersSegmented](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#listcontainerssegmented)，以傳回群組中的容器集合。

```javascript
const listContainers = async () => {
    return new Promise((resolve, reject) => {
        blobService.listContainersSegmented(null, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `${data.entries.length} containers`, containers: data.entries });
            }
        });
    });
};
```

群組的大小可透過 [ListContainersOptions](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice.listcontaineroptions?view=azure-node-latest) 設定。 呼叫 listContainersSegmented 會將 Blob 中繼資料傳回為 [ContainerResult](/nodejs/api/azure-storage/blobresult) 執行個體的陣列。 結果會以 5,000 的增量批次 (區段) 傳回。 如果容器中有 5,000 個以上的 Blob，結果中會包含 *continuationToken* 的值。 若要列出 Blob 容器的後續區段，您可以將接續權杖回傳給 listContainersSegment 作為第二個引數。

### <a name="create-a-container"></a>建立容器

CreateContainer 函式會呼叫 [createContainerIfNotExists](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#createcontainerifnotexists) 並為 Blob 設定適當的存取層級。

```javascript
const createContainer = async (containerName) => {
    return new Promise((resolve, reject) => {
        blobService.createContainerIfNotExists(containerName, { publicAccessLevel: 'blob' }, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Container '${containerName}' created` });
            }
        });
    });
};
```

**createContainerIfNotExists** 的第二個參數 (options) 會接受 [publicAccessLevel](https://docs.microsoft.com/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#createcontainerifnotexists) 的值。 publicAccessLevel 的 Blob 值會指定要公開的特定 Blob 資料。 此設定可和 container 存取層級相比較，後者可授與列出容器內容的能力。

使用 **createContainerIfNotExists** 可讓應用程式執行 createContainer 命令多次，而不會在容器早已存在時傳回錯誤。 在生產環境中，由於整個應用程式都使用相同的容器，因此您往往只會呼叫 **CreateIfNotExists** 一次。 在這些情況下，您可以透過入口網站或透過 Azure CLI 事先建立容器。

### <a name="upload-text"></a>上傳文字

*uploadString* 函式會呼叫 [createBlockBlobFromText](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#createblockblobfromtext) 以將任意字串寫入 (或覆寫) 至 blob 容器。

```javascript
const uploadString = async (containerName, blobName, text) => {
    return new Promise((resolve, reject) => {
        blobService.createBlockBlobFromText(containerName, blobName, text, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Text "${text}" is written to blob storage` });
            }
        });
    });
};
```
### <a name="upload-a-local-file"></a>上傳本機檔案

uploadLocalFile 函式會使用 [createBlockBlobFromLocalFile](/nodejs/api/azure-storage/blobservice#azure_storage_BlobService_createBlockBlobFromLocalFile) 將檔案從檔案系統上傳及寫入或覆寫到 Blob 儲存體。 

```javascript
const uploadLocalFile = async (containerName, filePath) => {
    return new Promise((resolve, reject) => {
        const fullPath = path.resolve(filePath);
        const blobName = path.basename(filePath);
        blobService.createBlockBlobFromLocalFile(containerName, blobName, fullPath, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Local file "${filePath}" is uploaded` });
            }
        });
    });
};
```
其他可供將內容上傳至 Blob 的方法包括使用[文字](/nodejs/api/azure-storage/blobservice#azure_storage_BlobService_createBlockBlobFromText)和[串流](/nodejs/api/azure-storage/blobservice#azure_storage_BlobService_createBlockBlobFromStream)。 若要確認檔案有上傳至您的 Blob 儲存體，您可以使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)來檢視您帳戶中的資料。

### <a name="list-the-blobs"></a>列出 blob

listBlobs 函式會呼叫 [listBlobsSegmented](/nodejs/api/azure-storage/blobservice#azure_storage_BlobService_createBlockBlobFromText) 方法以傳回容器中 Blob 中繼資料的清單。 

```javascript
const listBlobs = async (containerName) => {
    return new Promise((resolve, reject) => {
        blobService.listBlobsSegmented(containerName, null, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `${data.entries.length} blobs in '${containerName}'`, blobs: data.entries });
            }
        });
    });
};
```

呼叫 listBlobsSegmented 會將 Blob 中繼資料傳回為 [BlobResult](https://docs.microsoft.com/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice.blobresult?view=azure-node-latest) 執行個體的陣列。 結果會以 5,000 的增量批次 (區段) 傳回。 如果容器中有 5,000 個以上的 Blob，結果中會包含 **continuationToken** 的值。 若要列出 Blob 容器的後續區段，您可以將接續權杖回傳給 **listBlobSegmented** 作為第二個引數。

### <a name="download-a-blob"></a>下載 Blob

downloadBlob 函式會使用 [getBlobToText](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#getblobtotext) 將 Blob 的內容下載至指定的絕對檔案路徑。

```javascript
const downloadBlob = async (containerName, blobName) => {
    const dowloadFilePath = path.resolve('./' + blobName.replace('.txt', '.downloaded.txt'));
    return new Promise((resolve, reject) => {
        blobService.getBlobToText(containerName, blobName, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Blob downloaded "${data}"`, text: data });
            }
        });
    });
};
```
這裡所示的實作會變更，而來源會將 blob 的內容當作字串傳回。 您也可以下載 blob 作為[資料流](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#getblobtostream)，而且直接下載至[本機檔案](/javascript/api/azure-storage/azurestorage.services.blob.blobservice.blobservice?view=azure-node-latest#getblobtolocalfile)。

### <a name="delete-a-blob"></a>刪除 Blob

*deleteBlob* 函式會呼叫 [deleteBlobIfExists](/nodejs/api/azure-storage/blobservice#azure_storage_BlobService_deleteBlobIfExists) 函式。 顧名思義，如果 Blob 早已刪除，此函式就不會傳回錯誤。

```javascript
const deleteBlob = async (containerName, blobName) => {
    return new Promise((resolve, reject) => {
        blobService.deleteBlobIfExists(containerName, blobName, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Block blob '${blobName}' deleted` });
            }
        });
    });
};
```

### <a name="delete-a-container"></a>刪除容器

從 blob 服務外呼叫 deleteContainer 方法並傳入容器名稱，就會刪除容器。

```javascript
const deleteContainer = async (containerName) => {
    return new Promise((resolve, reject) => {
        blobService.deleteContainer(containerName, err => {
            if (err) {
                reject(err);
            } else {
                resolve({ message: `Container '${containerName}' deleted` });
            }
        });
    });
};
```

### <a name="calling-code"></a>呼叫程式碼

為了支援 JavaScript 的 async/await 語法，所有呼叫程式碼都會包裝在名為 execute 函式中。 接著會呼叫 execute 並當作 promise 處理。

```javascript
async function execute() {
    // commands 
}

execute().then(() => console.log("Done")).catch((e) => console.log(e));
```
下列程式碼全都在已放置 `// commands` 註解的 execute 函數內執行。

首先會宣告相關變數來指派名稱、範例內容，以及指向要上傳至 Blob 儲存體的本機檔案。

```javascript
const containerName = "demo";
const blobName = "quickstart.txt";
const content = "hello Node SDK";
const localFilePath = "./readme.md";
let response;
```

為了列出儲存體帳戶中的容器，系統會呼叫 listContainers 函式，而傳回的容器清單會記錄到輸出視窗。

```javascript
console.log("Containers:");
response = await listContainers();
response.containers.forEach((container) => console.log(` -  ${container.name}`));
```

取得容器清單之後，您即可使用陣列 *findIndex* 方法來查看您想要建立的容器是否已經存在。 如果容器不存在，則會建立容器。

```javascript
const containerDoesNotExist = response.containers.findIndex((container) => container.name === containerName) === -1;

if (containerDoesNotExist) {
    await createContainer(containerName);
    console.log(`Container "${containerName}" is created`);
}
```
接下來，會將字串和本機檔案上傳至 Blob 儲存體。

```javascript
await uploadString(containerName, blobName, content);
console.log(`Blob "${blobName}" is uploaded`);

response = await uploadLocalFile(containerName, localFilePath);
console.log(response.message);
```
列出 blob 與列出容器的程序相同。 呼叫 *listBlobs* 可傳回容器中的 blob 陣列，並且記錄到輸出視窗。

```javascript
console.log(`Blobs in "${containerName}" container:`);
response = await listBlobs(containerName);
response.blobs.forEach((blob) => console.log(` - ${blob.name}`));
```

若要下載 blob，則會擷取回應並用來存取 blob 的值。 回應中的 readableStreamBody 會轉換成為字串並記錄到輸出視窗。

```javascript
response = await downloadBlob(containerName, blobName);
console.log(`Downloaded blob content: "${response.text}"`);
```

最後，blob 和容器會從儲存體帳戶中刪除。

```javascript
await deleteBlob(containerName, blobName);
console.log(`Blob "${blobName}" is deleted`);

await deleteContainer(containerName);
console.log(`Container "${containerName}" is deleted`);
```

## <a name="clean-up-resources"></a>清除資源
寫入儲存體帳戶的所有資料都會在程式碼範例的結尾自動刪除。 

## <a name="resources-for-developing-nodejs-applications-with-blobs"></a>可供使用 Blob 開發 Node.js 應用程式的資源

請參閱以下可供使用 Blob 儲存體進行 Node.js 開發的額外資源：

### <a name="binaries-and-source-code"></a>二進位檔和原始程式碼

- 檢視及安裝 GitHub 上適用於 Azure 儲存體的 [Node.js 用戶端程式庫原始程式碼](https://github.com/Azure/azure-storage-node)。

### <a name="client-library-reference-and-samples"></a>用戶端程式庫參考和範例

- 如需 Node.js 用戶端程式庫的詳細資訊，請參閱 [Node.js API 參考](https://docs.microsoft.com/javascript/api/overview/azure/storage)。
- 探索使用 Node.js 用戶端程式庫所撰寫的 [Blob 儲存體範例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=nodejs&term=blob)。

## <a name="next-steps"></a>後續步驟

本快速入門會示範如何使用 Node.js 在本機磁碟和 Azure Blob 儲存體之間上傳檔案。 若要深入了解 Blob 儲存體的用法，請繼續閱讀 GitHub 存放庫。

> [!div class="nextstepaction"]
> [適用於 JavaScript 的 Azure 儲存體 SDK 存放庫](https://github.com/Azure/azure-storage-node)