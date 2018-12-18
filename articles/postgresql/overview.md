---
title: 「適用於 PostgreSQL 的 Azure 資料庫」關聯式資料庫服務的概觀。
description: 提供 Azure Database for PostgreSQL 關聯式資料庫服務的概觀。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 06/15/2018
ms.openlocfilehash: 86a8b824479173024b1a14bb8612073ffbfac468
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47092152"
---
# <a name="what-is-azure-database-for-postgresql"></a>什麼是 Azure Database for PostgreSQL？

適用於 PostgreSQL 的 Azure 資料庫是 Microsoft 雲端中以開放原始碼 [PostgreSQL](https://www.postgresql.org/) 資料庫引擎的社群版為基礎的關聯式資料庫服務，目標對象是開發人員。 Azure Database for PostgreSQL 提供：

- 內建高可用性但沒有任何額外成本
- 可預測的效能，使用預付型方案定價方式
- 視需要在幾秒內進行縮放
- 受到保護，可保護機密的待用資料和移動中資料
- 最多 35 天的自動化備份和時間點還原
- 企業級安全性與合規性

上述所有功能幾乎不需要管理，而且免費提供。 這些功能可讓您專注於快速開發應用程式及加快上市時間，而不是將寶貴的時間和資源耗費在管理虛擬機器和基礎結構上。 此外，您可以使用開放原始碼工具與選擇的平台繼續開發應用程式，並提供企業所需的速度和效率而不需要學習新技能。 

此文章介紹 Azure Database for PostgreSQL 在效能、延展性和管理能力方面的核心概念和功能。 請參閱下列快速入門，以便開始使用產品：

- [使用 Azure 入口網站建立 Azure Database for PostgreSQL](quickstart-create-server-database-portal.md)
- [使用 Azure CLI 建立 Azure Database for PostgreSQL](quickstart-create-server-database-azure-cli.md)

如需一組 Azure CLI 範例，請參閱︰

- [Azure Database for PostgreSQL 的 Azuer CLI 範例](./sample-scripts-azure-cli.md)

## <a name="adjust-performance-and-scale-within-seconds"></a>在幾秒之內即可調整效能和規模
適用於 PostgreSQL 的 Azure 資料庫服務提供了三個定價層：基本、一般用途和記憶體最佳化。 每一層提供不同的資源功能，來支援您的資料庫工作負載。 您可以在小型資料庫中建置第一個應用程式，一個月只需少許花費，然後調整規模以滿足解決方案的需求。 動態延展性可讓您的資料庫以透明的方式回應快速變化的資源需求。 您只需支付您所需的資源費用。 如需詳細資訊，請參閱[定價層](concepts-pricing-tiers.md)。

## <a name="monitoring-and-alerting"></a>監視和警示
如何決定何時應增加或減少？ 您可以使用內建的 Azure 監視和警示功能。 透過這些工具，您可以根據目前或計畫中的效能或儲存體需求，快速評估相應增加或減少的影響。 如需詳細資訊，請參閱[警示](howto-alert-on-metric.md)。

## <a name="keep-your-app-and-business-running"></a>讓您的應用程式和業務持續運作
Azure 領先業界的 99.99% 可用性服務等級協定 (SLA) (Microsoft 受控資料中心的全球網路提供支援)，可協助讓您的應用程式 24 小時全年無休地運作。 使用每一 Azure Database for PostgreSQL 伺服器，您可以利用內建的安全性、容錯功能，以及可能需要另外設計、購買、建置和管理的資料保護功能。 使用適用於 PostgreSQL 的 Azure 資料庫時，每個定價層都會提供一組完整的業務持續性功能和選項，讓您能夠用來啟動和執行，並持續維持該狀態。 您可以使用[時間點還原](howto-restore-server-portal.md)將資料庫回復成先前的狀態，最久可至 35 天前。 此外，如果裝載資料庫的資料中心發生中斷情形，您可以從最新備份的異地備援複本還原資料庫。

## <a name="secure-your-data"></a>保護您的資料
Azure 資料庫服務一向重視資料安全性，Azure Database for PostgreSQL 以限制存取、保護靜止和移動中資料及協助監視活動等功能承襲了此傳統。 如需 Azure 平台安全性的相關資訊，請造訪 [Azure 信任中心 (英文)](https://www.microsoft.com/en-us/trustcenter/security)。

適用於 PostgreSQL 的 Azure 資料庫服務針對靜止資料使用儲存體加密。 包含備份在內的資料會在磁碟上加密 (不包括執行查詢時由引擎所建立的暫存檔案)。 該服務使用包含在 Azure 儲存體加密中的 AES 256 位元加密，且金鑰是由系統進行管理。 儲存體加密會一律啟用，且無法停用。

根據預設，適用於 PostgreSQL 的 Azure 資料庫服務已設為針對跨網路的動態資料需要 [SSL 連線安全性](./concepts-ssl-connection-security.md)。 在您的資料庫伺服器和用戶端應用程式之間強制使用 SSL 連線，可將伺服器與應用程式之間的資料流加密，有助於抵禦「中間人」攻擊。 (選擇性) 如果您的用戶端應用程式不支援 SSL 連線能力，您可以停用需要 SSL 才能連接到您資料庫服務的功能。

## <a name="contacts"></a>連絡人
若您對適用於 PostgreSQL 的 Azure 資料庫有任何疑問或建議，請傳送電子郵件給適用於 PostgreSQL 的 Azure 資料庫小組 ([@Ask Azure DB for PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com))。 請注意，這不是技術支援的別名。

此外，請根據您的情況考量下列連絡要點：
- 若要連絡 Azure 支援，[請從 Azure 入口網站提出票證](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
- 若要修正您的帳戶問題，請在 Azure 入口網站中提出[支援要求](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)。
- 若要提供意見反應或要求新功能，請透過 [UserVoice](https://feedback.azure.com/forums/597976-azure-database-for-postgresql) 建立項目。

## <a name="next-steps"></a>後續步驟
- 如需成本比較和計算機，請參閱[定價頁面](https://azure.microsoft.com/pricing/details/postgresql/)。
- 從[建立您的第一個 Azure Database for PostgreSQL](./quickstart-create-server-database-portal.md) 開始。
- 使用 Python、PHP、Ruby、C\#、Java、Node.js 建立您的第一個應用程式：[連線庫](./concepts-connection-libraries.md)
