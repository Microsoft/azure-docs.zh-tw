---
title: Azure 資料庫安全性檢查清單 | Microsoft Docs
description: 本文提供一組 Azure 資料庫安全性的檢查清單。
services: security
documentationcenter: na
author: unifycloud
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: tomsh
ms.openlocfilehash: 7712196caefeea8a62045d432d762fbd16f42011
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33893961"
---
# <a name="azure-database-security-checklist"></a>Azure 資料庫安全性檢查清單

為了協助改善安全性，Azure 資料庫包含數個內建安全性控制項，您可用來限制和控制存取。

其中包含：

-   防火牆，可讓您建立依 IP 位址限制連線能力的[防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)
-   可從 Azure 入口網站存取的伺服器層級防火牆
-   可從 SSMS 存取的資料庫層級防火牆規則
-   使用安全的連接字串來保護資料庫連線
-   使用存取管理
-   資料加密
-   SQL Database 稽核
-   SQL Database 威脅偵測

## <a name="introduction"></a>簡介
雲端運算需要許多應用程式使用者、資料庫管理員和程式設計人員所不熟悉的新安全性典範。 因此，有些組織會因為意識到安全性風險而對實作資料管理的雲端基礎結構顯得遲疑不決。 不過，透過進一步了解 Microsoft Azure 和 Microsoft Azure SQL Database 內建的安全性功能，可以大幅減輕這層疑慮。

## <a name="checklist"></a>檢查清單
我們建議您先閱讀 [Azure 資料庫安全性最佳做法](https://docs.microsoft.com/azure/security/azure-database-security-best-practices)一文，再檢閱這份檢查清單。 在您了解最佳做法之後，您就能夠充分利用這份檢查清單。 您可以接著使用這份檢查清單，確保您已解決 Azure 資料庫安全性的重大問題。


|檢查清單類別| 說明|
| ------------ | -------- |
|**保護資料**||
| <br> 移動/傳輸中加密| <ul><li>[傳輸層安全性](https://docs.microsoft.com/windows-server/security/tls/transport-layer-security-protocol)，用於在資料移至網路時進行資料加密。</li><li>資料庫要求來自用戶端的安全通訊以透過 TLS (傳輸層安全性) 的 [TDS (表格式資料流)](https://msdn.microsoft.com/en-in/library/dd357628.aspx) 通訊協定為基礎。</li></ul> |
|<br>待用加密| <ul><li>[透明資料加密](http://go.microsoft.com/fwlink/?LinkId=526242)，適用於非作用中資料實際以任何數位形式儲存時。</li></ul>|
|**控制存取**||  
|<br> 資料庫存取 | <ul><li>[驗證](https://docs.microsoft.com/azure/sql-database/sql-database-control-access) (Azure Active Directory 驗證) AD 驗證會使用由 Azure Active Directory 管理的身分識別。</li><li>[授權](https://docs.microsoft.com/azure/sql-database/sql-database-control-access)會授與使用者所需的最低權限。</li></ul> |
|<br>應用程式存取| <ul><li>[資料列層級安全性](https://msdn.microsoft.com/library/dn765131) (使用安全性原則，同時根據使用者的身分識別、角色或執行內容限制資料列層級存取)。</li><li>[動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started) (使用權限和原則，對不具權限的使用者進行遮罩處理，以限制敏感性資料的揭露)</li></ul>|
|**主動式監視**||  
| <br>追蹤和偵測| <ul><li>[稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)可追蹤資料庫事件，並將事件寫入您 [Azure 儲存體帳戶](https://docs.microsoft.com/azure/storage/storage-create-storage-account)中的稽核記錄/活動記錄。</li><li>使用 [Azure 監視器活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)來追蹤 Azure 資料庫健康情況。</li><li>[威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection)會偵測異常資料庫活動，指出資料庫有潛在的安全性威脅。 </li></ul> |
|<br>Azure 資訊安全中心| <ul><li>[資料監視](https://docs.microsoft.com/azure/security-center/security-center-enable-auditing-on-sql-databases) 使用 Azure 資訊安全中心作為 SQL 和其他 Azure 服務的集中式安全性監視解決方案。</li></ul>|       

## <a name="conclusion"></a>結論
Azure 資料庫是強固的資料庫平台，具有完整的安全性功能，能符合許多組織及法務相容性需求。 您可以輕鬆保護資料，方法是控制實體存取您的資料，以及透過透明資料加密、資料格等級加密或資料列等級安全性，使用檔案、資料行或資料列等級的各種資料安全性選項。 Always Encrypted 也會啟用針對加密資料的作業，從而簡化應用程式更新的程序。 接著，存取 SQL Database 活動的稽核記錄可為您提供所需要的資訊，讓您知道存取資料的方式及時間。

## <a name="next-steps"></a>後續步驟
只需要幾個簡單步驟，您就可以讓資料庫預防惡意使用者或未經授權的存取。 您會在本教學課程中學到：

- 設定伺服器和/或資料庫的[防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)。
- 使用[加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/sql-server-encryption)來保護您的資料。
- 啟用 [SQL Database 稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)。

