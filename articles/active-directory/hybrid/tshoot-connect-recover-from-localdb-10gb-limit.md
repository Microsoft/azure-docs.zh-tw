---
title: Azure AD Connect：如何從 LocalDB 10 GB 的限制問題復原 | Microsoft Docs
description: 本主題說明當遇到 LocalDB 10 GB 限制的問題時，如何復原 Azure AD Connect 同步處理服務。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 41d081af-ed89-4e17-be34-14f7e80ae358
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: e78e7b5b4dba0bfea4f3398ca20b381a291fe44f
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46307817"
---
# <a name="azure-ad-connect-how-to-recover-from-localdb-10-gb-limit"></a>Azure AD Connect：如何從 LocalDB 10-GB 的限制復原
Azure AD Connect 需要 SQL Server 資料庫來儲存身分識別資料。 您可以使用 Azure AD Connect 安裝的預設 SQL Server 2012 Express LocalDB 或使用您自己的完整 SQL。 SQL Server Express 會實行 10 GB 的大小限制。 使用 LocalDB 且達到這個限制時，Azure AD Connect 同步處理服務無法再啟動或正確同步處理。 本文提供復原步驟。

## <a name="symptoms"></a>徵兆
有兩個常見的徵兆︰

* Azure AD Connect 同步處理服務**執行**，但無法同步處理並出現 “stopped-database-disk-full” 錯誤。

* Azure AD Connect 同步處理服務**無法啟動**。 當您嘗試啟動服務時，它會失敗並出現 6323 事件及「SQL Server 磁碟空間用完，因此伺服器發生錯誤」的錯誤訊息。

## <a name="short-term-recovery-steps"></a>短期復原步驟
本節提供的步驟可收回 Azure AD Connect 同步處理服務所需的 DB 空間以繼續作業。 步驟包括：
1. [判斷同步處理服務狀態](#determine-the-synchronization-service-status)
2. [壓縮資料庫](#shrink-the-database)
3. [刪除執行記錄資料](#delete-run-history-data)
4. [縮短執行歷程記錄資料的保留期間](#shorten-retention-period-for-run-history-data)

### <a name="determine-the-synchronization-service-status"></a>判斷同步處理服務狀態
首先，判斷同步處理服務是否仍在執行中︰

1. 以系統管理員身分登入您的 Azure AD Connect 伺服器。

2. 移至 [服務控制管理員]。

3. 檢查 **Microsoft Azure AD Sync** 的狀態。


4. 如果正在執行，請勿停止或重新啟動服務。 略過[壓縮資料庫](#shrink-the-database)步驟並移至[刪除執行記錄資料](#delete-run-history-data)步驟。

5. 如果非執行中，請嘗試啟動服務。 如果服務成功啟動，略過[壓縮資料庫](#shrink-the-database)步驟並移至[刪除執行記錄資料](#delete-run-history-data)步驟。 否則，請以[壓縮資料庫](#shrink-the-database)步驟繼續進行。

### <a name="shrink-the-database"></a>壓縮資料庫
請使用壓縮作業釋出足夠的 DB 空間，以啟動同步處理服務。 它會藉由在資料庫中移除空格來釋放 DB 空間。 這個步驟是最佳方式，因為不保證一律可以復原空間。 若要深入了解壓縮作業，請閱讀[壓縮資料庫](https://msdn.microsoft.com/library/ms189035.aspx)文章。

> [!IMPORTANT]
> 如果您可以取得要執行的同步處理服務，請略過此步驟。 不建議壓縮 SQL DB，因為它可能會導致因片段增加的效能不佳。

針對 Azure AD Connect 所建立的資料庫名稱是 **ADSync**。 若要執行壓縮作業，您必須以系統管理員或資料庫的 DBO 登入。 在 Azure AD Connect 安裝期間，會授與系統管理員權限給下列帳戶︰
* 本機系統管理員
* 用來執行 Azure AD Connect 安裝的使用者帳戶。
* 做為 Azure AD Connect 同步處理服務內容之作業系統的同步處理服務帳戶。
* 在安裝期間建立的本機群組 ADSyncAdmins。

1. 藉由將 `%ProgramFiles%\Microsoft Azure AD Sync\Data` 下的 **ADSync.mdf** 和 **ADSync_log.ldf** 檔案複製到安全的位置來備份資料庫。

2. 啟動新的 PowerShell 工作階段。

3. 瀏覽至資料夾 `%ProgramFiles%\Microsoft SQL Server\110\Tools\Binn`。

4. 使用系統管理員或資料庫 DBO 的認證，藉由執行 `./SQLCMD.EXE -S “(localdb)\.\ADSync” -U <Username> -P <Password>` 命令來啟動 **sqlcmd** 公用程式。

5. 若要壓縮資料庫，請在 sqlcmd 提示字元 (1>)，輸入 `DBCC Shrinkdatabase(ADSync,1);`，在下一行後面接著 `GO`。

6. 如果作業成功，請嘗試重新啟動同步處理服務。 如果您可以啟動同步處理服務，請移至[刪除執行記錄資料](#delete-run-history-data)步驟。 否則請連絡支援人員。

### <a name="delete-run-history-data"></a>刪除執行記錄資料
根據預設，Azure AD Connect 最多會保留七天的執行歷程記錄資料。 在此步驟中，我們會刪除歷程記錄資料來收回 DB 空間，讓 Azure AD Connect 同步處理服務可以再次啟動同步處理。

1.  前往 [開始] → [同步處理服務] 來啟動**同步處理服務管理員**。

2.  移至 [作業] 索引標籤。

3.  選取 [動作] 下方的 [清除執行]...

4.  您可以選擇 [清除所有執行] 或 [清除之前的執行...]**<date>** 選項。 建議您一開始先清除執行超過兩天的歷程記錄資料。 如果您遇到 DB 大小的問題，則選擇 [清除所有執行] 選項。

### <a name="shorten-retention-period-for-run-history-data"></a>縮短執行歷程記錄資料的保留期間
此步驟是要降低在多個同步處理循環之後遇到 10 GB 限制問題的可能性。

1. 開啟新的 PowerShell 工作階段。

2. 執行 `Get-ADSyncScheduler` 並記下 PurgeRunHistoryInterval 屬性，其會指定目前的保留期限。

3. 執行 `Set-ADSyncScheduler -PurgeRunHistoryInterval 2.00:00:00` 以將保留期限設為兩天。 適當調整保留期限。

## <a name="long-term-solution--migrate-to-full-sql"></a>長期解決方案 – 移轉至完整的 SQL
一般情況下，問題顯示 10 GB 資料庫大小不足，Azure AD Connect 無法再同步處理您的內部部署 Active Directory 到 Azure AD。 建議您改為使用完整版的 SQL Server。 您無法直接將現有的 Azure AD Connect 部署取代為完整版的 SQL 資料庫 LocalDB。 相反地，您必須部署新的 Azure AD Connect 伺服器與 SQL 的完整版本。 建議您將部署新 Azure AD Connect 伺服器 (含 SQL DB) 做為預備伺服器的變換移轉，其位於現有 Azure AD Connect 伺服器 (含 LocalDB) 旁。 
* 如需有關如何使用 Azure AD Connect 設定遠端 SQL 的指示，請參閱 [Azure AD Connect 的自訂安裝](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-custom)文章。
* 如需 Azure AD Connect 升級的變換移轉指示，請參閱 [Azure AD Connect︰從舊版升級至最新版本](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-upgrade-previous-version#swing-migration)文章。

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
