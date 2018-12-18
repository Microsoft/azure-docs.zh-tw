---
title: 在 Azure HDInsight Spark 上使用 Caffe 進行分散式深入學習
description: 在 Azure HDInsight Spark 上使用 Caffe 進行分散式深入學習
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/17/2017
ms.openlocfilehash: a7873996d83dbc79b4d44c58bd964c274f9c7709
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39622910"
---
# <a name="use-caffe-on-azure-hdinsight-spark-for-distributed-deep-learning"></a>在 Azure HDInsight Spark 上使用 Caffe 進行分散式深入學習


## <a name="introduction"></a>簡介

深入學習的影響遍及醫療保健到傳輸到製造商等等。 公司轉型為深度學習來解決艱難的問題，例如[影像分類](http://blogs.microsoft.com/next/2015/12/10/microsoft-researchers-win-imagenet-computer-vision-challenge/)[語音辨識](http://googleresearch.blogspot.jp/2015/08/the-neural-networks-behind-google-voice.html)、物件辨識和機器轉譯。 

有[許多受歡迎的架構](https://en.wikipedia.org/wiki/Comparison_of_deep_learning_software)，包括 [Microsoft 認知工具組](https://www.microsoft.com/en-us/research/product/cognitive-toolkit/)、[Tensorflow](https://www.tensorflow.org/)MXNet、Theano 等等。Caffe 是其中一個最著名的非符號 (必要) 類神經網路架構，並廣泛用在許多方面，包括電腦視覺。 此外，[CaffeOnSpark](http://yahoohadoop.tumblr.com/post/139916563586/caffeonspark-open-sourced-for-distributed-deep) 使用 Apache Spark 結合了 Caffe，這樣的深入學習可以輕鬆地使用在現有的 Hadoop 叢集。 您可以使用結合 Spark ETL 管線的深入學習，降低系統複雜性和延遲，以學習完整的解決方案。

[HDInsight](https://azure.microsoft.com/services/hdinsight/) 是雲端 Hadoop 供應項目，為 Spark、Hive、Hadoop、HBase、Storm、Kafka 和 ML 服務提供最佳化開放原始碼分析叢集。 HDInsight 受 99.9% SLA 的支援。 每個巨量資料技術及 ISV 應用程式都可輕鬆部署為受控叢集，以提供企業所需的安全性和監視功能。

本文示範如何為 HDInsight 叢集安裝 [Caffe on Spark](https://github.com/yahoo/CaffeOnSpark)。 本文也會使用內建 MNIST 示範，說明如何使用 HDInsight Spark 在 CPU 上使用分散式深入學習。

完成此工作有四個步驟：

1. 在所有節點上安裝必要的相依性
2. 在前端節點上建置適用於 HDInsight 的 Spark Caffe
3. 將所需的程式庫分散至所有背景工作角色節點
4. 撰寫 Caffe 模型，並以分散方式執行它。

由於 HDInsight 是 PaaS 的解決方案，它提供絕佳的平台功能 - 因此可以輕鬆執行某些工作。 在此部落格文章中經常使用的其中一個功能稱為[指令碼動作](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux)，您可以用來執行 shell 命令以自訂叢集節點 (前端節點、背景工作角色節點或邊緣節點)。

## <a name="step-1--install-the-required-dependencies-on-all-the-nodes"></a>步驟 1︰在所有節點上安裝必要的相依性

若要開始，您必須安裝相依性。 Caffe 網站和 [CaffeOnSpark 網站](https://github.com/yahoo/CaffeOnSpark/wiki/GetStarted_yarn) \(英文\) 提供了一些有用的 wiki，可安裝 Spark on YARN 模式的相依性。 HDInsight 也使用 Spark on YARN 模式。 不過，您必須為 HDInsight 平台再加入些許相依性。 因此，您會使用指令碼動作，並在所有的前端節點和背景工作角色節點上執行。 此指令碼動作需要大約 20 分鐘，因為這些相依性也取決於其他封裝。 您應該將它放在 HDInsight 叢集可以存取的某個位置，例如 GitHub 位置或預設 BLOB 儲存體帳戶。

    #!/bin/bash
    #Please be aware that installing the below will add additional 20 mins to cluster creation because of the dependencies
    #installing all dependencies, including the ones mentioned in http://caffe.berkeleyvision.org/install_apt.html, as well a few packages that are not included in HDInsight, such as gflags, glog, lmdb, numpy
    #It seems numpy will only needed during compilation time, but for safety purpose you install them on all the nodes

    sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler maven libatlas-base-dev libgflags-dev libgoogle-glog-dev liblmdb-dev build-essential  libboost-all-dev python-numpy python-scipy python-matplotlib ipython ipython-notebook python-pandas python-sympy python-nose

    #install protobuf
    wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
    sudo tar xzvf protobuf-2.5.0.tar.gz -C /tmp/
    cd /tmp/protobuf-2.5.0/
    sudo ./configure
    sudo make
    sudo make check
    sudo make install
    sudo ldconfig
    echo "protobuf installation done"


指令碼動作中有兩個步驟。 第一個步驟是安裝所有必要的程式庫。 這些程式庫包含編譯 Caffe (例如 gflags、glog) 和執行 Caffe (例如 numpy) 的必要程式庫。 您將針對 CPU 最佳化使用 libatlas，但您一律可以依照 CaffeOnSpark wiki 安裝其他最佳化程式庫，例如 MKL 或 CUDA (適用於 GPU)。

第二個步驟是在執行階段期間下載、編譯和安裝 Caffe 的 protobuf 2.5.0。 [需要](https://github.com/yahoo/CaffeOnSpark/issues/87) Protobuf 2.5.0，但這個版本不在 Ubuntu 16 以套件形式提供，因此您需要從原始程式碼編譯它。 另外在網際網路上還有一些關於如何加以編譯的資源。 如需詳細資訊，請參閱[這裡](http://jugnu-life.blogspot.com/2013/09/install-protobuf-25-on-ubuntu.html)。

若要開始，您可以僅對叢集執行此指令碼動作至所有背景工作角色節點和前端節點 (適用於 HDInsight 3.5)。 您可以在現有的叢集上執行指令碼動作，或在叢集建立期間使用指令碼動作。 如需有關指令碼動作的詳細資訊，請參閱文件[這裡](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux#view-history-promote-and-demote-script-actions)。

![安裝相依性的指令碼動作](./media/apache-spark-deep-learning-caffe/Script-Action-1.png)


## <a name="step-2-build-caffe-on-spark-for-hdinsight-on-the-head-node"></a>步驟 2︰在前端節點上建置適用於 HDInsight 的 Spark Caffe

第二個步驟是在前端節點上建立 Caffe，然後將已編譯的程式庫傳送到所有背景工作角色節點。 在此步驟中，您必須[使用 SSH 連線到前端節點](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix)。 完成之後，您必須遵循 [CaffeOnSpark 建置程序](https://github.com/yahoo/CaffeOnSpark/wiki/GetStarted_yarn) \(英文\)。 使用以下指令碼，加上幾個額外步驟，就能建置 CaffeOnSpark。 

    #!/bin/bash
    git clone https://github.com/yahoo/CaffeOnSpark.git --recursive
    export CAFFE_ON_SPARK=$(pwd)/CaffeOnSpark

    pushd ${CAFFE_ON_SPARK}/caffe-public/
    cp Makefile.config.example Makefile.config
    echo "INCLUDE_DIRS += ${JAVA_HOME}/include" >> Makefile.config
    #Below configurations might need to be updated based on actual cases. For example, if you are using GPU, or using a different BLAS library, you may want to update those settings accordingly.
    echo "CPU_ONLY := 1" >> Makefile.config
    echo "BLAS := atlas" >> Makefile.config
    echo "INCLUDE_DIRS += /usr/include/hdf5/serial/" >> Makefile.config
    echo "LIBRARY_DIRS += /usr/lib/x86_64-linux-gnu/hdf5/serial/" >> Makefile.config
    popd

    #compile CaffeOnSpark
    pushd ${CAFFE_ON_SPARK}
    #always clean up the environment before building (especially when rebuiding), or there will be errors such as "failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (proto) on project caffe-distri: An Ant BuildException has occured: exec returned: 2"
    make clean 
    #the build step usually takes 20~30 mins, since it has a lot maven dependencies
    make build 
    popd
    export LD_LIBRARY_PATH=${CAFFE_ON_SPARK}/caffe-public/distribute/lib:${CAFFE_ON_SPARK}/caffe-distri/distribute/lib

    hadoop fs -mkdir -p wasb:///projects/machine_learning/image_dataset

    ${CAFFE_ON_SPARK}/scripts/setup-mnist.sh
    hadoop fs -put -f ${CAFFE_ON_SPARK}/data/mnist_*_lmdb wasb:///projects/machine_learning/image_dataset/

    ${CAFFE_ON_SPARK}/scripts/setup-cifar10.sh
    hadoop fs -put -f ${CAFFE_ON_SPARK}/data/cifar10_*_lmdb wasb:///projects/machine_learning/image_dataset/

    #put the already compiled CaffeOnSpark libraries to wasb storage, then read back to each node using script actions. This is because CaffeOnSpark requires all the nodes have the libarries
    hadoop fs -mkdir -p /CaffeOnSpark/caffe-public/distribute/lib/
    hadoop fs -mkdir -p /CaffeOnSpark/caffe-distri/distribute/lib/
    hadoop fs -put CaffeOnSpark/caffe-distri/distribute/lib/* /CaffeOnSpark/caffe-distri/distribute/lib/
    hadoop fs -put CaffeOnSpark/caffe-public/distribute/lib/* /CaffeOnSpark/caffe-public/distribute/lib/

您可能需要執行 CaffeOnSpark 的文件顯示以外的作業。 變更為：
- 只變更為 CPU 並使用 libatlas 於此特定目的。
- 將資料集放到 Blob 儲存體，也就是可以存取所有背景工作角色節點以供稍後使用的共用位置。
- 將已編譯的 Caffe 程式庫放至 Blob 儲存體，稍後您會將這些程式庫複製到使用指令碼動作以避免額外編譯時間的所有節點。


### <a name="troubleshooting-an-ant-buildexception-has-occurred-exec-returned-2"></a>疑難排解︰發生 Ant BuildException︰exec 傳回︰2

當第一次嘗試建置 CaffeOnSpark 時，有時它會說

    failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (proto) on project caffe-distri: An Ant BuildException has occured: exec returned: 2

藉由 "make clean" 清除程式碼儲存機制，然後執行 "make build" 即可解決此問題，只要您有正確的相依性。

### <a name="troubleshooting-maven-repository-connection-time-out"></a>疑難排解︰Maven 儲存機制連線逾時

有時候 Maven 會顯示連線逾時錯誤，看起來如以下程式碼片段︰

    Retry:
    [INFO] Downloading: https://repo.maven.apache.org/maven2/com/twitter/chill_2.11/0.8.0/chill_2.11-0.8.0.jar
    Feb 01, 2017 5:14:49 AM org.apache.maven.wagon.providers.http.httpclient.impl.execchain.RetryExec execute
    INFO: I/O exception (java.net.SocketException) caught when processing request to {s}->https://repo.maven.apache.org:443: Connection timed out (Read failed)

您必須在幾分鐘後重試。


### <a name="troubleshooting-test-failure-for-caffe"></a>疑難排解︰Caffe 測試失敗

您可能會在進行 CaffeOnSpark 最終檢查時看到測試失敗。 這可能與 UTF-8 編碼方式相關，但應該不會影響 Caffe 的使用方式

    Run completed in 32 seconds, 78 milliseconds.
    Total number of tests run: 7
    Suites: completed 5, aborted 0
    Tests: succeeded 6, failed 1, canceled 0, ignored 0, pending 0
    *** 1 TEST FAILED ***

## <a name="step-3-distribute-the-required-libraries-to-all-the-worker-nodes"></a>步驟 3︰將所需的程式庫分散至所有背景工作角色節點

下一步是將程式庫 (基本上為 CaffeOnSpark/caffe-public/distribute/lib/ 和 CaffeOnSpark/caffe-distri/distribute/lib/ 中的程式庫) 分散至所有節點。 在步驟 2 中，您會將這些程式庫放在 BLOB 儲存體，且在此步驟中，您會使用指令碼動作，將它複製到所有的前端節點和背景工作角色節點。

因此，請執行指令碼動作，如下列程式碼片段所示：

    #!/bin/bash
    hadoop fs -get wasb:///CaffeOnSpark /home/changetoyourusername/

確定您需要指向您叢集所特定的正確位置)

因為在步驟 2 中，您會將它放在可供所有節點存取的 BLOB 儲存體，在此步驟中您只是將它複製到所有節點。

## <a name="step-4-compose-a-caffe-model-and-run-it-in-a-distributed-manner"></a>步驟 4：撰寫 Caffe 模型，並以分散方式執行它

在執行上述步驟之後，就會安裝 Caffe。 下一個步驟是撰寫 Caffe 模型。 

Caffe 是使用「快速架構」，其中針對撰寫模型，您只需要定義設定檔，而完全無需撰寫程式碼 (在大部分情況下)。 現在讓我們來看看那裡。 

您要訓練的模型是 MNIST 訓練的範例模型。 手寫數字的 MNIST 資料庫有 60,000 個範例的訓練集，以及 10,000 個範例的測試集。 它是一組可從 NIST 提供之較大集的子集。 數字已大小正規化且在固定大小的影像置中。 CaffeOnSpark 有一些指令碼可下載資料集，並將它轉換成正確的格式。

CaffeOnSpark 針對 MNIST 訓練提供一些網路拓樸範例。 它具有不錯的網路架構 (網路的拓撲) 分割設計和最佳化。 在此情況下，需要兩個檔案︰ 

"Solver" 檔案 (${CAFFE_ON_SPARK}/data/lenet_memory_solver.prototxt) 用於監視最佳化並產生參數更新。 例如，它會定義要使用 CPU 還是 GPU、趨勢為何，以及有多少反覆運算等等。它也會定義程式應該使用哪個神經網路拓撲 (也就是您需要的第二個檔案)。 如需 Solver 的詳細資訊，請參閱 [Caffe 文件](http://caffe.berkeleyvision.org/tutorial/solver.html) \(英文\)。

在此範例中，因為您要使用 CPU 而不是 GPU，應該將最後一行變更為︰

    # solver mode: CPU or GPU
    solver_mode: CPU

![Caffe 組態](./media/apache-spark-deep-learning-caffe/Caffe-1.png)

您可以視需要變更其他行。

第二個檔案 (${CAFFE_ON_SPARK}/data/lenet_memory_train_test.prototxt) 會定義神經網路看起來如何，以及相關的輸入和輸出檔。 您也需要更新檔案以反映訓練資料的位置。 變更 lenet_memory_train_test.prototxt (您需要指向叢集特定的正確位置) 中的下列部分︰

- 將 "file:/Users/mridul/bigml/demodl/mnist_train_lmdb" 變更為 "wasb:///projects/machine_learning/image_dataset/mnist_train_lmdb"
- 將 "file:/Users/mridul/bigml/demodl/mnist_test_lmdb/" 變更為 "wasb:///projects/machine_learning/image_dataset/mnist_test_lmdb"

![Caffe 組態](./media/apache-spark-deep-learning-caffe/Caffe-2.png)

如需有關如何定義網路的詳細資訊，請參閱 [MNIST 資料集上的 Caffe 文件](http://caffe.berkeleyvision.org/gathered/examples/mnist.html) \(英文\)

針對本文目的，您會使用此 MNIST 範例。 從前端節點執行下列命令：

    spark-submit --master yarn --deploy-mode cluster --num-executors 8 --files ${CAFFE_ON_SPARK}/data/lenet_memory_solver.prototxt,${CAFFE_ON_SPARK}/data/lenet_memory_train_test.prototxt --conf spark.driver.extraLibraryPath="${LD_LIBRARY_PATH}" --conf spark.executorEnv.LD_LIBRARY_PATH="${LD_LIBRARY_PATH}" --class com.yahoo.ml.caffe.CaffeOnSpark ${CAFFE_ON_SPARK}/caffe-grid/target/caffe-grid-0.1-SNAPSHOT-jar-with-dependencies.jar -train -features accuracy,loss -label label -conf lenet_memory_solver.prototxt -devices 1 -connection ethernet -model wasb:///mnist.model -output wasb:///mnist_features_result

上述命令會將必要的檔案 (lenet_memory_solver.prototxt) 和 (lenet_memory_train_test.prototxt) 分散到每個 YARN 容器。 此命令也會將每個 Spark 驅動程式/執行程式的相關路徑設為 LD_LIBRARY_PATH。 LD_LIBRARY_PATH 是在先前的程式碼片段中定義，並且指向包含 CaffeOnSpark 程式庫的位置。 

## <a name="monitoring-and-troubleshooting"></a>監視與疑難排解

因為您將使用 YARN 叢集模式，在此情況下，Spark 驅動程式會排定至任意容器 (和任意的背景工作角色節點)，在主控台輸出中應該只會看到如下︰

    17/02/01 23:22:16 INFO Client: Application report for application_1485916338528_0015 (state: RUNNING)

如果您想要知道發生了什麼事，通常需要取得 Spark 驅動程式的記錄檔，會有更多的資訊。 在此情況下，您需要前往 YARN UI 以尋找相關的 YARN 記錄檔。 您可以藉由此 URL 取得 YARN UI： 

    https://yourclustername.azurehdinsight.net/yarnui
   
![YARN UI](./media/apache-spark-deep-learning-caffe/YARN-UI-1.png)

您可以看看此特定應用程式配置了多少資源。 您可以按一下 [排程器] 連結，然後您會看到這個應用程式有 9 個執行中的容器。 您將要求 YARN 提供 8 個執行程式，以及另一個容器供驅動程式程序使用。 

![YARN 排程器](./media/apache-spark-deep-learning-caffe/YARN-Scheduler.png)

如果發生失敗，您要檢查驅動程式記錄檔或容器記錄檔。 針對驅動程式記錄檔，您可以按一下 YARN UI 中的應用程式識別碼，然後按一下 [記錄] 按鈕。 驅動程式記錄檔會寫入 stderr。

![YARN UI 2](./media/apache-spark-deep-learning-caffe/YARN-UI-2.png)

例如，您可能會在下方看到一些來自驅動程式記錄檔的錯誤，表示配置太多的執行程式。

    17/02/01 07:26:06 ERROR ApplicationMaster: User class threw exception: java.lang.IllegalStateException: Insufficient training data. Please adjust hyperparameters or increase dataset.
    java.lang.IllegalStateException: Insufficient training data. Please adjust hyperparameters or increase dataset.
        at com.yahoo.ml.caffe.CaffeOnSpark.trainWithValidation(CaffeOnSpark.scala:261)
        at com.yahoo.ml.caffe.CaffeOnSpark$.main(CaffeOnSpark.scala:42)
        at com.yahoo.ml.caffe.CaffeOnSpark.main(CaffeOnSpark.scala)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:627)

有時候，可能會在執行程式而不是驅動程式發生此問題。 在此情況下，您需要檢查容器記錄檔。 您一律可以取得容器記錄檔，然後取得失敗的容器。 例如，您在執行 Caffe 時可能會遇到此失敗。

    17/02/01 07:12:05 WARN YarnAllocator: Container marked as failed: container_1485916338528_0008_05_000005 on host: 10.0.0.14. Exit status: 134. Diagnostics: Exception from container-launch.
    Container id: container_1485916338528_0008_05_000005
    Exit code: 134
    Exception message: /bin/bash: line 1: 12230 Aborted                 (core dumped) LD_LIBRARY_PATH=/usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64:/home/xiaoyzhu/CaffeOnSpark/caffe-public/distribute/lib:/home/xiaoyzhu/CaffeOnSpark/caffe-distri/distribute/lib /usr/lib/jvm/java-8-openjdk-amd64/bin/java -server -Xmx4608m '-Dhdp.version=' '-Detwlogger.component=sparkexecutor' '-DlogFilter.filename=SparkLogFilters.xml' '-DpatternGroup.filename=SparkPatternGroups.xml' '-Dlog4jspark.root.logger=INFO,console,RFA,ETW,Anonymizer' '-Dlog4jspark.log.dir=/var/log/sparkapp/${user.name}' '-Dlog4jspark.log.file=sparkexecutor.log' '-Dlog4j.configuration=file:/usr/hdp/current/spark2-client/conf/log4j.properties' '-Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl' -Djava.io.tmpdir=/mnt/resource/hadoop/yarn/local/usercache/xiaoyzhu/appcache/application_1485916338528_0008/container_1485916338528_0008_05_000005/tmp '-Dspark.driver.port=43942' '-Dspark.history.ui.port=18080' '-Dspark.ui.port=0' -Dspark.yarn.app.container.log.dir=/mnt/resource/hadoop/yarn/log/application_1485916338528_0008/container_1485916338528_0008_05_000005 -XX:OnOutOfMemoryError='kill %p' org.apache.spark.executor.CoarseGrainedExecutorBackend --driver-url spark://CoarseGrainedScheduler@10.0.0.13:43942 --executor-id 4 --hostname 10.0.0.14 --cores 3 --app-id application_1485916338528_0008 --user-class-path file:/mnt/resource/hadoop/yarn/local/usercache/xiaoyzhu/appcache/application_1485916338528_0008/container_1485916338528_0008_05_000005/__app__.jar > /mnt/resource/hadoop/yarn/log/application_1485916338528_0008/container_1485916338528_0008_05_000005/stdout 2> /mnt/resource/hadoop/yarn/log/application_1485916338528_0008/container_1485916338528_0008_05_000005/stderr

    Stack trace: ExitCodeException exitCode=134: /bin/bash: line 1: 12230 Aborted                 (core dumped) LD_LIBRARY_PATH=/usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64:/home/xiaoyzhu/CaffeOnSpark/caffe-public/distribute/lib:/home/xiaoyzhu/CaffeOnSpark/caffe-distri/distribute/lib /usr/lib/jvm/java-8-openjdk-amd64/bin/java -server -Xmx4608m '-Dhdp.version=' '-Detwlogger.component=sparkexecutor' '-DlogFilter.filename=SparkLogFilters.xml' '-DpatternGroup.filename=SparkPatternGroups.xml' '-Dlog4jspark.root.logger=INFO,console,RFA,ETW,Anonymizer' '-Dlog4jspark.log.dir=/var/log/sparkapp/${user.name}' '-Dlog4jspark.log.file=sparkexecutor.log' '-Dlog4j.configuration=file:/usr/hdp/current/spark2-client/conf/log4j.properties' '-Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl' -Djava.io.tmpdir=/mnt/resource/hadoop/yarn/local/usercache/xiaoyzhu/appcache/application_1485916338528_0008/container_1485916338528_0008_05_000005/tmp '-Dspark.driver.port=43942' '-Dspark.history.ui.port=18080' '-Dspark.ui.port=0' -Dspark.yarn.app.container.log.dir=/mnt/resource/hadoop/yarn/log/application_1485916338528_0008/container_1485916338528_0008_05_000005 -XX:OnOutOfMemoryError='kill %p' org.apache.spark.executor.CoarseGrainedExecutorBackend --driver-url spark://CoarseGrainedScheduler@10.0.0.13:43942 --executor-id 4 --hostname 10.0.0.14 --cores 3 --app-id application_1485916338528_0008 --user-class-path file:/mnt/resource/hadoop/yarn/local/usercache/xiaoyzhu/appcache/application_1485916338528_0008/container_1485916338528_0008_05_000005/__app__.jar > /mnt/resource/hadoop/yarn/log/application_1485916338528_0008/container_1485916338528_0008_05_000005/stdout 2> /mnt/resource/hadoop/yarn/log/application_1485916338528_0008/container_1485916338528_0008_05_000005/stderr

        at org.apache.hadoop.util.Shell.runCommand(Shell.java:933)
        at org.apache.hadoop.util.Shell.run(Shell.java:844)
        at org.apache.hadoop.util.Shell$ShellCommandExecutor.execute(Shell.java:1123)
        at org.apache.hadoop.yarn.server.nodemanager.DefaultContainerExecutor.launchContainer(DefaultContainerExecutor.java:225)
        at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:317)
        at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:83)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)


    Container exited with a non-zero exit code 134

在此情況下，您需要取得失敗的容器識別碼 (在上述案例中，它是 container_1485916338528_0008_05_000005)。 然後您需要執行 

    yarn logs -containerId container_1485916338528_0008_03_000005

自前端節點。 在檢查容器失敗之後，它是由使用 lenet_memory_solver.prototxt 中的 GPU 模式 (您應該改用 CPU 模式) 所引起。

    17/02/01 07:10:48 INFO LMDB: Batch size:100
    WARNING: Logging before InitGoogleLogging() is written to STDERR
    F0201 07:10:48.309725 11624 common.cpp:79] Cannot use GPU in CPU-only Caffe: check mode.


## <a name="getting-results"></a>取得結果

因為您將配置 8 個執行程式，且網路拓撲很簡單，應該只需花費大約 30 分鐘的時間執行結果。 從命令列中，您可以看到此模型已放到 wasb:///mnist.model，並將結果放到名為 wasb:///mnist_features_result 的資料夾。

您可以取得結果，方法為執行

    hadoop fs -cat hdfs:///mnist_features_result/*

結果如下：

    {"SampleID":"00009597","accuracy":[1.0],"loss":[0.028171852],"label":[2.0]}
    {"SampleID":"00009598","accuracy":[1.0],"loss":[0.028171852],"label":[6.0]}
    {"SampleID":"00009599","accuracy":[1.0],"loss":[0.028171852],"label":[1.0]}
    {"SampleID":"00009600","accuracy":[0.97],"loss":[0.0677709],"label":[5.0]}
    {"SampleID":"00009601","accuracy":[0.97],"loss":[0.0677709],"label":[0.0]}
    {"SampleID":"00009602","accuracy":[0.97],"loss":[0.0677709],"label":[1.0]}
    {"SampleID":"00009603","accuracy":[0.97],"loss":[0.0677709],"label":[2.0]}
    {"SampleID":"00009604","accuracy":[0.97],"loss":[0.0677709],"label":[3.0]}
    {"SampleID":"00009605","accuracy":[0.97],"loss":[0.0677709],"label":[4.0]}

SampleID 代表 MNIST 資料集的識別碼，且標籤是此模型識別的數字。


## <a name="conclusion"></a>結論

在本文件中，您已嘗試執行簡單的範例來安裝 CaffeOnSpark。 HDInsight 是完全受控雲端分散式計算平台，並且是在大型資料集上執行機器學習和進階分析工作負載的最佳位置，而針對分散式深入學習，您可以使用 Caffe on HDInsight Spark 來執行深入學習工作。


## <a name="seealso"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和機器學習：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習：使用 HDInsight 中的 Spark 來預測食物檢查結果](apache-spark-machine-learning-mllib-ipython.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)

