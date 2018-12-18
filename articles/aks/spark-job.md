---
title: 透過 Azure Kubernetes Service (AKS) 執行 Apache Spark 作業
description: 使用 Azure Kubernetes Service (AKS) 執行 Apache Spark 作業
services: container-service
author: lenadroid
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 03/15/2018
ms.author: alehall
ms.custom: mvc
ms.openlocfilehash: cb23c21fd22a35a3e8a5920a94aa5a89fe966cfa
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33934940"
---
# <a name="running-apache-spark-jobs-on-aks"></a>在 AKS 上執行 Apache Spark 作業

[Apache Spark][apache-spark] 是進行大規模資料處理的快速引擎。 從 [Spark 2.3.0 版][spark-latest-release] 開始，Apache Spark 支援與 Kubernetes 叢集的原生整合。 Azure Kubernetes Service (AKS) 是在 Azure 中執行的受控 Kubernetes 環境。 本文件詳述在 Azure Kubernetes Service (AKS) 叢集上準備和執行 Apache Spark 作業的做法。

## <a name="prerequisites"></a>先決條件

為了要完成本文中的步驟，您需要下列項目。

* 對 Kubernetes 和 [Apache Spark][spark-quickstart] 的基本了解。
* [Docker Hub][docker-hub] 帳戶，或 [Azure Container Registry][acr-create]。
* 在您的開發系統上[安裝][azure-cli] Azure CLI。
* 在您的系統上安裝 [JDK 8][java-install]。
* 在您的系統上安裝 SBT ([Scala 建置工具][sbt-install])。
* 在您的系統上安裝 Git 命令列工具。

## <a name="create-an-aks-cluster"></a>建立 AKS 叢集

Spark 會用於大規模的資料處理，而且需要將 Kubernetes 節點的大小調整為符合 Spark 資源需求。 針對您的 Azure Kubernetes Service (AKS) 節點，我們建議使用 `Standard_D3_v2` 大小的最小值。

如果您需要符合此最小值建議的 AKS 叢集，請執行下列命令。

建立叢集的資源群組。

```azurecli
az group create --name mySparkCluster --location eastus
```

以大小為 `Standard_D3_v2` 的節點建立 AKS 叢集。

```azurecli
az aks create --resource-group mySparkCluster --name mySparkCluster --node-vm-size Standard_D3_v2
```

連線到 AKS 叢集。

```azurecli
az aks get-credentials --resource-group mySparkCluster --name mySparkCluster
```

如果您使用 Azure Container Registry (ACR) 來儲存容器映像，請設定 AKS 與 ACR 之間的驗證。 請參閱 [ACR 驗證文件][acr-aks]以取得這些步驟。

## <a name="build-the-spark-source"></a>建置 Spark 來源

在 AKS 叢集上執行 Spark 作業之前，您需要建置 Spark 原始程式碼，並將其封裝至容器映像。 Spark 來源包含可用來完成此程序的指令碼。

將 Spark 專案存放庫複製到您的開發系統。

```bash
git clone -b branch-2.3 https://github.com/apache/spark
```

變更為已複製之存放庫的目錄，然後將 Spark 來源的路徑儲存為變數。

```bash
cd spark
sparkdir=$(pwd)
```

如果您已安裝多個 JDK 版本，針對目前的工作階段設定 `JAVA_HOME` 使用第 8 版。

```bash
export JAVA_HOME=`/usr/libexec/java_home -d 64 -v "1.8*"`
```

執行下列命令來建置具有 Kubernetes 支援的 Spark 原始程式碼。

```bash
./build/mvn -Pkubernetes -DskipTests clean package
```

下列命令會建立 Spark 容器映像，並將之推送到容器映像登錄。 請使用容器登錄的名稱取代 `registry.example.com`，並使用想要使用的標記取代 `v1`。 如果使用 Docker Hub，這個值會是登錄名稱。 如果使用 Azure Container Registry (ACR)，這個值會是 ACR 登入伺服器名稱。

```bash
REGISTRY_NAME=registry.example.com
REGISTRY_TAG=v1
```

```bash
./bin/docker-image-tool.sh -r $REGISTRY_NAME -t $REGISTRY_TAG build
```

將容器映像推送到您的容器映像登錄。

```bash
./bin/docker-image-tool.sh -r $REGISTRY_NAME -t $REGISTRY_TAG push
```

## <a name="prepare-a-spark-job"></a>準備 Spark 作業

接下來，準備 Spark 作業。 jar 檔案是用來保存 Spark 作業，當執行 `spark-submit` 命令時需要該檔案。 可以透過公開 URL 存取 jar，或是預先封裝在容器映像內。 在此範例中，會建立範例 jar 來計算 Pi 的值。 然後會將這個 jar 上傳至 Azure 儲存體。 如果您有現有的 jar，可以自由取代

建立目錄，您可以在其中為 Spark 作業建立專案。

```bash
mkdir myprojects
cd myprojects
```

從範本建立新的 Scala 專案。

```bash
sbt new sbt/scala-seed.g8
```

出現提示時，輸入 `SparkPi` 作為專案名稱。

```bash
name [Scala Seed Project]: SparkPi
```

瀏覽至新建立的專案目錄。

```bash
cd sparkpi
```

執行下列命令以新增 SBT 外掛程式，該外掛程式可以將專案封裝為 jar 檔案。

```bash
touch project/assembly.sbt
echo 'addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.6")' >> project/assembly.sbt
```

執行這些命令以將範例程式碼複製到新建立的專案，並新增所有必要的相依性。

```bash
EXAMPLESDIR="src/main/scala/org/apache/spark/examples"
mkdir -p $EXAMPLESDIR
cp $sparkdir/examples/$EXAMPLESDIR/SparkPi.scala $EXAMPLESDIR/SparkPi.scala

cat <<EOT >> build.sbt
// https://mvnrepository.com/artifact/org.apache.spark/spark-sql
libraryDependencies += "org.apache.spark" %% "spark-sql" % "2.3.0" % "provided"
EOT

sed -ie 's/scalaVersion.*/scalaVersion := "2.11.11",/' build.sbt
sed -ie 's/name.*/name := "SparkPi",/' build.sbt
```

若要將專案封裝到 jar，請執行下列命令。

```bash
sbt assembly
```

成功封裝之後，您應該會看到類似以下的輸出。

```bash
[info] Packaging /Users/me/myprojects/sparkpi/target/scala-2.11/SparkPi-assembly-0.1.0-SNAPSHOT.jar ...
[info] Done packaging.
[success] Total time: 10 s, completed Mar 6, 2018 11:07:54 AM
```

## <a name="copy-job-to-storage"></a>將作業複製到儲存體

建立 Azure 儲存體帳戶和容器以保存 jar 檔案。

```azurecli
RESOURCE_GROUP=sparkdemo
STORAGE_ACCT=sparkdemo$RANDOM
az group create --name $RESOURCE_GROUP --location eastus
az storage account create --resource-group $RESOURCE_GROUP --name $STORAGE_ACCT --sku Standard_LRS
export AZURE_STORAGE_CONNECTION_STRING=`az storage account show-connection-string --resource-group $RESOURCE_GROUP --name $STORAGE_ACCT -o tsv`
```

使用下列命令將 jar 檔案上傳至 Azure 儲存體帳戶。

```bash
CONTAINER_NAME=jars
BLOB_NAME=SparkPi-assembly-0.1.0-SNAPSHOT.jar
FILE_TO_UPLOAD=target/scala-2.11/SparkPi-assembly-0.1.0-SNAPSHOT.jar

echo "Creating the container..."
az storage container create --name $CONTAINER_NAME
az storage container set-permission --name $CONTAINER_NAME --public-access blob

echo "Uploading the file..."
az storage blob upload --container-name $CONTAINER_NAME --file $FILE_TO_UPLOAD --name $BLOB_NAME

jarUrl=$(az storage blob url --container-name $CONTAINER_NAME --name $BLOB_NAME | tr -d '"')
```

變數 `jarUrl` 現在包含 jar 檔案的可公開存取路徑。

## <a name="submit-a-spark-job"></a>提交 Spark 作業

使用下列程式碼，在另一個命令列中啟動 kube proxy。

```bash
kubectl proxy
```

瀏覽回到 Spark 存放庫的根目錄。

```bash
cd $sparkdir
```

使用 `spark-submit` 提交作業。

```bash
./bin/spark-submit \
  --master k8s://http://127.0.0.1:8001 \
  --deploy-mode cluster \
  --name spark-pi \
  --class org.apache.spark.examples.SparkPi \
  --conf spark.executor.instances=3 \
  --conf spark.kubernetes.container.image=$REGISTRY_NAME/spark:$REGISTRY_TAG \
  $jarUrl
```

這項作業會啟動 Spark 作業，它會將作業狀態串流至您的殼層工作階段。 執行作業時，您可以使用 kubectl get pods 命令查看 Spark 驅動程式 Pod 和執行程式 Pod。 開啟第二個終端機工作階段來執行下列命令。

```console
$ kubectl get pods

NAME                                               READY     STATUS     RESTARTS   AGE
spark-pi-2232778d0f663768ab27edc35cb73040-driver   1/1       Running    0          16s
spark-pi-2232778d0f663768ab27edc35cb73040-exec-1   0/1       Init:0/1   0          4s
spark-pi-2232778d0f663768ab27edc35cb73040-exec-2   0/1       Init:0/1   0          4s
spark-pi-2232778d0f663768ab27edc35cb73040-exec-3   0/1       Init:0/1   0          4s
```

執行作業時，您也可以存取 Spark UI。 在第二個終端機工作階段中，使用 `kubectl port-forward` 命令來提供 Spark UI 的存取權。

```bash
kubectl port-forward spark-pi-2232778d0f663768ab27edc35cb73040-driver 4040:4040
```

若要存取 Spark UI，請在瀏覽器中開啟位址 `127.0.0.1:4040`。

![Spark UI](media/aks-spark-job/spark-ui.png)

## <a name="get-job-results-and-logs"></a>取得作業結果和記錄

作業完成之後，驅動程式 Pod 會處於「已完成」狀態。 使用下列命令取得 Pod 的名稱。

```bash
kubectl get pods --show-all
```

輸出：

```bash
NAME                                               READY     STATUS      RESTARTS   AGE
spark-pi-2232778d0f663768ab27edc35cb73040-driver   0/1       Completed   0          1m
```

使用 `kubectl logs` 命令以從 Spark 驅動程式 Pod 取得記錄。 以您的驅動程式 Pod 的名稱取代 Pod 名稱。

```bash
kubectl logs spark-pi-2232778d0f663768ab27edc35cb73040-driver
```

在這些記錄中，您可以看到 Spark 作業的結果，那就是 Pi 的值。

```bash
Pi is roughly 3.152155760778804
```

## <a name="package-jar-with-container-image"></a>使用容器映像封裝 jar

在上述範例中，Spark jar 檔案已上傳至 Azure 儲存體。 另一個選項是將 jar 檔案封裝至自訂建置的 Docker 映像。

若要這樣做，請找出 Spark 映像 (位於 `$sparkdir/resource-managers/kubernetes/docker/src/main/dockerfiles/spark/` 目錄) 的 `dockerfile`。 為 `WORKDIR` 與 `ENTRYPOINT` 宣告之間某個位置的 Spark 作業 `jar` 新增 `ADD` 陳述式。

將 jar 路徑更新為您的開發系統上 `SparkPi-assembly-0.1.0-SNAPSHOT.jar` 檔案的位置。 您也可以使用自己的自訂 jar 檔案。

```bash
WORKDIR /opt/spark/work-dir

ADD /path/to/SparkPi-assembly-0.1.0-SNAPSHOT.jar SparkPi-assembly-0.1.0-SNAPSHOT.jar

ENTRYPOINT [ "/opt/entrypoint.sh" ]
```

使用隨附的 Spark 指令碼建置並推送映像。

```bash
./bin/docker-image-tool.sh -r <your container repository name> -t <tag> build
./bin/docker-image-tool.sh -r <your container repository name> -t <tag> push
```

執行作業時，並非指出遠端 jar URL，而是可以將 `local://` 配置用於 Docker 映像中 jar 檔案的路徑。

```bash
./bin/spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=3 \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///opt/spark/work-dir/<your-jar-name>.jar
```

> [!WARNING]
> 引述自 Spark [文件][spark-docs]：「Kubernetes 排程器目前為實驗性。 在未來的版本中，可組態、容器映像和進入點可能會有行為上的變更。」

## <a name="next-steps"></a>後續步驟

如需詳細資料，請參閱 Spark 文件。

> [!div class="nextstepaction"]
> [Spark 文件][spark-docs]

<!-- LINKS - external -->
[apache-spark]: https://spark.apache.org/
[docker-hub]: https://docs.docker.com/docker-hub/
[java-install]: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[sbt-install]: https://www.scala-sbt.org/1.0/docs/Setup.html
[spark-docs]: https://spark.apache.org/docs/latest/running-on-kubernetes.html
[spark-latest-release]: https://spark.apache.org/releases/spark-release-2-3-0.html
[spark-quickstart]: https://spark.apache.org/docs/latest/quick-start.html


<!-- LINKS - internal -->
[acr-aks]: https://docs.microsoft.com/azure/container-registry/container-registry-auth-aks
[acr-create]: https://docs.microsoft.com/azure/container-registry/container-registry-get-started-azure-cli
[aks-quickstart]: https://docs.microsoft.com/azure/aks/
[azure-cli]: https://docs.microsoft.com/cli/azure/?view=azure-cli-latest
[storage-account]: https://docs.microsoft.com/azure/storage/common/storage-azure-cli
