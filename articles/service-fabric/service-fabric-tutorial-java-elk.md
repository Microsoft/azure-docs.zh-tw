---
title: 在 Azure 中使用 ELK 監視 Service Fabric 中的應用程式 | Microsoft Docs
description: 在本教學課程，了解如何設定 ELK 和監視 Service Fabric 應用程式。
services: service-fabric
documentationcenter: java
author: suhuruli
manager: msfussell
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/26/2018
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: 938d8efeaa88cc5bebbf33e525132a030f1b3c7c
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37112498"
---
# <a name="tutorial-monitor-your-service-fabric-applications-using-elk"></a>教學課程：使用 ELK 監視 Service Fabric 應用程式

本教學課程是一個系列的第四部分。 它會顯示如何使用 ELK (Elasticsearch、Logstash 和 Kibana) 監視在 Azure 中執行的 Service Fabric 應用程式。

在本系列的第一部分中，您了解如何：
> [!div class="checklist"]
> * 在 Azure 中設定 ELK 伺服器
> * 設定 Logstash 以從事件中樞接收記錄
> * 在 Kibana 中將平台和應用程式記錄視覺化

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [建置 Java Service Fabric Reliable Services 應用程式](service-fabric-tutorial-create-java-app.md)
> * [在本機叢集上部署及偵錯應用程式](service-fabric-tutorial-debug-log-local-cluster.md)
> * [將應用程式部署至 Azure 叢集](service-fabric-tutorial-java-deploy-azure.md)
> * 設定應用程式的監視和診斷
> * [設定 CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>先決條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* 設定您的應用程式以將記錄發出至在[第二部分](service-fabric-tutorial-debug-log-local-cluster.md)中指定的位置。
* 完成[第三部分](service-fabric-tutorial-java-deploy-azure.md)，並將執行中 Service Fabric 叢集設定為將記錄傳送至事件中樞。
* 事件中樞內具有「接聽」權限的原則，以及來自系列第三部分的相關聯主索引鍵。

## <a name="download-the-voting-sample-application"></a>下載投票應用程式範例

如果您未在[本教學課程系列的第一部分](service-fabric-tutorial-create-java-app.md)中建置投票應用程式範例，可以下載它。 在命令視窗中執行下列命令，將範例應用程式存放庫複製到本機電腦。

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

## <a name="create-an-elk-server-in-azure"></a>在 Azure 中建立 ELK 伺服器

您可以使用為此教學課程預先設定的 ELK 環境，而如果您已經有此環境，請跳至「設定 Logstash」一節。 不過，如果您沒有此環境，下列步驟會在 Azure 中建立此環境。

1. 在 Azure 中建立由 [Bitnami](https://ms.portal.azure.com/#create/bitnami.elk4-6) 認證的 ELK。 基於本教學課程的目的，建立此伺服器時，不需遵循任何特定的規格。

2. 在 Azure 入口網站中移至您的資源，並進入 [支援 + 疑難排解] 區段下的 [開機診斷] 索引標籤。 接下來，按一下 [序列記錄] 索引標籤。

    ![開機診斷](./media/service-fabric-tutorial-java-elk/bootdiagnostics.png)
3. 您必須在密碼的記錄上執行搜尋，才能存取 Kibana 執行個體。 這類似於下面程式碼片段：

    ```bash
    [   25.932766] bitnami[1496]: #########################################################################
    [   25.948656] bitnami[1496]: #                                                                       #
    [   25.962448] bitnami[1496]: #        Setting Bitnami application password to '[PASSWORD]'           #
    [   25.978137] bitnami[1496]: #        (the default application username is 'user')                   #
    [   26.004770] bitnami[1496]: #                                                                       #
    [   26.029413] bitnami[1496]: #########################################################################
    ```

4. 在 Azure 入口網站中伺服器的 [概觀] 頁面上按 [連線] 按鈕，以取得登入詳細資料。

    ![VM 連線](./media/service-fabric-tutorial-java-elk/vmconnection.png)

5. 使用下列命令，透過 SSH 連線至裝載 ELK 映像的伺服器

    ```bash
    ssh [USERNAME]@[CONNECTION-IP-OF-SERVER]

    Example: ssh testaccount@104.40.63.157
    ```

## <a name="set-up-elk"></a>設定 ELK

1. 第一步是載入 ELK 環境

    ```bash
    sudo /opt/bitnami/use_elk
    ```

2. 如果您使用現有的環境，則必須執行下列命令來停止 Logstash 服務

    ```bash
    sudo /opt/bitnami/ctlscript.sh stop logstash
    ```

3. 執行下列命令來安裝適用於事件中樞的 Logstash 外掛程式。

    ```bash
    logstash-plugin install logstash-input-azureeventhub
    ```

4. 使用下列內容建立或修改現有的 Logstash 組態檔：如果您要建立此檔案，它必須建立於```/opt/bitnami/logstash/conf/access-log.conf``` (如果在 Azure 中使用 ELK Bitnami 映像)。

    ```json
    input
    {
        azureeventhub
        {
            key => "[RECEIVER-POLICY-KEY-FOR-EVENT-HUB]"
            username => "[RECEIVER-POLICY-NAME]"
            namespace => "[EVENTHUB-NAMESPACENAME]"
            eventhub => "[EVENTHUB-NAME]"
            partitions => 4
        }
    }

    output {
         elasticsearch {
             hosts => [ "127.0.0.1:9200" ]
         }
     }
    ```

5. 若要驗證組態，請執行下列命令：

    ```bash
    /opt/bitnami/logstash/bin/logstash -f /opt/bitnami/logstash/conf/ --config.test_and_exit
    ```

6. 啟動 Logstash 服務

    ```bash
    sudo /opt/bitnami/ctlscript.sh start logstash
    ```

7. 檢查您的 Elasticsearch 以確定您正在接收資料

    ```bash
    curl 'localhost:9200/_cat/indices?v'
    ```

8. 存取 Kibana 儀表板 (**http://SERVER-IP**) 並輸入 Kibana 的使用者名稱和密碼。 如果您在 Azure 中使用 ELK 映像，則預設使用者名稱為 'user'，而密碼則取自 [開機診斷]。

    ![Kibana](./media/service-fabric-tutorial-java-elk/kibana.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中啟動並執行 ELK 伺服器
> * 將伺服器設定成從 Service Fabric 叢集接收診斷資訊

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [使用 Jenkins 設定 CI/CD](service-fabric-tutorial-java-jenkins.md)
