您必須具有 Azure 的帳戶，才能搭配使用 Azure CLI 與 Resource Manager 命令和範本，使用資源群組來部署 Azure 資源和工作負載。 如果您沒有帳戶，可以取得 [此處免費試用的 Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)。

如果您尚未安裝 Azure CLI 並且連接至您的訂用帳戶，請參閱[安裝 Azure CLI](../articles/cli-install-nodejs.md)，將模式設定為 `arm` 與 `azure config mode arm`，並連接至 Azure 與 `azure login` 命令。

## <a name="cli-versions-to-complete-the-task"></a>用以完成工作的 CLI 版本
您可以使用下列其中一個 CLI 版本來完成工作︰

- Azure 傳統 CLI：適用於傳統與資源管理部署模型的 CLI (此文章)
- [Azure CLI](../articles/virtual-machines/linux/cli-manage.md)：適用於資源管理部署模型的新一代 CLI

## <a name="basic-azure-resource-manager-commands-in-azure-classic-cli"></a>Azure 傳統 CLI 中的基本 Azure Resource Manager 命令

此文章涵蓋搭配 Azure 傳統 CLI 來管理 Azure 訂用帳戶中之資源 (主要是 VM) 並與其互動的基本命令。  如需特定命令列參數和選項的詳細說明，您可以輸入 `azure <command> <subcommand> --help` 或 `azure help <command> <subcommand>` 來使用線上命令說明和選項。

> [!NOTE]
> 這些範例不包含以範本為基礎的作業，這些作業一般建議在資源管理員中針對 VM 部署使用。 如需資訊，請參閱[搭配 Azure Resource Manager 使用 Azure CLI](../articles/xplat-cli-azure-resource-manager.md) 和[使用 Azure Resource Manager 範本和 Azure CLI 部署與管理虛擬機器](../articles/virtual-machines/linux/create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
> 
> 

| Task | Resource Manager |
| --- | --- | --- |
| 建立最基本的 VM |`azure vm quick-create [options] <resource-group> <name> <location> <os-type> <image-urn> <admin-username> <admin-password>`<br/><br/>(從 `azure vm image list` 命令取得 `image-urn`。 如需範例，請參閱[這篇文章](../articles/virtual-machines/linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。) |
| 建立 Linux VM |`azure  vm create [options] <resource-group> <name> <location> -y "Linux"` |
| 建立 Windows VM |`azure  vm create [options] <resource-group> <name> <location> -y "Windows"` |
| 列出 VM |`azure  vm list [options]` |
| 取得 VM 的相關資訊 |`azure  vm show [options] <resource_group> <name>` |
| 啟動 VM |`azure vm start [options] <resource_group> <name>` |
| 停止 VM |`azure vm stop [options] <resource_group> <name>` |
| 解除配置 VM |`azure vm deallocate [options] <resource-group> <name>` |
| 重新啟動 VM |`azure vm restart [options] <resource_group> <name>` |
| 刪除 VM |`azure vm delete [options] <resource_group> <name>` |
| 擷取 VM |`azure vm capture [options] <resource_group> <name>` |
| 從使用者映像建立 VM |`azure  vm create [options] –q <image-name> <resource-group> <name> <location> <os-type>` |
| 從特殊化磁碟建立 VM |`azue  vm create [options] –d <os-disk-vhd> <resource-group> <name> <location> <os-type>` |
| 將資料磁碟新增至 VM |`azure  vm disk attach-new [options] <resource-group> <vm-name> <size-in-gb> [vhd-name]` |
| 從 VM 移除資料磁碟 |`azure  vm disk detach [options] <resource-group> <vm-name> <lun>` |
| 將泛型延伸模組新增至 VM |`azure  vm extension set [options] <resource-group> <vm-name> <name> <publisher-name> <version>` |
| 將 VM 存取延伸模組新增至 VM |`azure vm reset-access [options] <resource-group> <name>` |
| 將 Docker 延伸模組新增至 VM |`azure  vm docker create [options] <resource-group> <name> <location> <os-type>` |
| 移除 VM 延伸模組 |`azure  vm extension set [options] –u <resource-group> <vm-name> <name> <publisher-name> <version>` |
| 取得 VM 資源的使用量 |`azure vm list-usage [options] <location>` |
| 取得所有可用的 VM 大小 |`azure vm sizes [options]` |

## <a name="next-steps"></a>後續步驟
* 如需超越基本 VM 管理的其他 CLI 命令範例，請參閱 [Azure CLI 與 Azure Resource Manager 搭配使用](../articles/virtual-machines/azure-cli-arm-commands.md)。
