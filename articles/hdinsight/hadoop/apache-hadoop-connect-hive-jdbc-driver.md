---
title: 透過 JDBC 驅動程式查詢 Hive - Azure HDInsight
description: 從 Java 應用程式使用 JDBC 驅動程式，將 Hive 查詢提交到 HDInsight 上的 Hadoop。 以程式設計方式連接和透過 SQuirrel SQL 用戶端連接。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 04/02/2018
ms.author: jasonh
ms.openlocfilehash: da2b3484f80f7116664cf5a25c7de99da723f202
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041687"
---
# <a name="query-hive-through-the-jdbc-driver-in-hdinsight"></a>在 HDInsight 中透過 JDBC 驅動程式查詢 Hive

[!INCLUDE [ODBC-JDBC-selector](../../../includes/hdinsight-selector-odbc-jdbc.md)]

了解如何從 Java 應用程式使用 JDBC 驅動程式將 Hive 查詢提交到 Azure HDInsight 中的 Hadoop。 本文件中的資訊會示範如何以程式設計方式以及如何從 SQuirrel SQL 用戶端連線。

如需有關 Hive JDBC 介面的詳細資訊，請參閱 [HiveJDBCInterface](https://cwiki.apache.org/confluence/display/Hive/HiveJDBCInterface)。

## <a name="prerequisites"></a>必要條件

* HDInsight 叢集上的 Hadoop。

  > [!IMPORTANT]
  > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 3.3 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* [SQuirreL SQL](http://squirrel-sql.sourceforge.net/)。 SQuirreL 是 JDBC 用戶端應用程式。

* [Java Developer Kit (JDK) 第 7 版](https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 或更高版本。

* [Apache Maven](https://maven.apache.org)。 Maven 是 Java 專案的專案建置系統，由與本文相關聯的專案所使用。

## <a name="jdbc-connection-string"></a>JDBC 連接字串

透過 443 在 Azure 上建立 HDInsight 叢集的 JDBC 連接，並使用 SSL 保護流量。 背後有叢集的公用閘道器會將流量重新導向至 HiveServer2 實際接聽的連接埠。 下列連接字串會顯示用於 HDInsight 的格式：

    jdbc:hive2://CLUSTERNAME.azurehdinsight.net:443/default;transportMode=http;ssl=true;httpPath=/hive2

將 `CLUSTERNAME` 替換為 HDInsight 叢集的名稱。

## <a name="authentication"></a>驗證

建立連線時，您必須使用 HDInsight 叢集系統管理員名稱和密碼來通過叢集閘道器驗證。 從 SQuirreL SQL 之類的 JDBC 用戶端連接時，您必須在用戶端設定中輸入系統管理員名稱和密碼。

從 Java 應用程式建立連接時，您必須使用該名稱和密碼。 例如，下列 Java 程式碼會使用連接字串、系統管理員名稱和密碼開啟新的連接：

```java
DriverManager.getConnection(connectionString,clusterAdmin,clusterPassword);
```

## <a name="connect-with-squirrel-sql-client"></a>使用 SQuirreL SQL 用戶端連接

SQuirreL SQL 是可用來從遠端以 HDInsight 叢集執行 Hive 查詢的 JDBC 用戶端。 下列步驟假設您已安裝 SQuirreL SQL。

1. 建立包含檔案的目錄。 例如： `mkdir hivedriver`。

2. 從命令列中，使用下列命令從 HDInsight 叢集將檔案進行複製：

    ```bash
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hadoop-client/hadoop-common.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hadoop-client/hadoop-auth.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hadoop-client/lib/log4j-*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hadoop-client/lib/slf4j-*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/hive-*-1.2*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/httpclient-*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/httpcore-*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/libthrift-*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/libfb*.jar .
    scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/commons-logging-*.jar .
    ```

    將 `USERNAME` 取代為叢集的 SSH 使用者帳戶名稱。 將 `CLUSTERNAME` 取代為 HDInsight 叢集名稱。

3. 啟動 SQuirreL SQL 應用程式。 從視窗左側選取 [驅動程式]。

    ![視窗左側的 [驅動程式] 索引標籤](./media/apache-hadoop-connect-hive-jdbc-driver/squirreldrivers.png)

4. 從 [驅動程式] 對話方塊上方的圖示，選取 [+] 圖示以建立驅動程式。

    ![[驅動程式] 圖示](./media/apache-hadoop-connect-hive-jdbc-driver/driversicons.png)

5. 在 [新增驅動程式] 對話方塊中，新增下列資訊：

    * **名稱**：Hive
    * **範例 URL**：`jdbc:hive2://localhost:443/default;transportMode=http;ssl=true;httpPath=/hive2`
    * **額外類別路徑**︰使用 [新增] 按鈕來新增稍早下載的所有 jar 檔案
    * **類別名稱**：org.apache.hive.jdbc.HiveDriver

   ![新增驅動程式對話方塊](./media/apache-hadoop-connect-hive-jdbc-driver/adddriver.png)

   按一下 [確定] 儲存這些變更。

6. 在 SQuirreL SQL 視窗的左側選取 [別名]。 然後按一下 [+] 圖示來建立連線別名。

    ![新增別名](./media/apache-hadoop-connect-hive-jdbc-driver/aliases.png)

7. 在 [新增別名] 對話方塊中使用下列值。

    * **名稱**：HDInsight 上的 Hive

    * **驅動程式**︰使用下拉式清單來選取 [Hive] 驅動程式

    * **URL**：`jdbc:hive2://CLUSTERNAME.azurehdinsight.net:443/default;transportMode=http;ssl=true;httpPath=/hive2`

        將 **CLUSTERNAME** 取代為 HDInsight 叢集的名稱。

    * **使用者名稱**︰HDInsight 叢集的叢集登入帳戶名稱。 預設值為 `admin`。

    * **密碼**︰叢集登入帳戶的密碼。

 ![[新增別名] 對話方塊](./media/apache-hadoop-connect-hive-jdbc-driver/addalias.png)

    > [!IMPORTANT] 
    > 使用 [測試] 按鈕來確認連接能正常運作。 當 [連接到︰HDInsight 上的 Hive] 對話方塊出現時，請選取 [連接] 來執行測試。 如果測試成功，您會看到 [連線成功] 對話方塊。 如果發生錯誤，請參閱[疑難排解](#troubleshooting)。

    若要儲存連線別名，請使用 [新增別名] 對話方塊底部的 [確定] 按鈕。

8. 從 SQuirreL SQL 頂端的 [連接到] 下拉式清單選取 [HDInsight 上的 Hive]。 出現提示時，請選取 [連接]。

    ![連接對話方塊](./media/apache-hadoop-connect-hive-jdbc-driver/connect.png)

9. 連接後，請在 SQL 查詢對話方塊中輸入下列查詢，然後選取 [執行] 圖示。 結果區域應該會顯示查詢的結果。

        select * from hivesampletable limit 10;

    ![sql 查詢對話方塊，包括結果](./media/apache-hadoop-connect-hive-jdbc-driver/sqlquery.png)

## <a name="connect-from-an-example-java-application"></a>從範例 Java 應用程式連接

您可以在 [https://github.com/Azure-Samples/hdinsight-java-hive-jdbc](https://github.com/Azure-Samples/hdinsight-java-hive-jdbc) 上找到使用 Java 用戶端在 HDInsight 上查詢 Hive 的範例。 遵循儲存機制中的指示建置和執行範例。

## <a name="troubleshooting"></a>疑難排解

### <a name="unexpected-error-occurred-attempting-to-open-an-sql-connection"></a>嘗試開啟 SQL 連接時，發生意外的錯誤

**徵狀**︰連線到 HDInsight 叢集 3.3 版或更新版本時，您可能會收到發生非預期錯誤的錯誤。 此錯誤的堆疊追蹤開頭為下列幾行︰

```java
java.util.concurrent.ExecutionException: java.lang.RuntimeException: java.lang.NoSuchMethodError: org.apache.commons.codec.binary.Base64.<init>(I)V
at java.util.concurrent.FutureTas...(FutureTask.java:122)
at java.util.concurrent.FutureTask.get(FutureTask.java:206)
```

**原因**：此錯誤是由 SQuirreL 所隨附的舊版 commons-codec.jar 檔案所造成。

**解決方案**︰若要修正此錯誤，請使用下列步驟：

1. 從您的 HDInsight 叢集下載 commons-codec jar 檔案。

        scp USERNAME@CLUSTERNAME:/usr/hdp/current/hive-client/lib/commons-codec*.jar ./commons-codec.jar

2. 結束 SQuirreL，然後前往系統上安裝 SQuirreL 的目錄。 在 SquirreL 目錄的 `lib` 目錄下，使用從 HDInsight 叢集下載的版本來取代現有的 commons-codec.jar。

3. 重新啟動 SQuirreL。 連接到 HDInsight 上的 Hive 時應該不會再出現此錯誤。

## <a name="next-steps"></a>後續步驟

現在您已學會如何搭配 Hive 使用 JDBC，接著請使用下列連結來探索 Azure HDInsight 的其他使用方式。

* [在 Azure HDInsight 中使用 Microsoft Power BI 將 Hive 資料視覺化](apache-hadoop-connect-hive-power-bi.md)。
* [在 Azure HDInsight 中使用 Power BI 將互動式查詢 Hive 資料視覺化](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)。
* [使用 Zeppelin 在 Azure HDInsight 中執行 Hive 查詢](./../hdinsight-connect-hive-zeppelin.md)。
* [使用 Microsoft Hive ODBC 驅動程式將 Excel 連線到 HDInsight](apache-hadoop-connect-excel-hive-odbc-driver.md)。
* [使用 Power Query 將 Excel 連線到 Hadoop](apache-hadoop-connect-excel-power-query.md)。
* [使用 Data Lake Tools for Visual Studio 連線至 Azure HDInsight 及執行 Hive 查詢](apache-hadoop-visual-studio-tools-get-started.md)。
* [使用適用於 Visual Studio Code 的 Azure HDInsight 工具](../hdinsight-for-vscode.md)。
* [將資料上傳至 HDInsight](../hdinsight-upload-data.md)
* [〈搭配 HDInsight 使用 Hivet〉](hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)
* [搭配 HDInsight 使用 MapReduce 工作](hdinsight-use-mapreduce.md)
