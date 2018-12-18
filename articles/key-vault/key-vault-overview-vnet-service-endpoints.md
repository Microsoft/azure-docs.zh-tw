---
ms.assetid: ''
title: Azure Key Vault 的 VNET 服務端點 | Microsoft Docs
description: Key Vault 的虛擬網路服務端點概觀
services: key-vault
author: amitbapat
ms.author: ambapat
manager: mbaldwin
ms.date: 08/31/2018
ms.service: key-vault
ms.workload: identity
ms.topic: conceptual
ms.openlocfilehash: c2696d5eb22443b565c48ef4f96d6e4a25827606
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44294999"
---
# <a name="virtual-network-service-endpoints-for-azure-key-vault"></a>Azure Key Vault 的虛擬網路服務端點

Key Vault 的虛擬網路服務端點可讓您將指定虛擬網路和/或 IPv4 的存取限制於 IPv4 (網際網路通訊協定第 4 版) 位址範圍的清單。 任何從這些來源之外連線到金鑰保存庫的呼叫端都會被拒絕存取。 如果客戶已選擇允許「信任的 Microsoft 服務」，例如 Office 365 Exchange Online、Office 365 SharePoint Online、Azure Compute、Azure Resource Manager、Azure 備份等，就會讓來自這些服務的連線通過防火牆。 當然，這類呼叫端仍必須出示有效的 AAD 權杖，而且必須具有權限 (如存取原則設定) 才能執行要求的作業。 深入了解[虛擬網路服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)的相關技術詳細資訊。

## <a name="usage-scenarios"></a>使用案例

根據預設，您可以將 [Key Vault 防火牆和虛擬網路](key-vault-network-security.md)設定成拒絕存取所有網路流量 (包括網際網路流量)。 可授與存取特定 Azure 虛擬網路和/或公用網際網路 IP 位址範圍的流量，讓您為應用程式建置安全的網路界限。

> [!NOTE]
> Key Vault 防火牆和虛擬網路規則只適用於金鑰保存庫[資料平面](../key-vault/key-vault-secure-your-key-vault.md#data-plane-access-control)。 Key Vault 控制平面作業 (例如金鑰保存庫的建立、刪除、修改作業、設定存取原則、設定防火牆和虛擬網路規則) 不受防火牆和虛擬網路規則影響。

例如，
* 如果您使用 Key Vault 來儲存加密金鑰、應用程式祕密、憑證，您會想要禁止從公用網際網路存取您的金鑰保存庫。
* 您想要鎖定您金鑰保存庫的存取權，以致只有您的應用程式或少數幾個指定的主機可以連線到您的金鑰保存庫
* 您有在 Azure 虛擬網路 (VNET) 中執行的應用程式，而此 VNET 已針對所有的輸入和輸出流量進行鎖定。 您的應用程式仍然需要連線至金鑰保存庫，以擷取祕密或憑證，或使用密碼編譯金鑰。

## <a name="configure-key-vault-firewalls-and-virtual-networks"></a>設定 Key Vault 防火牆和虛擬網路

以下是設定防火牆和虛擬網路所需的步驟。 無論您使用哪個介面 (PowerShell、CLI、Azure 入口網站) 來設定防火牆與虛擬網路規則，這些步驟都維持不變。
1. 選擇性，但強烈建議採用：啟用[金鑰保存庫記錄](key-vault-logging.md)，以查看詳細的存取記錄。 當防火牆和虛擬網路規則防止存取金鑰保存庫時，這可協助您進行診斷。
2. 針對目標虛擬網路和子網路中啟用 [金鑰保存庫的服務端點]
3. 設定金鑰保存庫的防火牆和虛擬網路規則，以限制從特定的虛擬網路、子網路和 IPv4 位址範圍存取該金鑰保存庫。
4. 如果有任何信任的 Microsoft 服務必須存取此金鑰保存庫，您必須啟用此選項，讓 [信任的 Azure 服務] 連線能夠至金鑰保存庫。

如需詳細的逐步指示，請參閱[設定 Azure Key Vault 防火牆和虛擬網路](key-vault-network-security.md)。

> [!IMPORTANT]
> 防火牆規則一旦生效，只有在呼叫端要求源自允許的虛擬網路或 IPV4 位址範圍時，才可以執行所有的 Key Vault [資料平面](../key-vault/key-vault-secure-your-key-vault.md#data-plane-access-control)作業。 這也適用於從 Azure 入口網站存取金鑰保存庫。 雖然使用者可以從 Azure 入口網站瀏覽金鑰保存庫，但是如果其用戶端電腦不在允許的清單中，他們就無法列出金鑰/祕密/憑證。 這也會影響其他 Azure 服務的 [金鑰保存庫選擇器]。 使用者可以看到金鑰保存庫清單，但是如果防火牆規則阻止其用戶端電腦，則不會列出金鑰。


> [!NOTE]
> * 最多允許 127 條 VNET 規則和 127 條 IPv4 規則。 
> * 不支援使用 "/31" 或 "/32" 前置詞大小的小型位址範圍。 這些範圍應該使用個別的 IP 位址規則設定。
> * 只有公用 IP 位址允許使用 IP 網路規則。 IP 規則中不允許保留私人網路的 IP 位址範圍 (如 RFC 1918 中所定義)。 私人網路包括以 *10.*\*、*172.16.*\* 和 *192.168.*\* 開頭的位址。 
> * 目前僅支援 IPv4 位址。

## <a name="trusted-services"></a>信任的服務
以下是已啟用 [允許信任的服務] 選項的情況下，允許存取金鑰保存庫的信任服務清單。

|信任的服務|使用案例|
| --- | --- |
|Azure 虛擬機器部署服務|[Deploy Certificates to VMs from customer-managed Key Vault (將憑證從客戶管理的 Key Vault 部署到 VM)](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/)|
|Azure Resource Manager 範本部署服務|[在部署期間傳遞安全值](../azure-resource-manager/resource-manager-keyvault-parameter.md)|
|Azure Disk Encryption 磁碟區加密服務|允許在 VM 部署期間存取 BitLocker 金鑰 (Windows VM) 或 DM 複雜密碼 (Linux VM) 和金鑰加密金鑰，以啟用 [Azure 磁碟加密](../security/azure-security-disk-encryption.md)|
|Azure 備份|使用[Azure 備份](../backup/backup-introduction-to-azure-backup.md)，在 Azure VM 備份期間允許相關金鑰和祕密的備份和還原|
|Exchange Online 和 SharePoint Online|允許使用[客戶金鑰](https://support.office.com/en-us/article/Controlling-your-data-in-Office-365-using-Customer-Key-f2cd475a-e592-46cf-80a3-1bfb0fa17697)存取服務加密的客戶金鑰。|
|Azure 資訊保護|允許存取 [Azure 資訊保護](https://docs.microsoft.com/azure/information-protection/what-is-information-protection)的租用戶金鑰|
|應用程式服務|[透過 Key Vault 部署 Azure Web 應用程式憑證](https://blogs.msdn.microsoft.com/appserviceteam/2016/05/24/deploying-azure-web-app-certificate-through-key-vault/)|
|Azure SQL|[Azure SQL Database 和資料倉儲具有「攜帶您自己的金鑰」支援的的透明資料加密](../sql-database/transparent-data-encryption-byok-azure-sql.md?view=sql-server-2017&viewFallbackFrom=azuresqldb-current)|
|Azure 儲存體|[使用 Azure Key Vault 中客戶管理的金鑰進行儲存體服務加密](../storage/common/storage-service-encryption-customer-managed-keys.md)|
|Azure Data Lake Store|使用客戶管理的金鑰[在 Azure Data Lake Store 中加密資料](../data-lake-store/data-lake-store-encryption.md)|



> [!NOTE]
> 必須將相關金鑰保存庫存取原則設定為允許對應的服務取得金鑰保存庫的存取權。

## <a name="next-steps"></a>後續步驟

* [保護您的金鑰保存庫](key-vault-secure-your-key-vault.md)
* [設定 Azure Key Vault 防火牆和虛擬網路](key-vault-network-security.md)