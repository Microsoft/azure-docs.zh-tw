---
title: Azure 中 Linux VM 的概觀 | Microsoft Docs
description: 描述 Azure 計算、儲存體和網路服務與 Linux 虛擬機器。
services: virtual-machines-linux
documentationcenter: virtual-machines-linux
author: rickstercdn
manager: jeconnoc
editor: ''
ms.assetid: 7965a80f-ea24-4cc2-bc43-60b574101902
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: overview
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/29/2017
ms.author: rclaus
ms.custom: H1Hack27Feb2017, mvc
ms.openlocfilehash: 6ed1ef6019e339768c4ba6a62d8b4d99d844f254
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949516"
---
# <a name="azure-and-linux"></a>Azure 和 Linux
Microsoft Azure 集結了各種整合式公用雲端服務且數量不斷增加，包括分析、虛擬機器、資料庫、行動、網路、儲存體和 Web&mdash;因此很適合用來裝載您的解決方案。  Microsoft Azure 提供可調整的運算平台，可讓您在需要時用多少就付多少，而不需投資內部部署的硬體。  Azure 可供您將您的方案相應增加和放大至任何您需要的規模，以滿足您所服務之客戶的需求。

如果您熟悉 Amazon AWS 的各項功能，您可以查看 Azure 與 AWS 的對照 [定義對應文件](https://azure.microsoft.com/campaigns/azure-vs-aws/mapping/)。

## <a name="regions"></a>區域
Microsoft Azure 資源分散在世界各地的多個地理區域。  「區域」代表單一地理區域中的多個資料中心。 Azure 目前 (截至 2018 年 8 月為止) 已在全球 42 個區域正式推出，並即將在另外 12 個區域推出 - 全球適用的區域比任何其他雲端提供者都還要多。 在下列頁面可以找到最新的現有和全新推出的區域清單：

* [Azure 區域](https://azure.microsoft.com/regions/)

## <a name="availability"></a>可用性
Azure 已宣布推出業界領先的單一執行個體虛擬機器 99.9% 服務等級協定 (前提是您部署的所有磁碟都是進階儲存體的 VM)。  為了讓您的部署符合標準的 99.95% VM 服務等級協定資格，您還必須在可用性設定組內部署兩個或更多執行工作負載的 VM。 可用性設定組可確保您的 VM 會分散在 Azure 資料中心內的多個容錯網域，以及部署至具有不同維護期間的主機。 完整 [Azure SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) 說明保證的 Azure 整體可用性。

## <a name="managed-disks"></a>受控磁碟

受控磁碟可在背景中為您處理 Azure 儲存體帳戶的建立和管理作業，確保您不需要擔心儲存體帳戶的延展性限制。 您可指定磁碟大小和效能層級 (標準或進階)，而 Azure 會建立和管理磁碟。 當您新增磁碟或相應增加和減少 VM 時，都不必擔心所使用的儲存體。 如果您要建立新的 VM，請[使用 Azure CLI](quick-create-cli.md) 或 Azure 入口網站來建立具有受控 OS 和資料磁碟的 VM。 如果您有具備非受控磁碟的 VM，您可以[將 VM 轉換成由受控磁碟提供支援](convert-unmanaged-to-managed-disks.md)。

您也可以在每個 Azure 區域中使用單一儲存體帳戶管理自訂映像，並使用映像在相同訂用帳戶中建立數百個 VM。 如需受控磁碟的詳細資訊，請參閱 [Azure 受控磁碟概觀](../linux/managed-disks-overview.md)。

## <a name="azure-virtual-machines--instances"></a>Azure 虛擬機器和執行個體
Microsoft Azure 支援執行由多家合作夥伴提供和維護的眾多熱門 Linux 散發套件。  您可以在 Azure Marketplace 中找到 Red Hat Enterprise、CentOS、SUSE Linux Enterprise、Debian、Ubuntu、CoreOS、RancherOS 及 FreeBSD 等散發套件。 Microsoft 與各個 Linux 社群積極合作，以便為[經 Azure 背書的 Linux 散發套件](endorsed-distros.md)清單新增更多版本選項。

如果您慣用的 Linux 散發套件選項目前未出現在映像庫中，您可以[建立 Linux VHD 並上傳到 Azure](create-upload-generic.md)來「自備 Linux」VM。

Azure 虛擬機器可供您靈活部署各種運算方案。 您可以在大部分的作業系統 (Windows、Linux，或來自持續增加合作夥伴的自訂建立作業系統)，部署幾乎任何工作負載和任何語言。 還是沒找到您在尋找的映像嗎？  別擔心，您也可以使用您在內部部署中擁有的映像。

## <a name="vm-sizes"></a>VM 大小
您使用的 VM [大小](sizes.md)是由所您要執行的工作負載所決定。 您所選的大小會決定例如處理電源、記憶體和儲存體容量等因素。 Azure 提供了各種不同的大小，以支援許多類型的用法。

Azure 可依據 VM 的大小和作業系統，以[每小時](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)價格方式收費。 針對不足一小時的部分，Azure 只會收取已使用分鐘數的費用。 儲存體是個別定價與收費。

## <a name="automation"></a>自動化
為了實現適當的 DevOps 文化特性，所有基礎結構都必須是程式碼。  當所有基礎結構都留存在程式碼中時，便可以輕鬆地重新建立基礎結構 (Phoenix 伺服器)。  Azure 可以與所有主要的自動化工具 (例如 Ansible、Chef、SaltStack 及 Puppet) 搭配運作。  Azure 也有自己的自動化工具：

* [Azure 範本](create-ssh-secured-vm-from-template.md)
* [Azure VMAccess](using-vmaccess-extension.md)

Azure 將針對支援 [cloud-init](http://cloud-init.io/) 的大多數 Linux 散發套件推出 cloud-init 支援。  目前 Canonical 的 Ubuntu VM 部署預設即已啟用 cloud-init。  Red Hats RHEL、CentOS 及 Fedora 支援 cloud-init，不過由 Red Hat 維護的 Azure 映像目前並未安裝 cloud-init。  若要在 Red Hat 系列 OS 上使用 cloud-init，您必須建立已安裝 cloud-init 的自訂映像。

* [在 Azure Linux VM 上使用 cloud-init](using-cloud-init.md)

## <a name="quotas"></a>配額
每個 Azure 訂用帳戶都有適當的預設配額限制，而此限制會在您要為專案部署大量 VM 時產生影響。 每一訂用帳戶目前的限制是每一區域 20 個 VM。  只要提出支援票證來要求增加限制，即可快速且輕鬆地提高配額限制。  如需有關配額限制的詳細資料：

* [Azure 訂用帳戶服務限制](../../azure-subscription-service-limits.md)

## <a name="partners"></a>合作夥伴
Microsoft 與合作夥伴密切合作，以確保更新可用的映像並針對 Azure 執行階段進行最佳化。  如需 Azure 合作夥伴的詳細資訊，請參閱下列連結：

* [Azure 背書散發套件上的 Linux](endorsed-distros.md)
* SUSE - [Azure Marketplace - SUSE Linux Enterprise Server](https://azuremarketplace.microsoft.com/en-us/marketplace/apps?search=%27SUSE%27)
* Red Hat - [Azure Marketplace - Red Hat Enterprise Linux 7.2](https://azure.microsoft.com/marketplace/partners/redhat/redhatenterpriselinux72/)
* Canonical - [Azure Marketplace - Ubuntu Server 16.04 LTS](https://azure.microsoft.com/marketplace/partners/canonical/ubuntuserver1604lts/)
* Debian - [Azure Marketplace - Debian 8 "Jessie"](https://azure.microsoft.com/marketplace/partners/credativ/debian8/)
* FreeBSD - [Azure Marketplace - FreeBSD 10.3](https://azure.microsoft.com/marketplace/partners/microsoft/freebsd103/)
* CoreOS - [Azure Marketplace - CoreOS (Stable)](https://azure.microsoft.com/marketplace/partners/coreos/coreosstable/)
* RancherOS - [Azure Marketplace - RancherOS](https://azure.microsoft.com/marketplace/partners/rancher/rancheros/)
* Bitnami - [適用於 Azure 的 Bitnami 程式庫](https://azure.bitnami.com/)
* Mesosphere - [Azure Marketplace - Azure 上的 Mesosphere DC/OS](https://azure.microsoft.com/marketplace/partners/mesosphere/dcosdcos/)
* Docker - [Azure Marketplace - 與 Docker Swarm 搭配使用的 Azure Container Service](https://azure.microsoft.com/marketplace/partners/microsoft/acsswarms/)
* Jenkins - [Azure Marketplace - CloudBees Jenkins Platform](https://azure.microsoft.com/marketplace/partners/cloudbees/jenkins-platformjenkins-platform/)

## <a name="getting-started-with-linux-on-azure"></a>在 Azure 上開始使用 Linux
若要開始使用 Azure，您需要一個 Azure 帳戶、安裝 Azure CLI，以及一組 SSH 公開和私密金鑰。

### <a name="sign-up-for-an-account"></a>註冊帳戶
使用「Azure 雲端」的第一個步驟就是註冊 Azure 帳戶。  請前往 [Azure 帳戶註冊](https://azure.microsoft.com/pricing/free-trial/) 頁面來開始註冊。

### <a name="install-the-cli"></a>安裝 CLI
有了您的新 Azure 帳戶之後，您便可以立即開始使用 Azure 入口網站，這是一個 Web 型系統管理面板。  若要透過命令列管理 Azure 雲端，您需安裝 `azure-cli`。  請在您的 Mac 或 Linux 工作站上安裝 [Azure CLI ](/cli/azure/install-azure-cli)。

### <a name="create-an-ssh-key-pair"></a>建立 SSH 金鑰組
現在您已擁有 Azure 帳戶、Azure Web 入口網站及 Azure CLI。  下一個步驟是建立 SSH 金鑰組，使用此金鑰組即可透過 SSH 連線到 Linux，而不需使用密碼。  [在 Linux 和 Mac 上建立 SSH 金鑰](mac-create-ssh-keys.md)，以便啟用無密碼登入功能並提升安全性。

### <a name="create-a-vm-using-the-cli"></a>使用 CLI 來建立 VM
使用 CLI 來建立 Linux VM 是一個讓您不需離開正在工作的終端機即可快速部署 VM 的方法。  所有您可以在 Web 入口網站上指定的項目，透過命令列旗標或參數也都可以指定。  

* [使用 CLI 建立 Linux VM](quick-create-cli.md)

### <a name="create-a-vm-in-the-portal"></a>在入口網站中建立 VM
在 Azure Web 入口網站中建立 Linux VM 可讓您輕鬆點選各種選項來進行部署。  您將不使用命令列旗標或參數，而是會看到含有各種選項和設定的美觀 Web 版面配置。  所有您可以透過命令列介面使用的項目，在入口網站中也都可以使用。

* [使用入口網站建立 Linux VM](quick-create-portal.md)

### <a name="log-in-using-ssh-without-a-password"></a>使用 SSH 登入而不提供密碼
VM 現在已在 Azure 上執行，而您已經可以登入。  使用密碼透過 SSH 登入既不安全又費時。  使用 SSH 金鑰是最安全的登入方式，也是最省時的登入方式。  當您透過入口網站或 CLI 建立 Linux VM 時，有兩種驗證選擇。  如果您為 SSH 選擇了密碼，Azure 就會將 VM 設定成允許透過密碼登入。  如果您選擇使用 SSH 公開金鑰，Azure 就會將 VM 設定成只允許透過 SSH 金鑰登入，並停用密碼登入。 若要藉由只允許使用 SSH 金鑰登入的方式來保護 Linux VM，請在透過入口網站或 CLI 建立 VM 時，使用 SSH 公用金鑰選項。

## <a name="related-azure-components"></a>相關的 Azure 元件
## <a name="storage"></a>儲存體
* [Microsoft Azure 儲存體簡介](../../storage/common/storage-introduction.md)
* [使用 azure-cli 將磁碟新增到 Linux VM](add-disk.md)
* [如何在 Azure 入口網站中將資料磁碟連結到 Linux VM](attach-disk-portal.md)

## <a name="networking"></a>網路功能
* [虛擬網路概觀](../../virtual-network/virtual-networks-overview.md)
* [Azure 中的 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)
* [對 Azure 中的 Linux VM 開啟連接埠](nsg-quickstart.md)
* [在 Azure 入口網站中建立完整格式的網域名稱](portal-create-fqdn.md)

## <a name="containers"></a>容器
* [Azure 中的虛擬機器和容器](containers.md)
* [Azure 容器服務簡介](../../container-service/container-service-intro.md)
* [部署 Azure 容器服務叢集](../../container-service/dcos-swarm/container-service-deployment.md)

## <a name="next-steps"></a>後續步驟
您現在已概略了解 Azure 上的 Linux。  下一個步驟是深入了解並建立一些 VM！

* [探索透過 AzureCLI 執行一般工作的指令碼範例清單 (不斷增加中)](cli-samples.md)
