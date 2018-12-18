---
title: 快速入門：使用 Azure 資料總管 Python 程式庫查詢資料
description: 在本快速入門中，您將了解如何使用 Python，從 Azure 資料總管查詢資料。
services: data-explorer
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: quickstart
ms.date: 09/24/2018
ms.openlocfilehash: 17504cec6ddf98aca8ddfc6c91d21d34055902f6
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394845"
---
# <a name="quickstart-query-data-using-the-azure-data-explorer-python-library"></a>快速入門：使用 Azure 資料總管 Python 程式庫查詢資料

Azure 資料總管是一項快速又可高度調整的資料探索服務，可用於處理記錄和遙測資料。 Azure 資料總管提供一個[適用於 Python 的資料用戶端程式庫](https://github.com/Azure/azure-kusto-python/tree/master/azure-kusto-data) \(英文\)。 此程式庫可讓您從程式碼中查詢資料。 在本快速入門中，您要連線到「說明叢集」上的資料表，我們已設定此叢集來協助學習。 您接著要查詢該叢集上的資料表，並傳回結果。

本快速入門也可用來作為 [Azure Notebook](https://notebooks.azure.com/ManojRaheja/libraries/KustoPythonSamples/html/QueryKusto.ipynb) \(英文\)。

## <a name="prerequisites"></a>必要條件

* 屬於 Azure Active Directory (AAD) 成員的組織電子郵件帳戶

* 安裝在您開發電腦上的 [Python](https://www.python.org/downloads/)

## <a name="install-the-data-library"></a>安裝資料程式庫

安裝 *azure-kusto-data*。

```python
pip install azure-kusto-data==0.0.13
```

## <a name="add-import-statements-and-constants"></a>新增 Import 陳述式和常數

從程式庫匯入類別，以及 *pandas* (資料分析程式庫)。

```python
from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder
from azure.kusto.data.exceptions import KustoServiceError
from azure.kusto.data.helpers import dataframe_from_result_table
import pandas as pd
```

為了驗證應用程式，Azure 資料總管會使用您的 AAD 租用戶識別碼。 若要尋找您的租用戶識別碼，請使用下列 URL，並以您的網域取代 *YourDomain*。

```
https://login.windows.net/<YourDomain>/.well-known/openid-configuration/
```

例如，如果您的網域為 *contoso.com*，則 URL 會是：[https://login.windows.net/contoso.com/.well-known/openid-configuration/](https://login.windows.net/contoso.com/.well-known/openid-configuration/)。 按一下此 URL 來查看結果；第一行如下所示。

```
"authorization_endpoint":"https://login.windows.net/6babcaad-604b-40ac-a9d7-9fd97c0b779f/oauth2/authorize"
```

在此案例中，租用戶識別碼為 `6babcaad-604b-40ac-a9d7-9fd97c0b779f`。 先設定 AAD_TENANT_ID 的值，然後再執行此程式碼。

```python
AAD_TENANT_ID = "<TenantId>"
KUSTO_CLUSTER = "https://help.kusto.windows.net/"
KUSTO_DATABASE  = "Samples"
```

現在來建構連接字串。 此範例使用服務驗證來存取叢集。 您也可以使用 AAD 應用程式憑證、AAD 應用程式金鑰，以及 AAD 使用者和密碼。

```python
KCSB = KustoConnectionStringBuilder.with_aad_device_authentication(KUSTO_CLUSTER)
KCSB.authority_id = AAD_TENANT_ID
```

## <a name="connect-to-azure-data-explorer-and-execute-a-query"></a>連線到 Azure 資料總管並執行查詢

針對叢集執行查詢，然後將輸出儲存於資料框架中。 此程式碼執行時，會傳回與下列類似的訊息：若要登入，請使用網頁瀏覽器開啟頁面 https://microsoft.com/devicelogin，並輸入程式碼 F3W4VWZDM 來驗證。 請遵循下列步驟來登入，然後返回以執行下一個程式碼區塊。

```python
KUSTO_CLIENT  = KustoClient(KCSB)
KUSTO_QUERY  = "StormEvents | sort by StartTime desc | take 10"

RESPONSE = KUSTO_CLIENT.execute(KUSTO_DATABASE, KUSTO_QUERY)
```

## <a name="explore-data-in-dataframe"></a>在 DataFrame 中探索資料

當您進入登入之後，查詢會傳回結果，並將其儲存於資料框架中。 您可以像處理任何其他資料框架一樣來處理結果。

```python
df = dataframe_from_result_table(RESPONSE.primary_results[0])
df
```

您應該會看到來自 StormEvents 資料表的前十個結果。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [快速入門：使用 Azure 資料總管 Python 程式庫內嵌資料](python-ingest-data.md)