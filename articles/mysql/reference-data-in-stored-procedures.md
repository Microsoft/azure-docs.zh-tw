---
title: 適用於 MySQL 的 Azure 資料庫之資料帶入複寫預存程序
description: 本文介紹用於資料帶入複寫的所有預存程序。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 08/31/2018
ms.openlocfilehash: fb1a1b31d90df0022e5973de3ae2f55fb4c36701
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665941"
---
# <a name="azure-database-for-mysql-data-in-replication-stored-procedures"></a>適用於 MySQL 的 Azure 資料庫之資料帶入複寫預存程序

資料帶入複寫可讓您將來自在內部部署執行的 MySQL 伺服器、虛擬機器中或由其他雲端提供者所代管的資料庫服務的資料，同步處理到適用於 MySQL 的 Azure 資料庫服務。

下列預存程序可用來設定或移除主體和複本之間的資料輸入複寫。

|**預存程序名稱**|**輸入參數**|**輸出參數**|**使用方式注意事項**|
|-----|-----|-----|-----|
|*mysql.az_replication_change_master*|master_host<br/>master_user<br/>master_password<br/>master_port<br/>master_log_file<br/>master_log_pos<br/>master_ssl_ca|N/A|若要使用 SSL 模式傳送資料，將 CA 憑證的內容傳入 master_ssl_ca 參數。 </br><br>如果不使用 SSL 傳輸資料，將空字串傳入 master_ssl_ca 參數。|
|*mysql.az_replication _start*|N/A|N/A|開始複寫。|
|*mysql.az_replication _stop*|N/A|N/A|停止複寫。|
|*mysql.az_replication _remove_master*|N/A|N/A|移除主體與複本之間的複寫關聯性。|
|*mysql.az_replication_skip_counter*|N/A|N/A|略過一個複寫錯誤。|

若要在適用於 MySQL 的 Azure 資料庫中主體和複本之間設定資料輸入複寫，請參閱[如何設定資料輸入複寫](howto-data-in-replication.md)。
