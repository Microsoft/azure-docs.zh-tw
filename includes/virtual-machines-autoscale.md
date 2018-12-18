當您使用[虛擬機器擴展集](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)和 [Azure 監視器的自動調整功能](../articles/monitoring-and-diagnostics/monitoring-overview-autoscale.md)時，可以輕鬆地[自動縮放](../articles/monitoring-and-diagnostics/insights-autoscale-best-practices.md)您的[虛擬機器 (VM)](../articles/virtual-machines/windows/overview.md)。 您的 VM 必須是擴展集的成員才能自動縮放。 本文提供的資訊可讓您深入了解如何使用自動和手動方法，以垂直和水平方式調整 VM。

## <a name="horizontal-or-vertical-scaling"></a>水平和垂直調整

Azure 監視器的自動調整功能只會水平調整，亦即增加 (相應放大) 或減少 (相應縮小) VM 數目。 在雲端的情況下，水平調整會更有彈性，因為它可讓您有機會執行數千台 VM 來處理負載。 您可以自動或手動變更擴展集的容量 (或執行個體計數) 來進行水平調整。 

垂直調整會保持相同數量的 VM，但會讓 VM 的能力變強 (相應增加) 或變弱 (相應減少)。 會以諸如記憶體、CPU 速度或磁碟空間等屬性來測量能力。 垂直調整取決於是否有較大容量的硬體可供使用，因此會很快達到上限，而且會因區域而異。 此外，垂直調整通常會需要 VM 停止並重新啟動。 您進行垂直調整的方法為在擴展集中設定 VM 設定的新大小。

您可以在 [Azure 自動化](../articles/automation/automation-intro.md)中使用 Runbook，輕鬆地[將擴展集中的 VM](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-vertical-scale-reprovision.md) 相應增加或相應減少。

## <a name="create-a-virtual-machine-scale-set"></a>建立虛擬機器擴展集

擴展集可讓您將完全相同的 VM 以集合的方式進行部署和管理。 您可以建立 Linux 或 Windows 擴展集，方法為使用 [Azure 入口網站](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-portal-create.md)、[Azure PowerShell](../articles/virtual-machines/windows/tutorial-create-vmss.md) 或 [Azure CLI](../articles/virtual-machines/linux/tutorial-create-vmss.md)。 您也可以使用 SDK (例如 [Python](https://azure.microsoft.com/develop/python/) 或 [Node.js](/nodejs/azure))，或直接使用 [REST API](/rest/api/compute/virtualmachinescalesets) 來建立及管理規模集。 會透過將計量和規則套用至擴展集來完成自動調整 VM。

## <a name="configure-autoscale-for-a-scale-set"></a>設定自動調整擴展集

自動調整會提供正確數目的 VM 來處理您應用程式上的負載。 它可讓您新增 VM 來處理增加的負載，並可藉由移除閒置 VM 來節省成本。 您要以一組規則作為基礎來指定 VM 的最小和最大數目。 設定下限可確保即使沒有負載應用程式也會一直執行。 設定上限值可限制每小時可能產生的總成本。

當您使用 [Azure PowerShell](../articles/monitoring-and-diagnostics/insights-powershell-samples.md#create-and-manage-autoscale-settings) 或 [Azure CLI](https://docs.microsoft.com/cli/azure/monitor/autoscale-settings) 建立擴展集時，可以啟用自動調整。 您也可以在建立擴展集之後加以啟用。 您可以使用 [Azure Resource Manager 範本](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-windows-autoscale.md)來建立擴展集、安裝擴充功能，並設定自動調整。 在 Azure 入口網站中，從 Azure 監視器啟用自動調整，或從擴展集設定啟用自動調整。

![啟用自動調整](./media/virtual-machines-autoscale/virtual-machines-autoscale-enable.png)
 
### <a name="metrics"></a>度量

Azure 監視器的自動調整功能可讓您以[計量](../articles/monitoring-and-diagnostics/insights-autoscale-common-metrics.md)作為基礎，將執行中的 VM 數目相應增加或相應減少。 根據預設，VM 會提供磁碟、網路和 CPU 使用量的基本主機層級計量。 當您使用診斷擴充功能設定診斷資料的集合時，其他的客體 OS 效能計數器就可供磁碟、CPU 和記憶體使用。

![計量準則](./media/virtual-machines-autoscale/virtual-machines-autoscale-criteria.png)

如果您的應用程式需要以無法透過主機使用的計量作為基礎進行調整，擴展集中的 VM 就必須安裝 [Linux 診斷擴充功能](../articles/virtual-machines/linux/diagnostic-extension.md)或 [Windows 診斷擴充功能](../articles/virtual-machines/windows/ps-extensions-diagnostics.md)。 如果您使用 Azure 入口網站來建立擴展集，也必須透過您所需的診斷設定，使用 Azure PowerShell 或 Azure CLI 來安裝擴充功能。
 
### <a name="rules"></a>規則

[規則](../articles/monitoring-and-diagnostics/monitoring-autoscale-scale-by-custom-metric.md)會結合計量與要執行的動作。 符合規則條件時，就會觸發一個或多個自動調整動作。 例如，如果平均 CPU 使用量超過 85%，您可能會定義規則，將 VM 數目增加為 1 個。

![自動調整動作](./media/virtual-machines-autoscale/virtual-machines-autoscale-actions.png)
 
### <a name="notifications"></a>通知

您可以[設定觸發程序](../articles/monitoring-and-diagnostics/insights-autoscale-to-webhook-email.md)，從而以您所建立的自動調整規則作為基礎，呼叫特定的 web URL 或傳送電子郵件。 Webhook 可讓您將 Azure 警示通知路由到其他系統進行後處理或自訂通知。

## <a name="manually-scale-vms-in-a-scale-set"></a>在擴展集中手動調整 VM 大小

### <a name="horizontal"></a>水平

您可以新增或移除 VM，方法是變更擴展集的容量。 在 Azure 入口網站中，您可以減少或增加擴展集中的 VM 數目 (顯示為**執行個體計數**)，方法是將調整螢幕上的覆寫條件列向左或向右滑動。

使用 Azure PowerShell 時，您必須使用 [Get-AzureRmVmss](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmss) 來取得擴展集物件。 然後，您要將 **sku.capacity** 屬性設定為您需要的 VM 數目，並以 [Update-AzureRmVmss](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvmss) 來更新擴展集。 使用 Azure CLI 時，您要透過 [az vmss scale](https://docs.microsoft.com/cli/azure/vmss#az_vmss_scale) 命令的 **--new-capacity** 參數來變更容量。

### <a name="vertical"></a>Vertical

您可以在擴展集的 [大小] 畫面上，手動變更 Azure 入口網站中的 VM 大小。 您可以使用 Azure PowerShell 搭配 Get-AzureRmVmss，設定映像參考 SKU 屬性，然後使用 [Update-AzureRmVmss](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvmss) 和 [Update-AzureRmVmssInstance](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvmssinstance)。

## <a name="next-steps"></a>後續步驟

- 請在[擴展集的設計考量](../articles/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview.md)中深入了解擴展集。

