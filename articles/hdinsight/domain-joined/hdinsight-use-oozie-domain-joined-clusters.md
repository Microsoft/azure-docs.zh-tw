---
title: 具有企業安全性套件之 Azure HDInsight 叢集中的 Apache Hadoop Oozie 工作流程
description: 在 Linux 型 HDInsight 企業安全性套件中使用 Hadoop Oozie。 了解如何定義 Oozie 工作流程，以及提交 Oozie 作業。
services: hdinsight
ms.service: hdinsight
author: omidm1
ms.author: omidm
ms.reviewer: mamccrea
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 563a4c14d78b7edd228c998817f44c6b3f14efe7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46947483"
---
# <a name="run-apache-oozie-in-hdinsight-hadoop-clusters-with-enterprise-security-package"></a>在具有企業安全性套件的 HDInsight Hadoop 叢集中執行 Apache Oozie
Oozie 是可管理 Hadoop 作業的工作流程和協調系統。 Oozie 已與 Hadoop 堆疊整合，並支援下列作業：
- Apache MapReduce
- Apache Pig
- Apache Hive
- Apache Sqoop

您也可以使用 Oozie 來排程系統的特定作業，例如 Java 程式或 Shell 指令碼。

## <a name="prerequisite"></a>必要條件
- 具有企業安全性套件 (ESP) 的 Azure HDInsight Hadoop 叢集。 請參閱[設定具有 ESP 的 HDInsight 叢集](./apache-domain-joined-configure-using-azure-adds.md)。

    > [!NOTE]
    > 如需在非 ESP 的叢集上使用 Oozie 的詳細指示，請參閱[在 Linux 型 Azure HDInsight 中使用 Hadoop Oozie 工作流程](../hdinsight-use-oozie-linux-mac.md)。

## <a name="connect-to-an-esp-cluster"></a>連線到 ESP 叢集

如需安全殼層 (SSH) 的詳細資訊，請參閱[使用 SSH 連線至 HDInsight (Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

1. 使用 SSH 連線到 HDInsight 叢集：  
 ```bash
ssh [DomainUserName]@<clustername>-ssh.azurehdinsight.net
 ```

2. 若要確認 Kerberos 驗證是否成功，請使用 `klist` 命令。 如果不成功，請使用 `kinit` 啟動 Kerberos 驗證。

3. 登入 HDInsight 閘道，以註冊存取 Azure Data Lake Storage 所需的 OAuth 權杖：   
     ```bash
     curl -I -u [DomainUserName@Domain.com]:[DomainUserPassword] https://<clustername>.azurehdinsight.net
     ```

    狀態回應碼 **200 OK** 表示註冊成功。 如果收到未經授權的回應 (例如 401)，請檢查使用者名稱與密碼。

## <a name="define-the-workflow"></a>定義工作流程
Oozie 工作流程定義會以 Hadoop 程序定義語言 (hPDL) 撰寫。 hPDL 是一種 XML 流程定義語言。 採用下列步驟定義工作流程：

1.  設定網域使用者的工作區：
 ```bash
hdfs dfs -mkdir /user/<DomainUser>
cd /home/<DomainUserPath>
cp /usr/hdp/<ClusterVersion>/oozie/doc/oozie-examples.tar.gz .
tar -xvf oozie-examples.tar.gz
hdfs dfs -put examples /user/<DomainUser>/
 ```
將 `DomainUser` 取代為網域使用者名稱。 將 `DomainUserPath` 取代為網域使用者的主目錄路徑。 將 `ClusterVersion` 取代您的叢集 Hortonworks Data Platform (HDP) 版本。

2.  使用以下陳述式建立和編輯新的檔案：
 ```bash
nano workflow.xml
 ```

3. 在 nano 編輯器開啟後，請輸入下列 XML 做為檔案內容：
 ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <workflow-app xmlns="uri:oozie:workflow:0.4" name="map-reduce-wf">
       <credentials>
          <credential name="metastore_token" type="hcat">
             <property>
                <name>hcat.metastore.uri</name>
                <value>thrift://hn0-<clustername>.<Domain>.com:9083</value>
             </property>
             <property>
                <name>hcat.metastore.principal</name>
                <value>hive/_HOST@<Domain>.COM</value>
             </property>
          </credential>
          <credential name="hs2-creds" type="hive2">
             <property>
                <name>hive2.server.principal</name>
                <value>${jdbcPrincipal}</value>
             </property>
             <property>
                <name>hive2.jdbc.url</name>
                <value>${jdbcURL}</value>
             </property>
          </credential>
       </credentials>
       <start to="mr-test" />
       <action name="mr-test">
          <map-reduce>
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <prepare>
                <delete path="${nameNode}/user/${wf:user()}/examples/output-data/mrresult" />
             </prepare>
             <configuration>
                <property>
                   <name>mapred.job.queue.name</name>
                   <value>${queueName}</value>
                </property>
                <property>
                   <name>mapred.mapper.class</name>
                   <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                   <name>mapred.reducer.class</name>
                   <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                   <name>mapred.map.tasks</name>
                   <value>1</value>
                </property>
                <property>
                   <name>mapred.input.dir</name>
                   <value>/user/${wf:user()}/${examplesRoot}/input-data/text</value>
                </property>
                <property>
                   <name>mapred.output.dir</name>
                   <value>/user/${wf:user()}/${examplesRoot}/output-data/mrresult</value>
                </property>
             </configuration>
          </map-reduce>
          <ok to="myHive2" />
          <error to="fail" />
       </action>
       <action name="myHive2" cred="hs2-creds">
          <hive2 xmlns="uri:oozie:hive2-action:0.2">
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <jdbc-url>${jdbcURL}</jdbc-url>
             <script>${hiveScript2}</script>
             <param>hiveOutputDirectory2=${hiveOutputDirectory2}</param>
          </hive2>
          <ok to="myHive" />
          <error to="fail" />
       </action>
       <action name="myHive" cred="metastore_token">
          <hive xmlns="uri:oozie:hive-action:0.2">
             <job-tracker>${jobTracker}</job-tracker>
             <name-node>${nameNode}</name-node>
             <configuration>
                <property>
                   <name>mapred.job.queue.name</name>
                   <value>${queueName}</value>
                </property>
             </configuration>
             <script>${hiveScript1}</script>
             <param>hiveOutputDirectory1=${hiveOutputDirectory1}</param>
          </hive>
          <ok to="end" />
          <error to="fail" />
       </action>
       <kill name="fail">
          <message>Oozie job failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
       </kill>
       <end name="end" />
    </workflow-app>
 ```
4. 將 `clustername` 取代為叢集的名稱。 

5. 若要儲存檔案，請選取 Ctrl+X。 輸入 `Y`。 然後，選取 **Enter** 鍵。

    工作流程分成兩個部分：
    *   **[認證] 區段。** 此區段會採用將用來驗證 Oozie 動作的認證：

       此範例使用的是 Hive 動作的驗證。 若要深入了解，請參閱[動作驗證](https://oozie.apache.org/docs/4.2.0/DG_ActionAuthentication.html)。

       認證服務允許 Oozie 動作模擬使用者存取 Hadoop 服務。

    *   **[動作] 區段。** 此區段會有三個動作：map-reduce、Hive server 2 和 Hive server 1：

      - map-reduce 動作會從 Oozie 套件為輸出彙總字數的 map-reduce 執行範例。

       - Hive server 2 和 Hive server 1 動作會對隨 HDInsight 提供的範例 Hive 資料表執行查詢。

        Hive 動作會透過動作元素中的關鍵字 `cred`，使用 [認證] 區段中定義的認證來進行驗證。

6. 使用下列命令將 `workflow.xml` 檔案複製到 `/user/<domainuser>/examples/apps/map-reduce/workflow.xml`：
     ```bash
    hdfs dfs -put workflow.xml /user/<domainuser>/examples/apps/map-reduce/workflow.xml
     ```

7. 將 `domainuser` 取代為您網域的使用者名稱。

## <a name="define-the-properties-file-for-the-oozie-job"></a>定義 Oozie 作業的內容檔案

1. 使用以下陳述式建立和編輯作業內容的新檔案：

   ```bash
   nano job.properties
   ```

2. 在 nano 編輯器開啟後，請使用下列 XML 做為檔案的內容：

   ```bash
       nameNode=adl://home
       jobTracker=headnodehost:8050
       queueName=default
       examplesRoot=examples
       oozie.wf.application.path=${nameNode}/user/[domainuser]/examples/apps/map-reduce/workflow.xml
       hiveScript1=${nameNode}/user/${user.name}/countrowshive1.hql
       hiveScript2=${nameNode}/user/${user.name}/countrowshive2.hql
       oozie.use.system.libpath=true
       user.name=[domainuser]
       jdbcPrincipal=hive/hn0-<ClusterShortName>.<Domain>.com@<Domain>.COM
       jdbcURL=[jdbcurlvalue]
       hiveOutputDirectory1=${nameNode}/user/${user.name}/hiveresult1
       hiveOutputDirectory2=${nameNode}/user/${user.name}/hiveresult2
   ```
  
   a. 將 `domainuser` 取代為您網域的使用者名稱。  
   b. 將 `ClusterShortName` 取代為叢集的簡短名稱。 例如，如果叢集名稱為 https:// *[範例連結]* sechadoopcontoso.azurehdisnight.net，則 `clustershortname` 為叢集的前六個字元：**sechad**。  
   c. 將 `jdbcurlvalue` 取代為 Hive 組態中的 JDBC URL。 範例為 jdbc:hive2://headnodehost:10001/;transportMode=http。      
   d. 若要儲存檔案，請選取 Ctrl+X、輸入 `Y`，然後選取 **Enter**。

   執行 Oozie 作業時，此內容檔案必須存在於本機。

## <a name="create-custom-hive-scripts-for-oozie-jobs"></a>建立 Oozie 作業的自訂 Hive 指令碼
您可以為 Hive server 1 和 Hive server 2 建立兩個 Hive 指令碼，如以下幾節所說明。
### <a name="hive-server-1-file"></a>Hive server 1 檔案
1.  建立和編輯 Hive server 1 動作的檔案：
    ```bash
    nano countrowshive1.hql
    ```

2.  建立指令碼：
    ```sql
    INSERT OVERWRITE DIRECTORY '${hiveOutputDirectory1}' 
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
    select devicemake from hivesampletable limit 2;
    ```

3.  將檔案儲存至 Hadoop 分散式檔案系統 (HDFS)：
    ```bash
    hdfs dfs -put countrowshive1.hql countrowshive1.hql
    ```

### <a name="hive-server-2-file"></a>Hive server 2 檔案
1.  建立和編輯 Hive server 2 動作的欄位：
    ```bash
    nano countrowshive2.hql
    ```

2.  建立指令碼：
    ```sql
    INSERT OVERWRITE DIRECTORY '${hiveOutputDirectory1}' 
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
    select devicemodel from hivesampletable limit 2;
    ```

3.  將檔案儲存至 HDFS：
    ```bash
    hdfs dfs -put countrowshive2.hql countrowshive2.hql
    ```

## <a name="submit-oozie-jobs"></a>提交 Oozie 作業
針對 ESP 叢集提交 Oozie 作業，類似於在非 ESP 的叢集中提交 Oozie 作業。

如需詳細資訊，請參閱[在 Linux 型 Azure HDInsight 上搭配 Hadoop 使用 Oozie 來定義並執行工作流程](../hdinsight-use-oozie-linux-mac.md)。

## <a name="results-from-an-oozie-job-submission"></a>提交 Oozie 作業的結果
系統會為使用者執行 Oozie 作業。 因此，Apache YARN 和 Apache Ranger 稽核記錄會顯示以模擬使用者身分執行的作業。 Oozie 作業的命令列介面輸出如下列程式碼所示：


```bash
    Job ID : 0000015-180626011240801-oozie-oozi-W
    ------------------------------------------------------------------------------------------------
    Workflow Name : map-reduce-wf
    App Path      : adl://home/user/alicetest/examples/apps/map-reduce/wf.xml
    Status        : SUCCEEDED
    Run           : 0
    User          : alicetest
    Group         : -
    Created       : 2018-06-26 19:25 GMT
    Started       : 2018-06-26 19:25 GMT
    Last Modified : 2018-06-26 19:30 GMT
    Ended         : 2018-06-26 19:30 GMT
    CoordAction ID: -
    
    Actions
    ------------------------------------------------------------------------------------------------
    ID                      Status  Ext ID          ExtStatus   ErrCode
    ------------------------------------------------------------------------------------------------
    0000015-180626011240801-oozie-oozi-W@:start:    OK  -           OK      -
    ------------------------------------------------------------------------------------------------
    0000015-180626011240801-oozie-oozi-W@mr-test    OK  job_1529975666160_0051  SUCCEEDED   -
    ------------------------------------------------------------------------------------------------
    0000015-180626011240801-oozie-oozi-W@myHive2    OK  job_1529975666160_0053  SUCCEEDED   -
    ------------------------------------------------------------------------------------------------
    0000015-180626011240801-oozie-oozi-W@myHive OK  job_1529975666160_0055  SUCCEEDED   -
    ------------------------------------------------------------------------------------------------
    0000015-180626011240801-oozie-oozi-W@end    OK  -           OK      -
    -----------------------------------------------------------------------------------------------
```

Hive server 2 動作的 Ranger 稽核記錄會顯示為使用者執行動作的 Oozie。 只有叢集管理員可以看到 Ranger 和 YARN 檢視。

## <a name="configure-user-authorization-in-oozie"></a>設定 Oozie 中的使用者授權
Oozie 本身即具有使用者授權設定，可以防止使用者停止或刪除其他使用者的作業。 若要啟用此設定，請將 `oozie.service.AuthorizationService.security.enabled` 設定為 `true`。 

如需詳細資訊，請參閱 [Oozie 安裝和設定](https://oozie.apache.org/docs/3.2.0-incubating/AG_Install.html)。

針對無法使用或不支援 Ranger 外掛程式的元件 (例如 Hive server 1)，則只能進行粗略的 HDFS 授權。 細微的授權僅透過 Ranger 外掛程式提供。

## <a name="get-the-oozie-web-ui"></a>取得 Oozie Web UI
Oozie Web UI 可讓您用網頁檢視叢集上 Oozie 作業的狀態。 若要取得 Web UI，請在 ESP 叢集中執行下列步驟：

1. 新增[邊緣節點](../hdinsight-apps-use-edge-node.md)，並啟用 [SSH Kerberos 驗證](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 遵循 [Oozie Web UI](../hdinsight-use-oozie-linux-mac.md) 的步驟，啟用邊緣節點的 SSH 通道，並存取 Web UI。

## <a name="next-steps"></a>後續步驟
* [在 Linux 型 Azure HDInsight 上搭配 Hadoop 使用 Oozie 來定義並執行工作流程](../hdinsight-use-oozie-linux-mac.md)。
* [使用以時間為基礎的 Oozie 協調器](../hdinsight-use-oozie-coordinator-time.md)。
* [使用 SSH 連線到 HDInsight (Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md#domainjoined)。
