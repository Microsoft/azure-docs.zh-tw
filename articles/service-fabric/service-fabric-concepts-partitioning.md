---
title: 分割 Service Fabric 服務 | Microsoft Docs
description: 描述如何分割 Service Fabric 具狀態服務。 資料分割可讓資料儲存在本機電腦上，因此可以一起調整資料和計算。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: 3b7248c8-ea92-4964-85e7-6f1291b5cc7b
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/30/2017
ms.author: msfussell
ms.openlocfilehash: ae7eba9997c4f567eb7b07e23ab42c9ac7740698
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388104"
---
# <a name="partition-service-fabric-reliable-services"></a>分割 Service Fabric 可靠服務
這篇文章介紹分割 Azure Service Fabric 可靠服務的基本概念。 [GitHub](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Services/AlphabetPartitions)上也提供本文中使用的原始碼。

## <a name="partitioning"></a>分割
分割不是 Service Fabric 所獨有。 事實上，它是建置可調整服務的核心模式。 廣義上，我們可以將分割視為將狀態 (資料) 和計算分成較小的可存取單位來改善延展性和效能。 [資料分割][wikipartition]是一種知名的分割形式，也稱為分區化。

### <a name="partition-service-fabric-stateless-services"></a>分割 Service Fabric 無狀態服務
對於無狀態服務，您可以將資料分割視為邏輯單元，其中包含服務的一個或多個執行個體。 圖 1 顯示無狀態服務有 5 個執行個體分散到使用一個資料分割的叢集。

![無狀態服務](./media/service-fabric-concepts-partitioning/statelessinstances.png)

實際上有兩種無狀態服務解決方案。 第一種是將狀態保存在外部的服務，例如 Azure SQL 資料庫 (例如儲存工作階段資訊和資料的網站)。 第二種是僅限計算的服務 (例如計算機或影像縮圖)，不管理任何持續性狀態。

在任一情況下，分割無狀態服務是很少見的情況，通常是藉由新增更多執行個體來達成延展性和可用性。 當您必須符合特殊的路由要求時，才會想要考慮對無狀態服務執行個體使用多個資料分割。

例如，試想有一個情況，其中識別碼在某個範圍內的使用者應只由特定的服務執行個體來服務。 另一個您會分割無狀態服務的例子是當您有真正分割的後端時 (例如分區化 SQL 資料庫)，而且您想要控制哪一個服務執行個體應該寫入資料庫分區，或在需要有後端中使用的相同分割資訊的無狀態服務內執行其他準備工作。 這幾種情況也可以透過不同方式解決，不一定需要服務分割。

本逐步解說的其餘部分著重於具狀態服務。

### <a name="partition-service-fabric-stateful-services"></a>分割 Service Fabric 具狀態服務
Service Fabric 提供一流的方法來分割狀態 (資料)，讓您輕鬆開發可調整的具狀態服務。 在概念上，您可以將具狀態服務視為一種縮放單位，透過叢集中節點之間分散和平衡的 [複本](service-fabric-availability-services.md) ，此縮放單位非常可靠。

在提及 Service Fabric 具狀態服務時，分割過程是指決定特定的服務資料分割負責服務完整狀態的一部分  (如前所述，資料分割是一組[複本](service-fabric-availability-services.md))。 Service Fabric 最棒的一點是將資料分割放在不同節點上。 這可讓它們在節點的資源限制內成長。 資料需求成長時，資料分割也會成長，Service Fabric 會重新平衡節點之間的資料分割。 這可確保持續有效率地使用硬體資源。

舉例來說，假設您一開始叢集有 5 個節點、服務設定為有 10 個資料分割，以及目標為三個複本。 在此情況下，Service Fabric 會將複本平衡並分散到叢集，最後每個節點會有 2 個主要 [複本](service-fabric-availability-services.md) 。
如果您現在需要將我們的叢集相應放大到 10 個節點，Service Fabric 會在所有 10 個節點之間重新平衡主要 [複本](service-fabric-availability-services.md) 。 同樣地，如果調降為 5 個節點，Service Fabric 會在 5 個節點之間重新平衡所有複本。  

圖 2 顯示調整叢集之前和之後 10 個資料分割的分佈。

![具狀態服務](./media/service-fabric-concepts-partitioning/partitions.png)

如此一來，因為來自用戶端要求會分散到各電腦而達成相應放大，應用程式的整體效能獲得改善，也減少競爭存取資料區塊的情況。

## <a name="plan-for-partitioning"></a>規劃分割
實作服務之前，一定要考慮相應放大所需的分割策略。方法不同，但全部都著重於應用程式必須達到的目的。 在這篇文章中，讓我們看一些更重要的層面。

第一步先思考必須分割的狀態結構是個不錯的方法。

我們來看一個簡單的範例。 如果您要為一次全國性選舉建置服務，您可以為該國的每個城市建立資料分割。 然後，您可以將城市中每一個人的投票存放在對應到該城市的資料分割中。 圖 3 說明一組人及其居住城市。

![簡單資料分割](./media/service-fabric-concepts-partitioning/cities.png)

由於城市人口差異極大，最後可能是某些資料分割包含大量資料 (例如西雅圖)，而另外一些資料分割只有極少狀態 (例如柯克蘭)。 狀態數量不平均的資料分割有什麼影響？

如果您回顧一下範例，就會明白保存西雅圖投票的資料分割會比柯克蘭資料分割的流量更多。 根據預設，Service Fabric 會確保每個節點上的主要和次要複本數目大約相同。 所以您最後可能是保存複本的一些節點處理有較多流量，而另外一些節點有較少流量。 您可能想要避免叢集中發生像這樣的冷熱不均。

若要避免這種情況，您應該從分割觀點來做兩件事：

* 試著分割狀態，使它平均分散到所有資料分割。
* 報告此服務每個複本的負載 (如需有關如何進行的資訊，請參閱[度量和負載](service-fabric-cluster-resource-manager-metrics.md)上的這篇文章)。 Service Fabric 能夠報告服務取用的負載，例如記憶體數量或記錄數量。 根據報告的計量，Service Fabric 會偵測到某些資料分割處理的負載高於其他資料分割，然後將複本移至更適合的節點，以重新平衡叢集，使整體沒有節點多載。

有時候您不知道給定的資料分割中有多少資料。 所以一般是建議兩者都做，首先是採用分割策略將資料平均分散到資料分割，其次是報告負載。  第一種方法可避免投票範例中的情況，第二種方法有助於消除一段期間內短暫的存取或負載差異。

分割規劃的另一方面是一開始選擇正確的資料分割數目。
從 Service Fabric 的觀點來看，並不阻止您一開始就使用高於案例預期的資料分割數目。
事實上，假設資料分割的最大數目是有效的方法。

在少數情況下，最後需要的磁碟分割可能比一開始選擇的數目更多。 因為您不能在事後變更資料分割計數，您需要套用一些進階的分割方法，例如建立相同服務類型的新服務執行個體。 您也需要實作一些用戶端邏輯，根據用戶端程式碼必須維護的用戶端知識，將要求路由傳送至正確的服務執行個體

分割規劃的另一項考量是可用的電腦資源。 因為狀態需要存取與儲存，您會受下列限制約束：

* 網路頻寬限制
* 系統記憶體限制
* 磁碟儲存體限制

如果在執行中的叢集遇到資源限制，該怎麼辦？ 答案是您可以輕易地調相應放大群集來滿足新的需求。

[容量規劃指南](service-fabric-capacity-planning.md) 提供如何判斷叢集需要多少節點的指導方針。

## <a name="get-started-with-partitioning"></a>開始進行分割
本節描述如何開始分割您的服務。

Service Fabric 有三個資料分割配置可選擇：

* 範圍分割 (亦稱為 UniformInt64Partition)。
* 具名分割。 採用此模型的應用程式通常是在有界限集合內有可分割為值區的資料。 一些常見做為具名資料分割索引鍵的資料欄位範例包括區域、郵遞區號、客戶群組或其他商務界限。
* 單一分割。 服務不需要任何額外的路由時，通常會使用單一資料分割。 例如，無狀態服務依預設使用此分割配置。

具名和單一分割配置是特殊形式的範圍資料分割。 根據預設，Visual Studio 的 Service Fabric 範本使用範圍分割，因為這是最常見和最有用的配置。 這篇文章的其餘部分著重於範圍分割配置。

### <a name="ranged-partitioning-scheme"></a>範圍分割配置
此功能用來指定整數範圍 (透過低索引鍵和高索引鍵來識別) 和資料分割數目 (n)。 它會建立 n 個資料分割，各負責處理整體資料分割索引鍵範圍的一個非重疊子範圍。 範例：具有低索引鍵 0、高索引鍵 99 及計數 4 的範圍分割配置，將會建立 4 個資料分割，如下所示。

![定界分割](./media/service-fabric-concepts-partitioning/range-partitioning.png)

常見的方法是根據資料集內的唯一索引鍵建立雜湊。 索引鍵的某些常見範例可能是汽車識別號碼 (VIN)、員工識別碼或唯一字串。 接著，您可以使用此唯一索引鍵產生雜湊程式碼，經過索引鍵範圍的模數運算，做為您的索引鍵。 您可以指定可允許索引鍵範圍的上限和下限。

### <a name="select-a-hash-algorithm"></a>選取雜湊演算法
雜湊中的重要部分即為選取雜湊演算法。 無論目標是分組彼此靠近的類似索引鍵 (位置敏感雜湊)，或應該將活動廣泛分散到所有資料分割 (分散雜湊)，需要考量何者較常用。

良好分散雜湊演算法的特性包括容易計算、衝突小且平均分佈索引鍵。 [FNV-1](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function) 雜湊演算法是高效率雜湊演算法的一個很好例子。

一般選擇雜湊程式碼演算法的絕佳資源是 [雜湊函數的維基百科頁面](http://en.wikipedia.org/wiki/Hash_function)。

## <a name="build-a-stateful-service-with-multiple-partitions"></a>建置具有多個資料分割的具狀態服務
讓我們使用多個資料分割建立第一個可靠的具狀態服務。 在本例中，您將建立一個非常簡單的應用程式，其中，您想要將所有以相同字母開頭的姓氏儲存在相同的資料分割中。

撰寫任何程式碼之前，您必須考慮資料分割和資料分割索引鍵。 您需要 26 個資料分割，每個字母英文各一個資料分割，但如何處理低和高索引鍵？
因為我們真的想要每個字母有一個資料分割，我們可以把 0 當做低索引鍵，25 當做高索引鍵，因為每個字母就是它自己的索引鍵。

> [!NOTE]
> 這是簡化的案例，因為現實上分佈會不平均。 "S" 或 "M" 字母開頭的姓氏比 "X" 或 "Y" 開頭的姓氏更常見。
> 
> 

1. 開啟 [Visual Studio] > [檔案] > [新增] > [專案]。
2. 在 [新增專案]  對話方塊中，選擇 Service Fabric 應用程式
3. 將專案命名為 "AlphabetPartitions"。
4. 在 [建立服務] 對話方塊中，選擇 [具狀態] 服務，命名為 "Alphabet.Processing"，如下圖所示。
       ![Visual Studio 中的新增服務對話方塊][1]

  <!--  ![Stateful service screenshot](./media/service-fabric-concepts-partitioning/createstateful.png)-->

5. 設定資料分割數目。 開啟位於 AlphabetPartitions 專案的 ApplicationPackageRoot 資料夾中的 ApplicationManifest.xml 檔案，將參數 Processing_PartitionCount 更新為 26，如下所示。
   
    ```xml
    <Parameter Name="Processing_PartitionCount" DefaultValue="26" />
    ```
   
    您也需要在 ApplicationManifest.xml 中更新 StatefulService 元素的 LowKey 和 HighKey 屬性，如下所示。
   
    ```xml
    <Service Name="Processing">
      <StatefulService ServiceTypeName="ProcessingType" TargetReplicaSetSize="[Processing_TargetReplicaSetSize]" MinReplicaSetSize="[Processing_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[Processing_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    ```
6. 為了能夠存取服務，請在 Alphabet.Processing 服務的 ServiceManifest.xml (位於 PackageRoot 資料夾) 中加入 endpoint 元素，以便在連接埠上開啟端點，如下所示：
   
    ```xml
    <Endpoint Name="ProcessingServiceEndpoint" Port="8089" Protocol="http" Type="Internal" />
    ```
   
    現在，服務已設定為接聽有 26 個資料分割的內部端點。
7. 接下來，您必須覆寫 Processing 類別的 `CreateServiceReplicaListeners()` 方法。
   
   > [!NOTE]
   > 在這個範例中，我們假設您使用簡單的 HttpCommunicationListener。 如需可靠服務通訊的詳細資訊，請參閱 [Reliable Service 通訊模型](service-fabric-reliable-services-communication.md)。
   > 
   > 
8. 關於複本接聽的 URL，建議的模式是下列格式： `{scheme}://{nodeIp}:{port}/{partitionid}/{replicaid}/{guid}`。
    所以您可以設定通訊接聽程式接聽正確的端點並使用此模式。
   
    相同電腦上可能裝載此服務的多個複本，因此複本的此位址必須是唯一的。 這就是為什麼我們在 URL 中有資料分割識別碼 + 複本識別碼。 只要 URL 首碼是唯一的，HttpListener 就可以在相同連接埠上的多個位址接聽。
   
    在進階案例中，次要複本也會接聽唯讀要求，所以有額外 GUID。 在這種情況下，從主要轉換到次要時，您想要確保使用新的唯一位址以強制用戶端重新解析位址。 '+' 在此做為位址，因此複本會接聽所有可用的主機 (IP、FQDM、localhost 等等)。下列程式碼顯示範例。
   
    ```CSharp
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
         return new[] { new ServiceReplicaListener(context => this.CreateInternalListener(context))};
    }
    private ICommunicationListener CreateInternalListener(ServiceContext context)
    {
   
         EndpointResourceDescription internalEndpoint = context.CodePackageActivationContext.GetEndpoint("ProcessingServiceEndpoint");
         string uriPrefix = String.Format(
                "{0}://+:{1}/{2}/{3}-{4}/",
                internalEndpoint.Protocol,
                internalEndpoint.Port,
                context.PartitionId,
                context.ReplicaOrInstanceId,
                Guid.NewGuid());
   
         string nodeIP = FabricRuntime.GetNodeContext().IPAddressOrFQDN;
   
         string uriPublished = uriPrefix.Replace("+", nodeIP);
         return new HttpCommunicationListener(uriPrefix, uriPublished, this.ProcessInternalRequest);
    }
    ```
   
    另外值得注意的是，發佈的 URL 稍微不同於接聽 URL 首碼。
    接聽 URL 提供給 HttpListener。 發佈的 URL 是發佈給 Service Fabric 命名服務 (用於服務探索) 的 URL。 用戶端會透過該探索服務來要求這個位址。 用戶端取得的位址必須有節點的實際 IP 或 FQDN 才能連接。 所以您必須以節點的 IP 或 FQDN 取代 '+'，如上所示。
9. 最後一個步驟是將處理邏輯加入至服務，如下所示。
   
    ```CSharp
    private async Task ProcessInternalRequest(HttpListenerContext context, CancellationToken cancelRequest)
    {
        string output = null;
        string user = context.Request.QueryString["lastname"].ToString();
   
        try
        {
            output = await this.AddUserAsync(user);
        }
        catch (Exception ex)
        {
            output = ex.Message;
        }
   
        using (HttpListenerResponse response = context.Response)
        {
            if (output != null)
            {
                byte[] outBytes = Encoding.UTF8.GetBytes(output);
                response.OutputStream.Write(outBytes, 0, outBytes.Length);
            }
        }
    }
    private async Task<string> AddUserAsync(string user)
    {
        IReliableDictionary<String, String> dictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<String, String>>("dictionary");
   
        using (ITransaction tx = this.StateManager.CreateTransaction())
        {
            bool addResult = await dictionary.TryAddAsync(tx, user.ToUpperInvariant(), user);
   
            await tx.CommitAsync();
   
            return String.Format(
                "User {0} {1}",
                user,
                addResult ? "sucessfully added" : "already exists");
        }
    }
    ```
   
    `ProcessInternalRequest` 讀取用來呼叫資料分割的查詢字串參數的值，並呼叫 `AddUserAsync` 將 lastname 加入可靠的字典 `dictionary`。
10. 讓我們將無狀態服務加入至專案，瞭解如何呼叫特定的資料分割。
    
    這項服務做為簡單的 Web 介面，將接受 lastname 做為查詢字串參數、決定資料分割索引鍵，然後將它傳送給 Alphabet.Processing 服務來處理。
11. 在 [建立服務] 對話方塊中，選擇 [無狀態] 服務，命名為 "Alphabet.Web"，如下圖所示。
    
    ![無狀態服務螢幕擷取畫面](./media/service-fabric-concepts-partitioning/createnewstateless.png)上也提供本文中使用的原始碼。
12. 更新 Alphabet.WebApi 服務的 ServiceManifest.xml 中的端點資訊，以開啟連接埠，如下所示。
    
    ```xml
    <Endpoint Name="WebApiServiceEndpoint" Protocol="http" Port="8081"/>
    ```
13. 您需要傳回 Web 類別中 ServiceInstanceListeners 的集合。 同樣地，您可以選擇實作簡單的 HttpCommunicationListener。
    
    ```CSharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[] {new ServiceInstanceListener(context => this.CreateInputListener(context))};
    }
    private ICommunicationListener CreateInputListener(ServiceContext context)
    {
        // Service instance's URL is the node's IP & desired port
        EndpointResourceDescription inputEndpoint = context.CodePackageActivationContext.GetEndpoint("WebApiServiceEndpoint")
        string uriPrefix = String.Format("{0}://+:{1}/alphabetpartitions/", inputEndpoint.Protocol, inputEndpoint.Port);
        var uriPublished = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
        return new HttpCommunicationListener(uriPrefix, uriPublished, this.ProcessInputRequest);
    }
    ```
14. 現在您需要實作處理邏輯。 有要求傳入時，HttpCommunicationListener 會呼叫 `ProcessInputRequest` 。 讓我們繼續並加入下列程式碼。
    
    ```CSharp
    private async Task ProcessInputRequest(HttpListenerContext context, CancellationToken cancelRequest)
    {
        String output = null;
        try
        {
            string lastname = context.Request.QueryString["lastname"];
            char firstLetterOfLastName = lastname.First();
            ServicePartitionKey partitionKey = new ServicePartitionKey(Char.ToUpper(firstLetterOfLastName) - 'A');
    
            ResolvedServicePartition partition = await this.servicePartitionResolver.ResolveAsync(alphabetServiceUri, partitionKey, cancelRequest);
            ResolvedServiceEndpoint ep = partition.GetEndpoint();
    
            JObject addresses = JObject.Parse(ep.Address);
            string primaryReplicaAddress = (string)addresses["Endpoints"].First();
    
            UriBuilder primaryReplicaUriBuilder = new UriBuilder(primaryReplicaAddress);
            primaryReplicaUriBuilder.Query = "lastname=" + lastname;
    
            string result = await this.httpClient.GetStringAsync(primaryReplicaUriBuilder.Uri);
    
            output = String.Format(
                    "Result: {0}. <p>Partition key: '{1}' generated from the first letter '{2}' of input value '{3}'. <br>Processing service partition ID: {4}. <br>Processing service replica address: {5}",
                    result,
                    partitionKey,
                    firstLetterOfLastName,
                    lastname,
                    partition.Info.Id,
                    primaryReplicaAddress);
        }
        catch (Exception ex) { output = ex.Message; }
    
        using (var response = context.Response)
        {
            if (output != null)
            {
                output = output + "added to Partition: " + primaryReplicaAddress;
                byte[] outBytes = Encoding.UTF8.GetBytes(output);
                response.OutputStream.Write(outBytes, 0, outBytes.Length);
            }
        }
    }
    ```
    
    讓我們帶您逐步了解。 程式碼將查詢字串參數 `lastname` 的第一個字母讀取為字元。 並從姓氏第一個字母的十六進位值中擷取 `A` 的十六進位值，判斷此字母的分割區索引鍵。
    
    ```CSharp
    string lastname = context.Request.QueryString["lastname"];
    char firstLetterOfLastName = lastname.First();
    ServicePartitionKey partitionKey = new ServicePartitionKey(Char.ToUpper(firstLetterOfLastName) - 'A');
    ```
    
    請記得，在此範例中，我們使用 26 個資料分割，每個資料分割有一個資料分割索引鍵。
    接下來，我們使用 `servicePartitionResolver` 物件的 `ResolveAsync` 方法，取得此索引鍵的服務資料分割 `partition`。 `servicePartitionResolver` 定義為
    
    ```CSharp
    private readonly ServicePartitionResolver servicePartitionResolver = ServicePartitionResolver.GetDefault();
    ```
    
    `ResolveAsync` 方法接受服務 URI、資料分割索引鍵和取消語 Token 做為參數。 處理服務的服務 URI 是 `fabric:/AlphabetPartitions/Processing`。 接下來，我們會取得資料分割的端點。
    
    ```CSharp
    ResolvedServiceEndpoint ep = partition.GetEndpoint()
    ```
    
    最後，我們建置端點 URL 加上查詢字串，並呼叫處理服務。
    
    ```CSharp
    JObject addresses = JObject.Parse(ep.Address);
    string primaryReplicaAddress = (string)addresses["Endpoints"].First();
    
    UriBuilder primaryReplicaUriBuilder = new UriBuilder(primaryReplicaAddress);
    primaryReplicaUriBuilder.Query = "lastname=" + lastname;
    
    string result = await this.httpClient.GetStringAsync(primaryReplicaUriBuilder.Uri);
    ```
    
    處理完成後，我們就寫回輸出。
15. 最後一個步驟是測試服務。 Visual Studio 會在本機和雲端部署中使用應用程式參數。 若要在本機測試有 26 個資料分割的服務，您需要更新 AlphabetPartitions 專案的 ApplicationParameters 資料夾中的 `Local.xml` 檔案，如下所示：
    
    ```xml
    <Parameters>
      <Parameter Name="Processing_PartitionCount" Value="26" />
      <Parameter Name="WebApi_InstanceCount" Value="1" />
    </Parameters>
    ```
16. 完成部署之後，您可以在 Service Fabric 總管中檢查服務及其所有資料分割。
    
    ![Service Fabric 總管螢幕擷取畫面](./media/service-fabric-concepts-partitioning/sfxpartitions.png)
17. 您可以在瀏覽器中輸入 `http://localhost:8081/?lastname=somename`來測試分割邏輯。 您會看到以相同字母開頭的每個姓氏儲存在相同的資料分割中。
    
    ![瀏覽器螢幕擷取畫面](./media/service-fabric-concepts-partitioning/samplerunning.png)

範例的完整原始程式碼位於 [GitHub](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Services/AlphabetPartitions)。

## <a name="reliable-services-and-actor-forking-subprocesses"></a>Reliable Services 和 Reliable Actor 分叉處理子程序
Service Fabric 不支援 Reliable Services 和後續 Reliable Actors 分叉處理子程序。 舉例來說，不支援的原因是 [CodePackageActivationContext](https://docs.microsoft.com/dotnet/api/system.fabric.codepackageactivationcontext?view=azure-dotnet) 無法用來註冊不支援的子程序，且取消 Token 只會傳送到已註冊的程序；在父程序收到取消 Token 之後，子程序並未關閉，因而導致各種問題，例如升級失敗。 

## <a name="next-steps"></a>後續步驟
如需 Service Fabric 概念的資訊，請參閱下列項目：

* [Service Fabric 服務的可用性](service-fabric-availability-services.md)
* [Service Fabric 服務的延展性](service-fabric-concepts-scalability.md)
* [Service Fabric 應用程式的容量規劃](service-fabric-capacity-planning.md)

[wikipartition]: https://en.wikipedia.org/wiki/Partition_(database)

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
