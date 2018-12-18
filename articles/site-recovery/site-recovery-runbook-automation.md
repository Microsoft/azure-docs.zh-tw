---
title: 在 Azure Site Recovery 中將 Azure 自動化 Runbook 新增至復原方案 | Microsoft Docs
description: 了解 Azure Site Recovery 如何協助您使用 Azure 自動化來擴充復原方案。 了解如何在復原至 Azure 期間完成複雜的工作。
services: site-recovery
documentationcenter: ''
author: ruturaj
manager: gauravd
editor: ''
ms.assetid: ecece14d-5f92-4596-bbaf-5204addb95c2
ms.service: site-recovery
ms.devlang: powershell
ms.tgt_pltfrm: na
ms.topic: article
ms.workload: storage-backup-recovery
ms.date: 07/06/2018
ms.author: ruturajd@microsoft.com
ms.openlocfilehash: 55160f3c43d8cbfc5f8b3e6aaf26bcb911387c52
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578760"
---
# <a name="add-azure-automation-runbooks-to-recovery-plans"></a>將 Azure 自動化 Runbook 新增至復原方案
在本文中，我們說明如何將 Azure Site Recovery 與 Azure 自動化整合在一起，以協助您擴充復原方案。 復原方案可以協調使用 Site Recovery 保護的 VM 復原。 復原方案可複寫至次要雲端，也可以複寫至 Azure。 復原方案也有助於讓復原「保持一致精確」、「可重複執行」及「自動化」。 如果您將 VM 容錯移轉至 Azure，與 Azure 自動化的整合可擴充復原方案。 您可以使用它來執行 Runbook，以提供功能強大的自動化工作。

如果您是 Azure 自動化的新手，您可以[註冊](https://azure.microsoft.com/services/automation/)並[下載範例指令碼](https://azure.microsoft.com/documentation/scripts/)。 如需詳細資訊，以及了解如何使用[復原方案](./site-recovery-create-recovery-plans.md)來協調復原至 Azure，請參閱 [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/)。

在本文中，我們說明如何將 Azure 自動化 Runbook 整合至您的復原方案。 我們使用範例，將先前需要手動介入的基本工作自動化。 我們也會說明如何將多重步驟復原轉換成單鍵復原動作。

## <a name="customize-the-recovery-plan"></a>自訂復原方案
1. 移至 [Site Recovery] 復原方案資源刀鋒視窗。 在此範例中，復原方案新增了兩個 VM 來進行復原。 若要開始新增 Runbook，請按一下 [自訂] 索引標籤。

    ![按一下 [自訂] 按鈕](media/site-recovery-runbook-automation-new/essentials-rp.png)


2. 以滑鼠右鍵按一下 [群組 1: 開始]，然後選取 [新增後續動作]。

    ![以滑鼠右鍵按一下 [群組 1: 開始] 並新增後續動作](media/site-recovery-runbook-automation-new/customize-rp.png)

3. 按一下 [Choose a script] (選擇指令碼)。

4. 在 [更新動作] 刀鋒視窗中，將指令碼命名為 **Hello World**。

    ![[更新動作] 刀鋒視窗](media/site-recovery-runbook-automation-new/update-rp.png)

5. 輸入自動化帳戶名稱。
    >[!NOTE]
    > 自動化帳戶可以位於任何 Azure 區域中。 自動化帳戶必須與 Azure Site Recovery 保存庫位於相同的訂用帳戶中。

6. 在您的自動化帳戶中，選取一個 Runbook。 此 Runbook 是在執行復原方案時，於復原第一個群組後執行的指令碼。

7. 若要儲存指令碼，請按一下 [確定]。 指令碼會新增至 [群組 1: 後續步驟]。

    ![[群組 1: 開始] 的後續動作](media/site-recovery-runbook-automation-new/addedscript-rp.PNG)


## <a name="considerations-for-adding-a-script"></a>新增指令碼的考量

* 若要選擇**刪除步驟**或**更新指令碼**，請以滑鼠右鍵按一下指令碼。
* 指令碼可以在從內部部署電腦容錯移轉至 Azure 時，在 Azure 上執行； 也可以在從 Azure 容錯回復至內部部署電腦時，於關機前在 Azure 上當做主要站台指令碼來執行。
* 指令碼執行時會插入復原方案內容。 下列範例顯示內容變數：

    ```
            {"RecoveryPlanName":"hrweb-recovery",

            "FailoverType":"Test",

            "FailoverDirection":"PrimaryToSecondary",

            "GroupId":"1",

            "VmMap":{"7a1069c6-c1d6-49c5-8c5d-33bfce8dd183":

                    { "SubscriptionId":"7a1111111-c1d6-49c5-8c5d-111ce8dd183",

                    "ResourceGroupName":"ContosoRG",

                    "CloudServiceName":"pod02hrweb-Chicago-test",

                    "RoleName":"Fabrikam-Hrweb-frontend-test",

                    "RecoveryPointId":"TimeStamp"}

                    }

            }
    ```

    下表列出內容中每個變數的名稱和描述。

    | **變數名稱** | **說明** |
    | --- | --- |
    | RecoveryPlanName |正在執行之方案的名稱。 此變數可協助您根據復原方案名稱，而採取不同的動作。 您也可以重複使用指令碼。 |
    | FailoverType |指定容錯移轉是測試、已計劃，還是未計劃。 |
    | FailoverDirection |指定復原是主要或次要站台。 |
    | GroupID |識別復原方案執行時方案內的群組編號。 |
    | VmMap |群組中所有 VM 的陣列。 |
    | VMMap 索引鍵 |每個 VM 的唯一索引鍵 (GUID)。 與 VM 的適用 Azure Virtual Machine Manager (VMM) 識別碼相同。 |
    | SubscriptionId |建立 VM 的 Azure 訂用帳戶識別碼。 |
    | RoleName |正在復原之 Azure VM 的名稱。 |
    | CloudServiceName |在其下建立 VM 的 Azure 雲端服務名稱。 |
    | resourceGroupName|在其下建立 VM 的 Azure 資源群組名稱。 |
    | RecoveryPointId|VM 復原時間的時間戳記。 |

* 確定自動化帳戶具有下列模組：
    * AzureRM.profile
    * AzureRM.Resources
    * AzureRM.Automation
    * AzureRM.Network
    * AzureRM.Compute

所有模組都必須是相容版本。 確保所有模組均相容的簡單做法，就是使用所有模組的最新版本。

### <a name="access-all-vms-of-the-vmmap-in-a-loop"></a>在迴圈中存取 VMMap 的所有 VM
使用下列程式碼循環存取 Microsoft VMMap 的所有 VM：

```
$VMinfo = $RecoveryPlanContext.VmMap | Get-Member | Where-Object MemberType -EQ NoteProperty | select -ExpandProperty Name
$vmMap = $RecoveryPlanContext.VmMap
 foreach($VMID in $VMinfo)
 {
     $VM = $vmMap.$VMID                
             if( !(($VM -eq $Null) -Or ($VM.ResourceGroupName -eq $Null) -Or ($VM.RoleName -eq $Null))) {
         #this check is to ensure that we skip when some data is not available else it will fail
 Write-output "Resource group name ", $VM.ResourceGroupName
 Write-output "Rolename " = $VM.RoleName
     }
 }

```

> [!NOTE]
> 當指令碼是開機群組的前置動作時，資源群組名稱和角色名稱值會是空的。 只有當該群組的 VM 成功地容錯移轉，才會填入值。 指令碼是開機群組的後續動作。

## <a name="use-the-same-automation-runbook-in-multiple-recovery-plans"></a>在多個復原方案中使用相同的自動化 Runbook

您可以透過外部變數，在多個復原方案中使用單一指令碼。 您可以使用 [Azure 自動化變數](../automation/automation-variables.md)，儲存您在復原方案執行時可傳遞的參數。 您可以在變數前面加上復原方案名稱，為每個復原方案建立個別變數。 然後，使用這些變數作為參數。 您可以變更參數而不需要變更指令碼，但仍變更指令碼的運作方式。

### <a name="use-a-simple-string-variable-in-a-runbook-script"></a>在 Runbook 指令碼中使用簡單的字串變數

在此範例中，指令碼會接受網路安全性群組 (NSG) 的輸入，並將其套用至復原方案的 VM。

若要讓指令碼偵測正在執行的復原方案，請使用復原方案內容：

```
workflow AddPublicIPAndNSG {
    param (
          [parameter(Mandatory=$false)]
          [Object]$RecoveryPlanContext
    )

    $RPName = $RecoveryPlanContext.RecoveryPlanName
```

若要套用現有的 NSG，您必須知道 NSG 名稱和 NSG 資源群組名稱。 使用這些變數作為復原方案指令碼的輸入。 若要這樣做，請在自動化帳戶資產中建立兩個變數。 在變數名稱前面加上您要建立參數之復原方案的名稱。

1. 建立變數來儲存 NSG 名稱。 在變數名稱前面加上復原方案的名稱。

    ![建立 NSG 名稱變數](media/site-recovery-runbook-automation-new/var1.png)

2. 建立變數來儲存 NSG 的資源群組名稱。 在變數名稱前面加上復原方案的名稱。

    ![建立 NSG 資源群組名稱](media/site-recovery-runbook-automation-new/var2.png)


3.  在指令碼中，使用下列參考程式碼來取得變數值：

    ```
    $NSGValue = $RecoveryPlanContext.RecoveryPlanName + "-NSG"
    $NSGRGValue = $RecoveryPlanContext.RecoveryPlanName + "-NSGRG"

    $NSGnameVar = Get-AutomationVariable -Name $NSGValue
    $RGnameVar = Get-AutomationVariable -Name $NSGRGValue
    ```

4.  在 Runbook 中使用變數，將 NSG 套用至已容錯移轉之 VM 的網路介面：

    ```
    InlineScript {
    if (($Using:NSGname -ne $Null) -And ($Using:NSGRGname -ne $Null)) {
            $NSG = Get-AzureRmNetworkSecurityGroup -Name $Using:NSGname -ResourceGroupName $Using:NSGRGname
            Write-output $NSG.Id
            #Apply the NSG to a network interface
            #$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
            #Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd `
            #  -AddressPrefix 192.168.1.0/24 -NetworkSecurityGroup $NSG
        }
    }
    ```

針對每個復原方案，建立獨立變數讓您可以重複使用指令碼。 在開頭加上復原方案名稱。 如需此案例的完整端對端指令碼，請參閱 [Add a public IP and NSG to VMs during test failover of a Site Recovery recovery plan](https://gallery.technet.microsoft.com/Add-Public-IP-and-NSG-to-a6bb8fee) (在 Site Recovery 復原方案的測試容錯移轉期間將公用 IP 和 NSG 新增至 VM)。


### <a name="use-a-complex-variable-to-store-more-information"></a>使用複雜變數來儲存詳細資訊

假設您想要一個用來在特定 VM 上開啟公用 IP 的指令碼。 在另一個案例中，您可能想要將不同的 NSG 套用至不同的 VM (並非所有 VM)。 您可以讓指令碼可重複使用於任何復原方案。 每個復原方案可以有任意數目的 VM。 例如，SharePoint 復原有兩個前端。 基本企業營運 (LOB) 應用程式只有一個前端。 您無法為每個復原方案建立個別變數。

在下列範例中，我們使用新的技術，並在 Azure 自動化帳戶資產中建立[複雜變數](https://docs.microsoft.com/powershell/module/servicemanagement/azure/set-azureautomationvariable)。 您可以藉由指定多個值來執行這項作業。 您必須使用 Azure PowerShell 來完成下列步驟：

1. 在 PowerShell 中，登入您的 Azure 訂用帳戶：

    ```
    Connect-AzureRmAccount
    $sub = Get-AzureRmSubscription -Name <SubscriptionName>
    $sub | Select-AzureRmSubscription
    ```

2. 若要儲存參數，請使用復原方案的名稱來建立複雜變數：

    ```
    $VMDetails = @{"VMGUID"=@{"ResourceGroupName"="RGNameOfNSG";"NSGName"="NameOfNSG"};"VMGUID2"=@{"ResourceGroupName"="RGNameOfNSG";"NSGName"="NameOfNSG"}}
        New-AzureRmAutomationVariable -ResourceGroupName <RG of Automation Account> -AutomationAccountName <AA Name> -Name <RecoveryPlanName> -Value $VMDetails -Encrypted $false
    ```

3. 在此複雜變數中，**VMDetails** 是受保護 VM 的 VM 識別碼。 您若要取得 VM 識別碼，請在 Azure 入口網站中，檢視 VM 屬性。 下列螢幕擷取畫面顯儲存兩個 VM 之詳細資料的變數：

    ![使用 VM 識別碼作為 GUID](media/site-recovery-runbook-automation-new/vmguid.png)

4. 在您的 Runbook 中使用此變數。 如果在復原方案內容中找到指定的 VM GUID，請將 NSG 套用至 VM：

    ```
    $VMDetailsObj = Get-AutomationVariable -Name $RecoveryPlanContext.RecoveryPlanName
    ```

4. 在您的 Runbook 中，循環存取復原方案內容的 VM。 檢查 VM 是否存在於 **$VMDetailsObj** 中。 如果存在的話，則存取變數的屬性來套用 NSG：

    ```
        $VMinfo = $RecoveryPlanContext.VmMap | Get-Member | Where-Object MemberType -EQ NoteProperty | select -ExpandProperty Name
        $vmMap = $RecoveryPlanContext.VmMap

        foreach($VMID in $VMinfo) {
            Write-output $VMDetailsObj.value.$VMID

            if ($VMDetailsObj.value.$VMID -ne $Null) { #If the VM exists in the context, this will not b Null
                $VM = $vmMap.$VMID
                # Access the properties of the variable
                $NSGname = $VMDetailsObj.value.$VMID.'NSGName'
                $NSGRGname = $VMDetailsObj.value.$VMID.'NSGResourceGroupName'

                # Add code to apply the NSG properties to the VM
            }
        }
    ```

您可以將相同的指令碼用於不同的復原方案。 藉由在不同的變數中儲存對應至復原方案的值，來輸入不同的參數。

## <a name="sample-scripts"></a>範例指令碼

若要將範例指令碼部署至您的自動化帳戶，請按一下 [部署至 Azure] 按鈕。

[![部署至 Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

如需其他範例，請觀看下列影片。 該影片示範如何將兩層式 WordPress 應用程式復原至 Azure：


> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]


## <a name="additional-resources"></a>其他資源
* [Azure 自動化服務執行身分帳戶](../automation/automation-create-runas-account.md)
* [Azure 自動化概觀](http://msdn.microsoft.com/library/azure/dn643629.aspx "Azure 自動化概觀")
* [Azure 自動化範例指令碼](http://gallery.technet.microsoft.com/scriptcenter/site/search?f\[0\].Type=User&f\[0\].Value=SC%20Automation%20Product%20Team&f\[0\].Text=SC%20Automation%20Product%20Team "Azure 自動化範例指令碼")

## <a name="next-steps"></a>後續步驟
[深入了解](site-recovery-failover.md) 如何執行容錯移轉。
