# <a name="container-service-frequently-asked-questions"></a>容器服務常見問題集

## <a name="orchestrators"></a>Orchestrator

### <a name="which-container-orchestrators-do-you-support-on-azure-container-service"></a>Azure Container Service 上支援哪些容器 Orchestrator？ 

支援開放原始碼的 DC/OS、Docker Swarm 和 Kubernetes。 如需詳細資訊，請參閱[概觀](../articles/container-service/kubernetes/container-service-intro-kubernetes.md)。
 
### <a name="do-you-support-docker-swarm-mode"></a>是否支援 Docker Swarm 模式？ 

目前不支援 Swarm 模式，但已排入服務的規劃藍圖之中。 

### <a name="does-azure-container-service-support-windows-containers"></a>Azure Container Service 是否支援 Windows 容器？  

目前支援 Linux 容器使用所有 Orchestrator 。 支援 Windows 容器使用 Kubernetes 處於預覽階段。

### <a name="do-you-recommend-a-specific-orchestrator-in-azure-container-service"></a>是否建議在 Azure Container Service 中使用特定 Orchestrator？ 
我們一般不建議使用特定 Orchestrator。 如果您有使用其中一種受支援 Orchestrator 的經驗，則可以在 Azure Container Service 中套用該經驗。 不過，資料趨勢表明，DC/OS 已在生產環境中證明適用於巨量資料和 IoT 工作負載，Kubernetes 適合用於雲端原生的工作負載，而 Docker Swarm 則眾所皆知可與 Docker 工具整合且容易上手。

根據您的情況，您也可以使用其他 Azure 服務建置並管理自訂的容器解決方案。 這些服務包括[虛擬機器](../articles/virtual-machines/linux/overview.md)、[Service Fabric](../articles/service-fabric/service-fabric-overview.md)、[Web Apps](../articles/app-service/app-service-web-overview.md) 和 [Batch](../articles/batch/batch-technical-overview.md)。  

### <a name="what-is-the-difference-between-azure-container-service-and-acs-engine"></a>Azure Container Service 和 ACS 引擎有何不同？ 
Azure Container Service 是具有 SLA 保證的 Azure 服務，其功能可在 Azure 入口網站、Azure 命令列工具和 Azure API 中使用。 此服務可讓您快速實作和管理執行標準容器協調流程工具的叢集，但設定選擇相對較少。 

[ACS 引擎](http://github.com/Azure/acs-engine)是開放原始碼專案，可讓進階使用者自訂各種層級的叢集組態。 因為能改變基礎結構和軟體的組態，所以我們沒有為 ACS 引擎提供 SLA。 其支援的處理是透過 GitHub 上的開放原始碼專案，而非透過 Microsoft 官方管道。 

如需其他詳細資訊，請參閱我們的[容器支援原則](https://support.microsoft.com/en-us/help/4035670/support-policy-for-containers)。

## <a name="cluster-management"></a>叢集管理

### <a name="how-do-i-create-ssh-keys-for-my-cluster"></a>如何為叢集建立 SSH 金鑰？

您可以在作業系統上使用標準工具來建立 SSH RSA 公用和私用金鑰組，以針對叢集的 Linux 虛擬機器進行驗證。 如需相關步驟，請參閱 [OS X 及 Linux](../articles/virtual-machines/linux/mac-create-ssh-keys.md) 或 [Windows](../articles/virtual-machines/linux/ssh-from-windows.md) 指引。 

如果您使用 [Azure CLI 命令](../articles/container-service/dcos-swarm/container-service-create-acs-cluster-cli.md)部署容器服務叢集，系統會自動為叢集產生 SSH 金鑰。

### <a name="how-do-i-create-a-service-principal-for-my-kubernetes-cluster"></a>如何為 Kubernetes 叢集建立服務主體？

另外需要 Azure Active Directory 服務主體識別碼和密碼，才能在 Azure Container Service 中建立 Kubernetes 叢集。 如需詳細資訊，請參閱[關於 Kubernetes 叢集的服務主體](../articles/container-service/kubernetes/container-service-kubernetes-service-principal.md)。

如果您使用 [Azure CLI 命令](../articles/container-service/dcos-swarm/container-service-create-acs-cluster-cli.md)部署 Kubernetes 叢集，系統會自動為叢集產生服務主體認證。

### <a name="how-large-a-cluster-can-i-create"></a>可以建立多大的叢集？
您可以建立含有 1、3 或 5 個主要節點的叢集。 您最多可以選擇 100 個代理程式節點。

> [!IMPORTANT]
> 如需較大的叢集且視您針對節點選擇的 VM 大小而定，您可能需要增加訂用帳戶的核心配額。 若要要求增加配額，可免費[開啟線上客戶支援要求](../articles/azure-supportability/how-to-create-azure-support-request.md)。 如果您使用 [Azure 免費帳戶](https://azure.microsoft.com/free/)，您只能使用有限數目的 Azure 計算核心。
> 

### <a name="how-do-i-increase-the-number-of-masters-after-a-cluster-is-created"></a>如何在建立叢集後增加主要主機的數目？ 
叢集建立之後，主要主機的數目就已固定，且無法變更。 在建立叢集時，您最好選取多個主要節點，以便享有高可用性。

### <a name="how-do-i-increase-the-number-of-agents-after-a-cluster-is-created"></a>如何在建立叢集後增加代理程式的數目？ 
您可以使用 Azure 入口網站或命令列工具來調整叢集中的代理程式數目。 請參閱[調整 Azure Container Service 叢集](../articles/container-service/kubernetes/container-service-scale.md)。

### <a name="what-are-the-urls-of-my-masters-and-agents"></a>主要主機和代理程式的 URL 為何？ 
Azure Container Service 中的叢集資源 URL 是根據您所提供的 DNS 名稱前置詞和您為部署選擇的 Azure 區域名稱來產生。 例如，主要節點的完整網域名稱 (FQDN) 形式如下︰

``` 
DNSnamePrefix.AzureRegion.cloudapp.azure.net
```

您可以在 Azure 入口網站、Azure 資源總管或其他 Azure 工具中找到叢集的常用 URL。

### <a name="how-do-i-tell-which-orchestrator-version-is-running-in-my-cluster"></a>如何分辨在我的叢集中執行哪個 Orchestrator 版本？

* DC/OS︰請參閱 [Mesosphere 文件](https://docs.mesosphere.com/1.7/usage/cli/command-reference/)
* Docker Swarm：請執行 `docker version`
* Kubernetes：請執行 `kubectl version`

### <a name="how-do-i-upgrade-the-orchestrator-after-deployment"></a>如何在部署之後升級 Orchestrator？

目前，Azure Container Service 並未提供工具，可以升級您部署在叢集上的 Orchestrator 版本。 如果 Container Service 支援較新版本，您可以部署新的叢集。 另一個選項是使用 Orchestrator 專屬工具，在適用的地方就地升級叢集。 例如，請參閱 [DC/OS 升級](https://dcos.io/docs/1.8/administration/upgrading/)。
 
### <a name="where-do-i-find-the-ssh-connection-string-to-my-cluster"></a>哪裡可以找到叢集的 SSH 連接字串？

您可以在 Azure 入口網站或使用 Azure 命令列工具找到連接字串。 

1. 在入口網站中，瀏覽至叢集部署的資源群組。  

2. 按一下 [概觀]，然後按一下 [基本資訊] 下 [部署] 的連結。 

3. 在 [部署歷程記錄] 刀鋒視窗中，按一下名稱開頭為 **microsoft-acs** 且後接部署日期的部署。 範例︰microsoft-acs-201701310000。  

4. 在 [摘要] 頁面的 [輸出] 底下，提供了一些叢集連結。 **SSHMaster0** 會提供容器服務叢集中第一個主要主機的 SSH 連接字串。 

如先前所述，您也可以使用 Azure 工具來尋找主要主機的 FQDN。 請使用建立叢集時所指定的主要主機 FQDN 和使用者名稱，透過 SSH 連線到主要主機。 例如︰

```bash
ssh userName@masterFQDN –A –p 22 
```

如需詳細資訊，請參閱[連接到 Azure Container Service 叢集](../articles/container-service/kubernetes/container-service-connect.md)。

### <a name="my-dns-name-resolution-isnt-working-on-windows-what-should-i-do"></a>我的 DNS 名稱解析在 Windows 中沒有作用。 我該怎麼辦？

Windows 上有一些其修正仍在主動地慢慢淘汰的已知 DNS 問題。請確定您所使用的是最新更新的 acs-engine 和 Windows 版本 (已安裝 [KB4074588](https://www.catalog.update.microsoft.com/Search.aspx?q=KB4074588) 和 [KB4089848](https://www.catalog.update.microsoft.com/Search.aspx?q=KB4089848))，以便環境可從此版本獲益。 否則，請參閱下表中的風險降低步驟：

| DNS 徵兆 | 因應措施  |
|-------------|-------------|
|當工作負載容器不穩定而當機時，便會清除網路命名空間 | 重新部署任何受影響的服務 |
| 服務 VIP 存取中斷 | 設定 [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)，使其一律讓一個標準 (非特殊權限的) Pod 保持執行 |
|當容器執行所在的節點變得無法使用，DNS 查詢就可能會失敗，而導致「負數快取項目」 | 在受影響的容器內執行下列命令： <ul><li> `New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord`</li><li>`New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord`</li><li>`Restart-Service dnscache` </li></ul><br> 如果這仍無法解決問題，則請嘗試徹底停用 DNS 快取： <ul><li>`Set-Service dnscache -StartupType disabled`</li><li>`Stop-Service dnscache`</li></ul> |

## <a name="next-steps"></a>後續步驟

* [深入了解](../articles/container-service/kubernetes/container-service-intro-kubernetes.md) Azure Container Service。
* 使用[入口網站](../articles/container-service/dcos-swarm/container-service-deployment.md)或 [Azure CLI ](../articles/container-service/dcos-swarm/container-service-create-acs-cluster-cli.md) 部署容器服務叢集。
