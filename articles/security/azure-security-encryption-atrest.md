---
title: Microsoft Azure 資料靜態加密 | Microsoft Docs
description: 此文章提供 Microsoft Azure 資料靜態加密概觀、整體功能及一般考量。
services: security
documentationcenter: na
author: barclayn
manager: mbaldwin
editor: TomSh
ms.assetid: 9dcb190e-e534-4787-bf82-8ce73bf47dba
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/20/2018
ms.author: barclayn
ms.openlocfilehash: 71351a979e8bb443864c975b91539c527bea7f2a
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033422"
---
# <a name="azure-data-encryption-at-rest"></a>Azure 資料靜態加密

Microsoft Azure 內有數項工具，可根據公司的安全性和合規性需求來保護資料。 此文章著重於：

- 在整個 Microsoft Azure 中保護資料的方式
- 討論參與資料保護實作的各種元件，
- 檢閱不同金鑰管理保護方法的優缺點。 

靜態加密是常見的安全性需求。 在 Azure 中，組織不需要實作和管理的成本，以及自訂金鑰管理解決方案的風險，就可以達到待用加密。 組織可以選擇讓 Azure 完全管理靜態加密。 此外，組織擁有各種選項能密切管理加密或加密金鑰。

## <a name="what-is-encryption-at-rest"></a>什麼是靜態加密？

待用加密就是將存留的資料進行編碼 (加密)。 Azure 中的靜態加密設計是使用對稱加密，根據簡單概念模型，快速將大量資料加密及解密：

- 對稱式加密金鑰可用來在寫入儲存體時將資料加密。 
- 當資料準備好在記憶體中使用時，相同的加密金鑰可用來將資料解密。
- 資料可能會進行分割，每個分割區可能會使用不同的金鑰。
- 金鑰必須儲存在安全的位置，並包含限制存取某些身分識別並記錄金鑰使用方法的存取控制原則。 資料加密金鑰通常會使用非對稱式加密進行加密，以進一步限制存取。

在實務上，金鑰管理和控制情節，以及級別和可用性保證，都需要其他建構。 以下將說明 Microsoft Azure 靜態加密的概念和元件。

## <a name="the-purpose-of-encryption-at-rest"></a>靜態加密的目的

待用加密可為儲存的資料 (待用) 提供資料保護。 對靜態資料的攻擊包含嘗試取得儲存資料的硬體之實體存取，並洩露內含的資料。 在這類攻擊中，伺服器的硬碟可能已在維護期間受到錯誤的處理，讓攻擊者能夠移除硬碟。 稍後，攻擊者會將硬碟放入受他們控制下的電腦，以嘗試存取資料。 

靜態加密旨在防止攻擊者存取未加密的資料，方法是確保資料在磁碟上時就已加密。 如果攻擊者取得了使用這類加密資料的硬碟，但無法存取加密金鑰，攻擊者就難以輕易洩漏資料。 在這類情況中，攻擊者必須嘗試攻擊加密的資料，而存取這類資料也比存取硬碟上未加密的資料更複雜且耗用更多資源。 基於這個理由，強烈建議您使用靜態加密，且對許多組織而言是高優先順序的需求。 

組織致力於資料治理及合規性之際，也可能需要使用待用加密。 產業和政府規定 (例如 HIPAA、PCI 和 FedRAMP 等) 就資料保護和加密需求制定了具體的保護措施。 待用加密是符合這其中部分規範所需的必要手段。

除了合規性和法規要求，靜態加密應該視為深度防禦的平台功能。 雖然 Microsoft 會提供服務、應用程式和資料的符合規範平台、全方位的設施和實體安全性、資料存取控制與稽核，但請務必提供其他「重疊」的安全性措施，以防其中一個其他安全性量值失敗。 靜態加密會提供這類額外的防禦機制。

Microsoft 致力於提供跨雲端服務的靜態加密選項，以及為提供客戶適用的加密金鑰管理性，並可存取會顯示何時使用加密金鑰的記錄。 此外，Microsoft 也朝向讓所有客戶資料預設為靜態加密的目標。

## <a name="azure-encryption-at-rest-components"></a>Azure 靜態加密元件

如先前所述，靜態加密的目的，是使用祕密加密金鑰將保存在磁碟上的資料進行加密。 若要達到這個目標，就必須提供安全的金鑰建立、儲存、存取控制，以及加密金鑰管理。 儘管細節可能有所差異，但 Azure 服務「待用加密」實作能以下圖所示的術語來說明。

![元件](./media/azure-security-encryption-atrest/azure-security-encryption-atrest-fig1.png)

### <a name="azure-key-vault"></a>Azure 金鑰保存庫

加密金鑰的儲存位置以及這些金鑰的存取控制是靜態加密模型的核心。 金鑰必須是高度安全的，但需可由指定使用者進行管理，且可用於特定服務。 針對 Azure 服務，Azure Key Vault 是建議的金鑰儲存體解決方案，並提供常見的跨服務管理體驗。 金鑰是儲存在金鑰保存庫並在其中管理，可以將存取金鑰保存庫提供給使用者或服務。 Azure Key Vault 支援客戶建立金鑰或匯入客戶金鑰，可供在客戶管理的加密金鑰情節下使用。

### <a name="azure-active-directory"></a>Azure Active Directory

使用 Azure Key Vault 中儲存的金鑰加以管理或存取，從而進行靜態加密及靜態解密的權限，可以提供給 Azure Active Directory 帳戶。 

### <a name="key-hierarchy"></a>金鑰階層

在靜態加密實作中，會使用一個以上的加密金鑰。 非對稱加密可用於建立金鑰存取和管理所需的信任和驗證。 對稱式加密對於大量加密和解密效率較高，允許更強式的加密和更佳的效能。 限制使用單一加密金鑰會降低金鑰洩露的風險，以及在必須更換金鑰時的重新加密成本。 Azure 待用加密模型使用下列金鑰類型組成的金鑰階層：

- **資料加密金鑰 (DEK)** – 用於將分割區或資料區塊加密的對稱 AES256 金鑰。  單一資源可能會有多個分割區和多個資料加密金鑰。 使用不同金鑰將每個資料區塊進行加密，會使密碼編譯分析攻擊更加困難。 資源提供者或要將特定區塊加密和解密的應用程式執行個體都需要存取 DEK。 當新的金鑰取代 DEK 時，只有在相關聯區塊中的資料才需要使用新的金鑰重新加密。
- **金鑰加密金鑰 (KEK)** – 用來將資料加密金鑰加密的非對稱式加密金鑰。 使用金鑰加密金鑰可讓資料加密金鑰本身進行加密和控制。 可存取 KEK 的實體可能不同於需要 DEK 的實體。 實體可以代理存取 DEK，以限制每個 DEK 只能由特定分割區存取。 因為需要 KEK 才能將 DEK 解密，KEK 實際上就是單一點，藉以透過刪除 KEK 來有效地刪除 DEK。

使用金鑰加密金鑰加密的資料加密金鑰會分別儲存，只有可存取金鑰加密金鑰的實體能取得任何使用該金鑰加密的資料加密金鑰。 支援不同模型的金鑰儲存。 我們稍後將在下一節更深入討論每個模型。

## <a name="data-encryption-models"></a>資料加密模型

若要了解 Azure 中的各種資源提供者如何實作待用加密，就務必了解各種加密模式及其優缺點。 這些定義會跨 Azure 中的所有資源提供者進行共用，以確保為通用的語言和分類法。 

伺服器端加密有三種情節：

- 使用服務管理金鑰的伺服器端加密
    - Azure 資源提供者執行加密和解密作業
    - Microsoft 管理金鑰
    - 完整的雲端功能

- 在 Azure Key Vault 中使用客戶管理金鑰的伺服器端加密
    - Azure 資源提供者執行加密和解密作業
    - 客戶透過 Azure Key Vault 控制金鑰
    - 完整的雲端功能

- 在客戶控制的硬體上使用客戶管理金鑰的伺服器端加密
    - Azure 資源提供者執行加密和解密作業
    - 客戶在客戶控制的硬體上控制金鑰
    - 完整的雲端功能

針對用戶端加密，請考慮下列各項：

- Azure 服務無法查看解密的資料
- 客戶在內部部署環境 (或其他安全的儲存區中) 管理或儲存金鑰。 金鑰無法供 Azure 服務使用
- 縮減的雲端功能

Azure 中支援的模型如先前所述區分成兩個主要群組：「用戶端加密」和「伺服器端加密」。 請注意，使用了獨立的加密靜態模型，Azure 服務一律建議使用諸如 TLS 或 HTTPS 等安全的傳輸。 因此，傳輸中的加密應該由傳輸通訊協定來處理，且不應作為要使用哪個靜態加密模型的主要判斷因素。

### <a name="client-encryption-model"></a>用戶端加密模型

用戶端加密模型是指由服務或呼叫應用程式在資源提供者或 Azure 外部執行的加密。 加密可由 Azure 中的服務應用程式或客戶資料中心內執行的應用程式執行。 在任一案例中，利用此加密模型時，Azure 資源提供者無需以任何方式解密的能力或具有加密金鑰的存取權，即可接收加密的 blob 資料。 在此模型中，金鑰管理是由呼叫服務/應用程式所完成，且對 Azure 服務不透明。

![用戶端](./media/azure-security-encryption-atrest/azure-security-encryption-atrest-fig2.png)

### <a name="server-side-encryption-model"></a>伺服器端加密模型

伺服器端加密模型是指 Azure 服務所執行的加密。 在這個模型中，資源提供者會執行加密和解密作業。 例如，Azure 儲存體可能會在純文字作業中接收資料，並在內部執行加密和解密。 資源提供者可能會使用由 Microsoft 或客戶管理的加密金鑰，根據提供的設定而定。 

![伺服器](./media/azure-security-encryption-atrest/azure-security-encryption-atrest-fig3.png)

### <a name="server-side-encryption-key-management-models"></a>伺服器端加密金鑰管理模型

每個伺服器端靜態加密模型都表示金鑰管理的特殊特性。 這包括加密金鑰所建立及儲存的位置和方式，以及存取模型和金鑰輪替程序。 

#### <a name="server-side-encryption-using-service-managed-keys"></a>使用服務管理金鑰的伺服器端加密

對許多客戶而言，基本需求就是確保在靜態時會將資料加密。 使用服務管理金鑰的伺服器端加密會啟用這個模型，方法是允許客戶標示特定的資源 (儲存體帳戶、SQL DB 等) 以進行加密，並將諸如金鑰發佈、輪替和備份等所有金鑰管理層面保留給 Microsoft。 大部分支援靜態加密的 Azure 服務通常會支援這個將加密金鑰管理卸載至 Azure 的模型。 Azure 資源提供者會建立金鑰、將它們放置在安全的儲存體，並在需要時擷取它們。 這表示服務具有金鑰的完整存取權，且服務可完整控制認證生命週期管理。

![受控](./media/azure-security-encryption-atrest/azure-security-encryption-atrest-fig4.png)

伺服器端加密使用管理服務的金鑰，因此能快速解決為客戶提供具有低額外負荷之靜態加密的需求。 情況允許時，客戶通常會開啟目標訂用帳戶和資源提供者的 Azure 入口網站，並勾選方塊以指示要加密資料。 在部分 Resource Manager 中，使用服務管理金鑰的伺服器端加密預設為開啟。

使用 Microsoft 管理金鑰的伺服器端加密表示服務可完整存取儲存及管理金鑰。 雖然部分客戶會因為認為可獲得更高的安全性而要管理金鑰，但在評估此模型時，應該考慮自訂金鑰儲存解決方案的相關成本和風險。 在許多情況下，組織可能會判斷資源限制或內部部署解決方案的風險可能會大於在雲端管理靜態加密金鑰的風險。  不過，對於需要控制加密金鑰的建立或生命週期，或是使用與管理服務不同的人員來管理服務的加密金鑰 (也就是，將金鑰管理與服務的整體管理模型隔離) 的組織而言，此模型可能並不足夠。

##### <a name="key-access"></a>金鑰存取

使用服務管理金鑰的伺服器端加密時，金鑰建立、儲存和存取服務都是由服務所管理。 一般而言，基本的 Azure 資源提供者會將資料加密金鑰儲存在接近資料、快速可用且可存取的儲存區中，而金鑰加密金鑰則是儲存在安全的內部儲存區中。

**優點**

- 簡單設定
- Microsoft 會管理金鑰輪替、備份與備援
- 客戶沒有與實作相關聯的成本，或自訂金鑰管理配置的風險。

**缺點**

- 沒有加密金鑰 (金鑰規格、生命週期、撤銷等) 的客戶控制權
- 無法從服務的整體管理模型將金鑰管理隔離

#### <a name="server-side-encryption-using-customer-managed-keys-in-azure-key-vault"></a>在 Azure Key Vault 中使用客戶管理的金鑰的伺服器端加密 

在需要將靜態資料加密並控制加密金鑰的情節中，客戶可以透過 Key Vault 中的客戶管理金鑰來使用伺服器端加密。 某些服務可能只會在 Azure Key Vault 中儲存根金鑰加密金鑰，並將加密的資料加密金鑰儲存在較接近資料的內部位置。 在該案例中，客戶可以將自己的金鑰帶到 Key Vault (BYOK – 自備金鑰) 或產生新的金鑰，並使用它們來加密所需的資源。 資源提供者執行加密和解密作業時，它會使用設定的金鑰作為所有加密作業的根金鑰。 

##### <a name="key-access"></a>金鑰存取

在 Azure Key Vault 中使用客戶管理金鑰的伺服器端加密模型所牽涉的服務，會視需要存取金鑰來加密和解密。 透過存取控制原則，服務可以存取靜態加密金鑰。 此原則授與服務身分識別存取，以接收金鑰。 代表相關聯的訂用帳戶所執行的 Azure 服務，可以使用該訂用帳戶中的身分識別來進行設定。 服務可以執行 Azure Active Directory 驗證，並接收驗證權杖，會將其本身識別為代表訂用帳戶的該服務。 接著會將該權杖提供給 Key Vault，從而取得已獲得存取權的金鑰。

針對使用加密金鑰的作業，可以授與服務識別存取任何下列作業：解密、加密、解除包裝金鑰、包裝金鑰、驗證、簽署、取得、列出、更新、建立、匯入、刪除、備份和還原。

若要取得用於將靜態資料加密或解密的金鑰，Resource Manager 服務執行個體所要執行的服務識別必須擁有「解除包裝金鑰」(可取得解密金鑰) 和「包裝金鑰」(建立新的金鑰時，可將金鑰插入保存庫金鑰)。


>[!NOTE] 
>如需關於 Key Vault 授權的詳細資訊，請參閱 [Azure Key Vault 文件](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault)中的「保護您的金鑰保存庫」頁面。 

**優點**

- 完整控制所使用的金鑰 – 加密金鑰會在客戶控制下的 Key Vault 中進行管理。
- 能夠將多個服務加密到一個主機
- 可從服務的整體管理模型將金鑰管理隔離
- 可跨區域定義服務與金鑰位置

**缺點**

- 客戶對於金鑰存取管理擁有完全責任
- 客戶對於金鑰生命週期管理擁有完全責任
- 安裝與設定的其他額外負荷

#### <a name="server-side-encryption-using-service-managed-keys-in-customer-controlled-hardware"></a>在客戶控制的硬碟上，使用服務管理金鑰的伺服器端加密

某些 Azure 服務可允許「裝載您自己的金鑰」(HYOK) 金鑰管理模型。 當需要加密待用資料並在不受 Microsoft 控制的專屬存放庫中管理金鑰時，便適用此管理模式。 在此模型中，服務必須從外部網站擷取金鑰。 效能和可用性保證會受到影響，且設定會更為複雜。 此外，因為服務在加密和解密作業期間可存取 DEK，此模型的整體安全性保證會與客戶在 Azure Key Vault 中管理金鑰時類似。  因此，此模型不適用於大部分的組織，除非組織有特定金鑰管理需求。 由於這些限制，大部分的 Azure 服務不支援在客戶所控制的硬體中使用伺服器管理金鑰的伺服器端加密。

##### <a name="key-access"></a>金鑰存取

在客戶所控制的硬體中使用服務管理金鑰的伺服器端加密時，金鑰會保留在客戶設定的系統上。 支援這個模型的 Azure 服務可提供方法，來建立與客戶提供之金鑰儲存區的安全連線。

**優點**

- 完整控制所使用的根金鑰 – 金鑰加密會由客戶提供的儲存區進行管理
- 能夠將多個服務加密到一個主機
- 可從服務的整體管理模型將金鑰管理隔離
- 可跨區域定義服務與金鑰位置

**缺點**

- 對於金鑰儲存、安全性、效能和可用性擁有完全責任
- 對於金鑰存取管理擁有完全責任
- 對於金鑰生命週期管理擁有完全責任
- 安裝、設定和持續維護的成本高昂
- 客戶資料中心與 Azure 資料中心之間網路可用性的相依性增加。

## <a name="encryption-at-rest-in-microsoft-cloud-services"></a>Microsoft 雲端服務中的靜態加密

這三種雲端模型中全都是使用 Microsoft 雲端服務：IaaS、PaaS、SaaS。 下列範例說明它們如何符合每個模型：

- 軟體服務是指「軟體即服務」或 SaaS，具有雲端所提供的應用程式，例如 Office 365。
- 平台服務是指客戶在其應用程式中利用雲端，會針對諸如儲存、分析和服務匯流排功能等項目使用雲端。
- 基礎結構服務或基礎結構即服務 (IaaS) 可供客戶在其中部署裝載於雲端中且可能運用其他雲端服務的作業系統和應用程式。

### <a name="encryption-at-rest-for-saas-customers"></a>SaaS 客戶的靜態加密

軟體即服務 (SaaS) 客戶通常會啟用或可在每個服務中使用靜態加密。 Office 365 有幾個選項可供客戶確認或啟用靜態加密。 如需 Office 365 服務的相關資訊，請參閱 Office 365 的資料加密技術。

### <a name="encryption-at-rest-for-paas-customers"></a>PaaS 客戶的靜態加密

平台即服務 (PaaS) 客戶的資料通常是位於應用程式執行環境，以及用來儲存客戶資料的任何 Azure 資源提供者。 若要查看您可使用的靜態加密選項，請檢查下表以查看您所使用的儲存體和應用程式平台。 在支援的位置中，會為每個資源提供者提供啟用靜態加密指示的連結。 

### <a name="encryption-at-rest-for-iaas-customers"></a>IaaS 客戶的靜態加密

基礎結構即服務 (IaaS) 客戶可以有各種不同的使用中服務和應用程式。 IaaS 服務可以在其 Azure 裝載的虛擬機器和使用 Azure 磁碟加密的 VHD 中啟用靜態加密。 

#### <a name="encrypted-storage"></a>加密的儲存體

諸如 PaaS、IaaS 等解決方案可以運用儲存資料靜態加密的其他 Azure 服務。 在這些情況下，您可以啟用每個已使用 Azure 服務所提供的靜態加密支援。 下表列舉出主要儲存體、服務和應用程式平台，以及支援的待用加密模型。 在支援的位置中，會為啟用靜態加密的指示提供連結。 

#### <a name="encrypted-compute"></a>加密的計算

完整的靜態加密中，不得以未加密的形式來保存資料。 使用時，在記憶體中載入資料的伺服器上，會以各種方式在本機上保留資料，包括 Windows 分頁檔、損毀傾印及應用程式可能會執行的任何記錄。 若要確保此資料會進行靜態加密，IaaS 應用程式可以在 Azure IaaS 虛擬機器 (Windows 或 Linux) 和虛擬磁碟上使用 Azure 磁碟加密。 

#### <a name="custom-encryption-at-rest"></a>自訂靜態加密

建議您盡可能讓 IaaS 應用程式運用任何已使用 Azure 服務所提供的 Azure 磁碟加密及靜態加密選項。 在某些情況下，例如異常加密需求或非以 Azure 為基礎的儲存體，IaaS 應用程式的開發人員可能需要自行實作靜態加密。 IaaS 解決方案的開發人員可以運用某些 Azure 元件，更妥善與 Azure 管理和客戶期望進行整合。 具體來說，開發人員應該使用 Azure Key Vault 服務來提供安全的金鑰儲存體，並為客戶提供一致的金鑰管理選項，以及大部分 Azure 平台服務的選項。 此外，自訂的解決方案應該使用 Azure 受控服務識別，讓服務帳戶可存取加密金鑰。 如需關於 Azure Key Vault 和受控服務身分識別的開發人員資訊，請參閱其個別的 SDK。

## <a name="azure-resource-providers-encryption-model-support"></a>Azure 資源提供者加密模型支援

每個 Microsoft Azure 服務都支援一或多個靜態加密模型。 不過，針對某些服務，一或多個加密模型不可能適用。 支援客戶管理金鑰案例的服務所支援的金鑰類型，可能只有 Azure Key Vault 所支援用於金鑰加密金鑰的一小部分。 此外，這些服務可能會在不同的排程發行這些案例和金鑰類型的支援。 本章節針對每個主要 Azure 資料儲存體服務描述撰寫此文章時的靜態加密支援。

### <a name="azure-disk-encryption"></a>Azure 磁碟加密

任何使用 Azure 基礎結構即服務 (IaaS) 功能的客戶都可透過 Azure 磁碟加密讓其 IaaS VM 和磁碟達到靜態加密。 如需有關 Azure 磁碟加密的資訊，請參閱 [Azure 磁碟加密文件](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)。

#### <a name="azure-storage"></a>Azure 儲存體

所有 Azure 儲存體服務 (Blob 儲存體、佇列儲存體、資料表儲存體和 Azure 檔案) 都支援伺服器端靜態加密，某些服務支援客戶管理的金鑰和用戶端加密。  

- 伺服器端：所有 Azure 儲存體服務預設會使用服務管理的金鑰啟用伺服器端加密，這對應用程式是透明的。 如需詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)。 Azure Blob 儲存體和 Azure 檔案也支援 Azure Key Vault 中的 RSA 2048 位元客戶管理金鑰。 如需詳細資訊，請參閱[使用 Azure Key Vault 中客戶管理的金鑰進行儲存體服務加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption-customer-managed-keys)。
- 用戶端：Azure Blob、資料表和佇列支援用戶端加密。 當您使用用戶端加密時，客戶會將資料加密，並上傳資料做為加密的 blob。 金鑰管理是由客戶完成。 如需詳細資訊，請參閱 [Microsoft Azure 儲存體的用戶端加密和 Azure Key Vault](https://docs.microsoft.com/azure/storage/storage-client-side-encryption)。


#### <a name="azure-sql-database"></a>連接字串

Azure SQL Database 目前支援 Microsoft 受控服務端和用戶端加密等狀況的待用加密。

伺服器加密的支援目前是透過稱為「透明資料加密」的 SQL 功能所提供。 一旦 Azure SQL Database 客戶啟用 TDE 後，就會為他們自動建立和管理金鑰。 可以在資料庫和伺服器等級啟用靜態加密。 自 2017 年 6 月起，[透明資料加密 (TDE)](https://msdn.microsoft.com/library/bb934049.aspx) 依預設會在新建立的資料庫上啟用。 Azure SQL Database 支援 Azure Key Vault 中的 RSA 2048 位元客戶管理金鑰。 如需詳細資訊，請參閱 [Azure SQL Database 和資料倉儲的透明資料加密與攜帶您自己的金鑰支援](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-byok-azure-sql?view=azuresqldb-current)。

透過 [Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx) 功能可支援 Azure SQL Database 資料的用戶端加密。 Always Encrypted 會使用用戶端所建立及儲存的金鑰。 客戶可以將主要金鑰儲存在 Windows 憑證存放區、Azure Key Vault 或硬體安全性模組中。 SQL 使用者可以使用 SQL Server Management Studio 來選擇他們要用來加密資料行的金鑰。

|                                  |                    | **加密模型和金鑰管理** |                   |                    |
|----------------------------------|--------------------|--------------------|--------------------|--------------------|
|                                  | **使用服務管理金鑰的伺服器端**     | **在 Key Vault 中使用客戶管理的伺服器端**             |  **在內部部署中使用客戶管理的伺服器端**                  | **使用用戶端管理的用戶端**      |
| **儲存體和資料庫**        |                    |                    |                    |                    |                    |
| 磁碟 (IaaS)                      | -                  | 是，RSA 2048 位元  | 是               | -                  |
| SQL Server (IaaS)                | 是                | 是，RSA 2048 位元  | 是                | 是                |
| Azure SQL (資料庫/資料倉儲) | 是                | 是，RSA 2048 位元  | -                  | 是                |
| Azure 儲存體 (區塊/分頁 Blob) | 是                | 是，RSA 2048 位元  | -                  | 是                |
| Azure 儲存體 (檔案)            | 是                | 是，RSA 2048 位元  | -                  | -                  |
| Azure 儲存體 (資料表、佇列)   | 是                | -                  | -                  | 是                |
| Cosmos DB (文件 DB)          | 是                | -                  | -                  | -                  |
| StorSimple                       | 是                | -                  | -                  | 是                |
| Backup                            | -                  | -                  | -                  | 是                |
| **智慧和分析**   |                    |                    |                    |                    |
| Azure Data Factory               | 是                | -                  | -                  | -                  |
| Azure Machine Learning           | -                  | 預覽，RSA 2048 位元 | -                  | -                  |
| Azure 串流分析           | 是                | -                  | -                  | -                  |
| HDInsight (Azure Blob 儲存體)   | 是                | -                  | -                  | -                  |
| HDInsight (Data Lake 儲存體)    | 是                | -                  | -                  | -                  |
| Azure Data Lake Store            | 是                | 是，RSA 2048 位元  | -                  | -                  |
| Azure 資料目錄               | 是                | -                  | -                  | -                  |
| Power BI                         | 是                | -                  | -                  | -                  |
| **IoT 服務**                 |                    |                    |                    |                    |
| IoT 中樞                          | -                  | -                  | -                  | 是                |
| 服務匯流排                      | 是                | -                  | -                  | 是                |
| 事件中樞                       | 是                | -                  | -                  | -                  |


## <a name="conclusion"></a>結論

對於 Microsoft 而言，保護儲存在 Azure 服務內的客戶資料是至關重要的。 所有的 Azure 託管服務都致力於提供靜態加密選項。 諸如 Azure 儲存體、Azure SQL Database 與金鑰分析和智慧服務等基本的服務，都已經提供待用加密選項。 這些服務有部分會支援客戶控制的金鑰和用戶端加密，以及服務管理的金鑰和加密。 Microsoft Azure 服務正廣泛提高靜態加密可用性，新的選項已規劃提供預覽，並即將於幾個月內公開上市。
