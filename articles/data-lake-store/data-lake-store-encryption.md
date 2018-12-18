---
title: Azure Data Lake Store 中的加密 | Microsoft Docs
description: Azure Data Lake Store 中的加密可協助您保護資料，實作企業安全性原則，並符合合規性需求。 本文提供設計概觀，並討論實作的一些技術層面。
services: data-lake-store
documentationcenter: ''
author: esung22
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: yagupta
ms.openlocfilehash: c3f79348cb5d80639d76ad1a8ba82fcfa56ebed1
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43311353"
---
# <a name="encryption-of-data-in-azure-data-lake-store"></a>Azure Data Lake Store 中的資料加密

Azure Data Lake Store 中的加密可協助您保護資料，實作企業安全性原則，並符合合規性需求。 本文提供設計概觀，並討論實作的一些技術層面。

Data Lake Store 同時支援待用資料和傳輸中資料的加密。 對於待用資料，Data Lake Store 支援「預設開啟」、透明加密。 以下是這些詞彙的更詳細說明︰

* **預設開啟**︰建立新的 Data Lake Store 帳戶時，預設設定會啟用加密。 因此，Data Lake Store 中儲存的資料一律會在儲存於持續性媒體之前進行加密。 這是所有資料的行為，建立帳戶之後便無法變更。
* **透明**：Data Lake Store 會在保存資料前自動加密，以及在擷取之前將資料解密。 系統管理員會在 Data Lake Store 層級設定和管理加密。 資料存取 API 不會變更。 因此，因為加密而與 Data Lake Store 互動的應用程式和服務也不需要變更。

傳輸中的資料 (也稱為移動中的資料) 也一律會在 Data Lake Store 中加密。 除了在儲存至持續性媒體之前加密資料，也一律會使用 HTTPS 保護傳輸中的資料。 HTTPS 是 Data Lake Store REST 介面支援的唯一通訊協定。 下圖顯示如何在 Data Lake Store 中將資料加密︰

![Data Lake Store 中的資料加密圖表](./media/data-lake-store-encryption/fig1.png)


## <a name="set-up-encryption-with-data-lake-store"></a>使用 Data Lake Store 設定加密

在帳戶建立期間會設定 Data Lake Store 的加密，且預設永遠啟用。 您可以自行管理金鑰，或允許 Data Lake Store 為您管理 (這是預設值)。

如需詳細資訊，請參閱[開始使用](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal)。

## <a name="how-encryption-works-in-data-lake-store"></a>Data Lake Store 中的加密運作方式

下列資訊涵蓋如何管理主要加密金鑰，並說明您可以使用於 Data Lake Store 資料加密的三種金鑰類型。

### <a name="master-encryption-keys"></a>主要加密金鑰

Data Lake Store 提供兩種管理主要加密金鑰 (MEK) 的模式。 現在，假設主要加密金鑰是最上層金鑰。 需要有主要加密金鑰的存取權，才能將任何儲存在 Data Lake Store 中的資料解密。

管理主要加密金鑰的兩種模式如下︰

*   服務管理的金鑰
*   客戶管理的金鑰

這兩種模式都會將主要加密金鑰儲存在 Azure Key Vault 中進行保護。 Key Vault 是 Azure 上完全受控、高度安全的服務，可用來保護加密金鑰。 如需詳細資訊，請參閱 [Key Vault](https://azure.microsoft.com/services/key-vault)。

以下是兩種管理 MEK 模式所提供的簡潔功能比較。

|  | 服務管理的金鑰 | 客戶管理的金鑰 |
| --- | --- | --- |
|儲存資料的方式|一律在儲存之前加密。|一律在儲存之前加密。|
|主要加密金鑰的儲存位置|Key Vault|Key Vault|
|任何加密金鑰是否會完全儲存在 Key Vault 外部？ |否|否|
|Key Vault 是否可以擷取 MEK？|否。 MEK 儲存於 Key Vault 之後，就只能用來加密和解密。|否。 MEK 儲存於 Key Vault 之後，就只能用來加密和解密。|
|誰擁有 Key Vault 執行個體和 MEK？|Data Lake Store 服務|您擁有 Key Vault 執行個體，其屬於您自己的 Azure 訂用帳戶。 Key Vault 中的 MEK 可以由軟體或硬體管理。|
|您是否可以撤銷 Data Lake Store 服務對於 MEK 的存取權？|否|是。 您可以管理 Key Vault 中的存取控制清單，並移除 Data Lake Store 服務之服務識別的存取控制項目。|
|您是否可以永久刪除 MEK？|否|是。 如果您從 Key Vault 中刪除 MEK，任何人 (包括 Data Lake Store 服務) 都無法將 Data Lake Store 帳戶中的資料解密。 <br><br> 如果從 Key Vault 中刪除 MEK 之前，您已明確進行備份，即可還原 MEK，進而復原資料。 不過，如果從 Key Vault 中刪除 MEK 之前，您尚未進行備份，則 Data Lake Store 帳戶中的資料之後就永遠無法解密。|


除了誰負責管理 MEK 及其所在 Key Vault 執行個體的這項差異，這兩種模式的其餘設計都相同。

選擇主要加密金鑰的模式時，請務必記住下列幾點：

*   您可以在佈建 Data Lake Store 帳戶時，選擇使用客戶管理的金鑰或服務管理的金鑰。
*   佈建 Data Lake Store 帳戶之後，就無法變更模式。

### <a name="encryption-and-decryption-of-data"></a>資料的加密和解密

資料加密設計使用三種金鑰類型。 下表提供摘要：

| Key                   | 縮寫 | 相關聯的項目 | 儲存位置                             | 類型       | 注意                                                                                                   |
|-----------------------|--------------|-----------------|----------------------------------------------|------------|---------------------------------------------------------------------------------------------------------|
| 主要加密金鑰 | MEK          | Data Lake Store 帳戶 | Key Vault                              | 非對稱 | 它可由 Data Lake Store 或您管理。                                                              |
| 資料加密金鑰   | DEK          | Data Lake Store 帳戶 | 永續性儲存體，由 Data Lake Store 服務管理 | 對稱  | DEK 是由 MEK 加密。 已加密的 DEK 會儲存在持續性媒體上。 |
| 區塊加密金鑰  | BEK          | 資料區塊 | None                                         | 對稱  | BEK 衍生自 DEK 和資料區塊。                                                      |

下圖說明這些概念：

![資料加密中的加密](./media/data-lake-store-encryption/fig2.png)

#### <a name="pseudo-algorithm-when-a-file-is-to-be-decrypted"></a>將檔案解密時的虛擬演算法︰
1.  檢查 Data Lake Store 帳戶的 DEK 是否已快取並可供使用。
    - 若非如此，則從永續性儲存體讀取加密的 DEK，並將它傳送至 Key Vault 進行解密。 快取記憶體中已解密的 DEK。 其現在已可供使用。
2.  對於檔案中的每個資料區塊。
    - 從永續性儲存體讀取已加密的資料區塊。
    - 從 DEK 和已加密的資料區塊產生 BEK。
    - 使用 BEK 將資料解密。


#### <a name="pseudo-algorithm-when-a-block-of-data-is-to-be-encrypted"></a>將資料區塊加密時的虛擬演算法︰
1.  檢查 Data Lake Store 帳戶的 DEK 是否已快取並可供使用。
    - 若非如此，則從永續性儲存體讀取加密的 DEK，並將它傳送至 Key Vault 進行解密。 快取記憶體中已解密的 DEK。 其現在已可供使用。
2.  從 DEK 產生資料區塊的唯一 BEK。
3.  經由使用 AES-256 加密的 BEK，將資料區塊加密。
4.  在永續性儲存體上儲存已加密的資料區塊。

> [!NOTE] 
> 無論在持續性媒體上或從記憶體中進行快取，儲存 DEK 時一律會以 MEK 進行加密。

## <a name="key-rotation"></a>金鑰輪替

當您使用客戶受控的金鑰時，您可以輪替 MEK。 若要了解如何使用客戶管理的金鑰來設定 Data Lake Store 帳戶，請參閱[開始使用](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal)。

### <a name="prerequisites"></a>必要條件

當您設定 Data Lake Store 帳戶時，您已選擇使用自己的金鑰。 建立帳戶之後，就無法變更此選項。 下列步驟假設您使用客戶管理的金鑰 (也就是，您已從 Key Vault 中選擇自己的金鑰)。

請注意，如果您使用預設選項進行加密，一律會使用 Data Lake Store 所管理的金鑰將您的資料加密。 使用此選項時，您無法輪替金鑰，因為金鑰是由 Data Lake Store 管理。

### <a name="how-to-rotate-the-mek-in-data-lake-store"></a>如何在 Data Lake Store 中輪替 MEK

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 瀏覽至儲存與您的 Data Lake Store 帳戶相關聯之金鑰的 Key Vault 執行個體。 選取 [金鑰]。

    ![Key Vault 的螢幕擷取畫面](./media/data-lake-store-encryption/keyvault.png)

3.  選取與您的 Data Lake Store 帳戶相關聯的金鑰，並建立此金鑰的新版本。 請注意，Data Lake Store 目前只支援新版金鑰的金鑰輪替。 它不支援不同金鑰的輪替。

   ![[金鑰] 視窗的螢幕擷取畫面 (醒目提示 [新增版本])](./media/data-lake-store-encryption/keynewversion.png)

4.  瀏覽至 Data Lake Store 儲存體帳戶，然後選取 [加密]。

    ![Data Lake Store 儲存體帳戶視窗的螢幕擷取畫面 (醒目提示 [加密])](./media/data-lake-store-encryption/select-encryption.png)

5.  有一則訊息會通知您，新本的金鑰已可使用。 按一下 [輪替金鑰]，將金鑰更新為新的版本。

    ![Data Lake Store 視窗的螢幕擷取畫面 (含有訊息並醒目提示 [輪替金鑰])](./media/data-lake-store-encryption/rotatekey.png)

這項作業應在 2 分鐘內完成，且預計不會因為金鑰輪替而停機。 在作業完成後，就會使用新版的金鑰。

> [!IMPORTANT]
> 金鑰輪替作業完成之後，舊版本的金鑰將不再主動用於加密資料。  不過，在極少數非預期的失敗情況下 (甚至影響到資料的備援複本)，系統可能會從仍使用舊版金鑰的備份中還原資料。 為確保可在這些極少數情況下存取您的資料，請保留舊版加密金鑰的複本。 請參閱 [Data Lake Store 中資料的災害復原指引](data-lake-store-disaster-recovery-guidance.md)，以了解災害復原規劃的最佳作法。 