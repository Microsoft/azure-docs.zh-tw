---
title: 教學課程 - 使用適用於 Node.js 的 Azure Batch 用戶端程式庫 | Microsoft Docs
description: 了解 Azure Batch 的基本概念和使用 Node.js 建置簡單的解決方案。
services: batch
author: shwetams
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: nodejs
ms.topic: hero-article
ms.workload: big-compute
ms.date: 05/22/2017
ms.author: shwetams
ms.openlocfilehash: 807fd49a54c82b0930134beb8413e14c1c28b278
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39115556"
---
# <a name="get-started-with-batch-sdk-for-nodejs"></a>開始使用適用於 Node.js 的 Batch SDK

了解如何使用 [Azure Batch Node.js SDK](/javascript/api/overview/azure/batch) 在 Node.js 中建置 Batch 用戶端的基本概念。 我們會逐步了解批次應用程式的案例，然後使用 Node.js 用戶端加以設定。  

## <a name="prerequisites"></a>必要條件
本文假設您已具備 Node.js 的使用知識並熟悉 Linux。 同時假設您的 Azure 帳戶設有存取權限，可建立 Batch 和儲存體服務。

建議您先閱讀 [Azure Batch 技術概觀](batch-technical-overview.md)，再進行本文概述的步驟。

## <a name="the-tutorial-scenario"></a>教學課程案例
讓我們一起了解 Batch 工作流程案例。 我們有以 Python 撰寫的簡單指令碼，可從 Azure Blob 儲存體容器下載所有 csv 檔案並將它們轉換成 JSON。 若要平行處理多個儲存體帳戶容器，我們可以將此指令碼部署為 Azure Batch 作業。

## <a name="azure-batch-architecture"></a>Azure Batch 架構
下圖描述如何使用 Azure Batch 和 Node.js 用戶端調整 Python 指令碼。

![Azure Batch 案例](./media/batch-nodejs-get-started/BatchScenario.png)

Node.js 用戶端會使用準備工作部署批次作業 (稍後詳細說明) 以及一組工作 (視儲存體帳戶中容器數目)。 您可以從 GitHub 存放庫下載指令碼。

* [Node.js 用戶端](https://github.com/Azure/azure-batch-samples/blob/master/Node.js/GettingStarted/nodejs_batch_client_sample.js)
* [準備工作 Shell 指令碼](https://github.com/Azure/azure-batch-samples/blob/master/Node.js/GettingStarted/startup_prereq.sh)
* [Python csv 至 JSON 處理器](https://github.com/Azure/azure-batch-samples/blob/master/Node.js/GettingStarted/processcsv.py)

> [!TIP]
> 指定連結中的 Node.js 用戶端不包含要部署為 Azure 函式應用程式的特定程式碼。 您可以參考下列連結，以取得建立函式應用程式的指示。
> - [建立函式應用程式](../azure-functions/functions-create-first-azure-function.md)
> - [建立計時器觸發程序函式](../azure-functions/functions-bindings-timer.md)
>
>

## <a name="build-the-application"></a>建置應用程式

現在，我們要依照程序逐步建置 Node.js 用戶端︰

### <a name="step-1-install-azure-batch-sdk"></a>步驟 1：安裝 Azure Batch SDK

您可以使用 npm 安裝命令來安裝適用於 Node.js 的 Azure Batch SDK。

`npm install azure-batch`

此命令會安裝最新版的 azure-batch 節點 SDK。

>[!Tip]
> 在 Azure Function 應用程式中，您可以移至 Azure Function 的 [設定] 索引標籤中的 [Kudu 主控台] 以執行 npm 安裝命令。 在此情況下，安裝適用於 Node.js 的 Azure Batch SDK。
>
>

### <a name="step-2-create-an-azure-batch-account"></a>步驟 2：建立 Azure Batch 帳戶

您可以從 [Azure 入口網站](batch-account-create-portal.md)或從命令列 ([Powershell](batch-powershell-cmdlets-get-started.md) /[Azure cli](/cli/azure)) 加以建立。

以下是透過 Azure CLI 命令建立 Batch 帳戶的命令。

建立資源群組，如果您想要建立 Batch 帳戶的地方已經有一個，請略過此步驟︰

`az group create -n "<resource-group-name>" -l "<location>"`

接下來，建立 Azure Batch 帳戶。

`az batch account create -l "<location>"  -g "<resource-group-name>" -n "<batch-account-name>"`

每個 Batch 帳戶有其對應的存取金鑰。 需要有這些金鑰，才能在 Azure Batch 帳戶中建立進一步的資源。 在生產環境中，最好使用 Azure Key Vault 來儲存這些金鑰。 然後您可以建立應用程式的服務主體。 使用此服務主體，應用程式可以建立 OAuth 權杖以存取 Key Vault 中的金鑰。

`az batch account keys list -g "<resource-group-name>" -n "<batch-account-name>"`

複製並儲存要在後續步驟中使用的金鑰。

### <a name="step-3-create-an-azure-batch-service-client"></a>步驟 3︰建立 Azure Batch 服務用戶端
下列程式碼片段會先匯入 azure-batch Node.js 模組，然後建立 Batch 服務用戶端。 您必須先使用從上一個步驟複製的 Batch 帳戶金鑰來建立 SharedKeyCredentials 物件。

```nodejs
// Initializing Azure Batch variables

var batch = require('azure-batch');

var accountName = '<azure-batch-account-name>';

var accountKey = '<account-key-downloaded>';

var accountUrl = '<account-url>'

// Create Batch credentials object using account name and account key

var credentials = new batch.SharedKeyCredentials(accountName,accountKey);

// Create Batch service client

var batch_client = new batch.ServiceClient(credentials,accountUrl);

```

在 Azure 入口網站的 [概觀] 索引標籤中可以找到 Azure Batch URI。 其格式如下︰

`https://accountname.location.batch.azure.com`

請參考螢幕擷取畫面：

![Azure Batch URI](./media/batch-nodejs-get-started/azurebatchuri.png)



### <a name="step-4-create-an-azure-batch-pool"></a>步驟 4︰建立 Azure Batch 集區
Azure Batch 集區是由多個 VM (也稱為 Batch 節點) 所組成。 Azure Batch 服務會在這些節點上部署工作並加以管理。 您可以為您的集區定義下列組態參數。

* 虛擬機器映像的類型
* 虛擬機器節點的大小
* 虛擬機器節點的數目

> [!Tip]
> 虛擬機器節點的大小和數目主要取決於您想要平行執行的工作數目以及工作本身。 我們建議進行測試，以判斷理想的數目和大小。
>
>

下列程式碼片段會建立組態參數物件。

```nodejs
// Creating Image reference configuration for Ubuntu Linux VM
var imgRef = {publisher:"Canonical",offer:"UbuntuServer",sku:"14.04.2-LTS",version:"latest"}

// Creating the VM configuration object with the SKUID
var vmconfig = {imageReference:imgRef,nodeAgentSKUId:"batch.node.ubuntu 14.04"}

// Setting the VM size to Standard F4
var vmSize = "STANDARD_F4"

//Setting number of VMs in the pool to 4
var numVMs = 4
```

> [!Tip]
> 如需 Azure Batch 及其 SKU 識別碼可用的 Linux VM 映像清單，請參閱[虛擬機器映像清單](batch-linux-nodes.md#list-of-virtual-machine-images)。
>
>

一旦定義集區組態，您就可以建立 Azure Batch 集區。 Batch 集區命令會建立 Azure 虛擬機器節點，並備妥它們以便用於接收要執行的工作。 每個集區都應該有後續步驟中參考的唯一識別碼。

下列程式碼片段會建立 Azure Batch 集區。

```nodejs
// Create a unique Azure Batch pool ID
var poolid = "pool" + customerDetails.customerid;
var poolConfig = {id:poolid, displayName:poolid,vmSize:vmSize,virtualMachineConfiguration:vmconfig,targetDedicatedComputeNodes:numVms,enableAutoScale:false };
// Creating the Pool for the specific customer
var pool = batch_client.pool.add(poolConfig,function(error,result){
    if(error!=null){console.log(error.response)};
});
```

您可以檢查已建立的集區狀態，先確定狀態處於「作用中」，再繼續將作業提交至該集區。

```nodejs
var cloudPool = batch_client.pool.get(poolid,function(error,result,request,response){
        if(error == null)
        {

            if(result.state == "active")
            {
                console.log("Pool is active");
            }
        }
        else
        {
            if(error.statusCode==404)
            {
                console.log("Pool not found yet returned 404...");    

            }
            else
            {
                console.log("Error occurred while retrieving pool data");
            }
        }
        });
```

以下是 pool.get 函式所傳回的範例結果物件。

```
{ id: 'processcsv_201721152',
  displayName: 'processcsv_201721152',
  url: 'https://<batch-account-name>.centralus.batch.azure.com/pools/processcsv_201721152',
  eTag: '<eTag>',
  lastModified: 2017-03-27T10:28:02.398Z,
  creationTime: 2017-03-27T10:28:02.398Z,
  state: 'active',
  stateTransitionTime: 2017-03-27T10:28:02.398Z,
  allocationState: 'resizing',
  allocationStateTransitionTime: 2017-03-27T10:28:02.398Z,
  vmSize: 'standard_a1',
  virtualMachineConfiguration:
   { imageReference:
      { publisher: 'Canonical',
        offer: 'UbuntuServer',
        sku: '14.04.2-LTS',
        version: 'latest' },
     nodeAgentSKUId: 'batch.node.ubuntu 14.04' },
  resizeTimeout:
   { [Number: 900000]
     _milliseconds: 900000,
     _days: 0,
     _months: 0,
     _data:
      { milliseconds: 0,
        seconds: 0,
        minutes: 15,
        hours: 0,
        days: 0,
        months: 0,
        years: 0 },
     _locale:
      Locale {
        _calendar: [Object],
        _longDateFormat: [Object],
        _invalidDate: 'Invalid date',
        ordinal: [Function: ordinal],
        _ordinalParse: /\d{1,2}(th|st|nd|rd)/,
        _relativeTime: [Object],
        _months: [Object],
        _monthsShort: [Object],
        _week: [Object],
        _weekdays: [Object],
        _weekdaysMin: [Object],
        _weekdaysShort: [Object],
        _meridiemParse: /[ap]\.?m?\.?/i,
        _abbr: 'en',
        _config: [Object],
        _ordinalParseLenient: /\d{1,2}(th|st|nd|rd)|\d{1,2}/ } },
  currentDedicated: 0,
  targetDedicated: 4,
  enableAutoScale: false,
  enableInterNodeCommunication: false,
  maxTasksPerNode: 1,
  taskSchedulingPolicy: { nodeFillType: 'Spread' } }
```


### <a name="step-4-submit-an-azure-batch-job"></a>步驟 4︰提交 Azure Batch 作業
Azure Batch 作業是相似工作的邏輯群組。 在我們的案例中，這是「將 csv 處理成 JSON」。 這裡的每個工作都可能會處理每個 Azure 儲存體容器中存在的 csv 檔案。

這些工作會以平行方式執行並且部署於多個節點 (由 Azure Batch 服務協調)。

> [!Tip]
> 您可以使用 [maxTasksPerNode](http://azure.github.io/azure-sdk-for-node/azure-batch/latest/Pool.html#add) 屬性來指定可以在單一節點上同時執行的工作數目上限。
>
>

#### <a name="preparation-task"></a>準備工作

建立的 VM 節點是空白的 Ubuntu 節點。 您通常需要安裝一組程式作為必要條件。
一般來說，對於 Linux 節點，您可以有在實際工作執行前安裝先決條件的殼層指令碼。 不過，它可能是任何可程式化的可執行檔。
此範例中的[殼層指令碼](https://github.com/shwetams/azure-batchclient-sample-nodejs/blob/master/startup_prereq.sh)會安裝 Python-pip 和適用於 Python 的 Azure 儲存體 SDK。

您可以在 Azure 儲存體帳戶上傳指令碼，並產生 SAS URI 來存取指令碼。 使用 Azure 儲存體 Node.js SDK 也可以自動執行此程序。

> [!Tip]
> 作業的準備工作只會在需要執行特定工作的 VM 節點上執行。 如果您想要在所有節點上安裝必要條件 (不管其上執行的工作為何)，可以在新增集區時使用 [startTask](http://azure.github.io/azure-sdk-for-node/azure-batch/latest/Pool.html#add) 屬性。 您可以使用下列準備工作定義以供參考。
>
>

準備工作是在 Azure Batch 作業提交期間指定。 以下是準備工作組態參數︰

* **識別碼**︰準備工作的唯一識別碼
* **命令列**︰要執行工作可執行檔的命令列
* **resourceFiles**︰提供執行此工作所需下載之檔案詳細資料的物件陣列。  其選項如下：
    - blobSource︰檔案的 SAS URI
    - filePath︰要下載並儲存檔案的本機路徑
    - fileMode︰僅適用於 Linux 節點，fileMode 為八進位格式 (預設值是 0770)
* **waitForSuccess**︰如果設為 true，此工作不會在準備工作失敗時執行
* **runElevated**︰如果需要提高的權限才能執行工作，請將它設定為 true。

下列程式碼片段顯示準備工作指令碼組態範例︰

```nodejs
var job_prep_task_config = {id:"installprereq",commandLine:"sudo sh startup_prereq.sh > startup.log",resourceFiles:[{'blobSource':'Blob SAS URI','filePath':'startup_prereq.sh'}],waitForSuccess:true,runElevated:true}
```

如果不需安裝任何必要條件，您的工作即可執行，您可以略過準備工作。 下列程式碼會建立一項作業，其顯示名稱為「處理 csv 檔案」。

 ```nodejs
 // Setting up Batch pool configuration
 var pool_config = {poolId:poolid}
 // Setting up Job configuration along with preparation task
 var jobId = "processcsvjob"
 var job_config = {id:jobId,displayName:"process csv files",jobPreparationTask:job_prep_task_config,poolInfo:pool_config}
 // Adding Azure batch job to the pool
 var job = batch_client.job.add(job_config,function(error,result){
     if(error != null)
     {
         console.log("Error submitting job : " + error.response);
     }});
```


### <a name="step-5-submit-azure-batch-tasks-for-a-job"></a>步驟 5︰提交作業的 Azure 批次工作

現在已建立處理 csv 作業，讓我們為該作業建立工作。 假設我們有四個容器，我們必須建立四項工作，每個容器一項工作。

如果我們查看 [Python 指令碼](https://github.com/shwetams/azure-batchclient-sample-nodejs/blob/master/processcsv.py)，它接受兩個參數︰

* 容器名稱︰要從中下載檔案的儲存體容器
* 模式︰檔案名稱模式的選擇性參數

假設我們有四個容器 "con1"、"con2"、"con3"、"con4"，下列程式碼顯示如何將工作提交至我們稍早建立的 Azure 批次作業「處理 csv」。

```nodejs
// storing container names in an array
var container_list = ["con1","con2","con3","con4"]
    container_list.forEach(function(val,index){           

           var container_name = val;
           var taskID = container_name + "_process";
           var task_config = {id:taskID,displayName:'process csv in ' + container_name,commandLine:'python processcsv.py --container ' + container_name,resourceFiles:[{'blobSource':'<blob SAS URI>','filePath':'processcsv.py'}]}
           var task = batch_client.task.add(poolid,task_config,function(error,result){
                if(error != null)
                {
                    console.log(error.response);     
                }
                else
                {
                    console.log("Task for container : " + container_name + "submitted successfully");
                }



           });

    });
```

此程式碼會將多個工作新增至集區。 而每項工作會在所建立 VM 集區中的節點上執行。 如果工作數目超過集區或 maxTasksPerNode 屬性中的 VM 數目，工作會等到節點可用為止。 Azure Batch 會自動處理此協調流程。

入口網站有工作與作業狀態的詳細檢視。 您也可以使用此清單並取得 Azure 節點 SDK 中的函式。 文件[連結](http://azure.github.io/azure-sdk-for-node/azure-batch/latest/Job.html)中會提供詳細資料。

## <a name="next-steps"></a>後續步驟

- 如果您不熟悉這項服務，我們建議檢閱 [Azure Batch 功能概觀](batch-api-basics.md) 一文。
- 請參閱 [Batch Node.js 參考](/javascript/api/overview/azure/batch)以探索 Batch API。

