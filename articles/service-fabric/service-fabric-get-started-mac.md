---
title: 在 Mac OS X 上設定開發環境以搭配 Azure Service Fabric 運作 | Microsoft Docs
description: 安裝執行階段、SDK 和工具，並建立本機開發叢集。 完成此設定之後，您就可以開始在 Mac OS X 上建置應用程式。
services: service-fabric
documentationcenter: linux
author: suhuruli
manager: timlt
editor: ''
ms.assetid: bf84458f-4b87-4de1-9844-19909e368deb
ms.service: service-fabric
ms.devlang: linux
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/17/2017
ms.author: suhuruli
ms.openlocfilehash: 618180f9cbd1c65863e3e039bac40d1cc493bc37
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918495"
---
# <a name="set-up-your-development-environment-on-mac-os-x"></a>在 Mac OS X 上設定開發環境
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [OSX](service-fabric-get-started-mac.md)
>
>  

您可以建置 Azure Service Fabric 應用程式，以使用 Mac OS X 在 Linux 叢集上執行。本文件涵蓋如何設定您的 Mac 進行開發。

## <a name="prerequisites"></a>必要條件
Azure Service Fabric 不會在 Mac OS X 上以原生方式執行。若要執行本機 Service Fabric 叢集，我們提供預先設定的 Docker 容器映像。 開始之前，您需要：

* 至少 4 GB 的 RAM。
* 最新版 [Docker](https://www.docker.com/)。

>[!TIP]
>
>若要在 Mac 上安裝 Docker，請遵循 [Docker 文件](https://docs.docker.com/docker-for-mac/install/#what-to-know-before-you-install)中的步驟。 在安裝之後，請[確認您的安裝](https://docs.docker.com/docker-for-mac/#check-versions-of-docker-engine-compose-and-machine)。
>

## <a name="create-a-local-container-and-set-up-service-fabric"></a>建立本機容器並設定 Service Fabric
若要設定本機的 Docker 容器，並在容器上執行 Service Fabric 叢集，請執行下列步驟：

1. 使用下列設定更新您主機上的 Docker 精靈設定，然後重新啟動 Docker 精靈： 

    ```json
    {
        "ipv6": true,
        "fixed-cidr-v6": "fd00::/64"
    }
    ```
    您可以直接在 Docker 安裝路徑的 daemon.json 檔案中更新這些設定。
    
    >[!NOTE]
    >
    >每一台機器上的 daemon.json 檔案位置會各不相同。 例如，~/Library/Containers/com.docker.docker/Data/database/com.docker.driver.amd64-linux/etc/docker/daemon.json。
    >
    >建議的方法是直接修改 Docker 中的精靈組態設定。 選取 [Docker 圖示]，然後選取 [喜好設定] > [精靈] > [進階]。
    >
    >測試大型應用程式時，建議提高配置給 Docker 的資源。 選取 [Docker 圖示]，然後選取 [進階] 來調整核心數目和記憶體，即可完成此作業。

2. 在新目錄中建立一個名為 `Dockerfile` 的檔案，以建置 Service Fabric 映像：

    ```dockerfile
    FROM microsoft/service-fabric-onebox
    WORKDIR /home/ClusterDeployer
    RUN ./setup.sh
    #Generate the local
    RUN locale-gen en_US.UTF-8
    #Set environment variables
    ENV LANG=en_US.UTF-8
    ENV LANGUAGE=en_US:en
    ENV LC_ALL=en_US.UTF-8
    EXPOSE 19080 19000 80 443
    #Start SSH before running the cluster
    CMD /etc/init.d/ssh start && ./run.sh
    ```

    >[!NOTE]
    >您可以調整這個檔案，以在您的容器中新增其他程式或相依性。
    >例如，新增 `RUN apt-get install nodejs -y` 將允許以客體可執行檔形式支援 `nodejs` 應用程式。
    
    >[!TIP]
    > 根據預設，這會提取包含最新版 Service Fabric 的映像。 如需特定版本，請造訪 [Docker Hub](https://hub.docker.com/r/microsoft/service-fabric-onebox/) 頁面。

3. 若要從 `Dockerfile` 建置可重複使用的映像，請開啟終端機並 `cd` 到直接保留的 `Dockerfile`，然後執行：

    ```bash 
    docker build -t mysfcluster .
    ```
    
    >[!NOTE]
    >這項作業需要一些時間，但只需要執行一次。

4. 您現在可以在需要時快速啟動 Service Fabric 的本機複本，方法是執行以下項目：

    ```bash 
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mysfcluster
    ```

    >[!TIP]
    >為容器執行個體提供名稱，以便能以更容易辨識的方式來處理。 
    >
    >如果您的應用程式正在特定連接埠上接聽，這些連接埠就必須使用額外的 `-p` 標籤來加以指定。 例如，如果您的應用程式正在連接埠 8080 上接聽，請新增下列 `-p` 標籤：
    >
    >`docker run -itd -p 19080:19080 -p 8080:8080 --name sfonebox microsoft/service-fabric-onebox`
    >

5. 叢集會在短時間內啟動，您可以使用下列命令來檢視記錄，或跳到儀表板檢視叢集健康情況 [http://localhost:19080](http://localhost:19080)：

    ```bash 
    docker logs sftestcluster
    ```



6. 當您完成時，請使用此命令來停止和清除容器：

    ```bash 
    docker rm -f sftestcluster
    ```

### <a name="known-limitations"></a>已知限制 
 
 在 Mac 的容器中執行的本機叢集已知有下列限制： 
 
 * DNS 服務未執行，且不受支援 ([問題 #132](https://github.com/Microsoft/service-fabric/issues/132))

## <a name="set-up-the-service-fabric-cli-sfctl-on-your-mac"></a>設定 Mac 上的 Service Fabric CLI (sfctl)

請遵循 [Service Fabric CLI](service-fabric-cli.md#cli-mac) 的指示在您的 Mac 安裝 Service Fabric CLI (`sfctl`)。
CLI 命令支援與 Service Fabric 實體 (包括叢集、應用程式和服務) 互動。

1. 若要在部署應用程式前連線到叢集，請執行下列命令。 

```bash
sfctl cluster select --endpoint http://localhost:19080
```

## <a name="create-your-application-on-your-mac-by-using-yeoman"></a>在 Mac 上使用 Yeoman 建立應用程式

Service Fabric 提供的 Scaffolding 工具可協助您從終端機使用 Yeoman 範本產生器建立 Service Fabric 應用程式。 請使用下列步驟，確保 Service Fabric Yeoman 範本產生器可在您的機器上運作：

1. 您的 Mac 上必須安裝 Node.js 和節點套件管理員 (NPM)。 您可以使用 [HomeBrew](https://brew.sh/) 安裝軟體，如下所示：

    ```bash
    brew install node
    node -v
    npm -v
    ```
2. 在您的機器上透過 NPM 安裝 [Yeoman](http://yeoman.io/) 範本產生器：

    ```bash
    npm install -g yo
    ```
3. 遵循快速入門[文件](service-fabric-get-started-linux.md)中的步驟，安裝您想要的 Yeoman 產生器。 若要使用 Yeoman 建立 Service Fabric 應用程式，請遵循下列步驟：

    ```bash
    npm install -g generator-azuresfjava       # for Service Fabric Java Applications
    npm install -g generator-azuresfguest      # for Service Fabric Guest executables
    npm install -g generator-azuresfcontainer  # for Service Fabric Container Applications
    ```
4. 若要在 Mac 上建置 Service Fabric Java 應用程式，您必須在主機機器上安裝 JDK 1.8 版和 Gradle。 您可以使用 [HomeBrew](https://brew.sh/) 安裝軟體，如下所示： 

    ```bash
    brew update
    brew cask install java
    brew install gradle
    ```

## <a name="deploy-your-application-on-your-mac-from-the-terminal"></a>在 Mac 上從終端機部署應用程式

在建立並建置 Service Fabric 應用程式後，您就可以使用 [Service Fabric CLI](service-fabric-cli.md#cli-mac) 部署應用程式：

1. 在 Mac 上連線到容器執行個體內執行的 Service Fabric 叢集：

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. 從您的專案目錄內執行安裝指令碼：

    ```bash
    cd MyProject
    bash install.sh
    ```

## <a name="set-up-net-core-20-development"></a>設定 .NET Core 2.0 開發

安裝 [.NET Core 2.0 SDK for Mac](https://www.microsoft.com/net/core#macos) 以開始[建立 C# Service Fabric 應用程式](service-fabric-create-your-first-linux-application-with-csharp.md)。 .NET Core 2.0 Service Fabric 應用程式的套件裝載於 NuGet.org (目前處於預覽狀態)。

## <a name="install-the-service-fabric-plug-in-for-eclipse-on-your-mac"></a>在 Mac 上安裝適用於 Eclipse 的 Service Fabric 外掛程式

Azure Service Fabric 會針對 Java IDE 提供 Eclipse Neon (或更新版本) 的外掛程式。 外掛程式可簡化建立、建置和部署 Java 服務的程序。 若要安裝或更新為最新版的 Eclipse Service Fabric 外掛程式，請遵循[這些步驟](service-fabric-get-started-eclipse.md#install-or-update-the-service-fabric-plug-in-in-eclipse)。 [Eclipse 的 Service Fabric 文件](service-fabric-get-started-eclipse.md)中的其他步驟也適用：建置應用程式、將服務新增至應用程式、解除安裝應用程式等。

最後一個步驟是使用與您的主機共用的路徑將容器具現化。 外掛程式需要這種類型的具現化，才能在您的 Mac 上使用 Docker 容器。 例如︰

```bash
docker run -itd -p 19080:19080 -v /Users/sayantan/work/workspaces/mySFWorkspace:/tmp/mySFWorkspace --name sfonebox microsoft/service-fabric-onebox
```

屬性的定義如下：
* `/Users/sayantan/work/workspaces/mySFWorkspace` 是您 Mac 上的工作區完整路徑。
* `/tmp/mySFWorkspace` 是工作區所應對應的容器內路徑。

>[!NOTE]
> 
>如果您的工作區使用不同的名稱/路徑，請在 `docker run` 命令中更新這些值。
> 
>如果您使用 `sfonebox` 以外的名稱來啟動容器，請在您 Service Fabric 執行者 Java 應用程式中的 testclient.sh 檔案更新該名稱值。
>

## <a name="next-steps"></a>後續步驟
<!-- Links -->
* [使用 Yeoman 在 Linux 上建立和部署第一個 Service Fabric Java 應用程式](service-fabric-create-your-first-linux-application-with-java.md)
* [在 Linux 上使用適用於 Eclipse 的 Service Fabric 外掛程式建立和部署第一個 Service Fabric Java 應用程式](service-fabric-get-started-eclipse.md)
* [在 Azure 入口網站中建立 Service Fabric 叢集](service-fabric-cluster-creation-via-portal.md)
* [使用 Azure Resource Manager 來建立 Service Fabric 叢集](service-fabric-cluster-creation-via-arm.md)
* [了解 Service Fabric 應用程式模型](service-fabric-application-model.md)
* [使用 Service Fabric CLI 管理應用程式](service-fabric-application-lifecycle-sfctl.md)
* [在 Windows 上準備 Linux 開發環境](service-fabric-local-linux-cluster-windows.md)

<!-- Images -->
[cluster-setup-script]: ./media/service-fabric-get-started-mac/cluster-setup-mac.png
[sfx-mac]: ./media/service-fabric-get-started-mac/sfx-mac.png
[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-mac/sf-eclipse-plugin-install.png
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
