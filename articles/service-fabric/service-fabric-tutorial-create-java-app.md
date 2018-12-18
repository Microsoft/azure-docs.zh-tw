---
title: 在 Azure 中的 Service Fabric 上建立 Java 應用程式 | Microsoft Docs
description: 在本教學課程中，了解如何建立含有前端的可靠服務 Java 應用程式、建立具狀態可靠服務後端，以及將應用程式部署到叢集。
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
ms.date: 09/01/2018
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: e4552157cab846356c57a135d4e273f5a545bce9
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43667212"
---
# <a name="tutorial-create-an-application-with-a-java-web-api-front-end-service-and-a-stateful-back-end-service-on-service-fabric"></a>教學課程：在 Service Fabric 上使用 Java Web API 前端服務和具狀態後端服務建立應用程式

本教學課程是一個系列的第一部分。 當您完成時，您會有一個 Voting 應用程式，其 Java Web 前端會將投票結果儲存在叢集中具狀態的後端服務。 此教學課程系列要求您具備一部運作中的 Mac OSX 或 Linux 開發人員電腦。 如果您不需要以手動建立 Voting 應用程式，可以[下載已完成應用程式的原始程式碼](https://github.com/Azure-Samples/service-fabric-java-quickstart)並直接前往[逐步解說投票範例應用程式](service-fabric-tutorial-create-java-app.md#walk-through-the-voting-sample-application)。

![本機 Voting 應用程式](./media/service-fabric-tutorial-create-java-app/votingjavalocal.png)

在系列的第一部分中，您將了解如何：

> [!div class="checklist"]
> * 建立 Java 具狀態可靠服務
> * 建立 Java 無狀態 Web 應用程式服務
> * 使用服務遠端處理來與具狀態服務進行通訊
> * 在本機 Service Fabric 叢集上部署應用程式

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * 建置 Java Service Fabric Reliable Services 應用程式
> * [在本機叢集上部署及偵錯應用程式](service-fabric-tutorial-debug-log-local-cluster.md)
> * [將應用程式部署至 Azure 叢集](service-fabric-tutorial-java-deploy-azure.md)
> * [設定應用程式的監視和診斷](service-fabric-tutorial-java-elk.md)
> * [設定 CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* 在 [Mac](service-fabric-get-started-mac.md) 或 [Linux](service-fabric-get-started-linux.md) 上設定開發環境。 請遵循指示來安裝 Eclipse 外掛程式、 Gradle、Service Fabric SDK 和 Service Fabric CLI (sfctl)。

## <a name="create-the-front-end-java-stateless-service"></a>建立前端 Java 無狀態服務

首先，建立 Voting 應用程式的 Web 前端。 Java 無狀態服務代表輕量型 HTTP 伺服器，其裝載由 AngularJS 支援的 Web UI。 此無狀態服務會處理使用者的要求，並將其當作具狀態服務的遠端程序呼叫傳送以儲存選票。 

1. 啟動 Eclipse。

2. 透過 [檔案]->[新增]->[其他]->[Service Fabric]->[Service Fabric 專案] 來建立專案。

    ![Eclipse 中的 [新增專案] 對話方塊](./media/service-fabric-tutorial-create-java-app/create-sf-proj-wizard.png)

3. 在 [ServiceFabric 專案精靈] 對話方塊中，將專案命名為 **Voting**，然後按 [下一步]。

    ![在新增服務對話方塊中選擇 Java 具狀態服務](./media/service-fabric-tutorial-create-java-app/name-sf-proj-wizard.png) 

4. 在 [新增服務] 頁面上，選擇 [具狀態服務]，並將服務命名為 **VotingWeb**。 按一下 [完成] 以建立專案。

    ![建立無狀態服務]( ./media/service-fabric-tutorial-create-java-app/createvotingweb.png)

    Eclipse 會建立應用程式和服務專案並顯示在 [套件總管] 中。

    ![建立應用程式後的套件總管]( ./media/service-fabric-tutorial-create-java-app/eclipse-package-explorer.png)

下表提供前一個螢幕擷取畫面之套件總管中每個項目的簡短描述。 
| **套件總管項目** | **說明** |
| --- | --- |
| PublishProfiles | 包含 JSON 檔案，可描述本機和 Azure Service Fabric 叢集的設定檔詳細資料。 部署應用程式時，外掛程式會使用這些檔案的內容。 |
| 指令碼 | 包含協助程式指令碼，可從命令列用來快速管理您的應用程式和叢集。 |
| VotingApplication | 包含已推送至 Service Fabric 叢集的 Service Fabric 應用程式。 |
| VotingWeb | 包含前端無狀態服務原始程式檔，以及相關的 gradle 組建檔案。 |
| build.gradle | 用來管理專案的 Gradle 檔案。 |
| settings.gradle | 包含此資料夾中的 Gradle 專案名稱。 |

### <a name="add-html-and-javascript-to-the-votingweb-service"></a>將 HTML 和 Javascript 新增至 VotingWeb 服務

若要新增可由無狀態服務轉譯的 UI，請在 VotingApplication/VotingWebPkg/Code 中新增 HTML 檔案。 然後，無狀態 Java 服務中嵌入的輕量型 HTTP 伺服器會轉譯此 HTML 檔案。

1. 展開 VotingApplication 目錄以到達 VotingApplication/VotingWebPkg/Code 目錄。

2. 以滑鼠右鍵按一下 [Code] 目錄，然後按一下 [新增]->[資料夾]。

3. 將資料夾命名為 wwwroot，然後按一下 [完成]。

    ![Eclipse 建立 wwwroot 資料夾](./media/service-fabric-tutorial-create-java-app/create-wwwroot-folder.png)

4. 將檔案新增至名為 **index.html** 的 **wwwroot**，並將下列內容貼到這個檔案中。

```html
<!DOCTYPE html>
<html>
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
<script src="http://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.4/ui-bootstrap-tpls.min.js"></script>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
<body>

<script>
var app = angular.module('VotingApp', ['ui.bootstrap']); 
app.controller("VotingAppController", ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {
    $scope.votes = [];
    
    $scope.refresh = function () {
        $http.get('getStatelessList')
            .then(function successCallback(response) {
        $scope.votes = Object.assign(
          {},
          ...Object.keys(response.data) .
            map(key => ({[decodeURI(key)]: response.data[key]}))
        )
        },
        function errorCallback(response) {
            alert(response);
        });
    };

    $scope.remove = function (item) {            
       $http.get("removeItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };

    $scope.add = function (item) {
        if (!item) {return;}        
        $http.get("addItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });        
    };
}]);
</script>

<div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-offset-2">
                <form style="width:50% ! important;" class="center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item" />
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr />

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="(key, value)  in votes">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(key)">
                            <span class="pull-left">
                                {{key}}
                            </span>
                            <span class="badge pull-right">
                                {{value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
```

### <a name="update-the-votingwebjava-file"></a>更新 VotingWeb.java 檔案

在 **VotingWeb** 子專案中，開啟 VotingWeb/src/statelessservice/VotingWeb.java 檔案。 **VotingWebService** 服務是進入無狀態服務的閘道，負責設定前端 API 的通訊接聽程式。

以下列內容取代檔案中 **createServiceInstanceListeners** 方法的內容並儲存變更。

```java
@Override
protected List<ServiceInstanceListener> createServiceInstanceListeners() {

    EndpointResourceDescription endpoint = this.getServiceContext().getCodePackageActivationContext().getEndpoint(webEndpointName);
    int port = endpoint.getPort();
    
    List<ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
    listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationListener(context, port)));
    return listeners;
}
```

### <a name="add-the-httpcommunicationlistenerjava-file"></a>新增 HTTPCommunicationListener.java 檔案

HTTP 通訊接聽程式可作為控制器，以設定 HTTP 伺服器及公開用來定義投票動作的 API。 以滑鼠右鍵按一下 VotingWeb/src/statelessservice 資料夾中的 [statelessservice] 套件，然後選取 [新增] -> [檔案]。  將檔案命名為 HttpCommunicationListener.java，然後按一下 [完成]。

將檔案內容取代為下列項目，然後儲存變更。  稍後，在[更新 HttpCommunicationListener.java 檔案](#updatelistener_anchor)中，這個檔案會經過修改，以從後端服務轉譯、讀取和寫入投票資料。  現在，接聽程式只會傳回 Voting 應用程式的靜態 HTML。

```java
// ------------------------------------------------------------
//  Copyright (c) Microsoft Corporation.  All rights reserved.
//  Licensed under the MIT License (MIT). See License.txt in the repo root for license information.
// ------------------------------------------------------------

package statelessservice;

import com.google.gson.Gson;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.Headers;

import java.io.File;
import java.io.OutputStream;
import java.io.FileInputStream;

import java.net.InetSocketAddress;
import java.net.URI;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.logging.Level;
import java.util.logging.Logger;

import microsoft.servicefabric.services.communication.runtime.CommunicationListener;
import microsoft.servicefabric.services.runtime.StatelessServiceContext;
import microsoft.servicefabric.services.client.ServicePartitionKey;
import microsoft.servicefabric.services.remoting.client.ServiceProxyBase;
import microsoft.servicefabric.services.communication.client.TargetReplicaSelector;
import system.fabric.CancellationToken;

public class HttpCommunicationListener implements CommunicationListener {

    private static final Logger logger = Logger.getLogger(HttpCommunicationListener.class.getName());

    private static final String HEADER_CONTENT_TYPE = "Content-Type";
    private static final int STATUS_OK = 200;
    private static final int STATUS_NOT_FOUND = 404; 
    private static final int STATUS_ERROR = 500;
    private static final String RESPONSE_NOT_FOUND = "404 (Not Found) \n";
    private static final String MIME = "text/html";  
    private static final String ENCODING = "UTF-8";

    private static final String ROOT = "wwwroot/";
    private static final String FILE_NAME = "index.html";
    private StatelessServiceContext context;
    private com.sun.net.httpserver.HttpServer server;
    private ServicePartitionKey partitionKey;
    private final int port;

    public HttpCommunicationListener(StatelessServiceContext context, int port) {
        this.partitionKey = new ServicePartitionKey(0); 
        this.context = context;
        this.port = port;
    }

    // Called by openAsync when the class is instantiated 
    public void start() {
        try {
            logger.log(Level.INFO, "Starting Server");
            server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(this.port), 0);
        } catch (Exception ex) {
            logger.log(Level.SEVERE, null, ex);
            throw new RuntimeException(ex);
        }

        // Responsible for rendering the HTML layout described in the previous step
        server.createContext("/", new HttpHandler() {
            @Override
            public void handle(HttpExchange t) {
                try {
                    File file = new File(ROOT + FILE_NAME).getCanonicalFile();
    
                    if (!file.isFile()) {
                      // Object does not exist or is not a file: reject with 404 error.
                      t.sendResponseHeaders(STATUS_NOT_FOUND, RESPONSE_NOT_FOUND.length());
                      OutputStream os = t.getResponseBody();
                      os.write(RESPONSE_NOT_FOUND.getBytes());
                      os.close();
                    } else {    
                      Headers h = t.getResponseHeaders();
                      h.set(HEADER_CONTENT_TYPE, MIME);
                      t.sendResponseHeaders(STATUS_OK, 0);
    
                      OutputStream os = t.getResponseBody();
                      FileInputStream fs = new FileInputStream(file);
                      final byte[] buffer = new byte[0x10000];
                      int count = 0;
                      while ((count = fs.read(buffer)) >= 0) {
                        os.write(buffer,0,count);
                      }

                      fs.close();
                      os.close();
                    }  
                } catch (Exception e) {
                    logger.log(Level.WARNING, null, e);
                }
            }
        });

        /*
        [Replace this entire comment block in the 'Connect the services' section]
        */

        server.setExecutor(null);
        server.start();
    }

    //Helper method to parse raw HTTP requests
    private Map<String, String> queryToMap(String query){
        Map<String, String> result = new HashMap<String, String>();
        for (String param : query.split("&")) {
            String pair[] = param.split("=");
            if (pair.length>1) {
                result.put(pair[0], pair[1]);
            }else{
                result.put(pair[0], "");
            }
        }
        return result;
    }

    //Called closeAsync when the service is shut down
    private void stop() {
        if (null != server)
            server.stop(0);
    }

    //Called by the Service Fabric runtime when this service is created on a node
    @Override
    public CompletableFuture<String> openAsync(CancellationToken cancellationToken) {
        this.start();
                    logger.log(Level.INFO, "Opened Server");
        String publishUri = String.format("http://%s:%d/", this.context.getNodeContext().getIpAddressOrFQDN(), port);
        return CompletableFuture.completedFuture(publishUri);
    }

    //Called by the Service Fabric runtime when the service is shut down
    @Override
    public CompletableFuture<?> closeAsync(CancellationToken cancellationToken) {
        this.stop();
        return CompletableFuture.completedFuture(true);
    }

    //Called by the Service Fabric runtime to forcibly shut this listener down
    @Override
    public void abort() {
        this.stop();
    }
}
```

### <a name="configure-the-listening-port"></a>設定接聽連接埠

建立 VotingWebService 前端服務時，Service Fabric 會選取供服務接聽的連接埠。  VotingWeb 服務會充當此應用程式的前端，可接受外部流量，因此讓我們將該服務繫結至固定的已知連接埠。 在 [套件總管] 中，開啟 VotingApplication/VotingWebPkg/ServiceManifest.xml。  在 [資源] 區段中尋找 [端點] 資源，並將 [連接埠] 的值變更為 8080，或變更為另一個連接埠。 若要在本機上部署並執行應用程式，則必須在您的電腦上開啟應用程式接聽連接埠，並讓此連接埠可供使用。 將下列程式碼片段貼在 **ServiceManifest** 元素內 (也就是 ```<DataPackage>``` 元素的正下方)。

```xml
<Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
        <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
    </Endpoints>
  </Resources>
```

## <a name="add-a-stateful-back-end-service-to-your-application"></a>將具狀態後端服務新增到應用程式

現在已完成 Java Web API 服務的基本架構，讓我們繼續完成具狀態後端服務。

Service Fabric 可讓您使用可靠集合，直接在服務內以一致且可靠的方式儲存資料。 可靠集合是一組高可用性和可靠的集合類別。 任何用過 Java 集合的人都很熟悉這些類別的使用方式。

1. 在 [套件總管] 中，以滑鼠右鍵按一下應用程式專案中的 [Voting]，然後選擇 [Service Fabric] > [新增 Service Fabric 服務]。

2. 在 [新增服務] 對話方塊中，選擇 [具狀態服務]，並將服務命名為 **VotingDataService**，然後按一下 [新增服務]。

    建立服務專案後，您的應用程式中會有兩個服務。 隨著您繼續組建應用程式，您可以用相同的方式新增更多服務。 每個服務都可以獨立設定版本和升級。

3. Eclipse 會建立服務專案並顯示在 [套件總管] 中。

    ![Controllers\HomeController.cs](./media/service-fabric-tutorial-create-java-app/packageexplorercompletejava.png)

### <a name="add-the-votingdataservicejava-file"></a>新增 VotingDataService.java 檔案

VotingDataService.java 檔案包含一些方法，其中包含用於從可靠集合中擷取、新增和移除選票的邏輯。 將下列 **VotingDataService** 類別方法新增至 VotingDataService/src/statefulservice/VotingDataService.java 檔案。

```java
package statefulservice;

import java.util.HashMap;
import java.util.ArrayList;
import java.util.List; 
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.atomic.AtomicInteger;

import microsoft.servicefabric.services.communication.runtime.ServiceReplicaListener;

import microsoft.servicefabric.services.runtime.StatefulService;

import microsoft.servicefabric.services.remoting.fabrictransport.runtime.FabricTransportServiceRemotingListener;

import microsoft.servicefabric.data.ReliableStateManager;
import microsoft.servicefabric.data.Transaction;
import microsoft.servicefabric.data.collections.ReliableHashMap;
import microsoft.servicefabric.data.utilities.AsyncEnumeration;
import microsoft.servicefabric.data.utilities.KeyValuePair;

import system.fabric.StatefulServiceContext; 

import rpcmethods.VotingRPC; 

class VotingDataService extends StatefulService implements VotingRPC {
    private static final String MAP_NAME = "votesMap";
    private ReliableStateManager stateManager;
    
    protected VotingDataService (StatefulServiceContext statefulServiceContext) {
        super (statefulServiceContext);
    }

    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        this.stateManager = this.getReliableStateManager();
        ArrayList<ServiceReplicaListener> listeners = new ArrayList<>();
        
        listeners.add(new ServiceReplicaListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));
        
        return listeners;
    }
    
    // Method that will be invoked via RPC from the front end to retrieve the complete set of votes in the map
    public CompletableFuture<HashMap<String,String>> getList() {
        HashMap<String, String> tempMap = new HashMap<String, String>();

        try {                    

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();
                        
            Transaction tx = stateManager.createTransaction();
            AsyncEnumeration<KeyValuePair<String, String>> kv = votesMap.keyValuesAsync(tx).get();
            while (kv.hasMoreElementsAsync().get()) {
                KeyValuePair<String, String> k = kv.nextElementAsync().get();
                tempMap.put(k.getKey(), k.getValue()); 
            }
            
            tx.close();                    
            

        } catch (Exception e) {
            e.printStackTrace();
        }  
        
        return CompletableFuture.completedFuture(tempMap);
    }
    
    // Method that will be invoked via RPC from the front end to add an item to the votes list or to increase the
    // vote count for a particular item
    public CompletableFuture<Integer> addItem(String itemToAdd) {
        AtomicInteger status = new AtomicInteger(-1); 

        try {
            
            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();                    
            
            Transaction tx = stateManager.createTransaction();
            votesMap.computeAsync(tx, itemToAdd, (k, v) -> {
                if (v == null) {
                    return "1";
                }
                else {
                    int numVotes = Integer.parseInt(v);
                    numVotes = numVotes + 1;                            
                    return Integer.toString(numVotes);
                }
            }).get(); 
            
            tx.commitAsync().get();
            tx.close(); 

            status.set(1);                            
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }
    
    // Method that will be invoked via RPC from the front end to remove an item
    public CompletableFuture<Integer> removeItem(String itemToRemove) {
        AtomicInteger status = new AtomicInteger(-1); 
        try {
            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();
            
            Transaction tx = stateManager.createTransaction();
            votesMap.removeAsync(tx, itemToRemove).get();
            tx.commitAsync().get();
            tx.close();                    
            
            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        return CompletableFuture.completedFuture(new Integer(status.get()));
    }
    
}
```

## <a name="create-the-communication-interface-to-your-application"></a>建立您的應用程式的通訊介面

現在已建立無狀態前端服務和後端服務的基本架構。 下一步是連線這兩項服務。 前端和後端服務都會利用稱為 VotingRPC 的介面，該介面可定義 Voting 應用程式的作業。 前端和後端服務都會實作這個介面，以啟用這兩項服務之間的遠端程序呼叫 (RPC)。 Eclipse 不支援新增 Gradle 子專案，所以必須以手動方式新增包含此介面的套件。

1. 在 [套件總管] 中，以滑鼠右鍵按一下 **Voting** 專案，然後按一下 [新增] -> [資料夾]。 將資料夾命名為 **VotingRPC/src/rpcmethods**。

    ![建立 VotingRPC 套件](./media/service-fabric-tutorial-create-java-app/createvotingrpcpackage.png)

3. 在 *Voting/VotingRPC/src/rpcmethods* 之下建立一個名為 VotingRPC.java 的檔案，並將下列內容貼在 **VotingRPC.java** 檔案中。 

    ```java
    package rpcmethods;
    
    import java.util.ArrayList;
    import java.util.concurrent.CompletableFuture;
    import java.util.List;
    import java.util.HashMap;
    
    import microsoft.servicefabric.services.remoting.Service;
    
    public interface VotingRPC extends Service {
        CompletableFuture<HashMap<String, String>> getList();
    
        CompletableFuture<Integer> addItem(String itemToAdd);
    
        CompletableFuture<Integer> removeItem(String itemToRemove);
    }
    ``` 

4. 在 Voting/VotingRPC 目錄中建立名為 build.gradle 的檔案，並將下列內容貼在此檔案中。 這個 gradle 檔案用來建置和建立由其他服務匯入的 jar 檔案。 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/out"
        delete "${projectDir}/VotingRPC.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0')
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
        
        manifest {
                attributes(
                'Main-Class': 'rpcmethods.VotingRPC')
            baseName "VotingRPC"
            destinationDir = file('./')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA' 
    }
    
    defaultTasks 'clean', 'jar'
    ```

5. 在 Voting/settings.gradle 檔案中，新增以下一行以在組建中包含新建立的專案。 

    ```gradle 
    include ':VotingRPC'
    ```

6. 在 Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java 檔案中，以下列內容取代註解區塊。  

    ```java
    server.createContext("/getStatelessList", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {                    
                t.sendResponseHeaders(STATUS_OK,0);
                OutputStream os = t.getResponseBody();
                
                HashMap<String,String> list = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").getList().get();
                String json = new Gson().toJson(list);
                os.write(json.getBytes(ENCODING));                   
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    
    server.createContext("/removeItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                OutputStream os = t.getResponseBody();
                URI r = t.getRequestURI();     
    
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToRemove = params.get("item");                    
                
                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").removeItem(itemToRemove).get();
                
                if (num != 1) 
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
    
        }
    });
    
    server.createContext("/addItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                URI r = t.getRequestURI();
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToAdd = params.get("item");
                
                OutputStream os = t.getResponseBody();
                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").addItem(itemToAdd).get();
                if (num != 1) 
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    ```
7. 在 Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java 檔案的頂端新增適當的 import 陳述式。 

    ```java
    import rpcmethods.VotingRPC; 
    ```

在這個階段，前端、後端和 RPC 介面的功能都已完成。 下一個階段是先適當地設定 Gradle 指令碼，再部署至 Service Fabric 叢集。 

## <a name="walk-through-the-voting-sample-application"></a>逐步解說投票範例應用程式
投票應用程式包含兩個服務：
- Web 前端服務 (VotingWeb) - Java Web 前端服務，可提供網頁並公開 API 來與後端服務通訊。
- 後端服務 (VotingDataService) - Java Web 服務，可定義透過遠端程序呼叫 (RPC) 叫用來保存選票的方法。

![應用程式圖表](./media/service-fabric-tutorial-create-java-app/walkthroughjavavoting.png)

在應用程式中執行動作 (新增項目、投票、移除項目) 時，會發生下列事件：
1. JavaScript 會將適當的要求當作 HTTP 要求，傳送至 Web 前端服務中的 Web API。

2. Web 前端服務會使用 Service Fabric 內建的服務遠端處理功能來尋找要求，並將要求轉送至後端服務。 

3. 後端服務會定義一些方法，以更新可靠字典中的結果。 此可靠字典的內容會複寫到叢集中的多個節點並保存在磁碟上。 應用程式的所有資料都會儲存在叢集中。 

## <a name="configure-gradle-scripts"></a>設定 Gradle 指令碼 

在這一節中會設定專案的 Gradle 指令碼。 

1. 以下列內容取代 Voting/build.gradle 檔案的內容。

    ```gradle 
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    subprojects {
        apply plugin: 'java'
    } 
        
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

2. 以下列內容取代 Voting/VotingWeb/build.gradle 檔案的內容。

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/VotingWeb.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
        
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingWebPkg/Code/lib")
            include('lib*.so')
        }
    }
    
    compileJava.dependsOn(explodeDeps)
    
    jar {
        from configurations.compile.collect {(it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
        
        manifest {
            attributes(
                'Main-Class': 'statelessservice.VotingWebServiceHost')
            baseName "VotingWeb"
            destinationDir = file('../VotingApplication/VotingWebPkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA' 
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ``` 

3. 取代 Voting/VotingDataService/build.gradle 檔案的內容。 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/VotingDataService.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
        
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    compileJava.dependsOn(explodeDeps)
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingDataServicePkg/Code/lib")
            include('lib*.so')
        }
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
        
        manifest {
            attributes('Main-Class': 'statefulservice.VotingDataServiceHost')
    
            baseName "VotingDataService"
            destinationDir = file('../VotingApplication/VotingDataServicePkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA' 
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

## <a name="deploy-application-to-local-cluster"></a>將應用程式部署至本機叢集

此時，應用程式已準備好部署到本機 Service Fabric 叢集。

1. 在 [套件總管] 中，以滑鼠右鍵按一下 **Voting** 專案，然後按一下 [Service Fabric] -> [建置應用程式] 以建置應用程式。

2. 執行本機 Service Fabric 叢集。 此步驟取決於您的開發環境 (Mac 或 Linux)。

    如果您使用 Mac，請利用下列命令執行本機叢集：使用您自有工作區的路徑取代傳入 **-v** 參數的命令。

    ```bash
    docker run -itd -p 19080:19080 -p 8080:8080 -p --name sfonebox servicefabricoss/service-fabric-onebox
    ``` 

    如果您在 Linux 機器上執行，可使用下列命令啟動本機叢集： 

    ```bash 
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

4. 在 Eclipse 的 [套件總管] 中，以滑鼠右鍵按一下 **Voting** 專案，然後按一下 [Service Fabric] -> [發佈應用程式...]。 
5. 在 [發佈應用程式] 視窗中，從下拉式清單中選取 **Local.json**，然後按一下 [發佈]。
6. 請移至您的網頁瀏覽器並存取 **http://localhost:8080**，以檢視本機 Service Fabric 叢集上執行中的應用程式。 

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 建立 Java 服務成為具狀態可靠服務
> * 建立 Java 服務成為具狀態 Web 服務
> * 新增 Java 介面，以處理您的服務之間的遠端程序呼叫 (RPC)
> * 設定您的 Gradle 指令碼
> * 建置應用程式並將它部署到本機 Service Fabric 叢集

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [在本機叢集上偵錯和記錄應用程式](service-fabric-tutorial-debug-log-local-cluster.md)
