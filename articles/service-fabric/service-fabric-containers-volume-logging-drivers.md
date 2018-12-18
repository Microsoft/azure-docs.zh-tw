---
title: Service Fabric Azure 檔案服務磁碟區驅動程式 (預覽) | Microsoft Docs
description: Service Fabric 支援使用 Azure 檔案服務以備份來自您容器的磁碟區。 這目前為預覽狀態。
services: service-fabric
documentationcenter: other
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: ab49c4b9-74a8-4907-b75b-8d2ee84c6d90
ms.service: service-fabric
ms.devlang: other
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 6/10/2018
ms.author: subramar
ms.openlocfilehash: 9bd370e8070816d62b22c1e3d5ad4b6cdd2da30a
ms.sourcegitcommit: 727a0d5b3301fe20f20b7de698e5225633191b06
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39144946"
---
# <a name="service-fabric-azure-files-volume-driver-preview"></a>Service Fabric Azure 檔案服務磁碟區驅動程式 (預覽)
Azure 檔案服務磁碟區外掛程式為 [Docker 磁碟區外掛程式](https://docs.docker.com/engine/extend/plugins_volume/) \(英文\)，針對 Docker 容器提供以 [Azure 檔案服務](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)為基礎的磁碟區。 此 Docker 磁碟區外掛程式會封裝為可部署至 Service Fabric 叢集的 Service Fabric 應用程式。 其目的是為部署至叢集的其他 Service Fabric 容器應用程式，提供以 Azure 檔案服務為基礎的磁碟區。

> [!NOTE]
> 6.255.389.9494 版的 Azure 檔案服務磁碟區外掛程式為預覽版，只在此文件中提供。 因為是預覽版本，此版本並**不**支援在生產環境中使用。
>

## <a name="prerequisites"></a>必要條件
* Azure 檔案服務磁碟區外掛程式的 Windows 版本只能在 [Windows Server 1709 版](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1709)、[Windows 10 1709 版](https://docs.microsoft.com/windows/whats-new/whats-new-windows-10-version-1709)或更新版本的作業系統上運作。 Azure 檔案服務磁碟區外掛程式的 Linux 版本可在 Service Fabric 所支援的所有作業系統版本上運作。

* Azure 檔案磁碟區外掛程式只適用於 Service Fabric 6.2 版及更新版本。

* 請依照 [Azure 檔案服務文件](https://docs.microsoft.com/azure/storage/files/storage-how-to-create-file-share) \(英文\) 中的指示，為 Service Fabric 容器應用程式建立一個檔案共用作為磁碟區使用。

* 您將會需要安裝[包含 Service Fabric 模組的 Powershell](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started) 或 [SFCTL](https://docs.microsoft.com/azure/service-fabric/service-fabric-cli)。

* 如果您使用 hyperv 容器，則必須在 ClusterManifest (本機叢集) 或在 ARM 範本 (Azure 叢集) 或 ClusterConfig.json (獨立叢集) 的 fabricSettings 區段中新增下列程式碼片段。 您需要磁碟區名稱，以及磁碟區在叢集上接聽的連接埠。 

在 ClusterManifest 中，必須在 Hosting 區段中新增下列內容。 在此範例中，磁碟區名稱是 **sfazurefile**，而它在叢集上接聽的連接埠為 **19300**。  

``` xml 
<Section Name="Hosting">
  <Parameter Name="VolumePluginPorts" Value="sfazurefile:19300" />
</Section>
```

在 ARM 範本 (適用於 Azure 部署) 或 ClusterConfig.json (適用於獨立部署) 的 fabricSettings 區段中 中，必須新增下列程式碼片段。 

```json
"fabricSettings": [
  {
    "name": "Hosting",
    "parameters": [
      {
          "name": "VolumePluginPorts",
          "value": "sfazurefile:19300"
      }
    ]
  }
]
```


## <a name="deploy-the-service-fabric-azure-files-application"></a>部署 Service Fabric Azure 檔案服務應用程式

為您的容器提供磁碟區的 Service Fabric 應用程式，可從此[連結](https://aka.ms/sfvolume)下載。 您可以透過 [PowerShell](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications)、[CLI](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-lifecycle-sfctl) 或 [FabricClient API](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications-fabricclient) 將應用程式部署到叢集。

1. 使用命令列將目錄變更至已下載應用程式套件的根目錄。

    ```powershell
    cd .\AzureFilesVolume\
    ```

    ```bash
    cd ~/AzureFilesVolume
    ```

2. 將應用程式套件複製到映像存放區。以適當的 [ApplicationPackagePath] 和 [ImageStoreConnectionString] 值執行下列命令：

    ```powershell
    Copy-ServiceFabricApplicationPackage -ApplicationPackagePath [ApplicationPackagePath] -ImageStoreConnectionString [ImageStoreConnectionString] -ApplicationPackagePathInImageStore AzureFilesVolumePlugin
    ```

    ```bash
    sfctl cluster select --endpoint https://testcluster.westus.cloudapp.azure.com:19080 --pem test.pem --no-verify
    sfctl application upload --path [ApplicationPackagePath] --show-progress
    ```

3. 註冊應用程式類型

    ```powershell
    Register-ServiceFabricApplicationType -ApplicationPathInImageStore AzureFilesVolumePlugin
    ```

    ```bash
    sfctl application provision --application-type-build-path [ApplicationPackagePath]
    ```

4. 在命令中建立應用程式以建立下方的應用程式，請記下 **ListenPort** 應用程式參數。 針對此應用程式參數所指定的值為連接埠，Azure 檔案服務磁碟區外掛程式會透過這個連接埠來接聽來自 Docker 精靈的要求。 請務必確認為應用程式提供的連接埠，沒有和叢集或應用程式所使用的任何其他連接埠衝突。

    ```powershell
    New-ServiceFabricApplication -ApplicationName fabric:/AzureFilesVolumePluginApp -ApplicationTypeName AzureFilesVolumePluginType -ApplicationTypeVersion 6.255.389.9494 -ApplicationParameter @{ListenPort='19100'}
    ```

    ```bash
    sfctl application create --app-name fabric:/AzureFilesVolumePluginApp --app-type AzureFilesVolumePluginType --app-version 6.255.389.9494 --parameter '{"ListenPort":"19100"}'
    ```

> [!NOTE]

> Windows Server 2016 Datacenter 不支援將 SMB 掛接對應至容器 ([該功能僅在 Windows Server 1709 版上受到支援](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/container-storage))。 這條件約束可避免在早於 1709 的版本上使用網路磁碟區對應和 Azure 檔案服務磁碟區驅動程式。
>   

### <a name="deploy-the-application-on-a-local-development-cluster"></a>在本機開發叢集上部署應用程式
Azure 檔案服務磁碟區外掛程式應用程式的預設服務執行個體計數為 -1，這表示針對叢集中的每個節點都有部署一個服務的執行個體。 但是，將 Azure 檔案服務磁碟區外掛程式應用程式部署到本機開發叢集上時，服務執行個體計數應指定為 1。 這可以透過 **InstanceCount** 應用程式參數來完成。 因此，將 Azure 檔案服務磁碟區外掛程式應用程式部署到本機開發叢集的命令為：

```powershell
New-ServiceFabricApplication -ApplicationName fabric:/AzureFilesVolumePluginApp -ApplicationTypeName AzureFilesVolumePluginType -ApplicationTypeVersion 6.255.389.9494 -ApplicationParameter @{ListenPort='19100';InstanceCount='1'}
```

```bash
sfctl application create --app-name fabric:/AzureFilesVolumePluginApp --app-type AzureFilesVolumePluginType --app-version 6.255.389.9494 --parameter '{"ListenPort": "19100","InstanceCount": "1"}'
```
## <a name="configure-your-applications-to-use-the-volume"></a>設定您的應用程式以使用磁碟區
下列程式碼片段會顯示如何在應用程式的應用程式資訊清單中指定以 Azure 檔案服務為基礎的磁碟區。 需要關注的特定元素為 **Volume** 標記：

```xml
?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WinNodeJsApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <Description>Calculator Application</Description>
    <Parameters>
      <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
      <Parameter Name="MyCpuShares" DefaultValue="3"></Parameter>
      <Parameter Name="MyStorageVar" DefaultValue="c:\tmp"></Parameter>
    </Parameters>
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeServicePackage" ServiceManifestVersion="1.0"/>
     <Policies>
       <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="hyperv">
            <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
            <RepositoryCredentials PasswordEncrypted="false" Password="****" AccountName="test"/>
            <Volume Source="azfiles" Destination="c:\VolumeTest\Data" Driver="sfazurefile">
                <DriverOption Name="shareName" Value="" />
                <DriverOption Name="storageAccountName" Value="" />
                <DriverOption Name="storageAccountKey" Value="" />
            </Volume>
       </ContainerHostPolicies>
   </Policies>
    </ServiceManifestImport>
    <ServiceTemplates>
        <StatelessService ServiceTypeName="StatelessNodeService" InstanceCount="5">
            <SingletonPartition></SingletonPartition>
        </StatelessService>
    </ServiceTemplates>
</ApplicationManifest>
```

Azure 檔案服務磁碟區外掛程式的驅動程式名稱為 **sfazurefile**。 這個值是針對應用程式資訊清單中 **Volume** 元素的 **Driver** 屬性所設定。

在上述程式碼片段的 **Volume** 元素中，Azure 檔案服務磁碟區外掛程式需要下列標記：
- **Source**：這是磁碟區的名稱。 使用者可以為他們的磁碟區選擇任何名稱。
- **Destination**：此標記是磁碟區在執行容器內所對應的位置。 因此，您的目的地不能是容器內的現有位置

如上述程式碼片段的 **DriverOption** 元素中所示，Azure 檔案服務磁碟區外掛程式支援下列驅動程式選項：

- **shareName**：為容器提供磁碟區之 Azure 檔案服務檔案共用的名稱
- **storageAccountName**：包含 Azure 檔案服務檔案共用之 Azure 儲存體帳戶的名稱
- **storageAccountKey**：包含 Azure 檔案服務檔案共用之 Azure 儲存體帳戶的存取金鑰

上述所有驅動程式選項皆為必要選項。

## <a name="using-your-own-volume-or-logging-driver"></a>使用您自己的磁碟區或記錄驅動程式
Service Fabric 也允許使用您自己的自訂[磁碟區](https://docs.docker.com/engine/extend/plugins_volume/)或[記錄](https://docs.docker.com/engine/admin/logging/overview/)驅動程式。 如果叢集上未安裝 Docker 磁碟區/記錄驅動程式，可以使用 RDP/SSH 通訊協定來手動安裝它。 您可以透過[虛擬機器擴展集啟動指令碼](https://azure.microsoft.com/resources/templates/201-vmss-custom-script-windows/)或 [SetupEntryPoint 指令碼](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-model#describe-a-service)，使用這些通訊協定執行安裝。

安裝[適用於 Azure 的 Docker 磁碟區驅動程式](https://docs.docker.com/docker-for-azure/persistent-data-volumes/)的指令碼範例如下所示：

```bash
docker plugin install --alias azure --grant-all-permissions docker4x/cloudstor:17.09.0-ce-azure1  \
    CLOUD_PLATFORM=AZURE \
    AZURE_STORAGE_ACCOUNT="[MY-STORAGE-ACCOUNT-NAME]" \
    AZURE_STORAGE_ACCOUNT_KEY="[MY-STORAGE-ACCOUNT-KEY]" \
    DEBUG=1
```

在您的應用程式中，若要使用自己安裝的磁碟區或記錄驅動程式，您必須在應用程式資訊清單 **ContainerHostPolicies** 底下的 **Volume** 和 **LogConfig** 元素中指定適當的值。

```xml
<ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="hyperv">
    <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
    <RepositoryCredentials PasswordEncrypted="false" Password="****" AccountName="test"/>
    <LogConfig Driver="[YOUR_LOG_DRIVER]" >
        <DriverOption Name="test" Value="vale"/>
    </LogConfig>
    <Volume Source="c:\workspace" Destination="c:\testmountlocation1" IsReadOnly="false"></Volume>
    <Volume Source="[MyStorageVar]" Destination="c:\testmountlocation2" IsReadOnly="true"> </Volume>
    <Volume Source="myvolume1" Destination="c:\testmountlocation2" Driver="[YOUR_VOLUME_DRIVER]" IsReadOnly="true">
        <DriverOption Name="[name]" Value="[value]"/>
    </Volume>
</ContainerHostPolicies>
```

指定磁碟區外掛程式時，Service Fabric 會使用指定的參數自動建立磁碟區。 **[磁碟區]** 元素的 **[來源]** 標記是磁碟區的名稱，而 **[驅動程式]** 標記會指定磁碟區驅動程式的外掛程式。 [目的地] 標記是 [來源] 在執行容器內所對應的位置。 因此，您的目的地不能是您容器內的現有位置。 您可以使用 [DriverOption] 標記來指定選項，如下所示：

```xml
<Volume Source="myvolume1" Destination="c:\testmountlocation4" Driver="azure" IsReadOnly="true">
           <DriverOption Name="share" Value="models"/>
</Volume>
```

先前資訊清單程式碼片段中所示的磁碟區支援應用程式參數 (尋找 `MyStorageVar` 以取得使用範例)。

如果指定了 Docker 記錄驅動程式，您就必須在叢集中部署代理程式 (或容器) 來處理記錄。 [DriverOption] 標記可以用來指定記錄驅動程式的選項。

## <a name="next-steps"></a>後續步驟
* 若要查看容器範例 (包括磁碟區驅動程式)，請瀏覽 [Service Fabric 容器範例](https://github.com/Azure-Samples/service-fabric-containers) \(英文\)
* 若要將容器部署到 Service Fabric 叢集，請參閱[在 Service Fabric 上部署容器](service-fabric-deploy-container.md)一文
