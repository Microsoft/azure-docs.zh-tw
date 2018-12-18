---
title: 使用適用於 JavaScript 的 Azure 儲存體 v10 SDK (預覽) 上傳、下載、列出及刪除 Blob
description: 使用 Azure 儲存體，在 Node.js 中建立、上傳及刪除 Blob 和容器
services: storage
author: craigshoemaker
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 09/19/2018
ms.author: cshoe
ms.openlocfilehash: 6e23e888a1c90e1c6c7eecf25491f048e9077f11
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857885"
---
# <a name="quickstart-upload-download-list-and-delete-blobs-using-azure-storage-v10-sdk-for-javascript-preview"></a>快速入門：使用適用於 JavaScript 的 Azure 儲存體 v10 SDK (預覽) 上傳、下載、列出及刪除 Blob

在本快速入門中，您將了解如何在 Node.js 中，使用[適用於 JavaScript 的 Azure 儲存體 v10 SDK](https://github.com/Azure/azure-storage-js) \(英文\)，來上傳、下載、列出及刪除 Blob，以及管理容器。

若要完成本快速入門，您需要 [Azure 訂用帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="download-the-sample-application"></a>下載範例應用程式

本快速入門中的[應用程式範例](https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git)是簡單的 Node.js 主控台應用程式。 若要開始，請使用下列命令將存放庫複製到您的機器：

```bash
git clone https://github.com/Azure-Samples/azure-storage-js-v10-quickstart.git
```

接下來，變更應用程式的資料夾：

```bash
cd azure-storage-js-v10-quickstart
```

現在，在您最愛的程式碼編輯環境中開啟該資料夾。

## <a name="configure-your-storage-credentials"></a>設定儲存體認證

執行應用程式之前，您必須先提供儲存體帳戶的安全性認證。 存放庫範例包含名為 .env.example 的檔案。 藉由移除 *.example* 副檔名來將這個檔案重新命名，從而產生名為 *.env* 的檔案。 在 *.env* 檔案內部，於 *AZURE_STORAGE_ACCOUNT_NAME* 和 *AZURE_STORAGE_ACCOUNT_ACCESS_KEY* 金鑰之後新增您的帳戶名稱和存取金鑰值。

## <a name="install-required-packages"></a>安裝必要的套件

在應用程式目錄中，執行 npm install 來安裝應用程式的所需套件。

```bash
npm install
```

## <a name="run-the-sample"></a>執行範例
相依性現已安裝，因此您可以發出下列命令來來執行範例：

```bash
npm start
```

應用程式的輸出將類似下列範例：

```bash
Containers:
 - container-one
 - container-two
Container "demo" is created
Blob "quickstart.txt" is uploaded
Local file "./readme.md" is uploaded
Blobs in "demo" container:
 - quickstart.txt
 - readme-stream.md
 - readme.md
Blob downloaded blob content: "hello!"
Blob "quickstart.txt" is deleted
Container "demo" is deleted
Done
```
如果您在本快速入門中使用新的儲存體帳戶，則可能看不到 "*Containers*" 標籤底下所列的容器名稱。

## <a name="understanding-the-code"></a>了解程式碼
此範例一開始會從 Azure Blob 儲存體命名空間匯入一些類別和函式。 匯入的每個項目都將在範例裡用到它們時，於內容中進行討論。

```javascript
const {
    Aborter,
    BlobURL,
    BlockBlobURL,
    ContainerURL,
    ServiceURL,
    SharedKeyCredential,
    StorageURL,
    uploadStreamToBlockBlob
} = require('@azure/storage-blob');
```

系統會根據適當的內容，從環境變數中讀取認證。

```javascript
if (process.env.NODE_ENV !== 'production') {
    require('dotenv').load();
}
```

*dotenv* 模組會在本機執行應用程式進行偵錯時載入環境變數。 值均定義於名為 *.env* 的檔案中，並載入到目前的執行內容。 在生產環境中，伺服器設定會提供這些值，這就是為什麼只有當指令碼「不是」在「生產」環境下執行時，這段程式碼才會執行。

模組的下一個區塊會匯入來協助連接檔案系統。

```javascript
const fs = require('fs');
const path = require('path');
```

這些模組的用途如下： 

- *fs* 為原生的 Node.js 模組，可用來處理檔案系統

- 需要有 *path*，才能判斷檔案的絕對路徑，此路徑會在將檔案上傳到 Blob 儲存體時使用

接著，會讀取環境變數值並保留於常數中。

```javascript
const STORAGE_ACCOUNT_NAME = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const ACCOUNT_ACCESS_KEY = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
```
下一組常數有助於在上傳作業期間顯示計算檔案大小的意圖。
```javascript
const ONE_MEGABYTE = 1024 * 1024;
const FOUR_MEGABYTES = 4 * ONE_MEGABYTE;
```
由 API 所提出的要求可以設定為在一段指定時間間隔之後逾時。 [Aborter](/javascript/api/%40azure/storage-blob/aborter?view=azure-node-preview) 類別負責管理要求的逾時方式，而後續的常數則可用來定義此範例中所使用的逾時。
```javascript
const ONE_MINUTE = 60 * 1000;
```
### <a name="calling-code"></a>呼叫程式碼

為了支援 JavaScript 的 *async/await* 語法，所有呼叫程式碼都會包裝在名為 *execute* 的函式中。 接著會呼叫 *execute* 並當作 promise 處理。

```javascript
async function execute() {
    // commands... 
}

execute().then(() => console.log("Done")).catch((e) => console.log(e));
```
下列程式碼全都在已放置 `// commands...` 註解的 execute 函式內執行。

首先會宣告相關變數來指派名稱、範例內容，以及指向要上傳至 Blob 儲存體的本機檔案。

```javascript
const containerName = "demo";
const blobName = "quickstart.txt";
const content = "hello!";
const localFilePath = "./readme.md";
```

帳戶認證可用來建立管線，其負責管理將要求傳送至 REST API 的方式。 管線是安全執行緒，會指定適用於重試原則、記錄、HTTP 回應還原序列化規則及更多項目的邏輯。

```javascript
const credentials = new SharedKeyCredential(STORAGE_ACCOUNT_NAME, ACCOUNT_ACCESS_KEY);
const pipeline = StorageURL.newPipeline(credentials);
const serviceURL = new ServiceURL(`https://${STORAGE_ACCOUNT_NAME}.blob.core.windows.net`, pipeline);
```
這個程式碼區塊中會使用下列類別：

- [SharedKeyCredential](/javascript/api/%40azure/storage-blob/sharedkeycredential?view=azure-node-preview) 類別負責包裝儲存體帳戶認證，以將其提供給要求管線。

- [StorageURL](/javascript/api/%40azure/storage-blob/storageurl?view=azure-node-preview) 類別負責建立新的管線。

- [ServiceURL](/javascript/api/%40azure/storage-blob/serviceurl?view=azure-node-preview) 會為 REST API 中所使用的 URL 設定模型。 此類別的執行個體可讓您執行如下的動作：列出容器，並提供內容資訊來產生容器 URL。

*ServiceURL* 的執行個體可與 [ContainerURL](/javascript/api/%40azure/storage-blob/containerurl?view=azure-node-preview) 和 [BlockBlobURL](/javascript/api/%40azure/storage-blob/blockbloburl?view=azure-node-preview) 執行個體搭配使用，以管理儲存體帳戶中的容器和 Blob。

```javascript
const containerURL = ContainerURL.fromServiceURL(serviceURL, containerName);
const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, blobName);
```
*containerURL* 和 *blockBlobURL* 變數會在整個範例中重複使用，以根據儲存體帳戶採取行動。 

此容器目前不存在儲存體帳戶內。 *ContainerURL* 的執行個體表示您可據以採取動作的 URL。 使用這個執行個體，您可以建立和刪除容器。 此容器的位置相當於如下的位置：

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo
```

*blockBlobURL* 可用來管理個別的 Blob，讓您能夠上傳、下載及刪除 Blob 內容。 此處所呈現的 URL 類似此位置：

```bash
https://<ACCOUNT_NAME>.blob.core.windows.net/demo/quickstart.txt
```
使用容器時，區塊 Blob 尚未存在。 *blockBlobURL* 變數稍後可用於透過上傳內容來建立 Blob。

### <a name="using-the-aborter-class"></a>使用 Aborter 類別

由 API 所提出的要求可以設定為在一段指定時間間隔之後逾時。 *Aborter* 類別負責管理要求逾時的方式。下列程式碼所建立的內容會指定一組要求需在 30 分鐘內執行。

```javascript
const aborter = Aborter.timeout(30 * ONE_MINUTE);
```
Aborters 讓您能夠透過下列方式控制要求：

- 針對要求批次指定給定的時間量
- 指定個別的要求必須在批次中執行的時間長度
- 讓您能夠取消要求
- 使用 Aborter.none 靜態成員來防止所有要求一起逾時

### <a name="show-container-names"></a>顯示容器名稱
帳戶可以儲存大量容器。 下列程式碼示範如何以分段方式列出容器，讓您能夠循環瀏覽大量容器。 *showContainerNames* 函式是 *ServiceURL* 和 *Aborter* 的過時執行個體。

```javascript
console.log("Containers:");
await showContainerNames(serviceURL, aborter);
```
*showContainerNames* 函式會使用 *listContainersSegment* 方法，向儲存體帳戶要求容器名稱的批次。
```javascript
async function showContainerNames(aborter, serviceURL) {

    let response;
    let marker;

    do {
        response = await serviceURL.listContainersSegment(aborter, marker);
        marker = response.marker;
        for(let container of response.containerItems) {
            console.log(` - ${ container.name }`);
        }
    } while (marker);
}
```
回應傳回時，則會逐一查看 *containerItems*，以將名稱記錄到主控台。 

### <a name="create-a-container"></a>建立容器

若要建立容器，可使用 *ContainerURL* 的 *create* 方法。

```javascript
await containerURL.create(aborter);
console.log(`Container: "${containerName}" is created`);
```
由於容器名稱會在呼叫 *ContainerURL.fromServiceURL(serviceURL, containerName)* 時定義，因此，只需呼叫 *create* 方法就能建立容器。

### <a name="upload-text"></a>上傳文字
若要將文字上傳到 Blob，請使用 *upload* 方法。
```javascript
await blockBlobURL.upload(aborter, content, content.length);
console.log(`Blob "${blobName}" is uploaded`);
```
這裡的文字及其長度均會傳遞到該方法。
### <a name="upload-a-local-file"></a>上傳本機檔案
若要將本機檔案上傳到容器，您需要容器 URL 和檔案的路徑。
```javascript
await uploadLocalFile(aborter, containerURL, localFilePath);
console.log(`Local file "${localFilePath}" is uploaded`);
```
*uploadLocalFile* 函式會呼叫 *uploadFileToBlockBlob* 函式，將檔案路徑和區塊 Blob 的目的地執行個體當成引數。
```javascript
async function uploadLocalFile(aborter, containerURL, filePath) {

    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath);
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    return await uploadFileToBlockBlob(aborter, filePath, blockBlobURL);
}
```
### <a name="upload-a-stream"></a>上傳串流
目前也支援上傳串流。 此範例會開啟本機檔案以作為串流來傳遞到 upload 方法。
```javascript
await uploadStream(containerURL, localFilePath, aborter);
console.log(`Local file "${localFilePath}" is uploaded as a stream`);
```
*uploadStream* 函式會呼叫 *uploadStreamToBlockBlob*，以將串流上傳到儲存體容器。
```javascript
async function uploadStream(aborter, containerURL, filePath) {

    filePath = path.resolve(filePath);

    const fileName = path.basename(filePath).replace('.md', '-stream.md');
    const blockBlobURL = BlockBlobURL.fromContainerURL(containerURL, fileName);

    const stream = fs.createReadStream(filePath, {
      highWaterMark: FOUR_MEGABYTES,
    });

    const uploadOptions = {
        bufferSize: FOUR_MEGABYTES,
        maxBuffers: 5,
    };

    return await uploadStreamToBlockBlob(
                    aborter, 
                    stream, 
                    blockBlobURL, 
                    uploadOptions.bufferSize, 
                    uploadOptions.maxBuffers);
}
```
上傳期間，*uploadStreamToBlockBlob* 會配置緩衝區，在需要重試時從串流中快取資料。 *maxBuffers* 值會在每個緩衝區建立個別上傳要求時，指定最多要使用多少個緩衝區。 在理想情況下，更多的緩衝區等同於更高的速度，但代價是需要更多的記憶體使用量。 當緩衝區數目夠高，使得瓶頸會轉換到網路或磁碟而非用戶端時，上傳速度就會穩定。

### <a name="show-blob-names"></a>顯示 Blob 名稱
就像帳戶可以包含許多容器，每個容器都可能包含大量 Blob。 對容器中每個 Blob 的存取權均可透過 *ContainerURL* 類別的執行個體來取得。 
```javascript
console.log(`Blobs in "${containerName}" container:`);
await showBlobNames(aborter, containerURL);
```
*showBlobNames* 函式會呼叫 *listBlobFlatSegment*，向容器要求 Blob 的批次。
```javascript
async function showBlobNames(aborter, containerURL) {

    let response;
    let marker;

    do {
        response = await containerURL.listBlobFlatSegment(aborter);
        marker = response.marker;
        for(let blob of response.segment.blobItems) {
            console.log(` - ${ blob.name }`);
        }
    } while (marker);
}
```
### <a name="download-a-blob"></a>下載 Blob
建立 Blob 之後，您可以使用 *download* 方法來下載內容。
```javascript
const downloadResponse = await blockBlobURL.download(aborter, 0);
const downloadedContent = downloadResponse.readableStreamBody.read(content.length).toString();
console.log(`Downloaded blob content: "${downloadedContent}"`);
```
回應會以串流形式傳回。 在此範例中，串流會轉換為字串以記錄到主控台。
### <a name="delete-a-blob"></a>刪除 Blob
*BlockBlobURL* 執行個體的 *delete* 方法會從容器中刪除 Blob。
```javascript
await blockBlobURL.delete(aborter)
console.log(`Block blob "${blobName}" is deleted`);
```
### <a name="delete-a-container"></a>刪除容器
*ContainerURL* 執行個體的 *delete* 方法會從儲存體帳戶中刪除容器。
```javascript
await containerURL.delete(aborter);
console.log(`Container "${containerName}" is deleted`);
```
## <a name="clean-up-resources"></a>清除資源
寫入儲存體帳戶的所有資料都會在程式碼範例的結尾自動刪除。 

## <a name="next-steps"></a>後續步驟

本快速入門示範如何使用 Node.js 來管理 Azure Blob 儲存體中的 Blob 和容器。 若要深入了解此 SDK 的用法，請參閱 GitHub 存放庫。

> [!div class="nextstepaction"]
> [適用於 JavaScript 的 Azure 儲存體 v10 SDK 存放庫](https://github.com/Azure/azure-storage-js)