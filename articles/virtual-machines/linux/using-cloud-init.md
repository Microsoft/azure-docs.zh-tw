---
title: Azure 中 Linux 虛擬機器的 cloud-init 支援概觀 | Microsoft Docs
description: Microsoft Azure 中的 cloud-init 功能概觀
services: virtual-machines-linux
documentationcenter: ''
author: rickstercdn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 195c22cd-4629-4582-9ee3-9749493f1d72
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 0f7660e8534a74eabe32611c4c01ae5587af7cee
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43188868"
---
# <a name="cloud-init-support-for-virtual-machines-in-azure"></a>Azure 中虛擬機器的 Cloud-init 支援
本文說明針對 [cloud-init](https://cloudinit.readthedocs.io) 存在的支援，以便在 Azure 中佈建時，設定虛擬機器 (VM) 或虛擬機器擴展集 (VMSS)。 一旦 Azure 佈建資源，這些 cloud-init 指令碼就會在初次開機時執行。  

## <a name="cloud-init-overview"></a>Cloud-Init 概觀
[Cloud-init (英文)](https://cloudinit.readthedocs.io) 是在 Linux VM 初次開機時，廣泛用來自訂它們的方法。 您可以使用 cloud-init 來安裝封裝和寫入檔案，或者設定使用者和安全性。 因為在初次開機程序期間時會呼叫 Cloud-init，因此不需要使用任何額外的步驟或必要的代理程式，就可以套用您的設定。  如需如何正確地設定 `#cloud-config` 檔案格式的詳細資訊，請參閱 [cloud-init 文件網站](http://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data) \(英文\)。  `#cloud-config` 檔案是以 base64 編碼的文字檔。

Cloud-init 也適用於散發套件。 例如，您不使用 **apt-get install** 或 **yum install** 來安裝套件。 您可以改為定義要安裝的套件清單。 Cloud-init 會針對您選取的散發套件自動使用原生的套件管理工具。

 我們一直積極地與背書的 Linux 發行版本合作夥伴合作，以便在 Azure Marketplace 中提供支援 Cloud-init 的映像。 這些映像會讓您的 Cloud-init 部署和設定順暢地與 VM 和 VM 擴展集 (VMSS) 運作。 下表概述目前支援 cloud-init 的映像在 Azure 平台上的可用性：

| 發行者 | 供應項目 | SKU | 版本 | cloud-init 就緒 |
|:--- |:--- |:--- |:--- |:--- |
|Canonical |UbuntuServer |18.04-LTS |最新 |是 | 
|Canonical |UbuntuServer |17.10 |最新 |是 | 
|Canonical |UbuntuServer |16.04-LTS |最新 |是 | 
|Canonical |UbuntuServer |14.04.5-LTS |最新 |是 |
|CoreOS |CoreOS |Stable |最新 |是 |
|OpenLogic |CentOS |7-CI |最新 |preview |
|RedHat |RHEL |7-RAW-CI |最新 |preview |

目前 Azure Stack 不支援使用 cloud-init 來佈建 RHEL 7.4 和 CentOS 7.4。

## <a name="what-is-the-difference-between-cloud-init-and-the-linux-agent-wala"></a>cloud-init 和 Linux 代理程式 (WALA) 之間有哪些差異？
WALA 是 Azure 平台專屬的代理程式，用於佈建及設定 VM，並處理 Azure 擴充功能。 我們一直在加強設定 VM 使用 cloud-init 代替 Linux 代理程式的工作，以讓現有的 cloud-init 客戶使用其目前的 cloud-init 指令碼。  如果您已經對設定 Linux 系統所使用的 cloud-init 指令碼有所投入，則**不需要其他任何設定**，就可以將其啟用。 

如果您在佈建時未包含 Azure CLI 參數 `--custom-data`，WALA 會採用所需的最低限度 VM 佈建參數，來佈建 VM 並使用預設值完成部署。  如果您參考 cloud-init `--custom-data` 參數，自訂資料 (個別設定或完整指令碼) 中包含的任何內容都會覆寫 WALA 預設值。 

VM 的 WALA 設定有時間限制，必須在 VM 佈建時間上限內運作。  套用到 VM 的 Cloud-init 設定沒有時間限制，且不會因為逾時而導致部署失敗。 

## <a name="deploying-a-cloud-init-enabled-virtual-machine"></a>部署支援 cloud-init 的虛擬機器
部署支援 cloud-init 的虛擬機器就像在部署期間參考支援 cloud-init 的發行版本一樣簡單。  Linux 發行版本維護者必須選擇啟用，並將 cloud-init 整合到其基礎 Azure 發行映像。 一旦確認您想要部署的映像支援 cloud-init，您就可以使用 Azure CLI 來部署映像。 

部署此映像的第一個步驟是使用 [az group create](/cli/azure/group#az_group_create) 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```
下一個步驟是在您目前的殼層中建立名為 *cloud-init.txt* 的檔案，並貼上下列設定。 針對此案例，在 Cloud Shell 中 (而不是本機電腦上) 建立該檔案。 您可以使用任何您想要的編輯器。 輸入 `sensible-editor cloud-init.txt` 可建立檔案，並查看可用的編輯器清單。 建議首先選擇使用 **nano** 編輯器。 請確定已正確複製整個 cloud-init 檔案，特別是第一行：

```yaml
#cloud-config
package_upgrade: true
packages:
  - httpd
```
按下 `ctrl-X` 以結束檔案、輸入 `y` 以儲存檔案，然後按下 `enter` 以在結束時確認檔案名稱。

最後一個步驟是使用 [az vm create](/cli/azure/vm#az_vm_create) 命令來建立 VM。 

下列範例會建立名為 *centos74* 的 VM，並建立 SSH 金鑰 (如果它們不存在於預設金鑰位置)。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。  使用 `--custom-data` 參數以傳入 cloud-init 組態檔。 如果您將檔案儲存於目前工作目錄之外的位置，請提供 cloud-init.txt 組態的完整路徑。 下列範例會建立名為 *centos74* 的 VM：

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud-init.txt \
  --generate-ssh-keys 
```

建立 VM 後，Azure CLI 會顯示您部署專屬的資訊。 記下 `publicIpAddress`。 此位址用來存取 VM。  系統需要花一些時間建立 VM、安裝套件以及啟動應用程式。 在 Azure CLI 將您返回提示字元之後，背景工作會繼續執行。 您可以透過 SSH 連線到 VM，並使用「疑難排解」一節中所述的步驟來檢視 cloud-init 記錄。 

## <a name="troubleshooting-cloud-init"></a>針對 cloud-init 進行疑難排解
一旦佈建 VM，Cloud-init 將會執行 `--custom-data` 中定義的所有模組和指令碼，以設定 VM。  如果您需要針對設定中的任何錯誤或遺漏進行疑難排解，則需要在 cloud-init 記錄 (位於 **/var/log/cloud-init.log**) 中搜尋模組名稱 (例如 `disk_setup` 或 `runcmd`)。

> [!NOTE]
> 並非每個模組失敗都會導致嚴重的 cloud-init 整體設定失敗。 例如，使用 `runcmd` 模組時，如果指令碼失敗，cloud-init 會因為執行了 runcmd 模組而仍然報告佈建成功，。

如需 cloud-init 記錄的詳細資訊，請參閱 [cloud-init 文件](http://cloudinit.readthedocs.io/en/latest/topics/logging.html) \(英文\) 

## <a name="next-steps"></a>後續步驟
如需設定變更的 cloud-init 範例，請參閱下列文件：
 
- [將額外的 Linux 使用者新增至 VM](cloudinit-add-user.md)
- [執行套件管理員以便在初次開機時更新現有的套件](cloudinit-update-vm.md)
- [變更虛擬機器本機的主機名稱](cloudinit-update-vm-hostname.md) 
- [安裝應用程式套件、更新組態檔案，以及插入金鑰](tutorial-automate-vm-deployment.md)
