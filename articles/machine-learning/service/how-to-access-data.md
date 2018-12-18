---
title: 使用 Azure Machine Learning 中的資料存放區來存取資料
description: 如何在定型期間使用資料存放區來存取資料儲存體
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: minxia
author: mx-iao
ms.reviewer: sgilley
ms.date: 09/24/2018
ms.openlocfilehash: 615ab592c040eedf7d31e3a3036f558ea6c09740
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990802"
---
# <a name="how-to-access-data-during-training"></a>如何在定型期間存取資料
在 Azure Machine Learning 服務中，資料存放區是 [Azure 儲存體](https://docs.microsoft.com/azure/storage/common/storage-introduction)的抽象概念。 資料存放區可參照 [Azure Blob](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction) 容器或 [Azure 檔案共用](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)作為底層儲存體。 資料存放區可讓您在 Azure Machine Learning 工作流程期間，透過 Python SDK 或 CLI 輕鬆存取資料儲存體並與它互動。

## <a name="create-a-datastore"></a>建立資料存放區
若要使用資料存放區，必須先有[工作區](concept-azure-machine-learning-architecture.md#workspace)。 您可以建立新工作區，或擷取現有工作區：

```Python
import azureml.core
from azureml.core import Workspace

ws = Workspace.from_config()
```

### <a name="use-the-default-datastore"></a>使用預設資料存放區
每個工作區都有一個可立即開始使用的預設資料存放區，讓您不需要花時間建立並設定自己的儲存體帳戶。

取得工作區的預設資料存放區：
```Python
ds = ws.get_default_datastore()
```

### <a name="register-a-datastore"></a>註冊資料存放區
如果您已經有現有的 Azure 儲存體，可以在工作區中將它註冊為資料存放區。 Azure Machine Learning 提供將 Azure Blob 容器或 Azure 檔案共用註冊為資料存放區的功能。 所有註冊方法都在 `Datastore` 類別上且形式為 `register_azure_*`。

#### <a name="azure-blob-container-datastore"></a>Azure Blob 容器資料存放區
註冊 Azure Blob 容器資料存放區：

```Python
ds = Datastore.register_azure_blob_container(workspace=ws, 
                                             datastore_name='your datastore name', 
                                             container_name='your azure blob container name',
                                             account_name='your storage account name', 
                                             account_key='your storage account key',
                                             create_if_not_exists=True)
```

#### <a name="azure-file-share-datastore"></a>Azure 檔案共用資料存放區
註冊 Azure 檔案共用資料存放區：

```Python
ds = Datastore.register_azure_file_share(workspace=ws, 
                                         datastore_name='your datastore name', 
                                         container_name='your file share name',
                                         account_name='your storage account name', 
                                         account_key='your storage account key',
                                         create_if_not_exists=True)
```

### <a name="get-an-existing-datastore"></a>取得現有資料庫
依名稱查詢註冊的資料存放區：
```Python
ds = Datastore.get(ws, datastore_name='your datastore name')
```

您也可以取得工作區的所有資料存放區：
```Python
datastores = ws.datastores()
for name, ds in datastores.items(),
    print(name, ds.datastore_type)"
```

為了方便起見，如果想要將已註冊的其中一個資料存放區設定為工作區的預設資料存放區，可以依照以下方式執行此作業：
```Python
ws.set_default_datastore('your datastore name')
```

## <a name="upload-and-download-data"></a>上傳及下載資料
### <a name="upload"></a>上傳
您可以使用 Python SDK 上傳目錄或個別檔案至資料存放區。

上傳目錄至資料存放區 `ds`：
```Python
ds.upload(src_dir='your source directory',
          target_path='your target path',
          overwrite=True,
          show_progress=True)
```
`target_path` 可指定檔案共用 (或 Blob 容器) 中的位置以進行上傳。 該位置預設為 `None`，這表示會將資料上傳至根目錄。 `overwrite=True` 將會覆寫位於 `target_path` 的任何現有資料。

或者，您可以透過資料存放區的 `upload_files()` 方法，將個別檔案清單上傳至資料存放區。

### <a name="download"></a>下載
同樣地，您可以從資料存放區下載資料至本機檔案系統。

```Python
ds.download(target_path='your target path',
            prefix='your prefix',
            show_progress=True)
```
`target_path` 是儲存下載資料的本機目錄位置。 若要指定檔案共用 (或 Blob 容器) 中資料夾的路徑以進行下載，請在 `prefix` 中提供該路徑。 如果 `prefix` 是 `None`，表示將會下載檔案共用 (或 Blob 容器) 的所有內容。

## <a name="access-datastores-for-training"></a>存取資料存放區以進行定型
您可以透過 Python SKD 於定型回合 (例如，定型或驗證資料) 期間在遠端計算目標上存取資料存放區。 

有兩種支援的方式可讓資料存放區能夠在遠端計算中使用：
* **掛接**  
`ds.as_mount()`：透過指定此掛接模式，資料存放區將會掛接至遠端計算以供您使用。 
* **下載/上傳**  
    * `ds.as_download(path_on_compute='your path on compute')`：使用此下載模式時，會將資料從資料存放區下載到遠端計算中透過 `path_on_compute` 指定的位置。
    * 相反地，也可以將您的定型回合所產生的資料上傳至資料存放區。 例如，如果定型指令碼在遠端計算的目前工作目錄中建立了一個 `foo.pkl` 檔案，可以指定在指令碼執行之後將它上傳至資料存放區：`ds.as_upload(path_on_compute='./foo.pkl')`。 這會將檔案上傳至您資料存放區的根目錄。
    
如果想要參照資料存放區中的特定資料夾或檔案，可以使用資料存放區的 **`path`** 函式。 例如，如果資料存放區有包含相對路徑 `./bar` 的目錄，且您只想下載此資料夾的內容至計算目標，可以依照以下方式執行此作業：`ds.path('./bar').as_download()`

任何 `ds` 或 `ds.path` 物件都會解析為 `"$AZUREML_DATAREFERENCE_XXXX"`，其值代表遠端計算上的掛接/下載路徑。 遠端計算上的資料存放區路徑可能與指令碼的執行路徑不同。

若要在定型期間存取資料存放區，可以透過 `script_params` 將它傳遞至定型指令碼中作為命令列引數：

```Python
from azureml.train.estimator import Estimator

script_params = {
    '--data_dir': ds.as_mount()
}

est = Estimator(source_directory='your code directory',
                script_params=script_params,
                compute_target=compute_target,
                entry_script='train.py')
```
`as_mount()` 是資料存放區的預設模式，因此您也可以直接將 `ds` 傳遞至 `'--data_dir'` 引數。

或者，您可以使用資料存放區清單傳遞至估算程式建構函式的 `inputs` 參數，以掛接或複製到資料存放區 (或從資料存放區複製)：

```Python
est = Estimator(source_directory='your code directory',
                compute_target=compute_target,
                entry_script='train.py',
                inputs=[ds1.as_download(), ds2.path('./foo').as_download(), ds3.as_upload(path_on_compute='./bar.pkl')])
```
上述程式碼將會：
* 在執行定型指令碼 `train.py` 之前，先將資料存放區 `ds1` 中的所有內容下載至遠端計算
* 在執行 `train.py` 之前，先將資料存放區 `ds2` 中的資料夾 `'./foo'` 下載至遠端計算
* 在執行指令碼之後，將檔案 `'./bar.pkl'` 從遠端計算上傳至資料存放區 `d3`

## <a name="next-steps"></a>後續步驟
* [將模型定型](how-to-train-ml-models.md)
