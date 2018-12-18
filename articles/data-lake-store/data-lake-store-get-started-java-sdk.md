---
title: Java SDK：Azure Data Lake Storage Gen1 的檔案系統作業 | Microsoft Docs
description: 使用 Azure Data Lake Storage Gen1 的 Java SDK 在 Data Lake Storage Gen1 上執行檔案系統作業，例如建立資料夾等。
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: d10e09db-5232-4e84-bb50-52efc2c21887
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: a91593c1bae3002be8b7e423b627f1baf19e86e2
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46129178"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-java-sdk"></a>使用 Java SDK 在 Azure Data Lake Storage Gen1 上進行檔案系統作業
> [!div class="op_single_selector"]
> * [.NET SDK](data-lake-store-data-operations-net-sdk.md)
> * [Java SDK](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

了解如何使用 Azure Data Lake Storage Gen1 Java SDK 來執行基本作業，例如建立資料夾、上傳和下載資料檔案等等。如需 Data Lake Storage Gen1 的詳細資訊，請參閱 [Azure Data Lake Storage Gen1](data-lake-store-overview.md)。

您可以在 [Azure Data Lake Storage Gen1 Java API 文件](https://azure.github.io/azure-data-lake-store-java/javadoc/)取用適用於 Data Lake Storage Gen1 的 Java SDK API 文件。

## <a name="prerequisites"></a>必要條件
* Java Development Kit (JDK 7 或更新版本，使用 Java 1.7 版或更新版本)
* Data Lake Storage Gen1 帳戶。 請遵循[利用 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示。
* [Maven](https://maven.apache.org/install.html). 本教學課程使用 Maven 來處理組建和專案相依性。 雖有可能不使用 Maven 或 Gradle 等組建系統進行建置，但這些系統讓相依性管理變得輕鬆許多。
* (選擇性) [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) 或 [Eclipse](https://www.eclipse.org/downloads/) 之類的 IDE。

## <a name="create-a-java-application"></a>建立 Java 應用程式
[GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/) 上可用的程式碼範例會逐步引導您完成在存放區中建立檔案，串連檔案、下載檔案，以及刪除存放區中一些檔案的程序。 本節文章會逐步解說程式碼的主要部分。

1. 從命令列或透過 IDE，使用 [mvn 原型](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)建立 Maven 專案。 如需有關如何使用 IntelliJ 建立 Java 專案的指示，請參閱[這裡](https://www.jetbrains.com/help/idea/2016.1/creating-and-running-your-first-java-application.html)。 如需有關如何使用 Eclipse 建立專案的指示，請參閱[這裡](http://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm)。 

2. 將下列相依性新增至 Maven **pom.xml** 檔案。 將下列程式碼片段新增至 **\</project>** 標記之前：
   
        <dependencies>
          <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>azure-data-lake-store-sdk</artifactId>
            <version>2.1.5</version>
          </dependency>
          <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-nop</artifactId>
            <version>1.7.21</version>
          </dependency>
        </dependencies>
   
    第一個相依性是使用來自 maven 存放庫的 Data Lake Storage Gen1 SDK (`azure-data-lake-store-sdk`)。 第二個相依性是指定要用於此應用程式的記錄架構 (`slf4j-nop`)。 Data Lake Storage Gen1 SDK 會使用 [slf4j](http://www.slf4j.org/) 記錄外觀，讓您從數個常用的記錄架構中進行選擇，例如 log4j、Java 記錄、logback 等，或是不予記錄。 在此範例中，我們停用記錄，因此會使用 **slf4j-nop** 繫結。 若要在應用程式中使用其他記錄選項，請參閱[這裡](http://www.slf4j.org/manual.html#projectDep)。

3. 在應用程式中新增下列 import 陳述式。

        import com.microsoft.azure.datalake.store.ADLException;
        import com.microsoft.azure.datalake.store.ADLStoreClient;
        import com.microsoft.azure.datalake.store.DirectoryEntry;
        import com.microsoft.azure.datalake.store.IfExists;
        import com.microsoft.azure.datalake.store.oauth2.AccessTokenProvider;
        import com.microsoft.azure.datalake.store.oauth2.ClientCredsTokenProvider;

        import java.io.*;
        import java.util.Arrays;
        import java.util.List;

## <a name="authentication"></a>驗證

* 如需了解應用程式的終端使用者驗證，請參閱[使用 Python 向 Data Lake Storage Gen1 進行終端使用者驗證](data-lake-store-end-user-authenticate-java-sdk.md)。
* 如需讓應用程式進行服務對服務驗證，請參閱[使用 Java 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-java.md)。

## <a name="create-a-data-lake-storage-gen1-client"></a>建立 Data Lake Storage Gen1 用戶端
建立 [ADLStoreClient](https://azure.github.io/azure-data-lake-store-java/javadoc/) 物件時，您需要指定 Data Lake Storage Gen1 帳戶名稱以及您向 Data Lake Storage Gen1 驗證時 (請參閱[驗證](#authentication)一節) 產生的權杖提供者。 Data Lake Storage Gen1 帳戶名稱必須是完整的網域名稱。 例如，以 **mydatalakestoragegen1.azuredatalakestore.net** 這類內容取代 **FILL-IN-HERE**。

    private static String accountFQDN = "FILL-IN-HERE";  // full account FQDN, not just the account name
    ADLStoreClient client = ADLStoreClient.createClient(accountFQDN, provider);

下列各節的程式碼片段是一些常見的檔案系統作業範例。 如需了解其他作業，您可以參閱 **ADLStoreClient** 物件的完整 [Data Lake Storage Gen1 Java SDK API 文件](https://azure.github.io/azure-data-lake-store-java/javadoc/)。

## <a name="create-a-directory"></a>建立目錄

下列程式碼片段會在您指定的 Data Lake Storage Gen1 帳戶的根目錄中建立目錄結構。

    // create directory
    client.createDirectory("/a/b/w");
    System.out.println("Directory created.");

## <a name="create-a-file"></a>建立檔案

下列程式碼片段會在目錄結構中建立檔案 (c.txt)，並將一些資料寫入檔案。

    // create file and write some content
    String filename = "/a/b/c.txt";
    OutputStream stream = client.createFile(filename, IfExists.OVERWRITE  );
    PrintStream out = new PrintStream(stream);
    for (int i = 1; i <= 10; i++) {
        out.println("This is line #" + i);
        out.format("This is the same line (%d), but using formatted output. %n", i);
    }
    out.close();
    System.out.println("File created.");

您也可以使用位元組陣列建立檔案 (d.txt)。

    // create file using byte arrays
    stream = client.createFile("/a/b/d.txt", IfExists.OVERWRITE);
    byte[] buf = getSampleContent();
    stream.write(buf);
    stream.close();
    System.out.println("File created using byte array.");

先前程式碼片段中使用的 `getSampleContent` 函式定義，是 [GitHub 上](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/)可取得之範例的一部分。 

## <a name="append-to-a-file"></a>附加到檔案

下列程式碼片段會將內容附加到現有的檔案。

    // append to file
    stream = client.getAppendStream(filename);
    stream.write(getSampleContent());
    stream.close();
    System.out.println("File appended.");

先前程式碼片段中使用的 `getSampleContent` 函式定義，是 [GitHub 上](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/)可取得之範例的一部分。

## <a name="read-a-file"></a>讀取檔案

下列程式碼片段會讀取 Data Lake Storage Gen1 帳戶中檔案的內容。

    // Read File
    InputStream in = client.getReadStream(filename);
    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
    String line;
    while ( (line = reader.readLine()) != null) {
        System.out.println(line);
    }
    reader.close();
    System.out.println();
    System.out.println("File contents read.");

## <a name="concatenate-files"></a>串連檔案

下列程式碼片段會串連 Data Lake Storage Gen1 帳戶中的兩個檔案。 如果成功，串連後的檔案會取代兩個原有的檔案。

    // concatenate the two files into one
    List<String> fileList = Arrays.asList("/a/b/c.txt", "/a/b/d.txt");
    client.concatenateFiles("/a/b/f.txt", fileList);
    System.out.println("Two files concatenated into a new file.");

## <a name="rename-a-file"></a>重新命名檔案

下列程式碼片段會重新命名 Data Lake Storage Gen1 帳戶中的檔案。

    //rename the file
    client.rename("/a/b/f.txt", "/a/b/g.txt");
    System.out.println("New file renamed.");

## <a name="get-metadata-for-a-file"></a>取得檔案的中繼資料

下列程式碼片段會擷取 Data Lake Storage Gen1 帳戶中檔案的中繼資料。

    // get file metadata
    DirectoryEntry ent = client.getDirectoryEntry(filename);
    printDirectoryInfo(ent);
    System.out.println("File metadata retrieved.");

## <a name="set-permissions-on-a-file"></a>設定檔案權限

下列程式碼片段會替您在上一節建立的檔案設定權限。

    // set file permission
    client.setPermission(filename, "744");
    System.out.println("File permission set.");

## <a name="list-directory-contents"></a>列出目錄內容

下列程式碼片段會以遞迴方式列出目錄的內容。

    // list directory contents
    List<DirectoryEntry> list = client.enumerateDirectory("/a/b", 2000);
    System.out.println("Directory listing for directory /a/b:");
    for (DirectoryEntry entry : list) {
        printDirectoryInfo(entry);
    }
    System.out.println("Directory contents listed.");

先前程式碼片段中使用的 `printDirectoryInfo` 函式定義，是 [GitHub 上](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/)可取得之範例的一部分。

## <a name="delete-files-and-folders"></a>刪除檔案和資料夾

下列程式碼片段會以遞迴方式刪除 Data Lake Storage Gen1 帳戶中的指定檔案和資料夾。

    // delete directory along with all the subdirectories and files in it
    client.deleteRecursive("/a");
    System.out.println("All files and folders deleted recursively");
    promptEnterKey();

## <a name="build-and-run-the-application"></a>建置並執行應用程式
1. 若要從 IDE 執行，請找到並按下 [執行] 按鈕。 若要從 Maven 執行，請使用 [exec: exec](http://www.mojohaus.org/exec-maven-plugin/exec-mojo.html)。
2. 若要產生您可以從命令列執行的獨立 jar，請使用 [Maven 組件外掛程式](http://maven.apache.org/plugins/maven-assembly-plugin/usage.html)建置內含所有相依性的 jar。 [GitHub 上的範例原始程式碼](https://github.com/Azure-Samples/data-lake-store-java-upload-download-get-started/blob/master/pom.xml)中的 pom.xml 有範例。

## <a name="next-steps"></a>後續步驟
* [瀏覽 Java SDK 的 JavaDoc](https://azure.github.io/azure-data-lake-store-java/javadoc/)
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)


