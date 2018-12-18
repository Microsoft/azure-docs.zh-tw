---
title: Azure 加密概觀 | Microsoft Docs
description: 了解 Azure 中的各種加密選項
services: security
documentationcenter: na
author: Barclayn
manager: MBaldwin
editor: TomShinder
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/18/2017
ms.author: barclayn
ms.openlocfilehash: 00c8b30b5351b7a6e4388b186fab70e3a3357ef4
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34366302"
---
# <a name="azure-encryption-overview"></a>Azure 加密概觀

本文提供如何在 Microsoft Azure 中使用加密的概觀。 它涵蓋加密的主要領域，包括待用加密、傳輸中加密，以及使用 Azure Key Vault 的金鑰管理。 每節都包含更詳細資訊的連結。

## <a name="encryption-of-data-at-rest"></a>待用資料加密

待用資料包含位於實體媒體之永續性儲存體中任何數位格式的資訊。 這個媒體包括磁性或光學媒體上的檔案、已封存的資料，以及資料備份。 Microsoft Azure 提供各種資料儲存體解決方案來符合不同的需求，包括檔案、磁碟、Blob 和資料表儲存體。 Microsoft 也提供加密來保護 [Azure SQL Database](../sql-database/sql-database-technical-overview.md)、[Azure Cosmos DB](../cosmos-db/introduction.md) 和 Azure Data Lake。

待用資料加密適用於軟體即服務 (SaaS)、平台即服務 (PaaS) 和基礎結構即服務 (IaaS) 雲端模型中的服務。 本文摘要說明並提供可協助您使用 Azure 加密選項的資源。

如需如何在 Azure 中加密待用資料的更詳細討論，請參閱 [Azure 待用資料加密](azure-security-encryption-atrest.md)。

## <a name="azure-encryption-models"></a>Azure 加密模型

Azure 支援各種加密模型，包括使用下列方式進行的伺服器端加密：服務管理的金鑰、Key Vault 中客戶管理的金鑰，或受客戶控制硬體上客戶管理的金鑰。 使用用戶端加密時，您可以在內部部署或另一個安全位置中管理和儲存金鑰。

### <a name="client-side-encryption"></a>用戶端加密

用戶端加密是在 Azure 外部執行。 其中包括：

- 由在客戶資料中心執行的應用程式或由服務應用程式加密的資料。
- Azure 收到時便已加密的資料。

使用用戶端加密時，雲端服務提供者無法存取加密金鑰，而且無法將此資料解密。 您會維持對金鑰的完全掌控。

### <a name="server-side-encryption"></a>伺服器端加密

這三種伺服器端加密模型提供不同的金鑰管理特性，您可以根據需求進行選擇：

- **服務管理的金鑰**：結合控制能力與便利性，而且額外負荷很低。

- **客戶管理的金鑰**：可讓您控制金鑰，包括支援「攜帶您自己的金鑰」(BYOK)，或允許您產生新的金鑰。

- **受客戶控制硬體中服務管理的金鑰**：可讓您在不受 Microsoft 控制的專屬存放庫中管理金鑰。 這個特性稱為「裝載您自己的金鑰」(HYOK)。 不過，設定很複雜，而且大多數 Azure 服務並不支援此模型。

### <a name="azure-disk-encryption"></a>Azure 磁碟加密

您可以使用 [Azure 磁碟加密](azure-security-disk-encryption.md)來保護 Windows 和 Linux 虛擬機器，其會使用 [Windows BitLocker](https://technet.microsoft.com/library/cc766295(v=ws.10).aspx) \(英文\) 技術和 Linux [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) \(英文\)，透過完整磁碟區加密來保護作業系統磁碟和資料磁碟。

加密金鑰和祕密會在您的 [Azure Key Vault 訂用帳戶](../key-vault/key-vault-whatis.md)中受到保護。 使用 Azure 備份服務，您可以備份及還原使用金鑰加密金鑰 (KEK) 設定的已加密虛擬機器 (VM)。

### <a name="azure-storage-service-encryption"></a>Azure 儲存體服務加密

Azure Blob 儲存體和 Azure 檔案共用中的待用資料可以在伺服器端和用戶端案例中進行加密。

[Azure 儲存體服務加密 (SSE)](../storage/common/storage-service-encryption.md) 可以在儲存資料前自動加密資料，並在您擷取資料時自動將資料解密。 此程序是在背景自動執行，完全不需要使用者介入。 「儲存體服務加密」會使用 256 位元的[進階加密標準 (AES) 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(英文\)，這是可供使用的最強區塊編碼器之一。 AES 會在背景自動處理加密、解密和金鑰管理。

### <a name="client-side-encryption-of-azure-blobs"></a>Azure Blob 的用戶端加密

您可以利用各種方式來執行 Azure Blob 的用戶端加密。

您可以使用適用於 .NET NuGet 套件的 Azure 儲存體用戶端程式庫，將用戶端應用程式中的資料加密，然後將它上傳到您的 Azure 儲存體。

若要深入了解並下載適用於 .NET NuGet 套件的 Azure 儲存體用戶端程式庫，請參閱 [Windows Azure 儲存體 8.3.0](https://www.nuget.org/packages/WindowsAzure.Storage) \(英文\)。

當搭配 Key Vault 使用用戶端加密時，您的資料會使用 Azure 儲存體用戶端 SDK 所產生的一次性對稱內容加密金鑰 (CEK) 進行加密。 使用金鑰加密金鑰 (KEK) 加密的 CEK 可以是對稱金鑰或非對稱金鑰組。 您可以在本機進行管理，或將它儲存在 Key Vault 中。 然後，加密的資料會上傳到 Azure 儲存體。

若要深入了解如何搭配 Key Vault 使用用戶端加密，並使用作法指示開始執行，請參閱[教學課程：在 Azure 儲存體中使用 Key Vault 加密和解密 Blob](../storage/storage-encrypt-decrypt-blobs-key-vault.md)。

最後，您也可以使用適用於 Java 的 Azure 儲存體用戶端程式庫，在將資料上傳到 Azure 儲存體前先執行用戶端加密，並且在將資料下載到用戶端時將它解密。 此程式庫也支援與 [Key Vault](https://azure.microsoft.com/services/key-vault/) 整合，以進行儲存體帳戶金鑰管理。

### <a name="encryption-of-data-at-rest-with-azure-sql-database"></a>使用 Azure SQL Database 加密待用資料

[Azure SQL Database](../sql-database/sql-database-technical-overview.md) 是 Azure 中的一般用途關聯式資料庫服務，可支援關聯式資料、JSON、空間和 XML 等結構。 SQL Database 透過透明資料加密 (TDE) 功能支援伺服器端加密，並透過 Always Encrypted 功能支援用戶端加密。

#### <a name="transparent-data-encryption"></a>透明資料加密

[TDE](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-tde) 可用來使用資料庫加密金鑰 (DEK) 即時加密 [SQL Server](https://www.microsoft.com/sql-server/sql-server-2016)、[Azure SQL Database](../sql-database/sql-database-technical-overview.md) 和 [Azure SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)資料檔案，該金鑰儲存在資料庫開機記錄中，以在復原期間使用。

TDE 會使用 AES 和三重資料加密標準 (3DES) 加密演算法來保護資料和記錄檔。 資料庫檔案的加密會在頁面層級執行。 已加密資料庫中的頁面會在寫入磁碟之前進行加密，並會在讀入記憶體時進行解密。 現在預設會在新建立的 Azure SQL Database 上啟用 TDE。

#### <a name="always-encrypted-feature"></a>Always Encrypted 功能

透過 Azure SQL 中的 [Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine) 功能，您可以先在用戶端應用程式內將資料加密，然後將它儲存於 Azure SQL Database 中。 您也可以將內部部署資料庫系統管理作業委派給第三方，並持續將擁有且可檢視資料的人員與可管理資料但不應存取資料的人員區隔開來。

#### <a name="cell-level-or-column-level-encryption"></a>資料格層級或資料行層級的加密

使用 Azure SQL Database 時，您可以使用 Transact-SQL 將對稱式加密套用到資料行。 這個方法稱為[資料格層級加密或資料行層級加密 (CLE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/encrypt-a-column-of-data)，因為您可以使用它，透過不同的加密金鑰來加密資料的特定資料行或更特定的資料格。 這樣做可為您提供比 TDE 更細微的加密功能，它可以加密頁面中的資料。

CLE 具有內建函式，可供您使用對稱或非對稱金鑰、憑證的公開金鑰，或使用 3DES 的複雜密碼來加密資料。

### <a name="cosmos-db-database-encryption"></a>Cosmos DB 資料庫加密

[Azure Cosmos DB](../cosmos-db/database-encryption-at-rest.md) 是 Microsoft 之全域散發的多模型資料庫。 預設會加密儲存在 Cosmos DB 非揮發性儲存體 (固態硬碟) 中的使用者資料。 沒有可開啟或關閉它的控制項。 待用加密是使用數種安全性技術來實作的，這些技術包括安全金鑰儲存體系統、加密的網路，以及密碼編譯 API。 加密金鑰是由 Microsoft 管理，並根據 Microsoft 內部方針來輪替。

### <a name="at-rest-encryption-in-data-lake"></a>Data Lake 中的待用加密

[Azure Data Lake](../data-lake-store/data-lake-store-encryption.md) 是企業級的存放庫，可於單一位置收集每種資料類型，再正式定義任何需求或結構描述。 Data Lake Store 支援「預設開啟」的透明待用資料加密，您可以在建立帳戶期間進行設定。 根據預設，Azure Data Lake Store 會為您管理金鑰，但您可以選擇自行管理這些金鑰。

您有三種金鑰類型可用於加密和解密資料：主要加密金鑰 (MEK)、資料加密金鑰 (DEK) 和區塊加密金鑰 (BEK)。 MEK 可用來加密 DEK，並將它儲存在永續性媒體中，而 BEK 衍生自 DEK 和資料區塊。 如果您要管理自己的金鑰，您可以輪替 MEK。

## <a name="encryption-of-data-in-transit"></a>傳輸中資料加密

Azure 提供許多機制，可在將資料從一個位置移至另一個位置時，確保資料的隱私權。

### <a name="tlsssl-encryption-in-azure"></a>Azure 的 TLS/SSL 加密

在雲端服務與客戶之間移動資料時，Microsoft 使用[傳輸層安全性](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS) 通訊協定來保護資料。 Microsoft 資料中心會與連線到 Azure 服務的用戶端系統進行交涉以達成 TLS 連線。 TLS 提供增強式驗證、訊息隱私權、完整性 (可偵測訊息竄改、攔截和偽造)、互通性、演算法彈性，以及方便部署和使用。

[完整轉寄密碼](https://en.wikipedia.org/wiki/Forward_secrecy) (PFS) \(英文\) 會透過唯一金鑰來保護客戶的用戶端系統與 Microsoft 雲端服務之間的連線。 這些連線也會使用 RSA 型 2048 位元加密金鑰長度。 這種組合讓其他人很難攔截和存取傳輸中的資料。

### <a name="azure-storage-transactions"></a>Azure 儲存體交易

當您透過 Azure 入口網站與 Azure 儲存體互動時，所有交易都會透過 HTTPS 進行。 您也可以透過 HTTPS 使用儲存體 REST API 來與 Azure 儲存體互動。 透過啟用儲存體帳戶所需的安全傳輸，您可於呼叫 REST API 來存取儲存體帳戶中的物件時強制使用 HTTPS。

共用存取簽章 ([SAS](../storage/storage-dotnet-shared-access-signature-part-1.md)) 可用來委派 Azure 儲存體物件的存取權，包含一個選項，可指定在您使用共用存取簽章時只能使用 HTTPS 通訊協定。 這個方法可確保任何傳送具有 SAS 權杖的連結的人都會使用正確的通訊協定。

用來存取 Azure 檔案共用的 [SMB 3.0](https://technet.microsoft.com/library/dn551363(v=ws.11).aspx#BKMK_SMBEncryption) \(英文\) 支援加密，而且可在 Windows Server 2012 R2、Windows 8、Windows 8.1 和 Windows 10 中找到。 它允許跨區域存取，甚至可在電腦上存取。

用戶端加密會先將資料加密，然後傳送到 Azure 儲存體執行個體，因此在網路中傳送時會處於加密狀態。

### <a name="smb-encryption-over-azure-virtual-networks"></a>Azure 虛擬網路上的 SMB 加密 

透過在執行 Windows Server 2012 或更新版本的 VM 中使用 [SMB 3.0](https://support.microsoft.com/help/2709568/new-smb-3-0-features-in-the-windows-server-2012-file-server) \(機器翻譯\)，您可以透過將在 Azure 虛擬網路上傳輸的資料加密，來保護資料傳輸的安全。 透過加密資料，您可以協助防止竄改和竊聽攻擊。 系統管理員可以針對整部伺服器啟用 SMB 加密，或只針對特定共用來啟用。

根據預設，在針對共用或伺服器開啟 SMB 加密之後，只有 SMB 3.0 用戶端才能存取加密的共用。

## <a name="in-transit-encryption-in-vms"></a>VM 中的傳輸中加密

在執行 Windows 的 VM 之間來回傳輸的資料是根據連線本質透過數種方式加密的。

### <a name="rdp-sessions"></a>RDP 工作階段

您可以從 Windows 用戶端電腦或已安裝 RDP 用戶端的 Mac 電腦，使用[遠端桌面通訊協定 (RDP)](https://msdn.microsoft.com/library/aa383015(v=vs.85).aspx) \(英文\) 連線到 VM 並登入。 RDP 工作階段中透過網路傳輸的資料可由 TLS 保護。

您也可以使用遠端桌面連線到 Azure 中的 Linux VM。

### <a name="secure-access-to-linux-vms-with-ssh"></a>使用 SSH 對 Linux VM 進行安全存取

針對遠端管理，您可以使用[安全殼層](../virtual-machines/linux/ssh-from-windows.md) (SSH) 連線到在 Azure 中執行的 Linux VM。 SSH 是允許透過不安全連線進行安全登入的已加密連線通訊協定。 它是 Azure 中裝載之 Linux VM 的預設連線通訊協定。 使用 SSH 金鑰進行驗證時，不需要密碼就可以登入。 SSH 使用公開/私密金鑰組 (非對稱式加密) 進行驗證。

## <a name="azure-vpn-encryption"></a>Azure VPN 加密

您可以透過虛擬私人網路連線到 Azure，以建立安全通道來保護透過網路所傳送資料的隱私權。

### <a name="azure-vpn-gateways"></a>Azure VPN 閘道

您可以使用 [Azure VPN 閘道](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md)，透過公用連線在您的虛擬網路和內部部署位置之間傳送加密的流量，或在虛擬網路之間傳送流量。

站對站 VPN 使用 [IPsec](https://en.wikipedia.org/wiki/IPsec) \(英文\) 進行傳輸加密。 Azure VPN 閘道使用一組預設提案。 您可以設定 Azure VPN 閘道搭配特定密碼編譯演算法和金鑰長度使用自訂 IPsec/IKE 原則，而不是 Azure 預設原則集。

### <a name="point-to-site-vpns"></a>點對站 VPN

點對站 VPN 可讓個別用戶端電腦存取 Azure 虛擬網路。 [安全通訊端通道通訊協定 (SSTP)](https://technet.microsoft.com/library/2007.06.cableguy.aspx) \(英文\) 可用來建立 VPN 通道。 它可以周遊防火牆 (通道會顯示為 HTTPS 連線)。 您可以使用自己的內部公開金鑰基礎結構 (PKI) 根憑證授權單位 (CA)，來取得點對站連線能力。

您可以使用 Azure 入口網站搭配憑證驗證或 PowerShell，來設定可連線到虛擬網路的點對站 VPN 連線。

若要深入了解 Azure 虛擬網路的點對站 VPN 連線詳細資訊，請參閱：

[使用憑證驗證設定虛擬網路的點對站連線：Azure 入口網站](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md) 

[使用憑證驗證設定虛擬網路的點對站連線：PowerShell](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

### <a name="site-to-site-vpns"></a>站對站 VPN 

您可以使用站對站 VPN 閘道連線，透過 IPsec/IKE (IKEv1 或 IKEv2) VPN 通道，將內部部署網路連線到 Azure 虛擬網路。 此類型的連線需要內部部署 VPN 裝置，且該裝置必須已獲指派面向外部的公用 IP 位址。

您可以使用 Azure 入口網站、PowerShell 或 Azure CLI，來設定虛擬網路的站對站 VPN 連線。

如需詳細資訊，請參閱：

[在 Azure 入口網站中建立站對站連線](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)

[在 PowerShell 中建立站對站連線](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)

[使用 CLI 建立具有站對站 VPN 連線的虛擬網路](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)

## <a name="in-transit-encryption-in-data-lake"></a>Data Lake 中的傳輸中加密

傳輸中的資料 (也稱為移動中的資料) 也一律會在 Data Lake Store 中加密。 除了在將資料儲存至持續性媒體之前加密資料，也一律會使用 HTTPS 保護傳輸中的資料。 HTTPS 是 Data Lake Store REST 介面支援的唯一通訊協定。

若要深入了解 Data Lake 中的傳輸中資料加密，請參閱 [Data Lake Store 中的資料加密](../data-lake-store/data-lake-store-encryption.md)。

## <a name="key-management-with-key-vault"></a>使用 Key Vault 的金鑰管理

若未適當保護及管理金鑰，加密會變成沒有用。 Key Vault 是 Microsoft 建議使用的解決方案，可用來管理及控制雲端服務所使用加密金鑰的存取權。 存取金鑰的權限可透過 Azure Active Directory 帳戶指派給服務或使用者。

Key Vault 讓組織不需要設定、修補及維護硬體安全模組 (HSM) 和金鑰管理軟體。 當您使用 Key Vault 時，將會保有控制權。 Microsoft 永遠不會看到您的金鑰，而且應用程式無法直接存取它們。 您也可以在 HSM 中匯入或產生金鑰。

## <a name="next-steps"></a>後續步驟

- [Azure 安全性概觀](security-get-started-overview.md)
- [Azure 網路安全性概觀](security-network-overview.md)
- [Azure 資料庫安全性概觀](azure-database-security-overview.md)
- [Azure 虛擬機器安全性概觀](security-virtual-machines-overview.md)
- [待用資料加密](azure-security-encryption-atrest.md)
- [資料安全性與加密的最佳做法](azure-security-data-encryption-best-practices.md)
