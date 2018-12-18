---
title: 適用於 PostgreSQL 的 Azure 資料庫中的伺服器概念
description: 此文章提供設定及管理適用於 PostgreSQL 之 Azure 資料庫伺服器的考量和指導方針。
services: postgresql
author: rachel-msft
ms.author: raagyema
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 09/27/2018
ms.openlocfilehash: 8fcb5e8371d6c813eb7f0ab4d23a5aac5c41fb3b
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404634"
---
# <a name="azure-database-for-postgresql-servers"></a>適用於 PostgreSQL 的 Azure 資料庫伺服器
此文章提供使用適用於 PostgreSQL 之 Azure 資料庫伺服器的考量和指導方針。

## <a name="what-is-an-azure-database-for-postgresql-server"></a>什麼是適用於 PostgreSQL 的 Azure 資料庫伺服器？
適用於 PostgreSQL 的 Azure 資料庫伺服器是多個資料庫的中央管理點。 這與您可能已在內部部署領域中熟悉的 PostgreSQL 伺服器建構相同。 具體來說，PostgreSQL 服務會受到管理、提供效能保證、公開伺服器層級的存取和功能。

適用於 PostgreSQL 的 Azure 資料庫伺服器：

- 建立於 Azure 訂用帳戶內。
- 是資料庫的父資源。
- 可為資料庫提供命名空間。
- 是具有強式存留期語意 (刪除伺服器) 的容器，而且會刪除自主資料庫。
- 在一個區域中共置資源。
- 提供用來存取伺服器和資料庫的連接端點 (.postgresql.database.azure.com)。
- 提供適用於其資料庫的管理原則範圍︰登入、防火牆、使用者、角色、設定等等。
- 可在多個版本中使用。 如需詳細資訊，請參閱[支援的 PostgreSQL 資料庫版本](concepts-supported-versions.md)。
- 可由使用者擴充。 如需詳細資訊，請參閱 [PostgreSQL 擴充功能](concepts-extensions.md)。

在適用於 PostgreSQL 的 Azure 資料庫內，您可以建立一個或多個資料庫。 您可以選擇在每個伺服器建立單一資料庫以利用所有資源，或建立多個資料庫來共用資源。 定價結構是依據伺服器，以定價層、虛擬核心及儲存體 (GB) 的設定為基礎來形成的。 如需詳細資訊，請參閱[定價層](./concepts-pricing-tiers.md)。

## <a name="how-do-i-connect-and-authenticate-to-an-azure-database-for-postgresql-server"></a>如何連接及驗證適用於 PostgreSQL 的 Azure 資料庫伺服器？
下列項目有助於確保對資料庫的安全存取：

|||
|:--|:--|
| **驗證和授權** | 適用於 PostgreSQL 的 Azure 資料庫伺服器支援原生的 PostgreSQL 驗證。 您可以利用伺服器的系統管理員登入來連接和驗證伺服器。 |
| **通訊協定** | 此服務支援 PostgreSQL 所使用的訊息架構通訊協定。 |
| **TCP/IP** | TCP/IP 和 Unix 網域通訊端上支援此通訊協定。 |
| **防火牆** | 為了協助保護您的資料，防火牆規則會防止對您伺服器及其資料庫的所有存取，直到您指定哪些電腦擁有權限為止。 請參閱[適用於 PostgreSQL 的 Azure 資料庫伺服器防火牆規則](concepts-firewall-rules.md)。 |

## <a name="managing-your-server"></a>管理伺服器
您可以使用 [Azure 入口網站](https://portal.azure.com)或 [Azure CLI](/cli/azure/postgres)，來管理適用於 PostgreSQL 的 Azure 資料庫伺服器。

建立伺服器時，您可以設定管理使用者的認證。 管理使用者是伺服器上最高權限的使用者。 這屬於角色 azure_pg_admin。 此角色沒有完整的超級使用者權限。 

PostgreSQL 超級使用者屬性用於指派給 azure_superuser，這屬於受控服務。 您沒有此角色的存取權限。

適用於 PostgreSQL 的 Azure 資料庫伺服器有兩個預設資料庫： 
- **postgres** - 您可以在建立伺服器時連接的資料庫。
- **azure_maintenance** - 這個資料庫用來區隔提供受控服務的流程和使用者動作。 您無法此資料庫的存取權限。
- **azure_sys** - 查詢存放區的資料庫。 當查詢存放區已關閉時，此資料庫不會累積資料；這是預設設定。 如需詳細資訊，請參閱[查詢存放區概觀](concepts-query-store.md)主題。


## <a name="server-parameters"></a>伺服器參數
PostgreSQL 伺服器參數會判斷伺服器的設定。 在適用於 PostgreSQL 的 Azure 資料庫中，可以使用 Azure 入口網站或 Azure CLI 檢視和編輯參數清單。 

適用於 PostgreSQL 的 Azure 資料庫是 Postgres 的受控服務，其中的可設定參數是本機 Postgres 執行個體中參數的子集 (如需有關 Postgres 參數的詳細資訊，請參閱 [PostgreSQL 文件](https://www.postgresql.org/docs/9.6/static/runtime-config.html))。 適用於 PostgreSQL 的 Azure 資料庫伺服器會在建立時為每個參數啟用預設值。 需要伺服器重新啟動或是需要 superuser 存取之後變更才會生效的部分參數，無法由使用者設定。


## <a name="next-steps"></a>後續步驟
- 如需服務的概觀，請參閱[適用於 PostgreSQL 的 Azure 資料庫概觀](overview.md)。
- 如需有關以您**服務層**為依據之特定資源配額和限制的資訊，請參閱[服務層](concepts-pricing-tiers.md)。
- 如需連接到服務的相關資訊，請參閱[適用於 PostgreSQL 的 Azure 資料庫的連線庫](concepts-connection-libraries.md)。
- 透過 [Azure 入口網站](howto-configure-server-parameters-using-portal.md)或 [Azure CLI](howto-configure-server-parameters-using-cli.md) 檢視和編輯伺服器參數。
