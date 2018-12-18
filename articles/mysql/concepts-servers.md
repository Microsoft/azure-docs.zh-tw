---
title: 適用於 MySQL 的 Azure 資料庫中的伺服器概念
description: 本主題提供使用適用於 MySQL 之 Azure 資料庫伺服器的考量和指導方針。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 9d94f897546ea1e1190aab91e80eb9868224e5a7
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35265454"
---
# <a name="server-concepts-in-azure-database-for-mysql"></a>適用於 MySQL 的 Azure 資料庫中的伺服器概念
本文提供使用「適用於 MySQL 的 Azure 資料庫」伺服器的考量和指導方針。

## <a name="what-is-an-azure-database-for-mysql-server"></a>什麼是適用於 MySQL 的 Azure 資料庫伺服器？

適用於 MySQL 的 Azure 資料庫伺服器是多個資料庫的中央管理點。 這與您可能已在內部部署領域中熟悉的 MySQL 伺服器建構相同。 具體來說，適用於 MySQL 的 Azure 資料庫服務會受到管理、提供效能保證，以及公開伺服器層級的存取和功能。

適用於 MySQL 的 Azure 資料庫伺服器：

- 建立於 Azure 訂用帳戶內。
- 是資料庫的父資源。
- 可為資料庫提供命名空間。
- 是具有強式存留期語意 (刪除伺服器) 的容器，而且會刪除自主資料庫。
- 在一個區域中共置資源。
- 提供用來存取伺服器和資料庫的連接端點。
- 提供適用於其資料庫的管理原則範圍︰登入、防火牆、使用者、角色、設定等等。
- 可在多個版本中使用。 如需詳細資訊，請參閱[支援適用於 MySQL 之 Azure 資料庫的資料庫版本](./concepts-supported-versions.md)。

在適用於 MySQL Server 的 Azure 資料庫內，您可以建立一個或多個資料庫。 您可以選擇在每部伺服器上建立單一資料庫以使用所有資源，或建立多個資料庫來共用資源。 定價結構是依據伺服器，以定價層、虛擬核心及儲存體 (GB) 的設定為基礎來形成的。 如需詳細資訊，請參閱[定價層](./concepts-service-tiers.md)。

## <a name="how-do-i-connect-and-authenticate-to-an-azure-database-for-mysql-server"></a>如何連接及驗證適用於 MySQL 的 Azure 資料庫伺服器？

下列項目有助於確保對資料庫的安全存取。
|||
| :-- | :-- |
| **驗證和授權** | 適用於 MySQL 的 Azure 資料庫伺服器支援原生的 MySQL 驗證。 您可以利用伺服器的管理員登入來連接和驗證伺服器。 |
| **通訊協定** | 此服務支援 MySQL 所使用的訊息架構通訊協定。 |
| **TCP/IP** | TCP/IP 和 Unix 網域通訊端上支援此通訊協定。 |
| **防火牆** | 為了協助保護您的資料，防火牆規則會防止對您的資料庫伺服器的所有存取，直到您指定哪些電腦擁有權限為止。 請參閱[適用於 MySQL 的 Azure 資料庫伺服器防火牆規則](./concepts-firewall-rules.md)。 |
| **SSL** | 此服務支援在您的應用程式和資料庫伺服器之間強制執行 SSL 連接。  請參閱[在您的應用程式中設定 SSL 連線能力，以安全地連線至適用於 MySQL 的 Azure 資料庫](./howto-configure-ssl.md)。 |

## <a name="how-do-i-manage-a-server"></a>如何管理伺服器？
您可以使用 Azure 入口網站或 Azure CLI，來管理適用於 MySQL 的 Azure 資料庫伺服器。

## <a name="next-steps"></a>後續步驟
- 如需服務的概觀，請參閱[適用於 MySQL 的 Azure 資料庫概觀](./overview.md)
- 如需以您**服務層**為依據之特定資源配額和限制的相關資訊，請參閱[服務層](./concepts-service-tiers.md)
- 如需連接到服務的相關資訊，請參閱[適用於 MySQL 的 Azure 資料庫的連線庫](./concepts-connection-libraries.md)。
