---
title: Azure IoT Edge 裝置上的 Azure Blob 儲存體 | Microsoft Docs
description: 將 Azure Blob 儲存體模組部署到您的 IoT Edge 裝置，即可在邊緣儲存資料。
author: kgremban
manager: timlt
ms.author: kgremban
ms.reviewer: arduppal
ms.date: 09/20/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: b9e48eba4b46f024b056fe53b3b3df24feec802e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995664"
---
# <a name="store-data-at-the-edge-with-azure-blob-storage-on-iot-edge-preview"></a>在 IoT Edge (預覽) 使用 Azure Blob 儲存體，以便在邊緣儲存資料

IoT Edge 上的 Azure Blob 儲存體提供邊緣的區塊 Blob 儲存體解決方案。 IoT Edge 裝置上的 Blob 儲存體模組如同 Azure Blob 服務般運作，但是區塊 Blob 會在本機 IoT Edge 裝置上儲存。 您可以使用平常使用的相同 Azure 儲存體 SDK 方法或區塊 Blob API 呼叫存取您的 Blob。 

本文提供在對於 IoT Edge 裝置執行 Blob 服務的 IoT Edge 容器上部署 Azure Blob 儲存體的指示。 

>[!NOTE]
>IoT Edge 上的 Azure Blob 儲存體屬於[公開預覽](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 

## <a name="prerequisites"></a>必要條件

Azure IoT Edge 裝置：

* 您可以遵循 [Linux](quickstart-linux.md) 或 [Windows 裝置](quickstart.md)快速入門中的步驟，使用您的開發電腦或虛擬機器作為邊緣裝置。
* IoT Edge 模組上的 Azure Blob 儲存體支援下列裝置設定：

   | 作業系統 | 架構 |
   | ---------------- | ------------ |
   | Ubuntu Server 16.04 | AMD64 |
   | Ubuntu Server 18.04 | AMD64 |
   | Windows 10 IoT Core (10 月更新) | AMD64 |
   | Windows 10 IoT Enterprise (10 月更新) | AMD64 |
   | Windows Server 2019 | AMD64 |
   | Raspbian Stretch | ARM32 |

雲端資源：

* Azure 中的標準層 [IoT 中樞](../iot-hub/iot-hub-create-through-portal.md)。 


## <a name="deploy-blob-storage-to-your-device"></a>將 Blob 儲存體部署至您的裝置

IoT Edge 上的 Azure Blob 儲存體提供三個標準的容器映像，其中兩個適用於 Linux 容器 (AMD64 和 ARM32 架構)，其中一個適用於 Windows 容器 (AMD64)。 當您使用其中一個模組映像將 Blob 儲存體部署到您的 IoT Edge 裝置時，您可提供這三項資訊來設定您裝置的模組執行個體：

* [帳戶名稱] 和 [帳戶金鑰]。 若要與 Azure 儲存體保持一致，Blob 儲存體模組會使用帳戶名稱和帳戶金鑰來管理存取權。 帳戶名稱應該有 3 到 24 個字元，包含小寫字母和數字。 帳戶金鑰應該是 base64 編碼和 64 個位元組的長度。 您可以 [GeneratePlus](https://generate.plus/en/base64) 之類的工具產生金鑰。
* **本機儲存體選項**。 Blob 儲存體模組會在本機 IoT Edge 裝置儲存 Blob，因此，如果模組停止或重新啟動，Blob 仍會保留。 宣告應該在您的裝置上儲存 Blob 的現有[磁碟區](https://docs.docker.com/storage/volumes
)或本機資料夾路徑。 

有數種方式可將模組部署到 IoT Edge 裝置，而且這些方式皆適用於 IoT Edge 模組上的 Azure Blob 儲存體。 兩個最簡單的方法是使用 Azure 入口網站或 Visual Studio Code 範本。 

### <a name="azure-portal"></a>Azure 入口網站

若要透過 Azure 入口網站部署 Blob 儲存體，請依照[從 Azure 入口網站部署 Azure IoT Edge 模組](how-to-deploy-modules-portal.md)中的步驟進行。 您開始部署模組之前，請先複製映像 URI，並按照您的容器作業系統準備容器建立選項。 使用部署文章**設定部署資訊清單**區段中的這些值。 

提供 Blob 儲存體模組的映像 URI：**mcr.microsoft.com/azure-blob-storage**。 
   
在 [容器建立選項] 欄位使用下列 JSON 範本。 使用您的儲存體帳戶名稱、儲存體帳戶金鑰和儲存體目錄繫結來設定 JSON。  
   
   ```json
   {
       "Env":[
           "LOCAL_STORAGE_ACCOUNT_NAME=<your storage account name>",
           "LOCAL_STORAGE_ACCOUNT_KEY=<your storage account key>"
       ],
       "HostConfig":[
           "Binds":[
               "<storage directory bind>"
           ],
           "PortBindings":{
               "11002/tcp":[{"HostPort":"11002"}]
           }
       ]
   }
   ```   
   
在建立選項 JSON 中，使用任何名稱更新 `\<your storage account name\>`。 使用 64 位元組 base64 金鑰更新 `\<your storage account key\>`。 您可以使用 [GeneratePlus](https://generate.plus/en/base64) (可讓您選擇您的位元組長度) 之類的工具產生金鑰。 您將使用這些認證，從其他模組存取 Blob 儲存體。

在建立選項 JSON 中，按照您的容器作業系統更新 `<storage directory bind>`。 提供[磁碟區](https://docs.docker.com/storage/volumes/)的名稱，或您想要 Blob 模組儲存其資料的 IoT Edge 裝置目錄絕對路徑。  

   * Linux 容器：[\<儲存體路徑>:/blobroot]。 例如，/srv/containerdata:/blobroot。 或者，my-volume:/blobroot。 
   * Windows 容器：[\<儲存體路徑>:C:/BlobRoot]。 例如，C:/ContainerData:C:/BlobRoot。 或者，my-volume:C:/blobroot。 


您不需要提供登錄認證，即可存取 IoT Edge 上的 Azure Blob 儲存體，而且您不需要對於您的部署宣告任何路由。 

### <a name="visual-studio-code-templates"></a>Visual Studio Code 範本

Azure IoT Edge 提供 Visual Studio Code 中的範本協助您開發解決方案。 這些步驟需要您在您的開發機器上安裝 [Visual Studio Code](https://code.visualstudio.com/)，並設定 [Azure IoT Edge 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge)。

下列步驟可用來使用 Blob 儲存體模組建立新的 IoT Edge 解決方案，並設定部署資訊清單。 

1. 選取 [檢視] > [命令選擇區]。 

2. 在命令選擇區中，輸入並執行命令 **Azure IoT Edge: New IoT Edge Solution**。 

3. 請依照下列提示來建立新的解決方案： 

   1. **選取資料夾** - 瀏覽至要用來建立新解決方案的資料夾。  
   
   2. **為解決方案提供名稱** - 輸入解決方案的名稱，或接受預設值。
   
   3. **選取模組範本** - 選擇**現有模組 (輸入完整映像 URL)**。
   
   4. **為模組提供名稱** - 輸入可辨識的模組名稱，例如 **azureBlobStorage**。
   
   5. **提供模組的 Docker 映像** - 提供映像 URI：**mcr.microsoft.com/azure-blob-storage**

VS Code 會採用您提供的資訊、建立 IoT Edge 解決方案，然後將其載入至新的視窗。 

解決方案範本會建立部署資訊清單範本，其中包含 Blob 儲存體模組映像，但是您需要設定模組的建立選項。 

1. 在新的解決方案工作區中開啟 [deployment.template.json]，並找出 [模組] 區段。 

2. 刪除這次部署不需要的 **tempSensor** 模組。 

3. 複製下列程式碼，並在 Blob 儲存體模組的 [createOptions] 欄位貼上： 

   ```json
   {\"Env\": [\"LOCAL_STORAGE_ACCOUNT_NAME=$STORAGE_ACCOUNT_NAME\",\" LOCAL_STORAGE_ACCOUNT_KEY=$STORAGE_ACCOUNT_KEY\"],\"HostConfig\": {\"Binds\": [\"<storage directory bind>\"],\"PortBindings\": {\"11002/tcp\": [{\"HostPort\":\"11002\"}]}}}
   ```

   ![更新模組建立選項](./media/how-to-store-data-blob/create-options.png)

4. 在建立選項 JSON 中，按照您的容器作業系統更新 `<storage directory bind>`。 提供[磁碟區](https://docs.docker.com/storage/volumes/)的名稱，或您想要 Blob 模組儲存其資料的 IoT Edge 裝置目錄絕對路徑。  

   * Linux 容器：[\<儲存體路徑>:/blobroot]。 例如，/srv/containerdata:/blobroot。 或者，my-volume:/blobroot。
   * Windows 容器：[\<儲存體路徑>:C:/BlobRoot]。 例如，C:/ContainerData:C:/BlobRoot。 或者，my-volume:C:/blobroot。

5. 儲存 **deployment.template.json**。

6. 在解決方案工作區中開啟 **.env**。 

7. 對於 Blob 儲存體映像的，您不需要輸入任何容器登錄值，因為這是公開可用的值。 不過，請新增兩個新的環境變數： 

   ```env
   STORAGE_ACCOUNT_NAME=
   STORAGE_ACCOUNT_KEY=
   ```

8. 請為儲存體帳戶名稱提供任何名稱，並為儲存體帳戶金鑰提供 64 位元組的 base64 金鑰。 您可以 [GeneratePlus](https://generate.plus/en/base64) 之類的工具產生金鑰。 您將使用這些認證，從其他模組存取 Blob 儲存體。 

9. 儲存 **.env**。 

10. 以滑鼠右鍵按一下 **deployment.template.json**，然後選取 [產生 IoT Edge 部署資訊清單]。 

Visual Studio Code 會採用您在 deployment.template.json 和 .env 中提供的資訊，並使用這些資訊建立新的部署資訊清單檔。 隨即在解決方案工作區新 **config** 資料夾中建立部署資訊清單。 您有該檔案時，就可以依照[從 Visual Studio Code 部署 Azure IoT Edge 模組](how-to-deploy-modules-vscode.md)或[使用 Azure CLI 2.0 部署 Azure IoT Edge 模組](how-to-deploy-modules-cli.md)中的步驟進行。

## <a name="connect-to-your-blob-storage-module"></a>連接到您的 Blob 儲存體模組

您可以使用帳戶名稱和您為模組設定的帳戶金鑰來存取 IoT Edge 裝置上的 Blob 儲存體。 

指定 IoT Edge 裝置作為您提出的任何儲存體要求所用的 Blob 端點。 您可以使用 IoT Edge 裝置資訊和您設定的帳戶名稱，[建立明確儲存體端點的連接字串](../storage/common/storage-configure-connection-string.md#create-a-connection-string-for-an-explicit-storage-endpoint)。 

IoT Edge 上的 Azure Blob 儲存體所用的 Blob 端點是 `http://<IoT Edge device hostname>:11002/<account name>`。 

## <a name="deploy-multiple-instances"></a>部署多個執行個體

如果您想要部署 IoT Edge 上的多個 Azure Blob 儲存體執行個體，您只需要變更模組所繫結的 HostPort。 Blob 儲存體模組一律會公開容器的 11002 連接埠，但是您可以宣告哪個連接埠繫結至主機。 

編輯模組建立選項，以變更 HostPort 值：

```json
\"PortBindings\": {\"11002/tcp\": [{\"HostPort\":\"<port number>\"}]}
```

您連接到其他 Blob 儲存體模組時，請變更端點來指向已更新的主機連接埠。 

### <a name="try-it-out"></a>試做

Azure Blob 儲存體文件集包括有數種語言的範例程式碼的快速入門。 您可以執行這些範例變更 Blob 端點指向 Blob 儲存體模組，以便在 IoT Edge 上測試 Azure Blob 儲存體。

下列快速入門使用 IoT Edge 也支援的語言，因此您可以連同 Blob 儲存體模組將這些語言部署為 IoT Edge 模組：

* [.NET](../storage/blobs/storage-quickstart-blobs-dotnet.md)
* [Java](../storage/blobs/storage-quickstart-blobs-java.md)
* [Python](../storage/blobs/storage-quickstart-blobs-python.md)
* [Node.js](../storage/blobs/storage-quickstart-blobs-nodejs.md)

## <a name="supported-storage-operations"></a>支援的儲存體作業

IoT Edge 上的 Blob 儲存體模組使用相同的 Azure 儲存體 SDK，而且與區塊 Blob 端點的 Azure 儲存體 API 2018-03-28 版本一致。 日後的版本端視客戶的需求而定。 

IoT Edge 上的 Azure Blob 儲存體並未完全支援所有的 Azure Blob 儲存體作業。 下列各節詳細說明如何不支援的作業。 

### <a name="account"></a>帳戶

支援： 
* 列出容器

不支援： 
* 取得和設定 Blob 服務屬性
* 預檢 Blob 要求
* 取得 Blob 服務統計資料
* 取得帳戶資訊

### <a name="containers"></a>容器

支援： 
* 建立和刪除容器
* 取得容器屬性和中繼資料
* 列出 Blob

不支援： 
* 取得和設定容器 ACL
* 租用容器
* 設定容器中繼資料

### <a name="blobs"></a>Blob

支援： 
* 放置、取得和刪除 Blob
* 取得和設定 Blob 屬性
* 取得和設定 Blob 中繼資料

不支援： 
* 租用 Blob
* 拍攝 Blob 的快照
* 複製和中止複製 Blob
* 取消刪除 Blob
* 設定 Blob 層

### <a name="block-blobs"></a>區塊 Blob

支援： 
* 放置區塊
* 放置和取得區塊清單

不支援：
* 從 URL 放置區塊

## <a name="next-steps"></a>後續步驟

深入了解 [Azure Blob 儲存體](../storage/blobs/storage-blobs-introduction.md)

