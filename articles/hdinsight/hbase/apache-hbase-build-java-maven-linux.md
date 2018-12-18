---
title: Java HBase 用戶端 - Azure HDInsight
description: 了解如何使用 Apache Maven 建置以 Java 為基礎的 Apache HBase 應用程式，然後將它部署至 Azure HDInsight 上的 HBase。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/30/2018
ms.author: jasonh
ms.openlocfilehash: d198db98c07c01a56be118e74ce2bf4b29b8c757
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048427"
---
# <a name="build-java-applications-for-apache-hbase"></a>建置 Apache HBase 的 Java 應用程式

了解如何在 Java 中建立 [Apache HBase](http://hbase.apache.org/) 應用程式。 然後使用此應用程式搭配 Azure HDInsight 上的 HBase。

本文件中的步驟使用 [Maven](http://maven.apache.org/) 來建立專案。 Maven是軟體專案管理和理解工具，可讓您建置 Java 專案的軟體、文件及報告。

> [!NOTE]
> 此文件中的步驟最近已在 HDInsight 3.6 中測試過。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="requirements"></a>需求

* [Java platform JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 8 或更新版本。

    > [!NOTE]
    > HDInsight 3.5 和更新版本需要 Java 8。 舊版的 HDInsight 需要 Java 7。

* [Maven](http://maven.apache.org/)

* [具有 HBase 的 Linux 型 Azure HDInsight 叢集](apache-hbase-tutorial-get-started-linux.md#create-hbase-cluster)

## <a name="create-the-project"></a>建立專案

1. 從開發環境的命令列中，將目錄變更至您想要建立專案的位置，例如 `cd code\hbase`。

2. 使用隨 Maven 一起安裝的 **mvn** 命令來產生專案的結構。

    ```bash
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

    > [!NOTE]
    > 如果您使用 PowerShell，則必須將 `-D` 參數放置在雙引號內。
    >
    > `mvn archetype:generate "-DgroupId=com.microsoft.examples" "-DartifactId=hbaseapp" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DinteractiveMode=false"`

    此命令會利用與 **artifactID** 參數 (此範例中為 **hbaseapp**) 相同的名稱來建立目錄。此目錄包含下列項目：

   * **pom.xml**：專案物件模型 ([POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)) 包含用來建置專案之資訊和組態的詳細資料。
   * **src**：包含 **main/java/com/microsoft/examples** 目錄的目錄，您將在此處撰寫應用程式。

3. 刪除 `src/test/java/com/microsoft/examples/apptest.java` 檔案。 此範例不會使用此方法。

## <a name="update-the-project-object-model"></a>更新專案物件模型

1. 編輯 `pom.xml` 檔案，並在 `<dependencies>` 區段內新增下列程式碼：

   ```xml
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>1.1.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.phoenix</groupId>
        <artifactId>phoenix-core</artifactId>
        <version>4.4.0-HBase-1.1</version>
    </dependency>
   ```

    此區段指出專案需要 **hbase-client** 和 **phoenix-core** 元件。 在編譯期間，會從預設的 Maven 儲存機制下載這些相依性。 您可以使用 [Maven 中央儲存機制搜尋](http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar) ，進一步了解此相依性的詳細資訊。

   > [!IMPORTANT]
   > hbase-client 的版本號碼必須符合隨附於 HDInsight 叢集的 HBase 版本。 您可以使用下表來尋找正確的版本號碼。

   | HDInsight 叢集版本 | 要使用的 HBase 版本 |
   | --- | --- |
   | 3.2 |0.98.4-hadoop2 |
   | 3.3、3.4、3.5 和 3.6 |1.1.2 |

    如需 HDInsight 版本和元件的詳細資訊，請參閱 [HDInsight 提供的 Hadoop 元件有什麼不同](../hdinsight-component-versioning.md)。

3. 將下列程式碼加入 **pom.xml** 檔案。 此文字必須位在檔案中的 `<project>...</project>` 標籤內，例如在 `</dependencies>` 和 `</project>` 之間。

   ```xml
    <build>
        <sourceDirectory>src</sourceDirectory>
        <resources>
        <resource>
            <directory>${basedir}/conf</directory>
            <filtering>false</filtering>
            <includes>
            <include>hbase-site.xml</include>
            </includes>
        </resource>
        </resources>
        <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.3</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
            </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.3</version>
            <configuration>
            <transformers>
                <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                </transformer>
            </transformers>
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

    此區段會設定包含 HBase 組態資訊的資源(`conf/hbase-site.xml`)。

   > [!NOTE]
   > 您也可以透過程式碼來設定組態值。 請參閱 `CreateTable` 範例中的註解。

    此區段也會設定 [Maven Compiler 外掛程式](http://maven.apache.org/plugins/maven-compiler-plugin/)與 [Maven Shade 外掛程式](http://maven.apache.org/plugins/maven-shade-plugin/)。 Compiler 外掛程式用來編譯拓撲。 Shade 外掛程式用來防止以 Maven 所建置的 JAR 封裝發生授權重複。 此外掛程式是用於防止 HDInsight 叢集在執行階段發生「重複的授權檔案」錯誤。 使用 maven-shade-plugin 搭配 `ApacheLicenseResourceTransformer` 實作可防止此錯誤。

    maven-shade-plugin 也會產生 uber jar，其中含有應用程式需要的所有相依性。

4. 儲存 `pom.xml` 檔案。

5. 在 `hbaseapp` 目錄中建立名為 `conf` 的目錄。 此目錄會用來保存連接至 HBase 的組態資訊。

6. 使用下列命令將 HBase 組態，從 HBase 叢集複製到 `conf` 目錄。 將 `USERNAME` 取代為您的 SSH 登入名稱。 將 `CLUSTERNAME` 取代為 HDInsight 叢集名稱：

    ```bash
    scp USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml
    ```

   如需使用 `ssh` 和 `scp` 的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="create-the-application"></a>建立應用程式

1. 移至 `hbaseapp/src/main/java/com/microsoft/examples` 目錄，並將 app.java 檔案重新命名為 `CreateTable.java`。

2. 開啟 `CreateTable.java` 檔案，並以下列文字取代現有的內容：

   ```java
    package com.microsoft.examples;
    import java.io.IOException;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.client.HBaseAdmin;
    import org.apache.hadoop.hbase.HTableDescriptor;
    import org.apache.hadoop.hbase.TableName;
    import org.apache.hadoop.hbase.HColumnDescriptor;
    import org.apache.hadoop.hbase.client.HTable;
    import org.apache.hadoop.hbase.client.Put;
    import org.apache.hadoop.hbase.util.Bytes;

    public class CreateTable {
        public static void main(String[] args) throws IOException {
        Configuration config = HBaseConfiguration.create();

        // Example of setting zookeeper values for HDInsight
        // in code instead of an hbase-site.xml file
        //
        // config.set("hbase.zookeeper.quorum",
        //            "zookeepernode0,zookeepernode1,zookeepernode2");
        //config.set("hbase.zookeeper.property.clientPort", "2181");
        //config.set("hbase.cluster.distributed", "true");
        //
        //NOTE: Actual zookeeper host names can be found using Ambari:
        //curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts"

        //Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
        config.set("zookeeper.znode.parent","/hbase-unsecure");

        // create an admin object using the config
        HBaseAdmin admin = new HBaseAdmin(config);

        // create the table...
        HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf("people"));
        // ... with two column families
        tableDescriptor.addFamily(new HColumnDescriptor("name"));
        tableDescriptor.addFamily(new HColumnDescriptor("contactinfo"));
        admin.createTable(tableDescriptor);

        // define some people
        String[][] people = {
            { "1", "Marcel", "Haddad", "marcel@fabrikam.com"},
            { "2", "Franklin", "Holtz", "franklin@contoso.com" },
            { "3", "Dwayne", "McKee", "dwayne@fabrikam.com" },
            { "4", "Rae", "Schroeder", "rae@contoso.com" },
            { "5", "Rosalie", "burton", "rosalie@fabrikam.com"},
            { "6", "Gabriela", "Ingram", "gabriela@contoso.com"} };

        HTable table = new HTable(config, "people");

        // Add each person to the table
        //   Use the `name` column family for the name
        //   Use the `contactinfo` column family for the email
        for (int i = 0; i< people.length; i++) {
            Put person = new Put(Bytes.toBytes(people[i][0]));
            person.add(Bytes.toBytes("name"), Bytes.toBytes("first"), Bytes.toBytes(people[i][1]));
            person.add(Bytes.toBytes("name"), Bytes.toBytes("last"), Bytes.toBytes(people[i][2]));
            person.add(Bytes.toBytes("contactinfo"), Bytes.toBytes("email"), Bytes.toBytes(people[i][3]));
            table.put(person);
        }
        // flush commits and close the table
        table.flushCommits();
        table.close();
        }
    }
   ```

    此程式碼是 **CreateTable** 類別，將會建立名為 **people** 的資料表，並填入一些預先定義的使用者。

3. 儲存 `CreateTable.java` 檔案。

4. 在 `hbaseapp/src/main/java/com/microsoft/examples` 目錄中，建立名稱為 `SearchByEmail.java` 的檔案。 使用下列文字做為此檔案的內容：

   ```java
    package com.microsoft.examples;
    import java.io.IOException;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.client.HTable;
    import org.apache.hadoop.hbase.client.Scan;
    import org.apache.hadoop.hbase.client.ResultScanner;
    import org.apache.hadoop.hbase.client.Result;
    import org.apache.hadoop.hbase.filter.RegexStringComparator;
    import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
    import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
    import org.apache.hadoop.hbase.util.Bytes;
    import org.apache.hadoop.util.GenericOptionsParser;

    public class SearchByEmail {
        public static void main(String[] args) throws IOException {
        Configuration config = HBaseConfiguration.create();

        // Use GenericOptionsParser to get only the parameters to the class
        // and not all the parameters passed (when using WebHCat for example)
        String[] otherArgs = new GenericOptionsParser(config, args).getRemainingArgs();
        if (otherArgs.length != 1) {
            System.out.println("usage: [regular expression]");
            System.exit(-1);
        }

        // Open the table
        HTable table = new HTable(config, "people");

        // Define the family and qualifiers to be used
        byte[] contactFamily = Bytes.toBytes("contactinfo");
        byte[] emailQualifier = Bytes.toBytes("email");
        byte[] nameFamily = Bytes.toBytes("name");
        byte[] firstNameQualifier = Bytes.toBytes("first");
        byte[] lastNameQualifier = Bytes.toBytes("last");

        // Create a regex filter
        RegexStringComparator emailFilter = new RegexStringComparator(otherArgs[0]);
        // Attach the regex filter to a filter
        //   for the email column
        SingleColumnValueFilter filter = new SingleColumnValueFilter(
            contactFamily,
            emailQualifier,
            CompareOp.EQUAL,
            emailFilter
        );

        // Create a scan and set the filter
        Scan scan = new Scan();
        scan.setFilter(filter);

        // Get the results
        ResultScanner results = table.getScanner(scan);
        // Iterate over results and print  values
        for (Result result : results ) {
            String id = new String(result.getRow());
            byte[] firstNameObj = result.getValue(nameFamily, firstNameQualifier);
            String firstName = new String(firstNameObj);
            byte[] lastNameObj = result.getValue(nameFamily, lastNameQualifier);
            String lastName = new String(lastNameObj);
            System.out.println(firstName + " " + lastName + " - ID: " + id);
            byte[] emailObj = result.getValue(contactFamily, emailQualifier);
            String email = new String(emailObj);
            System.out.println(firstName + " " + lastName + " - " + email + " - ID: " + id);
        }
        results.close();
        table.close();
        }
    }
   ```

    **SearchByEmail** 類別可用來依電子郵件地址查詢資料列。 因為此類別使用規則運算式篩選器，您可以在使用此類別時提供字串或規則運算式。

5. 儲存 `SearchByEmail.java` 檔案。

6. 在 `hbaseapp/src/main/hava/com/microsoft/examples` 目錄中，建立名稱為 `DeleteTable.java` 的檔案。 使用下列文字做為此檔案的內容：

   ```java
    package com.microsoft.examples;
    import java.io.IOException;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.client.HBaseAdmin;

    public class DeleteTable {
        public static void main(String[] args) throws IOException {
        Configuration config = HBaseConfiguration.create();

        // Create an admin object using the config
        HBaseAdmin admin = new HBaseAdmin(config);

        // Disable, and then delete the table
        admin.disableTable("people");
        admin.deleteTable("people");
        }
    }
   ```

    此類別會清理此範例中建立的 HBase 資料表，方法是停用並卸除 `CreateTable` 類別所建立的資料表。

7. 儲存 `DeleteTable.java` 檔案。

## <a name="build-and-package-the-application"></a>建置和封裝應用程式

1. 從 `hbaseapp` 目錄，使用下列命令來建置含有應用程式的 JAR 檔案：

    ```bash
    mvn clean package
    ```

    此命令會建置應用程式並封裝到 .jar 檔案中。

2. 命令完成時，`hbaseapp/target` 目錄就會包含名為 `hbaseapp-1.0-SNAPSHOT.jar` 的檔案。

   > [!NOTE]
   > `hbaseapp-1.0-SNAPSHOT.jar` 檔案是 uber jar。 它包含執行應用程式需要的所有相依性。


## <a name="upload-the-jar-and-run-jobs-ssh"></a>上傳 JAR 並執行作業 (SSH)

下列步驟使用 `scp` 將 JAR 複製到 HBase on HDInsight 叢集的主要前端節點。 接著，會使用 `ssh` 命令連接到該叢集並直接在前端節點上執行範例。

1. 若要將 jar 上傳到叢集，請使用下列命令︰

    ```bash
    scp ./target/hbaseapp-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:hbaseapp-1.0-SNAPSHOT.jar
    ```

    將 `USERNAME` 取代為您的 SSH 登入名稱。 將 `CLUSTERNAME` 取代為 HDInsight 叢集名稱。

2. 若要連線到 HBase 叢集，請使用下列命令︰

    ```bash
    ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    將 `USERNAME` 取代為您的 SSH 登入名稱。 將 `CLUSTERNAME` 取代為 HDInsight 叢集名稱。

3. 若要使用 Java 應用程式來建立 HBase 資料表，請使用下列命令：

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable
    ```

    此命令會建立名為 **people** 的 HBase 資料表，並填入資料。

4. 若要搜尋資料表中儲存的電子郵件地址，請使用以下命令：

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com
    ```

    您會收到下列結果：

        Franklin Holtz - ID: 2
        Franklin Holtz - franklin@contoso.com - ID: 2
        Rae Schroeder - ID: 4
        Rae Schroeder - rae@contoso.com - ID: 4
        Gabriela Ingram - ID: 6
        Gabriela Ingram - gabriela@contoso.com - ID: 6

5. 若要刪除資料表，請使用下列命令：

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable
    ```

## <a name="upload-the-jar-and-run-jobs-powershell"></a>上傳 JAR 並執行作業 (PowerShell)

下列步驟使用 Azure PowerShell 將 JAR 上傳到您 HBase 叢集的預設儲存體。 接著，會使用 HDInsight Cmdlet 從遠端執行範例。

1. 安裝並設定 Azure PowerShell 之後，請建立名為 `hbase-runner.psm1` 的檔案。 使用下列文字做為此檔案的內容：

   ```powershell
    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.CreateTable"
    -clusterName "MyHDInsightCluster"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "contoso.com"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "^r" -showErr
    #>

    function Start-HBaseExample {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
    #The class to run
    [Parameter(Mandatory = $true)]
    [String]$className,

    #The name of the HDInsight cluster
    [Parameter(Mandatory = $true)]
    [String]$clusterName,

    #Only used when using SearchByEmail
    [Parameter(Mandatory = $false)]
    [String]$emailRegex,

    #Use if you want to see stderr output
    [Parameter(Mandatory = $false)]
    [Switch]$showErr
    )

    Set-StrictMode -Version 3

    # Is the Azure module installed?
    FindAzure

    # Get the login for the HDInsight cluster
    $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

    # The JAR
    $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

    # The job definition
    $jobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
        -JarFile $jarFile `
        -ClassName $className `
        -Arguments $emailRegex

    # Get the job output
    $job = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds
    if($showErr)
    {
    Write-Host "STDERR"
    Get-AzureRmHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds `
                -DisplayOutputType StandardError
    }
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds
    }

    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    -Container "MyContainer"
    #>

    function Add-HDInsightFile {
        [CmdletBinding(SupportsShouldProcess = $true)]
        param(
            #The path to the local file.
            [Parameter(Mandatory = $true)]
            [String]$localPath,

            #The destination path and file name, relative to the root of the container.
            [Parameter(Mandatory = $true)]
            [String]$destinationPath,

            #The name of the HDInsight cluster
            [Parameter(Mandatory = $true)]
            [String]$clusterName,

            #If specified, overwrites existing files without prompting
            [Parameter(Mandatory = $false)]
            [Switch]$force
        )

        Set-StrictMode -Version 3

        # Is the Azure module installed?
        FindAzure

        # Get authentication for the cluster
        $creds=Get-Credential

        # Does the local path exist?
        if (-not (Test-Path $localPath))
        {
            throw "Source path '$localPath' does not exist."
        }

        # Get the primary storage container
        $storage = GetStorage -clusterName $clusterName

        # Upload file to storage, overwriting existing files if -force was used.
        Set-AzureStorageBlobContent -File $localPath `
            -Blob $destinationPath `
            -force:$force `
            -Container $storage.container `
            -Context $storage.context
    }

    function FindAzure {
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            throw "No active Azure subscription found! If you have a subscription, use the Connect-AzureRmAccount cmdlet to login to your subscription."
        }
    }

    function GetStorage {
        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName
        )
        $hdi = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        # Does the cluster exist?
        if (!$hdi)
        {
            throw "HDInsight cluster '$clusterName' does not exist."
        }
        # Create a return object for context & container
        $return = @{}
        $storageAccounts = @{}

        # Get storage information
        $resourceGroup = $hdi.ResourceGroup
        $storageAccountName=$hdi.DefaultStorageAccount.split('.')[0]
        $container=$hdi.DefaultStorageContainer
        $storageAccountKey=(Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
        # Get the resource group, in case we need that
        $return.resourceGroup = $resourceGroup
        # Get the storage context, as we can't depend
        # on using the default storage context
        $return.context = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
        # Get the container, so we know where to
        # find/store blobs
        $return.container = $container
        # Return storage accounts to support finding all accounts for
        # a cluster
        $return.storageAccount = $storageAccountName
        $return.storageAccountKey = $storageAccountKey

        return $return
    }
    # Only export the verb-phrase things
    export-modulemember *-*
   ```

    此檔案包含兩個模組：

   * **Add-HDInsightFile** - 用來將檔案上傳到叢集
   * **Start-HBaseExample** - 用來執行稍早建立的類別

2. 儲存 `hbase-runner.psm1` 檔案。

3. 開啟新的 Azure PowerShell 視窗，切換至 `hbaseapp` 目錄，然後執行下列命令：

    ```powershell
    PS C:\ Import-Module c:\path\to\hbase-runner.psm1
    ```

    將路徑變更為稍早建立之 `hbase-runner.psm1` 檔案的位置。 此命令會使用 Azure PowerShell 註冊該模組。

4. 使用下列命令將 `hbaseapp-1.0-SNAPSHOT.jar` 上傳到您的叢集。

    ```powershell
    Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName hdinsightclustername
    ```

    將 `hdinsightclustername` 取代為您的叢集名稱。 出現提示時，輸入叢集登入 (admin) 名稱和密碼。 此命令會將 `hbaseapp-1.0-SNAPSHOT.jar` 上傳至您叢集的主要儲存體中的 `example/jars` 位置。

5. 若要使用 `hbaseapp` 建立資料表，請使用下列命令：

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName hdinsightclustername
    ```

    將 `hdinsightclustername` 取代為您的叢集名稱。 出現提示時，輸入叢集登入 (admin) 名稱和密碼。

    此命令會在您 HDInsight 叢集上的 HBase 中建立名為 **people** 的資料表。 此命令不會在主控台視窗中顯示任何輸出。

6. 若要在資料表中搜尋項目，請使用下列命令：

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName hdinsightclustername -emailRegex contoso.com
    ```

    將 `hdinsightclustername` 取代為您的叢集名稱。 出現提示時，輸入叢集登入 (admin) 名稱和密碼。

    此命令會使用 `SearchByEmail` 類別來搜尋 `contactinformation` 資料行系列和 `email` 資料行包含字串 `contoso.com` 的任何資料列。 您應該會得到下列結果：

          Franklin Holtz - ID: 2
          Franklin Holtz - franklin@contoso.com - ID: 2
          Rae Schroeder - ID: 4
          Rae Schroeder - rae@contoso.com - ID: 4
          Gabriela Ingram - ID: 6
          Gabriela Ingram - gabriela@contoso.com - ID: 6

    使用 **fabrikam.com** 做為 `-emailRegex` 值會傳回電子郵件欄位中含有 **fabrikam.com** 的使用者。 您也可以使用規則運算式作為搜尋字詞。 例如，**^r** 會傳回開頭為字母 'r' 的電子郵件地址。

### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>使用 Start-HBaseExample 時沒有結果或傳回非預期的結果

請使用 `-showErr` 參數，以檢視執行工作時所產生的標準錯誤 (STDERR)。

## <a name="delete-the-table"></a>刪除資料表

練習完範例之後，請使用下列命令來刪除在此範例中使用的 **people** 資料表：

__從 `ssh` 工作階段__：

`yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable`

__從 Azure PowerShell__：

`Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName hdinsightclustername`

## <a name="next-steps"></a>後續步驟

[了解如何使用 SQuirreL SQL 搭配 HBase](apache-hbase-phoenix-squirrel-linux.md)
