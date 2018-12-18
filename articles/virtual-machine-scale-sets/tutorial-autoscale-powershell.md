---
title: 教學課程 - 使用 Azure PowerShell 自動調整擴展集 |Microsoft Docs
description: 了解如何使用 Azure PowerShell 範本隨著 CPU 需求的增加和減少自動調整虛擬機器擴展集
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 910faad143313ef68ed7ffbc08b8745c73340f23
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38630305"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-azure-powershell"></a>教學課程：使用 Azure PowerShell 自動調整虛擬機器擴展集
當建立擴展集時，您會定義您想要執行的 VM 執行個體數目。 當您的應用程式需求變更時，您可以自動增加或減少 VM 執行個體數目。 自動調整的能力可讓您在整個應用程式的生命週期中，跟上客戶的需求或對應用程式效能變更做出回應。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 使用擴展集的自動調整
> * 建立及使用自動調整規則
> * 對 VM 執行個體進行壓力測試，並觸發自動調整規則
> * 在需求降低時重新自動相應縮小

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。


## <a name="create-a-scale-set"></a>建立擴展集
若要更輕鬆地建立自動調整規則，請定義您擴展集的一些變數。 下列範例會在 myResourceGroup 資源群組和「美國東部」區域中定義 *myScaleSet* 擴展集的變數。 您的訂用帳戶識別碼是使用 [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription) 所取得。 如果您有多個與帳戶建立關聯的訂用帳戶，則只會傳回第一個訂用帳戶。 請調整名稱和訂用帳戶識別碼，如下所示：

```azurepowershell-interactive
$mySubscriptionId = (Get-AzureRmSubscription)[0].Id
$myResourceGroup = "myResourceGroup"
$myScaleSet = "myScaleSet"
$myLocation = "East US"
```

現在使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) 建立虛擬機器擴展集。 為了將流量散發到個別的虛擬機器執行個體，也會建立負載平衡器。 負載平衡器包含在 TCP 連接埠 80 上分配流量的規則，同時允許 TCP 連接埠 3389 上的遠端桌面流量以及 TCP 連接埠 5985 上的 PowerShell 遠端處理。 出現提示時，請為擴展集中的 VM 執行個體提供適當的系統管理認證：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName $myResourceGroup `
  -VMScaleSetName $myScaleSet `
  -Location $myLocation `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer"
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。

## <a name="create-a-rule-to-autoscale-out"></a>建立自動相應放大的規則
如果您的應用程式需求增加，您擴展集內 VM 執行個體上的負載也會跟著增加。 如果這樣的負載增加會持續而非只是短暫的需求，您就可以設定自動調整規則來增加擴展集中的 VM 執行個體數目。 建立這些 VM 執行個體並部署應用程式後，擴展集就會開始透過負載平衡器將流量分散給它們。 您可以控制要監視哪些計量 (例如 CPU 或磁碟)、應用程式負載必須符合給定的閾值多久，以及要將多少個 VM 執行個體新增至擴展集。

使用 [New-AzureRmAutoscaleRule](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleRule) 建立規則，以在平均 CPU 負載在 5 分鐘期間內大於 70% 時，增加擴展集中的 VM 執行個體數目。 當此規則觸發時，VM 執行個體數目會增加三個。

下列參數用於此規則：

| 參數               | 說明                                                                                                         | 值          |
|-------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
| *-MetricName*           | 要監視並套用擴展集動作的效能計量。                                                   | Percentage CPU |
| *-TimeGrain*            | 收集計量進行分析的頻率。                                                                   | 1 分鐘       |
| *-MetricStatistic*      | 定義應該如何彙總收集的計量以進行分析。                                                | 平均值        |
| *-TimeWindow*           | 在比較計量與閾值之前監視的時間長短。                                   | 5 分鐘      |
| *-Operator*             | 用以比較計量資料與閾值之間差異的運算子。                                                     | 大於   |
| *-Threshold*            | 讓自動調整規則觸發動作的值。                                                      | 70%            |
| *-ScaleActionDirection* | 定義擴展集在套用規則時該相應增加還是相應減少。                                             | 增加       |
| *-ScaleActionScaleType* | 指出 VM 執行個體數目應依特定值變更。                                    | 變更計數   |
| *-ScaleActionValue*     | 觸發此規則時，應該變更多少 VM 執行個體百分比。                                            | 3              |
| *-ScaleActionCooldown*  | 再次套用規則前需要等待多久時間，以便讓自動調整動作生效。 | 5 分鐘      |

下列範例建立保存此相應增加規則的 *myRuleScaleOut* 物件。 *-MetricResourceId* 使用先前針對訂用帳戶識別碼、資源群組名稱和擴展集名稱所定義的變數：

```azurepowershell-interactive
$myRuleScaleOut = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -TimeGrain 00:01:00 `
  -MetricStatistic "Average" `
  -TimeWindow 00:05:00 `
  -Operator "GreaterThan" `
  -Threshold 70 `
  -ScaleActionDirection "Increase" `
  –ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 3 `
  -ScaleActionCooldown 00:05:00
```


## <a name="create-a-rule-to-autoscale-in"></a>建立自動相應縮小的規則
在夜晚或週末，您的應用程式需求可能會減少。 若這樣的負載減少會持續一段時間，您就可以設定自動調整規則來減少擴展集中的 VM 執行個體數目。 這個相應縮小的動作可減少執行擴展集的成本，因為只會執行符合目前需求所需的執行個體數目。

使用 [New-AzureRmAutoscaleRule](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleRule) 建立另一個規則，以在平均 CPU 負載於 5 分鐘期間內低於 30% 時，減少擴展集中的 VM 執行個體數目。 觸發規則時，VM 執行個體數目會減少一個。下列範例會建立保存此相應增加規則的 *myRuleScaleDown* 物件。 *-MetricResourceId* 使用先前針對訂用帳戶識別碼、資源群組名稱和擴展集名稱所定義的變數：

```azurepowershell-interactive
$myRuleScaleIn = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator "LessThan" `
  -MetricStatistic "Average" `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection "Decrease" `
  –ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 1
```


## <a name="define-an-autoscale-profile"></a>定義自動調整設定檔
若要將自動調整規則與擴展集建立關聯，請建立設定檔。 自動調整設定檔會定義預設、最小和最大擴展集容量，並建立自動調整規則的關聯。 使用 [New-AzureRmAutoscaleProfile](/powershell/module/AzureRM.Insights/New-AzureRmAutoscaleProfile) 建立自動調整設定檔。 下列範例會設定 *2* 個 VM 執行個體的預設和最小容量，以及最多 *10* 個。 接著附加前面步驟中所建立的相應放大和相應縮小規則：

```azurepowershell-interactive
$myScaleProfile = New-AzureRmAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $myRuleScaleOut,$myRuleScaleIn `
  -Name "autoprofile"
```


## <a name="apply-autoscale-rules-to-a-scale-set"></a>將自動調整規則套用至擴展集
最後一個步驟是將自動調整設定檔套用至擴展集。 您的擴展集接著將可根據應用程式需求自動相應縮小或相應放大。 使用 [Add-AzureRmAutoscaleSetting](/powershell/module/AzureRM.Insights/Add-AzureRmAutoscaleSetting) 套用自動調整設定檔，如下所示：

```azurepowershell-interactive
Add-AzureRmAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -AutoscaleProfile $myScaleProfile
```


## <a name="generate-cpu-load-on-scale-set"></a>在擴展集上產生 CPU 負載
若要測試自動調整規則，請擴展集中的 VM 執行個體上產生 CPU 負載。 這種模擬的 CPU 負載會使自動調整規則相應放大，並增加 VM 執行個體數目。 當模擬的 CPU 負載隨後減輕時，自動調整規則即會相應縮小，並減少 VM 執行個體數目。

若要列出要連線至擴展集中之 VM 執行個體的 NAT 連接埠，請先透過 [Get-AzureRmLoadBalancer](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancer) 取得負載平衡器物件。 接著，使用 [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) 檢視輸入 NAT 規則：

```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzureRmLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzureRmLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

下列範例輸出顯示執行個體名稱、負載平衡器的公用 IP 位址，以及 NAT 規則將流量轉送到的連接埠號碼：

```powershell
Name        Protocol FrontendPort BackendPort
----        -------- ------------ -----------
myRDPRule.0 Tcp             50001        3389
myRDPRule.1 Tcp             50002        3389
```

規則的*名稱*會與 VM 執行個體的名稱相對應，如先前的 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm) 命令所示。 例如，若要連線至 VM 執行個體 *0*，您會使用 *myRDPRule.0* 並連線至連接埠 *50001*。 若要連線至 VM 執行個體 *1*，請使用 *myRDPRule.1* 中的值並連線至連接埠 *50002*。

使用 [Get-AzureRmPublicIpAddress](/powershell/module/AzureRM.Network/Get-AzureRmPublicIpAddress) 檢視負載平衡器的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIP | Select IpAddress
```

範例輸出︰

```powershell
IpAddress
---------
52.168.121.216
```

建立您第一個 VM 執行個體的遠端連線。 請為必要的 VM 執行個體指定您自己的公用 IP 位址和連接埠號碼，如先前的命令所示。 出現提示時，請輸入您在建立擴展集時所使用的認證 (在範例命令中預設為 *azureuser* 和 *P@ssw0rd!*)。 如果您使用 Azure Cloud Shell，請從本機 PowerShell 提示字元或遠端桌面用戶端執行此步驟。 下列範例會連線至 VM 執行個體 *0*：

```powershell
mstsc /v 52.168.121.216:50001
```

登入之後，請從工作列開啟 Internet Explorer。

- 選取 [確定]，以接受 [使用建議的安全性、隱私權與相容性設定] 的提示
- 在網址列中輸入 *http://download.sysinternals.com/files/CPUSTRES.zip*。
- 在啟用 [Internet Explorer 增強式安全性設定] 的情況下，選擇將 *http://download.sysinternals.com* 網域**新增**至信任網站清單。
- 在出現檔案下載的提示時，選取 [開啟]，然後選取並**執行** *CPUSTRES.EXE* 工具。

若要產生一些 CPU 負載，請勾選**作用中**執行緒的兩個方塊。 從這兩個執行緒的 [活動] 下拉式功能表中，選取 [最大值]。 您可以開啟工作管理員，確認 VM 的 CPU 負載是 100%。

![CPU Stress 公用程式會在 VM 執行個體上產生負載](media/tutorial-autoscale-powershell/cpu-stress-load.PNG)

將遠端桌面連線工作階段保持開啟，並開啟另一個遠端桌面連線工作階段。 使用先前的 [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) Cmdlet 中所列的連接埠號碼，連線至第二個 VM 執行個體：

```powershell
mstsc /v 52.168.121.216:50002
```

登入第二個 VM 執行個體後，請重複前述步驟以下載並執行 *CPUSTRES.EXE*。 同樣地，啟動兩個**作用中**執行緒，並將活動設為 [最大值]。

若要讓 **CPU Stress** 工具繼續執行，請將兩個遠端桌面連線工作階段保持開啟。


## <a name="monitor-the-active-autoscale-rules"></a>監視作用中的自動調整規則
若要監視擴展集中的 VM 執行個體數目，請使用 **while**。 約需要 5 分鐘的時間，自動調整規則才會開始執行相應放大程序，以回應 **CPUStress* 對每個 VM 執行個體產生的 CPU 負載：

```azurepowershell-interactive
while (1) {Get-AzureRmVmssVM `
    -ResourceGroupName $myResourceGroup `
    -VMScaleSetName $myScaleSet; sleep 10}
```

達到 CPU 閾值時，自動調整規則就會增加擴展集中的 VM 執行個體數目。 下列輸出顯示擴展集自動相應放大時所建立的三個 VM：

```powershell
ResourceGroupName         Name Location          Sku Capacity InstanceID ProvisioningState
-----------------         ---- --------          --- -------- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_2   eastus Standard_DS2                   2         Succeeded
MYRESOURCEGROUP   myScaleSet_3   eastus Standard_DS2                   3         Succeeded
MYRESOURCEGROUP   myScaleSet_4   eastus Standard_DS2                   4          Creating
MYRESOURCEGROUP   myScaleSet_5   eastus Standard_DS2                   5          Creating
MYRESOURCEGROUP   myScaleSet_6   eastus Standard_DS2                   6          Creating
```

與每個 VM 執行個體的遠端桌面連線工作階段中，關閉 **CPU Stress** 工具。 擴展集中的平均 CPU 負載會恢復正常。 再經過 5 分鐘之後，自動調整規則會相應縮小 VM 執行個體數目。 相應縮小動作會先移除具有最高識別碼的 VM 執行個體。 當擴展集使用可用性設定組或可用性區域時，相應縮小動作會平均散發到這些 VM 執行個體上。 下列範例輸出顯示擴展集自動相應縮小時所刪除的一個 VM 執行個體：

```powershell
MYRESOURCEGROUP   myScaleSet_6   eastus Standard_DS2                   6          Deleting
```

使用 `Ctrl-c` 結束 *while*。 擴展集會繼續每隔 5 分鐘相應縮小一次，並移除一個 VM 執行個體，直到達到最小執行個體計數 (兩個) 為止。


## <a name="clean-up-resources"></a>清除資源
若要移除您的擴展集與其他資源，請使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 刪除資源群組及其所有資源。 `-Force` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。 `-AsJob` 參數不會等待作業完成，就會將控制項傳回給提示字元。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Azure PowerShell 自動相應縮小或放大擴展集：

> [!div class="checklist"]
> * 使用擴展集的自動調整
> * 建立及使用自動調整規則
> * 對 VM 執行個體進行壓力測試，並觸發自動調整規則
> * 在需求降低時重新自動相應縮小

如需有效虛擬機器擴展集的更多範例，請參閱下列 Azure PowerShell 範例指令碼：

> [!div class="nextstepaction"]
> [Azure PowerShell 的擴展集指令碼範例](powershell-samples.md)
