---
title: Azure Service Fabric JAVA 用戶端 API | Microsoft Docs
description: 使用 Service Fabric 用戶端 REST API 規格產生並使用 Service Fabric JAVA 用戶端 API
services: service-fabric
documentationcenter: java
author: rapatchi
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/27/2017
ms.author: rapatchi
ms.openlocfilehash: 987959742335940dca8eb57c54d593aea90dec15
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37111179"
---
# <a name="azure-service-fabric-java-client-apis"></a>Azure Service Fabric JAVA 用戶端 API

Service Fabric 用戶端 API 允許在 Azure、內部佈署、本機開發機器或其他雲端上，部署及管理微服務型應用程式和 Service Fabric 叢集內的容器。 本文說明如何在 Service Fabric client REST API 上產生及使用 Service Fabric JAVA 用戶端 API

## <a name="generate-the-client-code-using-autorest"></a>使用 AutoRest 產生用戶端程式碼

[AutoRest](https://github.com/Azure/autorest) 工具可產生用戶端程式庫以存取 RESTful Web 服務。 輸入 AutoRest 的是一種規格，用以描述使用 OpenAPI 規格格式的 REST API。 [Service Fabric 用戶端 REST API](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/servicefabric/data-plane) 會遵循此規格。

請按照下述步驟，使用 AutoRest 工具產生 Service Fabric JAVA 用戶端程式碼。

1. 在電腦上安裝 nodejs 和 NPM

    如果您是使用 Linux：
    ```bash
    sudo apt-get install npm
    sudo apt install nodejs
    ```
    如果您是使用 Mac OS X：
    ```bash
    brew install node
    ```

2. 使用 NPM 安裝 AutoRest。
    ```bash
    npm install -g autorest
    ```

3. 在本機電腦中派生及複製 [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs) 存放庫，並從您電腦的終端機移動至複製的位置。


4. 請前往您複製存放庫中的下述位置。
    ```bash
    cd specification\servicefabric\data-plane\Microsoft.ServiceFabric\stable\6.0
    ```

    > [!NOTE]
    > 如果您的叢集版本不是 6.0.*，請前往穩定資料夾中的正確目錄。
    >   

5. 執行下列 autorest 命令以產生 java 用戶端程式碼。
    
    ```bash
    autorest --input-file= servicefabric.json --java --output-folder=[output-folder-name] --namespace=[namespace-of-generated-client]
    ```
   下有示範 autorest 使用方式的範例。
   
    ```bash
    autorest --input-file=servicefabric.json --java --output-folder=java-rest-api-code --namespace=servicefabricrest
    ```
   
   下列命令會採用 ``servicefabric.json`` 規格檔案作為輸入，並在 ``java-rest-api-     code`` 資料夾中產生 java 用戶端程式碼，並在 ``servicefabricrest`` 命名空間中括住程式碼。 在此步驟之後，會發現 ``java-rest-api-code`` 資料夾產生了兩個資料夾 ``models``、``implemenation``，以及 ``ServiceFabricClientAPIs.java`` 和 ``package-info.java`` 兩個檔案。


## <a name="include-and-use-the-generated-client-in-your-project"></a>在專案中加入並使用產生的客戶端

1. 將產生的程式碼正確地加入專案中。 建議使用所產生的程式碼來建立程式庫，並將此程式庫加入您的專案。
2. 如果您要建立程式庫，請將下列相依性加入您程式庫的專案中。 如果所用的方法不同，請加入合適的相依性。

    ```
        GroupId:  com.microsoft.rest
        Artifactid: client-runtime
        Version: 1.2.1
    ```
    舉例而言，如果使用 Maven 建立系統，請將下列內容加入您的 ``pom.xml`` 檔案：

    ```xml
        <dependency>
          <groupId>com.microsoft.rest</groupId>
          <artifactId>client-runtime</artifactId>
          <version>1.2.1</version>
        </dependency>
    ```

3. 使用下列程式碼建立 RestClient：

    ```java
        RestClient simpleClient = new RestClient.Builder()
            .withBaseUrl("http://<cluster-ip or name:port>")
            .withResponseBuilderFactory(new ServiceResponseBuilder.Factory())
            .withSerializerAdapter(new JacksonAdapter())
            .build();
        ServiceFabricClientAPIs client = new ServiceFabricClientAPIsImpl(simpleClient);
    ```
4. 使用用戶端物件，並視需要進行適當的呼叫。 以下是示範用戶端物件使用方式的一些範例。 假定在使用下列 API 之前，已建立應用程式封裝，並上傳到映像存放區。
    * 佈建應用程式
    
        ```java
            ApplicationTypeImageStorePath imageStorePath = new ApplicationTypeImageStorePath();
            imageStorePath.withApplicationTypeBuildPath("<application-path-in-image-store>");
            client.provisionApplicationType(imageStorePath);
        ```
    * 建立應用程式

        ```java
            ApplicationDescription applicationDescription = new ApplicationDescription();
            applicationDescription.withName("<application-uri>");
            applicationDescription.withTypeName("<application-type>");
            applicationDescription.withTypeVersion("<application-version>");
            client.createApplication(applicationDescription);
        ```

## <a name="understanding-the-generated-code"></a>了解產生的程式碼
您會發現每一種 API 有四個實作的多載。 如果有選擇性參數，則會發現另外四個包含選擇性參數的變異。 例如，請思考 API ``removeReplica``。
 1. **public void removeReplica(String nodeName, UUID partitionId, String replicaId, Boolean forceRemove, Long timeout)**
    * 這是 removeReplica API 呼叫的同步變數
 2. **public ServiceFuture<Void> removeReplicaAsync(String nodeName, UUID partitionId, String replicaId, Boolean forceRemove, Long timeout, final ServiceCallback<Void> serviceCallback)**
    * 如果您想要使用未來式非同步程式設計並使用回呼，可使用此 API 呼叫的變數
 3. **public Observable<Void> removeReplicaAsync(String nodeName, UUID partitionId, String replicaId)**
    * 如果您想要使用回應式非同步程式設計，可使用此 API 呼叫的變數
 4. **public Observable<ServiceResponse<Void>> removeReplicaWithServiceResponseAsync(String nodeName, UUID partitionId, String replicaId)**
    * 如果您想要使用回應式非同步程式設計，並處理 RAW REST 回應，可使用此 API 呼叫的變數

## <a name="next-steps"></a>後續步驟
* 了解 [Service Fabric REST API](https://docs.microsoft.com/rest/api/servicefabric/)

