---
title: 使用者驗證︰使用 Java 利用 Azure Active Directory 向 Azure Data Lake Storage Gen1 進行 | Microsoft Docs
description: 了解如何使用 Java 透過 Azure Active Directory 向 Azure Data Lake Storage Gen1 完成使用者驗證
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 47b975b3ea0cfa9d2fb2536236b0a8dfaef14503
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46126933"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-java"></a>使用 Java 向 Azure Data Lake Storage Gen1 進行使用者驗證
> [!div class="op_single_selector"]
> * [使用 Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [使用 .NET SDK](data-lake-store-end-user-authenticate-net-sdk.md)
> * [使用 Python](data-lake-store-end-user-authenticate-python.md)
> * [使用 REST API](data-lake-store-end-user-authenticate-rest-api.md)
> 
>   

在本文中，您會了解如何使用 Java SDK 向 Azure Data Lake Storage Gen1 進行使用者驗證。 如需使用 Java SDK 向 Data Lake Storage Gen1 進行服務對服務驗證，請參閱[使用 Java 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-java.md)。

## <a name="prerequisites"></a>必要條件
* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **建立 Active Directory 「原生」應用程式**。 您必須先完成[使用 Azure Active Directory 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)中的步驟。

* [Maven](https://maven.apache.org/install.html). 本教學課程使用 Maven 來處理組建和專案相依性。 雖有可能不使用 Maven 或 Gradle 等組建系統進行建置，但這些系統讓相依性管理變得輕鬆許多。

* (選擇性) [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) 或 [Eclipse](https://www.eclipse.org/downloads/) 之類的 IDE。

## <a name="end-user-authentication"></a>使用者驗證
1. 從命令列或透過 IDE，使用 [mvn 原型](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)建立 Maven 專案。 如需有關如何使用 IntelliJ 建立 Java 專案的指示，請參閱[這裡](https://www.jetbrains.com/help/idea/2016.1/creating-and-running-your-first-java-application.html)。 如需有關如何使用 Eclipse 建立專案的指示，請參閱[這裡](http://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm)。

2. 將下列相依性新增至 Maven **pom.xml** 檔案。 將下列程式碼片段新增至 **\</project>** 標記之前：
   
        <dependencies>
          <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>azure-data-lake-store-sdk</artifactId>
            <version>2.2.3</version>
          </dependency>
          <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-nop</artifactId>
            <version>1.7.21</version>
          </dependency>
        </dependencies>
   
    第一個相依性是使用來自 maven 儲存機制的 Data Lake Storage Gen1 SDK (`azure-data-lake-store-sdk`)。 第二個相依性是指定要用於此應用程式的記錄架構 (`slf4j-nop`)。 Data Lake Storage Gen1 SDK 會使用 [slf4j](http://www.slf4j.org/) 記錄外觀，讓您從數個熱門的記錄架構中進行選擇，例如 log4j、Java 記錄、logback 等或不記錄。 在此範例中，我們停用記錄，因此會使用 **slf4j-nop** 繫結。 若要在應用程式中使用其他記錄選項，請參閱[這裡](http://www.slf4j.org/manual.html#projectDep)。

3. 在應用程式中新增下列 import 陳述式。

        import com.microsoft.azure.datalake.store.ADLException;
        import com.microsoft.azure.datalake.store.ADLStoreClient;
        import com.microsoft.azure.datalake.store.DirectoryEntry;
        import com.microsoft.azure.datalake.store.IfExists;
        import com.microsoft.azure.datalake.store.oauth2.AccessTokenProvider;
        import com.microsoft.azure.datalake.store.oauth2.DeviceCodeTokenProvider;

4. 在 Java 應用程式中使用下列程式碼片段，來為您稍早使用 `DeviceCodeTokenProvider` 建立的 Active Directory 原生應用程式取得權杖。 以 Azure Active Directory 原生應用程式的實際值取代 **FILL-IN-HERE**。

        private static String nativeAppId = "FILL-IN-HERE";
            
        AccessTokenProvider provider = new DeviceCodeTokenProvider(nativeAppId);   

Data Lake Storage Gen1 SDK 提供簡便的方法，讓您管理與 Data Lake Storage Gen1 帳戶互動所需的安全性權杖。 不過，SDK 不會要求只能使用這些方法。 您也可以使用任何其他方法來取得權杖，像是使用 [Azure Active Directory SDK](https://github.com/AzureAD/azure-activedirectory-library-for-java)，或您自己的自訂程式碼。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Java SDK，使用使用者驗證向 Azure Data Lake Storage Gen1 進行驗證。 您現在可以看看下列文章，了解如何配合使用 Java SDK 與 Azure Data Lake Storage Gen1。

* [使用 Java SDK 對 Data Lake Storage Gen1 進行資料作業](data-lake-store-get-started-java-sdk.md)


