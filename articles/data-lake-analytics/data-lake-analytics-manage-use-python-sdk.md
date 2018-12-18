---
title: 使用 Python 來管理 Azure Data Lake Analytics
description: 本文說明如何使用 Python 來管理 Data Lake Analytics 帳戶、資料來源、使用者和作業。
services: data-lake-analytics
ms.service: data-lake-analytics
author: matt1883
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: d4213a19-4d0f-49c9-871c-9cd6ed7cf731
ms.topic: conceptual
ms.date: 06/08/2018
ms.openlocfilehash: f73ef118efbdfc94d8cb9b7d81717bd13511c785
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048277"
---
# <a name="manage-azure-data-lake-analytics-using-python"></a>使用 Python 來管理 Azure Data Lake Analytics
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

本文說明如何使用 Python 來管理 Azure Data Lake Analytics 帳戶、資料來源、使用者和作業。

## <a name="supported-python-versions"></a>支援的 Python 版本

* 使用 64 位元版 Python。
* 您可以使用在 **[Python.org 下載](https://www.python.org/downloads/)** \(英文\) 找到的標準 Python 散發套件。 
* 許多開發人員發現使用 **[Anaconda Python 散發套件](https://www.anaconda.com/download/)** \(英文\) 相當便利。  
* 本文是使用來自標準 Python 散發套件的 Python 3.6 版來撰寫的

## <a name="install-azure-python-sdk"></a>安裝 Azure Python SDK

請安裝下列模組：

* **azure-mgmt-resource** 模組包含適用於 Active Directory 等等的其他 Azure 模組。
* **azure-datalake-store** 模組包含 Azure Data Lake Store 檔案系統作業。 
* **azure-mgmt-datalake-store** 模組包含 Azure Data Lake Store 帳戶管理作業。
* **azure-mgmt-datalake-analytics** 模組包含 Azure Data Lake Analytics 作業。 

請先執行下列命令，以確保您擁有最新的 `pip`：

```
python -m pip install --upgrade pip
```

本文件是使用 `pip version 9.0.1` 撰寫的。

使用下列 `pip` 命令以從命令列安裝新模組：

```
pip install azure-mgmt-resource
pip install azure-datalake-store
pip install azure-mgmt-datalake-store
pip install azure-mgmt-datalake-analytics
```

## <a name="create-a-new-python-script"></a>建立新的 Python 指令碼

將下列程式碼貼到指令碼中：

```python
## Use this only for Azure AD service-to-service authentication
#from azure.common.credentials import ServicePrincipalCredentials

## Use this only for Azure AD end-user authentication
#from azure.common.credentials import UserPassCredentials

## Required for Azure Resource Manager
from azure.mgmt.resource.resources import ResourceManagementClient
from azure.mgmt.resource.resources.models import ResourceGroup

## Required for Azure Data Lake Store account management
from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
from azure.mgmt.datalake.store.models import DataLakeStoreAccount

## Required for Azure Data Lake Store filesystem management
from azure.datalake.store import core, lib, multithread

## Required for Azure Data Lake Analytics account management
from azure.mgmt.datalake.analytics.account import DataLakeAnalyticsAccountManagementClient
from azure.mgmt.datalake.analytics.account.models import DataLakeAnalyticsAccount, DataLakeStoreAccountInfo

## Required for Azure Data Lake Analytics job management
from azure.mgmt.datalake.analytics.job import DataLakeAnalyticsJobManagementClient
from azure.mgmt.datalake.analytics.job.models import JobInformation, JobState, USqlJobProperties

## Required for Azure Data Lake Analytics catalog management
from azure.mgmt.datalake.analytics.catalog import DataLakeAnalyticsCatalogManagementClient

## Use these as needed for your application
import logging, getpass, pprint, uuid, time
```

請執行此指令碼以確認可將模組匯入。

## <a name="authentication"></a>驗證

### <a name="interactive-user-authentication-with-a-pop-up"></a>使用快顯視窗進行互動式使用者驗證

不支援此方法。

### <a name="interactive-user-authentication-with-a-device-code"></a>使用裝置代碼進行互動式使用者驗證

```python
user = input('Enter the user to authenticate with that has permission to subscription: ')
password = getpass.getpass()
credentials = UserPassCredentials(user, password)
```

### <a name="noninteractive-authentication-with-spi-and-a-secret"></a>使用 SPI 和祕密進行非互動式驗證

```python
credentials = ServicePrincipalCredentials(client_id = 'FILL-IN-HERE', secret = 'FILL-IN-HERE', tenant = 'FILL-IN-HERE')
```

### <a name="noninteractive-authentication-with-api-and-a-certificate"></a>使用 API 和憑證進行非互動式驗證

不支援此方法。

## <a name="common-script-variables"></a>通用指令碼變數

以下是範例中使用的變數。

```python
subid= '<Azure Subscription ID>'
rg = '<Azure Resource Group Name>'
location = '<Location>' # i.e. 'eastus2'
adls = '<Azure Data Lake Store Account Name>'
adla = '<Azure Data Lake Analytics Account Name>'
```

## <a name="create-the-clients"></a>建立用戶端

```python
resourceClient = ResourceManagementClient(credentials, subid)
adlaAcctClient = DataLakeAnalyticsAccountManagementClient(credentials, subid)
adlaJobClient = DataLakeAnalyticsJobManagementClient( credentials, 'azuredatalakeanalytics.net')
```

## <a name="create-an-azure-resource-group"></a>建立 Azure 資源群組

```python
armGroupResult = resourceClient.resource_groups.create_or_update( rg, ResourceGroup( location=location ) )
```

## <a name="create-data-lake-analytics-account"></a>建立 Data Lake Analytics 帳戶

首先，建立一個存放區帳戶。

```python
adlsAcctResult = adlsAcctClient.account.create(
    rg,
    adls,
    DataLakeStoreAccount(
        location=location)
    )
).wait()
```
接著，建立一個使用該存放區的 ADLA 帳戶。

```python
adlaAcctResult = adlaAcctClient.account.create(
    rg,
    adla,
    DataLakeAnalyticsAccount(
        location=location,
        default_data_lake_store_account=adls,
        data_lake_store_accounts=[DataLakeStoreAccountInfo(name=adls)]
    )
).wait()
```

## <a name="submit-a-job"></a>提交作業

```python
script = """
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"""

jobId = str(uuid.uuid4())
jobResult = adlaJobClient.job.create(
    adla,
    jobId,
    JobInformation(
        name='Sample Job',
        type='USql',
        properties=USqlJobProperties(script=script)
    )
)
```

## <a name="wait-for-a-job-to-end"></a>等候工作結束

```python
jobResult = adlaJobClient.job.get(adla, jobId)
while(jobResult.state != JobState.ended):
    print('Job is not yet done, waiting for 3 seconds. Current state: ' + jobResult.state.value)
    time.sleep(3)
    jobResult = adlaJobClient.job.get(adla, jobId)

print ('Job finished with result: ' + jobResult.result.value)
```

## <a name="list-pipelines-and-recurrences"></a>列出管線和週期
視您作業是否有附加的管線或週期中繼資料而定，您可以列出管線和週期。

```python
pipelines = adlaJobClient.pipeline.list(adla)
for p in pipelines:
    print('Pipeline: ' + p.name + ' ' + p.pipelineId)

recurrences = adlaJobClient.recurrence.list(adla)
for r in recurrences:
    print('Recurrence: ' + r.name + ' ' + r.recurrenceId)
```

## <a name="manage-compute-policies"></a>管理計算原則

DataLakeAnalyticsAccountManagementClient 物件會提供方法，用以管理 Data Lake Analytics 帳戶的計算原則。

### <a name="list-compute-policies"></a>列出計算原則

下列程式碼會擷取 Data Lake Analytics 帳戶的計算原則清單。

```python
policies = adlaAccountClient.computePolicies.listByAccount(rg, adla)
for p in policies:
    print('Name: ' + p.name + 'Type: ' + p.objectType + 'Max AUs / job: ' + p.maxDegreeOfParallelismPerJob + 'Min priority / job: ' + p.minPriorityPerJob)
```

### <a name="create-a-new-compute-policy"></a>建立新的計算原則

下列程式碼會為 Data Lake Analytics 帳戶建立新的計算原則，其中是將指定使用者可用的 AU 上限設定為 50，而將作業最低優先順序設定為 250。

```python
userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde"
newPolicyParams = ComputePolicyCreateOrUpdateParameters(userAadObjectId, "User", 50, 250)
adlaAccountClient.computePolicies.createOrUpdate(rg, adla, "GaryMcDaniel", newPolicyParams)
```

## <a name="next-steps"></a>後續步驟

- 若要使用其他工具檢視同一個教學課程，請按一下頁面最上方的索引標籤選取器。
- 若要了解 U-SQL，請參閱 [開始使用 Azure Data Lake Analytics U-SQL 語言](data-lake-analytics-u-sql-get-started.md)。
- 針對管理工作，請參閱 [使用 Azure 入口網站管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-portal.md)。

