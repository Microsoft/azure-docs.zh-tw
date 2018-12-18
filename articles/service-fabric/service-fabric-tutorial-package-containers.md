---
title: 封裝和部署容器作為 Azure 中的 Service Fabric 應用程式 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Yeoman 來產生 Azure Service Fabric 應用程式定義以及封裝應用程式。
services: service-fabric
documentationcenter: ''
author: suhuruli
manager: timlt
editor: suhuruli
tags: servicefabric
keywords: Docker、容器、微服務、Service Fabric、Azure
ms.assetid: ''
ms.service: service-fabric
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/12/2017
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: 952550225f2bdd8559d72a9d283993451ae7f60b
ms.sourcegitcommit: a62cbb539c056fe9fcd5108d0b63487bd149d5c3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42616338"
---
# <a name="tutorial-package-and-deploy-containers-as-a-service-fabric-application-using-yeoman"></a>教學課程：使用 Yeoman 封裝和部署容器作為 Service Fabric 應用程式

本教學課程是一個系列中的第二部分。 本教學課程使用範本產生器工具 (Yeoman)，來產生 Service Fabric 應用程式定義。 此應用程式接著可用來將容器部署到 Service Fabric。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 安裝 Yeoman
> * 使用 Yeoman 建立應用程式封裝
> * 設定應用程式封裝中的設定來與容器搭配使用
> * 建置應用程式
> * 部署和執行應用程式
> * 清除應用程式

## <a name="prerequisites"></a>必要條件

* 使用推送到在本教學課程系列的[第 1 部分](service-fabric-tutorial-create-container-images.md)中建立之 Azure Container Registry 的容器映像。
* 已[設定](service-fabric-tutorial-create-container-images.md) Linux 開發環境。

## <a name="install-yeoman"></a>安裝 Yeoman

Service Fabric 提供 Scaffolding 工具，可協助您使用 Yeoman 範本產生器，從終端機建立應用程式。 依照下列步驟執行，以確定您具有 Yeoman 範本產生器。

1. 在電腦上安裝 nodejs 和 NPM。 請注意，Mac OSX 使用者將需使用封裝管理員 Homebrew

    ```bash
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
    nvm install node 
    ```
2. 從 NPM 在您的電腦上安裝 Yeoman 範本產生器

    ```bash
    npm install -g yo
    ```
3. 安裝 Service Fabric Yeoman 容器產生器

    ```bash 
    npm install -g generator-azuresfcontainer
    ```

## <a name="package-a-docker-image-container-with-yeoman"></a>使用 Yeoman 封裝 Docker 映像容器

1. 若要建立 Service Fabric 容器應用程式，在複製存放庫的 'container-tutorial' 目錄中，執行下列命令。

    ```bash
    yo azuresfcontainer
    ```
2. 請輸入 "TestContainer" 命名您的應用程式
3. 請輸入 "azurevotefront" 命名您的應用程式服務。
4. 為前端存放庫提供 ACR 中的容器映像路徑，例如 '\<acrName>.azurecr.io/azure-vote-front:v1'。 \<acrName> 欄位的值必須與上一個教學課程中所使用的值相同。
5. 按 Enter 鍵，讓 Commands 區段保留空白。
6. 將執行個體計數指定為 1。

下圖顯示執行 yo 命令的輸入和輸出：

```bash
? Name your application TestContainer
? Name of the application service: azurevotefront
? Input the Image Name: <acrName>.azurecr.io/azure-vote-front:v1
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/TestContainer/ApplicationManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/ServiceManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/config/Settings.xml
   create TestContainer/TestContainer/azurevotefrontPkg/code/Dummy.txt
   create TestContainer/install.sh
   create TestContainer/uninstall.sh
```

若要將其他容器服務新增至已使用 Yeoman 建立的應用程式，請執行下列步驟︰

1. 將目錄變更一個層級至 **TestContainer** 目錄，例如 *./TestContainer*
2. 執行 `yo azuresfcontainer:AddService`
3. 將服務命名為 'azurevoteback'
4. 為 Redis 提供容器映像路徑 - 'alpine:redis'
5. 按 Enter 鍵，讓 Commands 區段保留空白
6. 指定執行個體計數為 "1"。

用於新增所用服務的項目均會顯示：

```bash
? Name of the application service: azurevoteback
? Input the Image Name: alpine:redis
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/azurevotebackPkg/ServiceManifest.xml
   create TestContainer/azurevotebackPkg/config/Settings.xml
   create TestContainer/azurevotebackPkg/code/Dummy.txt
```

針對本教學課程的其餘部分，我們會在 **TestContainer** 目錄中運作。 例如，*./TestContainer/TestContainer*。 此目錄的內容應該如下所示。

```bash
$ ls
ApplicationManifest.xml azurevotefrontPkg azurevotebackPkg
```

## <a name="configure-the-application-manifest-with-credentials-for-azure-container-registry"></a>使用適用於 Azure Container Registry 的認證來設定應用程式資訊清單

針對要從 Azure Container Registry 提取容器映像的 Service Fabric，我們需要在 **ApplicationManifest.xml** 中提供認證。

登入您的 ACR 執行個體。 使用 **az acr login** 命令來完成此作業。 在建立容器登錄時，為它提供唯一名稱。

```bash
az acr login --name <acrName>
```

此命令在完成之後會傳回**登入成功**訊息。

接下來，執行下列命令以取得容器登錄的密碼。 Service Fabric 會使用此密碼來向 ACR 進行驗證以提取容器映像。

```bash
az acr credential show -n <acrName> --query passwords[0].value
```

在 **ApplicationManifest.xml** 中，針對前端服務，在 **ServiceManifestImport** 元素下方加入程式碼片段。 針對 **AccountName** 欄位插入您的 **acrName**，並針對 **Password** 欄位使用從前一個命令傳回的密碼。 本文件結尾會提供完整的 **ApplicationManifest.xml**。

```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code">
    <RepositoryCredentials AccountName="<acrName>" Password="<password>" PasswordEncrypted="false"/>
  </ContainerHostPolicies>
</Policies>
```

## <a name="configure-communication-and-container-port-to-host-port-mapping"></a>設定通訊和容器連接埠對主機連接埠的對應

### <a name="configure-communication-port"></a>設定通訊連接埠

設定 HTTP 端點，讓用戶端可以與您的服務通訊。 開啟 ./TestContainer/azurevotefrontPkg/ServiceManifest.xml 檔案，並在 **ServiceManifest** 元素中宣告端點資源。  新增通訊協定、連接埠和名稱。 在本教學課程中，服務會接聽連接埠 80。 下列程式碼片段放在資源中的 *ServiceManifest* 標記下方。

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
  </Endpoints>
</Resources>

```

同樣地，修改適用於後端服務的服務資訊清單。 開啟 *./TestContainer/azurevotebackPkg/ServiceManifest.xml* 檔案，並在 **ServiceManifest** 元素中宣告端點資源。 本教學課程會保留 redis 預設值 6379。 下列程式碼片段放在資源中的 *ServiceManifest* 標記下方。

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
  </Endpoints>
</Resources>
```

提供 **UriScheme**，就會自動向 Service Fabric 命名服務註冊容器端點以供搜尋。 本文結尾會針對後端服務提供完整的 ServiceManifest.xml 範例檔案作為範例。

### <a name="map-container-ports-to-a-service"></a>將容器連接埠對應至服務

為了公開叢集中的容器，我們也需要在 'ApplicationManifest.xml' 中建立連接埠繫結。 **PortBinding** 原則會參考我們在 **ServiceManifest.xml** 檔案中定義的 **Endpoints**。 將要求傳入這些端點，會對應至已在此處開啟並繫結的容器連接埠。 在 **ApplicationManifest.xml** 檔案中，新增下列程式碼，將連接埠 80 和 6379 繫結至端點。 本文件結尾會提供完整的 **ApplicationManifest.xml**。

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
</ContainerHostPolicies>
```

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
</ContainerHostPolicies>
```

### <a name="add-a-dns-name-to-the-backend-service"></a>將 DNS 名稱新增至後端服務

針對要將這個 DNS 名稱指派至後端服務的 Service Fabric，必須將名稱指定於 **ApplicationManifest.xml** 中。 將 **ServiceDnsName** 屬性新增至 **Service** 元素，如下所示：

```xml
<Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
  <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
    <SingletonPartition/>
  </StatelessService>
</Service>
```

前端服務會讀取環境變數，以了解 Redis 執行個體的 DNS 名稱。 此環境變數已經定義在用來產生 Docker 映像的 Docerkfile 中，這裡不需要採取任何動作。

```Dockerfile
ENV REDIS redisbackend.testapp
```

下列程式碼片段說明前端的 Python 程式碼如何挑選 Dockerfile 中所述的環境變數。 這裡不需要採取任何動作。

```python
# Get DNS Name
redis_server = os.environ['REDIS']

# Connect to the Redis store
r = redis.StrictRedis(host=redis_server, port=6379, db=0)
```

在本教學課程中，現在可以使用適用於服務封裝應用程式的範本來部署到叢集。 在後續的教學課程中，此應用程式會部署並執行於 Service Fabric 叢集中。

## <a name="create-a-service-fabric-cluster"></a>建立 Service Fabric 叢集

若要將應用程式部署到 Azure 中的叢集，請建立您自己的叢集。

合作對象叢集是 Azure 上裝載的免費、限時 Service Fabric 叢集。 這類叢集是由任何人皆可部署應用程式並了解平台的 Service Fabric 小組所執行。 若要存取合作對象叢集，請[遵循指示](http://aka.ms/tryservicefabric)。

如需在安全的合作對象叢集上執行管理作業，您可以使用 Service Fabric Explorer、CLI 或 Powershell。 若要使用 Service Fabric Explorer，您必須從合作對象叢集網站下載 PFX 檔案，並將憑證匯入憑證存放區 (Windows 或 Mac) 或瀏覽器本身 (Ubuntu)。 合作對象叢集中的自我簽署憑證沒有任何密碼。

若要使用 Powershell 或 CLI 執行管理作業，您需要 PFX (Powershell) 或 PEM (CLI)。 若要將 PFX 轉換成 PEM 檔案，請執行下列命令：

```bash
openssl pkcs12 -in party-cluster-1277863181-client-cert.pfx -out party-cluster-1277863181-client-cert.pem -nodes -passin pass:
```

如需建立您自己叢集的資訊，請參閱[在 Azure 上建立您的 Service Fabric 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)。

## <a name="build-and-deploy-the-application-to-the-cluster"></a>建置應用程式並部署到叢集

您可以使用 Service Fabric CLI，將應用程式部署到 Azure 叢集。 如果您的電腦上並未安裝 Service Fabric CLI，請依照[這裡](service-fabric-get-started-linux.md#set-up-the-service-fabric-cli)的指示來安裝它。

連線到 Azure 中的 Service Fabric 叢集。 以您自己的端點取代範例端點。 端點必須是類似以下的完整 URL。

```bash
sfctl cluster select --endpoint https://linh1x87d1d.westus.cloudapp.azure.com:19080 --pem party-cluster-1277863181-client-cert.pem --no-verify
```

使用 **TestContainer** 目錄中所提供的安裝指令碼，將應用程式封裝複製到叢集的映像存放區、註冊應用程式類型，以及建立應用程式的執行個體。

```bash
./install.sh
```

開啟瀏覽器，並瀏覽至 http://lin4hjim3l4.westus.cloudapp.azure.com:19080/Explorer 上的 Service Fabric Explorer。 展開 [應用程式] 節點，請注意，有一個適用於您應用程式類型的項目，另一個則適用於執行個體。

![Service Fabric Explorer][sfx]

為了連接到執行中應用程式，請開啟網頁瀏覽器並移至叢集 URL，例如 http://lin0823ryf2he.cloudapp.azure.com:80 。 您應會在 Web UI 中看到投票應用程式。

![votingapp][votingapp]

## <a name="clean-up"></a>清除

使用範本中提供的解除安裝指令碼，刪除叢集中的應用程式執行個體並取消註冊應用程式類型。 這個命令會花一些時間來清除執行個體，而且無法在此指令碼之後立即執行 'install'sh' 命令。

```bash
./uninstall.sh
```

## <a name="examples-of-completed-manifests"></a>已完成的資訊清單範例

### <a name="applicationmanifestxml"></a>ApplicationManifest.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ApplicationManifest ApplicationTypeName="TestContainerType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotefrontPkg" ServiceManifestVersion="1.0.0"/>
    <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myaccountname" Password="<password>" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
    </ContainerHostPolicies>
        </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotebackPkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
          <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
        </ContainerHostPolicies>
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="azurevotefront">
      <StatelessService ServiceTypeName="azurevotefrontType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
    <Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
      <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

### <a name="front-end-servicemanifestxml"></a>Front-end ServiceManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotefrontPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotefrontType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>acrName.azurecr.io/azure-vote-front:v1</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
    </Endpoints>
  </Resources>

 </ServiceManifest>
```

### <a name="redis-servicemanifestxml"></a>Redis ServiceManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotebackPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotebackType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>alpine:redis</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>
     <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
    </Endpoints>
  </Resources>
 </ServiceManifest>
```

## <a name="next-steps"></a>後續步驟

本教學課程會使用 Yeoman，將多個容器封裝為 Service Fabric 應用程式。 此應用程式接著會部署並執行於 Service Fabric 叢集上。 已完成下列步驟：

> [!div class="checklist"]
> * 安裝 Yeoman
> * 使用 Yeoman 建立應用程式封裝
> * 設定應用程式封裝中的設定來與容器搭配使用
> * 建置應用程式
> * 部署和執行應用程式
> * 清除應用程式

前進到下一個教學課程，以了解如何在 Service Fabric 中容錯移轉和調整應用程式。

> [!div class="nextstepaction"]
> [了解如何容錯移轉和調整應用程式](service-fabric-tutorial-containers-failover.md)

[votingapp]: ./media/service-fabric-tutorial-deploy-run-containers/votingapp.png
[sfx]: ./media/service-fabric-tutorial-deploy-run-containers/containerspackagetutorialsfx.png