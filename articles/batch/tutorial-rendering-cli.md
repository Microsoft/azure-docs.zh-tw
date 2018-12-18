---
title: 轉譯雲端中的場景 - Azure Batch
description: 教學課程 - 如何使用 Batch 轉譯服務和 Azure 命令列介面，透過 Arnold 轉譯 Autodesk 3ds Max 場景
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.topic: tutorial
ms.date: 09/25/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: ff52c0fa647dd0e86b22bcfdf7af04062a135f94
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392800"
---
# <a name="tutorial-render-a-scene-with-azure-batch"></a>教學課程：使用 Azure Batch 轉譯場景 

Azure Batch 提供了按使用次數付費的雲端規模轉譯功能。 Azure Batch 支援 Autodesk Maya、3ds Max、Arnold 與 V-Ray 等應用程式的轉譯。 此教學課程會示範使用 Azure 命令列介面透過 Batch 轉譯小型場景的步驟。 您會了解如何：

> [!div class="checklist"]
> * 將場景上傳至 Azure 儲存體
> * 建立 Batch 集區以供轉譯
> * 轉譯單一框架場景
> * 調整集區，以及轉譯多框架場景
> * 下載已轉譯的輸出

在此教學課程中，您會使用 [Arnold](https://www.autodesk.com/products/arnold/overview) 光線追蹤轉譯器，透過 Batch 轉譯 3ds Max 場景。 Batch 集區會使用 Azure Marketplace 映像，其中包含提供即用即付授權的預先安裝圖表與轉譯應用程式。

## <a name="prerequisites"></a>先決條件

您需要預付型訂用帳戶或其他 Azure 購買選項，以按使用量付費的方式，在 Batch 中使用轉譯應用程式。 如果您使用提供信用額度金額的免費 Azure 方案，則不支援按使用量付費授權。

此教學課程中的範例 3ds Max 場景 (連同範例 Bash 指令碼和 JSON 設定檔) 位於 [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/tree/master/batch/render-scene) 上。 3ds Max 場景來自 [Autodesk 3ds Max 範例檔案](http://download.autodesk.com/us/support/files/3dsmax_sample_files/2017/Autodesk_3ds_Max_2017_English_Win_Samples_Files.exe)。 (在 Creative Commons Attribution-NonCommercial-Share Alike 授權之下可取得 Autodesk 3ds Max 範例檔案。 Copyright © Autodesk, Inc.)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，此教學課程會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="create-a-batch-account"></a>建立批次帳戶：

如果您還沒有 Batch 帳戶，請在您的訂用帳戶中建立資源群組、Batch 帳戶，以及已連結的儲存體帳戶。 

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 下列範例會在 eastus2 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create \
    --name myResourceGroup \
    --location eastus2
```

使用 [az storage account create](/cli/azure/storage/account#az-storage-account-create) 命令在資源群組中建立 Azure 儲存體帳戶。 在此教學課程中，您會使用此儲存體帳戶來存放輸入 3ds Max 場景和已轉譯的輸出。

```azurecli-interactive
az storage account create \
    --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus2 \
    --sku Standard_LRS
```
使用 [az batch account create](/cli/azure/batch/account#az-batch-account-create) 命令建立 Batch 帳戶。 下列範例會在 myResourceGroup 中建立名為 mybatchaccount 的 Batch 帳戶，並連結您所建立的儲存體帳戶。  

```azurecli-interactive 
az batch account create \
    --name mybatchaccount \
    --storage-account mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus2
```

若要建立及管理計算集區和作業，您需要向 Batch 進行驗證。 使用 [az batch account login](/cli/azure/batch/account#az-batch-account-login) 命令登入帳戶。 登入之後，`az batch` 命令會使用此帳戶內容。 下列範例會根據 Batch 帳戶名稱和金鑰，使用共用金鑰驗證。 Batch 也支援透過 [Azure Active Directory](batch-aad-auth.md) 進行驗證，以便驗證個別使用者或自動執行的應用程式。

```azurecli-interactive 
az batch account login \
    --name mybatchaccount \
    --resource-group myResourceGroup \
    --shared-key-auth
```
## <a name="upload-a-scene-to-storage"></a>將場景上傳至儲存體

若要將輸入場景上傳至儲存體，您必須先存取儲存體帳戶，並建立 blob 的目的地容器。 若要存取 Azure 儲存體帳戶，請匯出 `AZURE_STORAGE_KEY` 和 `AZURE_STORAGE_ACCOUNT` 環境變數。 第一個 Bash Shell 命令會使用 [az storage account keys list](/cli/azure/storage/account/keys#az-storage-account-keys-list) 命令來取得第一個帳戶金鑰。 設定這些環境變數之後，您的儲存體命令就會使用此帳戶內容。

```azurecli-interactive
export AZURE_STORAGE_KEY=$(az storage account keys list --account-name mystorageaccount --resource-group myResourceGroup -o tsv --query [0].value)

export AZURE_STORAGE_ACCOUNT=mystorageaccount
```

現在，在儲存體帳戶中建立場景檔案的 Blob 容器。 下列範例會使用 [az storage container create](/cli/azure/storage/container#az-storage-container-create) 命令建立名為 scenefiles 且允許公用讀取存取的 blob 容器。

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name scenefiles
```

從 [GitHub](https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max) 將 `MotionBlur-Dragon-Flying.max` 場景下載到本機工作目錄。 例如︰

```azurecli-interactive
wget -O MotionBlur-DragonFlying.max https://github.com/Azure/azure-docs-cli-python-samples/raw/master/batch/render-scene/MotionBlur-DragonFlying.max
```

將本機工作目錄中的場景檔案上傳到 blob 容器。 下列範例會使用 [az storage blob upload-batch](/cli/azure/storage/blob#az-storage-blob-upload-batch) 命令，該命令可以上傳多個檔案：

```azurecli-interactive
az storage blob upload-batch \
    --destination scenefiles \
    --source ./
```

## <a name="create-a-rendering-pool"></a>建立轉譯集區

使用 [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create) 命令建立 Batch 集區以供轉譯。 在此範例中，您會在 JSON 檔案中指定集區設定。 在您目前的 Shell 中，建立名為 mypool.json 的檔案，然後複製並貼上下列內容。 請務必正確地複製所有文字。 (您可以從 [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/mypool.json) 下載此檔案。)


```json
{
  "id": "myrenderpool",
  "vmSize": "standard_d2_v2",
  "virtualMachineConfiguration": {
    "imageReference": {
      "publisher": "batch",
      "offer": "rendering-windows2016",
      "sku": "rendering",
      "version": "1.2.1"
    },
    "nodeAgentSKUId": "batch.node.windows amd64"
  },
  "targetDedicatedNodes": 0,
  "targetLowPriorityNodes": 1,
  "enableAutoScale": false,
  "applicationLicenses":[
         "3dsmax",
         "arnold"
      ],
  "enableInterNodeCommunication": false 
}
```
Batch 支援專用節點和[低優先順序節點](batch-low-pri-vms.md)，而您可以在集區中使用其中一種或同時使用兩種。 專用節點會保留給您的集區使用。 低優先順序節點則會以較低的價格從 Azure 中的剩餘容量提供。 如果 Azure 沒有足夠的容量，便無法使用低優先順序節點。 

指定的集區包含一個執行 Windows Server 映像 (內含 Batch 轉譯服務軟體) 的低優先順序節點。 此集區已獲得授權可透過 3ds Max 和 Arnold 進行轉譯。 在後面的步驟中，您可將集區調整為更多節點。

透過將 JSON 檔案傳遞至 `az batch pool create` 命令來建立集區：

```azurecli-interactive
az batch pool create \
    --json-file mypool.json
``` 
佈建集區需要幾分鐘的時間。 若要查看集區的狀態，請執行 [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show) 命令。 下列命令可取得集區的配置狀態：

```azurecli-interactive
az batch pool show \
    --pool-id myrenderpool \
    --query "allocationState"
```

繼續執行下列步驟以建立一個作業和數項工作，然而集區狀態一直在改變。 當配置狀態為 `steady` 且節點正在執行時，便已完全佈建集區。  

## <a name="create-a-blob-container-for-output"></a>建立輸入的 Blob 容器

在此教學課程的範例中，轉譯作業中的每個工作都會建立輸出檔案。 在排程作業之前，請在您的儲存體帳戶中建立 blob 容器作為輸出檔案的目的地。 下列範例會使用 [az storage container create](/cli/azure/storage/container#az-storage-container-create) 命令建立具有公用讀取存取權的 job-myrenderjob 容器。 

```azurecli-interactive
az storage container create \
    --public-access blob \
    --name job-myrenderjob
```

為了將輸出檔案寫入容器，Batch 需要使用共用存取簽章 (SAS) 權杖。 使用 [az storage account generate-sas](/cli/azure/storage/account#az-storage-account-generate-sas) 命令建立權杖。 此範例會建立可寫入帳戶中任何 blob 容器的權杖，而該權杖會在 2018 年 11 月 15 日到期：

```azurecli-interactive
az storage account generate-sas \
    --permissions w \
    --resource-types co \
    --services b \
    --expiry 2018-11-15
```

請記下命令所傳回的權杖，如下所示。 您會在後面的步驟中使用此權杖。

```
se=2018-11-15&sp=rw&sv=2017-04-17&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## <a name="render-a-single-frame-scene"></a>轉譯單一框架場景

### <a name="create-a-job"></a>建立工作

使用 [az batch job create](/cli/azure/batch/job#az-batch-job-create) 命令建立要在集區上執行的轉譯作業。 一開始作業沒有任何工作。

```azurecli-interactive
az batch job create \
    --id myrenderjob \
    --pool-id myrenderpool
```

### <a name="create-a-task"></a>建立工作

使用 [az batch task create](/cli/azure/batch/task#az-batch-task-create) 命令在作業中建立轉譯工作。 在此範例中，您會在 JSON 檔案中指定工作設定。 在您目前的 Shell 中，建立名為 myrendertask.json 的檔案，然後複製並貼上下列內容。 請務必正確地複製所有文字。 (您可以從 [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask.json) 下載此檔案。)

此工作會指定 3ds Max 命令，以轉譯 MotionBlur-DragonFlying.max 場景的單一框架。

修改 JSON 檔案中的 `blobSource` 和 `containerURL` 元素，讓它們包含您的儲存體帳戶名稱和 SAS 權杖。 

> [!TIP]
> 您的 `containerURL` 會以 SAS 權杖結尾，如下所示：
> 
> ```
> https://mystorageaccount.blob.core.windows.net/job-myrenderjob/$TaskOutput?se=2018-11-15&sp=rw&sv=2017-04-17&ss=b&srt=co&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
> ```

```json
{
  "id": "myrendertask",
  "commandLine": "cmd /c \"%3DSMAX_2018%3dsmaxcmdio.exe -secure off -v:5 -rfw:0 -start:1 -end:1 -outputName:\"dragon.jpg\" -w 400 -h 300 MotionBlur-DragonFlying.max\"",
  "resourceFiles": [
    {
        "blobSource": "https://mystorageaccount.blob.core.windows.net/scenefiles/MotionBlur-DragonFlying.max",
        "filePath": "MotionBlur-DragonFlying.max"
    }
  ],
    "outputFiles": [
        {
            "filePattern": "dragon*.jpg",
            "destination": {
                "container": {
                    "containerUrl": "https://mystorageaccount.blob.core.windows.net/job-myrenderjob/myrendertask/$TaskOutput?Add_Your_SAS_Token_Here"
                }
            },
            "uploadOptions": {
                "uploadCondition": "TaskSuccess"
            }
        }
    ],
  "userIdentity": {
    "autoUser": {
      "scope": "task",
      "elevationLevel": "nonAdmin"
    }
  }
}
```

請使用下列命令將工作新增至作業：

```azurecli-interactive
az batch task create \
    --job-id myrenderjob \
    --json-file myrendertask.json
```

Batch 會排程工作，而集區中的節點一旦可用，此工作就會執行。


### <a name="view-task-output"></a>檢視工作輸出

此工作需花費數分鐘的時間來執行。 使用 [az batch task show](/cli/azure/batch/task#az-batch-task-show) 命令來檢視工作的詳細資訊。

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id myrendertask
```

此工作會在計算節點上產生 dragon0001.jpg，並將它上傳至您儲存體帳戶中的 job-myrenderjob 容器。 若要檢視輸出，請使用 [az storage blob download](/cli/azure/storage/blob#az-storage-blob-download) 命令，將此檔案從儲存體下載到您的本機電腦。

```azurecli-interactive
az storage blob download \
    --container-name job-myrenderjob \
    --file dragon.jpg \
    --name dragon0001.jpg

```

在電腦上開啟 dragon.jpg。 已轉譯的影像如下所示：

![已轉譯的飛龍框架 1](./media/tutorial-rendering-cli/dragon-frame.png) 


## <a name="scale-the-pool"></a>調整集區

立即修改集區，以準備進行較大型的轉譯作業(具有多個框架)。 Batch 提供數種方式來調整計算資源，包括[自動調整](batch-automatic-scaling.md)，此種方式會隨著工作需求變化來新增或移除節點。 在此基本範例中，使用 [az batch pool resize](/cli/azure/batch/pool#az-batch-pool-resize) 命令，將集區中的低優先順序節點數目增加至 6：

```azurecli-interactive
az batch pool resize --pool-id myrenderpool --target-dedicated-nodes 0 --target-low-priority-nodes 6
```

此集區需花費數分鐘的時間來調整大小。 進行該程序時，請將後續工作設定為在現有的轉譯作業中執行。

## <a name="render-a-multiframe-scene"></a>轉譯多框架場景

如同在單一框架的範例中，使用 [az batch task create](/cli/azure/batch/task#az-batch-task-create) 命令，在名為 myrenderjob 的作業中建立轉譯工作。 在此，請在名為 myrendertask_multi.json 的 JSON 檔案中指定工作設定。 (您可以從 [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-cli-python-samples/master/batch/render-scene/json/myrendertask_multi.json) 下載此檔案。)這六項工作會各自指定 Arnold 命令列，以轉譯 3ds Max 場景 MotionBlur-DragonFlying.max 的一個框架。

在您目前的 Shell 中，建立名為 myrendertask_multi.json 的檔案，然後複製並貼上所下載檔案的內容。 修改 JSON 檔案中的 `blobSource` 和 `containerURL` 元素，使其包含您的儲存體帳戶名稱和 SAS 權杖。 請務必變更這六項工作各自的設定。 儲存檔案，然後執行下列命令以將工作排入佇列：

```azurecli-interactive
az batch task create --job-id myrenderjob --json-file myrendertask_multi.json
```

### <a name="view-task-output"></a>檢視工作輸出

此工作需花費數分鐘的時間來執行。 使用 [az batch task list](/cli/azure/batch/task#az-batch-task-list) 命令來檢視工作的狀態。 例如︰

```azurecli-interactive
az batch task list \
    --job-id myrenderjob \
    --output table
```

使用 [az batch task show](/cli/azure/batch/task#az-batch-task-show) 命令來檢視個別工作的詳細資訊。 例如︰

```azurecli-interactive
az batch task show \
    --job-id myrenderjob \
    --task-id mymultitask1
```
 
此工作會在計算節點上產生名為 dragon0002.jpg - dragon0007.jpg 的輸出檔案，並將它們上傳至您儲存體帳戶中的 job-myrenderjob 容器。 若要檢視輸出，請使用 [az storage blob download-batch](/cli/azure/storage/blob#az-storage-blob-download_batch) 命令，將檔案下載到本機電腦上的資料夾。 例如︰

```azurecli-interactive
az storage blob download-batch \
    --source job-myrenderjob \
    --destination .
```

開啟您電腦上的其中一個檔案。 已轉譯的框架 6 大致如下：

![已轉譯的飛龍框架 6](./media/tutorial-rendering-cli/dragon-frame6.png) 


## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令將資源群組、Batch 帳戶、集區和所有相關資源移除。 刪除資源，如下所示：

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 將場景上傳至 Azure 儲存體
> * 建立 Batch 集區以供轉譯
> * 使用 Arnold 轉譯單一框架場景
> * 調整集區，以及轉譯多框架場景
> * 下載已轉譯的輸出

若要深入了解雲端規模轉譯，請參閱 Batch 轉譯服務的選項。 

> [!div class="nextstepaction"]
> [Batch 轉譯服務](batch-rendering-service.md)
