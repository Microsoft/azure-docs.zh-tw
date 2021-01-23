---
title: 適用於 Azure Functions 的 Python 開發人員參考
description: 了解如何使用 Python 開發函式
ms.topic: article
ms.date: 11/4/2020
ms.custom: devx-track-python
ms.openlocfilehash: a13b4463d2a9c32a3487f839c0bf53b4c5bd2963
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98735838"
---
# <a name="azure-functions-python-developer-guide"></a>Azure Functions Python 開發人員指南

本文是使用 Python 開發 Azure Functions 的簡介。 本文假設您已閱讀過下列 [Azure Functions 開發人員指南](functions-reference.md)。

作為 Python 開發人員，您可能也會對下列其中一篇文章感興趣：

| 開始使用 | 概念| 案例/範例 |
| -- | -- | -- | 
| <ul><li>[使用 Visual Studio Code 的 Python 函式](./create-first-function-vs-code-csharp.md?pivots=programming-language-python)</li><li>[使用終端機/命令提示字元的 Python 函式](./create-first-function-cli-csharp.md?pivots=programming-language-python)</li></ul> | <ul><li>[開發人員指南](functions-reference.md)</li><li>[主機選項](functions-scale.md)</li><li>[效能 &nbsp; 考慮](functions-best-practices.md)</li></ul> | <ul><li>[透過 PyTorch 進行影像分類](machine-learning-pytorch.md)</li><li>[Azure 自動化範例](/samples/azure-samples/azure-functions-python-list-resource-groups/azure-functions-python-sample-list-resource-groups/)</li><li>[使用 TensorFlow 進行機器學習](functions-machine-learning-tensorflow.md)</li><li>[流覽 Python 範例](/samples/browse/?products=azure-functions&languages=python)</li></ul> |

## <a name="programming-model"></a>程式設計模型

Azure Functions 預期函式是 Python 指令碼中的無狀態方法，用來處理輸入及產生輸出。 根據預設，執行階段預期此方法將會實作為全域方法 (在 `__init__.py` 檔案中稱為 `main()`)。 您也可以指定[替代進入點](#alternate-entry-point)。

來自觸發程序和繫結的資料，透過方法屬性 (使用 *function.json* 檔案中定義的 `name` 屬性) 繫結至函式。 例如，下方的 _function.json_ 說明由名為 `req` HTTP 要求觸發的簡單函式：

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::

根據這個定義，包含函式程式碼的 `__init__.py` 檔案可能看起來像下列範例：

```python
def main(req):
    user = req.params.get('user')
    return f'Hello, {user}!'
```

您也可以使用 Python 型別註釋，在函式中明確地宣告參數類型並傳回型別。 這可協助您使用許多 Python 程式碼編輯器所提供的 IntelliSense 和自動完成功能。

```python
import azure.functions


def main(req: azure.functions.HttpRequest) -> str:
    user = req.params.get('user')
    return f'Hello, {user}!'
```

請使用 [azure.functions.*](/python/api/azure-functions/azure.functions?view=azure-python&preserve-view=true) 套件中所包含的 Python 註釋，以將輸入和輸出繫結至方法。

## <a name="alternate-entry-point"></a>替代進入點

您可以在 *function.json* 檔案中選擇性地指定 `scriptFile` 和 `entryPoint` 屬性，來變更函式的預設行為。 例如，下方的 _function.json_ 會告訴執行階段使用在 _main.py_ 檔案中的 `customentry()` 方法，以作為 Azure 函式的進入點。

```json
{
  "scriptFile": "main.py",
  "entryPoint": "customentry",
  "bindings": [
      ...
  ]
}
```

## <a name="folder-structure"></a>資料夾結構

Python 函式專案的建議資料夾結構看起來像下列範例：

```
 <project_root>/
 | - .venv/
 | - .vscode/
 | - my_first_function/
 | | - __init__.py
 | | - function.json
 | | - example.py
 | - my_second_function/
 | | - __init__.py
 | | - function.json
 | - shared_code/
 | | - __init__.py
 | | - my_first_helper_function.py
 | | - my_second_helper_function.py
 | - tests/
 | | - test_my_second_function.py
 | - .funcignore
 | - host.json
 | - local.settings.json
 | - requirements.txt
 | - Dockerfile
```
主要專案資料夾 ( # B0 project_root>) 可包含下列檔案：

* *local.settings.json*：在本機執行時用來儲存應用程式設定和連接字串。 此檔案不會發行至 Azure。 若要深入瞭解，請參閱 [local.settings.file](functions-run-local.md#local-settings-file)。
* *requirements.txt*：包含在發佈至 Azure 時，系統所安裝的 Python 套件清單。
* *host.json*：包含會對函式應用程式中所有函式產生影響的全域組態選項。 此檔案會發行至 Azure。 在本機執行時，不支援所有選項。 若要深入了解，請參閱[host.json](functions-host-json.md)。
* *vscode/*： (選用) 包含 store vscode configuration。 若要深入瞭解，請參閱 [VSCode 設定](https://code.visualstudio.com/docs/getstarted/settings)。
* *venv/*： (選擇性的) 包含本機開發所使用的 Python 虛擬環境。
* *Dockerfile*：在 [自訂容器](functions-create-function-linux-custom-image.md)中發佈您的專案時，會使用 (選用的) 。
* *測試/*： (選擇性) 包含函數應用程式的測試案例。
* *funcignore*： (選擇性) 宣告不應發佈至 Azure 的檔案。 通常，此檔案包含 `.vscode/` 忽略您的編輯器設定、忽略 `.venv/` 本機 Python 虛擬環境、 `tests/` 忽略測試案例，以及 `local.settings.json` 防止發行本機應用程式設定。

每個函式都具有本身的程式碼檔案和繫結設定檔 (function.json)。

當您將專案部署至 Azure 中的函式應用程式時，主要專案 (*<project_root>*) 資料夾的整個內容應該包含在套件中，而不是包含在封裝根目錄中的資料夾本身 `host.json` 。 我們建議您在此範例中，將您的測試與其他函式一起保留在資料夾中 `tests/` 。 如需詳細資訊，請參閱[單元測試](#unit-testing)。

## <a name="import-behavior"></a>匯入行為

您可以使用絕對和相對參考，在函式程式碼中匯入模組。 根據以上所示的資料夾結構，下列從函式檔案內的匯入工作 *<project_root> \my \_ first \_ function \\ _ \_ init \_ \_ . .py*：

```python
from shared_code import my_first_helper_function #(absolute)
```

```python
import shared_code.my_second_helper_function #(absolute)
```

```python
from . import example #(relative)
```

> [!NOTE]
>  使用絕對匯入語法時， *shared_code/* 資料夾必須包含 \_ \_ \_ \_ .py 檔案，以將它標示為 Python 套件。

下列 \_ \_ 應用程式匯 \_ \_ 入和超越最上層相對匯入已被取代，因為它不受靜態類型檢查程式支援，而且 Python 測試架構不支援：

```python
from __app__.shared_code import my_first_helper_function #(deprecated __app__ import)
```

```python
from ..shared_code import my_first_helper_function #(deprecated beyond top-level relative import)
```

## <a name="triggers-and-inputs"></a>觸發程序及輸入

在 Azure Functions 中輸入會分成兩個類別：觸發程序輸入和額外的輸入。 雖然其在 `function.json` 中是不同的，但在 Python 程式碼中的使用方式卻相同。  在本機執行時，觸發程序和輸入來源的連接字串或祕密會對應至 `local.settings.json` 檔案中的值，但在 Azure 中執行時，則會對應至應用程式設定。

例如，下列程式碼範例會示範兩者之間的差異：

```json
// function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "route": "items/{id}"
    },
    {
      "name": "obj",
      "direction": "in",
      "type": "blob",
      "path": "samples/{id}",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

```json
// local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsStorage": "<azure-storage-connection-string>"
  }
}
```

```python
# __init__.py
import azure.functions as func
import logging


def main(req: func.HttpRequest,
         obj: func.InputStream):

    logging.info(f'Python HTTP triggered function processed: {obj.read()}')
```

叫用此函式時，HTTP 要求會以 `req` 形式傳遞至函式。 系統會根據路由 URL 中的 _ID_ 從 Azure Blob 儲存體擷取輸入，並在函式主體中以 `obj` 形式提供。  在這裡，指定的儲存體帳戶是在 AzureWebJobsStorage 應用程式設定中找到的連接字串，這是函數應用程式所使用的同一個儲存體帳戶。


## <a name="outputs"></a>輸出

輸出可以使用傳回值和輸出參數來表示。 如果只有一個輸出，我們建議使用傳回的值。 若為多個輸出，您必須使用輸出參數。

若要使用函式的傳回值作為輸出繫結的值，應該將 `function.json` 中的繫結 `name` 屬性設定為 `$return`。

若要產生多個輸出，請使用 [`azure.functions.Out`](/python/api/azure-functions/azure.functions.out?view=azure-python&preserve-view=true) 介面提供的 `set()` 方法，將一值指派給繫結。 例如，下列函式可以將訊息推送至佇列，同時也會傳回 HTTP 回應。

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous"
    },
    {
      "name": "msg",
      "direction": "out",
      "type": "queue",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "direction": "out",
      "type": "http"
    }
  ]
}
```

```python
import azure.functions as func


def main(req: func.HttpRequest,
         msg: func.Out[func.QueueMessage]) -> str:

    message = req.params.get('body')
    msg.set(message)
    return message
```

## <a name="logging"></a>記錄

可以透過函式應用程式中的根 [`logging`](https://docs.python.org/3/library/logging.html#module-logging) 處理常式來存取 Azure Functions 執行階段記錄器。 這個記錄器會繫結至 Application Insights，並可讓您將函式執行期間遇到的警告和錯誤加上旗標。

下列範例在透過 HTTP 觸發程序叫用函式時，會記錄一則資訊訊息。

```python
import logging


def main(req):
    logging.info('Python HTTP trigger function processed a request.')
```

其他的記錄方法可讓您在不同追蹤層級寫入主控台記錄中︰

| 方法                 | 描述                                |
| ---------------------- | ------------------------------------------ |
| **`critical(_message_)`**   | 在根記錄器上寫入層級為 CRITICAL (重大) 的訊息。  |
| **`error(_message_)`**   | 在根記錄器上寫入層級為 ERROR (錯誤) 的訊息。    |
| **`warning(_message_)`**    | 在根記錄器上寫入層級為 WARNING (警告) 的訊息。  |
| **`info(_message_)`**    | 在根記錄器上寫入層級為 INFO (資訊) 的訊息。  |
| **`debug(_message_)`** | 在根記錄器上寫入層級為 DEBUG (偵錯) 的訊息。  |

若要深入了解記錄，請參閱[監視 Azure Functions](functions-monitoring.md)。

## <a name="http-trigger-and-bindings"></a>HTTP 觸發程序和繫結

HTTP 觸發程式是在檔案的 function.js中定義。 繫結的 `name` 必須符合函式中的具名引數。
在先前的範例中，使用的是繫結名稱 `req`。 這個參數是 [HttpRequest] 物件，而且會傳回 [HttpResponse] 物件。

從 [HttpRequest] 物件中，您可以取得要求標題、查詢參數、路由參數和訊息內文。

下列範例來自 [Python 的 HTTP 觸發程序範本](https://github.com/Azure/azure-functions-templates/tree/dev/Functions.Templates/Templates/HttpTrigger-Python)。

```python
def main(req: func.HttpRequest) -> func.HttpResponse:
    headers = {"my-http-header": "some-value"}

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!", headers=headers)
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             headers=headers, status_code=400
        )
```

在此函數中，`name` 查詢參數的值取自於 [HttpRequest] 物件的 `params` 參數。 JSON 編碼的訊息內文是使用 `get_json` 方法來讀取。

同樣地，您可以在傳回的 [HttpResponse] 物件中設定回應訊息的 `status_code` 和 `headers`。

## <a name="scaling-and-performance"></a>調整和效能

如需 Python 函式應用程式的調整和效能最佳做法，請參閱 [python 規模和效能文章](python-scale-performance-reference.md)。

## <a name="context"></a>Context

若要在執行期間取得函式的引動內容，請在其簽章中包含 [`context`](/python/api/azure-functions/azure.functions.context?view=azure-python&preserve-view=true) 引數。

例如：

```python
import azure.functions


def main(req: azure.functions.HttpRequest,
         context: azure.functions.Context) -> str:
    return f'{context.invocation_id}'
```

[**Context**](/python/api/azure-functions/azure.functions.context?view=azure-python&preserve-view=true) 類別具有下列字串屬性：

`function_directory` 函式執行所在的目錄。

`function_name` 函數的名稱。

`invocation_id` 目前函式叫用的識別碼。

## <a name="global-variables"></a>全域變數

不保證您應用程式的狀態將保留給未來執行使用。 不過，Azure Functions 執行階段通常會針對相同應用程式的多個執行重複使用相同的處理序。 為了快取昂貴計算的結果，請將其宣告為全域變數。

```python
CACHED_DATA = None


def main(req):
    global CACHED_DATA
    if CACHED_DATA is None:
        CACHED_DATA = load_json()

    # ... use CACHED_DATA in code
```

## <a name="environment-variables"></a>環境變數

在 Functions 中，[應用程式設定](functions-app-settings.md) (例如服務連接字串) 在執行期間會公開為環境變數。 您可以藉由宣告 `import os`，然後使用 `setting = os.environ["setting-name"]` 來存取這些設定。

下列範例會使用名為 `myAppSetting` 的索引鍵來取得[應用程式設定](functions-how-to-use-azure-function-app-settings.md#settings)：

```python
import logging
import os
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:

    # Get the setting named 'myAppSetting'
    my_app_setting_value = os.environ["myAppSetting"]
    logging.info(f'My app setting value:{my_app_setting_value}')
```

為了本機開發，應用程式設定會[保存在 local.settings.json 檔案中](functions-run-local.md#local-settings-file)。

## <a name="python-version"></a>Python 版本

Azure Functions 支援下列 Python 版本：

| Functions 版本 | Python <sup>*</sup> 版本 |
| ----- | ----- |
| 3.x | 3.9 (Preview)  <br/> 3.8<br/>3.7<br/>3.6 |
| 2.x | 3.7<br/>3.6 |

<sup>*</sup>官方 CPython 散發套件

若要在 Azure 中建立函式應用程式時要求特定的 Python 版本，請使用 [`az functionapp create`](/cli/azure/functionapp#az-functionapp-create) 命令的 `--runtime-version` 選項。 Functions 執行階段版本是由 `--functions-version` 選項設定。 Python 版本是在函式應用程式建立時設定的，而且無法變更。

在本機執行時，執行階段會使用可用的 Python 版本。

## <a name="package-management"></a>套件管理

使用 Azure Functions Core Tools 或 Visual Studio Code 在本機進行開發時，將所需的套件名稱和版本新增到 `requirements.txt` 檔案，並使用 `pip` 進行安裝。

例如，可以使用下列要求和 pip 命令從 PyPl 安裝 `requests` 套件。

```txt
requests==2.19.1
```

```bash
pip install -r requirements.txt
```

## <a name="publishing-to-azure"></a>發行到 Azure

當您準備發佈時，請確保所有可公開使用的相依性都列在 requirements.txt 檔案中，而此檔案位於專案目錄的根目錄中。

從發佈中排除的專案檔案和資料夾 (包括虛擬環境資料夾) 會列在 .funcignore 檔案中。

有三個組建動作支援將 Python 專案發佈至 Azure：遠端組建、本機組建和使用自訂相依性的組建。

您也可以使用 Azure Pipelines 來建立相依性，並使用持續傳遞 (CD) 發佈。 若要深入瞭解，請參閱 [使用 Azure DevOps 的持續傳遞](functions-how-to-azure-devops.md)。

### <a name="remote-build"></a>遠端組建

使用遠端組建時，伺服器上還原的相依性和原生相依性會符合生產環境。 這會導致較小的部署套件上傳。 在 Windows 上開發 Python 應用程式時，請使用遠端組建。 如果您的專案有自訂相依性，您可以 [使用具有額外索引 URL 的遠端組建](#remote-build-with-extra-index-url)。

相依性是根據 requirements.txt 檔案的內容從遠端取得的。 [遠端組建](functions-deployment-technologies.md#remote-build)是建議的組建方法。 根據預設，當您使用下列 [func Azure functionapp publish](functions-run-local.md#publish) 命令，將 Python 專案發佈至 Azure 時，Azure Functions Core Tools 會要求遠端組建。

```bash
func azure functionapp publish <APP_NAME>
```

在 Azure 中，請記得以您的函式應用程式名稱取代 `<APP_NAME>`。

根據預設，[適用於 Visual Studio Code 的 Azure Functions 擴充功能](./create-first-function-vs-code-csharp.md#publish-the-project-to-azure)也會要求遠端組建。

### <a name="local-build"></a>本機組建

相依性是根據 requirements.txt 檔案的內容在本機取得的。 您可以使用下列 [func azure functionapp publish](functions-run-local.md#publish) 命令，搭配本機組建進行發佈，以防止執行遠端組建。

```command
func azure functionapp publish <APP_NAME> --build local
```

在 Azure 中，請記得以您的函式應用程式名稱取代 `<APP_NAME>`。

使用 `--build local` 選項時，會從 requirements.txt 檔案讀取專案相依性，以及在本機下載並安裝這些相依套件。 專案檔案和相依性會從您的本機電腦部署至 Azure。 這會導致更大的部署套件上傳至 Azure。 如果基於某些原因，Core Tools 無法取得 requirements.txt 檔案中的相依性，則您必須使用自訂相依性選項來進行發佈。

在 Windows 本機開發時，不建議使用本機組建。

### <a name="custom-dependencies"></a>自訂相依性

當您的專案在 [Python 封裝索引](https://pypi.org/)中找不到相依性時，有兩種方式可建立專案。 組建方法取決於您建立專案的方式。

#### <a name="remote-build-with-extra-index-url"></a>具有額外索引 URL 的遠端組建

當您的封裝可從可存取的自訂套件索引中使用時，請使用遠端組建。 在發佈之前，請務必建立名為的 [應用程式設定](functions-how-to-use-azure-function-app-settings.md#settings) `PIP_EXTRA_INDEX_URL` 。 這項設定的值是自訂套件索引的 URL。 使用此設定會告訴您要使用選項來執行遠端組建 `pip install` `--extra-index-url` 。 若要深入瞭解，請參閱 [Python pip 安裝檔](https://pip.pypa.io/en/stable/reference/pip_install/#requirements-file-format)。

您也可以搭配使用基本驗證認證與額外的套件索引 Url。 若要深入瞭解，請參閱 Python 檔中的 [基本驗證認證](https://pip.pypa.io/en/stable/user_guide/#basic-authentication-credentials) 。

#### <a name="install-local-packages"></a>安裝本機套件

如果您的專案使用我們工具無法公開使用的套件，您可以將其放在 \_\_app\_\_/.python_packages 目錄中，供您的應用程式使用。 在發佈之前，請執行下列命令，在本機安裝相依性：

```command
pip install  --target="<PROJECT_DIR>/.python_packages/lib/site-packages"  -r requirements.txt
```

使用自訂相依性時，您應該使用 `--no-build` 發行選項，因為您已經將相依性安裝在專案資料夾中。

```command
func azure functionapp publish <APP_NAME> --no-build
```

在 Azure 中，請記得以您的函式應用程式名稱取代 `<APP_NAME>`。

## <a name="unit-testing"></a>單元測試

以 Python 撰寫的函式可以使用標準測試架構來測試，就像其他 Python 程式碼一樣。 對於大部分繫結，您可以從 `azure.functions` 套件建立適當類別的執行個體，以建立模擬輸入物件。 由於 [`azure.functions`](https://pypi.org/project/azure-functions/) 套件無法立即使用，請務必透過 `requirements.txt` 檔案加以安裝，如先前的[套件管理](#package-management)一節中所述。

以 *my_second_function* 為例，下列是 HTTP 觸發函式的模擬測試：

首先，我們需要建立檔案 *的<project_root>/my_second_function/function.js* ，並將此函式定義為 HTTP 觸發程式。

```json
{
  "scriptFile": "__init__.py",
  "entryPoint": "main",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

現在，我們可以執行 *my_second_function* 和 *shared_code。我的 _second_helper_function*。

```python
# <project_root>/my_second_function/__init__.py
import azure.functions as func
import logging

# Use absolute import to resolve shared_code modules
from shared_code import my_second_helper_function

# Define an http trigger which accepts ?value=<int> query parameter
# Double the value and return the result in HttpResponse
def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Executing my_second_function.')

    initial_value: int = int(req.params.get('value'))
    doubled_value: int = my_second_helper_function.double(initial_value)

    return func.HttpResponse(
      body=f"{initial_value} * 2 = {doubled_value}",
      status_code=200
    )
```

```python
# <project_root>/shared_code/__init__.py
# Empty __init__.py file marks shared_code folder as a Python package
```

```python
# <project_root>/shared_code/my_second_helper_function.py

def double(value: int) -> int:
  return value * 2
```

我們可以開始撰寫 HTTP 觸發程式的測試案例。

```python
# <project_root>/tests/test_my_second_function.py
import unittest

import azure.functions as func
from my_second_function import main

class TestFunction(unittest.TestCase):
    def test_my_second_function(self):
        # Construct a mock HTTP request.
        req = func.HttpRequest(
            method='GET',
            body=None,
            url='/api/my_second_function',
            params={'value': '21'})

        # Call the function.
        resp = main(req)

        # Check the output.
        self.assertEqual(
            resp.get_body(),
            b'21 * 2 = 42',
        )
```

在您的 `.venv` python 虛擬環境中，安裝您最愛的 python 測試架構 (例如 `pip install pytest`) 。 只要執行 `pytest tests` 檢查測試結果即可。

## <a name="temporary-files"></a>暫存檔案

`tempfile.gettempdir()` 方法會傳回暫存資料夾，在 Linux 上，這是 `/tmp`。 您的應用程式可以使用此目錄，來儲存您函式在執行期間所產生及使用的暫存檔案。

> [!IMPORTANT]
> 寫入臨時目錄的檔案不保證會在叫用之間持續存在。 在擴增期間，暫存檔案不會在執行個體之間共用。

下列範例會在暫存目錄 (`/tmp`) 中建立暫存檔案：

```python
import logging
import azure.functions as func
import tempfile
from os import listdir

#---
   tempFilePath = tempfile.gettempdir()
   fp = tempfile.NamedTemporaryFile()
   fp.write(b'Hello world!')
   filesDirListInTemp = listdir(tempFilePath)
```

建議您在與專案資料夾不同的資料夾中保存您的測試。 這讓您可免於將測試程式碼與應用程式一起部署。

## <a name="preinstalled-libraries"></a>預先安裝的程式庫

有一些程式庫隨附于 Python 函式執行時間。

### <a name="python-standard-library"></a>Python 標準程式庫

Python 標準程式庫包含每個 Python 散發套件隨附的內建 Python 模組清單。 這些程式庫大多可協助您存取系統功能，例如檔案 i/o。 在 Windows 系統上，這些程式庫會隨 Python 安裝。 在以 Unix 為基礎的系統上，它們是由套件集合提供。

若要查看這些程式庫清單的完整詳細資料，請流覽下列連結：

* [Python 3.6 標準程式庫](https://docs.python.org/3.6/library/)
* [Python 3.7 標準程式庫](https://docs.python.org/3.7/library/)
* [Python 3.8 標準程式庫](https://docs.python.org/3.8/library/)
* [Python 3.9 標準程式庫](https://docs.python.org/3.9/library/)

### <a name="azure-functions-python-worker-dependencies"></a>Azure Functions Python 背景工作相依性

Python 背景工作角色需要一組特定的程式庫。 您也可以在函式中使用這些程式庫，但它們不是 Python 標準的一部分。 如果您的函式依賴任何這些程式庫，則您的程式碼在 Azure Functions 外部執行時可能無法使用這些程式庫。 您可以在 [setup.py](https://github.com/Azure/azure-functions-python-worker/blob/dev/setup.py#L282)檔案的 [**安裝 \_ 需要**] 區段中找到詳細的相依性清單。

> [!NOTE]
> 如果函數應用程式的 requirements.txt 包含 `azure-functions-worker` 專案，請將它移除。 函式背景工作會由 Azure Functions 平臺自動管理，並定期以新功能和 bug 修正進行更新。 在 requirements.txt 中手動安裝舊版本的背景工作角色可能會造成未預期的問題。

### <a name="azure-functions-python-library"></a>Azure Functions Python 程式庫

每個 Python 背景工作更新都包含新版本的 [Azure Functions Python 程式庫 (Azure. ](https://github.com/Azure/azure-functions-python-library)函式) 。 這種方法可讓您更輕鬆地持續更新 Python 函數應用程式，因為每個更新都是回溯相容的。 您可以在 [azure 函式 PyPi](https://pypi.org/project/azure-functions/#history)中找到此程式庫的版本清單。

執行時間程式庫版本是由 Azure 所修正，且無法由 requirements.txt 覆寫。 `azure-functions`requirements.txt 中的專案僅供 linting 和客戶感知之用。

使用下列程式碼，在您的執行時間中追蹤 Python 函式程式庫的實際版本：

```python
getattr(azure.functions, '__version__', '< 1.2.1')
```

### <a name="runtime-system-libraries"></a>執行時間系統程式庫

如需 Python 背景工作 Docker 映射中預先安裝系統程式庫的清單，請遵循下列連結：

|  Functions 執行階段  | Debian 版本 | Python 版本 |
|------------|------------|------------|
| 2.x 版 | 延展  | [Python 3.6](https://github.com/Azure/azure-functions-docker/blob/master/host/2.0/stretch/amd64/python/python36/python36.Dockerfile)<br/>[Python 3.7](https://github.com/Azure/azure-functions-docker/blob/master/host/2.0/stretch/amd64/python/python37/python37.Dockerfile) |
| 3.x 版 | 剋星 | [Python 3.6](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python36/python36.Dockerfile)<br/>[Python 3.7](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python37/python37.Dockerfile)<br />[Python 3.8](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python38/python38.Dockerfile)<br/> [Python 3。9](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python39/python39.Dockerfile)|

## <a name="cross-origin-resource-sharing"></a>跨原始資源共用

[!INCLUDE [functions-cors](../../includes/functions-cors.md)]

Python 函式應用程式完全支援 CORS。

## <a name="known-issues-and-faq"></a>已知問題和常見問題集

以下是常見問題的疑難排解指南清單：

* [ModuleNotFoundError 和 ImportError](recover-python-functions.md#troubleshoot-modulenotfounderror)
* [無法匯入 ' cygrpc '](recover-python-functions.md#troubleshoot-cannot-import-cygrpc)

所有已知問題和功能要求則會使用 [GitHub 問題](https://github.com/Azure/azure-functions-python-worker/issues)清單追蹤。 如果您遇到問題，但在 GitHub 中卻找不到此問題，請開啟新的問題，並包含問題的詳細說明。

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱下列資源：

* [Azure Functions 套件 API 文件](/python/api/azure-functions/azure.functions?view=azure-python&preserve-view=true)
* [Azure Functions 的最佳做法](functions-best-practices.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
* [Blob 儲存體繫結](functions-bindings-storage-blob.md)
* [HTTP 和 Webhook 繫結](functions-bindings-http-webhook.md)
* [佇列儲存體繫結](functions-bindings-storage-queue.md)
* [計時器觸發程序](functions-bindings-timer.md)

[有任何問題嗎？請告訴我們。](https://aka.ms/python-functions-ref-survey)


[HttpRequest]: /python/api/azure-functions/azure.functions.httprequest?view=azure-python&preserve-view=true
[HttpResponse]: /python/api/azure-functions/azure.functions.httpresponse?view=azure-python&preserve-view=true