---
title: 在本機 Service Fabric 叢集上進行 Java 應用程式偵錯 | Microsoft Docs
description: 在本教學課程中，了解如何偵錯並從在本機叢集上執行的 Service Fabric Java 應用程式取得記錄。
services: service-fabric
documentationcenter: java
author: suhuruli
manager: mfussell
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
ms.openlocfilehash: c1a8b18062f61be9eb020beefd3ad741c41b55f8
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38652697"
---
# <a name="tutorial-debug-a-java-application-deployed-on-a-local-service-fabric-cluster"></a>教學課程：偵錯在本機 Service Fabric 叢集上部署的 Java 應用程式

本教學課程是一個系列的第二部分。 您會了解如何使用適用於 Service Fabric 應用程式的 Eclipse 來連結遠端偵錯工具。 此外，您會了解如何將執行中應用程式的記錄重新導向至開發人員方便取用的位置。

在本系列的第二部分中，您將瞭解如何：
> [!div class="checklist"]
> * 使用 Eclipse 部署 Java 應用程式
> * 將記錄重新導向至可設定的位置

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [建置 Java Service Fabric Reliable Services 應用程式](service-fabric-tutorial-create-java-app.md)
> * 在本機叢集上部署及偵錯應用程式
> * [將應用程式部署至 Azure 叢集](service-fabric-tutorial-java-deploy-azure.md)
> * [設定應用程式的監視和診斷](service-fabric-tutorial-java-elk.md)
> * [設定 CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>先決條件

開始進行本教學課程之前：

* 在 [Mac](service-fabric-get-started-mac.md) 或 [Linux](service-fabric-get-started-linux.md) 上設定開發環境。 請遵循指示來安裝 Eclipse 外掛程式、 Gradle、Service Fabric SDK 和 Service Fabric CLI (sfctl)。

## <a name="download-the-voting-sample-application"></a>下載投票應用程式範例

如果您未在[本教學課程系列的第一部分](service-fabric-tutorial-create-java-app.md)中建置投票應用程式範例，可以下載它。 在命令視窗中執行下列命令，將範例應用程式存放庫複製到本機電腦。

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

[建置解決方案並部署](service-fabric-tutorial-create-java-app.md#deploy-application-to-local-cluster)至本機開發叢集。

## <a name="debug-java-application-using-eclipse"></a>使用 Eclipse 部署 Java 應用程式

1. 在您的電腦上開啟 Eclipse IDE，然後按一下 [檔案] -> [匯入...]

2. 在快顯視窗中，選取 [一般] -> [將現有專案匯入工作區中] 選項，然後按 [下一步]。

3. 在 [匯入專案] 視窗中，選擇 [選取根目錄] 選項，然後挑選 [Voting] 目錄。 如果您遵循教學課程系列的第 1 部分，則 [Voting] 目錄位於 [Eclipse-workspace] 目錄中。

4. 更新您要偵錯之服務的 entryPoint.sh，使其以遠端偵錯參數啟動 Java 程序。 此教學課程會使用無狀態前端：Voting/VotingApplication/VotingWebPkg/Code/entryPoint.sh。此範例已設定連接埠 8001 來進行偵錯。

    ```bash
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingWeb.jar
    ```

5. 針對所要偵錯的服務，將執行個體計數或複本計數設定為 1，來更新「應用程式資訊清單」。 此設定可避免用於偵錯的連接埠發生衝突。 例如，針對無狀態服務，請設定 ``InstanceCount="1"``，而針對具狀態服務，則請將目標和最小複本集大小設定為 1，如下所示：``TargetReplicaSetSize="1" MinReplicaSetSize="1"``。

6. 在 Eclipse IDE 中，選取 [Run] (執行) -> [Debug Configurations] (偵錯組態) -> [Remote Java Application] (遠端 Java 應用程式)，按 [New] (新增) 按鈕，依照下列方式設定屬性並且按 [Apply] (套用)。

    ```
    Name: Voting
    Project: Voting
    Connection Type: Standard
    Host: localhost
    Port: 8001
    ```

7. 將中斷點放在 Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java 檔案的 109 行上。

8. 在 [套件總管] 中，以滑鼠右鍵按一下 **Voting** 專案，然後按一下 [Service Fabric] -> [發佈應用程式...]。

9. 在 [發佈應用程式] 視窗中，從下拉式清單中選取 **Local.json**，然後按一下 [發佈]。

10. 在 Eclipse IDE 中，選取 [Run] (執行) -> [Debug Configurations] (偵錯組態) -> [Remote Java Application] (遠端 Java 應用程式)，按一下您建立的 [Voting] 組態，然後按一下 [Debug] (偵錯)。

11. 請移至您的網頁瀏覽器並存取 **localhost:8080** 以叫用中斷點，然後在 Eclipse 中輸入**Debug perspective**。

## <a name="redirect-application-logs-to-custom-location"></a>將應用程式記錄重新導向至自訂位置

下列步驟逐步解說如何將預設 /var/log/syslog 位置中的應用程式記錄重新導向至自訂位置。

1. 目前，在 Service Fabric Linux 叢集中執行的應用程式支援挑選單一記錄檔。 因此，記錄一律會移至 /tmp/mysfapp0.0.log。 在下列位置 Voting/VotingApplication/VotingWebPkg/Code/logging.properties 建立名為 logging.properties 的檔案並新增下列內容。

    ```
    handlers = java.util.logging.FileHandler

    java.util.logging.FileHandler.level = ALL
    java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter

    # This value specifies your custom location. You will have to ensure this path has read and write access by the process running the SF Application
    java.util.logging.FileHandler.pattern = /tmp/mysfapp0.0.log
    ```

2. 在 Java 執行命令的 Voting/VotingApplication/VotingWebPkg/Code/entryPoint.sh 中新增下列參數：

    ```bash
    -Djava.util.logging.config.file=logging.properties
    ```

    下列範例顯示執行範例：

    ```bash
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -Djava.util.logging.config.file=logging.properties -jar VotingWeb.jar
    ```

在這個階段，您已了解如何在開發 Service Fabric Java 應用程式時偵錯及存取您的應用程式記錄。

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 使用 Eclipse 部署 Java 應用程式
> * 將記錄重新導向至可設定的位置

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [將應用程式部署至 Azure](service-fabric-tutorial-java-deploy-azure.md)