

VM 擴充功能可協助您：

* 修改安全性與身分識別的功能，例如重設帳戶值和使用反惡意程式碼
* 啟動、停止或設定監視和診斷
* 重設或安裝連線功能，例如 RDP 和 SSH
* 診斷、監視和管理您的 VM

也有許多其他功能。 會定期發行新的 VM 擴充功能。 這篇文章描述 Windows 和 Linux 的 Azure VM 代理程式，以及它們如何支援 VM 擴充功能。 如需依功能分類的 VM 延伸模組清單，請參閱 [Azure VM 延伸模組與功能](../articles/virtual-machines/extensions/features-windows.md)。

## <a name="azure-vm-agents-for-windows-and-linux"></a>適用於 Windows 和 Linux 的 Azure VM 代理程式
Azure 虛擬機器代理程式 (VM 代理程式) 是一個安全、輕量級程序，它會安裝、設定和移除 Azure 虛擬機器執行個體上的 VM 擴充功能。 VM 代理程式會做為 Azure VM 的安全本機控制服務。 代理程式載入的擴充功能提供特定功能，以提升使用執行個體時的產能。

有兩個 Azure VM 代理程式存在，一個適用於 Windows VM，一個適用於 Linux VM。

如果您想要讓虛擬機器執行個體使用一或多個 VM 擴充功能，執行個體必須安裝 VM 代理程式。 藉由使用 Azure 入口網站和來自 **Marketplace** 之映像所建立的虛擬機器映像，會在建立程序中自動安裝 VM 代理程式。 如果虛擬機器執行個體沒有 VM 代理程式，您可以在建立虛擬機器執行個體之後安裝 VM 代理程式。 或者，您可以在您稍後上傳的自訂 VM 映像中安裝代理程式。

> [!IMPORTANT]
> 這些 VM 代理程式非常精簡的服務，能夠進行虛擬機器執行個體的安全管理。 有些情況下您可能不想使用 VM 代理程式。 若是如此，請務必建立不會使用 Azure CLI 或 PowerShell 安裝 VM 代理程式的 VM。 雖然可以實際移除 VM 代理程式，但是執行個體上的 VM 擴充功能的行為並未定義。 因此，不支援移除已安裝的 VM 代理程式。
>

在下列情況下會啟用 VM 代理程式：

* 當您使用 Azure 入口網站和從 **Marketplace** 選取映像來建立 VM 的執行個體時，
* 當您使用 [New-AzureVM](https://msdn.microsoft.com/library/azure/dn495254.aspx) 或 [New-AzureQuickVM](https://msdn.microsoft.com/library/azure/dn495183.aspx) Cmdlet 來建立 VM 的執行個體時。 您可以藉由將 **–DisableGuestAgent** 參數新增至 [Add-AzureProvisioningConfig](https://msdn.microsoft.com/library/azure/dn495299.aspx) Cmdlet，來建立沒有 VM 代理程式的 VM。

* 當您在現有 VM 執行個體上手動下載並安裝 VM 代理程式時，將 **ProvisionGuestAgent** 值設定為 **true**。 您可以藉由使用 PowerShell 命令或 REST 呼叫，針對 Windows 和 Linux 代理程式使用這項技術。 (如果您以手動方式安裝 VM 代理程式之後未設定 **ProvisionGuestAgent** 值，則不會正確偵測到新增 VM 代理程式。)下列程式碼範例示範如何使用 PowerShell 執行此動作，其中 `$svc` 和 `$name` 引數都已經確定：

      $vm = Get-AzureVM –ServiceName $svc –Name $name
      $vm.VM.ProvisionGuestAgent = $TRUE
      Update-AzureVM –Name $name –VM $vm.VM –ServiceName $svc

* 當您建立 VM 映像時，會包含已安裝的 VM 代理程式。 一旦具有 VM 代理程式的映像存在，您可以將該映像上傳至 Azure。 若為 Windows VM，請下載 [Windows VM Agent.msi 檔案](http://go.microsoft.com/fwlink/?LinkID=394789) ，然後安裝 VM 代理程式。 若為 Linux VM，請從位於 <https://github.com/Azure/WALinuxAgent> 的 GitHub 存放庫安裝 VM 代理程式。 如需如何在 Linux 上安裝 VM 代理程式的詳細資訊，請參閱[Azure Linux VM 代理程式使用者指南](../articles/virtual-machines/extensions/agent-linux.md)。

> [!NOTE]
> 在 PaaS 中，VM 代理程式稱為 **WindowsAzureGuestAgent**，且在 Web 和背景工作角色 VM 中皆可使用。 (如需詳細資訊，請參閱 [Azure 角色架構](http://blogs.msdn.com/b/kwill/archive/2011/05/05/windows-azure-role-architecture.aspx))。角色 VM 的 VM 代理程式現已可將延伸模組加入雲端服務 VM，其方法與永續性虛擬機器相同。 角色 VM 和永續性 VM 上 VM 擴充功能之間最大的差異是新增 VM 擴充功能的時機。 使用角色 VM 時，擴充功能會先新增至雲端服務，然後新增至該雲端服務內的部署。
>
> 使用 [Get AzureServiceAvailableExtension](https://msdn.microsoft.com/library/azure/dn722498.aspx) Cmdlet，來列出所有可用的角色 VM 延伸模組。
>
>

## <a name="find-add-update-and-remove-vm-extensions"></a>尋找、加入、更新和移除 VM 擴充功能
如需這些工作的詳細資訊，請參閱[加入、尋找、更新及移除 Azure VM 延伸模組](../articles/virtual-machines/windows/classic/manage-extensions.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。
