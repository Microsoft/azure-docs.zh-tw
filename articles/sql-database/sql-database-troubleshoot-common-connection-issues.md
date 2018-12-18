---
title: 針對 Azure SQL Database 的常見連線問題進行疑難排解
description: 找出並解決 Azure SQL Database 常見之連接錯誤的步驟。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: dalechen
ms.author: daleche
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 276a2d6d717497538a6633a9b0926bb89b9e875c
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161092"
---
# <a name="troubleshoot-connection-issues-to-azure-sql-database"></a>針對 Azure SQL Database 連線問題進行疑難排解
連線到 Azure SQL Database 失敗時，您會收到[錯誤訊息](sql-database-develop-error-messages.md)。 本文是集中式主題，可協助您針對 Azure SQL Database 連線問題進行疑難排解。 本文除了介紹連線問題的[常見原因](#cause)，還推薦可協助您識別問題的[疑難排解工具](#try-the-troubleshooter-for-azure-sql-database-connectivity-issues)，以及提供疑難排解步驟來解決[暫時性錯誤](#troubleshoot-transient-errors)和[持續性或非暫時性錯誤](#troubleshoot-persistent-errors)。 

如果您遇到連線問題，請嘗試本文中所述的疑難排解步驟。
[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="cause"></a>原因
連線問題可能由下列任何一個原因所造成：

* 在設計應用程式的過程中，無法套用最佳作法和設計指南。  請參閱 [SQL Database 開發概觀](sql-database-develop-overview.md)以便開始使用。
* Azure SQL Database 重新設定
* 防火牆設定
* 連線逾時
* 不正確的登入資訊
* 部分 Azure SQL Database 資源已達上限

一般而言，Azure SQL Database 的連線問題可以分類如下：

* [暫時性錯誤 (短期或間歇性)](#troubleshoot-transient-errors)
* [持續性或非暫時性錯誤 (定期重複發生的錯誤)](#troubleshoot-persistent-errors)

## <a name="try-the-troubleshooter-for-azure-sql-database-connectivity-issues"></a>嘗試使用 Azure SQL Database 連線問題疑難排解工具
如果您遇到特定的連線錯誤，請嘗試使用[此工具](https://support.microsoft.com/help/10085/troubleshooting-connectivity-issues-with-microsoft-azure-sql-database)，此工具可協助您快速識別並解決您的問題。

## <a name="troubleshoot-transient-errors"></a>針對暫時性錯誤進行疑難排解

當應用程式連線到 Azure SQL Database 時，您會收到下列錯誤訊息︰

```
Error code 40613: "Database <x> on server <y> is not currently available. Please retry the connection later. If the problem persists, contact customer support, and provide them the session tracing ID of <z>"
```

> [!NOTE]
> 此錯誤訊息是通常是暫時性 (短期)。
> 
> 

當移動 (或重新設定) Azure 資料庫時會發生此錯誤，而且您的應用程式會失去與 SQL Database 的連接。 SQL Database 重新設定事件由於規劃的事件 (例如，軟體升級) 或未規劃的事件 (例如，處理序損毀或負載平衡) 而發生。 大部分的重新設定事件通常只是短期的，至多應在不到 60 秒的時間完成。 不過，這些事件可能偶爾會需要更長時間才能完成，例如當大型交易導致長時間執行的復原時。

### <a name="steps-to-resolve-transient-connectivity-issues"></a>解決暫時性連線問題的步驟

1. 檢查 [Microsoft Azure 服務儀表板](https://azure.microsoft.com/status) ，以取得應用程式報告錯誤期間發生的任何已知中斷。
2. 連接到雲端服務的應用程式 (例如 Azure SQL Database) 應該預期定期的重新設定事件，並實作應用程式重試邏輯來處理這些錯誤，而不是將這些錯誤當做應用程式錯誤呈現給使用者。 如需詳細資訊和一般重試策略，請檢閱[暫時性錯誤](sql-database-connectivity-issues.md)一節以及 [SQL Database 開發概觀](sql-database-develop-overview.md)的最佳作法和設計指南。 如需詳細資訊，請接著參閱 [適用於 SQL Database 和 SQL Server 的連線庫](sql-database-libraries.md) 的程式碼範例 。
3. 由於資料庫接近其資源限制，因此似乎是暫時性連線問題。 請參閱[資源限制](sql-database-resource-limits.md)。
4. 如果連線問題繼續發生，或如果您的應用程式發生錯誤的持續時間超過 60 秒，或如果您在一天當中，看到錯誤多次發生，請在 [Azure 支援](https://azure.microsoft.com/support/options)網站上選取 [取得支援]，來提出 Azure 支援要求。

## <a name="troubleshoot-persistent-errors"></a>針對持續性錯誤進行疑難排解
如果應用程式持續無法連接到 Azure SQL Database，通常表示下列其中一項發生問題︰

* 防火牆組態。 Azure SQL Database 或用戶端防火牆封鎖 Azure SQL Database 的連接。
* 用戶端上的網路重新設定：例如，新的 IP 位址或 Proxy 伺服器。
* 使用者錯誤︰例如，輸入錯誤的連線參數，例如連接字串中的伺服器名稱。

### <a name="steps-to-resolve-persistent-connectivity-issues"></a>解決永久性連線問題的步驟
1. 設定 [防火牆規則](sql-database-configure-firewall-settings.md) 允許用戶端 IP 位址。 對於臨時測試用途，請設定防火牆規則並使用 0.0.0.0 做為起始 IP 位址範圍，並使用 255.255.255.255 做為結束 IP 位址範圍。 這樣會開放伺服器供所有 IP 位址存取。 若這樣可解決您的連線問題，請移除此規則並針對已適當限制的 IP 位址或位址範圍建立防火牆規則。 
2. 在用戶端與網際網路之間的所有防火牆上，請確定開放連接埠 1433 供輸出連線使用。 檢閱[設定 Windows 防火牆以允許 SQL Server 存取](https://msdn.microsoft.com/library/cc646023.aspx)與[混合式身分識別所需的連接埠與通訊協定 (英文)](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-ports) 以取得有關必須針對 Azure Active Directory 驗證開放之其他連接埠的其他詳細資訊。
3. 請確認您的連接字串和其他連線設定。 請參閱 [連線能力問題主題](sql-database-connectivity-issues.md#connections-to-sql-database)中的「連接字串」一節。
4. 檢查儀表板中的服務健全狀況。 如果您認為沒有區域性停電，請參閱[從中斷復原](sql-database-disaster-recovery.md)以了解復原到新區域的步驟。

## <a name="next-steps"></a>後續步驟
* [搜尋 Microsoft Azure 相關文件](http://azure.microsoft.com/search/documentation/)
* [檢視 Azure SQL Database 服務的最新更新](http://azure.microsoft.com/updates/?service=sql-database)

## <a name="additional-resources"></a>其他資源
* [SQL Database 開發概觀](sql-database-develop-overview.md)
* [一般暫時性錯誤處理指導方針](../best-practices-retry-general.md)
* [SQL Database 和 SQL Server 的連線庫](sql-database-libraries.md)

