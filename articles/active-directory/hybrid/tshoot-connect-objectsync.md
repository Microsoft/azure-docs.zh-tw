---
title: Azure AD Connect：針對物件同步處理進行疑難排解 | Microsoft Docs
description: 本主題提供如何使用疑難排解工作來針對物件同步處理問題進行疑難排解的步驟。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: curtand
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/15/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: c810e121b751d098bd0fbda09db51c031f003460
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310237"
---
# <a name="troubleshoot-object-synchronization-with-azure-ad-connect-sync"></a>針對使用 Azure AD Connect 同步所執行的物件同步處理進行疑難排解
本文提供使用疑難排解工作，針對物件同步處理問題進行疑難排解的步驟。 若要查看疑難排解如何在 Azure Active Directory (Azure AD) Connect 中運作，請觀賞[這段短片](https://aka.ms/AADCTSVideo)。

## <a name="troubleshooting-task"></a>疑難排解工作
對於 1.1.749.0 版或更新版本的 Azure AD Connect 部署，請在精靈中使用疑難排解工作，針對物件同步處理問題進行疑難排解。 對於先前的版本，則請以[這裡](tshoot-connect-object-not-syncing.md)所述的方式手動疑難排解。

### <a name="run-the-troubleshooting-task-in-the-wizard"></a>在精靈中執行疑難排解工作
若要在精靈中執行疑難排解工作，請執行下列步驟：

1.  在您的 Azure AD Connect 伺服器上，使用 [以系統管理員身分執行] 選項開啟新的 Windows PowerShell 工作階段。
2.  執行 `Set-ExecutionPolicy RemoteSigned` 或 `Set-ExecutionPolicy Unrestricted`。
3.  啟動 Azure AD Connect 精靈。
4.  瀏覽至 [其他工作] 頁面，選取 [疑難排解]，然後按 [下一步]。
5.  在 [疑難排解] 頁面上，按一下 [啟動]，以在 PowerShell 中啟動疑難排解功能表。
6.  在主功能表中，選取 [針對物件同步處理進行疑難排解]。
![](media\tshoot-connect-objectsync\objsynch11.png)

### <a name="troubleshooting-input-parameters"></a>針對輸入參數進行疑難排解
疑難排解工作需要下列輸入參數：
1.  **物件辨別名稱** – 這是需要疑難排解之物件的辨別名稱
2.  **AD 連接器名稱** – 這是上述物件所在之 AD 樹系的名稱。
3.  Azure AD 租用戶全域管理員認證 ![](media\tshoot-connect-objectsync\objsynch1.png)

### <a name="understand-the-results-of-the-troubleshooting-task"></a>了解疑難排解工作的結果
疑難排解工作會執行下列檢查：

1.  偵測是否有 UPN 不符的情形 (如果物件是同步處理至 Azure Active Directory)
2.  檢查是否因為網域篩選而篩選物件
3.  檢查是否因為 OU 篩選而篩選物件
4.  檢查是否因為連結信箱，使物件同步遭到封鎖
5. 檢查物件是否是動態通訊群組，不應進行同步

本節其餘部分會說明工作所傳回的特定結果。 在每個案例中，工作都會提供分析並接著提供可用來解決問題的建議動作。

## <a name="detect-upn-mismatch-if-object-is-synced-to-azure-active-directory"></a>偵測是否有 UPN 不符的情形 (如果物件是同步處理至 Azure Active Directory)
### <a name="upn-suffix-is-not-verified-with-azure-ad-tenant"></a>系統不會向 Azure AD 租用戶確認 UPN 後置詞
當系統未向 Azure AD 租用戶確認 UserPrincipalName (UPN)/替代登入識別碼後置詞時，Azure Active Directory 就會將 UPN 後置詞替換為預設網域名稱 "onmicrosoft.com"。

![](media\tshoot-connect-objectsync\objsynch2.png)

### <a name="changing-upn-suffix-from-one-federated-domain-to-another-federated-domain"></a>將 UPN 後置詞從某個同盟網域變更為另一個同盟網域
Azure Active Directory 不允許將 UserPrincipalName (UPN)/替代登入識別碼後置詞的變更從某個同盟網域同步處理至另一個同盟網域。 這適用於向 Azure AD 租用戶確認且驗證類型為「同盟」的網域。

![](media\tshoot-connect-objectsync\objsynch3.png) 

### <a name="azure-ad-tenant-dirsync-feature-synchronizeupnformanagedusers-is-disabled"></a>已停用 Azure AD 租用戶 DirSync 功能 ‘SynchronizeUpnForManagedUsers’
當 Azure AD 租用戶 DirSync 功能 ‘SynchronizeUpnForManagedUsers’ 停用時，Azure Active Directory 不允許對具有受控驗證的授權使用者帳戶進行 UserPrincipalName/替代登入識別碼同步處理更新。

![](media\tshoot-connect-objectsync\objsynch4.png)

## <a name="object-is-filtered-due-to-domain-filtering"></a>因為網域篩選而篩選物件
### <a name="domain-is-not-configured-to-sync"></a>網域未設定為要同步處理
物件會因為未設定網域而超出範圍。 在下列範例中，由於物件所屬的網域已從同步處理篩選出來，所以物件會超出同步範圍。

![](media\tshoot-connect-objectsync\objsynch5.png)

### <a name="domain-is-configured-to-sync-but-is-missing-run-profilesrun-steps"></a>網域設定為要同步處理，但遺漏執行設定檔/執行步驟
物件會因為網域遺漏執行設定檔/執行步驟而超出範圍。 在下列範例中，由於物件所屬的網域遺漏完整匯入執行設定檔的執行步驟，所以物件會超出同步範圍。
![](media\tshoot-connect-objectsync\objsynch6.png)

## <a name="object-is-filtered-due-to-ou-filtering"></a>因為 OU 篩選而篩選物件
由於 OU 篩選設定所致，此物件會超出同步範圍。 在下列範例中，物件屬於 OU=NoSync,DC=bvtadwbackdc,DC=com。  這個 OU 不會包含在同步範圍內。</br>

![OU](./media/tshoot-connect-objectsync/objsynch7.png)

## <a name="linked-mailbox-issue"></a>連結信箱問題
連結信箱應與位在另一個信任帳戶樹系中的外部主要帳戶產生關聯。 如果沒有這類外部主要帳戶，則 Azure AD Connect 不會將對應至 Exchange 樹系中連結信箱的使用者帳戶同步至 Azure AD 租用戶。</br>
![連結信箱](./media/tshoot-connect-objectsync/objsynch12.png)

## <a name="dynamic-distribution-group-issue"></a>動態通訊群組問題
由於內部部署 Active Directory 和 Azure Active Directory 之間的各種差異，Azure AD Connect 不會將動態通訊群組同步至 Azure AD 租用戶。

![動態通訊群組](./media/tshoot-connect-objectsync/objsynch13.png)

## <a name="html-report"></a>HTML 報告
除了分析物件外，疑難排解工作也會產生具有物件一切已知資訊的 HTML 報告。 如有需要，可與支援小組分享這個 HTML 報告以進行進一步的疑難排解。

![](media\tshoot-connect-objectsync\objsynch8.png)

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
