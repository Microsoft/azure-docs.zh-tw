---
title: Azure AD Connect 同步：防止意外刪除 |Microsoft Docs
description: 本主題說明 Azure AD Connect 中的防止意外刪除 (可防止意外刪除) 功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 6b852cb4-2850-40a1-8280-8724081601f7
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 6a535d776da216029b905fb604ed378449a81082
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309707"
---
# <a name="azure-ad-connect-sync-prevent-accidental-deletes"></a>Azure AD Connect 同步處理：防止意外刪除
本主題說明 Azure AD Connect 中的防止意外刪除 (可防止意外刪除) 功能。

安裝 Azure AD Connect 時，依預設會啟用防止意外刪除的功能，並設定為不允許超過 500 個刪除項目的匯出。 這項功能是專門用來保護您免於意外的組態變更及內部部署目錄的變更，因為這會影響許多使用者和其他物件。

## <a name="what-is-prevent-accidental-deletes"></a>防止意外刪除是什麼
會看到多項刪除的常見案例包括：

* 變更未選取整個 [OU](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering) 或[網域](how-to-connect-sync-configure-filtering.md#domain-based-filtering)的[篩選](how-to-connect-sync-configure-filtering.md)。
* OU 中的所有物件遭到刪除。
* 重新命名 OU，結果使得其中的所有物件被視為不在同步處理範圍內。

您可以使用 PowerShell 的 `Enable-ADSyncExportDeletionThreshold` (這屬於隨 Azure Active Directory Connect 一起安裝的 AD Sync 模組) 進行變更的預設值是 500 個物件。 您應該將此值設定為符合您組織的大小。 由於同步排程器會每隔 30 分鐘執行一次，因此該值是 30 分鐘內看到的刪除數目。

如果有太多刪除預備要匯出到 Azure AD，則匯出會停止，且您會收到如下的電子郵件：

![防止意外刪除電子郵件](./media/how-to-connect-sync-feature-prevent-accidental-deletes/email.png)

> *Hello (技術連絡人)。有時身分識別同步處理服務偵測到的刪除數目會超過 (組織名稱) 所設定的刪除閾值。本次執行身分識別同步處理時，共傳送 (數目) 個物件進行刪除。這已到達或超過設定的 (數目) 個物件的刪除閾值。您需先確認要刪除這些項目，才可繼續進行。若要深入了解此電子郵件中列出的錯誤，請參見防止意外刪除。*
>
> 

當您查看匯出設定檔的 **Synchronization Service Manager** UI，您也會看到狀態 `stopped-deletion-threshold-exceeded`。
![防止意外刪除 Sync Service Manager UI](./media/how-to-connect-sync-feature-prevent-accidental-deletes/syncservicemanager.png)

如果這是非預期的結果，請進行調查，並採取修正動作。 若要查看哪些物件即將被刪除時，請執行下列作業：

1. 從 [開始] 功能表啟動 [同步處理服務]  。
2. 移至 [連接器] 。
3. 選取 **Azure Active Directory**類型的連接器。
4. 在右側的 [動作] 之下，選取 [搜尋連接器空間]。
5. 在 [範圍] 下的快顯中，選取 [中斷連線起點]，並選擇一個過去的時間。 按一下 [搜尋] 。 此頁面會顯示所有即將刪除的物件。 按一下每個項目，您就可以了解該物件的其他資訊。 您也可以按一下 [資料行設定]  ，新增其他屬性以顯示在方格中。

![搜尋連接器空間](./media/how-to-connect-sync-feature-prevent-accidental-deletes/searchcs.png)

如果想要刪除所有項目，請執行下列作業：

1. 若要擷取目前的刪除閾值，請執行 PowerShell Cmdlet `Get-ADSyncExportDeletionThreshold`。 提供 Azure AD 全域系統管理員帳戶與密碼。 預設值為 500。
2. 若要暫時停用此保護功能並刪除這些項目，請執行 PowerShell Cmdlet： `Disable-ADSyncExportDeletionThreshold`。 提供 Azure AD 全域系統管理員帳戶與密碼。
   ![認證](./media/how-to-connect-sync-feature-prevent-accidental-deletes/credentials.png)
3. 如果 Azure Active Directory Connector 仍處於選取狀態，請選取 [執行] 動作，再選取 [匯出]。
4. 若要重新啟用此保護功能，請執行 PowerShell Cmdlet： `Enable-ADSyncExportDeletionThreshold -DeletionThreshold 500`。 使用您在擷取目前刪除閾值時記下的值來取代 500。 提供 Azure AD 全域系統管理員帳戶與密碼。

## <a name="next-steps"></a>後續步驟
**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
