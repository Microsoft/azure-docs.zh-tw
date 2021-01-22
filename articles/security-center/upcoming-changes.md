---
title: 未來 Azure 資訊安全中心的重要變更
description: 您需要留意並提早規劃的各項 Azure 資訊安全中心變更
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/21/2021
ms.author: memildin
ms.openlocfilehash: 6d656dbb3fb50091fc8721ba40431b37f77c4ad6
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98662942"
---
# <a name="important-upcoming-changes-to-azure-security-center"></a>未來 Azure 資訊安全中心的重要變更

> [!IMPORTANT]
> 本頁中載明的資訊與預先發行產品或功能相關，且內容可能在公開上市之前修改。 Microsoft 對於此處提供的資訊不做任何明示或暗示的承諾或保證。

本頁內容可帶您了解為資訊安全中心規劃的各項變更， 並說明規劃的產品修改，且可能影響您的安全分數或工作流程等項目。

如果要尋找 6 個月之前的版本資訊，請前往[Azure 資訊安全中心的新功能](release-notes.md)。


## <a name="planned-changes"></a>規劃的變更

- [「套用系統更新」安全性控制已被取代的兩個建議](#two-recommendations-from-apply-system-updates-security-control-being-deprecated)
- [SQL 資料分類建議的增強功能](#enhancements-to-sql-data-classification-recommendation)
- [已新增 35 個預覽建議，以增加 Azure 安全性效能評定的涵蓋範圍](#35-preview-recommendations-being-added-to-increase-coverage-of-azure-security-benchmark)


### <a name="two-recommendations-from-apply-system-updates-security-control-being-deprecated"></a>「套用系統更新」安全性控制已被取代的兩個建議 

**變更的預估日期：** 2021年2月

下列兩項建議已排程在2021年2月淘汰：

- **您的機器應重新開機以套用系統更新**。 這可能會對您的安全分數造成些許影響。
- **監視代理程式應該安裝在您的電腦上**。 此建議僅適用于內部部署機器，而其中的一些邏輯將會轉移至另一個建議，而 **Log Analytics 代理程式健康情況問題應在您的電腦上解決**。 這可能會對您的安全分數造成些許影響。

建議您檢查連續匯出和工作流程自動化設定，以查看這些建議是否包含在其中。 此外，任何可能使用這些儀表板的儀表板或其他監視工具，都應隨之更新。

請在 [安全性建議參考頁面](recommendations-reference.md)中深入瞭解這些建議。


### <a name="enhancements-to-sql-data-classification-recommendation"></a>SQL 資料分類建議的增強功能

**變更的預估日期：** 2021季

**您 SQL 資料庫中的建議機密資料** 的最新版本應該分類為「套用 **資料分類**」安全性控制項將被取代，並以與 Microsoft 資料分類策略更一致的新版本取代。 因此：

- 建議將不再影響您的安全分數
- 安全性控制 ( [套用資料分類] ) 將不再影響您的安全分數
- 建議的識別碼也會變更 (目前 b0df6f56-862d-4730-8597-38c0fd4ebd59) 


### <a name="35-preview-recommendations-being-added-to-increase-coverage-of-azure-security-benchmark"></a>將新增 35 個預覽建議，以增加 Azure 安全性效能評定的涵蓋範圍

**變更的預估日期：** 2021 年 1 月

Azure 安全性效能評定是 Microsoft 針對以通用合規性架構為基礎的安全性和合規性最佳做法所撰寫的一組 Azure 特定指導方針。 [深入了解 Azure 安全性效能評定](../security/benchmarks/introduction.md)。

下列 35 個預覽建議將新增到 Azure 資訊安全中心，以增加 Azure 安全性效能評定的涵蓋範圍。

預覽建議不會將資源轉譯為「狀況不良」，這些項目也不會納入您安全分數的計算範圍。 您可以盡可能加以補救，以在預覽期間結束時計入您的分數。 若要了解如何回應這些建議，請參閱[Azure 資訊安全中心的補救建議](security-center-remediate-recommendations.md)。

| 安全性控制                     | 新建議                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 啟用待用加密            | - Azure Cosmos DB 帳戶應使用客戶自控金鑰加密待用資料<br>- Azure Machine Learning 工作區應使用客戶自控金鑰 (CMK) 進行加密<br>- 應為 MySQL 伺服器啟用自備金鑰資料保護<br>- 應為 PostgreSQL 伺服器啟用自備金鑰資料保護<br>- 認知服務帳戶應使用客戶自控金鑰 (CMK) 來啟用資料加密<br>- 應使用客戶自控金鑰 (CMK) 進行加密<br>- SQL 受控執行個體應使用客戶自控金鑰來加密待用資料<br>- SQL 伺服器應使用客戶自控金鑰來加密待用資料<br>- 儲存體帳戶應使用客戶自控金鑰 (CMK) 進行加密                                                                                                                                                              |
| 實作安全性最佳做法    | - 訂用帳戶應具有連絡人電子郵件地址以處理安全性問題<br> - 您的訂用帳戶上應啟用 Log Analytics 代理程式的自動化佈建<br> - 應針對高嚴重性警示啟用電子郵件通知<br> - 應已針對高嚴重性警示啟用傳送給訂用帳戶擁有者的電子郵件通知<br> - 金鑰保存庫應啟用清除保護<br> - 金鑰保存庫應已啟用虛刪除 |
| 管理存取權和權限        | - 函數應用程式應啟用「用戶端憑證 (傳入用戶端憑證)」 |
| 保護應用程式防範 DDoS 攻擊 | - 應為應用程式閘道啟用 Web 應用程式防火牆 (WAF)<br> - 應為 Azure Front Door Service 啟用 Web 應用程式防火牆 (WAF) |
| 限制未經授權的網路存取 | - 應該在 Key Vault 上啟用防火牆<br> - 應設定 Key Vault 的私人端點<br> - 應用程式設定應使用私人連結<br> - Azure Cache for Redis 應位於虛擬網路內<br> - Azure 事件方格網域應使用私人連結<br> - Azure 事件方格主題應使用私人連結<br> - Azure Machine Learning 工作區應使用私人連結<br> - Azure SignalR Service 應使用私人連結<br> - Azure Spring Cloud 應使用網路插入<br> - 不應允許不受限制的網路存取<br> - 容器登錄應使用私人連結<br> - 應為 MariaDB 伺服器停用公用網路存取<br> - 應為 MySQL 伺服器停用公用網路存取<br> - 應為 PostgreSQL 伺服器停用公用網路存取<br> - 儲存體帳戶應使用私人連結連線<br> - 儲存體帳戶應使用虛擬網路規則來限制網路存取<br> - VM 映像產生器範本應使用私人連結|
|                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

相關連結：

- [深入了解 Azure 安全性效能評定](../security/benchmarks/introduction.md)
- [深入了解適用於 MariaDB 的 Azure 資料庫](../mariadb/overview.md)
- [深入了解適用於 MySQL 的 Azure 資料庫](../mysql/overview.md)
- [深入了解適用於 PostgreSQL 的 Azure 資料庫](../postgresql/overview.md)





## <a name="next-steps"></a>後續步驟

如需產品的所有最近變更，請參閱 [Azure 資訊安全中心有哪些新功能？](release-notes.md)。