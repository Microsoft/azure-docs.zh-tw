---
title: 在 Linux 上設定開發環境 | Microsoft Docs
description: 在 Linux 上安裝執行階段和 SDK，並建立本機開發叢集。 完成此設定之後，您就可以開始建置應用程式。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: d552c8cd-67d1-45e8-91dc-871853f44fc6
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: 00164789d7f37277127878911c3f368a56ec7710
ms.sourcegitcommit: a62cbb539c056fe9fcd5108d0b63487bd149d5c3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42616967"
---
# <a name="prepare-your-development-environment-on-linux"></a>在 Linux 上準備您的開發環境
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [OSX](service-fabric-get-started-mac.md)
>
>  

若要在 Linux 開發機器上部署和執行 [Azure Service Fabric 應用程式](service-fabric-application-model.md) ，請安裝執行階段和通用 SDK。 您也可以安裝 Java 和 .NET Core 開發的選擇性 SDK。 

本文中的步驟假設您要在 Linux 上進行原生安裝，或使用 Service Fabric OneBox 容器映像 `microsoft/service-fabric-onebox`。

不支援在適用於 Linux 的 Windows 子系統上安裝 Service Fabric 執行階段。 您可以使用支援的 Azure Service Fabric 命令列介面 (CLI)，管理裝載於雲端或內部部署中其他地方的 Service Fabric 實體。 如需如何安裝 CLI 的資訊，請參閱[設定 Service Fabric CLI](./service-fabric-cli.md)。


## <a name="prerequisites"></a>必要條件

以下為支援開發的作業系統版本。

* Ubuntu 16.04 (`Xenial Xerus`)

    請確定已安裝 `apt-transport-https` 安裝套件。
         
    ```bash
    sudo apt-get install apt-transport-https
    ```
* Red Hat Enterprise Linux 7.4 (Service Fabric 預覽支援)


## <a name="installation-methods"></a>安裝方法

### <a name="script-installation-ubuntu"></a>指令碼安裝 (Ubuntu)

為了方便起見，提供的指令碼會隨著 **sfctl** CLI 安裝 Service Fabric 執行階段和 Service Fabric 通用 SDK。 請執行下一節中的手動安裝步驟。 您會看到安裝的內容和相關聯的授權。 執行指令碼，即表示您同意所要安裝之所有軟體的授權。

指令碼執行成功之後，您可以跳到[設定本機叢集](#set-up-a-local-cluster)。

```bash
sudo curl -s https://raw.githubusercontent.com/Azure/service-fabric-scripts-and-templates/master/scripts/SetupServiceFabric/SetupServiceFabric.sh | sudo bash
```

### <a name="manual-installation"></a>手動安裝
若要手動安裝 Service Fabric 執行階段和通用 SDK，請遵循本指南的其餘部分。

## <a name="update-your-apt-sources-or-yum-repositories"></a>更新 APT 來源或 Yum 存放庫
若要透過 apt-get 命令列工具安裝 SDK 和相關聯的執行階段套件，您必須先更新 Advanced Packaging Tool (APT) 來源。

### <a name="ubuntu"></a>Ubuntu

1. 開啟終端機。
2. 將 Service Fabric 存放庫新增至來源清單。

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] http://apt-mo.trafficmanager.net/repos/servicefabric/ xenial main" > /etc/apt/sources.list.d/servicefabric.list'
    ```

3. 將 `dotnet` 存放庫新增至來源清單。

    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    ```

4. 將新的 Gnu Privacy Guard (GnuPG 或 GPG) 金鑰新增至 APT keyring。

    ```bash
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 417A0893
    ```

5. 將官方的 Docker GPG 金鑰新增至 APT keyring。

    ```bash
    sudo apt-get install curl
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

6. 設定 Docker 存放庫。

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

7. 根據新增的存放庫重新整理套件清單。

    ```bash
    sudo apt-get update
    ```


### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4 (Service Fabric 預覽支援)

1. 開啟終端機。
2. 下載並安裝 Extra Packages for Enterprise Linux (EPEL)。

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum install epel-release-latest-7.noarch.rpm
    ```
3. 將 EfficiOS RHEL7 套件存放庫新增至您的系統。

    ```bash
    sudo wget -P /etc/yum.repos.d/ https://packages.efficios.com/repo.files/EfficiOS-RHEL7-x86-64.repo
    ```

4. 將 EfficiOS 套件簽署金鑰匯入至本機 GPG Keyring。

    ```bash
    sudo rpmkeys --import https://packages.efficios.com/rhel/repo.key
    ```

5. 將 Microsoft RHEL 存放庫新增至您的系統。

    ```bash
    curl https://packages.microsoft.com/config/rhel/7.4/prod.repo > ./microsoft-prod.repo
    sudo cp ./microsoft-prod.repo /etc/yum.repos.d/
    ```

6. 安裝 .NET SDK。

    ```bash
    yum install rh-dotnet20 -y
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-a-local-cluster"></a>針對本機叢集安裝和設定 Service Fabric SDK

在您更新來源後，就可以安裝 SDK。 安裝 Service Fabric SDK 套件、確認安裝，並接受授權合約。

### <a name="ubuntu"></a>Ubuntu

```bash
sudo apt-get install servicefabricsdkcommon
```

>   [!TIP]
>   下列命令會自動接受 Service Fabric 套件的授權︰
>   ```bash
>   echo "servicefabric servicefabric/accepted-eula-ga select true" | sudo debconf-set-selections
>   echo "servicefabricsdkcommon servicefabricsdkcommon/accepted-eula-ga select true" | sudo debconf-set-selections
>   ```

### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4 (Service Fabric 預覽支援)

```bash
sudo yum install servicefabricsdkcommon
```

SDK 安裝程式隨附的 Service Fabric 執行階段包含下表中的套件。 

 | | DotNetCore | Java | Python | NodeJS | 
--- | --- | --- | --- |---
Ubuntu | 2.0.0 | OpenJDK 1.8 | 內含於 npm | 最新 |
RHEL | - | OpenJDK 1.8 | 內含於 npm | 最新 |

## <a name="set-up-a-local-cluster"></a>設定本機叢集
在安裝完成之後，請啟動本機叢集。

1. 執行叢集安裝指令碼。

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

2. 開啟瀏覽器，前往 [Service Fabric Explorer](http://localhost:19080/Explorer) (`http://localhost:19080/Explorer`)。 叢集啟動時，您會看見 Service Fabric Explorer 儀表板。 叢集可能需要數分鐘的時間才能完成設定。 如果您的瀏覽器無法開啟 URL，或 Service Fabric Explorer 未顯示系統已就緒，請稍候幾分鐘，然後再試一次。

    ![Linux 上的 Service Fabric Explorer][sfx-linux]

    現在，您可以根據客體容器或來賓可執行檔，部署預先建置的 Service Fabric 應用程式套件或新的套件。 若要使用 Java 或 .NET Core SDK 建置新的服務，請遵循以下幾節所提供的選擇性設定步驟。


> [!NOTE]
> Linux 不支援獨立叢集。


> [!TIP]
> 如果您有可用的 SSD 磁碟，建議您透過 devclustersetup.sh 使用 `--clusterdataroot` 傳遞 SSD 資料夾路徑來獲得較佳的效能。

## <a name="set-up-the-service-fabric-cli"></a>設定 Service Fabric CLI

[Service Fabric CLI](service-fabric-cli.md) 包含可供與 Service Fabric 實體 (包括叢集和應用程式) 進行互動的命令。 若要安裝 CLI，請遵循 [Service Fabric CLI](service-fabric-cli.md) 的指示。


## <a name="set-up-yeoman-generators-for-containers-and-guest-executables"></a>設定容器和來賓可執行檔的 Yeoman 產生器
Service Fabric 提供的 Scaffolding 工具可協助您從終端機使用 Yeoman 範本產生器建立 Service Fabric 應用程式。 請遵循下列步驟來設定 Service Fabric Yeoman 範本產生器：

1. 在電腦上安裝 Node.js 和 npm。

    ```bash
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
    nvm install node
    ```
2. 在您的機器上透過 npm 安裝 [Yeoman](http://yeoman.io/) 範本產生器。

    ```bash
    npm install -g yo
    ```
3. 從 npm 安裝 Service Fabric Yeo 容器產生器和來賓可執行檔產生器。

    ```bash
    npm install -g generator-azuresfcontainer  # for Service Fabric container application
    npm install -g generator-azuresfguest      # for Service Fabric guest executable application
    ```

在您安裝產生器後，請分別執行 `yo azuresfguest` 或 `yo azuresfcontainer`，以建立來賓可執行檔或容器服務。

## <a name="set-up-net-core-20-development"></a>設定 .NET Core 2.0 開發

安裝 [.NET Core 2.0 SDK for Ubuntu](https://www.microsoft.com/net/core#linuxubuntu) 以開始[建立 C# Service Fabric 應用程式](service-fabric-create-your-first-linux-application-with-csharp.md)。 NuGet.org 會裝載 .NET Core 2.0 Service Fabric 應用程式的套件 (目前處於預覽狀態)。

## <a name="set-up-java-development"></a>設定 Java 開發

若要使用 Java 建置 Service Fabric 服務，請安裝 JDK 1.8 和 Gradle 來執行建置工作。 下列程式碼片段會隨著 Gradle 安裝 Open JDK 1.8。 系統會從 Maven 提取 Service Fabric Java 程式庫。


* Ubuntu

    ```bash
    sudo apt-get install openjdk-8-jdk-headless
    sudo apt-get install gradle
    ```

* Red Hat Enterprise Linux 7.4 (Service Fabric 預覽支援)

  ```bash
  sudo yum install java-1.8.0-openjdk-devel
  curl -s https://get.sdkman.io | bash
  sdk install gradle
  ```

您也必須安裝 Java 可執行檔的 Service Fabric Yeo 產生器。 請確定您已[安裝 Yeoman](#set-up-yeoman-generators-for-containers-and-guest-executables)，然後執行下列命令：

  ```bash
  npm install -g generator-azuresfjava
  ```
 
## <a name="install-the-eclipse-plug-in-optional"></a>安裝 Eclipse 外掛程式 (選擇性)

您可以從適用於 Java 開發人員或 Java EE 開發人員的 Eclipse 整合式開發環境 (IDE) 安裝適用於 Service Fabric 的 Eclipse 外掛程式。 除了 Service Fabric Java 應用程式之外，您可以使用 Eclipse 來建立 Service Fabric 來賓可執行檔應用程式和容器應用程式。

> [!IMPORTANT]
> Service Fabric 外掛程式需要 Eclipse Neon 或更新版本。 請參閱此注意事項之後的指示，以了解如何檢查您的 Eclipse 版本。 如果您已安裝較早版本的 Eclipse，可以從 [Eclipse 網站](https://www.eclipse.org)下載較新版本。 建議您不要在現有的 Eclipse 安裝之上進行安裝 (覆寫)。 請在執行安裝程式之前先移除舊版，或在不同的目錄中安裝較新版本。
> 
> 在 Ubuntu 上，建議您直接從 Eclipse 網站安裝，而不要使用套件安裝程式 (`apt` 或 `apt-get`)。 這麼做可確保您會取得最新版的 Eclipse。 您可以安裝適用於 Java 開發人員或 Java EE 開發人員的 Eclipse IDE。

1. 在 Eclipse 中，確定您已安裝 Eclipse Neon 或更新版本以及 Buildship 2.2.1 版或更新版本。 請選取 [說明] >  [關於 Eclipse] >  [安裝詳細資料]，檢查已安裝的元件版本。 您可以使用 [Eclipse Buildship：適用於 Gradle 的 Eclipse 外掛程式][buildship-update]的指示來更新 Buildship。

2. 若要安裝 Service Fabric 外掛程式，請選取 [說明]  >  [安裝新軟體]。

3. 在 [使用] 方塊中，輸入 **http://dl.microsoft.com/eclipse**。

4. 選取 [新增] 。

    ![可用的軟體頁面][sf-eclipse-plugin]

5. 選取 [ServiceFabric] 外掛程式，然後按 [下一步]。

6. 執行安裝步驟。 然後接受使用者授權合約。

如果您已安裝 Service Fabric Eclipse 外掛程式，請確定您擁有的是最新版本。 請選取 [說明] > [關於Eclipse] > [安裝詳細資料] 來檢查。 然後，在已安裝的外掛程式清單中搜尋 Service Fabric。如果有可用的較新版本，請選取 [更新]。

如需詳細資訊，請參閱[適用於 Eclipse Java 應用程式開發的 Service Fabric 外掛程式](service-fabric-get-started-eclipse.md)。

## <a name="update-the-sdk-and-runtime"></a>更新 SDK 和執行階段

若要更新為最新版的 SDK 和執行階段，請執行下列命令。

```bash
sudo apt-get update
sudo apt-get install servicefabric servicefabricsdkcommon
```
若要更新 Maven 提供的 Java SDK 二進位檔，您必須在 ``build.gradle`` 檔案中更新對應二進位檔的版本詳細資料，以指向最新版本。 若想知道您需要更新版本的確切位置，請在 [Service Fabric 入門範例](https://github.com/Azure-Samples/service-fabric-java-getting-started)中參考 任何 ``build.gradle`` 檔案。

> [!NOTE]
> 更新套件可能會導致本機開發叢集停止執行。 請遵循本文中的指示，在升級之後重新啟動本機叢集。

## <a name="remove-the-sdk"></a>移除 SDK
若要移除 Service Fabric SDK，請執行下列命令。

* Ubuntu

    ```bash
    sudo apt-get remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    sudo apt-get install -f
    ```


* Red Hat Enterprise Linux 7.4 (Service Fabric 預覽支援)

    ```bash
    sudo yum remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    ```

## <a name="next-steps"></a>後續步驟

* [使用 Yeoman 在 Linux 上建立和部署第一個 Service Fabric Java 應用程式](service-fabric-create-your-first-linux-application-with-java.md)
* [在 Linux 上使用適用於 Eclipse 的 Service Fabric 外掛程式建立和部署第一個 Service Fabric Java 應用程式](service-fabric-get-started-eclipse.md)
* [在 Linux 上建立第一個 CSharp 應用程式](service-fabric-create-your-first-linux-application-with-csharp.md)
* [在 OSX 上準備您的開發環境](service-fabric-get-started-mac.md)
* [在 Windows 上準備 Linux 開發環境](service-fabric-local-linux-cluster-windows.md)
* [使用 Service Fabric CLI 管理應用程式](service-fabric-application-lifecycle-sfctl.md)
* [Service Fabric Windows 與 Linux 的差異](service-fabric-linux-windows-differences.md)
* [在 Linux 叢集上將作業系統修補自動化](service-fabric-patch-orchestration-application-linux.md)
* [開始使用 Service Fabric CLI](service-fabric-cli.md)

<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png
