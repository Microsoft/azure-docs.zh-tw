---
title: 可用於 Azure 儲存體的安全性功能 | Microsoft Docs
description: 本文提供可用於 Azure 儲存體的核心 Azure 安全性功能概觀。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/17/2018
ms.author: terrylan
ms.openlocfilehash: 4ed4e73348db8cfffb6e79afaa9d196e242d7488
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42145757"
---
# <a name="azure-storage-security-overview"></a>Azure 儲存體安全性概觀

Azure 儲存體是現代應用程式的雲端儲存體解決方案，這些應用程式仰賴持續性、可用性和可調整性來滿足其客戶的需求。 Azure 儲存體提供一組完整的安全性功能。 您可以：

* 使用角色型存取控制 (RBAC) 與 Azure Active Directory 來保護儲存體帳戶的安全。
* 使用用戶端加密、HTTPS 或 SMB 3.0，保護應用程式和 Azure 之間傳輸中資料的安全。
* 設定當使用儲存體服務加密來將資料寫入 Azure 儲存體時自動加密資料。
* 將虛擬機器 (VM) 所使用的作業系統和資料磁碟設定為使用 Azure 磁碟加密來加密。
* 使用共用存取簽章 (SAS)，將委派存取權授與 Azure 儲存體中的資料物件。
* 使用分析來追蹤當某人存取儲存體時使用的驗證方法。

若要深入了解「Azure 儲存體」中的安全性，請參閱 [Azure 儲存體安全性指南](../storage/common/storage-security-guide.md)。 本指南提供 Azure 儲存體安全性功能的深入探討。 這些功能包括儲存體帳戶金鑰、傳輸中和待用資料加密，以及儲存體分析。

本文提供可與 Azure 儲存體搭配使用的 Azure 安全性功能概觀。 文中也會提供文章的連結，更詳細說明每個功能，好讓您能夠深入了解。

## <a name="role-based-access-control"></a>角色型存取控制

您可以使用角色型存取控制，協助保護儲存體帳戶。 對於想要強制執行資料存取安全性原則的組織，根據[需要知道](https://en.wikipedia.org/wiki/Need_to_know)和[最低權限](https://en.wikipedia.org/wiki/Principle_of_least_privilege)安全性原則限制存取權限是必須做的事。 在特定範圍將適當的 RBAC 角色指派給群組和應用程式，即可授與這些存取權限。 您可以使用 [內建的 RBAC 角色](../role-based-access-control/built-in-roles.md)(例如儲存體帳戶參與者) 將權限指派給使用者。

深入了解：

* [Azure Active Directory 角色型存取控制](../role-based-access-control/role-assignments-portal.md)

## <a name="delegated-access-to-storage-objects"></a>儲存體物件的委派存取權

共用存取簽章可提供您儲存體帳戶中資源的委派存取。 SAS 意謂著您可以將儲存體帳戶中物件的有限權限授與用戶端，讓該用戶端可以在一段指定期間內使用一組指定的權限進行存取。 您可以在不須分享您帳戶存取金鑰的情況下，授與這些有限的權限。 

SAS 是一種 URI，此 URI 會在其查詢參數中包含對儲存體資源進行驗證式存取所需的一切資訊。 若要使用 SAS 存取儲存體資源，用戶端只需將 SAS 提供給適當的建構函式或方法即可。

深入了解：

* [了解 SAS 模型](../storage/common/storage-dotnet-shared-access-signature-part-1.md)
* [透過 Blob 儲存體來建立及使用 SAS](../storage/blobs/storage-dotnet-shared-access-signature-part-2.md)

## <a name="encryption-in-transit"></a>傳輸中加密

傳輸中加密是在透過網路傳輸資料時用來保護資料的機制。 使用 Azure 儲存體時，您可以使用下列各項來保護資料：

* [傳輸層級加密](../storage/common/storage-security-guide.md#encryption-in-transit)，例如將資料傳入或傳出 Azure 儲存體時的 HTTPS。
* [連線加密](../storage/common/storage-security-guide.md#using-encryption-during-transit-with-azure-file-shares)，例如適用於 Azure 檔案共用的 SMB 3.0 加密。
* [用戶端加密](../storage/common/storage-security-guide.md#using-client-side-encryption-to-secure-data-that-you-send-to-storage)，可在將資料傳輸到儲存體之前加密資料，並在從儲存體傳出資料之後將資料解密。

深入了解用戶端加密︰

* [Microsoft Azure 儲存體的用戶端加密](https://blogs.msdn.microsoft.com/windowsazurestorage/2015/04/28/client-side-encryption-for-microsoft-azure-storage-preview/)
* [雲端安全性控制項系列︰加密傳輸中的資料](http://blogs.microsoft.com/cybertrust/2015/08/10/cloud-security-controls-series-encrypting-data-in-transit/)

## <a name="encryption-at-rest"></a>待用加密

對多組織來說，[待用資料加密](https://blogs.microsoft.com/cybertrust/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/) \(英文\) 是達到資料隱私權、合規性及資料主權的必要步驟。 有三個 Azure 功能可提供待用資料的加密：

* [儲存體服務加密](../storage/common/storage-security-guide.md#encryption-at-rest)可讓您要求儲存體服務在將資料寫入 Azure 儲存體時自動加密資料。
* [用戶端加密](../storage/common/storage-security-guide.md#client-side-encryption)也提供待用加密的功能。
* [Azure 磁碟加密](../storage/common/storage-security-guide.md#using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines)允許您加密 IaaS 虛擬機器所使用的 OS 磁碟和資料磁碟。

深入了解儲存體服務加密：

* [Azure 儲存體服務加密](https://azure.microsoft.com/services/storage/)適用於 [Azure Blob 儲存體](https://azure.microsoft.com/services/storage/blobs/)。 如需其他 Azure 儲存體類型的詳細資訊，請參閱 [Azure 檔案](https://azure.microsoft.com/services/storage/files/)、[磁碟 (進階儲存體)](https://azure.microsoft.com/services/storage/premium-storage/)、[資料表儲存體](https://azure.microsoft.com/services/storage/tables/)和[佇列儲存體](https://azure.microsoft.com/services/storage/queues/)。
* [待用資料的 Azure 儲存體服務加密](../storage/common/storage-service-encryption.md)

## <a name="azure-disk-encryption"></a>Azure 磁碟加密

適用於虛擬機器的 Azure 磁碟加密可協助您滿足組織的安全性和合規性需求。 它會使用您在 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 中控制的金鑰和原則，來將您的 VM 磁碟 (包括開機和資料磁碟) 加密。

適用於 VM 的磁碟加密可用於 Linux 與 Windows 作業系統。 也會使用金鑰保存庫，協助您防護、管理和稽核您的磁碟加密金鑰的使用情形。 VM 磁碟中的所有資料都會使用 Azure 儲存體帳戶中符合業界標準的加密技術進行待用加密。 Windows 的磁碟加密解決方案是建基於 [Microsoft BitLocker 磁碟機加密](https://technet.microsoft.com/library/cc732774.aspx)，而 Linux 解決方案是建基於 [dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt)。

深入了解

* [適用於 Windows 和 Linux IaaS 虛擬機器的 Azure 磁碟加密](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)

## <a name="azure-storage-firewalls-and-virtual-networks"></a>Azure 儲存體防火牆和虛擬網路

Azure 儲存體可讓您啟用儲存體帳戶的防火牆規則。 啟用之後，這些規則將會封鎖資料的傳入要求，包括來自其他 Azure 服務的要求。 但您可以設定例外狀況來允許流量。 您可以在現有的儲存體帳戶上或建立儲存體帳戶時，啟用防火牆規則。

您應該使用此功能來保護您的儲存體帳戶，只讓一組允許的特定網路存取儲存體帳戶。

如需有關 Azure 儲存體防火牆和虛擬網路的詳細資訊，請檢閱[設定 Azure 儲存體防火牆和虛擬網路](../storage/common/storage-network-security.md)一文

## <a name="azure-key-vault"></a>Azure 金鑰保存庫

Azure 磁碟加密會使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)，協助您控制及管理金鑰保存庫訂用帳戶中的磁碟加密金鑰與祕密。 它也可確保虛擬機器磁碟上的所有資料都會在 Azure 儲存體中進行待用加密。 您應使用金鑰保存庫來稽核金鑰和原則使用方式。

深入了解

* [什麼是 Azure 金鑰保存庫？](../key-vault/key-vault-whatis.md)
* [開始使用 Azure 金鑰保存庫](../key-vault/key-vault-get-started.md)
