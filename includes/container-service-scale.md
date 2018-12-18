在[部署 Azure Container Service 叢集](../articles/container-service/dcos-swarm/container-service-deployment.md)之後，您可能需要變更代理程式節點的數目。 例如，您可能需要更多代理程式，以便可以執行更多的容器應用程式或執行個體。 

您可以使用 Azure 入口網站或 Azure CLI 來變更 DC/OS、Docker Swarm 或 Kubernetes 叢集中的代理程式節點數目。 

## <a name="scale-with-the-azure-portal"></a>使用 Azure 入口網站進行調整

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽**容器服務**，然後按一下您想要修改的容器服務。
2. 在 [容器服務] 刀鋒視窗中，按一下 [代理程式]。
3. 在 [VM 計數] 中，輸入所需的代理程式節點數目。

    ![在入口網站中調整集區](./media/container-service-scale/container-service-scale-portal.png)

4. 若要儲存組態時，請按一下 [儲存]。

## <a name="scale-with-the-azure-cli"></a>使用 Azure CLI 進行調整

請確定您[已安裝](/cli/azure/install-az-cli2)最新的 Azure CLI 並登入 Azure 帳戶 (`az login`)。

### <a name="see-the-current-agent-count"></a>查看目前的代理程式計數
若要查看叢集中目前擁有的代理程式數目，請執行 `az acs show` 命令。 這會顯示叢集組態。 例如，下列命令會顯示資源群組 `myResourceGroup` 中名為 `containerservice-myACSName` 之容器服務的組態：

```azurecli
az acs show -g myResourceGroup -n containerservice-myACSName
```

此命令會在 `AgentPoolProfiles` 底下的 `Count` 值中傳回代理程式數目。

### <a name="use-the-az-acs-scale-command"></a>使用 az acs scale 命令
若要變更代理程式節點數目，請執行 `az acs scale` 命令，並提供**資源群組**、**容器服務名稱**及所需的**新代理程式計數**。 藉由使用較少或更高的數字，即可分別相應減少或相應增加。

例如，若要將先前叢集中的代理程式數目變更為 10，請輸入下列命令︰

```azurecli
az acs scale -g myResourceGroup -n containerservice-myACSName --new-agent-count 10
```

Azure CLI 會傳回 JSON 字串，這個字串代表容器服務的新組態，其中包括新的代理程式計數。

如需更多的命令選項，請執行 `az acs scale --help`。

## <a name="scaling-considerations"></a>調整考量

* 代理程式節點的數目必須介於 1 到 100 (含) 之間。 

* 核心配額可以限制叢集中的代理程式節點數目。

* 代理程式節點調整作業會套用至包含代理程式集區的 Azure 虛擬機器擴展集。 在 DC/OS 叢集中，只有私人集區中的代理程式節點會受到本文所示作業的調整。

* 您可以根據您在叢集中部署的 Orchestrator，個別地調整在叢集上執行之容器的執行個體數目。 例如，在 DC/OS 叢集中，使用 [Marathon UI](../articles/container-service/dcos-swarm/container-service-mesos-marathon-ui.md) 來變更容器應用程式的執行個體數目。


## <a name="next-steps"></a>後續步驟
* 請參閱[更多範例](../articles/container-service/dcos-swarm/container-service-create-acs-cluster-cli.md)，以了解如何搭配使用 Azure CLI 命令與 Azure Container Service。
* 深入了解 Azure Container Service 中的 [DC/OS 代理程式集區](../articles/container-service/dcos-swarm/container-service-dcos-agents.md)。

