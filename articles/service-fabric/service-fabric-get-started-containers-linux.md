---
title: 在 Linux 上建立 Azure Service Fabric 容器應用程式 | Microsoft Docs
description: 在 Azure Service Fabric 上建立第一個 Linux 容器應用程式。 使用您的應用程式建置 Docker 映像、將映像推送到容器登錄，建置和部署 Service Fabric 容器應用程式。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 1/09/2018
ms.author: ryanwi
ms.openlocfilehash: 133b0d483eb47a76ad2525a430b09b82259c118f
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698242"
---
# <a name="create-your-first-service-fabric-container-application-on-linux"></a>在 Linux 建立第一個 Service Fabric 容器應用程式
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

在 Service Fabric 叢集上的 Linux 容器中執行現有的應用程式，無需變更您的應用程式。 本文會逐步引導您建立包含 Python [Flask](http://flask.pocoo.org/) Web 應用程式的 Docker 映像，並將它部署到 Service Fabric 叢集。 您也將透過 [Azure Container Registry](/azure/container-registry/) 共用容器化應用程式。 本文假設您對 Docker 有基本認識。 您可藉由閱讀 [Docker 概觀](https://docs.docker.com/engine/understanding-docker/)來了解 Docker。

> [!NOTE]
> 本文適用於 Linux 開發環境。  Service Fabric 叢集執行階段與 Docker 執行階段必須在相同的作業系統上執行。  您無法在 Windows 叢集上執行 Linux 容器。

## <a name="prerequisites"></a>必要條件
* 執行下列項目的開發電腦︰
  * [Service Fabric SDK 和工具](service-fabric-get-started-linux.md)。
  * [Docker CE for Linux](https://docs.docker.com/engine/installation/#prior-releases). 
  * [Service Fabric CLI](service-fabric-cli.md)

* Azure Container Registry 中的登錄 - 在 Azure 訂用帳戶中[建立容器登錄](../container-registry/container-registry-get-started-portal.md)。 

## <a name="define-the-docker-container"></a>定義 Docker 容器
根據位於 Docker 中樞的 [Python 映像](https://hub.docker.com/_/python/)建立映像。 

在 Dockerfile 中指定您的 Docker 容器。 Dockerfile 包含下列相關指示：設定您容器內的環境、載入您要執行的應用程式，以及對應連接埠。 Dockerfile 是 `docker build` 命令的輸入，該命令可建立映像。 

建立空的目錄並建立 Dockerfile 檔案 (沒有副檔名)。 將下列內容新增至 Dockerfile 並儲存變更：

```
# Use an official Python runtime as a base image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

如需詳細資訊，請參閱 [Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)。

## <a name="create-a-basic-web-application"></a>建立基本 Web 應用程式
建立 Flask Web 應用程式，其會在連接埠 80 上接聽並傳回 "Hello World!"。 在相同的目錄中，建立 requirements.txt 檔案。 新增下列內容並儲存變更：
```
Flask
```

此外，建立 app.py 檔案並新增下列程式碼片段：

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    
    return 'Hello World!'

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

## <a name="build-the-image"></a>建立映像
執行 `docker build` 命令來建立可執行 Web 應用程式的映像。 開啟 PowerShell 視窗並瀏覽至 *c:\temp\helloworldapp*。 執行以下命令：

```bash
docker build -t helloworldapp .
```

此命令會使用 Dockerfile 中的指示建立新映像，將此映像命名 (-t 標記) 為 `helloworldapp`。 若要建置容器映像，會先從作為應用程式新增目的地的 Docker Hub 下載基礎映像。 

建置命令完成後，執行 `docker images` 命令以查看新映像的資訊︰

```bash
$ docker images
    
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              86838648aab6        2 minutes ago       194 MB
```

## <a name="run-the-application-locally"></a>在本機執行應用程式
先確認您的容器化應用程式在本機執行，再將它推送至容器登錄。 

執行此應用程式，並將您電腦的通訊埠 4000 對應至容器的已公開連接埠 80：

```bash
docker run -d -p 4000:80 --name my-web-site helloworldapp
```

name - 提供執行中容器的名稱 (而不是容器識別碼)。

連線到執行中的容器。 開啟 Web 瀏覽器並指向連接埠 4000 上傳回的 IP 位址，例如 " http://localhost:4000 "。 您應該會看到 "Hello World!" 標題 顯示在瀏覽器中。

![Hello World!][hello-world]

若要停止您的容器，請執行︰

```bash
docker stop my-web-site
```

從您的開發電腦刪除容器：

```bash
docker rm my-web-site
```

## <a name="push-the-image-to-the-container-registry"></a>將映像推送至容器登錄
確認應用程式在 Docker 中執行後，將映像推送至 Azure Container Registry 中您的登錄。

使用您的[登錄認證](../container-registry/container-registry-authentication.md)執行 `docker login` 登入容器登錄庫。

下列範例會傳遞 Azure Active Directory [service principal](../active-directory/develop/app-objects-and-service-principals.md) 的識別碼和密碼。 例如，您可能基於自動化案例已指派服務主體到您的登錄庫。 或者，您可以使用登錄使用者名稱和密碼進行登入。

```bash
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

下列命令會建立映像的標籤或別名，包含登錄的完整路徑。 這個範例會指定 `samples` 命名空間中的映像，以避免登錄根目錄雜亂。

```bash
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

將映像推送至容器登錄：

```bash
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="package-the-docker-image-with-yeoman"></a>使用 Yeoman 封裝 Docker 映像
適用於 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 產生器，可讓您輕鬆建立應用程式並新增容器映像。 讓我們使用 Yeoman 來建立具有單一 Docker 容器，名為 *SimpleContainerApp* 的應用程式。

若要建立 Service Fabric 容器應用程式，請開啟終端機視窗並執行 `yo azuresfcontainer`。 

將應用程式命名 (例如，`mycontainer`)，並將應用程式服務命名 (例如，`myservice`)。

針對映像名稱，在容器登錄中提供容器映像的 URL (例如，"myregistry.azurecr.io/samples/helloworldapp")。 

因為此映像已定義工作負載進入點，您不需要明確指定輸入命令 (命令會在容器內執行，這會讓容器在啟動後繼續執行)。 

指定執行個體計數為 "1"。

以適當的格式指定連接埠對應。 針對本文章，您需要提供 ```80:4000``` 當作連接埠對應。 這樣做之後，原本要傳送至主機電腦之連接埠 4000 的任何送入要求，都會被重新導向至容器上的連接埠 80。

![容器的 Service Fabric Yeoman 產生器][sf-yeoman]

## <a name="configure-container-repository-authentication"></a>設定容器存放庫驗證
 如果您的容器需要向私人存放庫進行驗證，則新增 `RepositoryCredentials`。 在本文中，新增 myregistry.azurecr.io 容器登錄的帳戶名稱和密碼。 確保該原則已新增在對應至正確服務套件的 'ServiceManifestImport' 標籤之下。

```xml
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyServicePkg" ServiceManifestVersion="1.0.0" />
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
   </ServiceManifestImport>
``` 


## <a name="configure-isolation-mode"></a>設定隔離模式
使用 6.3 執行階段版本，可針對 Linux 容器支援 VM 隔離，因而可針對容器支援兩種格式模式：處理序和 HyperV。 使用 HyperV 隔離模式，在每個容器與容器主機之間隔離核心。 HyperV 隔離是使用 [Clear Containers](https://software.intel.com/en-us/articles/intel-clear-containers-2-using-clear-containers-with-docker) \(英文\) 來實作的。 隔離模式是在應用程式資訊清單檔的 `ServicePackageContainerPolicy` 元素中針對 Linux 叢集所指定的。 可以指定的隔離模式有 `process`、`hyperv` 和 `default`。 預設值為處理序隔離模式。 下列程式碼片段顯示如何在應用程式資訊清單檔中指定隔離模式。

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyServicePkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ServicePackageContainerPolicy Hostname="votefront" Isolation="hyperv">
          <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
        </ServicePackageContainerPolicy>
    </Policies>
  </ServiceManifestImport>
```


## <a name="configure-resource-governance"></a>設定資源控管
[資源控管](service-fabric-resource-governance.md)可限制容器在主機上可使用的資源。 應用程式資訊清單中指定的 `ResourceGovernancePolicy` 元素是用於宣告服務程式碼封裝的資源限制。 下列資源可設定資源限制：記憶體、MemorySwap、CpuShares (CPU relative weight)、MemoryReservationInMB、BlkioWeight (BlockIO 相對權數)。 在此範例中，service package Guest1Pkg 會在其所在的叢集節點上獲得一個核心。 記憶體限制是絕對的，因此程式碼封裝會限制為 1024MB 的記憶體 (並具有同樣的彈性保證保留)。 程式碼套件 (容器或處理序) 無法配置超過此限制的記憶體，如果嘗試這麼做，將會導致發生記憶體不足的例外狀況。 若要讓資源限制強制能夠運作，應該為服務套件內的所有程式碼套件都指定記憶體限制。

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="MyServicePKg" ServiceManifestVersion="1.0.0" />
  <Policies>
    <ServicePackageResourceGovernancePolicy CpuCores="1"/>
    <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
  </Policies>
</ServiceManifestImport>
```




## <a name="configure-docker-healthcheck"></a>設定 Docker HEALTHCHECK 
從 6.1 版開始，Service Fabric 會自動將 [Docker HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck) 事件整合至其系統健康情況報告。 這表示，如果您的容器已啟用 **HEALTHCHECK**，每當 Docker 報告容器的健康情況狀態發生變更時，Service Fabric 就會報告健康情況。 如果 health_status 為「狀況良好」，則 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 中的健康情況報告會顯示 **OK (正常)**，如果 health_status 為「狀況不良」，則顯示 **WARNING (警告)**。 **HEALTHCHECK** 指令會指向針對監視容器健康情況而執行的實際檢查，該指令必須存在產生容器映像時使用的 Dockerfile 中。 

![HealthCheckHealthy][1]

![HealthCheckUnealthyApp][2]

![HealthCheckUnhealthyDsp][3]

您可以將 **HealthConfig** 選項指定為 ApplicationManifest 中 **ContainerHostPolicies** 的一部份，為每個容器設定 **HEALTHCHECK** 行為。

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="ContainerServicePkg" ServiceManifestVersion="2.0.0" />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <HealthConfig IncludeDockerHealthStatusInSystemHealthReport="true" RestartContainerOnUnhealthyDockerHealthStatus="false" />
      </ContainerHostPolicies>
    </Policies>
</ServiceManifestImport>
```
根據預設，*IncludeDockerHealthStatusInSystemHealthReport* 會設為 **true**，而 *RestartContainerOnUnhealthyDockerHealthStatus* 會設為 **false**。 如果 *RestartContainerOnUnhealthyDockerHealthStatus* 設為 **true**，則報告中重複出現狀況不良的容器就會重新啟動 (可能在其他節點上重新啟動)。

如果您需要停用整個 Service Fabric 叢集的 **HEALTHCHECK** 整合，就必須將 [EnableDockerHealthCheckIntegration](service-fabric-cluster-fabric-settings.md) 設為 **false**。

## <a name="deploy-the-application"></a>部署應用程式
建置應用程式後，可以使用 Service Fabric CLI 將它部署到本機叢集。

連接到本機 Service Fabric 叢集。

```bash
sfctl cluster select --endpoint http://localhost:19080
```

使用範本中所提供的安裝指令碼，將應用程式套件複製到叢集的映像存放區、註冊應用程式類型，以及建立應用程式的執行個體。

```bash
./install.sh
```

開啟瀏覽器並瀏覽至位於 http://localhost:19080/Explorer 的 Service Fabric Explorer (如果在 Mac OS X 上使用 Vagrant，請以 VM 的私人 IP 取代 localhost)。 展開 [應用程式] 節點，請注意，您的應用程式類型現在有一個項目，而另一個則是該類型的第一個執行個體。

連線到執行中的容器。 開啟 Web 瀏覽器並指向連接埠 4000 上傳回的 IP 位址，例如 " http://localhost:4000 "。 您應該會看到 "Hello World!" 標題 顯示在瀏覽器中。

![Hello World!][hello-world]


## <a name="clean-up"></a>清除
使用範本中提供的解除安裝指令碼，刪除本機開發叢集中的應用程式執行個體並取消註冊應用程式類型。

```bash
./uninstall.sh
```

將映像推送至容器登錄之後，您可以從開發電腦刪除本機映像︰

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>完整範例 Service Fabric 應用程式和服務資訊清單
以下是本文中使用的完整服務和應用程式資訊清單。

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="myservicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="myserviceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying containers 
      to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->
    
    <EnvironmentVariables>
      <!--
      <EnvironmentVariable Name="VariableName" Value="VariableValue"/>
      -->
    </EnvironmentVariables>
    
  </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="myServiceTypeEndpoint" UriScheme="http" Port="4000" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="mycontainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="myservicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="myservice">
      <!-- On a local development cluster, set InstanceCount to 1. On a multi-node production 
      cluster, set InstanceCount to -1 for the container service to run on every node in 
      the cluster.
      -->
      <StatelessService ServiceTypeName="myserviceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
## <a name="adding-more-services-to-an-existing-application"></a>將更多服務新增至現有的應用程式

若要將其他容器服務新增至已使用 yeoman 建立的應用程式，請執行下列步驟︰

1. 將目錄變更為現有應用程式的根目錄。 例如，如果 `MyApplication` 是 Yeoman 所建立的應用程式，則為 `cd ~/YeomanSamples/MyApplication`。
2. 執行 `yo azuresfcontainer:AddService`

<a id="manually"></a>


## <a name="configure-time-interval-before-container-is-force-terminated"></a>在容器被迫終止前設定時間間隔

您可以在啟動服務刪除 (或移至另一個節點) 之後，設定執行階段在容器移除前所要等候的間隔時間。 設定時間間隔可將 `docker stop <time in seconds>` 命令傳送至容器。  如需更多詳細資訊，請參閱 [docker stop](https://docs.docker.com/engine/reference/commandline/stop/)。 所要等候的時間間隔指定於 `Hosting` 區段之下。 下列叢集資訊清單程式碼片段示範如何設定等候間隔：


```json
{
        "name": "Hosting",
        "parameters": [
          {
                "name": "ContainerDeactivationTimeout",
                "value" : "10"
          },
          ...
        ]
}
```

預設時間間隔會設定為 10 秒。 因為此組態是動態的，所以叢集的僅限組態升級會更新逾時。 

## <a name="configure-the-runtime-to-remove-unused-container-images"></a>將執行階段設定為移除未使用的容器映像

您可以將 Service Fabric 叢集設定為從節點移除未使用的容器映像。 如果節點上存在太多容器映像，此組態允許重新擷取磁碟空間。 若要啟用此功能，請更新叢集資訊清單中的 `Hosting` 區段，如下列程式碼片段所示： 


```json
{
        "name": "Hosting",
        "parameters": [
          {
                "name": "PruneContainerImages",
                "value": "True"
          },
          {
                "name": "ContainerImagesToSkip",
                "value": "microsoft/windowsservercore|microsoft/nanoserver|microsoft/dotnet-frameworku|..."
          }
          ...
          }
        ]
} 
```

對於不應刪除的映像，您可以在 `ContainerImagesToSkip` 參數之下加以指定。 

## <a name="configure-container-image-download-time"></a>設定容器映像下載時間

Service Fabric 執行階段會配置 20 分鐘來下載及擷取容器映像，這適用於大部分的容器映像。 針對大型映像或緩慢的網路連線速度，您可能需要延長中止下載及擷取映像之前的等待時間。 您可以在叢集資訊清單的 **Hosting** 區段中使用 **ContainerImageDownloadTimeout** 屬性來設定此逾時，如下列程式碼片段所示：

```json
{
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
}
```


## <a name="set-container-retention-policy"></a>設定容器保留原則

為了協助診斷容器啟動的失敗情形，Service Fabric (6.1 版或更新版本) 可保留啟動終止或是無法啟動的容器。 此原則可以在 **ApplicationManifest.xml** 檔案中設定，如下列程式碼片段所示：

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

**ContainersRetentionCount** 設定會指定容器失敗時要保留的容器數目。 如果指定負數值，就會保留所有失敗的容器。 如果未指定 **ContainersRetentionCount** 屬性，就不會保留任何容器。 **ContainersRetentionCount** 屬性也支援應用程式參數，因此使用者可以為測試和生產叢集指定不同的值。 使用此功能時，請使用位置限制將容器服務鎖定在特定節點上，以避免容器服務移到其他節點上。 使用此功能的所有容器皆需要手動移除。

## <a name="start-the-docker-daemon-with-custom-arguments"></a>使用自訂引數啟動 Docker 精靈

若使用 6.2 版和更新版本的 Service Fabric 執行階段，您可以使用自訂引數啟動 Docker 精靈。 若指定了自訂引數，除了 `--pidfile` 引數外，Service Fabric 就不會再將任何其他引數傳遞至 Docker 引擎。 因此，不應該將 `--pidfile` 當作引數來傳遞。 此外，此引數也應該繼續讓 Docker 精靈在 Windows 的預設名稱管道 (若在 Linux 上，則是 Unix 網域通訊端) 上接聽，Service Fabric 才能與精靈通訊。 您可以在叢集資訊清單 [主控] 區段的 **ContainerServiceArguments** 底下指定自訂引數。 範例如下列程式碼片段所示： 
 

```json
{ 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
} 

```

## <a name="next-steps"></a>後續步驟
* 深入了解如何[在 Service Fabric 上執行容器](service-fabric-containers-overview.md)。
* 閱讀[在容器中部署 .NET 應用程式](service-fabric-host-app-in-a-container.md)教學課程。
* 深入了解 Service Fabric [應用程式生命週期](service-fabric-application-lifecycle.md)。
* 請查看 GitHub 上的[ Service Fabric 容器程式碼範例](https://github.com/Azure-Samples/service-fabric-containers)。

[hello-world]: ./media/service-fabric-get-started-containers-linux/HelloWorld.png
[sf-yeoman]: ./media/service-fabric-get-started-containers-linux/YoSF.png

[1]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[2]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png
