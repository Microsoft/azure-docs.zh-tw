---
title: Windows 虛擬機器概觀 - Azure | Microsoft Docs
description: 了解在 Azure 中建立及管理 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: fbae9c8e-2341-4ed0-bb20-fd4debb2f9ca
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/04/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 6ec151222bda3d87386cc3be4c54821775880795
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816832"
---
# <a name="overview-of-windows-virtual-machines-in-azure"></a>Azure 中的 Windows 虛擬機器概觀

Azure 虛擬機器 (VM) 是由 Azure 所提供的[隨選且可調整的數種運算資源](../../app-service/choose-web-site-cloud-service-vm.md)類型之一。 一般而言，當您對於運算環境所需的控制權比其他選擇可提供的還要多時，則您會選擇 VM。 本文提供您在建立 VM 之前應該的事項、建立方式及管理方式的相關資訊。

Azure VM 讓您能夠有彈性地進行虛擬化，而不需購買並維護執行它的實體硬體。 不過，您仍然需要執行工作來維護 VM，例如設定、修補和安裝在 VM 上執行的軟體。

Azure 虛擬機器可用於許多用途。 部分範例如下：

* **開發和測試** – Azure VM 提供了一個快速且簡單的方法，用以建立為應用程式撰寫程式碼並進行測試時所需之特定組態的電腦。
* **雲端中的應用程式** – 因為您應用程式的需求可能會變動，在 Azure VM 上執行它在經濟上是合理的。 當您需要 VM 時便支付額外的 VM，而當您不需要時便關閉這些 VM。
* **擴充的資料中心** – 可輕鬆將 Azure 虛擬網路中的虛擬機器連線到您組織的網路。

您的應用程式所使用的 VM 數目可以相應增加及相應放大為符合您需求的任何內容。

## <a name="what-do-i-need-to-think-about-before-creating-a-vm"></a>我在建立 VM 之前需要先考慮什麼？
當您在 Azure 中建置應用程式基礎結構時，總是會有許多[設計考量](/azure/architecture/reference-architectures/virtual-machines-windows?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 在您開始之前，仔細考量 VM 的這些層面很重要︰

* 應用程式資源的名稱
* 將儲存資源的位置
* VM 的大小
* 可建立的 VM 數目上限
* VM 上執行的作業系統
* VM 啟動後的設定
* VM 需要的相關資源

### <a name="naming"></a>命名
虛擬機器指派一個[名稱](/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)給它，而它設定一個電腦名稱作為作業系統的一部分。 VM 的名稱最多可為 15 個字元。

如果您使用 Azure 來建立作業系統磁碟，則電腦名稱和虛擬機器名稱為相同。 如果您[上傳並使用您自己的映像](upload-generalized-managed.md) (該映像包含先前所設定的作業系統)，並用它來建立虛擬機器，則名稱可能會不同。 我們建議您，當您上傳自己的映像檔時，將作業系統中的電腦名稱設為與虛擬機器的名稱相同。

### <a name="locations"></a>位置
Azure 中所建立的所有資源分散在世界各地的多個[地理區域](https://azure.microsoft.com/regions/)。 通常，當您建立 VM 時，區域稱為**位置**。 針對 VM，位置會指定虛擬硬碟所儲存的位置。

下表顯示一些您可以取得可用位置清單的方式。

| 方法 | 說明 |
| --- | --- |
| Azure 入口網站 |當您建立 VM 時，請從清單中選取位置。 |
| Azure PowerShell |使用 [Get-AzureRmLocation](/powershell/module/azurerm.resources/get-azurermlocation) 命令。 |
| REST API |使用[列出位置](https://docs.microsoft.com/rest/api/resources/subscriptions#Subscriptions_ListLocations)作業。 |
| Azure CLI |使用 [az account list-locations](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az_account_list_locations) 作業。 |

### <a name="vm-size"></a>VM 大小
您使用的 VM [大小](sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)是由所您要執行的工作負載所決定。 您所選的大小會決定例如處理電源、記憶體和儲存體容量等因素。 Azure 提供了各種不同的大小，以支援許多類型的用法。

Azure 可依據 VM 的大小和作業系統，以[每小時](https://azure.microsoft.com/pricing/details/virtual-machines/windows/)價格方式收費。 針對不足一小時的部分，Azure 只會收取已使用分鐘數的費用。 儲存體是個別定價與收費。

### <a name="vm-limits"></a>VM 限制
您的訂用帳戶都有預設[配額限制](../../azure-subscription-service-limits.md)，而此限制會在您要部署多個 VM 以供專案使用時造成影響。 每一訂用帳戶目前的限制是每一區域 20 個 VM。 只要[提出支援票證來要求增加](../../azure-supportability/resource-manager-core-quotas-request.md)，即可提高配額限制

### <a name="operating-system-disks-and-images"></a>作業系統磁碟和映像
虛擬機器是使用[虛擬硬碟 (VHD)](about-disks-and-vhds.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 來儲存其作業系統 (OS) 和資料。 VHD 也能夠使用於您可以選擇用來安裝 OS 的映像。 

Azure 提供許多 [Marketplace 映像](https://azure.microsoft.com/marketplace/virtual-machines/)來與不同版本和類型的 Windows Server 作業系統搭配使用。 Marketplace 映像是依映像發行者、供應項目、SKU 和版本 (版本通常會指定為最新版本) 來識別。 僅支援 64 位元作業系統。 如需所支援客體作業系統、角色和功能的詳細資訊，請參閱 [Microsoft Azure 虛擬機器的 Microsoft 伺服器軟體支援](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)。

下表顯示您可找到映像資訊的一些方法。

| 方法 | 說明 |
| --- | --- |
| Azure 入口網站 |當您選取要使用的影像時，會自動為您指定值。 |
| Azure PowerShell |[Get-AzureRMVMImagePublisher](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmimagepublisher) -Location *location*<BR>[Get-AzureRMVMImageOffer](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmimageoffer) -Location *location* -Publisher *publisherName*<BR>[Get-AzureRMVMImageSku](/powershell/module/azurerm.compute/get-azurermvmimagesku) -Location *location* -Publisher *publisherName* -Offer *offerName* |
| REST API |[列出映像發行者](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publishers)<BR>[列出映像供應項目](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publisher-offers)<BR>[列出映像 SKU](https://docs.microsoft.com/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus) |
| Azure CLI |[az vm image list-publishers](https://docs.microsoft.com/cli/azure/vm/image?view=azure-cli-latest#az_vm_image_list_publishers) --location *location*<BR>[az vm image list-offers](https://docs.microsoft.com/cli/azure/vm/image?view=azure-cli-latest#az_vm_image_list_offers) --location *location* --publisher *publisherName*<BR>[az vm image list-skus](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az_vm_list_skus) --location *location* --publisher *publisherName* --offer *offerName*|

您可以選擇[上傳並使用您自己的映像](upload-generalized-managed.md#upload-the-vhd-to-your-storage-account)，當您這麼做時，不會使用發行者名稱、供應項目和 SKU。

### <a name="extensions"></a>擴充功能
VM [擴充](extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)會透過 post 部署設定及自動化工作為您的 VM 提供其他功能。

可以使用擴充功能來完成這些常見工作︰

* **執行自訂指令碼** – [自訂指令碼延伸模組](extensions-customscript.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)可藉由當佈建 VM 時執行您的指令碼來協助您在 VM 上設定工作負載。
* **部署和管理組態** – [PowerShell 期望狀態組態 (DSC) 延伸模組](extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)可協助您設定 VM 上的 DSC 來管理組態和環境。
* **收集診斷資料** – [Azure 診斷擴充功能](extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)可協助您設定 VM 來收集診斷資料，用來監視您應用程式的健全狀況。

### <a name="related-resources"></a>相關資源
此資料表中的資源可供 VM 使用，且建立 VM 時必須存在或建立。

| 資源 | 必要 | 說明 |
| --- | --- | --- |
| [資源群組](../../azure-resource-manager/resource-group-overview.md) |是 |VM 必須包含在資源群組中。 |
| [儲存體帳戶](../../storage/common/storage-create-storage-account.md) |是 |VM 需要儲存體帳戶儲存其虛擬硬碟。 |
| [虛擬網路](../../virtual-network/virtual-networks-overview.md) |是 |VM 必須是虛擬網路的成員。 |
| [公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) |否 |可以有公用 IP 位址指派給 VM，以從遠端存取它。 |
| [網路介面](../../virtual-network/virtual-network-network-interface.md) |是 |VM 需要網路介面以在網路中進行通訊。 |
| [資料磁碟](attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |否 |VM 可以包含資料磁碟來擴充儲存體功能。 |

## <a name="how-do-i-create-my-first-vm"></a>如何建立第一個 VM？
有幾個選擇可供您建立 VM。 您所做的選擇取決於您所在的環境。 

下表提供協助您開始建立 VM 的資訊。

| 方法 | 文章 |
| --- | --- |
| Azure 入口網站 |[使用入口網站建立執行 Windows 的虛擬機器](../virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| 範本 |[利用 Resource Manager 範本建立 Windows 虛擬機器](ps-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| Azure PowerShell |[使用 PowerShell 建立 Windows VM](../virtual-machines-windows-ps-create.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| 用戶端 SDK |[使用 C# 部署 Azure 資源](csharp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |
| REST API |[建立或更新 VM](https://docs.microsoft.com/rest/api/compute/virtualmachines/virtualmachines-create-or-update) |
| Azure CLI |[使用 Azure CLI 建立 VM](https://docs.microsoft.com/azure/virtual-machines/scripts/virtual-machines-windows-cli-sample-create-vm) |

您希望它絕對不會發生，但偶爾會發生錯誤。 如果您發生這種情況，請查看[疑難排解在 Azure 中建立 Windows 虛擬機器時發生的 Resource Manager 部署問題](../troubleshooting/troubleshoot-deployment-new-vm-windows.md)中的資訊。

## <a name="how-do-i-manage-the-vm-that-i-created"></a>如何管理我所建立的 VM？
可以使用以瀏覽器為基礎的入口網站、支援指令碼處理的命令列工具，或直接透過 API 管理 VM。 您可能會執行的一些一般管理工作為取得 VM 的相關資訊、登入 VM、管理可用性，以及進行備份。

### <a name="get-information-about-a-vm"></a>取得 VM 的相關資訊
下表顯示您可以取得 VM 相關資訊的一些方式。

| 方法 | 說明 |
| --- | --- |
| Azure 入口網站 |在 [中樞] 功能表中，按一下 [虛擬機器]，然後從清單中選取 VM。 在 VM 的刀鋒視窗上，您可存取概觀資訊、設定值，並監視度量。 |
| Azure PowerShell |如需使用 PowerShell 來管理 VM 的相關資訊，請參閱[使用 Azure PowerShell 模組來建立和管理 Windows VM](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 |
| REST API |使用[取得 VM 資訊](https://docs.microsoft.com/rest/api/compute/virtualmachines/virtualmachines-get)作業來取得 VM 的相關資訊。 |
| 用戶端 SDK |如需使用 C# 來管理 VM 的相關資訊，請參閱[使用 Azure Resource Manager 和 C# 來管理 Azure 虛擬機器](csharp-manage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 |
| Azure CLI |如需使用 Azure CLI 管理 VM 的詳細資訊，請參閱 [Azure CLI 參考](https://docs.microsoft.com/cli/azure/vm)。 |

### <a name="log-on-to-the-vm"></a>登入 VM
您使用 Azure 入口網站中的 [連線] 按鈕[啟動遠端桌面 (RDP) 工作階段](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 嘗試使用遠端連線時，有時可能會出錯。 如果您發生這種情況，請參閱[疑難排解遠端桌面連線至執行 Windows 的 Azure 虛擬機器](../troubleshooting/troubleshoot-rdp-connection.md)中的說明資訊。

### <a name="manage-availability"></a>管理可用性
請務必了解如何[確保應用程式的高可用性](manage-availability.md)。 此組態需要建立多個 VM 以確保至少有一個 VM 正在執行。

為了讓您的部署符合 99.95 的 VM 服務等級協定資格，您必須部署兩個或更多在[可用性設定組](tutorial-availability-sets.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)內執行工作負載的 VM。 此組態可確保您的 VM 會分散多個容錯網域，且部署至具有不同維護期間的主機。 完整 [Azure SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) 說明保證的 Azure 整體可用性。

### <a name="back-up-the-vm"></a>備份 VM
[復原服務保存庫](../../backup/backup-introduction-to-azure-backup.md)可用來保護 Azure 備份和 Azure Site Recovery 服務中的資料和資產。 您可以使用復原服務保存庫，[使用 PowerShell 部署和管理 Resource Manager 部署之 VM 的備份](../../backup/backup-azure-vms-automation.md)。 

## <a name="next-steps"></a>後續步驟
* 如果您的目的是要使用 Linux VM，請查看 [Azure 與 Linux](../linux/overview.md)。
* 在[範例 Azure 基礎結構逐步解說](infrastructure-example.md)中深入了解設定基礎結構的指導方針。
