
要診斷 Microsoft Azure 雲端服務的問題，必須在問題發生時收集服務在虛擬機器上的記錄檔。 您可以視需要使用 AzureLogCollector 延伸模組，從一或多個雲端服務虛擬機器 (從 Web 角色和背景工作角色) 執行一次性的記錄收集作業，並將收集到的檔案傳輸到 Azure 儲存體帳戶，完全不必遠端登入任何虛擬機器。

> [!NOTE]
> 在 http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.asp 可以找到大部分記錄資訊的說明。
> 
> 

根據要收集的檔案類型，會有兩種收集模式。

* **僅限 Azure 客體代理程式記錄檔 (GA)**。 此收集模式包含所有與 Azure 客體代理程式和其他 Azure 元件相關的記錄。
* **所有的記錄檔 (完整)**。 此收集模式會收集 GA 模式中的所有檔案，外加：
  
  * 系統和應用程式事件記錄檔
  * HTTP 錯誤記錄檔
  * IIS 記錄檔
  * 安裝記錄檔
  * 其他系統記錄檔

在這兩種收集模式中，都可以使用下列結構的集合來指定其他資料收集資料夾：

* **名稱**：集合的名稱，作為具有所收集檔案之 ZIP 檔案內的子資料夾名稱。
* **位置**：要收集之檔案所在虛擬機器資料夾的路徑。
* **SearchPattern**：要收集之檔案的名稱模式。 預設值為 “\*”
* **遞迴**：如果要收集的檔案以遞迴方式位於指定位置底下。

## <a name="prerequisites"></a>先決條件
* 擁有儲存體帳戶可進行擴充，以儲存產生的 ZIP 檔案。
* 使用 Azure PowerShell Cmdlet 0.8.0 版或更新版本。 如需詳細資訊，請參閱 [Azure 下載](https://azure.microsoft.com/downloads/)。

## <a name="add-the-extension"></a>新增延伸模組
您可以使用 [Microsoft Azure PowerShell](https://msdn.microsoft.com/library/dn495240.aspx) Cmdlet 或[服務管理 REST API](https://msdn.microsoft.com/library/ee460799.aspx)，來新增 AzureLogCollector 延伸模組。

對於雲端服務，可以使用現有的 Azure Powershell Cmdlet **Set-AzureServiceExtension**來啟用雲端服務角色執行個體上的延伸模組。 每次透過此 Cmdlet 啟用此延伸模組時，都會在所選角色的選定角色執行個體上觸發記錄檔收集。

對於虛擬機器，可以使用現有的 Azure Powershell Cmdlet **Set-AzureVMExtension**來啟用虛擬機器上的延伸模組。 每次透過這些 Cmdlet 啟用此延伸模組時，都會在每個執行個體上觸發記錄檔收集。

就內部而言，此延伸模組會使用以 JSON 為基礎的 PublicConfiguration 和 PrivateConfiguration。 以下是公用和私人組態的範例 JSON 配置。

### <a name="publicconfiguration"></a>PublicConfiguration

```json
{
    "Instances":  "*",
    "Mode":  "Full",
    "SasUri":  "SasUri to your storage account with sp=wl",
    "AdditionalData":
    [
      {
              "Name":  "StorageData",
              "Location":  "%roleroot%storage",
              "SearchPattern":  "*.*",
              "Recursive":  "true"
      },
      {
            "Name":  "CustomDataFolder2",
            "Location":  "c:\customFolder",
            "SearchPattern":  "*.log",
            "Recursive":  "false"
      },
    ]
}
```

### <a name="privateconfiguration"></a>PrivateConfiguration

```json
{

}
```

> [!NOTE]
> 此延伸模組不需要 **privateConfiguration**。 您可以只提供 **–PrivateConfiguration** 引數的空結構。
> 
> 

您可以依照下列兩個步驟之一，將 AzureLogCollector 新增至所選角色的一或多個雲端服務或虛擬機器執行個體，這會在每個 VM 上觸發收集的執行，並將收集到的檔案傳送至指定的 Azure 帳戶。

## <a name="adding-as-a-service-extension"></a>新增為服務延伸模組
1. 依照指示將 Azure PowerShell 連接到您的訂用帳戶。
2. 指定要新增並啟用 AzureLogCollector 延伸模組的服務名稱、位置、角色和角色執行個體。

  ```powershell
  #Specify your cloud service name
  $ServiceName = 'extensiontest2'

  #Specify the slot. 'Production' or 'Staging'
  $slot = 'Production'

  #Specified the roles on which the extension will be installed and enabled
  $roles = @("WorkerRole1","WebRole1")

  #Specify the instances on which extension will be installed and enabled.  Use wildcard * for all instances
  $instances = @("*")

  #Specify the collection mode, "Full" or "GA"
  $mode = "GA"
  ```

3. 指定收集檔案的其他資料夾 (此步驟是選擇性的)。

  ```powershell
  #add one location
  $a1 = New-Object PSObject

  $a1 | Add-Member -MemberType NoteProperty -Name "Name" -Value "StorageData"
  $a1 | Add-Member -MemberType NoteProperty -Name "SearchPattern" -Value "*"
  $a1 | Add-Member -MemberType NoteProperty -Name "Location" -Value "%roleroot%storage"  #%roleroot% is normally E: or F: drive
  $a1 | Add-Member -MemberType NoteProperty -Name "Recursive" -Value "true"

  $AdditionalDataList+= $a1
  #more locations can be added....
  ```

   > [!NOTE]
   > 您可以使用權杖 `%roleroot%` 來指定角色的根磁碟機，因為它不會使用固定磁碟機。
   > 
   > 
4. 提供收集到的檔案要上傳到的 Azure 儲存體帳戶名稱和金鑰。

  ```powershell
  $StorageAccountName = 'YourStorageAccountName'
  $StorageAccountKey  = 'YourStorageAccountKey'
  ```

5. 依照下列方式呼叫 SetAzureServiceLogCollector.ps1 (包含在本文結尾處)，以啟用雲端服務的 AzureLogCollector 延伸模組。 執行完成後，您可以在 `https://YourStorageAccountName.blob.core.windows.net/vmlogs` 找到上傳的檔案

  ```powershell
  .\SetAzureServiceLogCollector.ps1 -ServiceName YourCloudServiceName  -Roles $roles  -Instances $instances –Mode $mode -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey -AdditionDataLocationList $AdditionalDataList
  ```

以下是傳遞至指令碼之參數的定義。 (也可以從下方加以複製。)

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
  [Parameter(Mandatory=$true)]
  [string]   $ServiceName,

  [Parameter(Mandatory=$false)]
  [string[]] $Roles ,

  [Parameter(Mandatory=$false)]
  [string[]] $Instances,

  [Parameter(Mandatory=$false)]
  [string]   $Slot = 'Production',

  [Parameter(Mandatory=$false)]
  [string]   $Mode = 'Full',

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountName,

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountKey,

  [Parameter(Mandatory=$false)]
  [PSObject[]] $AdditionDataLocationList = $null
)
```

* **ServiceName**：您的雲端服務名稱。
* **Roles**：一份角色清單，例如 “WebRole1” 或 ”WorkerRole1”。
* **Instances**：一份以逗號分隔的角色執行個體名稱 -- 所有角色執行個體皆使用萬用字元字串 ("*")。
* **Slot**：位置名稱。 「生產」或「預備」。
* **Mode**：收集模式。 「完整」或「GA」。
* **StorageAccountName**：用來儲存收集之資料的 Azure 儲存體帳戶名稱。
* **StorageAccountKey**：Azure 儲存體帳戶金鑰的名稱。
* **AdditionalDataLocationList**：下列結構的清單：

  ```powershell
  {
    String Name,
    String Location,
    String SearchPattern,
    Bool   Recursive
  }
  ```

## <a name="adding-as-a-vm-extension"></a>新增為 VM 延伸模組
依照指示將 Azure PowerShell 連接到您的訂用帳戶。

1. 指定服務名稱、VM 和收集模式。

  ```powershell
  #Specify your cloud service name
  $ServiceName = 'YourCloudServiceName'

  #Specify the VM name
  $VMName = "'YourVMName'"

  #Specify the collection mode, "Full" or "GA"
  $mode = "GA"

  Specify the additional data folder for which files will be collected (this step is optional).

  #add one location
  $a1 = New-Object PSObject

  $a1 | Add-Member -MemberType NoteProperty -Name "Name" -Value "StorageData"
  $a1 | Add-Member -MemberType NoteProperty -Name "SearchPattern" -Value "*"
  $a1 | Add-Member -MemberType NoteProperty -Name "Location" -Value "%roleroot%storage"  #%roleroot% is normally E: or F: drive
  $a1 | Add-Member -MemberType NoteProperty -Name "Recursive" -Value "true"

  $AdditionalDataList+= $a1
        #more locations can be added....
  ```
  
2. 提供收集到的檔案要上傳到的 Azure 儲存體帳戶名稱和金鑰。

  ```powershell
  $StorageAccountName = 'YourStorageAccountName'
  $StorageAccountKey  = 'YourStorageAccountKey'
  ```

3. 依照下列方式呼叫 SetAzureVMLogCollector.ps1 (包含在本文結尾處)，以啟用雲端服務的 AzureLogCollector 延伸模組。 執行完成後，您可以在 `https://YourStorageAccountName.blob.core.windows.net/vmlogs` 找到上傳的檔案

以下是傳遞至指令碼之參數的定義。 (也可以從下方加以複製。)

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
  [Parameter(Mandatory=$true)]
  [string]   $ServiceName,

  [Parameter(Mandatory=$false)]
  [string] $VMName ,

  [Parameter(Mandatory=$false)]
  [string]   $Mode = 'Full',

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountName,

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountKey,

  [Parameter(Mandatory=$false)]
  [PSObject[]] $AdditionDataLocationList = $null
)
```

* **ServiceName**：您的雲端服務名稱。
* **VMName**：虛擬機器的名稱。
* **Mode**：收集模式。 「完整」或「GA」。
* **StorageAccountName**：用來儲存收集之資料的 Azure 儲存體帳戶名稱。
* **StorageAccountKey**：Azure 儲存體帳戶金鑰的名稱。
* **AdditionalDataLocationList**：下列結構的清單：

  ```
  {
    String Name,
    String Location,
    String SearchPattern,
    Bool   Recursive
  }
  ```

## <a name="extention-powershell-script-files"></a>延伸模組 PowerShell 指令碼檔案
### <a name="setazureservicelogcollectorps1"></a>SetAzureServiceLogCollector.ps1

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
  [Parameter(Mandatory=$true)]
  [string]   $ServiceName,

  [Parameter(Mandatory=$false)]
  [string[]] $Roles ,

  [Parameter(Mandatory=$false)]
  [string[]] $Instances = '*',

  [Parameter(Mandatory=$false)]
  [string]   $Slot = 'Production',

  [Parameter(Mandatory=$false)]
  [string]   $Mode = 'Full',

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountName,

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountKey,

  [Parameter(Mandatory=$false)]
  [PSObject[]] $AdditionDataLocationList = $null
)

$publicConfig = New-Object PSObject

if ($Instances -ne $null -and $Instances.Count -gt 0)  #Instances should be seperated by ,
{
  $instanceText = $Instances[0]
  for ($i = 1;$i -lt $Instances.Count;$i++)
  {
    $instanceText = $instanceText+ "," + $Instances[$i]
  }
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Instances" -Value $instanceText
}
else  #For all instances if not specified.  The value should be a space or *
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Instances" -Value " "
}

if ($Mode -ne $null )
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value $Mode
}
else
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value "Full"
}

#
#we need to get the Sasuri from StorageAccount and containers
#
$context = New-AzureStorageContext -Protocol https -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

$ContainerName = "azurelogcollectordata"
$existingContainer = Get-AzureStorageContainer -Context $context |  Where-Object { $_.Name -like $ContainerName}
if ($existingContainer -eq $null)
{
  "Container ($ContainerName) doesn't exist. Creating it now.."
  New-AzureStorageContainer -Context $context -Name $ContainerName -Permission off
}

$ExpiryTime =  [DateTime]::Now.AddMinutes(120).ToString("o")
$SasUri = New-AzureStorageContainerSASToken -ExpiryTime $ExpiryTime -FullUri -Name $ContainerName -Permission rwl -Context $context
$publicConfig | Add-Member -MemberType NoteProperty -Name "SasUri" -Value $SasUri

#
#Add AdditionalData to collect data from additional folders
#
if ($AdditionDataLocationList -ne $null )
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "AdditionalData" -Value $AdditionDataLocationList
}

#
# Convert it to JSON format
#
$publicConfigJSON = $publicConfig | ConvertTo-Json
"publicConfig is:  $publicConfigJSON"

#we just provide a empty privateConfig object
$privateconfig = "{
}"

if ($Roles -ne $null)
{
  Set-AzureServiceExtension -Service $ServiceName -Slot $Slot -Role $Roles -ExtensionName 'AzureLogCollector' -ProviderNamespace Microsoft.WindowsAzure.Compute -PublicConfiguration $publicConfigJSON -PrivateConfiguration $privateconfig -Version 1.0 -Verbose
}
else
{
  Set-AzureServiceExtension -Service $ServiceName -Slot $Slot  -ExtensionName 'AzureLogCollector' -ProviderNamespace Microsoft.WindowsAzure.Compute -PublicConfiguration $publicConfigJSON -PrivateConfiguration $privateconfig -Version 1.0 -Verbose
}

#
#This is an optional step: generate a sasUri to the container so it can be shared with other people if nened
#
$SasExpireTime = [DateTime]::Now.AddMinutes(120).ToString("o")
$SasUri = New-AzureStorageContainerSASToken -ExpiryTime $ExpiryTime -FullUri -Name $ContainerName -Permission rl -Context $context
$SasUri = $SasUri + "&restype=container&comp=list"
Write-Output "The container for uploaded file can be accessed using this link:`r`n$sasuri"
```

### <a name="setazurevmlogcollectorps1"></a>SetAzureVMLogCollector.ps1

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
              [Parameter(Mandatory=$true)]
              [string]   $ServiceName,

              [Parameter(Mandatory=$false)]
              [string] $VMName ,

              [Parameter(Mandatory=$false)]
              [string]   $Mode = 'Full',

              [Parameter(Mandatory=$false)]
              [string]   $StorageAccountName,

              [Parameter(Mandatory=$false)]
              [string]   $StorageAccountKey,

              [Parameter(Mandatory=$false)]
              [PSObject[]] $AdditionDataLocationList = $null
        )

$publicConfig = New-Object PSObject
$publicConfig | Add-Member -MemberType NoteProperty -Name "Instances" -Value "*"

if ($Mode -ne $null )
{
    $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value $Mode
}
else
{
    $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value "Full"
}

#
#we need to get the Sasuri from StorageAccount and containers
#
$context = New-AzureStorageContext -Protocol https -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

$ContainerName = "azurelogcollectordata"
$existingContainer = Get-AzureStorageContainer -Context $context |  Where-Object { $_.Name -like $ContainerName}
if ($existingContainer -eq $null)
{
    "Container ($ContainerName) doesn't exist. Creating it now.."
    New-AzureStorageContainer -Context $context -Name $ContainerName -Permission off
}

$ExpiryTime =  [DateTime]::Now.AddMinutes(90).ToString("o")
$SasUri = New-AzureStorageContainerSASToken -ExpiryTime $ExpiryTime -FullUri -Name $ContainerName -Permission rwl -Context $context
$publicConfig | Add-Member -MemberType NoteProperty -Name "SasUri" -Value $SasUri

#
#Add AdditionalData to collect data from additional folders
#
if ($AdditionDataLocationList -ne $null )
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "AdditionalData" -Value $AdditionDataLocationList
}

#
# Convert it to JSON format
#
$publicConfigJSON = $publicConfig | ConvertTo-Json

Write-Output "PublicConfigurtion is: \r\n$publicConfigJSON"

#
#we just provide a empty privateConfig object
#
$privateconfig = "{
}"

if ($VMName -ne $null )
{
      $VM = Get-AzureVM -ServiceName $ServiceName -Name $VMName
      $VM.VM.OSVirtualHardDisk.OS

      if ($VM.VM.OSVirtualHardDisk.OS -like '*Windows*')
      {
            Set-AzureVMExtension -VM $VM -ExtensionName "AzureLogCollector" -Publisher Microsoft.WindowsAzure.Compute -PublicConfiguration $publicConfigJSON -PrivateConfiguration $privateconfig -Version 1.* | Update-AzureVM -Verbose

            #
            #We will check the VM status to find if operation by extension has been completed or not. The completion of the operation,either succeed or fail, can be indicated by
            #the presence of SubstatusList field.
            #
            $Completed = $false
            while ($Completed -ne $true)
            {
                    $VM = Get-AzureVM -ServiceName $ServiceName -Name $VMName
                    $status = $VM.ResourceExtensionStatusList | Where-Object {$_.HandlerName -eq "Microsoft.WindowsAzure.Compute.AzureLogCollector"}

                    if ( ($status.Code -ne 0) -and ($status.Status -like '*error*'))
                    {
                        Write-Output "Error status is returned: $($Status.ExtensionSettingStatus.FormattedMessage.Message)."
                          $Completed = $true
                    }
                    elseif (($status.ExtensionSettingStatus.SubstatusList -eq $null -or $status.ExtensionSettingStatus.SubstatusList.Count -lt 1))
                    {
                          $Completed = $false
                          Write-Output "Waiting for operation to complete..."
                    }
                    else
                    {
                          $Completed = $true
                          Write-Output "Operation completed."

                    $UploadedFileUri = $Status.ExtensionSettingStatus.SubStatusList[0].FormattedMessage.Message
                          $blob = New-Object Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob($UploadedFileUri)

                  #
                        # This is an optional step:  For easier access to the file, we can generate a read-only SasUri directly to the file
                          #
                          $ExpiryTimeRead =  [DateTime]::Now.AddMinutes(120).ToString("o")
                          $ReadSasUri = New-AzureStorageBlobSASToken -ExpiryTime $ExpiryTimeRead  -FullUri  -Blob  $blob.name -Container $blob.Container.Name -Permission r -Context $context

                        Write-Output "The uploaded file can be accessed using this link: $ReadSasUri"

                          #
                          #This is an optional step:  Remove the extension after we are done
                          #
                          Get-AzureVM -ServiceName $ServiceName -Name $VMName | Set-AzureVMExtension -Publisher Microsoft.WindowsAzure.Compute -ExtensionName "AzureLogCollector" -Version 1.* -Uninstall | Update-AzureVM -Verbose

                    }
                    Start-Sleep -s 5
            }
      }
      else
      {
          Write-Output "VM OS Type is not Windows, the extension cannot be enabled"
      }

}
else
{
  Write-Output "VM name is not specified, the extension cannot be enabled"
}
```

## <a name="next-steps"></a>後續步驟
現在，您可以從一個簡單的位置檢查或複製您的記錄檔。

