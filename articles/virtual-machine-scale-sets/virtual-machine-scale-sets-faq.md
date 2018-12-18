---
title: Azure 虛擬機器擴展集常見問題集 | Microsoft Docs
description: 取得關於虛擬機器擴展集常見問題集的解答。
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 76ac7fd7-2e05-4762-88ca-3b499e87906e
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/12/2017
ms.author: negat
ms.custom: na
ms.openlocfilehash: 9ec049ebbfaa8e6ee5dbf585488d5c2e8ff25802
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949701"
---
# <a name="azure-virtual-machine-scale-sets-faqs"></a>Azure 虛擬機器擴展集常見問題集

取得關於 Azure 中虛擬機器擴展集常見問題集的解答。

## <a name="top-frequently-asked-questions-for-scale-sets"></a>擴展集的主要常見問題集

**問：** 擴展集內可以有多少部 VM？

**答：** 擴展集可以有 0 到 1,000 部以平台映像為基礎的 VM，或 0-300 部以自訂映像為基礎的 VM。

**問：** 在擴展集內是否支援資料磁碟？

**答：** 是。 擴展集可以定義套用至集合中所有 VM 的連結資料磁碟組態。 如需詳細資訊，請參閱 [Azure 擴展集和連結的資料磁碟](virtual-machine-scale-sets-attached-disks.md)。 其他用於儲存資料的選項包括：

* Azure 檔案 (SMB 共用磁碟機)
* OS 磁碟機
* 暫存磁碟機 (本機，未受 Azure 儲存體支援)
* Azure 資料服務 (例如 Azure 資料表、Azure Blob)
* 外部資料服務 (例如遠端資料庫)

**問：** 哪些 Azure 區域支援擴展集？

**答：** 所有區域都支援擴展集。

**問：** 如何使用自訂映像建立擴展集？

**答：** 建立及擷取 VM 映像，然後使用該映像作為擴展集的來源。 如需有關如何建立及使用自訂 VM 映像的教學課程，您可以使用 [Azure CLI](tutorial-use-custom-image-cli.md) 或 [Azure PowerShell](tutorial-use-custom-image-powershell.md)

**問：** 如果我將擴展集容量從 20 減少為 15，哪些 VM 會被移除？

**答：** 虛擬機器會跨更新網域和容錯網域從擴展集平均地移除，以達到最大的可用性。 系統會先移除具有最高識別碼的 VM。

**問：** 如果我後續又將容量從 15 增加到 18 呢？

**答：** 如果您將容量增加到 18，則會建立 3 個新的 VM。 每次 VM 執行個體識別碼都會從上一個最高值遞增 (例如 20、21、22)。 VM 會平衡分散到容錯網域和更新網域。

**問：** 在擴展集內使用多個延伸模組時，是否可以強制執行「執行順序」？

**答：** 不是直接，但針對 customScript 擴充，您的指令碼可以等候另一個擴充完成。 您可以在部落格文章︰[Extension Sequencing in Azure virtual machine Scale Sets (Azure 虛擬機器擴展集中的擴充排序)](https://msftstack.wordpress.com/2016/05/12/extension-sequencing-in-azure-vm-scale-sets/) 中取得有關擴充排序的其他指導方針。

**問：** 擴展集是否可與 Azure 可用性設定組組搭配使用？

**答：** 區域 (非區域) 擴展集會使用「放置群組」，其可作為隱含的可用性設定組，並具有五個容錯網域和五個更新網域。 具有 100 部以上 VM 的擴展集會跨越多個放置群組。 如需放置群組的詳細資訊，請參閱[使用大型的虛擬機器擴展集](virtual-machine-scale-sets-placement-groups.md)。 VM 的可用性設定組可以存在於與 VM 擴展集相同的虛擬網路中。 常見組態是在可用性設定組中放入控制節點 VM (其通常需要唯一組態)，以及在擴展集中放入資料節點。

**問：** 擴展集是否可與 Azure 可用性區域搭配使用？

**答：** 可以！ 如需詳細資訊，請參閱[擴展集區域文件](./virtual-machine-scale-sets-use-availability-zones.md)。


## <a name="autoscale"></a>Autoscale

### <a name="what-are-best-practices-for-azure-autoscale"></a>什麼是 Azure 自動調整的最佳做法？

如需自動調整的最佳做法，請參閱[自動調整虛擬機器的最佳做法](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-autoscale-best-practices)。

### <a name="where-do-i-find-metric-names-for-autoscaling-that-uses-host-based-metrics"></a>哪裡可以找到使用主機型計量自動調整的計量名稱？

如需使用主機型計量之自動調整的計量名稱，請參閱[支援的 Azure 監視器度量](https://azure.microsoft.com/documentation/articles/monitoring-supported-metrics/)。

### <a name="are-there-any-examples-of-autoscaling-based-on-an-azure-service-bus-topic-and-queue-length"></a>是否有任何根據 Azure 服務匯流排主題和佇列長度自動調整的範例？

是。 如需根據 Azure 服務匯流排主題和佇列長度自動調整的範例，請參閱 [Azure 監視器的自動調整常用計量](https://azure.microsoft.com/documentation/articles/insights-autoscale-common-metrics/)。

如需服務匯流排佇列，請使用下列 JSON：

```json
"metricName": "MessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue"
```

如需儲存體佇列，請使用下列 JSON：

```json
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ClassicStorage/storageAccounts/mystorage/services/queue/queues/mystoragequeue"
```

以您資源的統一資源識別項 (URI) 取代範例值。


### <a name="should-i-autoscale-by-using-host-based-metrics-or-a-diagnostics-extension"></a>我應該使用主機型計量或診斷擴充功能進行自動調整？

您可以在 VM 上建立自動調整設定，以使用主機層級計量或客體 OS 型計量。

如需支援的計量資訊，請參閱 [Azure 監視器的自動調整常見計量](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-autoscale-common-metrics)。 

如需虛擬機器擴展集的完整範例，請參閱[使用虛擬機器擴展集的 Resource Manager 範本進行進階自動調整設定](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-advanced-autoscale-virtual-machine-scale-sets)。 

此範例使用主機層級 CPU 計量及訊息計數計量。



### <a name="how-do-i-set-alert-rules-on-a-virtual-machine-scale-set"></a>如何設定虛擬機器擴展集的警示規則？

您可以透過 PowerShell 或 Azure CLI，建立虛擬機器擴展集計量的警示。 如需詳細資訊，請參閱 [Azure 監視器 PowerShell 快速入門範例](https://azure.microsoft.com/documentation/articles/insights-powershell-samples/#create-alert-rules)和 [Azure 監視器跨平台 CLI 快速入門範例](https://azure.microsoft.com/documentation/articles/insights-cli-samples/#work-with-alerts)。

虛擬機器擴展集的 TargetResourceId 如下所示：

/subscriptions/yoursubscriptionid/resourceGroups/yourresourcegroup/providers/Microsoft.Compute/virtualMachineScaleSets/yourvmssname

您可以選擇任何 VM 效能計數器做為要設定警示的計量。 如需詳細資訊，請參閱 [Azure 監視器的自動調整常用計量](https://azure.microsoft.com/documentation/articles/insights-autoscale-common-metrics/)一文中的 [Resource Manager 型 Windows VM 的客體 OS 計量](https://azure.microsoft.com/documentation/articles/insights-autoscale-common-metrics/#guest-os-metrics-resource-manager-based-windows-vms)和 [Linux VM 的客體 OS 計量](https://azure.microsoft.com/documentation/articles/insights-autoscale-common-metrics/#guest-os-metrics-linux-vms)。

### <a name="how-do-i-set-up-autoscale-on-a-virtual-machine-scale-set-by-using-powershell"></a>如何使用 PowerShell 在虛擬機器擴展集上設定自動調整？

若要使用 PowerShell 在虛擬機器擴展集上設定自動調整，請參閱[自動調整虛擬機器擴展集](tutorial-autoscale-powershell.md)。 您也可以透過 [Azure CLI](tutorial-autoscale-cli.md) 與 [Azure 範本](tutorial-autoscale-template.md)設定自動調整規模


### <a name="if-i-have-stopped-deallocated-a-vm-is-that-vm-started-as-part-of-an-autoscale-operation"></a>如果我已停止 (已取消配置) VM，該 VM 是否在自動調整作業中啟動？

否。 如果自動調整規則需有額外的 VM 執行個體作為擴展集的一部分，則會建立新的 VM 執行個體。 已停止 (已取消配置) 的 VM 執行個體不會在自動調整事件中啟動。 不過，這些已停止 (已取消配置) 的 VM 可能會在執行個體數目相應縮小的自動調整事件中遭到刪除，同樣地系統可以根據 VM 執行個體識別碼的順序刪除任何 VM 執行個體。



## <a name="certificates"></a>憑證

### <a name="how-do-i-securely-ship-a-certificate-to-the-vm-how-do-i-provision-a-virtual-machine-scale-set-to-run-a-website-where-the-ssl-for-the-website-is-shipped-securely-from-a-certificate-configuration-the-common-certificate-rotation-operation-would-be-almost-the-same-as-a-configuration-update-operation-do-you-have-an-example-of-how-to-do-this"></a>如何安全地將憑證送至 VM？ 如何佈建虛擬機器擴展集來執行網站 (其中網站的 SSL 會安全地從憑證組態送出)？ (常見的憑證旋轉作業幾乎與組態更新作業相同。)您是否有如何進行這項操作的範例？ 

若要安全地將憑證送至 VM，您可以將客戶憑證直接安裝至客戶金鑰保存庫中的 Windows 憑證存放區。

使用下列 JSON：

```json
"secrets": [
    {
        "sourceVault": {
            "id": "/subscriptions/{subscriptionid}/resourceGroups/myrg1/providers/Microsoft.KeyVault/vaults/mykeyvault1"
        },
        "vaultCertificates": [
            {
                "certificateUrl": "https://mykeyvault1.vault.azure.net/secrets/{secretname}/{secret-version}",
                "certificateStore": "certificateStoreName"
            }
        ]
    }
]
```

程式碼支援 Windows 和 Linux。

如需詳細資訊，請參閱[建立或更新虛擬機器擴展集](https://msdn.microsoft.com/library/mt589035.aspx)。


### <a name="example-of-self-signed-certificate"></a>自我簽署憑證的範例

1.  在金鑰保存庫中建立自我簽署憑證。

    使用下列 PowerShell 命令：

    ```powershell
    Import-Module "C:\Users\mikhegn\Downloads\Service-Fabric-master\Scripts\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1"

    Connect-AzureRmAccount

    Invoke-AddCertToKeyVault -SubscriptionId <Your SubID> -ResourceGroupName KeyVault -Location westus -VaultName MikhegnVault -CertificateName VMSSCert -Password VmssCert -CreateSelfSignedCertificate -DnsName vmss.mikhegn.azure.com -OutputPath c:\users\mikhegn\desktop\
    ```

    此命令可讓您輸入 Azure Resource Manager 範本。

    如需如何在金鑰保存庫中建立自我簽署憑證的範例，請參閱 [Service Fabric 叢集安全性案例](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/)一文。

2.  變更 Resource Manager 範本。

    將這個屬性新增至 **virtualMachineProfile** 成為虛擬機器擴展集資源的一部分︰

    ```json 
    "osProfile": {
        "computerNamePrefix": "[variables('namingInfix')]",
        "adminUsername": "[parameters('adminUsername')]",
        "adminPassword": "[parameters('adminPassword')]",
        "secrets": [
            {
                "sourceVault": {
                    "id": "[resourceId('KeyVault', 'Microsoft.KeyVault/vaults', 'MikhegnVault')]"
                },
                "vaultCertificates": [
                    {
                        "certificateUrl": "https://mikhegnvault.vault.azure.net:443/secrets/VMSSCert/20709ca8faee4abb84bc6f4611b088a4",
                        "certificateStore": "My"
                    }
                ]
            }
        ]
    }
    ```
  

### <a name="can-i-specify-an-ssh-key-pair-to-use-for-ssh-authentication-with-a-linux-virtual-machine-scale-set-from-a-resource-manager-template"></a>是否可指定 SSH 金鑰組以便透過 Resource Manager 範本中的 Linux 虛擬機器擴展集來進行 SSH 驗證？  

是。 **osProfile** 的 REST API 類似於標準 VM REST API。 

在您的範本中包含 **osProfile**︰

```json 
"osProfile": {
    "computerName": "[variables('vmName')]",
    "adminUsername": "[parameters('adminUserName')]",
    "linuxConfiguration": {
        "disablePasswordAuthentication": "true",
        "ssh": {
            "publicKeys": [
                {
                    "path": "[variables('sshKeyPath')]",
                    "keyData": "[parameters('sshKeyData')]"
                }
            ]
        }
    }
}
```
 
此 JSON 區塊使用於 [101-vm-sshkey GitHub 快速入門範本](https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-sshkey/azuredeploy.json)。
 
OS 設定檔也會使用於 [grelayhost.json GitHub 快速入門範本](https://github.com/ExchMaster/gadgetron/blob/master/Gadgetron/Templates/grelayhost.json)。

如需詳細資訊，請參閱[建立或更新虛擬機器擴展集](https://msdn.microsoft.com/library/azure/mt589035.aspx#linuxconfiguration)。
  

### <a name="how-do-i-remove-deprecated-certificates"></a>如何移除已被取代的憑證？ 

若要移除已被取代的憑證，請從保存庫憑證清單中移除舊的憑證。 將您想要的所有憑證保留在清單中的電腦上。 這樣不會從所有的 VM 移除憑證。 但也不會將憑證新增至虛擬機器擴展集中所建立的新 VM。 

若要從現有的 VM 中移除憑證，請撰寫自訂指令碼擴充孤能，以從您的憑證存放區中手動移除憑證。
 
### <a name="how-do-i-inject-an-existing-ssh-public-key-into-the-virtual-machine-scale-set-ssh-layer-during-provisioning-i-want-to-store-the-ssh-public-key-values-in-azure-key-vault-and-then-use-them-in-my-resource-manager-template"></a>如何在佈建期間將現有的 SSH 公開金鑰插入至虛擬機器擴展集 SSH 層？ 我想要將 SSH 公開金鑰值儲存在 Azure Key Vault 中，然後在我的 Resource Manager 範本中使用它們。

如果您只透過 SSH 公開金鑰提供 VM，則不需要將公開金鑰放在 Key Vault 中。 公開金鑰不是密碼。
 
您可以在建立 Linux VM 時以純文字提供 SSH 公開金鑰：

```json
"linuxConfiguration": {
    "ssh": {
        "publicKeys": [
            {
                "path": "path",
                "keyData": "publickey"
            }
        ]
    }
```
 
linuxConfiguration 元素名稱 | 必要 | 類型 | 說明
--- | --- | --- | --- |  ---
ssh | 否 | 集合 | 指定 Linux OS 的 SSH 金鑰組態
path | 是 | 字串 | 指定 SSH 金鑰或憑證必須位於的 Linux 檔案路徑
keyData | 是 | 字串 | 指定 base64 編碼的 SSH 公開金鑰

如需範例，請參閱 [101-vm-sshkey GitHub 快速入門範本](https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-sshkey/azuredeploy.json)。

 
### <a name="when-i-run-update-azurermvmss-after-adding-more-than-one-certificate-from-the-same-key-vault-i-see-the-following-message"></a>當我在從相同金鑰保存庫新增一個以上的憑證之後執行 `Update-AzureRmVmss` 時，我會看到下列錯誤︰
 
>Update-AzureRmVmss: 清單密碼包含重複的 /subscriptions/<my-subscription-id>/resourceGroups/internal-rg-dev/providers/Microsoft.KeyVault/vaults/internal-keyvault-dev 執行個體，不允許這種情況。
 
如果您嘗試重新新增相同的保存庫，而不是對現有的來源保存庫使用新的保存庫憑證，就會發生這種情形。 如果您要新增其他密碼，`Add-AzureRmVmssSecret` 命令無法正常運作。
 
若要從相同金鑰保存庫新增更多密碼，請更新 $vmss.properties.osProfile.secrets[0].vaultCertificates 清單。
 
如需預期的輸入結構，請參閱[建立或更新虛擬機器擴展集](https://msdn.microsoft.com/library/azure/mt589035.aspx)。
 
在金鑰保存庫中的虛擬機器擴展集物件中尋找密碼。 然後，將憑證參考 (URL 及密碼存放區名稱) 新增至與保存庫相關聯的清單。

> [!NOTE] 
> 目前，您無法使用虛擬機器擴展集 API 從 VM 中移除憑證。
>

新的 VM 不會有舊的憑證。 不過，具有憑證且已經部署的 VM 會有舊的憑證。
 
### <a name="can-i-push-certificates-to-the-virtual-machine-scale-set-without-providing-the-password-when-the-certificate-is-in-the-secret-store"></a>當憑證位於密碼存放區時，可以將憑證推送至虛擬機器擴展集而無須提供密碼？

您不需要使用硬式編碼在指令碼中寫入密碼。 您可以使用您用來執行部署指令碼的權限動態擷取密碼。 如果您的指令碼會從密碼存放區金鑰保存庫移動憑證，則密碼存放區 `get certificate` 命令也會輸出 pfx 檔案的密碼。
 
### <a name="how-does-the-secrets-property-of-virtualmachineprofileosprofile-for-a-virtual-machine-scale-set-work-why-do-i-need-the-sourcevault-value-when-i-have-to-specify-the-absolute-uri-for-a-certificate-by-using-the-certificateurl-property"></a>虛擬機器擴展集的 virtualMachineProfile.osProfile 的 Secrets 屬性如何運作？ 當我必須使用 certificateUrl 屬性來指定憑證的絕對 URI 時，為什麼需要 sourceVault 值？ 

必須在 OS 設定檔的 Secrets 屬性中顯示 Windows 遠端管理 (WinRM) 憑證參考。 

指出來源保存庫的目的是要強制執行使用者的 Azure 雲端服務模型中存在的存取控制清單 (ACL) 原則。 如果未指定來源保存庫，則沒有權限可存取密碼或將密碼部署至金鑰保存庫的使用者便能夠透過計算資源提供者 (CRP)。 甚至不存在的資源也會有 ACL。

如果您提供不正確的來源保存庫識別碼，但為有效的金鑰保存庫 URL，則系統會在您輪詢作業時報告錯誤。
 
### <a name="if-i-add-secrets-to-an-existing-virtual-machine-scale-set-are-the-secrets-injected-into-existing-vms-or-only-into-new-ones"></a>如果我將密碼新增至現有虛擬機器擴展集，則新的密碼會插入現有的 VM，或只會插入新的 VM？ 

憑證會新增至所有 VM，甚至是既有的 VM。 如果虛擬機器擴展集 upgradePolicy 屬性設定為 **manual**，當您在 VM 上執行手動更新時，憑證會新增至 VM。
 
### <a name="where-do-i-put-certificates-for-linux-vms"></a>將 Linux VM 的憑證放在哪裡？

若要了解如何部署 Linux VM 的憑證，請參閱[將憑證從客戶管理的金鑰保存庫部署到 VM](https://blogs.technet.microsoft.com/kv/2015/07/14/deploy-certificates-to-vms-from-customer-managed-key-vault/)。
  
### <a name="how-do-i-add-a-new-vault-certificate-to-a-new-certificate-object"></a>如何將新的保存庫憑證新增至新的憑證物件？

若要將保存庫憑證新增至現有的密碼，請參閱下列 PowerShell 範例。 只使用一個密碼物件。
 
```powershell
$newVaultCertificate = New-AzureRmVmssVaultCertificateConfig -CertificateStore MY -CertificateUrl https://sansunallapps1.vault.azure.net:443/secrets/dg-private-enc/55fa0332edc44a84ad655298905f1809
 
$vmss.VirtualMachineProfile.OsProfile.Secrets[0].VaultCertificates.Add($newVaultCertificate)
 
Update-AzureRmVmss -VirtualMachineScaleSet $vmss -ResourceGroup $rg -Name $vmssName
```
 
### <a name="what-happens-to-certificates-if-you-reimage-a-vm"></a>如果您重新安裝映像 VM，憑證會發生什麼事？

如果您重新安裝 VM 映像，則會刪除憑證。 重新安裝映像會刪除整個 OS 磁碟。 
 
### <a name="what-happens-if-you-delete-a-certificate-from-the-key-vault"></a>如果您從 Key Vault 中刪除憑證會發生什麼事？

如果從金鑰保存庫中刪除密碼，然後您對所有 VM 執行 `stop deallocate` 而後再次啟動它們，您會發生失敗。 發生失敗的原因是 CRP 需要從金鑰保存庫擷取密碼，但無法這麼做。 在此案例中，您可以從虛擬機器擴展集模型刪除憑證。 

CRP 元件不會保存客戶密碼。 如果您對虛擬機器擴展集中的所有 VM 執行 `stop deallocate`，則會刪除快取。 在此案例中會從金鑰保存庫中擷取密碼。

您不會在相應放大時遇到此問題，因為 Azure Service Fabric 中有一個快取的密碼副本 (在單一網狀架構租用戶模型中)。
 
### <a name="why-do-i-have-to-specify-the-exact-location-for-the-certificate-url-httpsname-of-the-vaultvaultazurenet443secretsexact-location-as-indicated-in-service-fabric-cluster-security-scenarioshttpsazuremicrosoftcomdocumentationarticlesservice-fabric-cluster-security"></a>為什麼我必須指定憑證 URL 的確切位置 (https://<name of the vault>.vault.azure.net:443/secrets/<exact location>)，如 [Service Fabric 叢集安全性案例](https://azure.microsoft.com/documentation/articles/service-fabric-cluster-security/)所示？
 
Azure Key Vault 文件表示「取得密碼 REST API」應傳回最新版的密碼 (若未指定版本的話)。
 
方法 | URL
--- | ---
GET | https://mykeyvault.vault.azure.net/secrets/{secret-name}/{secret-version}?api-version={api-version}

將 {密碼-名稱} 替換為名稱，以及將 {密碼-版本} 替換為您想要擷取的密碼版本。 可能會排除密碼版本。 在此情況下會擷取目前的版本。
  
### <a name="why-do-i-have-to-specify-the-certificate-version-when-i-use-key-vault"></a>使用 Key Vault 時，為什麼必須指定憑證版本？

指定憑證版本的 Key Vault 需求目的是要讓使用者清楚了解其 VM 上所部署的憑證。

如果您建立 VM，然後更新金鑰保存庫中的密碼，新的憑證不會下載至您的 VM。 但您的 VM 似乎會參考它，且新的 VM 會取得新的密碼。 若要避免這個問題，您必須參考密碼版本。

### <a name="my-team-works-with-several-certificates-that-are-distributed-to-us-as-cer-public-keys-what-is-the-recommended-approach-for-deploying-these-certificates-to-a-virtual-machine-scale-set"></a>我的小組會使用發佈給我們做為 .cer 公開金鑰的多個憑證。 將這些憑證部署至虛擬機器擴展集的建議方法為何？

若要將 .cer 公開金鑰部署至虛擬機器擴展集，您可以產生僅包含 .cer 檔案的 .pfx 檔案。 若要這樣做，請使用 `X509ContentType = Pfx`。 例如，載入 .cer 檔案做為 C# 或 PowerShell 中的 x509Certificate2 物件，然後呼叫此方法。 

如需詳細資訊，請參閱 [X509Certificate.Export 方法 (X509ContentType, String)](https://msdn.microsoft.com/library/24ww6yzk(v=vs.110.aspx))。

### <a name="i-do-not-see-an-option-for-users-to-pass-in-certificates-as-base64-strings-most-other-resource-providers-have-this-option"></a>我看不到可供使用者傳入憑證做為 base64 字串的選項。 大部分其他資源提供者都有這個選項。

若要模擬傳入憑證做為 base64 字串，您可以在 Resource Manager 範本中擷取最新版的 URL。 在 Resource Manager 範本中包含下列 JSON 屬性︰

```json 
"certificateUrl": "[reference(resourceId(parameters('vaultResourceGroup'), 'Microsoft.KeyVault/vaults/secrets', parameters('vaultName'), parameters('secretName')), '2015-06-01').secretUriWithVersion]"
```
 
### <a name="do-i-have-to-wrap-certificates-in-json-objects-in-key-vaults"></a>我們必須在金鑰保存庫的 JSON 物件中包裝憑證嗎？

在虛擬機器擴展集和 VM 中，憑證必須包裝在 JSON 物件中。 

我們也支援內容類型 application/x-pkcs12。 如需使用 application/x-pkcs12 的相關指示，請參閱 [Azure Key Vault 中的 PFX 憑證](http://www.rahulpnath.com/blog/pfx-certificate-in-azure-key-vault/)。
 
我們目前不支援 .cer 檔案。 若要使用 .cer 檔案，請將它們匯出到 .pfx 容器。



## <a name="compliance-and-security"></a>合規性和安全性

### <a name="are-virtual-machine-scale-sets-pci-compliant"></a>虛擬機器擴展集符合 PCI 規範嗎？

虛擬機器擴展集是以 CRP 為基礎的精簡 API 層。 這兩個元件都在 Azure 服務樹狀目錄中計算平台的一部分。

就相容性的觀點而言，虛擬機器擴展集是 Azure 計算平台的基本部分。 它們會與 CRP 本身共用一個小組、工具、程序、部署方法、安全性控制、Just-In-Time (JIT) 編譯、監控、警示等。 虛擬機器擴展集符合支付卡產業 (PCI) 規範，因為 CRP 是目前 PCI 資料安全標準 (DSS) 證明的一部分。

如需詳細資訊，請參閱 [Microsoft 信任中心](https://www.microsoft.com/TrustCenter/Compliance/PCI)。

### <a name="does-azure-managed-service-identityhttpsdocsmicrosoftcomazureactive-directorymsi-overview-work-with-virtual-machine-scale-sets"></a>[Azure 受控服務識別](https://docs.microsoft.com/azure/active-directory/msi-overview)是否可以與虛擬機器擴展集搭配運作？

是。 您可以在 Azure 快速入門範本中看到某些範例 MSI 範本。 Linux：[https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-msi-linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-msi-linux) \(英文\)。 Windows：[https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-msi-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-msi-windows) \(英文\)。


## <a name="extensions"></a>擴充功能

### <a name="how-do-i-delete-a-virtual-machine-scale-set-extension"></a>如何刪除虛擬機器擴展集擴充功能？

若要刪除虛擬機器擴展集擴充功能，請使用下列 PowerShell 範例︰

```powershell
$vmss = Get-AzureRmVmss -ResourceGroupName "resource_group_name" -VMScaleSetName "vmssName" 

$vmss=Remove-AzureRmVmssExtension -VirtualMachineScaleSet $vmss -Name "extensionName"

Update-AzureRmVmss -ResourceGroupName "resource_group_name" -VMScaleSetName "vmssName" -VirtualMacineScaleSet $vmss
```
 
您可以在 `$vmss` 中找到 extensionName 值。
   
### <a name="is-there-a-virtual-machine-scale-set-template-example-that-integrates-with-log-analytics"></a>是否有與 Log Analytics 整合的虛擬機器擴展集範本範例？

如需與 Log Analytics 整合的虛擬機器擴展集範本範例，請參閱[部署 Azure Service Fabric 叢集並使用 Log Analytics 來啟用監視](https://github.com/krnese/AzureDeploy/tree/master/OMS/MSOMS/ServiceFabric)中的第二個範例。
   
### <a name="extensions-seem-to-run-in-parallel-on-virtual-machine-scale-sets-this-causes-my-custom-script-extension-to-fail-what-can-i-do-to-fix-this"></a>擴充功能似乎會在虛擬機器擴展集上平行執行。 這會導致自訂指令碼擴充功能失敗。 可以做什麼來修正這個問題？

若要了解虛擬機器擴展集中的擴充功能排序，請參閱 [Azure 虛擬機器擴展集中的擴充功能排序](https://msftstack.wordpress.com/2016/05/12/extension-sequencing-in-azure-vm-scale-sets/)。
 
 
### <a name="how-do-i-reset-the-password-for-vms-in-my-virtual-machine-scale-set"></a>如何重設虛擬機器擴展集中 VM 的密碼？

變更擴展集內虛擬機器密碼的方式主要有兩種。

- 直接變更虛擬機器擴展集模型。 適用於計算 API 2017-12-01 版和更新版本。

    直接在擴展集模型中更新系統管理員認證 (例如，使用 Azure 資源總管、PowerShell 或 CLI)。 擴展集更新之後，所有的新虛擬機器就會有新的認證。 現有的虛擬機器則要在重新安裝映像後，才會擁有新的認證。 

- 使用虛擬機器存取延伸模組來重設密碼。

    使用下列 PowerShell 範例：
    
    ```powershell
    $vmssName = "myvmss"
    $vmssResourceGroup = "myvmssrg"
    $publicConfig = @{"UserName" = "newuser"}
    $privateConfig = @{"Password" = "********"}
     
    $extName = "VMAccessAgent"
    $publisher = "Microsoft.Compute"
    $vmss = Get-AzureRmVmss -ResourceGroupName $vmssResourceGroup -VMScaleSetName $vmssName
    $vmss = Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmss -Name $extName -Publisher $publisher -Setting $publicConfig -ProtectedSetting $privateConfig -Type $extName -TypeHandlerVersion "2.0" -AutoUpgradeMinorVersion $true
    Update-AzureRmVmss -ResourceGroupName $vmssResourceGroup -Name $vmssName -VirtualMachineScaleSet $vmss
    ```


### <a name="how-do-i-add-an-extension-to-all-vms-in-my-virtual-machine-scale-set"></a>如何將擴充功能新增至虛擬機器擴展集中的所有 VM？

將更新原則設定為**自動**時，使用新的擴充屬性重新部署範本便會更新所有 VM。

如果更新原則設定為**手動**，請先更新擴充功能，然後手動更新 VM 中的所有執行個體。

  
### <a name="if-the-extensions-associated-with-an-existing-virtual-machine-scale-set-are-updated-are-existing-vms-affected-that-is-will-the-vms-not-match-the-virtual-machine-scale-set-model-or-are-they-ignored-when-an-existing-machine-is-service-healed-or-reimaged-are-the-scripts-that-are-currently-configured-on-the-virtual-machine-scale-set-executed-or-are-the-scripts-that-were-configured-when-the-vm-was-first-created-used"></a>如果更新與現有虛擬機器擴展集相關聯的擴充功能，現有的 VM 是否會受影響？ (也就是，VM「不」符合虛擬機器擴展集模型嗎？)或者會忽略它們？ 當現有機器為服務所修復或重新安裝映像時，是否會執行目前在虛擬機器擴展集上設定的指令碼，或是否會使用第一次建立 VM 時設定的指令？

如果已更新虛擬機器擴展集模型中的擴充定義且 upgradePolicy 屬性設定為 **automatic**，則會更新 VM。 如果 upgradePolicy 屬性設定為 **manual**，則會將擴充功能標示為不符合模型。 

如果現有 VM 為服務所修復，它會呈現重新開機狀態，且不會重新執行擴充功能。 若已重新安裝映像，就像是以來源映像取代 OS 磁碟機。 系統會執行最新模型的任何特製化項目，例如擴充功能。
 
### <a name="how-do-i-join-a-virtual-machine-scale-set-to-an-active-directory-domain"></a>如何將虛擬機器擴展集加入至 Active Directory 網域？

若要將虛擬機器擴展集加入至 Active Directory (AD) 網域，您可以定義擴充功能。 

若要定義擴充功能，請使用 JsonADDomainExtension 屬性︰

```json
"extensionProfile": {
    "extensions": [
        {
            "name": "joindomain",
            "properties": {
                "publisher": "Microsoft.Compute",
                "type": "JsonADDomainExtension",
                "typeHandlerVersion": "1.3",
                "settings": {
                    "Name": "[parameters('domainName')]",
                    "OUPath": "[variables('ouPath')]",
                    "User": "[variables('domainAndUsername')]",
                    "Restart": "true",
                    "Options": "[variables('domainJoinOptions')]"
                },
                "protectedsettings": {
                    "Password": "[parameters('domainJoinPassword')]"
                }
            }
        }
    ]
}
```
 
### <a name="my-virtual-machine-scale-set-extension-is-trying-to-install-something-that-requires-a-reboot-for-example-commandtoexecute-powershellexe--executionpolicy-unrestricted-install-windowsfeature-name-fs-resource-manager-includemanagementtools"></a>我的虛擬機器擴展集擴充功能正嘗試安裝需要重新開機的項目。 例如："commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted Install-WindowsFeature –Name FS-Resource-Manager –IncludeManagementTools"

如果虛擬機器擴展集擴充功能正嘗試安裝需要重新開機的項目，您可以使用 Azure 自動化預期狀態設定 (自動化 DSC) 擴充功能。 如果作業系統是 Windows Server 2012 R2，Azure 會提取 Windows Management Framework (WMF) 5.0 安裝程式，重新開機，然後繼續進行設定。 
 
### <a name="how-do-i-turn-on-antimalware-in-my-virtual-machine-scale-set"></a>如何在虛擬機器擴展集中開啟反惡意程式碼？

若要在虛擬機器擴展集上開啟反惡意程式碼，請使用下列 PowerShell 範例︰

```powershell
$rgname = 'autolap'
$vmssname = 'autolapbr'
$location = 'eastus'
 
# Retrieve the most recent version number of the extension.
$allVersions= (Get-AzureRmVMExtensionImage -Location $location -PublisherName "Microsoft.Azure.Security" -Type "IaaSAntimalware").Version
$versionString = $allVersions[($allVersions.count)-1].Split(".")[0] + "." + $allVersions[($allVersions.count)-1].Split(".")[1]
 
$VMSS = Get-AzureRmVmss -ResourceGroupName $rgname -VMScaleSetName $vmssname
echo $VMSS
Add-AzureRmVmssExtension -VirtualMachineScaleSet $VMSS -Name "IaaSAntimalware" -Publisher "Microsoft.Azure.Security" -Type "IaaSAntimalware" -TypeHandlerVersion $versionString
Update-AzureRmVmss -ResourceGroupName $rgname -Name $vmssname -VirtualMachineScaleSet $VMSS 
```

### <a name="i-need-to-execute-a-custom-script-thats-hosted-in-a-private-storage-account-the-script-runs-successfully-when-the-storage-is-public-but-when-i-try-to-use-a-shared-access-signature-sas-it-fails-this-message-is-displayed-missing-mandatory-parameters-for-valid-shared-access-signature-linksas-works-fine-from-my-local-browser"></a>我需要執行裝載於私人儲存體帳戶的自訂指令碼。 當儲存體為公用儲存體時，指令碼會執行成功，但是當我嘗試使用共用存取簽章 (SAS) 時，它就會失敗。 隨即顯示此訊息：「遺漏有效的共用存取簽章的強制參數」。 Link+SAS 可從我的本機瀏覽器正常運作。

若要執行裝載於私人儲存體帳戶的自訂指令碼，請使用儲存體帳戶金鑰和名稱來進行受保護的設定。 如需詳細資訊，請參閱[適用於 Windows 的自訂指令碼擴充功能](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-extensions-customscript/#template-example-for-a-windows-vm-with-protected-settings)。


## <a name="networking"></a>網路功能
 
### <a name="is-it-possible-to-assign-a-network-security-group-nsg-to-a-scale-set-so-that-it-applies-to-all-the-vm-nics-in-the-set"></a>是否可以將「網路安全性群組」(NSG) 指派給擴展集，以便將它套用至擴展集中的所有 VM NIC？

是。 您可以透過在網路設定檔的 networkInterfaceConfigurations 區段中參考「網路安全性群組」，將它直接套用至擴展集。 範例：

```json
"networkProfile": {
    "networkInterfaceConfigurations": [
        {
            "name": "nic1",
            "properties": {
                "primary": "true",
                "ipConfigurations": [
                    {
                        "name": "ip1",
                        "properties": {
                            "subnet": {
                                "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', variables('vnetName'), '/subnets/subnet1')]"
                            }
                "loadBalancerInboundNatPools": [
                                {
                                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/inboundNatPools/natPool1')]"
                                }
                            ],
                            "loadBalancerBackendAddressPools": [
                                {
                                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/backendAddressPools/addressPool1')]"
                                 }
                            ]
                        }
                    }
                ],
                "networkSecurityGroup": {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', variables('nsgName'))]"
                }
            }
        }
    ]
}
```

### <a name="how-do-i-do-a-vip-swap-for-virtual-machine-scale-sets-in-the-same-subscription-and-same-region"></a>如何在相同訂用帳戶和相同區域中進行虛擬機器擴展集的 VIP 交換？

如果您擁有兩個含 Azure Load Balancer 前端的虛擬機器擴展集，而且它們位於相同的訂用帳戶和區域，則您可以解除配置每個擴展集的公用 IP 位址，並指派給另一個。 例如，請參閱 [VIP 交換：Azure Resource Manager 中藍綠色版本部署](https://msftstack.wordpress.com/2017/02/24/vip-swap-blue-green-deployment-in-azure-resource-manager/) \(英文\)。 這確實意味著在網路層級取消配置/配置資源時會產生延遲。 更快的選項是搭配兩個後端集區和一個路由規則來使用 Azure 應用程式閘道。 或者，您可以透過 [Azure App Service](https://azure.microsoft.com/services/app-service/) 裝載應用程式，以提供在預備與生產位置之間快速切換的支援。
 
### <a name="how-do-i-specify-a-range-of-private-ip-addresses-to-use-for-static-private-ip-address-allocation"></a>如何指定要用於靜態私人 IP 位址配置的私人 IP 位址範圍？

系統會從您指定的子網路選取 IP 位址。 

虛擬機器擴展集 IP 位址的配置方法永遠是「動態」的，但這並不表示可以變更這些 IP 位址。 在此情況下，「動態」只表示您不會在 PUT 要求中指定 IP 位址。 使用子網路指定靜態設定。 
    
### <a name="how-do-i-deploy-a-virtual-machine-scale-set-to-an-existing-azure-virtual-network"></a>如何將虛擬機器擴展集部署至現有的 Azure 虛擬網路？ 

若要將虛擬機器擴展集部署至現有的 Azure 虛擬網路，請參閱[將虛擬機器擴展集部署至現有的虛擬網路](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-existing-vnet)。 

### <a name="how-do-i-add-the-ip-address-of-the-first-vm-in-a-virtual-machine-scale-set-to-the-output-of-a-template"></a>如何將虛擬機器擴展集中第一個 VM 的 IP 位址新增至範本的輸出？

若要將虛擬機器擴展集中第一個 VM 的 IP 位址新增至範本的輸出，請參閱 [Azure Resource Manager︰取得虛擬機器擴展集私人 IP](http://stackoverflow.com/questions/42790392/arm-get-vmsss-private-ips)。

### <a name="can-i-use-scale-sets-with-accelerated-networking"></a>我可以搭配加速的網路使用擴展集嗎？

是。 若要使用加速的網路，請在擴展集的 networkInterfaceConfigurations 設定中，將enableAcceleratedNetworking 設為 true。 例如
```json
"networkProfile": {
    "networkInterfaceConfigurations": [
    {
        "name": "niconfig1",
        "properties": {
        "primary": true,
        "enableAcceleratedNetworking" : true,
        "ipConfigurations": [
                ]
            }
            }
        ]
        }
    }
    ]
}
```

### <a name="how-can-i-configure-the-dns-servers-used-by-a-scale-set"></a>如何設定擴展集所使用的 DNS 伺服器？

若要建立含自訂 DNS 設定的虛擬機器擴展集，請將 dnsSettings JSON 封包加入至擴展集 networkInterfaceConfigurations 區段。 範例：
```json
    "dnsSettings":{
        "dnsServers":["10.0.0.6", "10.0.0.5"]
    }
```

### <a name="how-can-i-configure-a-scale-set-to-assign-a-public-ip-address-to-each-vm"></a>如何設定擴展集以將公用 IP 位址指派給每部 VM？

若要建立虛擬機器擴展集以將公用 IP 位址指派給每部虛擬機器，請確定 Microsoft.Compute/virtualMachineScaleSets 資源的 API 版本為 2017-03-30，並將 _publicipaddressconfiguration_ JSON 封包新增至擴展集 ipConfigurations 區段。 範例：

```json
    "publicipaddressconfiguration": {
        "name": "pub1",
        "properties": {
        "idleTimeoutInMinutes": 15
        }
    }
```

### <a name="can-i-configure-a-scale-set-to-work-with-multiple-application-gateways"></a>我可以設定擴展集以搭配多個應用程式閘道使用嗎？

是。 您可以將多個應用程式閘道後端位址集區的資源識別碼新增至擴展集網路設定檔之 _ipConfigurations_ 區段中的 _applicationGatewayBackendAddressPools_ 清單。

## <a name="scale"></a>調整

### <a name="in-what-case-would-i-create-a-virtual-machine-scale-set-with-fewer-than-two-vms"></a>在何種情況下，我會建立具有兩部以下 VM 的虛擬機器擴展集？

建立具有兩部以下 VM 的虛擬機器擴展集的原因之一，就是要使用虛擬機器擴展集的彈性屬性。 例如，您可部署沒有任何 VM 的虛擬機器擴展集來定義您的基礎結構，而不必支付 VM 執行成本。 然後，當您準備好要部署 VM 時，將虛擬機器擴展集的「容量」增加至生產執行個體計數。

建立具有兩部以下 VM 的虛擬機器擴展集的另一個原因，就是相較於使用具有離散 VM 的可用性設定組，您可能比較不擔心可用性。 虛擬機器擴展集讓您能夠使用可取代的無差異計算單位。 此種一致性是虛擬機器擴展集與可用性設定組的關鍵區別指標。 許多無狀態的工作負載都不會追蹤個別的單位。 如果工作負載減少，您可以相應減少為一個計算單位，然後在工作負載增加時相應增加為許多計算單位。

### <a name="how-do-i-change-the-number-of-vms-in-a-virtual-machine-scale-set"></a>如何變更虛擬機器擴展集中的 VM 數目？

若要在 Azure 入口網站中變更虛擬機器擴展集中的虛擬機器數目，請在虛擬機器擴展集屬性區段中，按一下 [調整] 刀鋒視窗並使用滑動軸。 如需變更執行個體數目的做法，請參閱[變更虛擬機器擴展集的執行個體計數](https://msftstack.wordpress.com/2016/05/13/change-the-instance-count-of-an-azure-vm-scale-set/)。

### <a name="how-do-i-define-custom-alerts-for-when-certain-thresholds-are-reached"></a>如何定義觸達特定臨界值時的自訂警示？

您在處理指定之臨界值的警示時有一些彈性。 例如，您可以定義自訂的 webhook。 下列 webhook 範例來自 Resource Manager 範本：

```json
{
    "type": "Microsoft.Insights/autoscaleSettings",
    "apiVersion": "[variables('insightsApi')]",
    "name": "autoscale",
    "location": "[parameters('resourceLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]"
    ],
    "properties": {
        "name": "autoscale",
        "targetResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/',  resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]",
        "enabled": true,
        "notifications": [
            {
                "operation": "Scale",
                "email": {
                    "sendToSubscriptionAdministrator": true,
                    "sendToSubscriptionCoAdministrators": true,
                    "customEmails": [
                        "youremail@address.com"
                    ]
                },
                "webhooks": [
                    {
                        "serviceUri": "https://events.pagerduty.com/integration/0b75b57246814149b4d87fa6e1273687/enqueue",
                        "properties": {
                            "key1": "custommetric",
                            "key2": "scalevmss"
                        }
                    }
                ]
            }
        ],
```

在此範例中，警示會在觸達臨界值時移至 Pagerduty.com。



## <a name="patching-and-operations"></a>修補和作業

### <a name="how-do-i-create-a-scale-set-in-an-existing-resource-group"></a>我如何在現有資源群組中建立擴展集？

您尚無法從 Azure 入口網站在現有資源群組中建立擴展集，但您從 Azure Resource Manager 範本部署擴展集時，可以指定現有的資源群組。 您也可以在使用 Azure PowerShell 或 CLI 建立擴展集時，指定現有的資源群組。

### <a name="can-we-move-a-scale-set-to-another-resource-group"></a>是否可以將擴展集移至另一個資源群組？

是，您可以將擴展集資源移至新的訂用帳戶或資源群組。

### <a name="how-to-i-update-my-virtual-machine-scale-set-to-a-new-image-how-do-i-manage-patching"></a>如何將虛擬機器擴展集更新為新的映像？ 如何管理修補？

若要將虛擬機器擴展集更新為新的映像，以及管理修補，請參閱[升級虛擬機器擴展集](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set)。

### <a name="can-i-use-the-reimage-operation-to-reset-a-vm-without-changing-the-image-that-is-i-want-reset-a-vm-to-factory-settings-rather-than-to-a-new-image"></a>是否可以使用重新安裝映像作業來重設 VM，而不變更映像？ (也就是，我想要將 VM 重設為原廠設定，而不是新的映像。)

是，您可以使用重新安裝映像作業來重設 VM，而不需變更映像。 不過，如果虛擬機器擴展集參考 `version = latest` 的平台映像，當您呼叫 `reimage` 時，您的 VM 可以更新為較新版本的 OS 映像。

如需詳細資訊，請參閱[管理虛擬機器擴展集中的所有 VM](https://docs.microsoft.com/rest/api/virtualmachinescalesets/manage-all-vms-in-a-set)。

### <a name="is-it-possible-to-integrate-scale-sets-with-azure-log-analytics"></a>是否有可能將擴展集與 Azure Log Analytics 整合？

是，您可以藉由在擴展集 VM 上安裝 Log Analytics 擴充功能來整合。 Azure CLI 的範例如下：
```
az vmss extension set --name MicrosoftMonitoringAgent --publisher Microsoft.EnterpriseCloud.Monitoring --resource-group Team-03 --vmss-name nt01 --settings "{'workspaceId': '<your workspace ID here>'}" --protected-settings "{'workspaceKey': '<your workspace key here'}"
```
您可以在 OMS 入口網站中找到所需的 workspaceId 和 workspaceKey。 在 [概觀] 頁面中，按一下 [設定] 圖格。 按一下頂端的 [連接的來源] 索引標籤。

注意：如果您的擴展集 _upgradePolicy_ 設為 Manual，您必須透過呼叫升級，將擴充功能套用至集合中的所有虛擬機器。 在 CLI 中，這會是 _az vmss update-instances_。

## <a name="troubleshooting"></a>疑難排解

### <a name="how-do-i-turn-on-boot-diagnostics"></a>如何開啟開機診斷？

若要開啟開機診斷，首先建立儲存體帳戶。 然後，將此 JSON 區塊放在虛擬機器擴展集 **virtualMachineProfile** 中，並更新此虛擬機器擴展集︰

```json
"diagnosticsProfile": {
    "bootDiagnostics": {
        "enabled": true,
        "storageUri": "http://yourstorageaccount.blob.core.windows.net"
    }
}
```

建立新的 VM 時，VM 的 InstanceView 會顯示螢幕擷取畫面等的詳細資料。 以下是範例：
 
```json
"bootDiagnostics": {
    "consoleScreenshotBlobUri": "https://o0sz3nhtbmkg6geswarm5.blob.core.windows.net/bootdiagnostics-swarmagen-4157d838-8335-4f78-bf0e-b616a99bc8bd/swarm-agent-9574AE92vmss-0_2.4157d838-8335-4f78-bf0e-b616a99bc8bd.screenshot.bmp",
    "serialConsoleLogBlobUri": "https://o0sz3nhtbmkg6geswarm5.blob.core.windows.net/bootdiagnostics-swarmagen-4157d838-8335-4f78-bf0e-b616a99bc8bd/swarm-agent-9574AE92vmss-0_2.4157d838-8335-4f78-bf0e-b616a99bc8bd.serialconsole.log"
  }
```


## <a name="virtual-machine-properties"></a>虛擬機器屬性

### <a name="how-do-i-get-property-information-for-each-vm-without-making-multiple-calls-for-example-how-would-i-get-the-fault-domain-for-each-of-the-100-vms-in-my-virtual-machine-scale-set"></a>如何取得每部 VM 的屬性資訊，而不進行多次呼叫？ 例如，如何針對虛擬機器擴展集中的 100 部 VM 取得各自的容錯網域？

若要取得每部 VM 的屬性資訊，而不進行多次呼叫，您可以對下列資源 URI 進行 REST API`GET`，藉此呼叫 `ListVMInstanceViews`：

/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft.Compute/virtualMachineScaleSets/<scaleset_name>/virtualMachines?$expand=instanceView&$select=instanceView

### <a name="can-i-pass-different-extension-arguments-to-different-vms-in-a-virtual-machine-scale-set"></a>是否可以將不同的擴充引數傳遞至虛擬機器擴展集中的不同 VM？

否，您無法將不同的擴充引數傳遞至虛擬機器擴展集中的不同 VM。 不過，擴充功能可以根據它們執行所在 VM 的唯一屬性來行動，例如根據電腦名稱。 擴充功能也可以在 http://169.254.169.254 上查詢執行個體中繼資料，以取得 VM 詳細資訊。

### <a name="why-are-there-gaps-between-my-virtual-machine-scale-set-vm-machine-names-and-vm-ids-for-example-0-1-3"></a>為何我的虛擬機器擴展集 VM 電腦名稱與 VM 識別碼之間會有落差？ 例如：0、1、3...

您的虛擬機器擴展集 VM 電腦名稱與 VM 識別碼之間有落差，是因為虛擬機器擴展集將 **overprovision** 屬性設定為預設值 **true**。 如果過度佈建設定為 **true**，則會建立超過要求的 VM。 然後會刪除額外的 VM。 在此情況下，您會取得更高的部署可靠性，但代價是連續的命名和連續的網路位址轉譯 (NAT) 規則。 

您可以將此屬性設定為 **false**。 若為小型虛擬機器擴展集，這就不會大幅影響部署可靠性。

### <a name="what-is-the-difference-between-deleting-a-vm-in-a-virtual-machine-scale-set-and-deallocating-the-vm-when-should-i-choose-one-over-the-other"></a>刪除虛擬機器擴展集中的 VM 與解除配置 VM 之間的差異為何？ 我何時應該選擇優先其中一個？

刪除虛擬機器擴展集中的 VM 與解除配置 VM 之間的主要差異在於 `deallocate` 不會刪除虛擬硬碟 (VHD)。 執行 `stop deallocate` 會產生相關的儲存體成本。 基於下列原因之一，您可能會採用其中一種做法：

- 您想要停止支付計算成本，但又想要保留 VM 的磁碟狀態。
- 您想要更快速地啟動一組 VM，則可相應放大虛擬機器擴展集。
  - 關於這種情況，您可能已建立自己的自動調整引擎，並想要更快速的端對端擴展。
- 您的虛擬機器擴展集並未平均分散於各容錯網域或更新網域。 這可能是因為您選擇性地刪除 VM，或是因為 VM 在過度佈建之後遭到刪除。 在虛擬機器擴展集上接續執行 `stop deallocate` 和 `start`，可讓 VM 平均分散於各容錯網域或更新網域。

