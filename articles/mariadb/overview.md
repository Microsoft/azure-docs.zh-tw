---
title: 「適用於 MariaDB 的 Azure 資料庫」關聯式資料庫服務的概觀
description: 「適用於 MariaDB 的 Azure 資料庫」關聯式資料庫服務的概觀。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: overview
ms.date: 09/24/2018
ms.custom: mvc
ms.openlocfilehash: 696ba68894b4ac5f73c234ea49883fd0aa3db4d5
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320912"
---
# <a name="what-is-azure-database-for-mariadb"></a>什麼是適用於 MariaDB 的 Azure 資料？

「適用於 MariaDB 的 Azure 資料庫」是 Microsoft 雲端中的關聯式資料庫服務。 「適用於 MariaDB 的 Azure 資料庫」以 [MariaDB Community Edition](https://mariadb.org/download/) 資料庫引擎作為基礎。 此服務目前為公開預覽狀態。 

「適用於 MariaDB 的 Azure 資料庫」提供：

- 內建高可用性但沒有任何額外成本。
- 可預測的效能，使用預付型方案計價方式。
- 視需要在幾秒內進行縮放。
- 受到保護，可保護機密的待用資料和移動中資料。
- 最多 35 天的自動化備份和時間點還原。
- 企業級安全性與合規性。

這些功能幾乎不需要管理。 全都免費提供。 適用於 MariaDB 的 Azure 資料庫可協助您快速開發應用程式，並加快上市時間。 您不必配置寶貴的時間和資源來管理虛擬機器和基礎結構。 您也可以繼續使用所選擇的開放原始碼工具及平台來開發應用程式。 提供貴公司所需的速度和效率，無需學習新技能便可做到這一切。

若要了解「適用於 MariaDB 的 Azure 資料庫」中的核心概念和功能 (包括效能、延展性和管理能力)，請參閱下列快速入門：

- [使用 Azure 入口網站建立適用於 MariaDB 的 Azure 資料庫伺服器](quickstart-create-mariadb-server-database-using-azure-portal.md)
- [使用 Azure CLI 建立適用於 MariaDB 的 Azure 資料庫伺服器](quickstart-create-mariadb-server-database-using-azure-cli.md)

<!--
For a set of Azure CLI samples, see:
- [Azure CLI samples for Azure Database for MariaDB](sample-scripts-azure-cli.md) 
-->

## <a name="adjust-performance-and-scale-within-seconds"></a>在幾秒之內即可調整效能和規模

在預覽中，「適用於 MariaDB 的 Azure 資料庫」服務提供了多個服務層：基本、一般用途和記憶體最佳化。 每個服務層提供不同的效能和功能，以支援輕量到重量級的資料庫工作負載。 您可以在小型資料庫中建置第一個應用程式，一個月只需少許花費，然後調整規模以滿足解決方案的需求。 動態延展性可協助資料庫以透明的方式回應快速變化的資源需求。 有資源需求時，您才需要就所需資源支付費用。 如需詳細資訊，請參閱[定價層](concepts-pricing-tiers.md)。

## <a name="monitoring-and-alerting"></a>監視和警示

如何決定何時要相應增加或減少？ 您可以使用「適用於 MariaDB 的 Azure 資料庫」內建的效能監視和警示功能，加上以虛擬核心為基礎的效能分級。 透過這些工具，您可以根據目前或計畫中的效能需求，快速評估相應增加或減少虛擬核心的影響。 <!--See [Alerts](howto-alert-on-metric.md) for details.-->

## <a name="keep-your-app-and-business-running"></a>讓您的應用程式和業務持續運作

Azure 領先業界的 99.99% 可用性 SLA (公開預覽期間不提供)，並由受 Microsoft 管理之資料中心的全球網路提供支援。 此網路可協助讓應用程式 24 小時全年無休地運作。 您可從「適用於 MariaDB 的 Azure 資料庫」內建的安全性、容錯及資料保護獲益。 使用「適用於 MariaDB 的 Azure 資料庫」，您可以使用時間點還原將伺服器回復成先前的狀態，最久可至 35 天前。

## <a name="secure-your-data"></a>保護您的資料

Azure 資料庫服務具有「適用於 MariaDB 的 Azure 資料庫」所秉承的資料安全性信念。 「適用於 MariaDB 的 Azure 資料庫」所提供的功能可限制存取、保護待用和移動中資料，並協助您監視活動。 如需 Azure 平台安全性的相關資訊，請造訪 [Azure 信任中心](https://www.microsoft.com/en-us/trustcenter/security) \(英文\)。

「適用於 MariaDB 的 Azure 資料庫」服務針對待用資料使用儲存體加密。 磁碟上的資料 (包括備份資料) 都會加密。 (磁碟上由引擎在執行查詢時所建立的暫存檔不會加密)。該服務使用包含在 Azure 儲存體加密中的 AES 256 位元加密。 金鑰則由系統管理。 儲存體加密會一律啟用，且無法停用。

根據預設，「適用於 MariaDB 的 Azure 資料庫」服務已設為針對跨網路的動態資料需要 [SSL 連線安全性](./concepts-ssl-connection-security.md)。 在您的資料庫伺服器和用戶端應用程式之間強制使用 SSL 連線，可將伺服器與應用程式之間的資料流加密，有助於抵禦「中間人」攻擊。 (選擇性) 如果用戶端應用程式不支援 SSL 連線能力，您可以停用需要 SSL 才能連線到資料庫服務的功能。

## <a name="contacts"></a>連絡人

您可以將對於「適用於 MariaDB 的 Azure 資料庫」的任何疑問或建議傳送給[「適用於 MariaDB 的 Azure 資料庫」小組](mailto:AskAzureDBforMariaDB@service.microsoft.com) (不是支援小組別名)。

您也可以使用下列連絡點：
- 若要連絡 Azure 支援，請在 Azure 入口網站中[提出支援要求](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
- 若要修正帳戶的問題，請在 Azure 入口網站中[提出支援要求](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)。
- 若要提供意見反應或要求新功能，請在 [Azure 意見反應論壇](https://feedback.azure.com/forums/915439-azure-database-for-mariadb)建立條目。

## <a name="next-steps"></a>後續步驟

您已看過「適用於 MariaDB 的 Azure 資料庫」的簡介，因此已經可以：
- 查看[定價](https://azure.microsoft.com/pricing/details/mariadb/)頁面的成本比較和計算機。 
- 從[建立您的第一部伺服器](quickstart-create-mariadb-server-database-using-azure-portal.md)開始。

<!--- - Build your first app using your preferred language: [Python](./connect-python.md) | [Node.JS](./connect-nodejs.md) | [Java](./connect-java.md) | [Ruby](./connect-ruby.md) | [PHP](./connect-php.md) | [.NET (C#)](./connect-csharp.md) | [Go](./connect-go.md) --->
