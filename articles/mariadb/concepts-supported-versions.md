---
title: 適用於 MariaDB 的 Azure 資料庫中支援的版本
description: 描述適用於 MariaDB 的 Azure 資料庫中支援的版本。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: aa83b9955839aaa03aa2ebf46c9e464dc75c8d3a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977497"
---
# <a name="supported-azure-database-for-mariadb-server-versions"></a>支援的適用於 MariaDB 的 Azure 資料庫伺服器版本
適用於 MariaDB 的 Azure 資料庫已使用 InnoDB 引擎從開放式原始碼 [MariaDB Server](https://downloads.mariadb.org/) 來開發。 適用於 MariaDB 的 Azure 資料庫目前支援下列版本：

## <a name="mariadb-version-10217"></a>MariaDB 版本 10.2.17
請參閱 [MariaDB 文件](https://downloads.mariadb.org/mariadb/10.2.17/)，以深入了解 MariaDB 10.2.17 中的改進功能與修正。

> [!NOTE]
> 在服務中，會使用閘道將連線重新導向到伺服器執行個體。 建立連線之後，MySQL 用戶端會顯示閘道中設定的 MariaDB 的版本，而非 MariaDB 伺服器執行個體上執行的實際版本。 若要判斷您的 MariaDB 伺服器執行個體的版本，請在 MySQL 提示字元命令下使用 `SELECT VERSION();` 命令。

## <a name="managing-updates-and-upgrades"></a>管理更新和升級
服務會自動管理對於次要版本更新的修補。

## <a name="next-steps"></a>後續步驟
如需以您**服務層**為依據之特定資源配額和限制的相關資訊，請參閱[服務層](./concepts-pricing-tiers.md)