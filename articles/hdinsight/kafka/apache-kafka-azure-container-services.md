---
title: 使用 Azure Kubernetes Service 搭配 HDInsight 上的 Kafka
description: 了解如何從 Azure Kubernetes Service (AKS) 中裝載的容器映像使用 HDInsight 上的 Kafka。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/07/2018
ms.openlocfilehash: 9170a771b50af4a031d747df547152beda52d01f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998299"
---
# <a name="use-azure-kubernetes-service-with-kafka-on-hdinsight"></a>使用 Azure Kubernetes Service 搭配 HDInsight 上的 Kafka

了解如何使用 Azure Kubernetes Service (AKS) 搭配 HDInsight 叢集上的 Kafka。 本文件中的步驟使用 AKS 中裝載的 Node.js 應用程式來確認與 Kafka 的連線能力。 此應用程式使用 [kafka-node](https://www.npmjs.com/package/kafka-node) 套件來與 Kafka 通訊。 它會使用 [Socket.io](https://socket.io/)，在瀏覽器用戶端與 AKS 中裝載的後端之間進行事件驅動傳訊。

[Apache Kafka](https://kafka.apache.org) 是開放原始碼分散式串流平台，可用來建置即時串流資料管線和應用程式。 Azure Kubernetes Service 會管理您裝載的 Kubernetes 環境，並讓您快速且輕鬆地部署容器化的應用程式。 使用 Azure 虛擬網路，您可以連接這兩項服務。

> [!NOTE]
> 本文件的重點在於讓 Azure Kubernetes Service 能與 HDInsight 上的 Kafka 通訊所需的步驟。 範例本身只是基本 Kafka 用戶端，用以證明此組態能運作。

## <a name="prerequisites"></a>必要條件

* [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
* Azure 訂用帳戶

本文件假設您熟悉建立和使用下列 Azure 服務：

* HDInsight 上的 Kafka
* Azure Kubernetes Service
* Azure 虛擬網路

本文件也會假設您已逐步進行 [Azure Kubernetes Service 教學課程](../../aks/tutorial-kubernetes-prepare-app.md)。 本教學課程會建立容器服務、建立 Kubernetes 叢集中、容器登錄，以及設定 `kubectl` 公用程式。

## <a name="architecture"></a>架構

### <a name="network-topology"></a>網路拓撲

HDInsight 和 AKS 均使用 Azure 虛擬網路作為計算資源的容器。 若要啟用 HDInsight 與 AKS 之間的通訊，您必須啟用其網路之間的通訊。 本文件中的步驟使用網路的虛擬網路對等互連。 其他連線 (例如 VPN) 也應該能運作。 如需對等互連詳細資訊，請參閱[虛擬網路對等互連](../../virtual-network/virtual-network-peering-overview.md)文件。


下圖說明本文件中使用的網路拓撲：

![一個虛擬網路中的 HDInsight、另一個虛擬網路中的 AKS，以及使用對等互連連線的網路](./media/apache-kafka-azure-container-services/kafka-aks-architecture.png)

> [!IMPORTANT]
> 系統不會啟用對等互連網路之間的名稱解析，所以會使用 IP 定址。 根據預設，Kafka on HDInsight 已設定為在用戶端連線時傳回主機名稱，而不是 IP 位址。 本文件中的步驟將 Kafka 修改為使用 IP 通告。

## <a name="create-an-azure-kubernetes-service-aks"></a>建立 Azure Kubernetes Service (AKS)

如果您還沒有 AKS 叢集，請使用下列其中一份文件來了解如何建立 AKS 叢集：

* [部署 Azure Kubernetes Service (AKS) 叢集 - 入口網站](../../aks/kubernetes-walkthrough-portal.md)
* [部署 Azure Kubernetes Service (AKS) 叢集 - CLI](../../aks/kubernetes-walkthrough.md)

> [!NOTE]
> AKS 會在安裝期間建立虛擬網路。 此網路會對等互連至在下一節中針對 HDInsight 建立的網路。

## <a name="configure-virtual-network-peering"></a>設定虛擬網路對等互連

1. 從 [Azure 入口網站](https://portal.azure.com)，選取 [資源群組]，然後尋找包含 AKS 叢集之虛擬網路的資源群組。 資源群組名稱為 `MC_<resourcegroup>_<akscluster>_<location>`。 `resourcegroup` 和 `akscluster` 項目是您在其中建立叢集的資源群組名稱，以及叢集的名稱。 `location` 是在其中建立叢集的位置。

2. 在資源群組中，選取 [虛擬網路] 資源。

3. 選取 [位址空間]。 請記下所列的位址空間。

4. 若要建立 HDInsight 的虛擬網路，請選取 [+ 建立資源]、[網路]，然後選取 [虛擬網路]。

    > [!IMPORTANT]
    > 在輸入新虛擬網路的值時，您使用的位址空間不得與 AKS 叢集網路所使用的位址空間重疊。

    針對您用於 AKS 叢集的虛擬網路使用相同的 [位置]。

    等到虛擬網路建立後，再繼續下一個步驟。

5. 若要設定 HDInsight 網路與 AKS 叢集網路之間的對等互連，請選取虛擬網路，然後選取 [對等互連]。 選取 [+ 新增] 並使用下列值來填入表單：

    * __名稱__︰輸入此對等互連組態的唯一名稱。
    * __虛擬網路__：使用此欄位來選取 **AKS 叢集**的虛擬網路。

    讓所有其他欄位保留預設值，然後選取 [確定] 來設定對等互連。

6. 若要設定 AKS 叢集網路與 HDInsight 網路之間的對等互連，請選取 [AKS 叢集虛擬網路]，然後選取 [對等互連]。 選取 [+ 新增] 並使用下列值來填入表單：

    * __名稱__︰輸入此對等互連組態的唯一名稱。
    * __虛擬網路__：使用此欄位來選取 __HDInsight 叢集__的虛擬網路。

    讓所有其他欄位保留預設值，然後選取 [確定] 來設定對等互連。

## <a name="install-kafka-on-hdinsight"></a>安裝 Kafka on HDInsight

在建立 Kafka on HDInsight 叢集時，您必須加入稍早為 HDInsight 建立的虛擬網路。 如需有關建立 Kafka 叢集的詳細資訊，請參閱[建立 Kafka 叢集](apache-kafka-get-started.md)文件。

> [!IMPORTANT]
> 在建立叢集時，您必須使用 [進階設定] 來加入您為 HDInsight 建立的虛擬網路。

## <a name="configure-kafka-ip-advertising"></a>設定 Kafka IP 通告

使用下列步驟將 Kafka 設定為通告 IP 位址 (而不是網域名稱)︰

1. 使用網頁瀏覽器，移至 https://CLUSTERNAME.azurehdinsight.net。 將 __CLUSTERNAME__ 取代為 HDInsight 叢集上 Kafka 的名稱。

    出現提示時，請使用叢集的 HTTPS 使用者名稱和密碼。 此時會顯示叢集的 Ambari Web UI。

2. 若要檢視 Kafka 上的資訊，請從左邊的清單選取 [Kafka]。

    ![反白顯示 Kafka 的服務清單](./media/apache-kafka-azure-container-services/select-kafka-service.png)

3. 若要檢視 Kafka 組態，請從正上方選取 [Configs (設定)]。

    ![Kafka 的 Configs (設定) 連結](./media/apache-kafka-azure-container-services/select-kafka-config.png)

4. 若要找出 __kafka-env__ 組態，請在右上角的 [Filter (篩選)] 欄位中輸入 `kafka-env`。

    ![Kafka 組態，找出 kafka-env](./media/apache-kafka-azure-container-services/search-for-kafka-env.png)

5. 若要設定 Kafka 公告 IP 位址，請在 [kafka-env-template] 欄位的底部加入下列文字︰

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

6. 若要設定 Kafka 接聽的介面，請在右上角的 [Filter (篩選)] 欄位中輸入 `listeners`。

7. 若要設定 Kafka 在所有網路介面上接聽，請將 [listeners (接聽程式)] 欄位的值變更為 `PLAINTEXT://0.0.0.0:9092`。

8. 若要儲存組態變更，請使用 [Save (儲存)] 按鈕。 輸入描述變更的文字訊息。 儲存變更後，請選取 [OK (確定)]。

    ![儲存組態按鈕](./media/apache-kafka-azure-container-services/save-button.png)

9. 若要避免重新啟動 Kafka 時發生錯誤，請使用 [Service Actions (服務動作)] 按鈕，然後選取 [Turn On Maintenance Mode (開啟維護模式)]。 選取 [OK (確定)] 以完成此作業。

    ![服務動作，反白顯示開啟維護](./media/apache-kafka-azure-container-services/turn-on-maintenance-mode.png)

10. 若要重新啟動 Kafka，請使用 [Restart (重新啟動)] 按鈕，然後選取 [Restart All Affected (重新啟動所有受影響項目)]。 確認重新啟動，然後在作業完成之後使用 [OK (確定)] 按鈕。

    ![重新啟動按鈕，反白顯示重新啟動所有受影響項目](./media/apache-kafka-azure-container-services/restart-button.png)

11. 若要停用維護模式，請使用 [Service Actions (服務動作)] 按鈕，然後選取 [Turn Off Maintenance Mode (關閉維護模式)]。 選取 [OK (確定)] 以完成此作業。

## <a name="test-the-configuration"></a>測試組態

此時，Kafka 與 Azure Kubernetes Service 會透過對等互連的虛擬網路進行通訊。 若要測試此連線，請使用下列步驟：

1. 建立測試應用程式所使用的 Kafka 主題。 如需有關建立 Kafka 主題的詳細資訊，請參閱[建立 Kafka 叢集](apache-kafka-get-started.md)文件。

2. 從 [https://github.com/Blackmist/Kafka-AKS-Test](https://github.com/Blackmist/Kafka-AKS-Test) 下載範例應用程式。

3. 編輯 `index.js` 檔案並變更下列幾行：

    * `var topic = 'mytopic'`：將 `mytopic` 取代為此應用程式使用的 Kafka 主題名稱。
    * `var brokerHost = '176.16.0.13:9092`：將 `176.16.0.13` 取代為您叢集的其中一部訊息代理程式主機的內部 IP 位址。

        若要尋找叢集中訊息代理程式主機 (背景工作節點) 的內部 IP 位址，請參閱 [Ambari REST API](../hdinsight-hadoop-manage-ambari-rest-api.md#example-get-the-internal-ip-address-of-cluster-nodes)文件。 挑選其中一個項目的 IP 位址，其中網域名稱開頭為 `wn`。

4. 從命令列，在 `src` 目錄中安裝相依項目，並使用 Docker 來建置可供部署的映像：

    ```bash
    docker build -t kafka-aks-test .
    ```

    > [!NOTE]
    > 此應用程式所需的套件會簽入存放庫中，因此您不需要使用 `npm` 公用程式來安裝它們。

5. 登入您的 Azure Container Registry (ACR) 並尋找 loginServer 名稱：

    ```bash
    az acr login --name <acrName>
    az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
    ```

    > [!NOTE]
    > 如果您不知道 Azure Container Registry 名稱，或是不熟悉使用 Azure CLI 來處理 Azure Kubernetes Service，請參閱 [AKS 教學課程](../../aks/tutorial-kubernetes-prepare-app.md)。

6. 以 ACR 的 loginServer 標記本機 `kafka-aks-test` 映像。 此外，將 `:v1` 新增至尾端以表示映像版本：

    ```bash
    docker tag kafka-aks-test <acrLoginServer>/kafka-aks-test:v1
    ```

7. 將映像推送至登錄：

    ```bash
    docker push <acrLoginServer>/kafka-aks-test:v1
    ```
    此作業需要幾分鐘才能完成。

8. 編輯 Kubernetes 資訊清單檔案 (`kafka-aks-test.yaml`)，並將 `microsoft` 取代為在步驟 4 擷取的 ACR loginServer 名稱。

9. 使用下列命令來部署資訊清單中的應用程式設定：

    ```bash
    kubectl create -f kafka-aks-test.yaml
    ```

10. 使用下列命令來監看應用程式的 `EXTERNAL-IP`：

    ```bash
    kubectl get service kafka-aks-test --watch
    ```

    指派外部 IP 位址後，請使用 __CTRL + C__ 結束監看

11. 開啟 Web 瀏覽器並輸入服務的外部 IP 位址。 您會抵達類似下圖的頁面：

    ![網頁的影像](./media/apache-kafka-azure-container-services/test-web-page.png)

12. 在欄位中輸入文字，然後選取 [傳送] 按鈕。 資料便會傳送給 Kafka。 然後應用程式中的 Kafka 取用者會讀取訊息，並將它新增至 [來自 Kafka 的訊息] 區段。

    > [!WARNING]
    > 您可能會收到同一則訊息很多次。 當您在連線之後重新整理瀏覽器，或開啟應用程式的多個瀏覽器連線時，通常會發生此問題。

## <a name="next-steps"></a>後續步驟

使用下列連結以了解如何使用 HDInsight 上的 Apache Kafka：

* [開始使用 HDInsight 上的 Kafka](apache-kafka-get-started.md)

* [使用 MirrorMaker 建立 Apache Kafka on HDInsight 複本](apache-kafka-mirroring.md)

* [使用 Apache Storm 搭配 HDInsight 上的 Kafka](../hdinsight-apache-storm-with-kafka.md)

* [使用 Apache Spark 搭配 Kafka on HDInsight](../hdinsight-apache-spark-with-kafka.md)

* [透過 Azure 虛擬網路連線到 Kafka](apache-kafka-connect-vpn-gateway.md)
