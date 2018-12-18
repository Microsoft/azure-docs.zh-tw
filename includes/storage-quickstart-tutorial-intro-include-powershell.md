## <a name="sign-in-to-azure"></a>登入 Azure

使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Connect-AzureRmAccount
```

如果您不知道要使用哪一個位置，您可以列出可用的位置。 當清單顯示之後，尋找您想要使用的位置。 此範例使用 **eastus**。 將此位置儲存在變數中並使用該變數，如此您在一處即可進行變更。

```powershell
Get-AzureRmLocation | select Location 
$location = "eastus"
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

```powershell
$resourceGroup = "myResourceGroup"
New-AzureRmResourceGroup -Name $resourceGroup -Location $location 
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/New-AzureRmStorageAccount) 建立可進行 LRS 複寫的一般用途儲存體帳戶，然後擷取儲存體帳戶內容，定義要使用的儲存體帳戶。 使用儲存體帳戶時，會參考內容而非重複提供認證。 這個範例會建立名為 mystorageaccount 的儲存體帳戶，其具有本機備援儲存體 (LRS) 和 blob 加密 (預設為啟用)。

```powershell
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $resourceGroup `
  -Name "mystorageaccount" `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind Storage

$ctx = $storageAccount.Context
```
