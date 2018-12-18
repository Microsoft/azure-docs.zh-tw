---
title: 在 Linux 上使用 C# 建立您的第一個 Azure Service Fabric 應用程式 | Microsoft Docs
description: 了解如何使用 C# 和 .NET Core 2.0 建立和部署 Service Fabric 應用程式。
services: service-fabric
documentationcenter: csharp
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: 5a96d21d-fa4a-4dc2-abe8-a830a3482fb1
ms.service: service-fabric
ms.devlang: csharp
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/11/2018
ms.author: subramar
ms.openlocfilehash: e917119e2d0f9b7b5cfa9ea145cc6e540486ac66
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44052591"
---
# <a name="create-your-first-azure-service-fabric-application"></a>建立第一個 Azure Service Fabric 應用程式
> [!div class="op_single_selector"]
> * [C# - Windows](service-fabric-create-your-first-application-in-visual-studio.md)
> * [Java - Linux (預覽)](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux (預覽)](service-fabric-create-your-first-linux-application-with-csharp.md)
>
>

Service Fabric 提供了在 Linux 上建置服務的 .NET Core 和 Java SDK。 在本教學課程中，我們會探討如何建立適用於 Linux 的應用程式以及在 NET Core 2.0 上使用 C# 建置服務。

## <a name="prerequisites"></a>必要條件
開始之前，請確定您已 [設定 Linux 開發環境](service-fabric-get-started-linux.md)。 如果您使用 Mac OS X，您可以 [使用 Vagrant 在虛擬機器中設定 Linux 一整體環境](service-fabric-get-started-mac.md)。

您也要安裝 [Service Fabric CLI](service-fabric-cli.md)

### <a name="install-and-set-up-the-generators-for-c"></a>安裝及設定 C# 的產生器
Service Fabric 提供的 Scaffolding 工具可協助您從終端機使用 Yeoman 範本產生器建立 Service Fabric 應用程式。 請遵循下列步驟來設定適用於 C# 的 Service Fabric Yeoman 範本產生器：

1. 在電腦上安裝 nodejs 和 NPM

   ```bash
   curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash 
   nvm install node 
   ```
2. 在電腦上從 NPM 安裝 [Yeoman](http://yeoman.io/) 範本產生器

  ```bash
  npm install -g yo
  ```
3. 從 NPM 安裝 Service Fabric Yeoman C# 應用程式產生器

  ```bash
  npm install -g generator-azuresfcsharp
  ```

## <a name="create-the-application"></a>建立應用程式
Service Fabric 應用程式可以包含一或多個服務，而每個服務在提供應用程式的功能時都有特定角色。 您在最後一個步驟安裝之適用於 C# 的 Service Fabric [Yeoman](http://yeoman.io/) 產生器，可讓您輕鬆建立第一個服務且稍後新增更多服務。 讓我們使用 Yeoman 來建立具有單一服務的應用程式。

1. 在終端機中，輸入下列命令以開始建置樣板︰`yo azuresfcsharp`
2. 為您的應用程式命名。
3. 選擇第一個服務的類型並加以命名。 基於本教學課程的用途，我們會選擇 Reliable Actor 服務。

   ![適用於 C# 的 Service Fabric Yeoman 產生器][sf-yeoman]

> [!NOTE]
> 如需選項的詳細資訊，請參閱 [Service Fabric 程式設計模型概觀](service-fabric-choose-framework.md)。
>
>

## <a name="build-the-application"></a>建置應用程式
Service Fabric Yeoman 範本包含建置指令碼，可用來從終端機建置應用程式 (在瀏覽至應用程式資料夾後)。

  ```sh
 cd myapp
 ./build.sh
  ```

## <a name="deploy-the-application"></a>部署應用程式

建置應用程式後，可以將它部署到本機叢集。

1. 連接到本機 Service Fabric 叢集。

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. 執行範本中所提供的安裝指令碼，將應用程式套件複製到叢集的映像存放區、註冊應用程式類型，以及建立應用程式的執行個體。

    ```bash
    ./install.sh
    ```

部署建置的應用程式與部署其他任何 Service Fabric 應用程式相同。 請參閱[使用 Service Fabric CLI 管理 Service Fabric 應用程式](service-fabric-application-lifecycle-sfctl.md)文件以取得詳細指示。

這些命令的參數可以在應用程式套件內產生的資訊清單中找到。

部署應用程式後，開啟瀏覽器並瀏覽至 [http://localhost:19080/Explorer](http://localhost:19080/Explorer) 的 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)。 接著展開 [應用程式] 節點，請注意，您的應用程式類型現在有一個項目，而另一個項目則在該類型的第一個執行個體。

> [!IMPORTANT]
> 若要將應用程式部署到 Azure 中的安全 Linux 叢集，您需要設定憑證來向 Service Fabric 執行階段驗證您的應用程式。 這樣做就能讓您的 Reliable Services 服務可與基礎 Service Fabric 執行階段 API 進行通訊。 若要深入了解，請參閱[將 Reliable Services 應用程式設定為在 Linux 叢集上執行](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters)。  
>

## <a name="start-the-test-client-and-perform-a-failover"></a>啟動測試用戶端並執行容錯移轉
動作項目專案沒有任何屬於自己的項目。 它們需要其他服務或用戶端傳送訊息給它們。 動作項目範本包含簡單的測試指令碼，您可以用來與動作項目服務互動。

1. 使用監看式公用程式執行指令碼，以查看動作項目服務的輸出。

   如果是 MAC OS X，您需要藉由執行下列其他命令，將 myactorsvcTestClient 資料夾複製到容器內的某些位置。
    
    ```bash
    docker cp  [first-four-digits-of-container-ID]:/home
    docker exec -it [first-four-digits-of-container-ID] /bin/bash
    cd /home
    ```
    
    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```
2. 在 Service Fabric Explorer 中，找出裝載動作項目服務主要複本的節點。 在以下的螢幕擷取畫面中是節點 3。

    ![在 Service Fabric Explorer 中尋找主要複本][sfx-primary]
3. 按一下您在上一個步驟中找到的節點，然後從 [動作] 功能表選取 [停用 (重新啟動)]  。 這個動作會重新啟動本機叢集中的其中一個節點，強制容錯移轉至在另一個節點上執行的次要複本。 當您執行這個動作時，請留意測試用戶端的輸出，並注意儘管是容錯移轉，計數器仍會繼續增加。

## <a name="adding-more-services-to-an-existing-application"></a>將更多服務新增至現有的應用程式

若要將其他服務新增至已使用 `yo` 建立的應用程式，請執行下列步驟︰
1. 將目錄變更為現有應用程式的根目錄。  例如，如果 `MyApplication` 是 Yeoman 所建立的應用程式，則為 `cd ~/YeomanSamples/MyApplication`。
2. 執行 `yo azuresfcsharp:AddService`

## <a name="next-steps"></a>後續步驟

* [使用 Service Fabric CLI 與 Service Fabric 叢集互動](service-fabric-cli.md)
* 了解 [Service Fabric 支援選項](service-fabric-support.md)
* [開始使用 Service Fabric CLI](service-fabric-cli.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-csharp/yeoman-csharp.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-csharp/sfx-primary.png
