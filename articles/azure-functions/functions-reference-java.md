---
title: 適用於 Azure Functions 的 Java 開發人員參考 | Microsoft Docs
description: 了解如何使用 Java 開發函式。
services: functions
documentationcenter: na
author: rloutlaw
manager: justhe
keywords: azure functions, 函式, 事件處理, webhook, 動態計算, 無伺服器架構, Java
ms.service: functions
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/10/2018
ms.author: routlaw
ms.openlocfilehash: bbc1c3426b52e71db84a988b39a1d76ac24b6168
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697006"
---
# <a name="azure-functions-java-developer-guide"></a>Azure Functions Java 開發人員指南

[!INCLUDE [functions-java-preview-note](../../includes/functions-java-preview-note.md)]

## <a name="programming-model"></a>程式設計模型 

您的 Azure 函式應該是無狀態類別方法，處理輸入及產生輸出。 雖然您可以寫入執行個體方法，但是函式不得相依於類別的任何執行個體欄位。 所有函式方法必須有 `public` 存取修飾詞。

您可以在專案中放入多個函式。 請勿將函式放入個別的 jar。

## <a name="triggers-and-annotations"></a>觸發程序和註解

 Azure 函式可透過觸發程序 (例如，HTTP 要求、計時器或資料的更新) 來叫用。 函式必須處理該觸發程序和任何其他輸入，以產生一或多個輸出。

請使用 [com.microsoft.azure.functions.annotation.*](/java/api/com.microsoft.azure.functions.annotation) 套件中所包含的 Java 註釋，以將輸入和輸出繫結至方法。 使用註釋的程式碼範例位於每個註釋的 [Java 參考文件](/java/api/com.microsoft.azure.functions.annotation)中，以及 Azure Functions 繫結參考文件中，例如 [HTTP 觸發程序](/azure/azure-functions/functions-bindings-http-webhook)的文件。

觸發程序輸入和輸出也可以定義於函式的 [function.json](/azure/azure-functions/functions-reference#function-code) 中，而不透過註釋來定義。 不建議您以這種方式使用 `function.json` 而不使用註釋。

> [!IMPORTANT] 
> 您必須在 [local.settings.json](/azure/azure-functions/functions-run-local#local-settings-file) 中設定 Azure 儲存體帳戶，以在本機執行 Azure 儲存體 Blob、佇列或資料表觸發程序。

使用註釋的範例：

```java
public class Function {
    public String echo(@HttpTrigger(name = "req", 
      methods = {"post"},  authLevel = AuthorizationLevel.ANONYMOUS) 
        String req, ExecutionContext context) {
        return String.format(req);
    }
}
```

撰寫的相同函式且沒有註釋：

```java
package com.example;

public class MyClass {
    public static String echo(String in) {
        return in;
    }
}
```

具有對應 `function.json`：

```json
{
  "scriptFile": "azure-functions-example.jar",
  "entryPoint": "com.example.MyClass.echo",
  "bindings": [
    {
      "type": "httpTrigger",
      "name": "req",
      "direction": "in",
      "authLevel": "anonymous",
      "methods": [ "post" ]
    },
    {
      "type": "http",
      "name": "$return",
      "direction": "out"
    }
  ]
}

```

## <a name="third-party-libraries"></a>第三方程式庫 

Azure Functions 支援使用第三方程式庫。 根據預設，專案的 `pom.xml` 檔案中所指定的相依性全都會在 `mvn package` 目標期間自動配套。 對於在 `pom.xml` 檔案中未指定為相依性的程式庫，請將其放入函式根目錄的 `lib` 目錄中。 放在 `lib` 目錄中的相依性會在執行階段新增至系統類別載入器。

## <a name="data-type-support"></a>資料類型支援

您可以針對輸入和輸出資料在 Java 中使用任何資料類型，包括原生類型。自訂 Java 類型及特殊 Azure 類型會在 `azure-functions-java-library` 套件中定義。 Azure Functions 執行階段會嘗試將收到的輸入轉換為您的程式碼要求的類型。

### <a name="strings"></a>字串

如果函式的對應輸入參數類型是 `String`，傳遞至函式方法的值會轉換成字串。 

### <a name="plain-old-java-objects-pojos"></a>純舊 Java 物件 (POJO)

如果函式的輸入簽章應該為 Java 類型，則使用 JSON 格式化的字串會轉換為 Java 類型。 這項轉換可讓您傳入 JSON，並與 Java 類型搭配使用。

作為函式輸入的 POJO 類型必須具有與在其中使用之函式方法相同的 `public` 存取修飾詞。 您不需要宣告 POJO 類別欄位 `public`。 例如，JSON 字串 `{ "x": 3 }` 可以轉換成下列 POJO 類型：

```Java
public class MyData {
    private int x;
}
```

### <a name="binary-data"></a>二進位資料

二進位資料在 Azure 函式程式碼中會表示為 `byte[]`。 藉由將 function.json 中的 `dataType` 欄位設定為 `binary`，將二進位輸入或輸出繫結至您的函式：

```json
 {
  "scriptFile": "azure-functions-example.jar",
  "entryPoint": "com.example.MyClass.echo",
  "bindings": [
    {
      "type": "blob",
      "name": "content",
      "direction": "in",
      "dataType": "binary",
      "path": "container/myfile.bin",
      "connection": "ExampleStorageAccount"
    },
  ]
}
```

然後在您的函式程式碼中使用它：

```java
// Class definition and imports are omitted here
public static String echoLength(byte[] content) {
}
```

空的輸入值可以是 `null` 以作為函式引數，但潛在空白值的建議處理方式為使用 `Optional<T>`。


## <a name="function-method-overloading"></a>函式方法多載

您可以使用相同名稱但是不同類型，多載函式方法。 例如，您可以在類別中同時具有 `String echo(String s)` 和 `String echo(MyType s)`。 Azure Functions 會根據輸入類型 (若為 HTTP 輸入，MIME 類型 `text/plain` 會產生 `String`，`application/json` 則代表 `MyType`) 決定要叫用的方法。

## <a name="inputs"></a>輸入

在 Azure Functions 中輸入會分成兩個類別：一個是觸發程序輸入，另一個是額外的輸入。 雖然它們在 `function.json` 中是不同的，在 Java 程式碼中的使用方式則相同。 讓我們以下列程式碼片段為例：

```java
package com.example;

import com.microsoft.azure.functions.annotation.*;

public class MyClass {
    @FunctionName("echo")
    public static String echo(
        @HttpTrigger(name = "req", methods = { "put" }, authLevel = AuthorizationLevel.ANONYMOUS, route = "items/{id}") String in,
        @TableInput(name = "item", tableName = "items", partitionKey = "Example", rowKey = "{id}", connection = "AzureWebJobsStorage") MyObject obj
    ) {
        return "Hello, " + in + " and " + obj.getKey() + ".";
    }

    public static class MyObject {
        public String getKey() { return this.RowKey; }
        private String RowKey;
    }
}
```

此函式觸發時，HTTP 要求會透過 `String in` 傳遞至函式。 系統會根據路由 URL 中的識別碼從 Azure 資料表儲存體擷取輸入，並於函式主體中以 `obj` 形式提供使用。

## <a name="outputs"></a>輸出

輸出可以使用傳回值或輸出參數來表示。 如果只有一個輸出，建議您使用傳回值。 若為多個輸出，您必須使用輸出參數。

傳回值是最簡單形式的輸出，您只要傳回任何類型的值，Azure Functions 執行階段就會嘗試將它封送處理回實際類型 (例如 HTTP 回應)。  您可以將任何輸出註釋套用至函式方法 (註釋的名稱屬性必須為 $return)，以定義傳回值的輸出。

若要產生多個輸出值，請使用 `azure-functions-java-library` 套件中定義的 `OutputBinding<T>` 類型。 如果您需要進行 HTTP 回應，同時將訊息推送至佇列，您可以撰寫如下的程式碼：

例如，複製函式的 Blob 內容可定義為下列程式碼。 `@StorageAccount` 註釋在此會用來防止複製 `@BlobTrigger` 和 `@BlobOutput` 的連線屬性。

```java
package com.example;

import com.microsoft.azure.functions.annotation.*;

public class MyClass {
    @FunctionName("copy")
    @StorageAccount("AzureWebJobsStorage")
    @BlobOutput(name = "$return", path = "samples-output-java/{name}")
    public static String copy(@BlobTrigger(name = "blob", path = "samples-input-java/{name}") String content) {
        return content;
    }
}
```

請使用 `OutputBinding<byte[]`> 來產生二進位輸出值 (針對參數)；針對傳回值，請直接使用 `byte[]`。

## <a name="specialized-types"></a>特殊類型

有時候函式必須對輸入和輸出有細微控制權。 在 `azure-functions-java-core` 套件中為您提供特殊類型，來管理要求資訊以及訂製 HTTP 觸發程序的傳回狀態：

| 特殊類型      |       目標        | 一般使用方式                  |
| --------------------- | :-----------------: | ------------------------------ |
| `HttpRequestMessage<T>`  |    HTTP 觸發程序     | 取得方法、標頭或查詢 |
| `HttpResponseMessage<T>` | HTTP 輸出繫結 | 傳回 200 以外的狀態   |

> [!NOTE] 
> 您也可以使用 `@BindingName` 註釋來取得 HTTP 標頭和查詢。 例如，`@BindingName("name") String query` 會逐一查看 HTTP 要求標頭和查詢，並將該值傳遞至方法。 例如，如果要求 URL 是 `http://example.org/api/echo?name=test`，則 `query` 是 `"test"`。

### <a name="metadata"></a>中繼資料

中繼資料會來自不同的來源，例如 HTTP 標頭、HTTP 查詢及[觸發程序中繼資料](/azure/azure-functions/functions-triggers-bindings#trigger-metadata-properties)。 請使用 `@BindingName` 註解搭配中繼資料名稱來取得值。

例如，如果要求的 URL 是 `http://{example.host}/api/metadata?name=test`，則下列程式碼片段中的 `queryValue` 將會是 `"test"`。

```Java
package com.example;

import java.util.Optional;
import com.microsoft.azure.functions.annotation.*;


public class MyClass {
    @FunctionName("metadata")
    public static String metadata(
        @HttpTrigger(name = "req", methods = { "get", "post" }, authLevel = AuthorizationLevel.ANONYMOUS) Optional<String> body,
        @BindingName("name") String queryValue
    ) {
        return body.orElse(queryValue);
    }
}
```

## <a name="execution-context"></a>執行內容

透過 `azure-functions-java-library` 套件中定義的 `ExecutionContext` 物件，與 Azure Functions 執行環境互動。 使用 `ExecutionContext` 物件以在您的程式碼中使用引動資訊和函式執行階段資訊。

### <a name="custom-logging"></a>自訂記錄

可以透過 `ExecutionContext` 物件存取函式執行階段記錄器。 這個記錄器會繫結至 Azure 監視，並可讓您將函式執行期間遇到的警告和錯誤加上旗標。

下列範例程式碼會在收到的要求主體是空白時，記錄一則警告訊息。

```java

import com.microsoft.azure.functions.*;
import com.microsoft.azure.functions.annotation.*;

public class Function {
    public String echo(@HttpTrigger(name = "req", methods = {"post"}, authLevel = AuthorizationLevel.ANONYMOUS) String req, ExecutionContext context) {
        if (req.isEmpty()) {
            context.getLogger().warning("Empty request body received by function " + context.getFunctionName() + " with invocation " + context.getInvocationId());
        }
        return String.format(req);
    }
}
```

## <a name="view-logs-and-trace"></a>檢視記錄檔和追蹤

您可以使用 Azure CLI 以串流 Java 標準流出與錯誤記錄檔，以及其他應用程式記錄檔。 首先，設定您的函式應用程式以使用 Azure CLI 寫入應用程式記錄：

```azurecli-interactive
az webapp log config --name functionname --resource-group myResourceGroup --application-logging true
```

若要使用 Azure CLI 串流 Function 應用程式的記錄輸出，請開啟新的命令列提示字元、Bash 或終端機工作階段，然後輸入下列命令：

```azurecli-interactive
az webapp log tail --name webappname --resource-group myResourceGroup
```
[az webapp log tail](/cli/azure/webapp/log) 命令有選項可使用 `--provider` 選項篩選輸出。 

若要使用 Azure CLI 將記錄檔做為單一 ZIP 檔案下載，請開啟新的命令列提示字元、Bash 或終端機工作階段，然後輸入下列命令：

```azurecli-interactive
az webapp log download --resource-group resourcegroupname --name functionappname
```

您必須先在 Azure 入口網站或 Azure CLI 中啟用檔案系統記錄，再執行此命令。

## <a name="environment-variables"></a>環境變數

為確保安全，請將祕密資訊 (例如金鑰或權杖) 保存在原始程式碼以外的地方。 藉由從環境變數讀取金鑰和權杖，以在函式程式碼中使用。

若要在於本機執行 Azure Functions 時設定環境變數，您可選擇將這些變數加入到 local.settings.json 檔案中。 如果函式專案根目錄中沒有該檔案，則可建立檔案。 該檔案看起來應該會像這樣：

```xml
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "AzureWebJobsDashboard": ""
  }
}
```

`values` 對應中的每個機碼/值對應會在執行階段時做為環境變數，可透過呼叫 `System.getenv("<keyname>")` 來存取，例如 `System.getenv("AzureWebJobsStorage")`。 可接受加入其他機碼/值組，而且是建議的做法。

> [!NOTE]
> 如果採取這種方式，請務必要將 local.settings.json 檔案新增至存放庫忽略檔案，讓系統不要認可該檔案。

由於您的程式碼現在仰賴這些環境變數，您現在可以登入 Azure 入口網站，在函數應用程式設定中設定相同的機碼/值組，如此程式碼功能在本機測試及部署至 Azure 時都會相同。

## <a name="next-steps"></a>後續步驟

如需 Azure Function Java 開發的詳細資訊，請參閱以下資源：

* [Azure Functions 的最佳做法](functions-best-practices.md)
* [Azure Functions 開發人員參考](functions-reference.md)
* [Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
- 使用 [Visual Studio Code](https://code.visualstudio.com/docs/java/java-azurefunctions)[IntelliJ](functions-create-maven-intellij.md) 和 [Eclipse](functions-create-maven-eclipse.md) 進行本機開發與偵錯。 
* [使用 Visual Studio Code 針對 Java Azure Functions 進行遠端偵錯](https://code.visualstudio.com/docs/java/java-serverless#_remote-debug-functions-running-in-the-cloud)
* [Azure Functions 適用的 Maven 外掛程式](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-functions-maven-plugin/README.md) (英文) - 透過 `azure-functions:add` 目標建立串流函式，並針對 [ZIP 檔案開發](deployment-zip-push.md)準備暫存目錄。
