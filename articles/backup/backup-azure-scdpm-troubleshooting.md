---
title: 使用 Azure 備份針對 System Center Data Protection Manager 進行疑難排解
description: 針對 System Center Data Protection Manager 的問題進行疑難排解。
services: backup
author: adigan
manager: shreeshd
ms.service: backup
ms.topic: conceptual
ms.date: 11/24/2017
ms.author: adigan
ms.openlocfilehash: d3776df8184523999433059e95bc72e1d3abb1c7
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34606438"
---
# <a name="troubleshoot-system-center-data-protection-manager"></a>針對 System Center Data Protection Manager 疑難排解

本文說明您在使用 Data Protection Manager 時可能遇到的問題解決方案。

如需 System Center Data Protection Manager 的最新版本資訊，請參閱 [System Center 文件](https://docs.microsoft.com/system-center/dpm/dpm-release-notes?view=sc-dpm-2016)。 您可以在[此對照表](https://docs.microsoft.com/system-center/dpm/dpm-protection-matrix?view=sc-dpm-2016)中深入了解對於 Data Protection Manager 的支援。


## <a name="error-replica-is-inconsistent"></a>錯誤：複本不一致

複本不一致的原因如下：
- 複本建立作業失敗。
- 變更日誌發生問題。
- 磁碟區層級篩選點陣圖包含錯誤。
- 來源電腦意外關機。
- 同步處理記錄溢位。
- 複本確實不一致。

若要解決此問題，請執行下列動作：
- 若要移除不一致狀態，請手動執行一致性檢查或安排每日一致性檢查。
- 確定您使用的是最新版的 Microsoft Azure 備份伺服器和 Data Protection Manager。
- 確定已啟用**自動一致性**設定。
- 嘗試從命令提示字元重新啟動服務。 使用 `net stop dpmra` 命令，然後使用 `net start dpmra`。
- 確定您符合網路連線能力和頻寬需求。
- 檢查來源電腦是否意外關機。
- 確定磁碟狀況良好且可為複本提供足夠空間。
- 確定不會同時執行重複的備份作業。

## <a name="error-online-recovery-point-creation-failed"></a>錯誤：線上復原點建立失敗

若要解決此問題，請執行下列動作：
- 確定您使用的是最新版的 Azure 備份代理程式。
- 嘗試在保護工作區中手動建立復原點。
- 確定您會在資料來源上執行一致性檢查。
- 確定您符合網路連線能力和頻寬需求。
- 當複本資料處於不一致狀態時，建立此資料來源的磁碟復原點。
- 確定複本存在且未遺失。
- 確定複本有足夠空間，可建立更新序列號碼 (USN) 日誌。

## <a name="error-unable-to-configure-protection"></a>錯誤：無法設定保護

當 Data Protection Manager 伺服器無法連絡受保護的伺服器時，就會發生此錯誤。 

若要解決此問題，請執行下列動作：
- 確定您使用的是最新版的 Azure 備份代理程式。
- 確定 Data Protection Manager 伺服器與受保護伺服器之間有連線能力 (網路/防火牆/Proxy)。
- 如果您要保護 SQL Server，請確定 [登入屬性] > **NT AUTHORITY\SYSTEM** 屬性會顯示已啟用 **sysadmin** 設定。

## <a name="error-server-not-registered-as-specified-in-vault-credential-file"></a>錯誤：未以保存庫認證檔案中指定的方式註冊伺服器

此錯誤會在 Data Protection Manager/Azure 備份伺服器資料的復原程序期間發生。 復原程序中所使用的保存庫認證檔案不屬於 Data Protection Manager/Azure 備份伺服器的復原服務保存庫。

若要解決此問題，請執行下列步驟：
1. 從已註冊 Data Protection Manager/Azure 備份伺服器的復原服務保存庫下載保存庫認證檔。
2. 嘗試使用最新下載的保存庫認證檔，向保存庫註冊伺服器。

## <a name="error-no-recoverable-data-or-selected-server-not-a-data-protection-manager-server"></a>錯誤：沒有可復原的資料或選取的伺服器不是 Data Protection Manager 伺服器

此錯誤的發生原因如下：
- 未向復原服務保存庫註冊任何其他 Data Protection Manager/Azure 備份伺服器。
- 伺服器尚未上傳中繼資料。
- 選取的伺服器不是 Data Protection Manager/Azure 備份伺服器。

在向復原服務保存庫註冊其他 Data Protection Manager/Azure 備份伺服器時，執行下列步驟來解決此問題：
1. 確定已安裝最新的 Azure 備份代理程式。
2. 確定已安裝最新代理程式之後，請等候一天，然後再開始復原程序。 夜間備份作業會將所有受保護備份的中繼資料上傳至雲端。 之後就能使用備份資料進行復原。

## <a name="error-provided-encryption-passphrase-doesnt-match-passphrase-for-server"></a>錯誤：提供的加密複雜密碼不符合伺服器的複雜密碼

此錯誤會在復原 Data Protection Manager/Azure 備份伺服器資料時的加密程序期間發生。 復原程序中使用的加密複雜密碼不符合伺服器的加密複雜密碼。 如此一來，代理程式就無法將資料解密，而復原會失敗。

> [!IMPORTANT]
> 如果您忘記或遺失加密複雜密碼，就沒有任何其他方法可用來復原資料。 唯一的選項是重新產生複雜密碼。 使用新的複雜密碼來加密未來的備份資料。
>
> 當您要復原資料時，一定要提供與 Data Protection Manager/Azure 備份伺服器相關聯的相同加密複雜密碼。 
>
