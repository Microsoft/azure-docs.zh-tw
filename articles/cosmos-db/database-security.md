---
title: 資料庫安全性 - Azure Cosmos DB | Microsoft Docs
description: 了解 Azure Cosmos DB 如何為您的資料提供資料庫保護和資料安全性。
keywords: nosql 資料庫安全性, 資訊安全性, 資料安全性, 資料庫加密, 資料庫保護, 安全性原則, 安全性測試
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 11/15/2017
ms.author: rafats
ms.openlocfilehash: d74aa6b7ef762b7971fd67654e7fc1a23a8d83cc
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037990"
---
# <a name="azure-cosmos-db-database-security"></a>Azure Cosmos DB 資料庫安全性

本文討論 Azure Cosmos DB 提供的資料庫安全性最佳作法和重要功能，協助您防止、偵測及因應資料庫缺口。
 
## <a name="whats-new-in-azure-cosmos-db-security"></a>Azure Cosmos DB 安全性有哪些新功能？

儲存在所有 Azure 區域之 Azure Cosmos DB 中的文件和備份現在可以使用待用加密。 這些區域內的新舊客戶都會自動套用加密靜止功能。 無須設定任何項目；您即可獲得跟之前相同的出色延遲、輸送量、可用性及功能，並擁有以下好處：知道資料在加密靜止下仍舊安全無虞。

## <a name="how-do-i-secure-my-database"></a>如何保護我的資料庫？ 

資料安全性是您、客戶和資料庫提供者之間共同的責任。 根據您選擇的資料庫提供者而定，您承擔的責任也有所不同。 如果您選擇內部部署解決方案，您需要負責從端點保護到硬體實體安全性的一切事項，這並不容易。 如果您選擇 PaaS 雲端資料庫提供者，例如 Azure Cosmos DB，您關切的領域會大幅縮小。 下圖取自 Microsoft 的[雲端運算共同責任 (英文)](https://aka.ms/sharedresponsibility) 技術白皮書，顯示您的責任如何藉助於 PaaS 提供者 (例如 Azure Cosmos DB) 而減輕。

![客戶和資料庫提供者的責任](./media/database-security/nosql-database-security-responsibilities.png)

上圖顯示高階雲端安全性元件，但對於您的資料庫解決方案，您需要特別關注哪些項目？ 還有，您如何比較不同的解決方案？ 

我們建議採用下列需求檢查清單來比較資料庫系統︰

- 網路安全性和防火牆設定
- 使用者驗證和細微的使用者控制
- 能夠將資料複寫到世界各地以防區域性失敗
- 能夠在不同資料中心之間執行容錯移轉
- 在資料中心內的區域資料複寫
- 自動資料備份
- 從備份中還原已刪除的資料
- 保護並隔離機密資料
- 監視攻擊
- 回應攻擊
- 能夠異地隔離資料以遵守資料控管限制
- 實際保護受保護資料中心內的伺服器
- 認證

雖然看似明顯，但最新的[大規模資料庫缺口](http://thehackernews.com/2017/01/mongodb-database-security.html)提醒我們注意以下這些簡單但很重要的需求︰
- 修補伺服器以保持最新狀態
- 預設為 HTTPS/SSL 加密
- 使用強式密碼的系統管理帳戶

## <a name="how-does-azure-cosmos-db-secure-my-database"></a>Azure Cosmos DB 如何保護我的資料庫？

讓我們回顧先前的清單 - Azure Cosmos DB 提供其中多少安全性需求？ 每一個。

讓我們詳細探究每一個。

|安全性需求|Azure Cosmos DB 的安全性方法|
|---|---|---|
|網路安全性|使用 IP 防火牆是保護資料庫的第一道防線。 Azure Cosmos DB 支援原則驅動的 IP 型存取控制，以提供輸入防火牆支援。 IP 型存取控制類似於傳統資料庫系統所使用的防火牆規則，但是功能更詳細，使得 Azure Cosmos DB 資料庫帳戶只能從一組核准的機器或雲端服務存取。 <br><br>Azure Cosmos DB 可讓您啟用特定的 IP 位址 (168.61.48.0)、IP 範圍 (168.61.48.0/8)，以及 IP 和範圍的組合。 <br><br>Azure Cosmos DB 會封鎖此允許清單之外的機器發出的所有要求。 因此，核准的機器和雲端服務發出的要求必須完成驗證程序，才能獲得資源的存取控制。<br><br>深入了解 [Azure Cosmos DB 防火牆支援](firewall-support.md)。|
|Authorization|Azure Cosmos DB 使用雜湊式訊息驗證碼 (HMAC) 來進行授權。 <br><br>每個要求都經過祕密帳戶金鑰的雜湊處理，而後續的 base 64 編碼雜湊會隨著每次呼叫 Azure Cosmos DB 一起傳送。 為了驗證要求，Azure Cosmos DB 服務會使用正確的祕密金鑰和屬性產生雜湊，然後比較此值與要求中的值。 如果兩個值相符，則會成功授權作業，並處理要求，否則授權會失敗，也會拒絕要求。<br><br>您可以使用[主要金鑰](secure-access-to-data.md#master-keys)或[資源權杖](secure-access-to-data.md#resource-tokens)，更細微地控制資源的存取，例如文件。<br><br>請參閱[保護 Azure Cosmos DB 資源的存取](secure-access-to-data.md)以深入了解。|
|使用者和權限|您可以使用帳戶的[主要金鑰](#master-key)，建立每個資料庫的使用者資源和權限資源。 [資源權杖](#resource-token)與資料庫中的權限相關聯，將決定使用者是否能夠存取 (讀寫、唯讀或無法存取) 資料庫中的應用程式資源。 應用程式資源包括容器、文件、附件、預存程序、觸發程序和 UDF。 然後，驗證期間會使用資源權杖來允許或拒絕存取資源。<br><br>請參閱[保護 Azure Cosmos DB 資源的存取](secure-access-to-data.md)以深入了解。|
|Azure 目錄整合 (RBAC)| 您也可以在 Azure 入口網站中使用存取控制 (IAM) 提供資料庫帳戶的存取權，如本表格之後的螢幕擷取畫面所示。 IAM 提供角色型存取控制，並與 Active Directory 整合。 您可以對個人或群組使用內建角色或自訂角色，如下圖所示。|
|全球複寫|Azure Cosmos DB 提供現成的全域散發，您只要按一下按鈕，就能將資料複寫至 Azure 的任何一個全球資料中心。 全球複寫可讓您擴大到全球規模，確保存取世界各地資料時的低延遲性。<br><br>在安全性方面，全域複寫可確保資料不受區域性失敗影響。<br><br>請參閱[將資料分散到全球](distribute-data-globally.md)以深入了解。|
|區域性容錯移轉|如果您已將資料複寫至多個資料中心，萬一某個區域資料中心離線，Azure Cosmos DB 會自動轉移您的作業。 您可以使用已複寫資料的區域，建立容錯移轉區域的優先順序清單。 <br><br>請參閱 [Azure Cosmos DB 的區域性容錯移轉](regional-failover.md)以深入了解。|
|區域複寫|即使在單一資料中心內，Azure Cosmos DB 也會自動複寫資料來達到高可用性，還可讓您選擇[一致性層級](consistency-levels.md)。 如此保證所有單一區域帳戶和所有多重區域帳戶具有鬆散的一致性，可達到 99.99% [可用性 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db)，而所有多重區域資料庫帳戶有 99.999% 的讀取可用性。|
|自動化線上備份|Azure Cosmos DB 資料庫會定期備份，並儲存在異地備援存放區。 <br><br>請參閱[使用 Azure Cosmos DB 進行自動線上備份及還原](online-backup-and-restore.md)以深入了解。|
|還原已刪除的資料|自動化線上備份可用來復原您不小心刪除的資料，最多可還原意外發生後 30 天內的資料。 <br><br>請參閱[使用 Azure Cosmos DB 進行自動線上備份及還原](online-backup-and-restore.md)以深入了解|
|保護並隔離機密資料|在列於[新功能](#whats-new)的區域中，所有資料均已加密靜止。<br><br>個人資料和其他機密資料可隔離至特定容器來進行讀寫，或限制為只對特定使用者提供唯讀存取權。|
|監視攻擊|您可以使用[稽核記錄和活動記錄](logging.md)，以監視帳戶的正常和異常活動。 您可以檢視資源上執行的作業、誰起始作業、作業何時發生、作業的狀態等等，如本表格之後的螢幕擷取畫面所示。|
|回應攻擊|一旦您連絡 Azure 支援來報告潛在的攻擊，就會展開 5 步驟的事件回應程序。 5 步驟程序的目標是在偵測到問題並展開調查之後，儘快恢復正常的服務安全性和作業。<br><br>請參閱[雲端的 Microsoft Azure 安全性回應](https://aka.ms/securityresponsepaper)以深入了解。|
|異地隔離|Azure Cosmos DB 可確保主權區域 (例如，德國、中國、US Gov) 的資料控管。|
|受保護的設施|Azure Cosmos DB 中的資料儲存在 Azure 受保護資料中心內的 SSD 上。<br><br>請參閱 [Microsoft 全球資料中心](https://www.microsoft.com/en-us/cloud-platform/global-datacenters)以深入了解|
|HTTPS/SSL/TLS 加密|用戶端對服務的所有 Azure Cosmos DB 互動都支援 SSL/TLS 1.2。 此外，資料中心內和跨資料中心的所有複寫也都強制使用 SSL/TLS 1.2。|
|待用加密|所有儲存至 Azure Cosmos DB 的資料都會進行待用加密。 若要深入了解，請參閱 [Azure Cosmos DB 待用加密](.\database-encryption-at-rest.md)|
|修補的伺服器|Azure Cosmos DB 是受控資料庫，會自動替您管理和修補伺服器。|
|使用強式密碼的系統管理帳戶|很難相信我們需要提到這個需求，但是與一些競爭者不同，Azure Cosmos DB 中的系統管理帳戶一定會有密碼。<br><br> 依預設已內建透過 SSL 和 HMAC 密碼型驗證的安全性。|
|安全性和資料保護認證|如需最新的認證清單，請參閱整體 (Azure 合規性網站)[https://www.microsoft.com/en-us/trustcenter/compliance/complianceofferings] 以及最新的 (Azure 合規性文件)[https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942]，內含所有認證 (請搜尋 Cosmos)。 如需更深入的資訊，請閱讀 2018 年 8 月 25 日的貼文 (Azure #CosmosDB：安全、私密、符合規範)[https://azure.microsoft.com/blog/azure-cosmosdb-secure-private-compliant/]，其中包含 SOCS 1/2 Type 2、HITRUST, PCI DSS Level 1、ISO 27001、HIPPAA、FedRAMP High 和其他許多資訊。

以下螢幕擷取畫面顯示在 Azure 入口網站中使用存取控制 (IAM) 的 Active directory 整合 (RBAC)：![Azure 入口網站中的存取控制 (IAM) - 示範資料庫安全性](./media/database-security/nosql-database-security-identity-access-management-iam-rbac.png)

以下螢幕擷取畫面顯示如何使用稽核記錄和活動記錄來監視您的帳戶：![Azure Cosmos DB 的活動記錄](./media/database-security/nosql-database-security-application-logging.png)

## <a name="next-steps"></a>後續步驟

如需主要金鑰和資源權杖的詳細資訊，請參閱[保護 Azure Cosmos DB 資料的存取](secure-access-to-data.md)。

如需稽核記錄的詳細資訊，請參閱 [Azure Cosmos DB 診斷記錄](logging.md)。

如需 Microsoft 認證的詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/)。
