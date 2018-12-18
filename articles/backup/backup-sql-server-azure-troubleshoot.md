---
title: 適用於 SQL Server VM 的 Azure 備份疑難排解指南 | Microsoft Docs
description: 關於將 SQL Server VM 備份至 Azure 的疑難排解資訊。
services: backup
documentationcenter: ''
author: markgalioto
manager: carmonm
editor: ''
keywords: ''
ms.assetid: ''
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/19/2018
ms.author: markgal;anuragm
ms.custom: ''
ms.openlocfilehash: 1c87382c2aae70b022fb391f80f7c75b0a4e5fe6
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36296955"
---
# <a name="troubleshoot-back-up-sql-server-on-azure"></a>針對 Azure 上的 SQL Server 備份進行疑難排解

本文針對保護 Azure 上的 SQL Server VM (預覽) 提供疑難排解資訊。

## <a name="public-preview-limitations"></a>公開預覽限制

若要檢視公開預覽限制，請參閱[在 Azure 中備份 SQL Server 資料庫](backup-azure-sql-database.md#public-preview-limitations)一文。

## <a name="sql-server-permissions"></a>SQL Server 權限

若要針對虛擬機器上的 SQL Server 資料庫設定保護功能，該虛擬機器上就必須安裝 **AzureBackupWindowsWorkload** 擴充功能。 如果您收到 **UserErrorSQLNoSysadminMembership** 錯誤，這表示 SQL 執行個體沒有所需的備份權限。 若要修正這個錯誤，請遵循[設定非 Marketplace SQL VM 的權限](backup-azure-sql-database.md#set-permissions-for-non-marketplace-sql-vms)中的步驟。

## <a name="troubleshooting-errors"></a>錯誤疑難排解

請使用下列表格中的資訊，來針對在 Azure 上保護 SQL Server 時所遇到的問題和錯誤進行疑難排解。

## <a name="backup-failures"></a>備份失敗

下列表格會依錯誤碼來組織。

### <a name="usererrorsqlpodoesnotsupportbackuptype"></a>UserErrorSQLPODoesNotSupportBackupType

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 此 SQL 資料庫不支援所要求的備份類型。 | 當資料庫復原模式不允許所要求的備份類型時便會發生。 此錯誤會於下列情況時發生： <br/><ul><li>使用簡單復原模式的資料庫不允許記錄備份。</li><li>master 資料庫不允許差異備份和記錄備份。</li></ul>如需詳細資訊，請參閱 [SQL 復原模式](https://docs.microsoft.com/sql/relational-databases/backup-restore/recovery-models-sql-server)文件。 | 如果在簡單復原模式下的資料庫記錄備份失敗，請嘗試下列其中一個選項：<ul><li>如果資料庫處於簡單復原模式，請停用記錄備份。</li><li>使用 [SQL 文件](https://docs.microsoft.com/sql/relational-databases/backup-restore/view-or-change-the-recovery-model-of-a-database-sql-server)將資料庫復原模式變更為「完整」或「大量記錄」。 </li><li> 如果您不想變更復原模式，而且所用來備份多個資料庫的標準原則無法變更，則請忽略此錯誤。 完整和差異備份會依排程運作。 記錄備份則會略過，這符合此案例的預期。</li></ul>如果是 master 資料庫，而且您已設定差異或記錄備份，請使用下列任何步驟：<ul><li>使用入口網站將 master 資料庫的備份原則排程變更為「完整」。</li><li>如果所用來備份多個資料庫的標準原則無法變更，則請忽略此錯誤。 完整備份會依排程運作。 差異或記錄備份則不會進行，這符合此案例的預期。</li></ul> |
| 作業遭到取消，原因是同一個資料庫上已在執行衝突的作業。 | 請參閱關於同時執行的[備份和還原限制部落格文章](https://blogs.msdn.microsoft.com/arvindsh/2008/12/30/concurrency-of-full-differential-and-log-backups-on-the-same-database)。| [使用 SQL Server Management Studio (SSMS) 來監視備份作業。](backup-azure-sql-database.md#manage-azure-backup-operations-for-sql-on-azure-vms) 衝突的作業失敗後，請重新啟動作業。|

### <a name="usererrorsqlpodoesnotexist"></a>UserErrorSQLPODoesNotExist

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| SQL 資料庫不存在。 | 資料庫已遭到刪除或重新命名。 | <ul><li>檢查資料庫是否已意外遭到刪除或重新命名。</li><li>如果資料庫意外遭到刪除，卻要繼續備份，請將資料庫還原到原始位置。</li><li>如果您已刪除資料庫，而且之後不再需要備份，則請在復原服務保存庫中，按一下[使用「刪除/保留資料」停止備份](backup-azure-sql-database.md#manage-azure-backup-operations-for-sql-on-azure-vms)。</li>|

### <a name="usererrorsqllsnvalidationfailure"></a>UserErrorSQLLSNValidationFailure

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 記錄鏈結中斷。 | 資料庫或 VM 使用其他備份解決方案來備份，而讓記錄鏈結截斷。|<ul><li>檢查是否有其他備份解決方案或指令碼正在使用中。 如果有，請停止其他備份解決方案。 </li><li>如果該備份屬於臨機操作記錄備份，請觸發完整備份來啟動新的記錄鏈結。 若為已排程的記錄備份，因為 Azure 備份服務會自動觸發完整備份進而修正此問題，所以不需要採取任何動作。</li>|

### <a name="usererroropeningsqlconnection"></a>UserErrorOpeningSQLConnection

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| Azure 備份無法連線至 SQL 執行個體。 | Azure 備份無法連線至 SQL 執行個體。 | 使用 Azure 入口網站錯誤功能表中的其他詳細資料，來縮小根本原因。 請參閱 [SQL 備份疑難排解](https://docs.microsoft.com/sql/database-engine/configure-windows/troubleshoot-connecting-to-the-sql-server-database-engine)以修正錯誤。<br/><ul><li>如果預設的 SQL 設定不允許遠端連線，請變更設定。 請參閱下列連結來變更設定。<ul><li>[https://msdn.microsoft.com/library/bb326495.aspx](https://msdn.microsoft.com/library/bb326495.aspx)</li><li>[https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-2-database-engine-error](https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-2-database-engine-error)</li><li>[https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-53-database-engine-error](https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-53-database-engine-error)</li></ul></li></ul><ul><li>如果有登入問題，請參閱下列連結來修正問題：<ul><li>[https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error](https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error)</li><li>[https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-18452-database-engine-error](https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-18452-database-engine-error)</li></ul></li></ul> |

### <a name="usererrorparentfullbackupmissing"></a>UserErrorParentFullBackupMissing

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 此資料來源的第一個完整備份遺失。 | 資料庫的完整備份遺失。 完整備份是記錄和差異備份的母體，因此必須先擷取完整備份才能觸發差異或記錄備份。 | 觸發臨機操作完整備份。   |

### <a name="usererrorbackupfailedastransactionlogisfull"></a>UserErrorBackupFailedAsTransactionLogIsFull

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 資料來源的交易記錄已滿，所以無法擷取備份。 | 資料庫的交易記錄空間已滿。 | 若要修正這個問題，請參閱 [SQL 文件](https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-9002-database-engine-error)。 |
| 此 SQL 資料庫不支援所要求的備份類型。 | Always On AG 次要複本不支援完整和差異備份。 | <ul><li>如果您已觸發臨機操作備份，則請觸發主要節點上的備份。</li><li>如果是透過原則來排程備份的話，請確定主要節點已註冊。 若要註冊節點，請[遵循步驟來探索 SQL Server 資料庫](backup-azure-sql-database.md#discover-sql-server-databases)。</li></ul> | 

## <a name="restore-failures"></a>還原失敗

還原作業失敗時會顯示下列錯誤碼。

### <a name="usererrorcannotrestoreexistingdbwithoutforceoverwrite"></a>UserErrorCannotRestoreExistingDBWithoutForceOverwrite 

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 目標位置上已經有同名的資料庫存在。 | 目標還原目的地已經有同名資料庫。  | <ul><li>變更目標資料庫名稱</li><li>或者，使用還原頁面中的強制覆寫選項</li> |

### <a name="usererrorrestorefaileddatabasecannotbeofflined"></a>UserErrorRestoreFailedDatabaseCannotBeOfflined

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 還原失敗，原因是無法讓資料庫離線。 | 執行還原時，必須讓目標資料庫離線。 Azure 備份無法讓這項資料離線。 | 使用 Azure 入口網站錯誤功能表中的其他詳細資料，來縮小根本原因。 如需詳細資訊，請參閱 [SQL 文件](https://docs.microsoft.com/sql/relational-databases/backup-restore/restore-a-database-backup-using-ssms)。 |


###  <a name="usererrorcannotfindservercertificatewiththumbprint"></a>UserErrorCannotFindServerCertificateWithThumbprint

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 在目標上找不到具有指紋的伺服器憑證。 | 目的地執行個體上的 master 資料庫沒有有效的加密指紋。 | 將來源執行個體上所使用的有效憑證指紋，匯入到目標執行個體。 |

## <a name="registration-failures"></a>註冊失敗

下面是註冊失敗的錯誤碼。

### <a name="fabricsvcbackuppreferencecheckfailedusererror"></a>FabricSvcBackupPreferenceCheckFailedUserError 

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| 無法符合 SQL Always On 可用性群組的備份喜好設定，原因是可用性群組有某些節點並未註冊。 | 執行備份所需的節點未註冊或無法連線。 | <ul><li>確定對此資料庫執行備份所需的所有節點均已註冊且狀況良好，然後重試該作業。</li><li>變更 SQL Always On 可用性群組的備份喜好設定。</li></ul> |

### <a name="vmnotinrunningstateusererror"></a>VMNotInRunningStateUserError

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| SQL 伺服器 VM 已關閉，無法供 Azure 備份服務存取。 | VM 已關閉 | 確定 SQL 伺服器正在執行。 |

### <a name="guestagentstatusunavailableusererror"></a>GuestAgentStatusUnavailableUserError

| 錯誤訊息 | 可能的原因 | 建議的動作 |
|---|---|---|
| Azure 備份服務使用 Azure VM 客體代理程式來執行備份，但目標伺服器上無法使用客體代理程式。 | 客體代理程式未啟用，或其狀況不良 | 手動[安裝 VM 客體代理程式](../virtual-machines/extensions/agent-windows.md)。 |

## <a name="next-steps"></a>後續步驟

如需 SQL Server VM 的 Azure 備份 (公開預覽) 詳細資訊，請參閱 [SQL VM 的 Azure 備份 (公開預覽)](../virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery.md#azbackup)。