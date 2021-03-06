---
title: 虛擬機器與虛擬機器擴展集的 Azure 磁碟加密 \(部分機器翻譯\)
description: 瞭解適用于虛擬機器 (Vm) 和 VM 擴展集的 Azure 磁片加密。 Azure 磁片加密適用于 Linux 和 Windows Vm。
author: msmbaldwin
ms.service: security
ms.topic: article
ms.author: mbaldwin
ms.date: 10/15/2019
ms.custom: seodec18
ms.openlocfilehash: 21194bf2fe76a7eb0ee034d4a502c20ee3032dd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "87543668"
---
# <a name="azure-disk-encryption-for-virtual-machines-and-virtual-machine-scale-sets"></a>虛擬機器與虛擬機器擴展集的 Azure 磁碟加密 \(部分機器翻譯\)

Azure 磁片加密可同時套用至 Linux 和 Windows 虛擬機器，以及虛擬機器擴展集。 

## <a name="linux-virtual-machines"></a>Linux 虛擬機器

下列文章提供加密 Linux 虛擬機器的指引。

### <a name="current-version-of-azure-disk-encryption"></a>Azure 磁碟加密的目前版本

- [適用於 Linux 虛擬機器的 Azure 磁碟加密概觀](../../virtual-machines/linux/disk-encryption-overview.md)
- [Linux VM 上的 Azure 磁碟加密案例](../../virtual-machines/linux/disk-encryption-linux.md)
- [使用 Azure CLI 來建立和加密 Linux VM](../../virtual-machines/linux/disk-encryption-cli-quickstart.md)
- [使用 Azure PowerShell 建立和加密 Linux 虛擬機器](../../virtual-machines/linux/disk-encryption-powershell-quickstart.md)
- [使用 Azure 入口網站來建立和加密 Linux VM](../../virtual-machines/linux/disk-encryption-portal-quickstart.md)
- [適用于 Linux 的 Azure 磁碟加密延伸模組架構](../../virtual-machines/extensions/azure-disk-enc-linux.md)
- [建立及設定適用於 Azure 磁碟加密的金鑰保存庫](../../virtual-machines/linux/disk-encryption-key-vault.md)
- [Azure 磁碟加密範例指令碼](../../virtual-machines/linux/disk-encryption-sample-scripts.md)
- [Azure 磁碟加密的疑難排解](../../virtual-machines/linux/disk-encryption-troubleshooting.md)
- [Azure 磁碟加密常見問題集](../../virtual-machines/linux/disk-encryption-faq.md)

### <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>具有 Azure AD (舊版的 Azure 磁片加密) 

- [Linux 虛擬機器 Azure AD 的 Azure 磁碟加密總覽](../../virtual-machines/linux/disk-encryption-overview-aad.md)
- [在 Linux Vm 上使用 Azure AD 案例的 Azure 磁碟加密](../../virtual-machines/linux/disk-encryption-linux.md)
- [使用 Azure AD (舊版建立和設定 Azure 磁碟加密的金鑰保存庫) ](../../virtual-machines/linux/disk-encryption-key-vault-aad.md)

## <a name="windows-virtual-machines"></a>Windows 虛擬機器

下列文章提供加密 Windows 虛擬機器的指引。

### <a name="current-version-of-azure-disk-encryption"></a>Azure 磁碟加密的目前版本

- [適用於 Windows 虛擬機器的 Azure 磁碟加密概觀](../../virtual-machines/windows/disk-encryption-overview.md)
- [Windows VM 上的 Azure 磁碟加密案例](../../virtual-machines/windows/disk-encryption-windows.md)
- [使用 Azure CLI 建立和加密 Windows 虛擬機器](../../virtual-machines/windows/disk-encryption-cli-quickstart.md)
- [使用 Azure PowerShell 建立和加密 Windows 虛擬機器](../../virtual-machines/windows/disk-encryption-powershell-quickstart.md)
- [使用 Azure 入口網站建立和加密 Windows VM](../../virtual-machines/windows/disk-encryption-portal-quickstart.md)
- [適用于 Windows 的 Azure 磁碟加密延伸模組架構](../../virtual-machines/extensions/azure-disk-enc-windows.md)
- [建立及設定適用於 Azure 磁碟加密的金鑰保存庫](../../virtual-machines/windows/disk-encryption-key-vault.md)
- [Azure 磁碟加密範例指令碼](../../virtual-machines/windows/disk-encryption-sample-scripts.md)
- [Azure 磁碟加密的疑難排解](../../virtual-machines/windows/disk-encryption-troubleshooting.md)
- [Azure 磁碟加密常見問題集](../../virtual-machines/windows/disk-encryption-faq.md)

### <a name="azure-disk-encryption-with-azure-ad-previous-version"></a>具有 Azure AD (舊版的 Azure 磁片加密) 

- [Windows 虛擬機器的 Azure AD Azure 磁碟加密總覽](../../virtual-machines/windows/disk-encryption-overview-aad.md)
- [在 Windows Vm 上使用 Azure AD 案例的 Azure 磁碟加密](../../virtual-machines/windows/disk-encryption-windows.md)
- [使用 Azure AD (舊版建立和設定 Azure 磁碟加密的金鑰保存庫) ](../../virtual-machines/windows/disk-encryption-key-vault-aad.md)

## <a name="virtual-machine-scale-sets"></a>虛擬機器擴展集

下列文章提供加密虛擬機器擴展集的指引。

- [虛擬機器擴展集的 Azure 磁碟加密總覽](../../virtual-machine-scale-sets/disk-encryption-overview.md) 
- [使用 Azure CLI 將虛擬機器擴展集加密](../../virtual-machine-scale-sets/disk-encryption-cli.md) 
- [使用 Azure Powershell 來加密虛擬機器擴展集](../../virtual-machine-scale-sets/disk-encryption-powershell.md)。
- [使用 Azure Resource Manager 加密虛擬機器擴展集](../../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md)
- [建立及設定適用於 Azure 磁碟加密的金鑰保存庫](../../virtual-machine-scale-sets/disk-encryption-key-vault.md)
- [使用搭配虛擬機器擴展集擴充功能排序的 Azure 磁碟加密](../../virtual-machine-scale-sets/disk-encryption-extension-sequencing.md)

## <a name="next-steps"></a>後續步驟

- [Azure 加密概觀](encryption-overview.md)
- [待用資料加密](encryption-atrest.md)
- [資料安全性與加密的最佳做法](data-encryption-best-practices.md)
