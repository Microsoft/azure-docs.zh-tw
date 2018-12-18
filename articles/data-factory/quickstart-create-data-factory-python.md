---
title: 使用 Python 建立 Azure 資料處理站 | Microsoft Docs
description: 建立 Azure 資料處理站，以將資料從 Azure Blob 儲存體中的某個位置複製到另一個位置。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: python
ms.topic: quickstart
ms.date: 01/22/2018
ms.author: shlo
ms.openlocfilehash: 5b8e1d894a0f2b7868433cb25a097a4cfcdc1796
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43095780"
---
# <a name="create-a-data-factory-and-pipeline-using-python"></a>使用 Python 建立資料處理站和管線
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [目前的版本](quickstart-create-data-factory-python.md)

Azure Data Factory 是雲端式資料整合服務，可讓您在雲端建立資料驅動工作流程，以便協調及自動進行資料移動和資料轉換。 使用 Azure Data Factory，您可以建立和排程資料驅動工作流程 (稱為管線)，這類工作流程可以從不同資料存放區內嵌資料，使用計算服務 (例如 Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics 和 Azure Machine Learning) 來處理/轉換資料，以及將輸出資料發佈至資料存放區 (例如 Azure SQL 資料倉儲)，以供商業智慧 (BI) 應用程式使用。

本快速入門說明如何使用 Python 來建立 Azure 資料處理站。 在此範例中的資料處理站會將資料從 Azure Blob 儲存體中的一個資料夾複製到其他資料夾。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件

* **Azure 儲存體帳戶**。 您需要使用 Blob 儲存體作為**來源**和**接收**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)一文，按照步驟來建立帳戶。
* 請遵循[此指示](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)**在 Azure Active Directory 中建立應用程式**。 記下這些值，您在稍後的步驟中會用到：**應用程式識別碼**、**驗證金鑰**和**租用戶識別碼**。 遵循相同文件中的指示，將應用程式指派給「**參與者**」角色。

### <a name="create-and-upload-an-input-file"></a>建立及上傳輸入檔案

1. 啟動 [記事本]。 複製下列文字，並在磁碟上儲存為 **input.txt** 檔案。

    ```
    John|Doe
    Jane|Doe
    ```
2.  使用 [Azure 儲存體總管](http://storageexplorer.com/)之類的工具建立 **adfv2tutorial** 容器，然後在容器中建立 **input**。 然後，將 **input.txt** 檔案上傳至 **input** 資料夾。

## <a name="install-the-python-package"></a>安裝 Python 封裝
1. 以系統管理員權限開啟終端機或命令提示字元。 
2. 首先，針對 Azure 管理資源安裝 Python 套件：

    ```
    pip install azure-mgmt-resource
    ```
3. 若要安裝適用於 Data Factory 的 Python 封裝，請執行下列命令：

    ```
    pip install azure-mgmt-datafactory
    ```

    [Data Factory 適用的 Python SDK](https://github.com/Azure/azure-sdk-for-python) 支援 Python 2.7、3.3、3.4、3.5 和 3.6。

## <a name="create-a-data-factory-client"></a>建立資料處理站用戶端

1. 建立名為 **datafactory.py** 的檔案。 新增下列陳述式以將參考新增至命名空間。

    ```python
    from azure.common.credentials import ServicePrincipalCredentials
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.datafactory import DataFactoryManagementClient
    from azure.mgmt.datafactory.models import *
    from datetime import datetime, timedelta
    import time
    ```
2. 新增會列印資訊的下列函式。

    ```python
    def print_item(group):
        """Print an Azure object instance."""
        print("\tName: {}".format(group.name))
        print("\tId: {}".format(group.id))
        if hasattr(group, 'location'):
            print("\tLocation: {}".format(group.location))
        if hasattr(group, 'tags'):
            print("\tTags: {}".format(group.tags))
        if hasattr(group, 'properties'):
            print_properties(group.properties)

    def print_properties(props):
        """Print a ResourceGroup properties instance."""
        if props and hasattr(props, 'provisioning_state') and props.provisioning_state:
            print("\tProperties:")
            print("\t\tProvisioning State: {}".format(props.provisioning_state))
        print("\n\n")

    def print_activity_run_details(activity_run):
        """Print activity run details."""
        print("\n\tActivity run details\n")
        print("\tActivity run status: {}".format(activity_run.status))    
        if activity_run.status == 'Succeeded':
            print("\tNumber of bytes read: {}".format(activity_run.output['dataRead']))       
            print("\tNumber of bytes written: {}".format(activity_run.output['dataWritten']))           
            print("\tCopy duration: {}".format(activity_run.output['copyDuration']))           
        else:
            print("\tErrors: {}".format(activity_run.error['message']))

    ```
3. 將下列程式碼新增至 **Main** 方法，以建立 DataFactoryManagementClient 類別的執行個體。 您會使用此物件來建立資料處理站、連結服務、資料集和管線。 您也可以使用此物件來監視管線執行的詳細資料。 將 **subscription_id** 變數設定為 Azure 訂用帳戶的識別碼。 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

    ```python   
    def main():

        # Azure subscription ID
        subscription_id = '<Specify your Azure Subscription ID>'

        # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
        rg_name = 'ADFTutorialResourceGroup'

        # The data factory name. It must be globally unique.
        df_name = '<Specify a name for the data factory. It must be globally unique>'

        # Specify your Active Directory client ID, client secret, and tenant ID
        credentials = ServicePrincipalCredentials(client_id='<Active Directory application/client ID>', secret='<client secret>', tenant='<Active Directory tenant ID>')
        resource_client = ResourceManagementClient(credentials, subscription_id)
        adf_client = DataFactoryManagementClient(credentials, subscription_id)

        rg_params = {'location':'eastus'}
        df_params = {'location':'eastus'}    
    ```

## <a name="create-a-data-factory"></a>建立 Data Factory

將下列程式碼新增至 **Main** 方法，以建立**資料處理站**。 如果您的資源群組已經存在，請將第一個 `create_or_update` 陳述式變成註解。

```python
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    #Create a data factory
    df_resource = Factory(location='eastus')
    df = adf_client.factories.create_or_update(rg_name, df_name, df_resource)
    print_item(df)
    while df.provisioning_state != 'Succeeded':
        df = adf_client.factories.get(rg_name, df_name)
        time.sleep(1)
```

## <a name="create-a-linked-service"></a>建立連結的服務

將下列程式碼新增至 **Main** 方法，以建立 **Azure 儲存體連結服務**。

您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在此快速入門中，您只需要建立一個 Azure 儲存體連結服務作為複製來源與接收存放區，在此範例中名為 "AzureStorageLinkedService"。 使用 Azure 儲存體帳戶的名稱和金鑰來取代 `<storageaccountname>` 和 `<storageaccountkey>`。

```python
    # Create an Azure Storage linked service
    ls_name = 'storageLinkedService'

    # IMPORTANT: specify the name and key of your Azure Storage account.
    storage_string = SecureString('DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<storageaccountkey>')

    ls_azure_storage = AzureStorageLinkedService(connection_string=storage_string)
    ls = adf_client.linked_services.create_or_update(rg_name, df_name, ls_name, ls_azure_storage)
    print_item(ls)
```
## <a name="create-datasets"></a>建立資料集
在本節中，您會建立兩個資料集：一個作為來源，另一個作為接收。

### <a name="create-a-dataset-for-source-azure-blob"></a>建立來源 Azure Blob 的資料集
將下列程式碼新增至 Main 方法，以建立 Azure Blob 資料集。 如需 Azure Blob 資料集屬性的相關資訊，請參閱 [Azure Blob 連接器](connector-azure-blob-storage.md#dataset-properties)文章。

您可以定義資料集來代表 Azure Blob 中的來源資料。 此 Blob 資料集會參考您在前一個步驟中建立的 Azure 儲存體連結服務。

```python
    # Create an Azure blob dataset (input)
    ds_name = 'ds_in'
    ds_ls = LinkedServiceReference(ls_name)
    blob_path= 'adfv2tutorial/input'
    blob_filename = 'input.txt'
    ds_azure_blob= AzureBlobDataset(ds_ls, folder_path=blob_path, file_name = blob_filename)
    ds = adf_client.datasets.create_or_update(rg_name, df_name, ds_name, ds_azure_blob)
    print_item(ds)
```

### <a name="create-a-dataset-for-sink-azure-blob"></a>建立接收 Azure Blob 的資料集
將下列程式碼新增至 Main 方法，以建立 Azure Blob 資料集。 如需 Azure Blob 資料集屬性的相關資訊，請參閱 [Azure Blob 連接器](connector-azure-blob-storage.md#dataset-properties)文章。

您可以定義資料集來代表 Azure Blob 中的來源資料。 此 Blob 資料集會參考您在前一個步驟中建立的 Azure 儲存體連結服務。

```python
    # Create an Azure blob dataset (output)
    dsOut_name = 'ds_out'
    output_blobpath = 'adfv2tutorial/output'
    dsOut_azure_blob = AzureBlobDataset(ds_ls, folder_path=output_blobpath)
    dsOut = adf_client.datasets.create_or_update(rg_name, df_name, dsOut_name, dsOut_azure_blob)
    print_item(dsOut)
```

## <a name="create-a-pipeline"></a>建立管線

將下列程式碼新增至 **Main** 方法，以建立**具有複製活動的管線**。

```python
    # Create a copy activity
    act_name =  'copyBlobtoBlob'
    blob_source = BlobSource()
    blob_sink = BlobSink()
    dsin_ref = DatasetReference(ds_name)
    dsOut_ref = DatasetReference(dsOut_name)
    copy_activity = CopyActivity(act_name,inputs=[dsin_ref], outputs=[dsOut_ref], source=blob_source, sink=blob_sink)

    #Create a pipeline with the copy activity
    p_name =  'copyPipeline'
    params_for_pipeline = {}
    p_obj = PipelineResource(activities=[copy_activity], parameters=params_for_pipeline)
    p = adf_client.pipelines.create_or_update(rg_name, df_name, p_name, p_obj)
    print_item(p)
```


## <a name="create-a-pipeline-run"></a>建立管線執行

將下列程式碼新增至 **Main** 方法，以**觸發管線執行**。

```python
    #Create a pipeline run.
    run_response = adf_client.pipelines.create_run(rg_name, df_name, p_name,
        {
        }
    )
```

## <a name="monitor-a-pipeline-run"></a>監視管線執行
若要監視管道執行，將下列程式碼新增至 **Main** 方法：

```python
    #Monitor the pipeline run
    time.sleep(30)
    pipeline_run = adf_client.pipeline_runs.get(rg_name, df_name, run_response.run_id)
    print("\n\tPipeline run status: {}".format(pipeline_run.status))
    activity_runs_paged = list(adf_client.activity_runs.list_by_pipeline_run(rg_name, df_name, pipeline_run.run_id, datetime.now() - timedelta(1),  datetime.now() + timedelta(1)))
    print_activity_run_details(activity_runs_paged[0])
```

現在，新增下列陳述式以在執行程式時叫用 **main** 方法：

```python
# Start the main method
main()
```

## <a name="full-script"></a>完整指令碼
以下是完整的 Python 程式碼：

```python
from azure.common.credentials import ServicePrincipalCredentials
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.datafactory import DataFactoryManagementClient
from azure.mgmt.datafactory.models import *
from datetime import datetime, timedelta
import time

def print_item(group):
    """Print an Azure object instance."""
    print("\tName: {}".format(group.name))
    print("\tId: {}".format(group.id))
    if hasattr(group, 'location'):
        print("\tLocation: {}".format(group.location))
    if hasattr(group, 'tags'):
        print("\tTags: {}".format(group.tags))
    if hasattr(group, 'properties'):
        print_properties(group.properties)
    print("\n")        

def print_properties(props):
    """Print a ResourceGroup properties instance."""
    if props and hasattr(props, 'provisioning_state') and props.provisioning_state:
        print("\tProperties:")
        print("\t\tProvisioning State: {}".format(props.provisioning_state))
    print("\n")

def print_activity_run_details(activity_run):
    """Print activity run details."""
    print("\n\tActivity run details\n")
    print("\tActivity run status: {}".format(activity_run.status))    
    if activity_run.status == 'Succeeded':
        print("\tNumber of bytes read: {}".format(activity_run.output['dataRead']))       
        print("\tNumber of bytes written: {}".format(activity_run.output['dataWritten']))           
        print("\tCopy duration: {}".format(activity_run.output['copyDuration']))           
    else:
        print("\tErrors: {}".format(activity_run.error['message']))

def main():

    # Azure subscription ID
    subscription_id = '<your Azure subscription ID>'

    # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
    rg_name = '<Azure resource group name>'

    # The data factory name. It must be globally unique.
    df_name = '<Your data factory name>'        

    # Specify your Active Directory client ID, client secret, and tenant ID
    credentials = ServicePrincipalCredentials(client_id='<Active Directory client ID>', secret='<client secret>', tenant='<tenant ID>')
    resource_client = ResourceManagementClient(credentials, subscription_id)
    adf_client = DataFactoryManagementClient(credentials, subscription_id)

    rg_params = {'location':'eastus'}
    df_params = {'location':'eastus'}

    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    # Create a data factory
    df_resource = Factory(location='eastus')
    df = adf_client.factories.create_or_update(rg_name, df_name, df_resource)
    print_item(df)
    while df.provisioning_state != 'Succeeded':
        df = adf_client.factories.get(rg_name, df_name)
        time.sleep(1)

    # Create an Azure Storage linked service
    ls_name = 'storageLinkedService'

    # Specify the name and key of your Azure Storage account
    storage_string = SecureString('DefaultEndpointsProtocol=https;AccountName=<storage account name>;AccountKey=<storage account key>')

    ls_azure_storage = AzureStorageLinkedService(connection_string=storage_string)
    ls = adf_client.linked_services.create_or_update(rg_name, df_name, ls_name, ls_azure_storage)
    print_item(ls)

    # Create an Azure blob dataset (input)
    ds_name = 'ds_in'
    ds_ls = LinkedServiceReference(ls_name)
    blob_path= 'adfv2tutorial/input'
    blob_filename = 'input.txt'
    ds_azure_blob= AzureBlobDataset(ds_ls, folder_path=blob_path, file_name = blob_filename)
    ds = adf_client.datasets.create_or_update(rg_name, df_name, ds_name, ds_azure_blob)
    print_item(ds)

    # Create an Azure blob dataset (output)
    dsOut_name = 'ds_out'
    output_blobpath = 'adfv2tutorial/output'
    dsOut_azure_blob = AzureBlobDataset(ds_ls, folder_path=output_blobpath)
    dsOut = adf_client.datasets.create_or_update(rg_name, df_name, dsOut_name, dsOut_azure_blob)
    print_item(dsOut)

    # Create a copy activity
    act_name =  'copyBlobtoBlob'
    blob_source = BlobSource()
    blob_sink = BlobSink()
    dsin_ref = DatasetReference(ds_name)
    dsOut_ref = DatasetReference(dsOut_name)
    copy_activity = CopyActivity(act_name,inputs=[dsin_ref], outputs=[dsOut_ref], source=blob_source, sink=blob_sink)

    # Create a pipeline with the copy activity
    p_name =  'copyPipeline'
    params_for_pipeline = {}
    p_obj = PipelineResource(activities=[copy_activity], parameters=params_for_pipeline)
    p = adf_client.pipelines.create_or_update(rg_name, df_name, p_name, p_obj)
    print_item(p)

    # Create a pipeline run
    run_response = adf_client.pipelines.create_run(rg_name, df_name, p_name,
        {
        }
    )

    # Monitor the pipeilne run
    time.sleep(30)
    pipeline_run = adf_client.pipeline_runs.get(rg_name, df_name, run_response.run_id)
    print("\n\tPipeline run status: {}".format(pipeline_run.status))
    activity_runs_paged = list(adf_client.activity_runs.list_by_pipeline_run(rg_name, df_name, pipeline_run.run_id, datetime.now() - timedelta(1),  datetime.now() + timedelta(1)))
    print_activity_run_details(activity_runs_paged[0])

# Start the main method
main()
```

## <a name="run-the-code"></a>執行程式碼
建置並啟動應用程式，然後確認管線執行。

主控台會印出建立資料處理站、連結服務、資料集、管線和管線執行的進度。 請等待出現複製活動執行詳細資料及讀取/寫入的資料大小。 然後，使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)之類的工具，檢查 Blob 已從 "inputBlobPath" 複製到 "outputBlobPath" (您在變數中指定)。

以下是範例輸出：

```json
Name: <data factory name>
Id: /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DataFactory/factories/<data factory name>
Location: eastus
Tags: {}

Name: storageLinkedService
Id: /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DataFactory/factories/<data factory name>/linkedservices/storageLinkedService

Name: ds_in
Id: /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DataFactory/factories/<data factory name>/datasets/ds_in

Name: ds_out
Id: /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DataFactory/factories/<data factory name>/datasets/ds_out

Name: copyPipeline
Id: /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DataFactory/factories/<data factory name>/pipelines/copyPipeline

Pipeline run status: Succeeded
Datetime with no tzinfo will be considered UTC.
Datetime with no tzinfo will be considered UTC.

Activity run details

Activity run status: Succeeded
Number of bytes read: 18
Number of bytes written: 18
Copy duration: 4
```


## <a name="clean-up-resources"></a>清除資源
若要刪除資料處理站，請將下列程式碼新增至程式：

```python
adf_client.factories.delete(rg_name,df_name)
```

## <a name="next-steps"></a>後續步驟
在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 瀏覽[教學課程](tutorial-copy-data-dot-net.md)以了解使用 Data Factory 的更多案例。
