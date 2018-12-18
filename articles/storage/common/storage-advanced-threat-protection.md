---
title: 監視 Azure 儲存體中的威脅
description: 設定 Azure 儲存體進階威脅防護來偵測帳戶活動中的異常狀況，並在對您帳戶所進行的嘗試存取可能有害時通知您。
services: storage
author: rmatchoro
ms.service: storage
ms.topic: article
ms.date: 09/24/2018
ms.author: ronmat
ms.manager: shaik
ms.openlocfilehash: 8b2ca2d5d6418d68cab847df80fc437e468249ed
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995632"
---
# <a name="azure-storage-advanced-threat-protection"></a>Azure 儲存體進階威脅防護

Azure 儲存體進階威脅防護會偵測帳戶活動中的異常狀況，並在對您帳戶所進行的嘗試存取可能有害時通知您。 這個防護層讓您不需是安全性專家或管理安全性監視系統，就能解決威脅。

威脅會經由定義在活動中發生異常狀況時所觸發的安全性警示而浮現出來。 這些警示會與 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)整合，其中包括可疑活動的詳細資料，以及如何調查威脅並進行補救的建議。 

> [!NOTE]
> Azure 儲存體進階威脅防護目前僅適用於 Blob 服務。 安全性警示會與 Azure 資訊安全中心整合，並透過電子郵件傳送給訂用帳戶管理員。

Azure 儲存體進階威脅防護會將讀取、寫入和刪除要求的診斷記錄內嵌至 Blob 服務，以用來進行威脅偵測。 若要從進階威脅防護調查警示，您需要[設定診斷記錄](storage-monitor-storage-account.md#configure-logging)來啟用適用於 Blob 服務的所有記錄層級。

## <a name="set-up-advanced-threat-protection-in-the-portal"></a>在入口網站上設定進階威脅防護

1. 在 [https://portal.azure.com](https://portal.azure.com/) 上啟動 Azure 入口網站。

2. 瀏覽至您要保護之 Azure 儲存體帳戶的設定頁面。 在 [設定] 頁面中，選取 [進階威脅防護]。

3. 在 [進階威脅防護] 設定刀鋒視窗中
    * **開啟**進階「威脅防護」
    * 按一下 [儲存]，以儲存新的或更新的進階威脅防護原則。

![開啟 Azure 儲存體進階威脅防護](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-turn-on.png)

## <a name="explore-anomalies"></a>探索異常狀況

發生儲存體活動異常狀況時，您會收到電子郵件通知，其中包含可疑安全性事件的相關資訊。 事件的詳細資料包括：

* 異常狀況的性質
* 儲存體帳戶名稱
* 儲存體類型
* 事件時間

該電子郵件還會提供關於可能原因和建議動作的詳細資料，以協助您調查並減輕潛在威脅。

![Azure 儲存體進階威脅防護警示電子郵件](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-alert-email.png)

您可以從 Azure 資訊安全中心的[安全性警示圖格](../../security-center/security-center-managing-and-responding-alerts.md#managing-security-alerts)，檢閱與管理目前的安全性警示。 按一下特定警示，即會提供適合用來調查目前威脅及解決未來威脅的詳細資料與動作。

![Azure 儲存體進階威脅防護警示電子郵件](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-alert.png)

## <a name="protection-alerts"></a>防護警示

警示是因為以不尋常且有危害意圖的方式嘗試存取或惡意探索儲存體帳戶而產生的。 這些事件會觸發下列警示：

* **從不尋常的位置存取**：當儲存體帳戶的存取模式變更時，就會觸發此警示。 例如，有人從不尋常的地理位置存取了儲存體帳戶時。 在某些情況下，警示會偵測到合法的動作 (新的應用程式或開發人員維護作業)。 在其他情況下，警示則會偵測惡意動作 (離職員工、外部攻擊者等)。

* **不尋常的資料擷取**：當儲存體帳戶的資料擷取模式變更時，就會觸發此警示。 例如，如果有人在儲存體帳戶中存取了不尋常的資料數量。 在某些情況下，警示會偵測合法的動作 (維護活動)。 在其他情況下，警示則會偵測惡意動作 (資料外洩/缺口、未經授權的資料傳輸)。

* **不尋常的匿名存取**：當儲存體帳戶的存取模式變更時，就會觸發此警示。 例如，假設有人以匿名方式存取了儲存體帳戶。 在某些情況下，警示會偵測使用公用讀取權限的合法存取。 在其他情況下，警示則會偵測未經授權的存取，該存取會惡意探索對容器及其 Blob 的公用讀取權限。

* **未預期的刪除**：根據儲存體帳戶的歷程記錄分析，當儲存體帳戶中發生一或多個未預期的刪除作業時，就會觸發此警示。 例如，假設有人使用新的應用程式並從新的 IP 位址執行了 *DeleteBlob* 作業。 在某些情況下，警示會偵測合法的動作 (系統管理員在商務旅行時會使用不同的瀏覽器)。 在其他情況下，警示則會偵測惡意動作 (攻擊者刪除資料)。 
 
* **存取權限變更**：當儲存體帳戶的存取權限發生未預期的變更時，就會觸發此警示。 例如，假設有人使用新的應用程式並從新的 IP 位址變更了儲存體帳戶的存取權限。 在某些情況下，警示會偵測合法的動作 (系統管理員在商務旅行時會使用不同的瀏覽器)。 在其他情況下，警示則會偵測惡意動作 (例如，攻擊者提高他們已獲存取權之帳戶的權限)。 

* **上傳 Azure 雲端服務封裝**：若以非預期方式將 Azure 雲端服務封裝 (*.cspkg* 檔案) 上傳到儲存體帳戶，就會觸發此警示。 例如，假設 *.cspkg* 檔案是從新的 IP 位址上傳的。 在某些情況下，警示會偵測合法的動作。 在其他情況下，警示則會偵測惡意動作 (例如，已上傳雲端服務封裝來準備部署惡意服務)。    
   

## <a name="next-steps"></a>後續步驟

* 深入了解[登入 Azure 儲存體帳戶](/rest/api/storageservices/About-Storage-Analytics-Logging)

* 深入了解 [Azure 資訊安全中心](../../security-center/security-center-intro.md)