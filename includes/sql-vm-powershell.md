
## <a name="start-your-powershell-session"></a>啟動 PowerShell 工作階段
首先，您必須安裝並執行最新的 [Azure PowerShell](http://msdn.microsoft.com/library/mt619274.aspx) 。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。

> [!NOTE]
> 本主題中的範例會使用 [Azure Resource Manager 部署模型](../articles/azure-resource-manager/resource-group-overview.md)，因此範例使用 [Azure Resource Manager Cmdlet](http://msdn.microsoft.com/library/azure/mt125356.aspx)。 
> 
> 

執行 [**Connect-AzureRmAccount**](https://docs.microsoft.com/powershell/module/azurerm.profile/connect-azurermaccount) Cmdlet，您會看到要輸入認證的登入畫面。 使用您用來登入 Azure 入口網站的相同認證。

    Connect-AzureRmAccount

如果您有多個訂用帳戶，請使用 Set-[**AzureRmContext**](https://docs.microsoft.com/powershell/module/azurerm.profile/set-azurermcontext) Cmdlet 選取您的 PowerShell 工作階段應該使用的訂用帳戶。 若要查看目前的 PowerShell 工作階段正在使用哪個訂用帳戶，請執行 [**Get-AzureRmContext**](https://docs.microsoft.com/powershell/module/azurerm.profile/get-azurermcontext)。 若要查看所有訂用帳戶，請執行 [**Get-AzureRmSubscription**](https://docs.microsoft.com/powershell/module/servicemanagement/azurerm.profile/get-azurermsubscription)。

    Set-AzureRmContext -SubscriptionId '4cac86b0-1e56-bbbb-aaaa-000000000000'

