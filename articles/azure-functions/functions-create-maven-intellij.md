---
title: 使用 Java 和 IntelliJ 建立 Azure 函數應用程式 | Microsoft Docs
description: 使用 Java 和 IntelliJ 建立 HTTP 觸發的簡單無伺服器應用程式並發佈到 Azure Functions 的操作說明指南。
services: functions
documentationcenter: na
author: jeffhollan
manager: jpconnock
keywords: azure functions, 函式, 事件處理, 計算, 無伺服器架構, java
ms.service: functions
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 07/01/2018
ms.author: jehollan
ms.custom: mvc, devcenter
ms.openlocfilehash: 317d8bf40ef152e5b7dae6406be29330feaaa8d3
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43842465"
---
# <a name="create-your-first-function-with-java-and-intellij-preview"></a>使用 Java 和 IntelliJ 建立您的第一個函式 (預覽)

> [!NOTE] 
> 適用於 Azure Functions 的 Java 目前為預覽版。

本文示範如何使用 IntelliJ IDEA 和 Apache Maven 建立[無伺服器](https://azure.microsoft.com/overview/serverless-computing/)函式專案、從 IDE 在您自己的電腦上測試此專案並加以偵錯，然後將它部署至 Azure Functions。 

<!-- TODO ![Access a Hello World function from the command line with cURL](media/functions-create-java-maven/hello-azure.png) -->

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>設定開發環境

若要使用 Java 和 IntelliJ 開發函數應用程式，您必須安裝下列項目：

-  [Java Developer Kit](https://www.azul.com/downloads/zulu/)第 8 版。
-  [Apache Maven](https://maven.apache.org) 3.0 版或更高版本。
-  [IntelliJ IDEA](https://www.jetbrains.com/idea/download) Community 或 Ultimate (含 Maven 工具)。
-  [Azure CLI](https://docs.microsoft.com/cli/azure)

> [!IMPORTANT] 
> JAVA_HOME 環境變數必須設定為 JDK 的安裝位置，才能完成本快速入門。

強烈建議您也要安裝 [Azure Functions Core Tools 第 2 版](functions-run-local.md#v2)，其提供用於撰寫、執行和偵錯 Azure Functions 的本機開發環境。 


## <a name="create-a-functions-project"></a>建立 Functions 專案

1. 在 IntelliJ IDEA 中，按一下 [Create New Project] \(建立新專案\)。  
1. 選取從 [Maven] 建立。
1. 選取 [Create from archetype] \(從 Archetype 建立\) 的核取方塊，並針對 [azure-functions-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype) 選取 [Add Archetype] \(新增 Archetype\)。
    - GroupId：com.microsoft.azure
    - ArtifactId：azure-functions-archetype
    - 版本：使用[中央存放庫](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype) \(英文\) 所提供的最新版本 
    ![IntelliJ Maven 建立](media/functions-create-first-java-intellij/functions-create-intellij.png)  
1. 按一下 [Next] \(下一步\)、輸入目前專案的詳細資料，最終按一下 [Finish] \(完成\)。

Maven 會以 _artifactId_ 名稱在新的資料夾中建立專案檔。 專案中產生的程式碼是 [HTTP 觸發](/azure/azure-functions/functions-bindings-http-webhook)的簡單函式，此函式會回應觸發 HTTP 要求的本文。

## <a name="run-functions-locally-in-the-ide"></a>在 IDE 中本機執行函式

> [!NOTE]
> [Azure Functions Core Tools 第 2 版](functions-run-local.md#v2)必須安裝，才能在本機執行和偵錯函式。

1. 選取以匯入變更，或確定[自動匯入](https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html) \(英文\) 已啟用。
1. 開啟 [Maven Projects] \(Maven 專案\) 工具列。
1. 在 [Lifecycle] \(生命週期\) 下方，按兩下 [package] \(封裝\) 以封裝並建置解決方案，然後建立目標目錄。
1. 在 [Plugins] \(外掛程式\) -> [azure-functions] 下方，按兩下 [azure-functions:run] 以啟動 Azure Functions 本機執行階段。  
  ![適用於 Azure Functions 的 Maven 工具列](media/functions-create-first-java-intellij/functions-intellij-java-maven-toolbar.png)  

當您完成測試函式時，關閉執行對話方塊。 一次只能有一個函式主機是作用中且在本機執行。

### <a name="debug-the-function-in-intellij"></a>對 IntelliJ 中的函式進行偵錯
若要以偵錯模式啟動函式主機，請新增 **-DenableDebug** 作為您執行函式時的引數。 您可以在終端機執行下列命令列，或在[maven goals](https://www.jetbrains.com/help/idea/maven-support.html#run_goal) 中加以設定。 然後函式主機就會開啟 5005 偵錯連接埠。 

```
mvn azure-functions:run -DenableDebug
```

若要在 IntelliJ 中偵錯，請在 [執行] 功能表中選取 [編輯組態]。 按一下 **+** 以新增 [遠端]。 填妥 [名稱] 和 [設定]，然後按一下 [確定] 以儲存組態。 安裝之後，按一下 [偵錯]「您的遠端組態名稱」或按 **Shift+F9** 開始偵錯。

![在 IntelliJ 中進行函式偵錯](media/functions-create-first-java-intellij/debug-configuration-intellij.PNG)

完成時，停止偵錯工具和執行中處理序。 一次只能有一個函式主機是作用中且在本機執行。

## <a name="deploy-the-function-to-azure"></a>將函式部署到 Azure

部署到 Azure Functions 的程序從 Azure CLI 使用帳戶認證。 [使用 Azure CLI 登入](/cli/azure/authenticate-azure-cli?view=azure-cli-latest)之後，才能繼續使用電腦的命令提示字元。

```azurecli
az login
```

使用 `azure-functions:deploy` Maven 目標，或在 [Maven 專案] 視窗中選取 azure-functions:deploy 選項，藉以將您的程式碼部署到新的函數應用程式。

```
mvn azure-functions:deploy
```

部署完成時，您會看到可用來存取 Azure 函式應用程式的 URL：

```output
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
```

## <a name="next-steps"></a>後續步驟

- 檢閱 [Java 函式開發人員指南](functions-reference-java.md)以了解開發 Java 函式的詳細資訊。
- 使用 `azure-functions:add`Maven 目標，將具有不同觸發程序的其他函式新增至您的專案。
