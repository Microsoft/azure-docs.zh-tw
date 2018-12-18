---
title: 建置 Windows 型 Azure HDInsight 的 Java HBase 應用程式
description: 了解如何使用 Apache Maven 建置以 Java 為基礎的 Apache HBase 應用程式，然後部署至以 Windows 為基礎的 Azure HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/05/2017
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: fc2bd03156d6005f1e10252583b4a13451f66dbf
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43092622"
---
# <a name="use-maven-to-build-java-applications-that-use-hbase-with-windows-based-hdinsight-hadoop"></a>使用 Maven 建置搭配使用 HBase 和以 Window 為基礎的 HDInsight (Hadoop) 的 Java 應用程式
了解如何使用 Apache Maven 以 Java 建立和建置 [Apache HBase](http://hbase.apache.org/) 應用程式。 然後在 Azure HDInsight (Hadoop) 中使用此應用程式。

[Maven](http://maven.apache.org/) 是軟體專案管理和理解工具，可讓您建置 Java 專案的軟體、文件及報告。 在本文中，您會了解如何用它來建立基本的 Java 應用程式，以便在 Azure HDInsight 叢集上建立、查詢和刪除 HBase 資料表。

> [!IMPORTANT]
> 本文件中的步驟需要一個使用 Windows 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="requirements"></a>需求
* [Java platform JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7 或更新版本
* [Maven](http://maven.apache.org/)
* 搭配使用 HBase 和以 Windows 為基礎的 HDInsight 叢集

    > [!NOTE]
    > 這份文件中的步驟已經過 HDInsight 叢集 3.2 版和 3.3 版的測試. 在範例中提供的預設值是 HDInsight 3.3 叢集。

## <a name="create-the-project"></a>建立專案
1. 從開發環境的命令列中，將目錄變更至您想要建立專案的位置，例如 `cd code\hdinsight`。
2. 使用隨 Maven 一起安裝的 **mvn** 命令來產生專案的結構。

        mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    此命令會在目前的位置建立目錄，其名稱由 **artifactID** 參數指定 (此範例中為 **hbaseapp**)。此目錄包含下列項目：

   * **pom.xml**：專案物件模型 ([POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)) 包含用來建置專案之資訊和組態的詳細資料。
   * **src**：含有 **main\java\com\microsoft\examples** 目錄的目錄，您將在此處撰寫應用程式。
3. 刪除 **src\test\java\com\microsoft\examples\apptest.java** 檔案，因為此範例中不會用到。

## <a name="update-the-project-object-model"></a>更新專案物件模型
1. 編輯 **pom.xml** 檔案，並在 `<dependencies>` 區段內加入下列程式碼：

        <dependency>
          <groupId>org.apache.hbase</groupId>
          <artifactId>hbase-client</artifactId>
          <version>1.1.2</version>
        </dependency>

    此區段會告知 Maven，表示專案需要 **hbase-client** 版本 **1.1.2**。 編譯時，將會從預設 Maven 儲存機制下載此相依性。 您可以使用 [Maven 中央儲存機制搜尋](http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar) ，進一步了解此相依性的詳細資訊。

   > [!IMPORTANT]
   > 版本號碼必須符合隨附於 HDInsight 叢集的 HBase 版本。 您可以使用下表來尋找正確的版本號碼。
   >
   >

   | HDInsight 叢集版本 | 要使用的 HBase 版本 |
   | --- | --- |
   | 3.2 |0.98.4-hadoop2 |
   | 3.3 |1.1.2 |

    如需 HDInsight 版本和元件的詳細資訊，請參閱 [HDInsight 提供的 Hadoop 元件有什麼不同](hdinsight-component-versioning.md)。
2. 如果您使用 HDInsight 3.3 叢集，也必須將下列內容加入 `<dependencies>` 區段︰

        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>4.4.0-HBase-1.1</version>
        </dependency>

    此相依性會載入 Phoenix 核心元件，可供 Hbase 1.1.x 版使用。
3. 將下列程式碼加入 **pom.xml** 檔案。 此區段必須在檔案中的 `<project>...</project>` 標籤內，例如在 `</dependencies>` 和 `</project>` 之間。

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
                    <source>1.7</source>
                    <target>1.7</target>
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

    `<resources>` 區段會設定包含 HBase 組態資訊的資源 (**conf\hbase-site.xml**)。

   > [!NOTE]
   > 您也可以透過程式碼來設定組態值。 相關作法請參閱接下來 **CreateTable** 範例中的註解。
   >
   >

    此 `<plugins>` 區段會設定 [Maven Compiler 外掛程式](http://maven.apache.org/plugins/maven-compiler-plugin/)和 [Maven Shade 外掛程式](http://maven.apache.org/plugins/maven-shade-plugin/)。 Compiler 外掛程式用來編譯拓撲。 Shade 外掛程式用來防止以 Maven 所建置的 JAR 封裝發生授權重複。 使用此項目的理由在於，重複的授權檔會導致 HDInsight 叢集在執行階段發生錯誤。 使用 maven-shade-plugin 搭配 `ApacheLicenseResourceTransformer` 實作可防止此錯誤。

    maven-shade-plugin 也會產生 uber jar (或 fat jar)，其含有應用程式需要的所有相依性。
4. 儲存 **pom.xml** 檔案。
5. 在 **conf** in the **hbaseapp** 的新目錄。 在 **conf** 目錄中，建立名為 **hbase-site.xml** 的檔案。 使用下列項目做為檔案的內容：

        <?xml version="1.0"?>
        <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
        <!--
        /**
          * Copyright 2010 The Apache Software Foundation
          *
          * Licensed to the Apache Software Foundation (ASF) under one
          * or more contributor license agreements.  See the NOTICE file
          * distributed with this work for additional information
          * regarding copyright ownership.  The ASF licenses this file
          * to you under the Apache License, Version 2.0 (the
          * "License"); you may not use this file except in compliance
          * with the License.  You may obtain a copy of the License at
          *
          *     http://www.apache.org/licenses/LICENSE-2.0
          *
          * Unless required by applicable law or agreed to in writing, software
          * distributed under the License is distributed on an "AS IS" BASIS,
          * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
          * See the License for the specific language governing permissions and
          * limitations under the License.
          */
        -->
        <configuration>
          <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
          </property>
          <property>
            <name>hbase.zookeeper.quorum</name>
            <value>zookeeper0,zookeeper1,zookeeper2</value>
          </property>
          <property>
            <name>hbase.zookeeper.property.clientPort</name>
            <value>2181</value>
          </property>
        </configuration>

    此檔案用來載入 HDInsight 叢集的 HBase 組態。

   > [!NOTE]
   > 這是最小的 hbase-site.xml 檔案，其包含 HDInsight 叢集最低限度的設定。

6. 儲存 **hbase-site.xml** 檔案。

## <a name="create-the-application"></a>建立應用程式
1. 移至 **hbaseapp\src\main\java\com\microsoft\examples** 目錄，並將 app.java 檔案重新命名為 **CreateTable.java**。
2. 開啟 **CreateTable.java** 檔案，並以下列程式碼取代現有的內容：

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
            // The following sets the znode root for Linux-based HDInsight
            //config.set("zookeeper.znode.parent","/hbase-unsecure");

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

    這是 **CreateTable** 類別，將會建立名為 **people** 的資料表，並填入一些預先定義的使用者。
3. 儲存 **CreateTable.java** 檔案。
4. 在 **hbaseapp\src\main\java\com\microsoft\examples** 目錄中，建立名為 **SearchByEmail.java** 的新檔案。 使用下列程式碼做為此檔案的內容：

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

            // Create a new regex filter
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

    **SearchByEmail** 類別可用來依電子郵件地址查詢資料列。 因為此類別使用規則運算式篩選器，您可以在使用此類別時提供字串或規則運算式。
5. 儲存 **SearchByEmail.java** 檔案。
6. 在 **hbaseapp\src\main\hava\com\microsoft\examples** 目錄中，建立名為 **DeleteTable.java** 的新檔案。 使用下列程式碼做為此檔案的內容：

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

    此類別是用來清理此範例，作法是停用並卸除 **CreateTable** 類別所建立的資料表。
7. 儲存 **DeleteTable.java** 檔案。

## <a name="build-and-package-the-application"></a>建置和封裝應用程式
1. 開啟命令提示字元，切換至 **hbaseapp** 目錄。
2. 使用下列命令來建置含有應用程式的 JAR：

        mvn clean package

    這會清除任何先前的組建成品、下載任何尚未安裝的相依性，然後建置並封裝應用程式。
3. 指令完成後，**hbaseapp\target** 目錄會包含一個名為 **hbaseapp-1.0-SNAPSHOT.jar** 的檔案。

   > [!NOTE]
   > **hbaseapp-1.0-SNAPSHOT.jar** 檔案是一個 uber jar (有時稱為 fat jar)，內含執行應用程式所需的所有相依性。

## <a name="upload-the-jar-file-and-start-a-job"></a>上傳 JAR 檔案並啟動工作
有許多方法可將檔案上傳至 HDInsight 叢集，如 [在 HDInsight 中將 Hadoop 工作的資料上傳](hdinsight-upload-data.md)中所述。 下列步驟會使用 Azure PowerShell。

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

1. 安裝並設定 Azure PowerShell 之後，建立名為 **hbase-runner.psm1** 的新檔案。 使用下列項目做為此檔案的內容：

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

    此檔案包含兩個模組：

   * **Add-HDInsightFile** - 用來將檔案上傳至 HDInsight
   * **Start-HBaseExample** - 用來執行稍早建立的類別
2. 儲存 **hbase-runner.psm1** 檔案。
3. 開啟新的 Azure PowerShell 視窗，切換至 **hbaseapp** 目錄，然後執行下列命令。

        PS C:\ Import-Module c:\path\to\hbase-runner.psm1

    將路徑變更為稍早建立的 **hbase-runner.psm1** 檔案的位置。 這會為此 Azure PowerShell 工作階段註冊模組。
4. 使用下列命令將 **hbaseapp-1.0-SNAPSHOT.jar** 上傳至 HDInsight 叢集。

        Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName hdinsightclustername

    將 **hdinsightclustername** 換成您的 HDInsight 叢集名稱。 此命令會將 **hbaseapp-1.0-SNAPSHOT.jar** 上傳至 HDInsight 叢集主要儲存體中的 **example/jars** 位置。
5. 檔案上傳之後，使用以下程式碼建立使用 **hbaseapp** 的資料表：

        Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName hdinsightclustername

    將 **hdinsightclustername** 換成您的 HDInsight 叢集名稱。

    此命令會在 HDInsight 叢集中建立名為 **people** 的新資料表。 此命令不會在主控台視窗中顯示任何輸出。
6. 若要在資料表中搜尋項目，請使用下列命令：

        Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName hdinsightclustername -emailRegex contoso.com

    將 **hdinsightclustername** 換成您的 HDInsight 叢集名稱。

    此命令會使用 **SearchByEmail** 類別來搜尋資料行系列 **contactinformation**、資料行 **email** 包含字串 **contoso.com** 的任何資料列。 您應該會得到下列結果：

          Franklin Holtz - ID: 2
          Franklin Holtz - franklin@contoso.com - ID: 2
          Rae Schroeder - ID: 4
          Rae Schroeder - rae@contoso.com - ID: 4
          Gabriela Ingram - ID: 6
          Gabriela Ingram - gabriela@contoso.com - ID: 6

    使用 **fabrikam.com** 做為 `-emailRegex` 值會傳回電子郵件欄位中含有 **fabrikam.com** 的使用者。 因為此搜尋是以規則運算式篩選器來實作，您也可以輸入像是 **^r** 這樣的規則運算式，其會傳回電子郵件以字母 'r' 開頭的項目。

## <a name="delete-the-table"></a>刪除資料表
練習完範例之後，請從 Azure PowerShell 工作階段中使用下列命令，以刪除此範例所使用的 **people** 資料表：

    Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName hdinsightclustername

將 **hdinsightclustername** 換成您的 HDInsight 叢集名稱。

## <a name="troubleshooting"></a>疑難排解
### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>使用 Start-HBaseExample 時沒有結果或傳回非預期的結果
請使用 `-showErr` 參數，以檢視執行工作時所產生的標準錯誤 (STDERR)。
