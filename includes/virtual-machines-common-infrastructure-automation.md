# <a name="use-infrastructure-automation-tools-with-virtual-machines-in-azure"></a>在 Azure 中使用基礎結構自動化工具搭配虛擬機器
若要以一致的方式大規模建立並管理 Azure 虛擬機器 (VM)，通常需要某種形式的自動化。 有許多工具和解決方案可讓您將完整的 Azure 基礎結構部署及管理生命週期自動化。 本文介紹的一些基礎結構自動化工具可供您在 Azure 中使用。 這些工具通常符合下列其中一種方法：

- 自動設定 VM
    - 工具包括 [Ansible](#ansible)、[Chef](#chef) 和 [Puppet](#puppet)。
    - VM 自訂的特定工具包括 [cloud-init](#cloud-init) (適用於 Linux VM)、[PowerShell 預期狀態設定 (DSC)](#powershell-dsc)，和 [Azure 自訂指令碼擴充功能](#azure-custom-script-extension) (適用於所有 Azure VM)。
 
- 將基礎結構管理自動化
    - 工具包括可將自訂 VM 映像建置自動化的 [Packer](#packer) 和可將基礎結構建置程序自動化的 [Terraform](#terraform)。
    - [Azure 自動化](#azure-automation)可在整個 Azure 和內部部署基礎結構內執行動作。

- 將應用程式部署和傳遞自動化
    - 範例包括 [Azure DevOps Services](#visual-studio-team-services) 和 [Jenkins](#jenkins)。

## <a name="ansible"></a>Ansible
[Ansible](https://www.ansible.com/) 是設定管理、建立 VM 或應用程式部署的自動化引擎。 Ansible 會使用無代理程式的模型，通常是使用 SSH 金鑰來驗證及管理目標電腦。 設定工作會定義在劇本中，並有幾個可用來執行特定工作的 Ansible 模組。 如需詳細資訊，請參閱 [Ansible 的運作方式](https://www.ansible.com/how-ansible-works)。

了解如何：

- [在 Linux 上安裝及設定 Ansible 來與 Azure 搭配使用](../articles/virtual-machines/linux/ansible-install-configure.md)。
- [建立 Linux 虛擬機器](../articles/virtual-machines/linux/ansible-create-vm.md)。
- [管理 Linux 虛擬機器](../articles/virtual-machines/linux/ansible-manage-linux-vm.md)。


## <a name="chef"></a>Chef
[Chef](https://www.chef.io/) 是自動化平台，可協助定義您基礎結構的設定、部署和管理方式。 其他元件包含應用程式生命週期自動化而非基礎結構的 Chef Habitat，以及可協助自動化與安全性和原則需求相容的 Chef InSpec。 Chef 用戶端是安裝在目標電腦上，包含一或多個中央 Chef 伺服器，可儲存和管理設定。 如需詳細資訊，請參閱 [Chef 概觀](https://docs.chef.io/chef_overview.html)。

了解如何：

- [從 Azure Marketplace 中部署 Chef 自動化](https://azuremarketplace.microsoft.com/marketplace/apps/chef-software.chef-automate?tab=Overview)。
- [在 Windows 上安裝 Chef 並建立 Azure VM](../articles/virtual-machines/windows/chef-automation.md)。


## <a name="puppet"></a>Puppet
[Puppet](https://www.puppet.com) 是符合企業需求的自動化平台，可處理應用程式傳遞和部署程序。 代理程式會安裝在目標電腦上，從而允許 Puppet Master 執行資訊清單，以定義 Azure 基礎結構和 VM 所需的設定。 Puppet 能夠與諸如 Jenkins 和 GitHub 等其他解決方案整合，以獲得提升的 DevOps 工作流程。 如需詳細資訊，請參閱 [Puppet 的運作方式](https://puppet.com/product/how-puppet-works)。

了解如何：

- [從 Azure Marketplace 部署 Puppet ](https://azuremarketplace.microsoft.com/marketplace/apps/puppet.puppet-enterprise-2017-2?tab=Overview)。


## <a name="cloud-init"></a>Cloud-init
[Cloud-init (英文)](https://cloudinit.readthedocs.io) 是在 Linux VM 初次開機時，廣泛用來自訂它們的方法。 您可以使用 cloud-init 來安裝封裝和寫入檔案，或者設定使用者和安全性。 因為在初次開機程序期間時會呼叫 Cloud-init，因此不需要使用任何額外的步驟或必要的代理程式，就可以套用您的設定。  如需如何正確地設定 `#cloud-config` 檔案格式的詳細資訊，請參閱 [cloud-init 文件網站](http://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data) \(英文\)。  `#cloud-config` 檔案是以 base64 編碼的文字檔。

Cloud-init 也適用於散發套件。 例如，您不使用 **apt-get install** 或 **yum install** 來安裝套件。 您可以改為定義要安裝的套件清單。 Cloud-init 會針對您選取的散發套件自動使用原生的套件管理工具。

 我們一直積極地與背書的 Linux 發行版本合作夥伴合作，以便在 Azure Marketplace 中提供支援 Cloud-init 的映像。 這些映像會讓您的 Cloud-init 部署和設定順暢地與 VM 和虛擬機器擴展集運作。 下表概述目前支援 cloud-init 的映像在 Azure 平台上的可用性：

| 發行者 | 供應項目 | SKU | 版本 | cloud-init 就緒
|:--- |:--- |:--- |:--- |:--- |:--- |
|Canonical |UbuntuServer |16.04-LTS |最新 |是 | 
|Canonical |UbuntuServer |14.04.5-LTS |最新 |是 |
|CoreOS |CoreOS |Stable |最新 |是 |
|OpenLogic |CentOS |7-CI |最新 |preview |
|RedHat |RHEL |7-RAW-CI |最新 |preview |

在 Azure 上深入了解 Cloud-init 的詳細資料：

- [Azure 中 Linux 虛擬機器的 Cloud-init 支援](../articles/virtual-machines/linux/using-cloud-init.md)
- [請使用 Cloud-init 嘗試自動化 VM 組態上的教學課程](../articles/virtual-machines/linux/tutorial-automate-vm-deployment.md)。


## <a name="powershell-dsc"></a>PowerShell DSC
[PowerShell 預期狀態設定 (DSC)](https://msdn.microsoft.com/powershell/dsc/overview) 是一個管理平台，可定義目標電腦的設定。 也可透過 [Open Management Infrastructure (OMI) 伺服器](https://collaboration.opengroup.org/omi/)在 Linux 上使用 DSC。

DSC 設定會定義要在電腦上安裝的項目，以及設定主機的方式。 本機設定管理員 (LCM) 引擎會在每個目標節點上執行，這些節點會以推送組態作為基礎來處理要求的動作。 提取伺服器是一項在中央主機上執行的 web 服務，可儲存 DSC 設定和相關聯的資源。 提取伺服器會與每個目標主機上的 LCM 引擎通訊，以提供必要的合規性相關設定和報告。

了解如何：

- [建立基本的 DSC 設定](https://msdn.microsoft.com/powershell/dsc/quickstart)。
- [設定 DSC 提取伺服器](https://msdn.microsoft.com/powershell/dsc/pullserver)。
- [使用 DSC for Linux](https://msdn.microsoft.com/powershell/dsc/lnxgettingstarted)。


## <a name="azure-custom-script-extension"></a>Azure 自訂指令碼擴充功能
適用於 [Linux](../articles/virtual-machines/linux/extensions-customscript.md) 或 [Windows](../articles/virtual-machines/windows/extensions-customscript.md) 的 Azure 自訂指令碼擴充功能會在 Azure VM 上下載並執行指令碼。 當您建立 VM 時，或在 VM 為使用中狀態之後，可以使用擴充功能。 

您可以從 Azure 儲存體或 GitHub 存放庫等任何公用位置下載指令碼。 您可以透過自訂指令碼擴充功能，以來源 VM 上執行的任何語言撰寫指令碼。 這些指令碼可用來安裝應用程式，或視需要設定 VM。 若要保護認證，可將密碼等機密資訊儲存在受保護的設定中。 這些認證只會在 VM 內進行解密。

了解如何：

- [使用 Azure CLI 建立 Linux VM，並使用自訂指令碼擴充功能](../articles/virtual-machines/scripts/virtual-machines-linux-cli-sample-create-vm-nginx.md?toc=%2fcli%2fazure%2ftoc.json)。
- [使用 Azure PowerShell 建立 Windows VM，並使用自訂指令碼擴充功能](../articles/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm-iis.md?toc=%2fpowershell%2fmodule%2ftoc.json)。


## <a name="packer"></a>Packer
當您在 Azure 中建立自訂的 VM 映像時，[Packer](https://www.packer.io) 會將建置程序自動化。 您可以使用 Packer 來定義作業系統，並執行後置設定指令碼，針對您的特定需求來自訂 VM。 一旦設定之後，就會擷取 VM 作為受控磁碟映像。 Packer 會將程序自動化，以建立來源 VM、網路和儲存體資源、執行設定指令碼，然後建立 VM 映像。

了解如何：

- [在 Azure 中使用 Packer 建立 Linux VM 映像](../articles/virtual-machines/linux/build-image-with-packer.md)。
- [在 Azure 中使用 Packer 建立 Windows VM 映像](../articles/virtual-machines/windows/build-image-with-packer.md)。


## <a name="terraform"></a>Terraform
[Terraform](https://www.terraform.io) 是自動化工具，可讓您使用單一範本格式語言 - HashiCorp 設定語言 (HCL) 來定義和建立完整的 Azure 基礎結構。 您可以使用 Terraform 來定義範本，將建立網路、儲存和指定應用程式解決方案的 VM 資源程序自動化。 您可以使用其他平台的現有 Terraform 範本搭配 Azure，來確保一致性並簡化基礎結構部署，而不需要轉換成 Azure Resource Manager 範本。

了解如何：

- [使用 Azure 安裝及設定 Terraform](../articles/virtual-machines/linux/terraform-install-configure.md)。
- [使用 Terraform 建立 Azure 基礎結構](../articles/virtual-machines/linux/terraform-create-complete-vm.md)。


## <a name="azure-automation"></a>Azure 自動化
[Azure 自動化](https://azure.microsoft.com/services/automation/)會使用 Runbook，在您的目標 VM 上處理一組工作。 Azure 自動化是用來管理現有的 VM，而非建立基礎結構。 Azure 自動化可以在 Linux 和 Windows VM 上執行，以及在具有混合式 Runbook 背景工作角色的內部部署虛擬或實體機器上執行。 Runbook 可以儲存在原始檔控制存放庫 (例如 GitHub) 中。 接著，這些 Runbook 就可以手動方式或依定義的排程執行。

Azure 自動化也會提供預期狀態設定 (DSC) 服務，可讓您建立一組指定 VM 之設定方式的定義。 然後，DSC 會確保已套用所需的設定，且 VM 保持一致。 Azure 自動化 DSC 會在 Windows 和 Linux 電腦上執行。

了解如何：

- [建立 PowerShell Runbook](../articles/automation/automation-first-runbook-textual-powershell.md)。
- [使用混合式 Runbook 背景工作角色來管理內部部署資源](../articles/automation/automation-hybrid-runbook-worker.md)。
- [使用 Azure Automation DSC](../articles/automation/automation-dsc-getting-started.md)。


## <a name="azure-devops-services"></a>Azure DevOps Services
[Azure DevOps Services](https://www.visualstudio.com/team-services/) 是一套工具，能協助您共用和追蹤程式碼、使用自動化的組建，並建立完整的持續整合及開發 (CI/CD) 管線。 Azure DevOps Services 會與 Visual Studio 和其他編輯器整合，以簡化使用方式。 Azure DevOps Services 還可以建立及設定 Azure VM，然後再將程式碼部署到這些 VM。

了解如何：

- [使用 Azure DevOps Services 建立持續整合管線](../articles/virtual-machines/windows/tutorial-vsts-iis-cicd.md)。


## <a name="jenkins"></a>Jenkins
[Jenkins](https://www.jenkins.io) 是持續整合伺服器，能協助部署及測試應用程式，並建立自動管線以供程式碼傳遞。 有數百個外掛程式可擴充核心 Jenkins 平台，您也可以透過 Webhook 與許多其他產品和解決方案進行整合。 您可以手動方式在 Azure VM 上安裝 Jenkins、從 Docker 容器內執行 Jenkins，或使用預先建置的 Azure Marketplace 映像。

了解如何：

- [在 Azure 中的 Linux VM 上以 Jenkins、GitHub 及 Docker 建立開發基礎結構](../articles/virtual-machines/linux/tutorial-jenkins-github-docker-cicd.md)。


## <a name="next-steps"></a>後續步驟
有許多不同的選項可供您在 Azure 中使用基礎結構自動化工具。 您可以自由使用最適合您需求和環境的解決方案。 若要開始使用並嘗試一些內建於 Azure 的工具，請參閱如何將自訂 [Linux](../articles/virtual-machines/linux/tutorial-automate-vm-deployment.md) 或 [Windows](../articles/virtual-machines/windows/tutorial-automate-vm-deployment.md) VM 自動化。
