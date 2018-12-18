---
title: 在 Linux 上建立 Azure Service Fabric Reliable Actors Java 應用程式 | Microsoft Docs
description: 了解如何在五分鐘內建立和部署 Java Service Fabric Reliable Actors 應用程式。
services: service-fabric
documentationcenter: java
author: rwike77
manager: timlt
editor: ''
ms.assetid: 02b51f11-5d78-4c54-bb68-8e128677783e
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/18/2018
ms.author: ryanwi
ms.openlocfilehash: 07f739243b80230fbf4914535ea65183c3590937
ms.sourcegitcommit: 0fa8b4622322b3d3003e760f364992f7f7e5d6a9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37020436"
---
# <a name="create-your-first-java-service-fabric-reliable-actors-application-on-linux"></a>在 Linux 上建立第一個 Java Service Fabric Reliable Actors 應用程式
> [!div class="op_single_selector"]
> * [C# - Windows](service-fabric-create-your-first-application-in-visual-studio.md)
> * [Java - Linux](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
>
>

本快速入門可協助您在短短幾分鐘內在 Linux 開發環境中建立第一個 Azure Service Fabric Java 應用程式。  當您完成時，您會有一個在本機開發叢集上執行的簡單 Java 單一服務應用程式。  

## <a name="prerequisites"></a>先決條件
開始之前，請在 [Linux 開發環境](service-fabric-get-started-linux.md)中安裝 Service Fabric SDK、Service Fabric CLI、Yeoman，然後設定 Java 開發環境，並設定開發叢集。 如果您使用 Mac OS X，您可以[使用 Docker 在 Mac 上設定開發環境](service-fabric-get-started-mac.md)。

另請安裝 [Service Fabric CLI](service-fabric-cli.md)。

### <a name="install-and-set-up-the-generators-for-java"></a>安裝及設定 Java 的產生器
Service Fabric 提供的 Scaffolding 工具可協助您從終端機使用 Yeoman 範本產生器建立 Service Fabric Java 應用程式。  如果尚未安裝 Yeoman，請參閱[在 Linux 上開始使用 Service Fabric](service-fabric-get-started-linux.md#set-up-yeoman-generators-for-containers-and-guest-executables)，以取得設定 Yeoman 的指示。 執行下列命令，以安裝 Java 的 Service Fabric Yeoman 範本產生器。

  ```bash
  sudo npm install -g generator-azuresfjava
  ```

## <a name="basic-concepts"></a>基本概念
若要開始使用 Reliable Actors，您只需要了解幾個基本概念：

* **動作項目服務**。 Reliable Actors 封裝在可在 Service Fabric 基礎結構內部署的 Reliable Services 中。 動作項目執行個體會在指定的服務執行個體中啟動。
* **動作項目註冊**。 和 Reliable Services 一樣，Reliable Actor 服務必須向 Service Fabric 執行階段註冊。 此外，動作項目類型必須向 Actor 執行階段註冊。
* **動作項目介面**。 動作項目介面用於定義動作項目的強型別公用介面。 在 Reliable Actor 模型術語中，動作項目介面定義動作項目可以了解並處理的訊息類型。 其他的動作項目或用戶端應用程式會使用動作項目介面將訊息「傳送」(非同步) 給動作項目。 Reliable Actors 可實作多個介面。
* **ActorProxy 類別**。 用戶端應用程式會使用 ActorProxy 類別來叫用透過動作項目介面公開的方法。 ActorProxy 類別提供兩個重要的功能：
  
  * 名稱解析︰它能夠在叢集中找到動作項目 (尋找裝載動作項目的叢集節點)。
  * 處理失敗：它可以重試方法叫用並重新解析動作項目位置，例如在需要動作項目重新定位至叢集中另一個節點失敗後進行。

值得一提的是下列與動作項目介面相關的規則︰

* 動作項目介面方法無法多載。
* 動作項目介面方法不能有 out、ref 或選擇性參數。
* 不支援泛型介面。

## <a name="create-the-application"></a>建立應用程式
Service Fabric 應用程式包含一或多個服務，而每個服務在提供應用程式的功能時都有特定角色。 您在上一節中安裝的產生器，可讓您輕鬆建立第一個服務且稍後新增更多服務。  您也可以使用適用於 Eclipse 的外掛程式，建立、建置及部署 Service Fabric Java 應用程式。 請參閱[使用 Eclipse 建立和部署第一個 Java 應用程式](service-fabric-get-started-eclipse.md)。 在此快速入門中，使用 Yeoman 建立具有單一服務的應用程式，該服務可儲存和取得計數器值。

1. 在終端機中，輸入 ``yo azuresfjava``。
2. 為您的應用程式命名。
3. 選擇第一個服務的類型並加以命名。 在本教學課程中，選擇 Reliable Actor 服務。 如需其他服務類型的詳細資訊，請參閱 [Service Fabric 程式設計模型概觀](service-fabric-choose-framework.md)。
   ![適用於 Java 的 Service Fabric Yeoman 產生器][sf-yeoman]

如果您將應用程式命名為 "HelloWorldActorApplication"，並將動作項目命名為 "HelloWorldActor"，就會建立下列 Scaffolding：

```bash
HelloWorldActorApplication/
├── build.gradle
├── HelloWorldActor
│   ├── build.gradle
│   ├── settings.gradle
│   └── src
│       └── reliableactor
│           ├── HelloWorldActorHost.java
│           └── HelloWorldActorImpl.java
├── HelloWorldActorApplication
│   ├── ApplicationManifest.xml
│   └── HelloWorldActorPkg
│       ├── Code
│       │   ├── entryPoint.sh
│       │   └── _readme.txt
│       ├── Config
│       │   ├── _readme.txt
│       │   └── Settings.xml
│       ├── Data
│       │   └── _readme.txt
│       └── ServiceManifest.xml
├── HelloWorldActorInterface
│   ├── build.gradle
│   └── src
│       └── reliableactor
│           └── HelloWorldActor.java
├── HelloWorldActorTestClient
│   ├── build.gradle
│   ├── settings.gradle
│   ├── src
│   │   └── reliableactor
│   │       └── test
│   │           └── HelloWorldActorTestClient.java
│   └── testclient.sh
├── install.sh
├── settings.gradle
└── uninstall.sh
```
## <a name="reliable-actors-basic-building-blocks"></a>Reliable Actors 項目基本建置組塊
稍早所述的基本概念轉化為 Reliable Actors 服務的基本建置組塊。

### <a name="actor-interface"></a>動作項目介面
包含動作項目的介面定義。 這個介面定義由動作項目實作與呼叫動作項目的用戶端所共用的動作項目合約，因此通常適合在不同於動作項目實作的地方定義該合約，並可由多個其他服務或用戶端應用程式共用。

`HelloWorldActorInterface/src/reliableactor/HelloWorldActor.java`：

```java
public interface HelloWorldActor extends Actor {
    @Readonly   
    CompletableFuture<Integer> getCountAsync();

    CompletableFuture<?> setCountAsync(int count);
}
```

### <a name="actor-service"></a>動作項目服務
這包含您的動作項目實作和動作項目註冊碼。 動作項目類別會實作動作項目介面。 這是您的動作項目工作之處。

`HelloWorldActor/src/reliableactor/HelloWorldActorImpl`：

```java
@ActorServiceAttribute(name = "HelloWorldActorService")
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class HelloWorldActorImpl extends FabricActor implements HelloWorldActor {
    private Logger logger = Logger.getLogger(this.getClass().getName());

    public HelloWorldActorImpl(FabricActorService actorService, ActorId actorId){
        super(actorService, actorId);
    }

    @Override
    protected CompletableFuture<?> onActivateAsync() {
        logger.log(Level.INFO, "onActivateAsync");

        return this.stateManager().tryAddStateAsync("count", 0);
    }

    @Override
    public CompletableFuture<Integer> getCountAsync() {
        logger.log(Level.INFO, "Getting current count value");
        return this.stateManager().getStateAsync("count");
    }

    @Override
    public CompletableFuture<?> setCountAsync(int count) {
        logger.log(Level.INFO, "Setting current count value {0}", count);
        return this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value);
    }
}
```

### <a name="actor-registration"></a>動作項目註冊
必須在 Service Fabric 執行階段中以某個服務類型註冊動作項目服務。 為了讓動作項目服務執行您的動作項目執行個體，也必須向動作項目服務註冊動作項目類型。 `ActorRuntime` 註冊方法會替動作項目執行這項工作。

`HelloWorldActor/src/reliableactor/HelloWorldActorHost`：

```java
public class HelloWorldActorHost {

private static final Logger logger = Logger.getLogger(HelloWorldActorHost.class.getName());
    
public static void main(String[] args) throws Exception {
        
        try {

            ActorRuntime.registerActorAsync(HelloWorldActorImpl.class, (context, actorType) -> new FabricActorService(context, actorType, (a,b)-> new HelloWorldActorImpl(a,b)), Duration.ofSeconds(10));
            Thread.sleep(Long.MAX_VALUE);
        } catch (Exception e) {
            logger.log(Level.SEVERE, "Exception occured", e);
            throw e;
        }
    }
}
```

## <a name="build-the-application"></a>建置應用程式
Service Fabric Yeoman 範本包含 [Gradle](https://gradle.org/) 的建置指令碼，可用來從終端機建置應用程式。
從 Maven 擷取的 Service Fabric Java 相依性。 若要建置和處理 Service Fabric Java 應用程式，您必須確定已安裝 JDK 和 Gradle。 如果尚未加以安裝，請參閱[在 Linux 上開始使用 Service Fabric](service-fabric-get-started-linux.md#set-up-java-development)，以取得安裝 JDK 和 Gradle 的指示。

若要建置和封裝應用程式，請執行下列指令碼：

  ```bash
  cd HelloWorldActorApplication
  gradle
  ```

## <a name="deploy-the-application"></a>部署應用程式
建置應用程式後，可以將它部署到本機叢集。

1. 連線到本機 Service Fabric 叢集 (此叢集必須[已設定並執行](service-fabric-get-started-linux.md#set-up-a-local-cluster))。

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. 執行範本中所提供的安裝指令碼，將應用程式套件複製到叢集的映像存放區、註冊應用程式類型，以及建立應用程式的執行個體。

    ```bash
    ./install.sh
    ```

部署建置的應用程式與部署其他任何 Service Fabric 應用程式相同。 請參閱[使用 Service Fabric CLI 管理 Service Fabric 應用程式](service-fabric-application-lifecycle-sfctl.md)文件以取得詳細指示。

這些命令的參數可以在應用程式套件內產生的資訊清單中找到。

部署應用程式後，開啟瀏覽器並瀏覽至 [http://localhost:19080/Explorer](http://localhost:19080/Explorer) 的 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)。
接著展開 [應用程式] 節點，請注意，您的應用程式類型現在有一個項目，而另一個項目則在該類型的第一個執行個體。

> [!IMPORTANT]
> 若要將應用程式部署到 Azure 中的安全 Linux 叢集，您需要設定憑證來向 Service Fabric 執行階段驗證您的應用程式。 這樣做讓您的 Reliable Actors 服務能夠與基礎 Service Fabric 執行階段 API 進行通訊。 若要深入了解，請參閱[將 Reliable Services 應用程式設定為在 Linux 叢集上執行](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters)。  
>

## <a name="start-the-test-client-and-perform-a-failover"></a>啟動測試用戶端並執行容錯移轉
動作項目不會自行執行任何動作，它們需要其他服務或用戶端傳送訊息給它們。 動作項目範本包含簡單的測試指令碼，您可以用來與動作項目服務互動。

> [!Note]
> 測試用戶端會使用 ActorProxy 類別來與動作項目通訊，但這些動作項目必須在與動作項目服務相同的叢集內執行，或共用相同的 IP 位址空間。  您可以在本機開發叢集所在的相同電腦上執行測試用戶端。  不過，若要與遠端叢集中的動作項目進行通訊，您必須在叢集上部署一個閘道來處理與動作項目進行的外部通訊。

1. 使用監看式公用程式執行指令碼，以查看動作項目服務的輸出。  測試指令碼會在動作項目上呼叫 `setCountAsync()` 方法來遞增計數器，在動作項目上呼叫 `getCountAsync()` 方法以取得新的計數器值，並對主控台顯示該值。

   如果是 MAC OS X，您需要藉由執行下列其他命令，將 HelloWorldTestClient 資料夾複製到容器內的某些位置。    
    
    ```bash
     docker cp HelloWorldTestClient [first-four-digits-of-container-ID]:/home
     docker exec -it [first-four-digits-of-container-ID] /bin/bash
     cd /home
     ```

    ```bash
    cd HelloWorldActorTestClient
    watch -n 1 ./testclient.sh
    ```

2. 在 Service Fabric Explorer 中，找出裝載動作項目服務主要複本的節點。 在以下的螢幕擷取畫面中是節點 3。 主要服務複本會處理讀取和寫入作業。  服務狀態變更會接著複寫至次要複本 (在底下螢幕擷取畫面中的節點 0 和 1 上執行)。

    ![在 Service Fabric Explorer 中尋找主要複本][sfx-primary]

3. 在 [節點] 中，按一下您在上一個步驟中找到的節點，然後從 [動作] 功能表選取 [停用 (重新啟動)]。 此動作會重新啟動執行主要服務複本的節點，並強制容錯移轉至在另一個節點上執行的其中一個次要複本。  該次要複本會提升為主要，不同節點上會建立另一個次要複本，而且主要複本會開始採取讀取/寫入作業。 當節點重新啟動時，請留意測試用戶端的輸出，並注意儘管是容錯移轉，計數器仍會繼續增加。

## <a name="remove-the-application"></a>移除應用程式
使用範本中提供的解除安裝指令碼來刪除應用程式執行個體、取消註冊應用程式套件，並從叢集的映像存放區中移除此應用程式套件。

```bash
./uninstall.sh
```

在 Service Fabric 總管中，您可看到不再出現於 [應用程式] 節點中的應用程式和應用程式類型。

## <a name="service-fabric-java-libraries-on-maven"></a>Maven 上的 Service Fabric Java 程式庫
Service Fabric Java 程式庫已裝載於 Maven 中。 您可以在專案的 ``pom.xml`` 或 ``build.gradle`` 中新增相依性，以從 **mavenCentral** 使用 Service Fabric Java 程式庫。 

### <a name="actors"></a>動作項目

您應用程式的 Service Fabric Reliable Actor 支援。

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-actors</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-actors:1.0.0'
  }
  ```

### <a name="services"></a>服務

您應用程式的 Service Fabric Reliable Services 支援。

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-services</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-services:1.0.0'
  }
  ```

### <a name="others"></a>其他
#### <a name="transport"></a>傳輸

Service Fabric Java 應用程式的傳輸層支援。 除非您在傳輸層進行程式設計，否則不需要明確地將此相依性新增至 Reliable Actor 或服務應用程式。

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf-transport</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf-transport:1.0.0'
  }
  ```

#### <a name="fabric-support"></a>網狀架構支援

Service Fabric 的系統層級支援，其可與原生 Service Fabric 執行階段交談。 您不需要明確地將此相依性新增至 Reliable Actor 或服務應用程式。 當您納入上述其他相依性時，可從 Maven 自動提取此相依性。

  ```XML
  <dependency>
      <groupId>com.microsoft.servicefabric</groupId>
      <artifactId>sf</artifactId>
      <version>1.0.0</version>
  </dependency>
  ```

  ```gradle
  repositories {
      mavenCentral()
  }
  dependencies {
      compile 'com.microsoft.servicefabric:sf:1.0.0'
  }
  ```

## <a name="next-steps"></a>後續步驟

* [使用 Eclipse 在 Linux 上建立第一個 Service Fabric Java 應用程式](service-fabric-get-started-eclipse.md)
* [深入了解 Reliable Actors](service-fabric-reliable-actors-introduction.md)
* [使用 Service Fabric CLI 與 Service Fabric 叢集互動](service-fabric-cli.md)
* 了解 [Service Fabric 支援選項](service-fabric-support.md)
* [開始使用 Service Fabric CLI](service-fabric-cli.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png
