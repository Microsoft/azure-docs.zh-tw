---
title: 建立 Azure Service Fabric 容器應用程式 | Microsoft Docs
description: 在 Azure Service Fabric 上建立第一個 Windows 容器應用程式。 使用 Python 應用程式建置 Docker 映像、將映像推送到容器登錄，建置和部署 Service Fabric 容器應用程式。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: vturecek
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/18/2018
ms.author: ryanwi
ms.openlocfilehash: 4a5ca4879f81533e3617ca9dfe9cdf8afcf2965b
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43700166"
---
# <a name="create-your-first-service-fabric-container-application-on-windows"></a>在 Windows 建立第一個 Service Fabric 容器應用程式
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

在 Service Fabric 叢集上的 Windows 容器中執行現有的應用程式，不需要變更您的應用程式。 本文會逐步引導您建立包含 Python [Flask](http://flask.pocoo.org/) Web 應用程式的 Docker 映像，並將它部署到 Service Fabric 叢集。 您也將透過 [Azure Container Registry](/azure/container-registry/) 共用容器化應用程式。 本文假設您對 Docker 有基本認識。 您可藉由閱讀 [Docker 概觀](https://docs.docker.com/engine/understanding-docker/)來了解 Docker。

> [!NOTE]
> 本文適用於 Windows 開發環境。  Service Fabric 叢集執行階段與 Docker 執行階段必須在相同的作業系統上執行。  您無法在 Linux 叢集上執行 Windows 容器。

## <a name="prerequisites"></a>必要條件
* 執行下列項目的開發電腦︰
  * Visual Studio 2015 或 Visual Studio 2017。
  * [Service Fabric SDK 和工具](service-fabric-get-started.md)。
  *  Docker for Windows。 [取得 Docker CE for Windows (穩定)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description)。 安裝並啟動 Docker 之後，以滑鼠右鍵按一下系統匣圖示，然後選取 [切換至 Windows 容器]。 這是執行以 Windows 為基礎的 Docker 映像時的必要步驟。

* 有三個或更多節點在具有容器的 Windows Server 上執行的 Windows 叢集。 

  在本文中，在您的叢集節點上執行且具有容器的 Windows Server 版本 (組建) 必須符合您開發電腦上的 Windows Server 版本。 這是因為您會在開發電腦上建立 Docker 映像，而且容器 OS 版本與其部署所在主機 OS 版本之間有相容性條件約束。 如需詳細資訊，請參閱 [Windows Server 容器作業系統和主機作業系統的相容性](#windows-server-container-os-and-host-os-compatibility)。 
  
  若要判斷具有您叢集所需容器的 Windows Server 版本，請在開發電腦上從 Windows 命令提示字元執行 `ver` 命令：

  * 如果版本包含 x.x.14323.x，則請在[建立叢集](service-fabric-cluster-creation-via-portal.md)時選取 [WindowsServer 2016-Datacenter-with-Containers] 作為作業系統。 您也可以對合作對象叢集[免費試用 Service Fabric](https://aka.ms/tryservicefabric)。
  * 如果版本包含 x.x.16299.x，則請在[建立叢集](service-fabric-cluster-creation-via-portal.md)時選取 [WindowsServerSemiAnnual Datacenter-Core-1709-with-Containers] 作為作業系統。 不過，您無法使用合作對象叢集。

* Azure Container Registry 中的登錄 - 在 Azure 訂用帳戶中[建立容器登錄](../container-registry/container-registry-get-started-portal.md)。

> [!NOTE]
> 系統支援將容器部署到在 Windows 10 上執行的 Service Fabric 叢集。  如需如何設定 Windows 10 以執行 Windows 容器的資訊，請參閱[本文](service-fabric-how-to-debug-windows-containers.md)。
>   

> [!NOTE]
> Service Fabric 6.2 版和更新版本支援將容器部署到在 Windows Server 1709 版上執行的叢集。  
> 

## <a name="define-the-docker-container"></a>定義 Docker 容器
根據位於 Docker 中樞的 [Python 映像](https://hub.docker.com/_/python/)建立映像。

在 Dockerfile 中指定您的 Docker 容器。 Dockerfile 包含下列相關指示：設定您容器內的環境、載入您要執行的應用程式，以及對應連接埠。 Dockerfile 是 `docker build` 命令的輸入，該命令可建立映像。

建立空的目錄並建立 Dockerfile 檔案 (沒有副檔名)。 將下列內容新增至 Dockerfile 並儲存變更：

```
# Use an official Python runtime as a base image
FROM python:2.7-windowsservercore

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

如需詳細資訊，請參閱 [Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)。

## <a name="create-a-basic-web-application"></a>建立基本 Web 應用程式
建立 Flask Web 應用程式，其會在連接埠 80 上接聽並傳回 `Hello World!`。 在相同的目錄中，建立 requirements.txt 檔案。 新增下列內容並儲存變更：
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

<a id="Build-Containers"></a>
## <a name="build-the-image"></a>建立映像
執行 `docker build` 命令來建立可執行 Web 應用程式的映像。 開啟 PowerShell 視窗，然後瀏覽至包含 Dockerfile 的目錄。 執行以下命令：

```
docker build -t helloworldapp .
```

此命令會使用 Dockerfile 中的指示建立新映像，將此映像命名 (-t 標記) 為 `helloworldapp`。 若要建置容器映像，會先從作為應用程式新增目的地的 Docker Hub 下載基礎映像。 

建置命令完成後，執行 `docker images` 命令以查看新映像的資訊︰

```
$ docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              8ce25f5d6a79        2 minutes ago       10.4 GB
```

## <a name="run-the-application-locally"></a>在本機執行應用程式
先確認您的映像在本機，再將它推送至容器登錄。 

執行應用程式：

```
docker run -d --name my-web-site helloworldapp
```

name - 提供執行中容器的名稱 (而不是容器識別碼)。

容器啟動後，尋找其 IP 位址，以便您從瀏覽器連線到執行中的容器︰
```
docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" my-web-site
```

如果該命令未傳回任何項目，請執行下列命令，以及檢查 IP 位址 的 **NetworkSettings**->**Networks** 元素：
```
docker inspect my-web-site
```

連線到執行中的容器。 開啟 Web 瀏覽器並指向傳回的 IP 位址，例如 " http://172.31.194.61 "。 您應該會看到 "Hello World!" 標題 顯示在瀏覽器中。

若要停止您的容器，請執行︰

```
docker stop my-web-site
```

從您的開發電腦刪除容器：

```
docker rm my-web-site
```

<a id="Push-Containers"></a>
## <a name="push-the-image-to-the-container-registry"></a>將映像推送至容器登錄
確認容器在開發電腦上執行後，將映像發送到 Azure Container Registry 中您的登錄。

使用您的[登錄認證](../container-registry/container-registry-authentication.md)執行 ``docker login`` 登入容器登錄庫。

下列範例會傳遞 Azure Active Directory [service principal](../active-directory/develop/app-objects-and-service-principals.md) 的識別碼和密碼。 例如，您可能基於自動化案例已指派服務主體到您的登錄庫。 或者，您可以使用登錄使用者名稱和密碼進行登入。

```
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

下列命令會建立映像的標籤或別名，包含登錄的完整路徑。 這個範例會指定 ```samples``` 命名空間中的映像，以避免登錄根目錄雜亂。

```
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

將映像推送至容器登錄：

```
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="create-the-containerized-service-in-visual-studio"></a>在 Visual Studio 中建立容器化服務
Service Fabric SDK 和工具會提供一個服務範本，協助您建立容器化應用程式。

1. 啟動 Visual Studio。 選取 [檔案] > [新增] > [專案]。
2. 選取 [Service Fabric 應用程式]，將它命名為 "MyFirstContainer"，然後按一下 [確定]。
3. 從 [服務範本] 的清單中選取 [容器]。
4. 在 [映像名稱] 中輸入 "myregistry.azurecr.io/samples/helloworldapp"，也就是您推送至容器存放庫的映像。
5. 指定服務的名稱，然後按一下 [確定]。

## <a name="configure-communication"></a>設定通訊
容器化服務需要端點進行通訊。 將 `Endpoint` 元素以及通訊協定、連接埠和類型新增至 ServiceManifest.xml 檔案。 在此範例中，會使用固定的連接埠 8081。 若未指定連接埠，則會從應用程式連接埠範圍中隨機選擇連接埠。 

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
  </Endpoints>
</Resources>
```
> [!NOTE]
> 透過適用的屬性值來宣告其他 EndPoint 元素，即可為服務新增其他 EndPoint。 每個連接埠只能宣告一個通訊協定值。

透過定義端點，Service Fabric 會將端點發佈至名稱服務。 在叢集中執行的其他服務可以解析容器。 您也可以使用[反向 Proxy](service-fabric-reverseproxy.md)來執行容器對容器通訊。 將 HTTP 接聽連接埠和您想要與之通訊的服務名稱提供給反向 Proxy，並將這些都設為環境變數，便可進行通訊。

服務會接聽特定連接埠 (在此範例中為 8081)。 當應用程式部署至 Azure 中的叢集時，叢集和應用程式都會在 Azure 負載平衡器後方執行。 Azure 負載平衡器必須開啟應用程式連接埠，以便輸入的流量可以通過其傳到服務。  您可以使用 [PowerShell 指令碼](./scripts/service-fabric-powershell-open-port-in-load-balancer.md)或 [Azure 入口網站](https://portal.azure.com)，在 Azure 負載平衡器中開啟這個連接埠。

## <a name="configure-and-set-environment-variables"></a>設定環境變數
可以為服務資訊清單中的每個程式碼套件指定環境變數。 所有服務都有這項功能，無論是部署為容器或處理程序或來賓可執行檔。 您可以覆寫應用程式資訊清單中環境變數的值，或在部署期間將它們指定為應用程式參數。

下列服務資訊清單 XML 程式碼片段示範如何指定程式碼封裝的環境變數：
```xml
<CodePackage Name="Code" Version="1.0.0">
  ...
  <EnvironmentVariables>
    <EnvironmentVariable Name="HttpGatewayPort" Value=""/>    
  </EnvironmentVariables>
</CodePackage>
```

在應用程式資訊清單中可以覆寫這些環境變數︰

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
    <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
  </EnvironmentOverrides>
  ...
</ServiceManifestImport>
```

## <a name="configure-container-port-to-host-port-mapping-and-container-to-container-discovery"></a>設定容器連接埠對主機連接埠對應，以及容器對容器探索
設定用來與容器通訊的主機連接埠。 連接埠繫結會將服務在容器內接聽的連接埠，對應至主機上的連接埠。 在 ApplicationManifest.xml 檔案的 `ContainerHostPolicies` 元素中，新增 `PortBinding` 元素。 在本文中，`ContainerPort` 為 80 (如 Dockerfile 所指定，容器會公開連接埠 80)，而 `EndpointRef` 為 "Guest1TypeEndpoint" (先前在服務資訊清單中定義的端點)。 通訊埠 8081 上服務的連入要求會對應到容器上的連接埠 80。

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```
> [!NOTE]
> 透過適用的屬性值來宣告其他 PortBinding 元素，即可為服務新增其他 PortBinding。

## <a name="configure-container-registry-authentication"></a>設定容器登錄驗證
將 `RepositoryCredentials` 新增至 ApplicationManifest.xml 檔案的 `ContainerHostPolicies` 中，以設定容器登錄驗證。 為 myregistry.azurecr.io 容器登錄新增帳戶和密碼，讓服務從存放庫中下載容器映像。

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```

建議您使用已部署至叢集中所有節點的加密憑證，加密存放庫密碼。 當 Service Fabric 將服務套件部署至叢集時，會使用加密憑證將加密文字解密。 Invoke-ServiceFabricEncryptText Cmdlet 會用來建立密碼的加密文字，其已新增至 ApplicationManifest.xml 檔案。

下列指令碼會建立新的自我簽署憑證，並將其匯出為 PFX 檔案。 憑證已匯入現有的金鑰保存庫，接著已部署至 Service Fabric 叢集。

```powershell
# Variables.
$certpwd = ConvertTo-SecureString -String "Pa$$word321!" -Force -AsPlainText
$filepath = "C:\MyCertificates\dataenciphermentcert.pfx"
$subjectname = "dataencipherment"
$vaultname = "mykeyvault"
$certificateName = "dataenciphermentcert"
$groupname="myclustergroup"
$clustername = "mycluster"

$subscriptionId = "subscription ID"

Login-AzureRmAccount

Select-AzureRmSubscription -SubscriptionId $subscriptionId

# Create a self signed cert, export to PFX file.
New-SelfSignedCertificate -Type DocumentEncryptionCert -KeyUsage DataEncipherment -Subject $subjectname -Provider 'Microsoft Enhanced Cryptographic Provider v1.0' `
| Export-PfxCertificate -FilePath $filepath -Password $certpwd

# Import the certificate to an existing key vault. The key vault must be enabled for deployment.
$cer = Import-AzureKeyVaultCertificate -VaultName $vaultName -Name $certificateName -FilePath $filepath -Password $certpwd

Set-AzureRmKeyVaultAccessPolicy -VaultName $vaultName -ResourceGroupName $groupname -EnabledForDeployment

# Add the certificate to all the VMs in the cluster.
Add-AzureRmServiceFabricApplicationCertificate -ResourceGroupName $groupname -Name $clustername -SecretIdentifier $cer.SecretId
```
使用 [Invoke-ServiceFabricEncryptText](/powershell/module/servicefabric/Invoke-ServiceFabricEncryptText?view=azureservicefabricps) Cmdlet 加密密碼。

```powershell
$text = "=P==/==/=8=/=+u4lyOB=+=nWzEeRfF="
Invoke-ServiceFabricEncryptText -CertStore -CertThumbprint $cer.Thumbprint -Text $text -StoreLocation Local -StoreName My
```

以 [Invoke-ServiceFabricEncryptText](/powershell/module/servicefabric/Invoke-ServiceFabricEncryptText?view=azureservicefabricps) Cmdlet 傳回的加密文字來取代密碼，並將 `PasswordEncrypted` 設定為 "true"。

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <RepositoryCredentials AccountName="myregistry" Password="MIIB6QYJKoZIhvcNAQcDoIIB2jCCAdYCAQAxggFRMIIBTQIBADA1MCExHzAdBgNVBAMMFnJ5YW53aWRhdGFlbmNpcGhlcm1lbnQCEFfyjOX/17S6RIoSjA6UZ1QwDQYJKoZIhvcNAQEHMAAEg
gEAS7oqxvoz8i6+8zULhDzFpBpOTLU+c2mhBdqXpkLwVfcmWUNA82rEWG57Vl1jZXe7J9BkW9ly4xhU8BbARkZHLEuKqg0saTrTHsMBQ6KMQDotSdU8m8Y2BR5Y100wRjvVx3y5+iNYuy/JmM
gSrNyyMQ/45HfMuVb5B4rwnuP8PAkXNT9VLbPeqAfxsMkYg+vGCDEtd8m+bX/7Xgp/kfwxymOuUCrq/YmSwe9QTG3pBri7Hq1K3zEpX4FH/7W2Zb4o3fBAQ+FuxH4nFjFNoYG29inL0bKEcTX
yNZNKrvhdM3n1Uk/8W2Hr62FQ33HgeFR1yxQjLsUu800PrYcR5tLfyTB8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBybgM5NUV8BeetUbMR8mJhgFBrVSUsnp9B8RyebmtgU36dZiSObDsI
NtTvlzhk11LIlae/5kjPv95r3lw6DHmV4kXLwiCNlcWPYIWBGIuspwyG+28EWSrHmN7Dt2WqEWqeNQ==" PasswordEncrypted="true"/>
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```

## <a name="configure-isolation-mode"></a>設定隔離模式
Windows 支援兩種容器隔離模式：分別為處理序和 Hyper-V。 在處理序隔離模式中，在相同主機電腦上執行的所有容器都與主機共用核心。 在 Hyper-V 隔離模式中，會在每個 Hyper-V 容器與容器主機之間隔離核心。 隔離模式是在應用程式資訊清單檔中指定的 `ContainerHostPolicies` 元素。 可以指定的隔離模式有 `process`、`hyperv` 和 `default`。 預設值為 Windows Server 主機上的程序隔離模式。 在 Windows 10 主機上，僅支援 Hyper-V 隔離模式，因此不論其隔離模式設定為何，容器都會以 Hyper-V 隔離模式執行。 下列程式碼片段顯示如何在應用程式資訊清單檔中指定隔離模式。

```xml
<ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
```
   > [!NOTE]
   > 可在 Ev3 和具有巢狀虛擬化支援的 Dv3 Azure SKU 上使用 hyperv 隔離模式。 
   >
   >

## <a name="configure-resource-governance"></a>設定資源控管
[資源控管](service-fabric-resource-governance.md)可限制容器在主機上可使用的資源。 應用程式資訊清單中指定的 `ResourceGovernancePolicy` 元素是用於宣告服務程式碼封裝的資源限制。 下列資源可設定資源限制：記憶體、MemorySwap、CpuShares (CPU relative weight)、MemoryReservationInMB、BlkioWeight (BlockIO 相對權數)。 在此範例中，service package Guest1Pkg 會在其所在的叢集節點上獲得一個核心。 記憶體限制是絕對的，因此程式碼封裝會限制為 1024MB 的記憶體 (並具有同樣的彈性保證保留)。 程式碼套件 (容器或處理序) 無法配置超過此限制的記憶體，如果嘗試這麼做，將會導致發生記憶體不足的例外狀況。 若要讓資源限制強制能夠運作，應該為服務套件內的所有程式碼套件都指定記憶體限制。

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
  <Policies>
    <ServicePackageResourceGovernancePolicy CpuCores="1"/>
    <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
  </Policies>
</ServiceManifestImport>
```
## <a name="configure-docker-healthcheck"></a>設定 Docker HEALTHCHECK 

從 6.1 版開始，Service Fabric 會自動將 [Docker HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck) 事件整合至其系統健康情況報告。 這表示，如果您的容器已啟用 **HEALTHCHECK**，每當 Docker 報告容器的健康情況狀態發生變更時，Service Fabric 就會報告健康情況。 如果 health_status 為「狀況良好」，則 [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 中的健康情況報告會顯示 **OK (正常)**，如果 health_status 為「狀況不良」，則顯示 **WARNING (警告)**。 **HEALTHCHECK** 指令會指向針對監視容器健康情況而執行的實際檢查，該指令必須存在產生容器映像時使用的 Dockerfile 中。 

![HealthCheckHealthy][3]

![HealthCheckUnealthyApp][4]

![HealthCheckUnhealthyDsp][5]

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

## <a name="deploy-the-container-application"></a>部署容器應用程式
儲存所有變更，並建置應用程式。 若要發佈您的應用程式，以滑鼠右鍵按一下 [方案總管] 中的 **MyFirstContainer**，然後選取 [發佈]。

在 [連線端點] 中，輸入叢集的管理端點。 例如，"containercluster.westus2.cloudapp.azure.com:19000"。 在 [Azure 入口網站](https://portal.azure.com)中，您可以在叢集的 [概觀] 索引標籤中找到用戶端連線端點。

按一下 [發佈] 。

[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) 是一個 Web 型工具，可檢查和管理 Service Fabric 叢集中的應用程式與節點。 開啟瀏覽器並瀏覽至 http://containercluster.westus2.cloudapp.azure.com:19080/Explorer/ 然後遵循應用程式部署。 此應用程式會進行部署，但在叢集節點中下載映像之前 (視映像大小而定，這可能需要一些時間) 會處於錯誤狀態︰![錯誤][1]

應用程式處於 ```Ready``` 狀態時便已準備就緒︰![就緒][2]

開啟瀏覽器並瀏覽至 http://containercluster.westus2.cloudapp.azure.com:8081 。 您應該會看到 "Hello World!" 標題 顯示在瀏覽器中。

## <a name="clean-up"></a>清除
當叢集在執行時，您需要繼續支付費用，請考慮[刪除您的叢集](service-fabric-cluster-delete.md)。 [合作對象叢集](https://try.servicefabric.azure.com/)會在幾個小時後自動刪除。

將映像推送至容器登錄之後，您可以從開發電腦刪除本機映像︰

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="windows-server-container-os-and-host-os-compatibility"></a>Windows Server 容器作業系統和主機作業系統的相容性

Windows Server 容器在主機 OS 的所有版本之間不相容。 例如︰
 
- 使用 Windows Server 1709 版本所建置的 Windows Server 容器無法在執行 Windows Server 2016 版本的主機上運作。 
- 使用 Windows Server 2016 版本所建置的 Windows Server 容器在執行 Windows Server 1709 版本的主機上只能以 hyperv 隔離模式運作。 
- 透過使用 Windows Server 2016 所建置的 Windows Server 容器，可能必須確保在執行 Windows Server 2016 的主機上以程序隔離模式執行時，容器 OS 和主機 OS 的修訂版本相同。
 
若要深入了解，請參閱 [Windows 容器版本相容性](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility)。

在建置容器並部署到 Service Fabric 叢集時，請考慮主機 OS 和容器 OS 的相容性。 例如︰

- 請確定您部署的容器具有的 OS 與您叢集節點上的 OS 相容。
- 請確定為容器應用程式所指定的隔離模式，與其部署所在節點上的容器 OS 支援一致。
- 請考慮叢集節點或容器的 OS 升級可能會對其相容性造成何種影響。 

我們建議您利用下列作法，確定容器已正確部署在 Service Fabric 叢集上：

- 使用具有 Docker 映像的明確映像標記來指定用於建立容器的 Windows Server OS 版本。 
- 使用您應用程式資訊清單檔案中的 [OS 標記](#specify-os-build-specific-container-images)，來確定您的應用程式在不同的 Windows Server 版本和升級之間相容。

> [!NOTE]
> 使用 Service Fabric 6.2 版和更新版本，您可以在 Windows 10 主機上部署以 Windows Server 2016 為基礎的容器。 在 Windows 10 上，不論應用程式資訊清單中所設定的隔離模式為何，容器都會以 Hyper-V 隔離模式執行。 若要深入了解，請參閱[設定隔離模式](#configure-isolation-mode)。   
>
 
## <a name="specify-os-build-specific-container-images"></a>指定作業系統組建專屬的容器映像 

Windows Server 容器在不同的 OS 版本之間可能不相容。 例如，使用 Windows Server 2016 所建置的 Windows Server 容器無法在 Windows Server 1709 版本上以程序隔離模式運作。 因此，如果叢集節點更新為最新版本，則使用較早 OS 版本建置的容器服務可能會失敗。 為了避免執行階段 6.1 版和更新版本發生這種情況，Service Fabric 支援為每個容器指定多個 OS 映像，並以應用程式資訊清單中的 OS 組建版本標記它們。 您可藉由在 Windows 命令提示字元執行 `winver` 來取得 OS 組建版本。 更新應用程式資訊清單，並指定每個作業系統版本的映像覆寫後，再更新節點上的作業系統。 下列程式碼片段會示範如何在應用程式資訊清單 (**ApplicationManifest.xml**) 中指定多個容器映像：


```xml
      <ContainerHostPolicies> 
         <ImageOverrides> 
           <Image Name="myregistry.azurecr.io/samples/helloworldappDefault" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1701" Os="14393" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1709" Os="16299" /> 
         </ImageOverrides> 
      </ContainerHostPolicies> 
```
WIndows Server 2016 的組建版本為 14393，而 Windows Server 1709 版本的組建版本為 16299。 服務資訊清單可繼續為每個容器服務指定唯一一個映像，如下所示：

```xml
<ContainerHost>
    <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName> 
</ContainerHost>
```

   > [!NOTE]
   > 作業系統組建版本標記功能只適用於在 Windows 上的 Service Fabric
   >

如果虛擬機器上的基礎作業系統是組建 16299 (1709 版本)，Service Fabric 會挑選對應至該版 Windows Server 的容器映像。 如果未標記的容器映像也隨著已標記的容器映像一起提供，則 Service Fabric 會將未標記的映像視為可跨版本使用的其中一個容器映像。 明確地標記容器映像以免在升級期間發生問題。

未標記的容器映會覆寫 ServiceManifest 中提供的映像。 因此映像 "myregistry.azurecr.io/samples/helloworldappDefault" 會覆寫 ServiceManifest 中的 ImageName "myregistry.azurecr.io/samples/helloworldapp"。

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>完整範例 Service Fabric 應用程式和服務資訊清單
以下是本文中使用的完整服務和應用程式資訊清單。

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Guest1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="Guest1Type" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->    
    <EnvironmentVariables>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
    </EnvironmentVariables>

  </CodePackage>

  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to
           listen. Please note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="MyFirstContainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Guest1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
    <EnvironmentOverrides CodePackageRef="FrontendService.Code">
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
    </EnvironmentOverrides>
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="MIIB6QYJKoZIhvcNAQcDoIIB2jCCAdYCAQAxggFRMIIBTQIBADA1MCExHzAdBgNVBAMMFnJ5YW53aWRhdGFlbmNpcGhlcm1lbnQCEFfyjOX/17S6RIoSjA6UZ1QwDQYJKoZIhvcNAQEHMAAEg
gEAS7oqxvoz8i6+8zULhDzFpBpOTLU+c2mhBdqXpkLwVfcmWUNA82rEWG57Vl1jZXe7J9BkW9ly4xhU8BbARkZHLEuKqg0saTrTHsMBQ6KMQDotSdU8m8Y2BR5Y100wRjvVx3y5+iNYuy/JmM
gSrNyyMQ/45HfMuVb5B4rwnuP8PAkXNT9VLbPeqAfxsMkYg+vGCDEtd8m+bX/7Xgp/kfwxymOuUCrq/YmSwe9QTG3pBri7Hq1K3zEpX4FH/7W2Zb4o3fBAQ+FuxH4nFjFNoYG29inL0bKEcTX
yNZNKrvhdM3n1Uk/8W2Hr62FQ33HgeFR1yxQjLsUu800PrYcR5tLfyTB8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBybgM5NUV8BeetUbMR8mJhgFBrVSUsnp9B8RyebmtgU36dZiSObDsI
NtTvlzhk11LIlae/5kjPv95r3lw6DHmV4kXLwiCNlcWPYIWBGIuspwyG+28EWSrHmN7Dt2WqEWqeNQ==" PasswordEncrypted="true"/>
        <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
      </ContainerHostPolicies>
      <ServicePackageResourceGovernancePolicy CpuCores="1"/>
      <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this
         application type is created. You can also create one or more instances of service type using the
         ServiceFabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Guest1">
      <StatelessService ServiceTypeName="Guest1Type" InstanceCount="[Guest1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

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

若使用 6.2 版和更新版本的 Service Fabric 執行階段，您可以使用自訂引數啟動 Docker 精靈。 若指定了自訂引數，除了 `--pidfile` 引數外，Service Fabric 就不會再將任何其他引數傳遞至 Docker 引擎。 因此，不應該將 `--pidfile` 當作引數來傳遞。 此外，此引數也應該繼續讓 Docker 精靈在 Windows 的預設名稱管道 (若在 Linux 上，則是 Unix 網域通訊端) 上接聽，Service Fabric 才能與精靈通訊。 您可以在叢集資訊清單 [主控] 區段的 **ContainerServiceArguments** 底下傳遞自訂引數，如下列程式碼片段所示： 
 

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

[1]: ./media/service-fabric-get-started-containers/MyFirstContainerError.png
[2]: ./media/service-fabric-get-started-containers/MyFirstContainerReady.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[4]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[5]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png
