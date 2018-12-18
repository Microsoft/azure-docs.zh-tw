---
title: Azure Service Fabric DNS 服務 | Microsoft Docs
description: 使用 Service Fabric 的 DNS 服務來從叢集內部探索微服務。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: vturecek
ms.assetid: 47f5c1c1-8fc8-4b80-a081-bc308f3655d3
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 7/20/2018
ms.author: msfussell
ms.openlocfilehash: 70ea33f2c3e3a79d1754b72d12e8fc27689e38ea
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387933"
---
# <a name="dns-service-in-azure-service-fabric"></a>Azure Service Fabric 中的 DNS 服務
「DNS 服務」是一個選用的系統服務，您可以在叢集中啟用以使用 DNS 通訊協定來探索其他服務。 

許多服務 (特別是容器化服務) 都可透過既有的 URL 來定址。 使用標準 DNS 通訊協定來解析這些服務，而不使用 Service Fabric 命名服務通訊協定，是有其效益的。 DNS 服務可讓您將 DNS 服務對應到某個服務名稱，再由此解析端點 IP 位址。 這類功能可保有容器化服務在不同平台間的可攜性，並簡化「隨即轉移」案例，因為您可以使用現有的服務 URL，而不需要重寫程式碼以使用命名服務。 

DNS 服務會將 DNS 名稱對應到服務名稱，接著服務名稱會由命名服務解析並傳回服務端點。 服務的 DNS 名稱是在建立時提供的。 下圖顯示 DNS 服務運用於無狀態服務的情形。

![服務端點](./media/service-fabric-dnsservice/stateless-dns.png)

從 Service Fabric 6.3 版開始，Service Fabric DNS 通訊協定已進行擴充，而包含為分割的具狀態服務定址的配置。 這些擴充功能可讓您使用具狀態服務 DNS 名稱和分割區名稱的組合，來解析特定分割區的 IP 位址。 三種分割配置均受到支援：

- 具名分割
- 定界分割
- 單一分割

下圖顯示 DNS 服務運用於分割具狀態服務的情形。

![具狀態服務端點](./media/service-fabric-dnsservice/stateful-dns.png)

DNS 服務不支援動態連接埠。 若要解決動態連接埠上所公開的服務，請使用[反向 Proxy 服務](./service-fabric-reverseproxy.md)。

## <a name="enabling-the-dns-service"></a>啟用 DNS 服務
當您使用入口網站建立叢集時，預設會在 [叢集組態] 功能表的 [包括 DNS 服務] 核取方塊中啟用 DNS 服務：

![透過入口網站啟用 DNS 服務](./media/service-fabric-dnsservice/enable-dns-service.png)

如果您不使用入口網站來建立叢集，或是要更新現有叢集，則需要在範本中啟用 DNS 服務：

- 若要部署新叢集，您可以使用[範例範本](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype)或建立自己的 Resource Manager 範本。 
- 若要更新現有叢集，您可以在入口網站上瀏覽至叢集的資源群組，然後按一下 [自動化指令碼] 以使用可反映叢集目前狀態和群組中其他資源的範本。 若要深入了解，請參閱[從資源群組匯出範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-export-template#export-the-template-from-resource-group)。

擁有範本後，您可以使用下列步驟來啟用 DNS 服務：

1. 檢查 `Microsoft.ServiceFabric/clusters` 資源的 `apiversion` 是否已設定為 `2017-07-01-preview` 或更新版本，如果不是請加以更新，如下列範例所示：

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. 現在，請以下列其中一種方式啟用 DNS 服務：

   - 若要使用預設設定啟用 DNS 服務，將其新增至 `properties` 區段內的 `addonFeatures`區段，如下列範例所示：

       ```json
           "properties": {
              ...

              "addonFeatures": [
                "DnsService"
              ],
              ...
           }
       ```
   - 若要以預設值以外的設定啟用服務，請將 `DnsService` 區段新增至 `properties` 區段內的 `fabricSettings` 區段。 在此案例中，您不需要將 DnsService 新增至 `addonFeatures`。 若要深入了解可為 DNS 服務設定的屬性，請參閱 [DNS 服務設定](./service-fabric-cluster-fabric-settings.md#dnsservice)。

       ```json
           "properties": {
             ...  
             "fabricSettings": [
               ...
               {
                 "name": "DnsService",
                 "parameters": [
                   {
                     "name": "IsEnabled",
                     "value": "true"
                   },
                   {
                     "name": "PartitionSuffix",
                     "value": "--"
                   },
                   {
                     "name": "PartitionPrefix",
                     "value": "--"
                   }
                 ]
               },
               ...
              ]
            }
       ```
1. 在使用您的變更將叢集範本更新之後，請加以套用，使升級完成。 升級完成後，DNS 系統服務就會開始在叢集中執行。 服務名稱是 `fabric:/System/DnsService`，而且您可以在 Service Fabric Explorer 的 [系統] 服務區段下找到它。 


## <a name="setting-the-dns-name-for-your-service"></a>為您的服務設定 DNS 名稱
您可以在 ApplicationManifest.xml 檔案中，透過預設服務的宣告為您的服務設定 DNS 名稱，或透過 PowerShell 命令來設定。

服務的 DNS 名稱在整個叢集中都是可解析的，因此請務必確保 DNS 名稱在叢集中的唯一性。 

強烈建議您使用 `<ServiceDnsName>.<AppInstanceName>` 的命名配置；例如，`service1.application1`。 如果使用 Docker Compose 來部署應用程式，系統會自動使用此命名配置為服務指派 DNS 名稱。

### <a name="setting-the-dns-name-for-a-default-service-in-the-applicationmanifestxml"></a>在 ApplicationManifest.xml 中為預設服務設定 DNS 名稱
在 Visual Studio 或慣用的編輯器中開啟您的專案，然後開啟 ApplicationManifest.xml 檔案。 移至預設服務區段，然後為每個服務新增 `ServiceDnsName` 屬性。 下列範例示範如何將服務的 DNS 名稱設定為 `service1.application1`

```xml
    <Service Name="Stateless1" ServiceDnsName="service1.application1">
      <StatelessService ServiceTypeName="Stateless1Type" InstanceCount="[Stateless1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
```
部署完應用程式之後，在 Service Fabric Explorer 中的服務執行個體會顯示此執行個體的 DNS 名稱，如下圖所示： 

![服務端點](./media/service-fabric-dnsservice/service-fabric-explorer-dns.png)

下列範例會將具狀態服務的 DNS 名稱設定為 `statefulsvc.app`。 服務會使用具名的資料配置。 請注意，分割區名稱採用小寫。 這是將在 DNS 查詢中作為目標的分割區所需符合的要求；如需詳細資訊，請參閱[對具狀態服務分割區進行 DNS 查詢](https://docs.microsoft.com/azure/service-fabric/service-fabric-dnsservice#preview-making-dns-queries-on-a-stateful-service-partition)。

```xml
    <Service Name="Stateful1" ServiceDnsName="statefulsvc.app" />
      <StatefulService ServiceTypeName="ProcessingType" TargetReplicaSetSize="2" MinReplicaSetSize="2">
        <NamedPartition>
         <Partition Name="partition1" />
         <Partition Name="partition2" />
        </NamedPartition>
      </StatefulService>
    </Service>
```

### <a name="setting-the-dns-name-for-a-service-using-powershell"></a>使用 Powershell 為服務設定 DNS 名稱
您可以在建立服務時，使用 `New-ServiceFabricService` Powershell 命令為服務設定 DNS 名稱。 下列範例會建立 DNS 名稱為 `service1.application1` 的新無狀態服務

```powershell
    New-ServiceFabricService `
    -Stateless `
    -PartitionSchemeSingleton `
    -ApplicationName `fabric:/application1 `
    -ServiceName fabric:/application1/service1 `
    -ServiceTypeName Service1Type `
    -InstanceCount 1 `
    -ServiceDnsName service1.application1
```

## <a name="preview-making-dns-queries-on-a-stateful-service-partition"></a>[預覽] 對具狀態服務分割區進行 DNS 查詢
從 Service Fabric 6.3 版開始，Service Fabric DNS 服務已可支援服務分割區的查詢。

將在 DNS 查詢中使用的分割區會受到下列命名限制：

   - 分割區名稱應與 DNS 相容。
   - 不應使用多標籤分割區名稱 (在名稱中包含點 '.')。
   - 分割區名稱應採用小寫。

以分割區為目標的 DNS 查詢會採用下列格式：

```
    <First-Label-Of-Partitioned-Service-DNSName><PartitionPrefix><Target-Partition-Name>< PartitionSuffix>.<Remaining- Partitioned-Service-DNSName>
```
其中：

- *First-Label-Of-Partitioned-Service-DNSName* 是服務 DNS 名稱的第一個部分。
- *PartitionPrefix* 是可在叢集資訊清單的 DnsService 區段中設定的值，或可透過叢集的 Resource Manager 範本設定的值。 預設值為 "-"。 若要深入了解，請參閱 [DNS 服務設定](./service-fabric-cluster-fabric-settings.md#dnsservice)。
- *Target-Partition-Name* 是分割區的名稱。 
- *PartitionSuffix* 是可在叢集資訊清單的 DnsService 區段中設定的值，或可透過叢集的 Resource Manager 範本設定的值。 預設值是空字串。 若要深入了解，請參閱 [DNS 服務設定](./service-fabric-cluster-fabric-settings.md#dnsservice)。
- *Remaining-Partitioned-Service-DNSName* 是服務 DNS 名稱的其餘部分。

下列範例顯示對於在叢集上執行、`PartitionPrefix` 和 `PartitionSuffix` 採用預設設定的分割服務所做的 DNS 查詢： 

- 若要對 DNS 名稱為 `backendrangedschemesvc.application`、使用定界分割配置的服務解析其分割區 "0"，請使用 `backendrangedschemesvc-0.application`。
- 若要對 DNS 名稱為 `backendnamedschemesvc.application`、使用具名分割配置的服務解析其分割區 “first”，請使用 `backendnamedschemesvc-first.application`。

DNS 服務會傳回分割區主要複本的 IP 位址。 若未指定分割區，則服務會傳回隨機選取分割區的主要複本 IP 位址。

## <a name="using-dns-in-your-services"></a>在您的服務中使用 DNS
如果您部署多個服務，您可以使用 DNS 名稱來尋找作為通訊對象的其他服務端點。 DNS 服務適用於無狀態服務，且在 Service Fabric 6.3 版和更新版本中適用於具狀態服務。 針對在 Service Fabric 6.3 之前的版本上執行的具狀態服務，您可以使用 HTTP 呼叫的內建[反向 Proxy 服務](./service-fabric-reverseproxy.md)，來呼叫特定的服務分割區。 

DNS 服務不支援動態連接埠。 您可以使用反向 Proxy 服務來解析使用動態連接埠的服務。

下列程式碼說明如何透過 DNS 呼叫無狀態服務。 這只是一般的 HTTP 呼叫，其中您會提供 DNS 名稱、連接埠和任何選用路徑，作為 URL 的一部分。

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

下列程式碼顯示對具狀態服務的特定分割區所發出的呼叫。 在此案例中，DNS 名稱包含分割區名稱 (partition1)。 此呼叫採用 `PartitionPrefix` 和 `PartitionSuffix` 皆為預設值的叢集。

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service2-partition1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

## <a name="known-issues"></a>已知問題
* 針對 Service Fabric versions 6.3 與更新版本，針對 DNS 名稱中包含連字號之服務名稱進行 DNS 查閱會發生問題。 如需有關此問題的詳細資訊，請追蹤下列 [GitHub 問題](https://github.com/Azure/service-fabric-issues/issues/1197) \(英文\)。 接下來的 6.3 更新中將包含此問題的修正程式。 

## <a name="next-steps"></a>後續步驟
若要深入了解叢集內的服務通訊，請參閱[連接服務並與其進行通訊](service-fabric-connect-and-communicate-with-services.md)

