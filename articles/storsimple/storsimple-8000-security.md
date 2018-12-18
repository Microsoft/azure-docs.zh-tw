---
title: StorSimple 8000 系列安全性 |Microsoft Docs
description: 說明保護內部部署和雲端中之 StorSimple 服務、裝置和資料的安全性和隱私權功能。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 05/18/2018
ms.author: alkohli
ms.openlocfilehash: 578fb3f8bfe68ccd9bbade0ad04f3a811a249c08
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37906384"
---
# <a name="storsimple-security-and-data-protection"></a>StorSimple 安全性和資料保護

## <a name="overview"></a>概觀

安全性是任何人在採用新技術時的主要考量，尤其是當技術用於機密或專屬資料時。 評估不同的技術時，您必須考慮因為資料保護所增加的風險和成本。 Microsoft Azure StorSimple 提供資料保護的安全性和隱私權解決方案，以協助確保：

* **機密性** – 只有已授權的實體才能檢視您的資料。
* **完整性** – 只有已授權的實體才能修改或刪除您的資料。

Microsoft Azure StorSimple 解決方案包含四個彼此互動的主要元件：

* **在 Microsoft Azure 中託管的 StorSimple 裝置管理員服務** – 可用來設定和佈建 StorSimple 裝置的管理服務。
* **StorSimple 裝置** – 可安裝在您的資料中心的實體裝置。 所有產生資料的主機和用戶端都會連接至 StorSimple 裝置，此裝置會管理資料，並將它依適當情況移至 Azure 雲端。
* **連接至裝置的用戶端/主機** – 在基礎結構中，連接至 StorSimple 裝置並產生需要被保護的資料的用戶端。
* **雲端儲存體** – Azure 雲端中儲存資料的位置。

下列各節說明可協助保護各個元件及其中資料的 StorSimple 安全性功能。 它還包含您可能會遇到的 Microsoft Azure StorSimple 安全性問題清單及其對應解答。

## <a name="storsimple-device-manager-service-protection"></a>StorSimple 裝置管理員服務保護

StorSimple 裝置管理員服務是裝載於 Microsoft Azure 的管理服務，可用來管理您組織所採購的所有 StorSimple 裝置。 您可以使用您的組織認證，透過網頁瀏覽器登入 Azure 入口網站來存取 StorSimple 裝置管理員服務。

若要存取 StorSimple 裝置管理員服務，您的組織需有內含 StorSimple 的 Azure 訂用帳戶。 您的訂用帳戶控管您在 Azure 入口網站中可存取的功能。 如果您的組織沒有 Azure 訂用帳戶，但您想要了解更多相關資訊，請參閱 [以組織身分註冊 Azure](../active-directory/fundamentals/sign-up-organization.md)。

因為 StorSimple 裝置管理員服務裝載於 Azure 中，所以會受到 Azure 安全性功能的保護。 如需有關 Microsoft Azure 所提供的安全性功能的詳細資訊，請移至 [Microsoft Azure 信任中心](https://azure.microsoft.com/support/trust-center/security/)。

## <a name="storsimple-device-protection"></a>StorSimple 裝置保護

StorSimple 裝置是包含固態硬碟 (SSD) 和硬碟 (HDD) 的內部部署混合式存放裝置，提供備援控制器和自動容錯移轉功能。 控制器可管理儲存體分層、將目前使用的 (或熱) 資料放在本機儲存體 (在 StorSimple 裝置或在內部部署伺服器中)，而將不常使用的資料移到雲端。

只有已授權的 StorSimple 裝置才能加入您在 Azure 訂用帳戶中建立的 StorSimple 裝置管理員服務。 若要授權裝置，您必須提供服務註冊金鑰才能向 StorSimple 裝置管理員服務註冊該裝置。 服務註冊金鑰是在 Azure 入口網站中產生的 128 位元隨機金鑰。

![服務註冊金鑰](./media/storsimple-security/ServiceRegistrationKey.png)

若要了解如何取得服務註冊金鑰，請移至 [步驟 2：取得服務註冊金鑰](storsimple-8000-deployment-walkthrough-u2.md#step-2-get-the-service-registration-key)。

服務註冊金鑰是包含 100 個以上字元的長金鑰。 您可以複製金鑰並將它以文字檔方式儲存在安全的位置中，如有必要，您可以使用此金鑰來授權其他裝置。 如果您在註冊完第一個裝置之後遺失服務註冊金鑰，可以透過 StorSimple 裝置管理員服務產生新的金鑰。 這不會對現有裝置的作業有任何影響。

裝置註冊完後，它會使用權杖與 Microsoft Azure 通訊。 裝置註冊後便不會用到服務註冊金鑰。

> [!NOTE]
> 建議您在每次使用服務註冊金鑰之後，都重新產生該金鑰。


## <a name="protect-your-storsimple-solution-via-passwords"></a>透過密碼保護您的 StorSimple 解決方案

密碼是電腦安全性重要的一環，且廣泛用於 StorSimple 解決方案，可協助確保只有已獲授權的使用者才能存取您的資料。 StorSimple 可讓您設定下列密碼：

* StorSimple 裝置系統管理員密碼
* Challenge Handshake 驗證通訊協定 (CHAP) 啟動器和目標密碼
* StorSimple Snapshot Manager 密碼

### <a name="windows-powershell-for-storsimple-and-the-storsimple-device-administrator-password"></a>Windows PowerShell for StorSimple 和 StorSimple 裝置系統管理員密碼

Windows PowerShell for StorSimple 是一個可讓您管理 StorSimple 裝置的命令列介面。 Windows PowerShell for StorSimple 的功能包括：可讓您註冊您的裝置、在您的裝置上設定網路介面、安裝特定類型的更新，以及透過存取支援工作階段及變更裝置狀態來疑難排解您的裝置。 藉由連線至裝置上的序列主控台或使用 Windows PowerShell 遠端處理，您可以存取 Windows PowerShell for StorSimple。

您可以透過 HTTPS 或 HTTP 執行 PowerShell 遠端處理。 如果已啟用透過 HTTPS 進行遠端管理，則您必須從裝置下載遠端管理憑證，並將它安裝在遠端用戶端。 如需有關 PowerShell 遠端執行功能的詳細資訊，請移至 [遠端連接至 StorSimple 裝置](storsimple-8000-remote-connect.md)。

使用 Windows PowerShell for StorSimple 連接至裝置之後，您必須提供裝置系統管理員密碼才能登入裝置。

![裝置系統管理員密碼](./media/storsimple-security/DeviceAdminPW.png)

請記住下列最佳做法：

* 遠端管理依預設為關閉。 您可以使用 StorSimple 裝置管理員服務將其啟用。 最佳的安全性做法是，您應該只在真正需要遠端存取的期間才將它啟用。
* 如果您變更密碼，請務必通知所有遠端存取使用者，他們才不會遇到非預期的連線中斷。
* StorSimple 裝置管理員服務無法擷取現有的密碼，只能重設密碼。 建議您將所有密碼儲存在安全的地方，讓您在忘記密碼時無需重設密碼。 如果您需要重設密碼，重設之前請務必通知所有使用者。

您可以使用裝置的序列連接存取 Windows PowerShell 介面。 您也可以使用提供額外安全性的 HTTP 或 HTTPS 遠端存取該介面。 HTTPS 提供比序列或 HTTP 連線更高層級的安全性。 不過，若要使用 HTTPS，您必須首先在將存取裝置的用戶端電腦上安裝憑證。 您可以從 StorSimple 裝置管理員服務中的 [裝置設定] 頁面下載遠端存取憑證。 如果遠端存取的憑證遺失，則您必須下載新的憑證，並將它散佈到所有授權使用遠端管理的用戶端。

### <a name="challenge-handshake-authentication-protocol-chap-initiator-and-target-passwords"></a>Challenge Handshake 驗證通訊協定 (CHAP) 啟動器和目標密碼

CHAP 是 StorSimple 裝置用來驗證遠端用戶端身分識別的一種驗證配置。 此驗證會以共用密碼為基礎。 CHAP 可以是單向 (單向) 或相互 (雙向)。 使用單向 CHAP 時，目標 (StorSimple 裝置) 會驗證啟動器 (主機)。 相互或反相 CHAP 會要求目標驗證啟動器，然後啟動器驗證目標。 您可以設定 StorSimple 使用任一種方法。

設定 CHAP 時，請注意下列事項：

* CHAP 使用者名稱不得超過 233 個字元。
* CHAP 密碼必須介於 12 到 16 個字元。 嘗試使用較長的使用者名稱或密碼將會導致 Windows 主機上發生驗證錯誤。
* 您不能針對 CHAP 啟動器和 CHAP 目標使用相同的密碼。
* 設定密碼之後，您可以加以變更但無法擷取。 如果密碼已變更，請務必通知所有遠端存取使用者，好讓他們能夠順利連接至 StorSimple 裝置。

如需有關 CHAP 及如何為 StorSimple 解決方案設定 CHAP 的詳細資訊，請移至 [為 StorSimple 裝置設定 CHAP](storsimple-8000-configure-chap.md)。

### <a name="storsimple-snapshot-manager-password"></a>StorSimple Snapshot Manager 密碼

StorSimple Snapshot Manager 是一個 Microsoft Management Console (MMC) 嵌入式管理單元，可使用磁碟區群組和 Windows 磁碟區陰影複製服務產生應用程式一致備份。 此外，您可以使用 StorSimple Snapshot Manager 建立備份排程及複製或還原磁碟區。

將裝置設定成使用 StorSimple Snapshot Manager 時，您將必須提供 StorSimple Snapshot Manager 密碼。 此密碼最初是在 Windows PowerShell for StorSimple 中於註冊時設定。 您也可以從 StorSimple 裝置管理員服務來設定及變更此密碼。 此密碼可使用 StorSimple Snapshot Manager 驗證裝置。

![StorSimple Snapshot Manager 密碼](./media/storsimple-security/SnapshotMgrPassword.png)

StorSimple Snapshot Manager 密碼必須是 14 到 15 個字元，且必須包含 3 個以上的大寫、小寫、數字和特殊字元的組合。 設定 StorSimple Snapshot Manager 密碼之後，您可以加以變更但無法進行擷取。 如果您變更密碼，請務必通知所有遠端使用者。

如需有關 StorSimple Snapshot Manager 的詳細資訊，請移至 [什麼是 StorSimple Snapshot Manager？](storsimple-what-is-snapshot-manager.md)

### <a name="password-best-practices"></a>密碼最佳作法

建議您使用下列指導方針，以協助確保 StorSimple 密碼強度夠強並且受到嚴密保護：

* 每三個月變更您的密碼。 每年會強制變更密碼。
* 使用強式密碼。 如需詳細資訊，請移至 [建立強式密碼並保護它們](http://blogs.microsoft.com/cybertrust/2014/08/25/create-stronger-passwords-and-protect-them/)。
* 針對不同的存取機制一律使用不同的密碼。您所指定的每個密碼都應該是唯一的。
* 請勿與未經授權存取 StorSimple 裝置的任何人分享密碼。
* 請勿在其他人面前談論密碼或提示密碼的格式。
* 如果您懷疑帳戶或密碼被盜，請向資訊安全部門回報此事件。
* 將所有密碼視為敏感的機密資訊。 

## <a name="storsimple-data-protection"></a>StorSimple 資料保護

本節說明可保護傳輸中和已儲存之資料的 StorSimple 安全性功能。

如其他小節所述，使用密碼來授權和驗證使用者之後，他們才能存取您的 StorSimple 解決方案。 另一個安全性考量是，在儲存系統之間傳輸以及儲存資料時，如何保護資料以防止未經授權使用者進行存取。 下列各節說明 StorSimple 所提供的資料保護功能。

> [!NOTE]
> 針對儲存在 StorSimple 裝置和 Microsoft Azure 儲存體中的資料，重複資料刪除提供額外的保護。 刪除重複資料時，資料物件會與用來對應和存取這些資料物件的中繼資料分開儲存：沒有可用的儲存層級內容可用來根據磁碟區結構、檔案系統或檔案名稱重新建構資料。


## <a name="protect-data-flowing-through-the-service"></a>保護流經服務的資料

StorSimple 裝置管理員服務的主要目的是管理和設定 StorSimple 裝置。 StorSimple 裝置管理員服務可在 Microsoft Azure 中執行。 您可以使用 Azure 入口網站來輸入裝置組態資料，接著 Microsoft Azure 會使用 StorSimple 裝置管理員服務將資料傳送到裝置。 StorSimple 使用非對稱金鑰組的系統，可協助確保即使 Azure 服務遭到入侵，也不會導致儲存的資訊洩漏。

![傳輸中的資料加密](./media/storsimple-security/DataEncryption.png)

非對稱金鑰系統可協助保護流經服務的資料，如下所示：

1. 使用非對稱公開和私用金鑰組的資料加密憑證會在裝置上產生，並用來保護資料的安全。 註冊第一個裝置時即會產生金鑰。
2. 資料加密憑證金鑰會匯出成為受服務資料加密金鑰 (也就是強式 128 位元金鑰，會在註冊期間由第一部裝置隨機產生) 保護的個人資訊交換 (.pfx) 檔案。
3. 憑證的公開金鑰會以安全的方式提供給 StorSimple 裝置管理員服務，而私密金鑰仍屬裝置所有。
4. 輸入服務的資料會使用公開金鑰進行加密，並使用儲存在裝置上的私密金鑰進行解密，以確保 Azure 服務無法對流向裝置的資料進行解密。

只有在第一個向服務註冊的裝置上，才會產生服務資料加密金鑰。 向服務註冊的所有後續裝置則必須使用相同的服務資料加密金鑰。

> [!IMPORTANT]
> 請務必建立一份服務資料加密金鑰副本，並將其儲存在安全的位置。 儲存服務資料加密金鑰的副本時，必須讓已經授權的人員可以進行存取，並可以輕鬆地將它傳送到裝置系統管理員。
> 
> 如果服務資料加密金鑰遺失，Microsoft 支援人員可協助您擷取該金鑰，但前提是您至少要有一個裝置處於線上狀態。 建議您在擷取服務資料加密金鑰後將其變更。

若要變更服務資料加密金鑰和對應的資料加密憑證，請依照[針對 StorSimple 裝置管理員服務變更服務資料加密金鑰](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)中的步驟進行。 變更加密金鑰時，必須使用新的金鑰更新所有裝置。 因此，建議您在所有裝置都在線上時變更金鑰。 如果裝置處於離線狀態，您可以在其他時間變更金鑰。 金鑰已過期的裝置仍然可以執行備份，但在金鑰更新之前將無法還原資料。

服務資料加密金鑰和資料加密憑證不會過期。 不過，建議您每年變更服務資料加密金鑰，以防金鑰外洩。

## <a name="protect-data-at-rest"></a>保護靜態資料的安全

StorSimple 裝置會根據使用頻率，將資料儲存在本機階層和雲端中加以管理。 連線到裝置的所有主機電腦會將資料都傳送到裝置，然後裝置會視需要將資料移至雲端。 裝置中的資料會經由網際網路安全地傳輸到雲端。 每個裝置都會有一個可瀏覽其上所有共用磁碟區的 iSCSI 目標。 所有資料在傳送至雲端儲存體之前都會進行加密。 

![雲端儲存體加密金鑰](./media/storsimple-security/CloudStorageEncryption.png)

為了協助確保移至雲端之資料的安全性和完整性，StorSimple 可讓您定義雲端儲存體加密金鑰，如以下所示：

* 建立磁碟區容器時，您可以指定雲端儲存體加密金鑰。 您無法修改或稍後新增金鑰。
* 磁碟區容器中的所有磁碟區會共用相同的加密金鑰。 如果您要對特定磁碟區使用不同加密形式，建議您建立新的磁碟區容器來裝載該磁碟區。
* 在 StorSimple 裝置管理員服務中輸入雲端儲存體加密金鑰時，此金鑰會使用服務資料加密金鑰的公開部分進行加密，然後傳送至裝置。
* 雲端儲存體加密金鑰不會儲存在服務中的任何地方，只有此裝置才知道儲存位置。
* 指定雲端儲存體加密金鑰是選用選項。 您可以將已在主機上加密的資料傳送至裝置。

### <a name="additional-security-best-practices"></a>其他的安全性最佳作法

* 分割流量：透過部署完全分開的網路並在無法使用實體隔離的位置使用 VLAN，將您的 iSCSI SAN 從公司 LAN 中的使用者流量隔離。 iSCSI 存放裝置的專用網路會保證您業務關鍵資料的安全性和效能。 不建議在公司 LAN 上將存放裝置與使用者流量混用，這會增加延遲並造成網路失敗。
* 針對主機端的網路安全性，請使用支援 TCP/IP 卸載引擎 (TOE) 的網路介面。 TOE 透過在網路介面卡上處理 TCP 來減少 CPU 的負載。

## <a name="protect-data-via-storage-accounts"></a>透過儲存體帳戶保護資料安全

每個 Microsoft Azure 訂閱可以建立一或多個儲存體帳戶。 (儲存體帳戶會提供唯一的命名空間，以供儲存在 Azure 雲端中的資料使用)。儲存體帳戶的存取權受到與該儲存體帳戶相關聯的訂用帳戶和存取金鑰控制。

建立儲存體帳戶時，Microsoft Azure 會產生兩個 512 位元儲存體存取金鑰，當 StorSimple 裝置存取儲存體帳戶時，可以使用其中一個進行驗證。 請注意，這些金鑰中只有一個會是使用中狀態。 其他金鑰會被保留，讓您可以定期輪替金鑰。 若要輪替金鑰，您必須將次要金鑰的狀態設定為使用中，然後刪除主要金鑰。 然後，您可以建立要在下一個輪替期間使用的新金鑰。 (基於安全性理由，許多資料中心需要金鑰輪替)。

建議您按照這些最佳作法進行金鑰輪替：

* 請時常輪替儲存體帳戶金鑰，以確保未經授權的使用者無法存取您的儲存體帳戶。
* Azure 系統管理員應該使用 Azure 入口網站的 [儲存體] 區段來直接存取儲存體帳戶，以定期變更或重新產生主要或次要金鑰。

## <a name="protect-data-via-encryption"></a>透過加密保護資料安全

StorSimple 會使用下列加密演算法，來保護儲存在 StorSimple 解決方案中或在 StorSimple 解決方案元件之間流動的資料。

| 演算法 | 金鑰長度 | 通訊協定/應用程式/註解 |
| --- | --- | --- |
| RSA |2048 |Azure 入口網站會使用 RSA PKCS 1 v1.5 來加密傳送至裝置的組態資料：例如儲存體帳戶認證、StorSimple 裝置組態，以及雲端儲存體加密金鑰。 |
| AES |256 |AES 搭配 CBC 可用來先加密服務資料加密金鑰的公開部分，然後才將該金鑰從 StorSimple 裝置傳送到 Azure 入口網站。 StorSimple 裝置也會用它來加密資料，之後才將資料傳送至雲端儲存體帳戶。 |

## <a name="storsimple-cloud-appliance-security"></a>StorSimple 雲端設備安全性

[!INCLUDE [storsimple Cloud Appliance security](../../includes/storsimple-virtual-device-security.md)]

## <a name="managing-personal-inforamation"></a>管理個人資訊

實體和虛擬系列的 StorSimple 裝置管理員會收集下列主要執行個體中的個人資訊：

- 警示使用者設定，其中會設定使用者的電子郵件地址。 系統管理員可以檢視與清除此資訊。 這同時適用於 StorSimple 8000 系列裝置與 StorSimple 虛擬陣列。
 * 若要檢視及清除 StorSimple 8000 系列的設定，請遵循[檢視及管理 StorSimple 警示](storsimple-8000-manage-alerts.md#configure-alert-settings)中的步驟
 * 若要檢視及清除 StorSimple 虛擬陣列的設定，請遵循[檢視及管理 StorSimple 警示](storsimple-virtual-array-manage-alerts.md#configure-alert-settings)中的步驟
- 可以存取位於共用上之資料的使用者。 顯示可存取共用資料之使用者的清單，並且可檢視。 刪除共用時，也會一併刪除此清單。 這僅適用於 StorSimple 虛擬陣列。
 * 若要檢視可存取或刪除共用之使用者的清單，請遵循[管理 StorSimple 虛擬陣列上的共用](storsimple-virtual-array-manage-shares.md)中的步驟

如需詳細資訊，請在[信任中心](https://www.microsoft.com/trustcenter)檢閱 Microsoft 隱私權原則。

## <a name="frequently-asked-questions-faq"></a>常見問題集 (FAQ)

下面是一些有關安全性與 Microsoft Azure StorSimple 的問題和解答。

**問：** 我的服務遭到入侵。 我接下來該怎麼做？

**答：** 您應該立即變更服務資料加密金鑰，以及將資料分層時所使用之儲存體帳戶的儲存體帳戶金鑰。 如需相關指示，請移至：

* [變更服務資料加密金鑰](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)
* [儲存體帳戶的金鑰輪替](storsimple-8000-manage-storage-accounts.md#key-rotation-of-storage-accounts)

**問：** 新的 StorSimple 裝置要求我提供服務註冊金鑰。 該如何擷取此金鑰？

**答：** 當您最初建立 StorSimple 裝置管理員服務時已建立此金鑰。 使用 StorSimple 裝置管理員服務連線至裝置時，您可以透過快速啟動頁面來檢視或重新產生服務註冊金鑰。 產生新的服務註冊金鑰不會影響現有的已註冊裝置。 如需相關指示，請移至：

* [檢視或重新產生服務註冊金鑰](storsimple-8000-manage-service.md##regenerate-the-service-registration-key)

**問：** 我遺失服務資料加密金鑰。 該怎麼辦？

**答：** 請連絡「Microsoft 支援服務」。 他們可以登入您裝置上的支援工作階段，協助您擷取金鑰 (假設至少一部裝置在線)。 您取得服務資料加密金鑰之後，請立即變更，以確保只有您自己知道新的金鑰。 如需相關指示，請移至：

* [變更服務資料加密金鑰](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)

**問：** 我已授權裝置進行服務資料加密金鑰變更，但未啟動金鑰變更程序。 我該怎麼辦？

**答：** 如果逾時期間已到期，您將需要重新授權裝置進行服務資料加密金鑰變更，然後重新啟動此程序。

**問：** 我已變更服務資料加密金鑰，但無法在 4 小時內更新其他裝置。 是否必須重新啟動？

**答：** 這 4 小時的期限僅針對起始變更。 在已經授權的 StorSimple 裝置上啟動更新程序之後，在所有裝置更新之前此授權都是有效的。

**問：** 我們的 StorSimple 系統管理員已離職。 我該怎麼辦？

**答：** 變更並重設 StorSimple 裝置的存取密碼，並且變更服務資料加密金鑰，以確保未經授權的人員不知道新的資訊。 如需相關指示，請移至：

* [使用 StorSimple 裝置管理員服務變更您的 StorSimple 密碼](storsimple-8000-change-passwords.md)
* [變更服務資料加密金鑰](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)
* [為 StorSimple 裝置設定 CHAP](storsimple-8000-configure-chap.md)

**問：** 我想要提供 StorSimple Snapshot Manager 密碼給連接至 StorSimple 裝置的主機，但無法取得密碼。 我該怎麼辦？

**答：** 如果忘記密碼，您應該建立新密碼。 然後，請務必通知所有現有的使用者密碼已變更，並要求他們更新其用戶端以使用新密碼。 如需相關指示，請移至：

* [變更 StorSimple Snapshot Manager 密碼](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password)
* [驗證裝置](storsimple-snapshot-manager-manage-devices.md#authenticate-a-device)

**問：** 裝置上用於遠端存取 Windows PowerShell for StorSimple 的憑證已變更。 如何更新遠端存取用戶端？

**答：** 您可以從 StorSimple 裝置管理員服務下載新的憑證，然後將它安裝在遠端存取用戶端的憑證存放區。 如需相關指示，請移至：

* [Import-Certificate cmdlet](https://docs.microsoft.com/powershell/module/pkiclient/import-certificate)

**問：** 如果 StorSimple 裝置管理員服務遭到入侵，我的資料是否仍受保護？

**答：** 在網頁瀏覽器中檢視服務組態資料時，一律會使用您的公開金鑰將它加密。 因為服務無私密金鑰的存取權，所以無法看到任何資料。 StorSimple 裝置管理員服務遭到入侵也不會有任何影響，因為 StorSimple 裝置管理員服務中不會儲存任何金鑰。

**問：** 如果有人取得資料加密憑證的存取權，我的資料會外洩嗎？

**答：** Microsoft Azure 會以加密格式儲存客戶的資料加密金鑰 (.pfx 檔案)。 因為 .pfx 檔案已加密，且 StorSimple 服務沒有服務資料加密金鑰可解密 .pfx 檔案，所以只是存取 .pfx 檔案並不會曝露任何機密。

**問：** 如果政府機構向 Microsoft 索取我的資料，會發生什麼情況？

**答：** 由於所有資料在服務上都已加密，而私密金鑰是與裝置存放在一起，因此政府機構必須向客戶索取資料。



## <a name="next-steps"></a>後續步驟

[部署您的 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。

