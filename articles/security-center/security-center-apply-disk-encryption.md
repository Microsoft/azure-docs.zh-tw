---
title: 在 Azure 資訊安全中心中套用磁碟加密 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「套用磁碟加密」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 6cc7824a-8d6b-4a5f-ab40-e3bbaebc4a91
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2017
ms.author: terrylan
ms.openlocfilehash: bcfa6ead0eac4a8a631c0388466534406c7a594b
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42887333"
---
# <a name="apply-disk-encryption-in-azure-security-center"></a>在 Azure 資訊安全中心中套用磁碟加密
如果您有未使用 Azure 磁碟加密進行加密的 Windows 或 Linux VM 磁碟，Azure 資訊安全中心會建議您套用磁碟加密。 磁碟加密可讓您替 Windows 和 Linux IaaS VM 磁碟加密。  建議您的 VM 上的作業系統和資料磁碟區都進行加密。

磁碟加密使用 Windows 的業界標準 [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) 功能和 Linux 的 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt)功能。 這些功能為 OS 和資料提供加密來協助保護資料安全，以符合您的組織安全性和合規性承諾。 磁碟加密與 [Azure 金鑰保存庫](https://azure.microsoft.com/documentation/services/key-vault/)整合，可幫助您控制和管理您的金鑰保存庫訂用帳戶中的磁碟加密金鑰和密碼，同時確保 VM 磁碟中的所有資料會在您的 [Azure 儲存體](https://azure.microsoft.com/documentation/services/storage/)中輕鬆加密。

> [!NOTE]
> 下列 Windows 伺服器作業系統支援 Azure 磁碟加密 - Windows Server 2008 R2、Windows Server 2012 和 Windows Server 2012 R2。 下列 Linux 伺服器作業系統支援磁碟加密 - Ubuntu、CentOS、SUSE 和 SUSE Linux Enterprise Server (SLES)。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取 [套用磁碟加密]。
2. 在 [套用磁碟加密]  刀鋒視窗中，您會看到建議進行磁碟加密的 VM 清單。
3. 依照指示將加密套用至這些 VM。

![][1]

若要加密已由資訊安全中心識別為需要加密的 Azure 虛擬機器，建議您執行下列步驟︰

* 安裝並設定 Azure PowerShell。 這可讓您執行必要的 PowerShell 命令，以便設定用來加密 Azure 虛擬機器的必要條件。
* 取得並執行 Azure 磁碟加密先決條件 Azure PowerShell 指令碼。
* 加密虛擬機器。

[使用 Azure PowerShell 為 Windows IaaS VM 加密](../security/quick-encrypt-vm-powershell.md)可引導您完成這些步驟。 本主題假設您使用 Windows 用戶端電腦，並從中設定磁碟加密。

有許多方法可以用於 Azure 虛擬機器。 如果您已經很熟悉 Azure PowerShell 或 Azure CLI，可能會想要使用其他方法。 若要深入了解其他這些方法，請參閱 [Azure 磁碟加密](../security/azure-security-disk-encryption.md)。

## <a name="see-also"></a>另請參閱
本文件說明如何實作 Azure 資訊安全中心建議的「套用磁碟加密」。 若要深入了解磁碟加密，請參閱下列主題：

* [Azure 金鑰保存庫的加密和金鑰管理](https://azure.microsoft.com/documentation/videos/azurecon-2015-encryption-and-key-management-with-azure-key-vault/) (影片，36 分 39 秒) -- 了解如何使用 IaaS VM 和 Azure 金鑰保存庫的磁碟加密管理功能，協助保護您的資料。
* [Azure 磁碟加密](../security/azure-security-disk-encryption-overview.md) (文件) -- 了解如何為 Windows 和 Linux VM 啟用磁碟加密。

如要深入了解資訊安全中心，請參閱下列主題：

* [設定 Azure 資訊安全中心的安全性原則](security-center-policies.md) -- 了解如何設定安全性原則。
* [Azure 資訊安全中心的安全性健康狀態監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健康狀態。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助您保護您的 Azure 資源。
* [Azure 安全性中心常見問題集](security-center-faq.md) -- 尋找使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-apply-disk-encryption/apply-disk-encryption.png
