---
title: 讀取和寫入大型資料檔 | Microsoft Docs
description: 在 Azure Machine Learning 實驗中讀取和寫入大型檔案。
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/10/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 4a2dff4dd57bdb0b010bbb4568d796f1e197a728
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971496"
---
# <a name="persisting-changes-and-working-with-large-files"></a>保存變更和處理大型檔案

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

利用 Azure Machine Learning 測試服務，您可以設定各種不同的執行目標。 某些目標屬於本機，例如本機電腦或本機電腦上的 Docker 容器。 其他則是遠端，例如在遠端電腦或在 HDInsight 叢集上的 Docker 容器。 如需詳細資訊，請參閱 [Azure Machine Learning 實驗執行服務概觀](experimentation-service-configuration.md)。 

在您可於目標上執行之前，您必須將專案資料夾複製到計算目標。 您必須這麼做，即使是利用針對此用途使用本機暫存資料夾的本機執行。 

## <a name="execution-isolation-portability-and-reproducibility"></a>執行隔離、可攜性和重現性
此設計的目的是確保執行的隔離、重現性和可攜性。 如果您在相同或另一個計算目標上執行相同的指令碼兩次，您會收到相同的結果。 在第一個執行期間所做的變更不應影響第二個執行的變更。 透過此設計，您可將計算目標視為無狀態的計算資源，其中的每一個與完成之後執行的作業之間沒有任何同質性。

## <a name="challenges"></a>挑戰
即使此設計可提供可攜性和重複性的優點，也會帶來一些獨特的挑戰。

### <a name="persisting-state-changes"></a>保存狀態變更
如果您的指令碼會修改計算內容上的狀態，則不會為您的下一個執行保存變更，也不會自動將它們傳播回用戶端電腦。 

更具體而言，如果您的指令碼建立子資料夾，或寫入檔案，該資料夾或檔案在執行之後將不會出現在專案目錄中。 檔案會儲存在計算目標環境中的暫存資料夾。 您可以將它們用於偵錯用途，但您不能永久依賴它們的存在。

### <a name="working-with-large-files-in-the-project-folder"></a>處理專案資料夾中的大型檔案

如果您的專案資料夾包含任何大型檔案，在每個執行的開始將資料夾複製到目標計算環境時，即會造成延遲。 即使執行會在本機發生，仍有應該避免的不必要磁碟流量。 基於這個理由，我們目前將最大專案大小限制在 25 MB。

## <a name="option-1-use-the-outputs-folder"></a>選項 1：使用 *outputs* 資料夾
如果下列所有條件都適用，最好採用此選項：
* 您的指令碼會產生檔案。
* 您預期檔案會隨著每個實驗變更。
* 您想要保留這些檔案的歷程記錄。 

常見使用案例如下：
* Training a model
* 建立資料集
* 隨著模型定型執行將圖形繪製為影像檔 

>[!Note]
> 執行之後，outputs 資料夾中的追蹤檔案大小上限是 512 MB。 這表示如果指令碼在 outputs 資料夾中產生大於 512 MB 的檔案，就不會在那裡收集該檔案。 

此外，您想要跨執行比較這些輸出，請選取輸出檔 (例如模型)，然後將它用於後續作業 (例如計分)。

您可以將檔案寫入名為 *outputs* 的資料夾 (相對於根目錄)。 該資料夾會受到實驗服務的特殊處理。 您的指令碼在執行期間於資料夾中建立的任何項目 (例如，模型檔、資料檔或繪製的影像檔 (合稱為_成品_))，均會在完成執行之後，複製到與您實驗帳戶相關聯的 Azure Blob 儲存體帳戶。 檔案會成為您執行歷程記錄記錄的一部分。

以下是將模型儲存在 *outputs* 資料夾的簡短程式碼範例：
```python
import os
import pickle

# m is a scikit-learn model. 
# we serialize it into a mode.plk file under the ./outputs folder.
with open(os.path.join('.', 'outputs', 'model.pkl'), 'wb') as f:    
    pickle.dump(m, f)
```
您可以下載任何成品，方法是瀏覽至 Azure Machine Learning Workbench 中特殊執行之執行詳細資料頁面的 [輸出檔案] 區段。 只要選取執行，然後選取 [下載] 按鈕。 此外，您可以在命令列介面 (CLI) 視窗中輸入 `az ml asset download` 命令。

如需更完整的範例，請參閱 _Classifying Iris_ (將蝴蝶花分類) 範例專案中的 `iris_sklearn.py` Python 指令碼。

## <a name="option-2-use-the-shared-folder"></a>選項 2：使用共用資料夾
如果您不需要針對每個執行保留這些檔案的歷程記錄，使用可跨獨立執行存取的共用資料夾，對下列情節而言是一個絕佳選項： 
- 您的指令碼需要從本機檔案 (例如 .csv 檔案)，或是文字檔或影像檔的目錄載入資料，以作為定型或測試資料。 
- 您的指令碼會處理未經處理的資料，並寫出中繼結果，例如凸顯文字或影像檔的定型資料，以便在後續定型執行中使用。 
- 您的指令碼會產生一個模型，而您後續的計分指令碼必須挑選該模型並用它來進行評估。 

務必注意，共用資料夾會本機存留於您選擇的計算目標上。 因此，此選項只有在您所有參考此共用資料夾的指令碼執行都在相同的計算目標上執行，而且計算目標不會在執行之間循環時才會運作。

利用共用資料夾功能，您可以讀取或寫入透過環境變數 `AZUREML_NATIVE_SHARE_DIRECTORY` 所識別的特定資料夾。 

### <a name="example"></a>範例
以下提供一些範例 Python 程式碼，以使用此共用資料夾來讀取和寫入文字檔：
```python
import os

# write to the shared folder
with open(os.environ['AZUREML_NATIVE_SHARE_DIRECTORY'] + 'test.txt', "w") as f1:
    f1.write(“Hello World”)

# read from the shared folder
with open(os.environ['AZUREML_NATIVE_SHARE_DIRECTORY'] + 'test.txt', "r") as f2:
    text = f2.read()
```

如需更完整的範例，請參閱 _Classifying Iris_ (將蝴蝶花分類) 範例專案中的 *iris_sklearn_shared_folder.py* 檔案。

在您可以使用此功能之前，必須先在代表 *aml_config* 資料夾中之目標執行內容的 *.compute* 檔案中設定一些簡單組態。 資料夾的實際路徑可以根據您選擇的計算目標和您設定的值而有所不同。

### <a name="configure-local-compute-context"></a>設定本機計算內容

若要在本機計算內容中啟用此功能，只要將下一行新增至您的 *.compute* 檔案 (其代表_本機_環境，通常名為 *local.compute*)。
```
# local.runconfig
...
nativeSharedDirectory: ~/.azureml/share
...
```

路徑 ~/.azureml/share 是預設的基底資料夾路徑。 您可以將它變更為指令碼執行可存取的任何本機絕對路徑。 實驗帳戶名稱、工作區名稱和專案名稱均會自動附加至基底目錄名稱，以組成共用目錄的完整路徑。 例如，如果您使用先前的預設值，即可將檔案寫入下列路徑 (以及從中擷取)：

```
# on Windows
C:\users\<username>\.azureml\share\<exp_acct_name>\<workspace_name>\<proj_name>\

# on macOS
/Users/<username>/.azureml/share/<exp_acct_name>/<workspace_name>/<proj_name>/
```

### <a name="configure-the-docker-compute-context-local-or-remote"></a>設定 Docker 計算內容 (本機或遠端)
若要在 Docker 計算內容上啟用此功能，您必須將下列兩行新增至您的本機或遠端 Docker *.compute* 檔案。

```
# docker.compute
...
sharedVolumes: true
nativeSharedDirectory: ~/.azureml/share
...
```
>[!IMPORTANT]
>使用 `AZUREML_NATIVE_SHARE_DIRECTORY` 環境變數來存取共用資料夾時，**sharedVolumes** 屬性必須設定為 *true*。 否則，執行就會失敗。

在 Docker 容器中執行的程式碼一律會以 */azureml-share/* 形式看見此共用資料夾。 Docker 容器所看見的資料夾路徑是不可設定的。 請勿在程式碼中使用此資料夾名稱。 相反地，一律使用環境變數名稱 `AZUREML_NATIVE_SHARE_DIRECTORY` 來參照此資料夾。 它會對應到 Docker 主機電腦或計算內容上的本機資料夾。 這個本機資料夾的基底目錄是 *.compute* 檔案中 `nativeSharedDirectory` 設定的可設定值。 主機電腦上共用資料夾的本機路徑 (如果您使用預設值) 如下所示：
```
# Windows
C:\users\<username>\.azureml\share\<exp_acct_name>\<workspace_name>\<proj_name>\

# macOS
/Users/<username>/.azureml/share/<exp_acct_name>/<workspace_name>/<proj_name>/

# Ubuntu Linux
/home/<username>/.azureml/share/<exp_acct_name>/<workspace_name>/<proj_name>/
```

>[!NOTE]
>本機磁碟上共用資料夾的路徑，無論在本機計算內容或本機 Docker 計算內容都是一樣的。 這表示，您甚至可以在本機執行和本機 Docker 執行之間共用檔案。

您可以直接在這些資料夾中放置輸入資料，並預期您在該電腦上的本機或 Docker 執行可以挑選它們。 您也可以從本機或 Docker 執行將檔案寫入此資料夾，並預期檔案會保存於該資料夾中，存留於執行生命週期中。

如需詳細資訊，請參閱 [Azure Machine Learning Workbench 執行組態檔](experimentation-service-configuration-reference.md)。

>[!NOTE]
>HDInsight 計算內容中不支援 `AZUREML_NATIVE_SHARE_DIRECTORY` 環境變數。 不過，您可以藉由明確地使用絕對 Azure Blob 儲存體路徑來往返讀取和寫入連接的 Blob 儲存體，輕鬆達到相同的結果。

## <a name="option-3-use-external-durable-storage"></a>選項 3：使用外部永久性儲存體

您當可以使用外部永久性存放區來保存執行期間的狀態。 在下列情節中此選項很有幫助：
- 您的輸入資料已經儲存於可從目標計算環境存取的永久性儲存體中。
- 檔案不需是執行歷程記錄的一部分。
- 檔案必須由跨各種運算環境的執行來共用。
- 檔案必須能夠存活計算內容本身。

此方法的其中一個範例是從您的 Python 或 PySpark 程式碼使用 Azure Blob 儲存體。 以下是一個簡短範例：

```python
from azure.storage.blob import BlockBlobService
from azure.storage.blob.models import PublicAccess
import glob
import os

ACCOUNT_NAME = "<your blob storage account name>"
ACCOUNT_KEY = "<account key>"
CONTAINER_NAME = "<container name>"

blob_service = BlockBlobService(account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY)

## Create a new container if necessary, or use an existing one
blob_service.create_container(CONTAINER_NAME, fail_on_exist=False, public_access=PublicAccess.Container)

# df is a pandas DataFrame
df.to_csv('mydata.csv', sep='\t', index=False)

# Export the mydata.csv file to Blob storage.
for name in glob.iglob('mydata.csv'):
    blob_service.create_blob_from_path(CONTAINER_NAME, 'single_file.csv', name)
```

以下也會提供將任何任意的 Azure Blob 儲存體連接至 HDI Spark 執行階段的簡短範例：
```python
def attach_storage_container(spark, account, key):
    config = spark._sc._jsc.hadoopConfiguration()
    setting = "fs.azure.account.key." + account + ".blob.core.windows.net"
    if not config.get(setting):
        config.set(setting, key)
 
attach_storage_container(spark, "<storage account name>", "<storage key>”)
```

## <a name="conclusion"></a>結論
由於 Azure Machine Learning 會藉由將整個專案資料夾複製到目標計算內容來執行指令碼，因此，請特別注意大型輸入、輸出和中繼檔案。 對於大型檔案交易，您可以使用特殊的輸出資料夾、可透過 `AZUREML_NATIVE_SHARE_DIRECTORY` 環境變數存取的共用資料夾，或外部永久性儲存體。 

## <a name="next-steps"></a>後續步驟
- 檢閱 [Azure Machine Learning Workbench 執行組態檔](experimentation-service-configuration-reference.md)文章。
- 請參閱 [Classifying Iris](tutorial-classifying-iris-part-1.md) (將蝴蝶花分類) 教學課程專案如何使用 outputs 資料夾來保存定型的模型。
