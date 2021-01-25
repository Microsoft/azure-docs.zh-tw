---
title: 快速入門：使用 Python 建立 Azure Data Factory
description: 使用資料處理站，以將資料從 Azure Blob 儲存體中的某個位置複製到另一個位置。
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.devlang: python
ms.topic: quickstart
ms.date: 01/15/2021
ms.custom: seo-python-october2019, devx-track-python
ms.openlocfilehash: 883b7df1d4dfd1b405f2952aa675db5948ab3ca3
ms.sourcegitcommit: c7153bb48ce003a158e83a1174e1ee7e4b1a5461
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98234285"
---
# <a name="quickstart-create-a-data-factory-and-pipeline-using-python"></a>快速入門：使用 Python 建立資料處理站和管線

> [!div class="op_single_selector" title1="選取您目前使用的 Data Factory 服務版本："]
> * [第 1 版](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [目前的版本](quickstart-create-data-factory-python.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

在本快速入門中，您會使用 Python 建立資料處理站。 在此範例中的資料處理站會將資料從 Azure Blob 儲存體中的一個資料夾複製到其他資料夾。

Azure Data Factory 是雲端式資料整合服務，可讓您建立資料驅動的工作流程，以便協調及自動進行資料移動和資料轉換。 您可以使用 Azure Data Factory 建立並排程資料驅動的工作流程 (稱為管線)。

管線可從不同資料存放區擷取資料。 管線會使用計算服務 (例如，Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics 和 Azure Machine Learning) 來處理或轉換資料。 管線可將輸出資料發佈至資料存放區 (例如 Azure Synapse Analytics)，供商業智慧 (BI) 應用程式使用。

## <a name="prerequisites"></a>Prerequisites

* 具有有效訂用帳戶的 Azure 帳戶。 [建立免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

* [Python 3.4+](https://www.python.org/downloads/)。

* [Azure 儲存體帳戶](../storage/common/storage-account-create.md)。

* [Azure 儲存體總管](https://storageexplorer.com/) (選擇性)。

* [Azure Active Directory 中的應用程式](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)。 記下這些值，以便在後續步驟中使用到：**應用程式識別碼**、**驗證金鑰** 和 **租用戶識別碼**。 依照同一篇文章中的指示，將應用程式指派給「參與者」  角色。

## <a name="create-and-upload-an-input-file"></a>建立及上傳輸入檔案

1. 啟動 [記事本]。 複製下列文字，並在磁碟上儲存為 **input.txt** 檔案。

    ```text
    John|Doe
    Jane|Doe
    ```
2.  使用 [Azure 儲存體總管](https://storageexplorer.com/)之類的工具建立 **adfv2tutorial** 容器，然後在容器中建立 **input**。 然後，將 **input.txt** 檔案上傳至 **input** 資料夾。

## <a name="install-the-python-package"></a>安裝 Python 封裝

1. 以系統管理員權限開啟終端機或命令提示字元。 
2. 首先，針對 Azure 管理資源安裝 Python 套件：

    ```python
    pip install azure-mgmt-resource
    ```
3. 若要安裝適用於 Data Factory 的 Python 封裝，請執行下列命令：

    ```python
    pip install azure-mgmt-datafactory
    ```

    [Data Factory 適用的 Python SDK](https://github.com/Azure/azure-sdk-for-python) 支援 Python 2.7、3.3、3.4、3.5、3.6 和 3.7。

4. 若要安裝適用於 Azure 身分識別驗證的 Python 套件，請執行下列命令：

    ```python
    pip install azure-identity
    ```
    > [!NOTE] 
    > "azure-identity" 套件可能會在某些常見的相依性上與 "azure-cli" 發生衝突。 如果您遇到任何驗證問題，請移除 "azure-cli" 及其相依性，或使用未安裝 "azure-cli" 套件的全新機器來使其正常執行。
    
## <a name="create-a-data-factory-client"></a>建立資料處理站用戶端

1. 建立名為 **datafactory.py** 的檔案。 新增下列陳述式以將參考新增至命名空間。

    ```python
    from azure.identity import ClientSecretCredential 
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
3. 將下列程式碼新增至 **Main** 方法，以建立 DataFactoryManagementClient 類別的執行個體。 您會使用此物件來建立資料處理站、連結服務、資料集和管線。 您也可以使用此物件來監視管線執行的詳細資料。 將 **subscription_id** 變數設定為 Azure 訂用帳戶的識別碼。 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析]  以找出 [Data Factory]  ：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

    ```python
    def main():

        # Azure subscription ID
        subscription_id = '<subscription ID>'

        # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
        rg_name = '<resource group>'

        # The data factory name. It must be globally unique.
        df_name = '<factory name>'

        # Specify your Active Directory client ID, client secret, and tenant ID
        credentials = ClientSecretCredential(client_id='<service principal ID>', client_secret='<service principal key>', tenant_id='<tenant ID>') 
        resource_client = ResourceManagementClient(credentials, subscription_id)
        adf_client = DataFactoryManagementClient(credentials, subscription_id)

        rg_params = {'location':'westus'}
        df_params = {'location':'westus'}
    ```

## <a name="create-a-data-factory"></a>建立 Data Factory

將下列程式碼新增至 **Main** 方法，以建立 **資料處理站**。 如果您的資源群組已經存在，請將第一個 `create_or_update` 陳述式變成註解。

```python
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    #Create a data factory
    df_resource = Factory(location='westus')
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
    ls_name = 'storageLinkedService001'

    # IMPORTANT: specify the name and key of your Azure Storage account.
    storage_string = SecureString(value='DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;EndpointSuffix=<suffix>')

    ls_azure_storage = LinkedServiceResource(properties=AzureStorageLinkedService(connection_string=storage_string)) 
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
    ds_ls = LinkedServiceReference(reference_name=ls_name)
    blob_path = '<container>/<folder path>'
    blob_filename = '<file name>'
    ds_azure_blob = DatasetResource(properties=AzureBlobDataset(
        linked_service_name=ds_ls, folder_path=blob_path, file_name=blob_filename)) 
    ds = adf_client.datasets.create_or_update(
        rg_name, df_name, ds_name, ds_azure_blob)
    print_item(ds)
```

### <a name="create-a-dataset-for-sink-azure-blob"></a>建立接收 Azure Blob 的資料集

將下列程式碼新增至 Main 方法，以建立 Azure Blob 資料集。 如需 Azure Blob 資料集屬性的相關資訊，請參閱 [Azure Blob 連接器](connector-azure-blob-storage.md#dataset-properties)文章。

您可以定義資料集來代表 Azure Blob 中的來源資料。 此 Blob 資料集會參考您在前一個步驟中建立的 Azure 儲存體連結服務。

```python
    # Create an Azure blob dataset (output)
    dsOut_name = 'ds_out'
    output_blobpath = '<container>/<folder path>'
    dsOut_azure_blob = DatasetResource(properties=AzureBlobDataset(linked_service_name=ds_ls, folder_path=output_blobpath))
    dsOut = adf_client.datasets.create_or_update(
        rg_name, df_name, dsOut_name, dsOut_azure_blob)
    print_item(dsOut)
```

## <a name="create-a-pipeline"></a>建立管線

將下列程式碼新增至 **Main** 方法，以建立 **具有複製活動的管線**。

```python
    # Create a copy activity
    act_name = 'copyBlobtoBlob'
    blob_source = BlobSource()
    blob_sink = BlobSink()
    dsin_ref = DatasetReference(reference_name=ds_name)
    dsOut_ref = DatasetReference(reference_name=dsOut_name)
    copy_activity = CopyActivity(name=act_name,inputs=[dsin_ref], outputs=[dsOut_ref], source=blob_source, sink=blob_sink)

    #Create a pipeline with the copy activity
    p_name = 'copyPipeline'
    params_for_pipeline = {}
    p_obj = PipelineResource(activities=[copy_activity], parameters=params_for_pipeline)
    p = adf_client.pipelines.create_or_update(rg_name, df_name, p_name, p_obj)
    print_item(p)
```

## <a name="create-a-pipeline-run"></a>建立管線執行

將下列程式碼新增至 **Main** 方法，以 **觸發管線執行**。

```python
    # Create a pipeline run
    run_response = adf_client.pipelines.create_run(rg_name, df_name, p_name, parameters={})
```

## <a name="monitor-a-pipeline-run"></a>監視管線執行

若要監視管道執行，將下列程式碼新增至 **Main** 方法：

```python
    # Monitor the pipeline run
    time.sleep(30)
    pipeline_run = adf_client.pipeline_runs.get(
        rg_name, df_name, run_response.run_id)
    print("\n\tPipeline run status: {}".format(pipeline_run.status))
    filter_params = RunFilterParameters(
        last_updated_after=datetime.now() - timedelta(1), last_updated_before=datetime.now() + timedelta(1))
    query_response = adf_client.activity_runs.query_by_pipeline_run(
        rg_name, df_name, pipeline_run.run_id, filter_params)
    print_activity_run_details(query_response.value[0])

```

現在，新增下列陳述式以在執行程式時叫用 **main** 方法：

```python
# Start the main method
main()
```

## <a name="full-script"></a>完整指令碼

以下是完整的 Python 程式碼：

```python
from azure.identity import ClientSecretCredential 
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


def main():

    # Azure subscription ID
    subscription_id = '<subscription ID>'

    # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
    rg_name = '<resource group>'

    # The data factory name. It must be globally unique.
    df_name = '<factory name>'

    # Specify your Active Directory client ID, client secret, and tenant ID
    credentials = ClientSecretCredential(client_id='<service principal ID>', client_secret='<service principal key>', tenant_id='<tenant ID>') 
    resource_client = ResourceManagementClient(credentials, subscription_id)
    adf_client = DataFactoryManagementClient(credentials, subscription_id)

    rg_params = {'location':'westus'}
    df_params = {'location':'westus'}
 
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    # Create a data factory
    df_resource = Factory(location='westus')
    df = adf_client.factories.create_or_update(rg_name, df_name, df_resource)
    print_item(df)
    while df.provisioning_state != 'Succeeded':
        df = adf_client.factories.get(rg_name, df_name)
        time.sleep(1)

    # Create an Azure Storage linked service
    ls_name = 'storageLinkedService001'

    # IMPORTANT: specify the name and key of your Azure Storage account.
    storage_string = SecureString(value='DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;EndpointSuffix=<suffix>')

    ls_azure_storage = LinkedServiceResource(properties=AzureStorageLinkedService(connection_string=storage_string)) 
    ls = adf_client.linked_services.create_or_update(rg_name, df_name, ls_name, ls_azure_storage)
    print_item(ls)

    # Create an Azure blob dataset (input)
    ds_name = 'ds_in'
    ds_ls = LinkedServiceReference(reference_name=ls_name)
    blob_path = '<container>/<folder path>'
    blob_filename = '<file name>'
    ds_azure_blob = DatasetResource(properties=AzureBlobDataset(
        linked_service_name=ds_ls, folder_path=blob_path, file_name=blob_filename))
    ds = adf_client.datasets.create_or_update(
        rg_name, df_name, ds_name, ds_azure_blob)
    print_item(ds)

    # Create an Azure blob dataset (output)
    dsOut_name = 'ds_out'
    output_blobpath = '<container>/<folder path>'
    dsOut_azure_blob = DatasetResource(properties=AzureBlobDataset(linked_service_name=ds_ls, folder_path=output_blobpath))
    dsOut = adf_client.datasets.create_or_update(
        rg_name, df_name, dsOut_name, dsOut_azure_blob)
    print_item(dsOut)

    # Create a copy activity
    act_name = 'copyBlobtoBlob'
    blob_source = BlobSource()
    blob_sink = BlobSink()
    dsin_ref = DatasetReference(reference_name=ds_name)
    dsOut_ref = DatasetReference(reference_name=dsOut_name)
    copy_activity = CopyActivity(name=act_name, inputs=[dsin_ref], outputs=[
                                 dsOut_ref], source=blob_source, sink=blob_sink)

    # Create a pipeline with the copy activity
    p_name = 'copyPipeline'
    params_for_pipeline = {}
    p_obj = PipelineResource(
        activities=[copy_activity], parameters=params_for_pipeline)
    p = adf_client.pipelines.create_or_update(rg_name, df_name, p_name, p_obj)
    print_item(p)

    # Create a pipeline run
    run_response = adf_client.pipelines.create_run(rg_name, df_name, p_name, parameters={})

    # Monitor the pipeline run
    time.sleep(30)
    pipeline_run = adf_client.pipeline_runs.get(
        rg_name, df_name, run_response.run_id)
    print("\n\tPipeline run status: {}".format(pipeline_run.status))
    filter_params = RunFilterParameters(
        last_updated_after=datetime.now() - timedelta(1), last_updated_before=datetime.now() + timedelta(1))
    query_response = adf_client.activity_runs.query_by_pipeline_run(
        rg_name, df_name, pipeline_run.run_id, filter_params)
    print_activity_run_details(query_response.value[0])


# Start the main method
main()
```

## <a name="run-the-code"></a>執行程式碼

建置並啟動應用程式，然後確認管線執行。

主控台會印出建立資料處理站、連結服務、資料集、管線和管線執行的進度。 請等待出現複製活動執行詳細資料及讀取/寫入的資料大小。 然後，使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)之類的工具，檢查 Blob 已從 "inputBlobPath" 複製到 "outputBlobPath" (您在變數中指定)。

以下是範例輸出：

```console
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
adf_client.factories.delete(rg_name, df_name)
```

## <a name="next-steps"></a>後續步驟

在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 瀏覽[教學課程](tutorial-copy-data-dot-net.md)以了解使用 Data Factory 的更多案例。
