---
title: 待用資料的 Azure 儲存體服務加密 | Microsoft Docs
description: 使用「Azure 儲存體服務加密」功能，在儲存資料時，於服務端加密 Azure 受控磁碟、Azure Blob 儲存體、Azure 檔案服務、Azure 佇列儲存體及 Azure 資料表儲存體，然後在擷取資料時使用它。
services: storage
author: lakasa
ms.service: storage
ms.topic: article
ms.date: 08/01/2018
ms.author: lakasa
ms.component: common
ms.openlocfilehash: c6001b0c95186dff53f33c73833b4cea30688db2
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44094887"
---
# <a name="azure-storage-service-encryption-for-data-at-rest"></a>待用資料的 Azure 儲存體服務加密
「待用資料的 Azure 儲存體服務加密」可協助您保護資料安全，以符合組織的安全性和合規性承諾。 使用此功能時，Azure 儲存體平台會先自動加密資料，再將資料保存到 Azure 受控磁碟；Azure Blob、佇列或資料表儲存體；或 Azure 檔案，然後在擷取資料之前將資料解密。 對使用者而言，「儲存體服務加密」中的加密、待用加密、解密及金鑰管理作業都是在背景中進行而看不見的。 所有寫入至 Azure 儲存體平台的資料都會透過 256 位元 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(可用的最強大區塊編碼器之一\) 進行加密。

系統會針對所有新的和現有的儲存體帳戶啟用「儲存體服務加密」且無法停用。 由於預設會保護您的資料，因此無須修改您的程式碼或應用程式，即可利用「儲存體服務加密」。

此功能會自動加密下列環境中的資料：

- Azure 儲存體服務：
    - Azure 受控磁碟
    - Azure Blob 儲存體
    - Azure 檔案
    - Azure 佇列儲存體
    - Azure 資料表儲存體。  
- 兩個效能層 (標準和進階)。
- 兩個部署模型 (Azure Resource Manager 和傳統)。

「儲存體服務加密」不會影響 Azure 儲存體服務的效能。

您可以搭配「儲存體服務加密」使用 Microsoft 管理的加密金鑰，或是使用自己的加密金鑰。 如需有關使用您自己金鑰的詳細資訊，請參閱[使用 Azure Key Vault 中客戶管理的金鑰進行儲存體服務加密](storage-service-encryption-customer-managed-keys.md)。

## <a name="view-encryption-settings-in-the-azure-portal"></a>在 Azure 入口網站中檢視加密設定
若要檢視「儲存體服務加密」的設定，請登入 [Azure 入口網站](https://portal.azure.com)並選取儲存體帳戶。 在 [設定] 窗格中，選取 [加密] 設定。

![顯示 [加密] 設定的入口網站螢幕擷取畫面](./media/storage-service-encryption/image1.png)

## <a name="faq-for-storage-service-encryption"></a>儲存體服務加密的常見問題集
**如何加密 Resource Manager 儲存體帳戶中的資料？**  
所有儲存體帳戶 (傳統和 Resource Manager) 都會啟用「儲存體服務加密」，背景加密程序將採用回溯方式，為啟用加密之前在儲存體帳戶中建立的所有現有檔案加密。

**建立儲存體帳戶時，是否會預設啟用「儲存體服務加密」？**  
是，所有儲存體帳戶和所有 Azure 儲存體服務都會啟用「儲存體服務加密」。

**我有 Resource Manager 儲存體帳戶。是否可以在該帳戶上啟用「儲存體服務加密」？**  
預設會在所有現有的 Resource Manager 儲存體帳戶上啟用「儲存體服務加密」。 針對 Azure Blob 儲存體、Azure 檔案服務、Azure 佇列儲存體及資料表儲存體，都提供此支援。 

**是否可以在我的儲存體帳戶上停用加密？**  
預設會啟用加密，且沒有任何佈建可停用您儲存體帳戶的加密。 

**如果啟用「儲存體服務加密」，Azure 儲存體的成本會增加多少？**  
沒有任何額外成本。

**我可以使用自己的加密金鑰嗎？**  
對於 Azure Blob 儲存體和 Azure 檔案服務，您可以使用自己的加密金鑰。 Azure 受控磁碟目前不支援客戶管理的金鑰。 如需詳細資訊，請參閱[使用 Azure Key Vault 中客戶管理的金鑰進行儲存體服務加密](storage-service-encryption-customer-managed-keys.md)。

**是否可以撤銷加密金鑰的存取權？**  
是，如果您在 Azure Key Vault 中[使用自己的加密金鑰](storage-service-encryption-customer-managed-keys.md)。

**儲存體服務加密和 Azure 磁碟加密有何不同？**  
Azure 磁碟加密提供 OS 型解決方案 (例如 BitLocker 及 Dm-crypt) 與 Azure KeyVault 之間的整合。 儲存體服務加密可在虛擬機器下的 Azure 儲存體平台層提供原生加密。

**我有傳統儲存體帳戶。是否可以在該帳戶上啟用「儲存體服務加密」？**  
所有儲存體帳戶 (傳統和 Resource Manager) 都會啟用「儲存體服務加密」。

**我要如何在我的傳統儲存體帳戶中加密資料？**  
因為加密功能預設為啟用，所以 Azure 儲存體服務中儲存的任何資料都會自動加密。 

**是否可以使用 Azure PowerShell 和 Azure CLI 來建立儲存體帳戶並啟用「儲存體服務加密」？**  
建立任何儲存體帳戶 (傳統或 Resource Manager) 時，預設都會啟用「儲存體服務加密」。 您可以使用 Azure PowerShell 和 Azure CLI 來確認帳戶屬性。

**我的儲存體帳戶設定為異地備援複寫。使用「儲存體服務加密」時，是否也會加密我的備援複本？**  
是，會加密儲存體帳戶的所有複本。 支援所有備援選項 -- 本地備援儲存體、區域備援儲存體、異地備援儲存體，以及讀取權限異地備援儲存體。

**是否只有在特定區域才允許使用「儲存體服務加密」？**  
「儲存體服務加密」可供所有區域使用。

**儲存體服務加密是否符合 FIPS 140-2 規範？**  
是，儲存體服務加密符合 FIPS 140-2 規範。

**如果我有任何問題或想要提供意見反應，要如何連絡相關人員？**  
如果有「儲存體服務加密」的任何相關問題或意見反應，請連絡 [ssediscussions@microsoft.com](mailto:ssediscussions@microsoft.com)。

## <a name="next-steps"></a>後續步驟
「Azure 儲存體」提供一組完整的安全性功能，這些功能一起可協助開發人員建置安全的應用程式。 如需詳細資料，請參閱[儲存體安全性指南](../storage-security-guide.md)。
