---
title: 適用於 MariaDB 的 Azure 資料庫的 SSL 連線能力
description: 用來設定適用於 MariaDB 的 Azure 資料庫與相關聯應用程式以適當使用 SSL 連線的資訊
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: dda5df58a83ddd3ce42fa887c3c32a3e23954920
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946641"
---
# <a name="ssl-connectivity-in-azure-database-for-mariadb"></a>適用於 MariaDB 的 Azure 資料庫中的 SSL 連線能力
適用於 MariaDB 的 Azure 資料庫支援使用安全通訊端層 (SSL)，將資料庫伺服器連接至用戶端應用程式。 在您的資料庫伺服器和用戶端應用程式之間強制使用 SSL 連線，可將伺服器與應用程式之間的資料流加密，有助於抵禦「中間人」攻擊。

## <a name="default-settings"></a>預設設定
根據預設，資料庫服務應該會設定為在連線到 MariaDB 時需要 SSL 連接。  建議盡可能地避免停用 SSL 選項。

透過 Azure 入口網站與 CLI 佈建新的適用於 MariaDB 的 Azure 資料庫伺服器時，預設會強制執行 SSL 連線。

Azure 入口網站中會顯示多種程式設計語言的連接字串。 這些連接字串包含連接到您資料庫所需的 SSL 參數。 在 Azure 入口網站中選取您的伺服器。 在下 [設定] 標題之下，選取 [連接字串]。 SSL 參數會根據連接器而有所不同，例如，"ssl=true" 或 "sslmode=require" 或 "sslmode=required" 及其他變化。

<!-- To learn how to enable or disable SSL connection when developing application, refer to [How to configure SSL](howto-configure-ssl.md).-->

## <a name="next-steps"></a>後續步驟
- 深入了解[伺服器防火牆規則](concepts-firewall-rules.md)
- 了解如何[設定 SSL](howto-configure-ssl.md)。
