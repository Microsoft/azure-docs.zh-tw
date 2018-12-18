---
title: 以 Java 建立您的第一個 Azure Service Fabric 可靠服務 | Microsoft Docs
description: 概述使用無狀態與具狀態服務來建立 Microsoft Azure Service Fabric 應用程式。
services: service-fabric
documentationcenter: java
author: suhuruli
manager: timlt
editor: ''
ms.assetid: 7831886f-7ec4-4aef-95c5-b2469a5b7b5d
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/02/2017
ms.author: suhuruli
ms.openlocfilehash: d4e3419241d44744f8a692896848edb6cebb56a0
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44049689"
---
# <a name="get-started-with-reliable-services"></a>開始使用 Reliable Service
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-quick-start.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-services-quick-start-java.md)
>
>

本文說明 Azure Service Fabric Reliable Services 的基本概念，並將逐步引導您建立及部署以 Java 撰寫的簡單 Reliable Services 應用程式。 此 Microsoft Virtual Academy 影片也示範如何建立無狀態的 Reliable Service：<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=DOX8K86yC_206218965">  
<img src="./media/service-fabric-reliable-services-quick-start-java/ReliableServicesJavaVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

## <a name="installation-and-setup"></a>安裝與設定
開始之前，確定機器上已設定 Service Fabric 開發環境。
如果您需要加以設定，請移至[在 Mac 上開始使用](service-fabric-get-started-mac.md)或[在 Linux 上開始使用](service-fabric-get-started-linux.md)。

## <a name="basic-concepts"></a>基本概念
若要開始使用 Reliable Services，您只需要了解幾個基本概念：

* **服務類型**：這是您的服務實作。 它是由您撰寫的類別定義，它會擴充 `StatelessService` 與其中使用的任何其他程式碼或相依性，以及名稱與版本號碼。
* **具名服務執行個體**：若要執行您的服務，可以建立您服務類型的具名執行個體，很像您建立類別類型的物件執行個體一樣。 服務執行個體實際上就是您所撰寫服務類型的物件具現化。
* **服務主機**：您建立的具名服務執行個體需要在主機內部執行。 服務主機只是您服務的執行個體可執行所在的程序。
* **服務註冊**：註冊可將所有項目結合在一起。 服務類型必須在服務主機的 Service Fabric 執行階段中註冊，以允許 Service Fabric 建立其執行個體來執行。  

## <a name="create-a-stateless-service"></a>建立無狀態服務
從建立 Service Fabric 應用程式開始著手。 適用於 Linux 的 Service Fabric SDK 包含 Yeoman 產生器，可提供具無狀態服務之 Service Fabric 應用程式的樣板。 執行下列 Yeoman 命令開始作業：

```bash
$ yo azuresfjava
```

遵循指示建立**可靠的無狀態服務**。 本教學課程中，將應用程式命名為 HelloWorldApplication，將服務命名為 HelloWorld。 結果會包含 `HelloWorldApplication` 和 `HelloWorld` 的目錄。

```bash
HelloWorldApplication/
├── build.gradle
├── HelloWorld
│   ├── build.gradle
│   └── src
│       └── statelessservice
│           ├── HelloWorldServiceHost.java
│           └── HelloWorldService.java
├── HelloWorldApplication
│   ├── ApplicationManifest.xml
│   └── HelloWorldPkg
│       ├── Code
│       │   ├── entryPoint.sh
│       │   └── _readme.txt
│       ├── Config
│       │   └── _readme.txt
│       ├── Data
│       │   └── _readme.txt
│       └── ServiceManifest.xml
├── install.sh
├── settings.gradle
└── uninstall.sh
```
### <a name="service-registration"></a>服務註冊
會取消必須在 Service Fabric 執行階段註冊。 服務類型是在 `ServiceManifest.xml` 中以及實作 `StatelessService` 的服務類別中定義。 服務註冊是在程序主要進入點中執行。 在此範例中，程序的主要進入點是 `HelloWorldServiceHost.java`：

```java
public static void main(String[] args) throws Exception {
    try {
        ServiceRuntime.registerStatelessServiceAsync("HelloWorldType", (context) -> new HelloWorldService(), Duration.ofSeconds(10));
        logger.log(Level.INFO, "Registered stateless service type HelloWorldType.");
        Thread.sleep(Long.MAX_VALUE);
    }
    catch (Exception ex) {
        logger.log(Level.SEVERE, "Exception in registration:", ex);
        throw ex;
    }
}
```

## <a name="implement-the-service"></a>實作服務

開啟 **HelloWorldApplication/HelloWorld/src/statelessservice/HelloWorldService.java**。 這個類別會定義服務類型，而且可以執行任何程式碼。 服務 API 為您的程式碼提供兩個進入點：

* 開放式的進入點方法 (稱為 `runAsync()`)，您可以在這裡開始執行任何工作負載，包括長時間執行的計算工作負載。

```java
@Override
protected CompletableFuture<?> runAsync(CancellationToken cancellationToken) {
    ...
}
```

* 通訊進入點，您可以在這裡插入選擇的通訊堆疊。 這就是您可以開始接收來自使用者和其他服務要求的地方。

```java
@Override
protected List<ServiceInstanceListener> createServiceInstanceListeners() {
    ...
}
```

本教學課程著重在 `runAsync()` 進入點方法。 這就是您可以立即開始執行程式碼的地方。

### <a name="runasync"></a>RunAsync
當服務的執行個體已放置並且可以執行時，平台會呼叫這個方法。 針對無狀態服務，這表示開啟服務執行個體的時間。 會提供取消語彙基元以協調服務執行個體何時需要關閉。 在 Service Fabric 中，服務執行個體的開啟/關閉循環可能會在整個服務存留期中發生許多次。 發生這種情形的原因有很多，包括：

* 系統可能為了資源平衡移動您的服務執行個體。
* 在您的程式碼中發生錯誤。
* 應用程式或系統已升級。
* 基礎硬體發生中斷。

此協調流程是由 Service Fabric 管理，可讓您的服務維持高度可用且正確平衡。

`runAsync()` 不應該同步封鎖。 您的 runAsync 實作應該傳回 CompletableFuture，以允許執行階段繼續執行。 如果您的工作負載需要實作長時間執行的工作，則該工作應該在 CompletableFuture 內完成。

#### <a name="cancellation"></a>取消
取消您的工作負載是由所提供的取消語彙基元協調的協同努力。 系統會先等待您的工作結束 (依據成功完成、取消或錯誤)，然後才繼續執行。 系統要求取消時，務必接受取消權杖、完成任何工作，並儘快結束 `runAsync()`。 下列範例示範如何處理取消事件：

```java
@Override
protected CompletableFuture<?> runAsync(CancellationToken cancellationToken) {

    // TODO: Replace the following sample code with your own logic
    // or remove this runAsync override if it's not needed in your service.

    return CompletableFuture.runAsync(() -> {
        long iterations = 0;
        while(true)
        {
        cancellationToken.throwIfCancellationRequested();
        logger.log(Level.INFO, "Working-{0}", ++iterations);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex){}
        }
    });
}
```

在這個無狀態服務範例中，計數會儲存在本機變數。 但是，因為這是無狀態服務，所儲存的值只針對其服務執行個體的目前生命週期而存在。 當服務移動或重新啟動時，值將會遺失。

## <a name="create-a-stateful-service"></a>建立具狀態服務
Service Fabric 導入了一種可設定狀態的新服務。 具狀態服務能夠可靠地在服務本身維持狀態，和使用它的程式碼共置。 狀態是由 Service Fabric 設為高度可用，而不需要將狀態保存到外部存放區。

若要將計數器值從無狀態轉換成高度可用且持續，即使服務移動或重新啟動亦然，您需要具狀態服務。

在與 HelloWorld 應用程式相同的目錄中，您可以執行 `yo azuresfjava:AddService` 命令以新增服務。 為您的架構選擇「可靠的具狀態服務」，並將服務命名為 "HelloWorldStateful"。 

您的應用程式現在應該有兩個服務：無狀態服務 HelloWorld 和具狀態服務 HelloWorldStateful。

具狀態服務與無狀態服務具有相同的進入點。 主要差異在於可以可靠地儲存狀態的狀態供應器可用性。 Service Fabric 隨附稱為「可靠的集合」的狀態供應器實作，它可讓您透過可靠狀態管理員建立複寫的資料結構。 具狀態可靠服務預設會使用此狀態供應器。

在 **HelloWorldStateful -> src** 中開啟 HelloWorldStateful.java，其中包含下列 RunAsync 方法：

```java
@Override
protected CompletableFuture<?> runAsync(CancellationToken cancellationToken) {
    Transaction tx = stateManager.createTransaction();
    return this.stateManager.<String, Long>getOrAddReliableHashMapAsync("myHashMap").thenCompose((map) -> {
        return map.computeAsync(tx, "counter", (k, v) -> {
            if (v == null)
                return 1L;
            else
                return ++v;
            }, Duration.ofSeconds(4), cancellationToken)
                .thenCompose((r) -> tx.commitAsync())
                .whenComplete((r, e) -> {
            try {
                tx.close();
            } catch (Exception e) {
                logger.log(Level.SEVERE, e.getMessage());
            }
        });
    });
}
```

### <a name="runasync"></a>RunAsync
`RunAsync()` 在具狀態和無狀態服務中的運作方式類似。 只不過在具狀態服務中，平台會代替您執行額外的工作，然後才執行 `RunAsync()`。 這項工作包含確保可靠狀態管理員和可靠的集合可以使用。

### <a name="reliable-collections-and-the-reliable-state-manager"></a>可靠的集合與可靠狀態管理員
```java
ReliableHashMap<String,Long> map = this.stateManager.<String, Long>getOrAddReliableHashMapAsync("myHashMap")
```

[ReliableHashMap](https://docs.microsoft.com/java/api/microsoft.servicefabric.data.collections._reliable_hash_map) 是一個字典實作，您可以在服務中可靠地儲存狀態。 有了 Service Fabric 和可靠的 HashMaps，您可以直接在服務中儲存資料，而不需要外部的持續性存放區。 可靠的 HashMaps 可讓您的資料具備高可用性。 Service Fabric 會藉由為您建立與管理服務的多個複本  來完成此作業。 它也提供 API 讓管理這些複本和其狀態轉換的複雜性抽象化。

可靠的集合可以儲存任何 JAVA 型別 (包括您的自訂型別)，不過有幾個需要注意的事項：

* Service Fabric 藉由在節點之間「複寫」狀態來使您的狀態高度可用，而可靠的 HashMap 會將您的資料儲存到每個複本上的本機磁碟。 這表示所有儲存在可靠的 HashMaps 中的項目必須「可序列化」。 
* 當您在可靠的 HashMaps 上認可交易時，物件會複寫以獲得高可用性。 儲存在可靠的 HashMaps 中的物件會保留在服務中的本機記憶體。 這表示您有物件的本機參考。
  
   很重要的一點是，您不要改變那些物件的本機執行個體而不在交易中的可靠集合上執行更新作業。 這是因為不會自動複寫對本機物件執行個體所做的變更。 您必須將物件重新插入到字典中，或在字典上使用其中一個「更新」方法。

可靠狀態管理員會為您管理可靠的 HashMaps。 在您的服務中，您可以隨時隨地以名稱向可靠狀態管理員要求可靠的集合。 可靠狀態管理員會確保您取回參考。 不建議您將可靠集合執行個體的參考儲存在類別成員變數或屬性中。 請特別小心以確保在服務生命週期中隨時將參考設定為執行個體。 可靠狀態管理員會為您處理這項工作，並且針對重複造訪最佳化。


### <a name="transactional-and-asynchronous-operations"></a>交易式和非同步作業
```java
return map.computeAsync(tx, "counter", (k, v) -> {
    if (v == null)
        return 1L;
    else
        return ++v;
    }, Duration.ofSeconds(4), cancellationToken)
        .thenCompose((r) -> tx.commitAsync())
        .whenComplete((r, e) -> {
    try {
        tx.close();
    } catch (Exception e) {
        logger.log(Level.SEVERE, e.getMessage());
    }
});
```

可靠 HashMaps 上的作業是非同步的。 這是因為具備可靠集合的寫入作業執行 I/O 作業以將資料複寫並保存至磁碟。

可靠的 HashMap 作業為「交易式」作業，因此您可以在多個可靠的 HashMaps 和作業之間維持狀態一致。 比方說，您可能會從可靠的字典取得一個工作項目、對它執行作業，然後將結果儲存在另一個可靠的 HashMap 中，全都在單一交易中完成。 這會被視為不可部分完成的作業，而且它可保證整個作業都會成功，或整個作業都會回復。 如果您從佇列取消項目之後，但在您儲存結果之前發生錯誤，那麼會回復整個交易，且項目會保持在佇列中進行處理。


## <a name="build-the-application"></a>建置應用程式

Yeoman 樣板包含可建置應用程式的 gradle 指令碼，以及可部署和移除應用程式的 bash 指令碼。 若要執行應用程式，先建置含 gradle的應用程式︰

```bash
$ gradle
```

這會產生可使用 Service Fabric CLI 來部署的 Service Fabric 應用程式套件。

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

## <a name="next-steps"></a>後續步驟

* [開始使用 Service Fabric CLI](service-fabric-cli.md)
