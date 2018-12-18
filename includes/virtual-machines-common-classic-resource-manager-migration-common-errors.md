# <a name="common-errors-during-classic-to-azure-resource-manager-migration"></a>從傳統到 Azure Resource Manager 移轉的常見錯誤
本文收錄將 IaaS 資源從 Azure 傳統部署模型移轉至 Azure Resource Manager 堆疊的常見錯誤和緩和措施。

## <a name="list-of-errors"></a>錯誤清單
| 錯誤字串 | 緩和 |
| --- | --- |
| 內部伺服器錯誤 |在某些情況下，這是隨著重試消失的暫時性錯誤。 如果持續發生，請[連絡 Azure 支援](../articles/azure-supportability/how-to-create-azure-support-request.md)，因為需要平台記錄檔的調查。 <br><br> **注意︰** 一旦支援小組追蹤事件，請不要嘗試任何自我緩和，這可能會使您的環境中產生非預期的結果。 |
| 移轉不支援 HostedService {hosted-service-name} 中的部署 {deployment-name}，因為它是 PaaS 部署 (Web/背景工作角色)。 |這會在部署包含 Web/背景工作角色時發生。 因為移轉僅支援虛擬機器，請從部署移除 Web/背景工作角色，然後再試一次移轉。 |
| 範本 {template-name} 部署失敗。 相互關聯識別碼 = {guid} |在移轉服務後端，我們可以使用 Azure Resource Manager 範本來建立 Azure Resource Manager 堆疊中的資源。 因為範本具有等冪性，通常您可以安全地重試移轉作業，以通過這項錯誤。 如果此錯誤持續存在，請[連絡 Azure 支援](../articles/azure-supportability/how-to-create-azure-support-request.md)，並給予他們相互關聯識別碼。 <br><br> **注意︰** 一旦支援小組追蹤事件，請不要嘗試任何自我緩和，這可能會使您的環境中產生非預期的結果。 |
| 虛擬網路 {virtual-network-name} 不存在。 |如果您在新的 Azure 入口網站中建立虛擬網路，會發生這種情形。 實際虛擬網路名稱遵循 "Group * <VNET name>" 模式 |
| HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 {extension-name}，在 Azure Resource Manager 中不支援。 建議您先從 VM 中將它解除安裝，再繼續進行移轉。 |XML 擴充，例如 BGInfo 1.*，在 Azure Resource Manager 中不支援。 因此，這些擴充功能不能移轉。 如果這些擴充功能保持安裝在虛擬機器上，它們會在完成移轉之前自動解除安裝。 |
| HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 VMSnapshot/VMSnapshotLinux，目前不支援移轉。 從 VM 解除安裝，並且在移轉完成之後使用 Azure Resource Manager 將它新增回來 |這是虛擬機器針對 Azure 備份進行設定的案例。 由於這是目前不支援的案例，請遵循 https://aka.ms/vmbackupmigration 的因應措施 |
| HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 {extension-name}，其狀態未從 VM 報告。 因此，無法移轉此 VM。 請確定擴充的狀態已報告，或從 VM 解除安裝擴充，然後重試移轉。 <br><br> HostedService {hosted-service-name} 中的 VM {vm-name} 包含擴充 {extension-name}，報告處理常式狀態：{handler-status}。 因此，無法移轉 VM。 請確定報告的擴充處理常式狀態是 {handler-status}，或從 VM 解除安裝，然後重試移轉。 <br><br> HostedService {hosted-service-name} 中 VM {vm-name} 的 VM 代理程式將整體代理程式狀態報告為「未就緒」。 因此，VM 可能不會移轉，如果它有可移轉的擴充。 請確定 VM 代理程式將整體代理程式狀態報告為「就緒」。 請參閱 https://aka.ms/classiciaasmigrationfaqs。 |Azure 客體代理程式與 VM 擴充需要 VM 儲存體帳戶的輸出網際網路存取，來填入其狀態。 狀態失敗的常見原因包括 <li> 封鎖輸出網際網路存取的網路安全性群組 <li> 如果 VNET 有內部部署 DNS 伺服器且 DNS 連線遺失 <br><br> 如果您持續看到不支援的狀態，您可以解除安裝擴充以略過這項檢查，然後繼續進行移轉。 |
| 移轉不支援 HostedService {hosted-service-name} 中的部署 {deployment-name}，因為它有多個可用性設定組。 |目前，只能移轉具有 1 或更少可用性設定組的託管服務。 若要解決這個問題，請將額外可用性設定組和這些可用性設定組中的虛擬機器移至不同的託管服務。 |
| 移轉不支援 HostedService {hosted-service-name} 中的部署 {deployment-name}，因為它有不屬於可用性設定組的 VM，即使 HostedService 包含一個 VM。 |此案例的因應措施是將所有虛擬機器都移到單一可用性設定組，或在託管服務中從可用性設定組移除所有虛擬機器。 |
| 儲存體帳戶/HostedService/虛擬網路 {virtual-network-name} 正在移轉，因此無法變更 |在資源上已完成「準備」移轉作業，並且觸發會對資源進行變更的作業時，就會發生這個錯誤。 因為在「準備」作業之後會鎖定管理平面，所以對資源的任何變更都會遭到封鎖。 若要解除鎖定管理平面，您可以執行「認可」移轉作業以完成移轉，或「中止」移轉作業以回復「準備」作業。 |
| HostedService {hosted-service-name} 不允許移轉，因為它有 VM {vm-name} 處於下列「狀態」：RoleStateUnknown。 只有在 VM 處於下列其中一種狀態時才允許移轉 - 執行中、已停止、已停止解除配置。 |VM 可能會在轉換狀態時進行，通常發生在 HostedService 上的更新作業期間，例如重新啟動、擴充安裝等。建議在嘗試移轉之前在 HostedService 上完成更新作業。 |
| HostedService {hosted-service-name} 中的部署 {deployment-name} 包含具有資料磁碟 {data-disk-name} 的 VM {vm-name}，其實體 blob 大小 {size-of-the-vhd-blob-backing-the-data-disk} 個位元組不符合 VM 資料磁碟邏輯大小 {size-of-the-data-disk-specified-in-the-vm-api} 個位元組。 移轉會繼續進行，但不需指定 Azure Resource Manager VM 的資料磁碟大小。 | 如果您已調整 VHD blob 的大小，但未更新 VM API 模型中的大小，則會發生此錯誤。 詳細的緩和步驟說明[如下](#vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes)。|
| 驗證具有雲端服務 {Cloud Service name} 中 VM {VM name} 之媒體連結 {data disk Uri} 的資料磁碟 {data disk name} 時發生儲存體例外狀況。 請確定此虛擬機器的 VHD 媒體連結可供存取 | 如果 VM 的磁碟已被刪除或不再可供存取，就可能發生此錯誤。 請確定 VM 的磁碟存在。|
| HostedService {cloud-service-name} 中的 VM {vm-name} 包含 Disk with MediaLink {vhd-uri}，其具有 Azure Resource Manager 不支援的 blob 名稱 {vhd-blob-name}。 | 當 blob 的名稱包含計算資源提供者目前不支援的 "/" 時，就會發生此錯誤。 |
| HostedService {cloud-service-name} 中的 Deployment {deployment-name} 不允許移轉，因為它不在區域範圍中。 請參閱 http://aka.ms/regionalscope 以便將此部署移到區域範圍。 | 在 2014 年，Azure 宣布網路資源會從叢集層級範圍移到區域範圍。 如需詳細資訊，請參閱 [http://aka.ms/regionalscope] (http://aka.ms/regionalscope)。 當正在移轉的部署還沒有可自動將它移至區域範圍的更新作業時，就會發生此錯誤。 最佳的解決方法是將端點新增至 VM 或將資料磁碟新增至 VM，然後再試一次移轉。 <br> 請參閱[如何在 Azure 中的傳統 Windows 虛擬機器上設定端點](../articles/virtual-machines/windows/classic/setup-endpoints.md#create-an-endpoint)或[將資料磁碟連結至使用傳統部署模型建立的 Windows 虛擬機器](../articles/virtual-machines/windows/classic/attach-disk.md)|
| 因為虛擬網路 {vnet-name} 有非閘道的 PaaS 部署，所以無法移轉。 | 當您有非閘道 PaaS 部署 (例如，連線到虛擬網路的應用程式閘道或 API 管理服務) 時，就會發生此錯誤。|


## <a name="detailed-mitigations"></a>詳細的緩和措施

### <a name="vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes"></a>VM 資料磁碟的實體 blob 大小位元組不符合 VM 資料磁碟的邏輯大小位元組。

當資料磁碟的邏輯大小與實際的 VHD blob 大小不符時會發生此問題。 使用下列命令，即可輕鬆確認此問題︰

#### <a name="verifying-the-issue"></a>確認問題

```PowerShell
# Store the VM details in the VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

# Display the data disk properties
# NOTE the data disk LogicalDiskSizeInGB below which is 11GB. Also note the MediaLink Uri of the VHD blob as we'll use this in the next step
$vm.VM.DataVirtualHardDisks


HostCaching         : None
DiskLabel           : 
DiskName            : coreosvm-coreosvm-0-201611230636240687
Lun                 : 0
LogicalDiskSizeInGB : 11
MediaLink           : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
SourceMediaLink     : 
IOType              : Standard
ExtensionData       : 

# Now get the properties of the blob backing the data disk above
# NOTE the size of the blob is about 15 GB which is different from LogicalDiskSizeInGB above
$blob = Get-AzureStorageblob -Blob "coreosvm-dd1.vhd" -Container vhds 

$blob

ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudPageBlob
BlobType          : PageBlob
Length            : 16106127872
ContentType       : application/octet-stream
LastModified      : 11/23/2016 7:16:22 AM +00:00
SnapshotTime      : 
ContinuationToken : 
Context           : Microsoft.WindowsAzure.Commands.Common.Storage.AzureStorageContext
Name              : coreosvm-dd1.vhd
```

#### <a name="mitigating-the-issue"></a>緩和問題

```PowerShell
# Convert the blob size in bytes to GB into a variable which we'll use later
$newSize = [int]($blob.Length / 1GB)

# See the calculated size in GB
$newSize

15

# Store the disk name of the data disk as we'll use this to identify the disk to be updated
$diskName = $vm.VM.DataVirtualHardDisks[0].DiskName

# Identify the LUN of the data disk to remove
$lunToRemove = $vm.VM.DataVirtualHardDisks[0].Lun

# Now remove the data disk from the VM so that the disk isn't leased by the VM and it's size can be updated
Remove-AzureDataDisk -LUN $lunToRemove -VM $vm | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       213xx1-b44b-1v6n-23gg-591f2a13cd16   Succeeded  

# Verify we have the right disk that's going to be updated
Get-AzureDisk -DiskName $diskName

AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : 
Location             : East US
DiskSizeInGB         : 11
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 0c56a2b7-a325-123b-7043-74c27d5a61fd
OperationStatus      : Succeeded

# Now update the disk to the new size
Update-AzureDisk -DiskName $diskName -ResizedSizeInGB $newSize -Label $diskName

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureDisk     cv134b65-1b6n-8908-abuo-ce9e395ac3e7 Succeeded 

# Now verify that the "DiskSizeInGB" property of the disk matches the size of the blob 
Get-AzureDisk -DiskName $diskName


AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : coreosvm-coreosvm-0-201611230636240687
Location             : East US
DiskSizeInGB         : 15
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 1v53bde5-cv56-5621-9078-16b9c8a0bad2
OperationStatus      : Succeeded

# Now we'll add the disk back to the VM as a data disk. First we need to get an updated VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

Add-AzureDataDisk -Import -DiskName $diskName -LUN 0 -VM $vm -HostCaching ReadWrite | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       b0ad3d4c-4v68-45vb-xxc1-134fd010d0f8 Succeeded      
```

### <a name="moving-a-vm-to-a-different-subscription-after-completing-migration"></a>在完成移轉後將 VM 移到不同的訂用帳戶

完成移轉程序之後，您可以將 VM 移到另一個訂用帳戶。 不過，如果 VM 上有參考 Key Vault 資源的秘密/憑證，則目前不支援進行此移動。 下列指示將可讓您解決此問題。 

#### <a name="powershell"></a>PowerShell
```powershell
$vm = Get-AzureRmVM -ResourceGroupName "MyRG" -Name "MyVM"
Remove-AzureRmVMSecret -VM $vm
Update-AzureRmVM -ResourceGroupName "MyRG" -VM $vm
```
#### <a name="azure-cli"></a>Azure CLI

```bash
az vm update -g "myrg" -n "myvm" --set osProfile.Secrets=[]
```
