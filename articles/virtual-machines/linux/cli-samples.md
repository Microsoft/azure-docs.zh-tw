---
title: Azure CLI 範例 | Microsoft Docs
description: Azure CLI 範例
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/08/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 1e4b7027ecbc6e70fadf8c540fa92903d276956b
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37931706"
---
# <a name="azure-cli-samples-for-linux-virtual-machines"></a>適用於 Linux 虛擬機器的 Azure CLI 範例

下表包含使用 Azure CLI 所建置之 Bash 指令碼的連結。

| | |
|---|---|
|**建立虛擬機器**||
| [建立虛擬機器](./../scripts/virtual-machines-linux-cli-sample-create-vm-quick-create.md?toc=%2fcli%2fazure%2ftoc.json) | 以最低限度的組態建立 Linux 虛擬機器。 |
| [建立完整設定的虛擬機器](./../scripts/virtual-machines-linux-cli-sample-create-vm.md?toc=%2fcli%2fazure%2ftoc.json) | 建立資源群組、虛擬機器和所有相關資源。|
| [建立高可用性的虛擬機器](./../scripts/virtual-machines-linux-cli-sample-nlb.md?toc=%2fcli%2fazure%2ftoc.json) | 依據高可用性和負載平衡組態建立數個虛擬機器。 |
| [建立 VM 並執行組態指令碼](./../scripts/virtual-machines-linux-cli-sample-create-vm-nginx.md?toc=%2fcli%2fazure%2ftoc.json) | 建立虛擬機器，並使用「Azure 自訂指令碼」擴充功能來安裝 NGINX。 |
| [建立已安裝 WordPress 的 VM](./../scripts/virtual-machines-linux-cli-sample-create-vm-wordpress.md?toc=%2fcli%2fazure%2ftoc.json) | 建立虛擬機器，並使用「Azure 自訂指令碼」擴充功能來安裝 WordPress。 |
| [從受控 OS 磁碟建立 VM](./../scripts/virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json) | 連結現有受控磁碟作為 OS 磁碟，以建立虛擬機器。 |
| [從快照集建立 VM](./../scripts/virtual-machines-linux-cli-sample-create-vm-from-snapshot.md?toc=%2fcli%2fmodule%2ftoc.json) | 先從快照集建立受控磁碟，然後連結新的受控磁碟作為 OS 磁碟，以從快照集建立虛擬機器。 |
|**管理儲存體**||
| [從 VHD 建立受控磁碟](../scripts/virtual-machines-linux-cli-sample-create-managed-disk-from-vhd.md?toc=%2fcli%2fmodule%2ftoc.json) | 從作為 OS 磁碟的特殊化 VHD 或從作為資料磁碟的 VHD 建立受控磁碟。  |
| [從快照集建立受控磁碟](../scripts/virtual-machines-linux-cli-sample-create-managed-disk-from-snapshot.md?toc=%2fcli%2fmodule%2ftoc.json) | 從快照集建立受控磁碟。 |
| [將受控磁碟複製到相同或不同的訂用帳戶](../scripts/virtual-machines-linux-cli-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json) | 將受控磁碟複製到與父受控磁碟相同區域中相同或不同的訂用帳戶。 
| [將快照集以 VHD 格式匯出至儲存體帳戶](../scripts/virtual-machines-linux-cli-sample-copy-snapshot-to-storage-account.md?toc=%2fcli%2fmodule%2ftoc.json) | 將受控快照集以 VDH 格式匯出至不同區域中的儲存體帳戶。 |
| [將快照集複製到相同或不同的訂用帳戶](../scripts/virtual-machines-linux-cli-sample-copy-snapshot-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json) | 將快照集複製到與父快照集相同區域中相同或不同的訂用帳戶。 |
|**網路虛擬機器**||
| [保護虛擬機器之間的網路流量](./../scripts/virtual-machines-linux-cli-sample-create-vm-nsg.md?toc=%2fcli%2fazure%2ftoc.json) | 建立兩部虛擬機器、所有相關資源以及內部和外部網路安全性群組 (NSG)。 |
|**保護虛擬機器**||
| [加密 VM 和資料磁碟](./../scripts/virtual-machines-linux-cli-sample-encrypt-vm.md?toc=%2fcli%2fazure%2ftoc.json) | 建立 Azure Key Vault、加密金鑰和服務主體，然後加密 VM。 |
|**監視虛擬機器**||
| [使用 Operations Management Suite 監視 VM](./../scripts/virtual-machines-linux-cli-sample-create-vm-oms.md?toc=%2fcli%2fazure%2ftoc.json) | 建立虛擬機器、安裝 Operations Management Suite 代理程式，並在 OMS 工作區中註冊 VM。  |
|**針對虛擬機器進行疑難排解**||
| [針對 VM 作業系統磁碟進行疑難排解](./../scripts/virtual-machines-linux-cli-sample-mount-os-disk.md?toc=%2fcli%2fazure%2ftoc.json) | 掛接一個 VM 的作業系統磁碟作為第二個 VM 上的資料磁碟。 |
| | |
