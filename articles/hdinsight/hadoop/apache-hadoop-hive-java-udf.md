---
title: 在 HDInsight 中搭配使用 Java 使用者定義函式 (UDF) 和 Hive - Azure
description: 了解如何建立能配合 Hive 使用的以 Java 為基礎的使用者定義函式 (UDF)。 此範例 UDF 會將文字字串的資料表轉換成小寫。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: b2a4c7ce3ac91ade497ca59a8c2ca4fe642811a9
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047498"
---
# <a name="use-a-java-udf-with-hive-in-hdinsight"></a>在 HDInsight 中搭配使用 Java UDF 和 Hive

了解如何建立能配合 Hive 使用的以 Java 為基礎的使用者定義函式 (UDF)。 此範例中的 Java UDF 會將文字字串的資料表轉換成全部小寫。

## <a name="requirements"></a>需求

* HDInsight 叢集 

    > [!IMPORTANT]
    > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

    本文件中的大部分步驟適用於以 Windows 和 Linux 為基礎的叢集。 不過，用來將編譯之 UDF 上傳到叢集並予以執行的步驟僅供以 Linux 為基礎的叢集專用。 對於適用於 Windows 型叢集的資訊，本文件備有相關連結。

* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) 8 或更新版本 (或同等功能版本，例如 OpenJDK)

* [Apache Maven](http://maven.apache.org/)

* 文字編輯器或 Java IDE

    > [!IMPORTANT]
    > 如果您是在 Windows 用戶端上建立 Python 檔案，就必須使用以 LF 做為行尾結束符號的編輯器。 如果您不確定編輯器是使用 LF 或 CRLF，請參閱[疑難排解](#troubleshooting)一節，以了解有關移除 CR 字元的步驟。

## <a name="create-an-example-java-udf"></a>建立範例 Java UDF 

1. 在命令列中，使用下列命令來建立新的 Maven 專案：

    ```bash
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=ExampleUDF -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

   > [!NOTE]
   > 如果您使用 PowerShell，必須將參數放置在引號內。 例如： `mvn archetype:generate "-DgroupId=com.microsoft.examples" "-DartifactId=ExampleUDF" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DinteractiveMode=false"`。

    此命令會建立名為 **exampleudf** 的目錄，其包含 Maven 專案。

2. 建立專案後，請刪除隨專案一同建立的 **exampleudf/src/test** 目錄。

3. 開啟 **exampleudf/pom.xml**，以下列 XML 取代現有的 `<dependencies>` 項目︰

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.3</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    ```

    這些項目能指定 HDInsight 3.6 隨附之 Hadoop 和 Hive 的版本。 在 [HDInsight 元件版本設定](../hdinsight-component-versioning.md) 文件中，您可以找到有關 HDInsight 隨附之 Hadoop 和 Hive 的版本資訊。

    在檔案結尾處 `</project>` 之前新增 `<build>` 區段。 此區段應該包含下列 XML：

    ```xml
    <build>
        <plugins>
            <!-- build for Java 1.8. This is required by HDInsight 3.6  -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!-- build an uber jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <!-- Keep us from getting a can't overwrite file error -->
                    <transformers>
                        <transformer
                                implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                        </transformer>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer">
                        </transformer>
                    </transformers>
                    <!-- Keep us from getting a bad signature error -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ```

    這些項目會定義如何建置專案。 具體來說，定義專案使用的 Java 版本以及如何建置 uberjar 以部署至叢集。

    完成變更後，儲存檔案。

4. 將 **exampleudf/src/main/java/com/microsoft/examples/App.java** 重新命名為 **ExampleUDF.java**，然後在編輯器中開啟檔案。

5. 以下文中的內容取代 **ExampleUDF.java** 檔案的內容，然後儲存檔案。

    ```java
    package com.microsoft.examples;

    import org.apache.hadoop.hive.ql.exec.Description;
    import org.apache.hadoop.hive.ql.exec.UDF;
    import org.apache.hadoop.io.*;

    // Description of the UDF
    @Description(
        name="ExampleUDF",
        value="returns a lower case version of the input string.",
        extended="select ExampleUDF(deviceplatform) from hivesampletable limit 10;"
    )
    public class ExampleUDF extends UDF {
        // Accept a string input
        public String evaluate(String input) {
            // If the value is null, return a null
            if(input == null)
                return null;
            // Lowercase the input string and return it
            return input.toLowerCase();
        }
    }
    ```

    此程式碼會實作接受字串值並傳回小寫版本之字串的 UDF。

## <a name="build-and-install-the-udf"></a>建置及安裝 UDF

1. 使用下列命令來編譯及封裝 UDF：

    ```bash
    mvn compile package
    ```

    此命令會建置 UDF 並將它封裝到 `exampleudf/target/ExampleUDF-1.0-SNAPSHOT.jar` 檔案。

2. 使用 `scp` 命令將檔案複製到 HDInsight 叢集。

    ```bash
    scp ./target/ExampleUDF-1.0-SNAPSHOT.jar myuser@mycluster-ssh.azurehdinsight
    ```

    將 `myuser` 取代為叢集的 SSH 使用者帳戶。 將 `mycluster` 取代為叢集名稱。 如果您使用密碼來保護 SSH 帳戶，系統會提示您輸入密碼。 如果您使用憑證，可能需要使用 `-i` 參數來指定私密金鑰檔案。

3. 使用 SSH 連線到叢集。

    ```bash
    ssh myuser@mycluster-ssh.azurehdinsight.net
    ```

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

4. 在 SSH 工作階段中，將 jar 檔案複製到 HDInsight 儲存體。

    ```bash
    hdfs dfs -put ExampleUDF-1.0-SNAPSHOT.jar /example/jars
    ```

## <a name="use-the-udf-from-hive"></a>從 Hive 使用 UDF

1. 在 SSH 工作階段中，使用以下命令來啟動 Beeline 用戶端。

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

    此命令假設您使用預設值 **admin** 來當做叢集的登入帳戶。

2. 一旦到達 `jdbc:hive2://localhost:10001/>` 提示後，請輸入下列命令以將 UDF 新增至 Hive，並將它公開為函式。

    ```hiveql
    ADD JAR wasb:///example/jars/ExampleUDF-1.0-SNAPSHOT.jar;
    CREATE TEMPORARY FUNCTION tolower as 'com.microsoft.examples.ExampleUDF';
    ```

    > [!NOTE]
    > 此範例假設 Azure 儲存體是叢集的預設儲存體。 如果您的叢集改為使用 Data Lake Store，請將 `wasb:///` 值變更為 `adl:///`。

3. 使用 UDF 將從資料表擷取的值轉換成小寫字串。

    ```hiveql
    SELECT tolower(deviceplatform) FROM hivesampletable LIMIT 10;
    ```

    此查詢會從資料表選取裝置平台 (Android、Windows、iOS 等)、將字串轉換為小寫，然後再加以顯示。 此輸出看起來類似下列文字：

        +----------+--+
        |   _c0    |
        +----------+--+
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        +----------+--+

## <a name="next-steps"></a>後續步驟

如需 Hive 的其他操作方法，請參閱 [搭配 HDInsight 使用 Hive](hdinsight-use-hive.md)。

如需 Hive 使用者定義函式的詳細資訊，請造訪 apache.org 並參閱 Hive wiki 的 [Hive 運算子和使用者定義函式](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF) 一節。
