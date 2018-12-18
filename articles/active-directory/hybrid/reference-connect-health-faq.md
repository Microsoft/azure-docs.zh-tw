---
title: Azure Active Directory Connect Health 常見問題集 - Azure | Microsoft Docs
description: 此常見問題集會回答 Azure AD Connect Health 的相關問題。 這個常見問題集涵蓋使用服務的相關問題，包括計費模型、功能、限制及支援。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: curtand
ms.assetid: f1b851aa-54d7-4cb4-8f5c-60680e2ce866
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: billmath
ms.openlocfilehash: 264e7f3d885535304470c833e228660ba51fefc6
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306387"
---
# <a name="azure-ad-connect-health-frequently-asked-questions"></a>Azure AD Connect Health 常見問題集
本文會回答有關 Azure Active Directory (Azure AD) Connect Health 的常見問題 (FAQ)。 這些常見問題涵蓋如何使用服務的相關問題，包括計費模型、功能、限制及支援。

## <a name="general-questions"></a>一般問題
**問：我管理多個 Azure AD 目錄。我如何切換到包含 Azure Active Directory Premium 的租用戶？**

若要在不同的 Azure AD 租用戶之間切換，請在右上角選取目前登入的 [使用者名稱]，然後選擇適當的帳戶。 如果此處未列出帳戶，請選取 [登出]，然後使用已啟用 Azure Active Directory Premium 之目錄的全域系統管理員認證登入。

**問︰Azure AD Connect Health 支援哪個版本的身分識別角色？**

下表列出各角色與支援的作業系統版本。

|角色| 作業系統/版本|
|--|--|
|Active Directory Federation Services (AD FS)| <ul> <li> Windows Server 2008 R2 </li><li> Windows Server 2012  </li> <li>Windows Server 2012 R2 </li> <li> Windows Server 2016  </li> </ul>|
|Azure AD Connect | 版本 1.0.9125 或更高版本|
|Active Directory Domain Services (AD DS)| <ul> <li> Windows Server 2008 R2 </li><li> Windows Server 2012  </li> <li>Windows Server 2012 R2 </li> <li> Windows Server 2016  </li> </ul>|

請注意，服務所提供的功能可能會因為角色和作業系統而有所不同。 換句話說，並非所有作業系統版本都能使用所有功能。 如需詳細資料，請參閱功能描述。

**問︰我需要多少授權才能監視基礎結構？**

* 第一個 Connect Health 代理程式至少需要一個 Azure AD Premium 授權。
* 每多一個註冊代理程式，就需要再 25 個 Azure AD Premium 授權。
* 代理程式計數等於所有受監視角色 (AD FS、Azure AD Connect 及/或 AD DS) 的註冊代理程式總數。
* AAD Connect Health 授權不必對特定使用者指派授權。 您只需具有必要數量的有效授權即可。

您也可以在 [Azure AD 定價頁面](https://aka.ms/aadpricing)找到授權資訊。

範例：

| 已註冊的代理程式 | 需要的授權 | 監視組態範例 |
| ------ | --------------- | --- |
| 1 | 1 | 1 個 Azure AD Connect 伺服器 |
| 2 | 26| 1 個 Azure AD Connect 伺服器、1 個網域控制站 |
| 3 | 51 | 1 個 Active Directory Federation Services (AD FS) 伺服器、1 個 AD FS Proxy、1 個網域控制站 |
| 4 | 76 | 1 個 AD FS 伺服器、1 個 AD FS Proxy、2 個網域控制站 |
| 5 | 101 | 1 個 Azure AD Connect 伺服器、1 個 AD FS 伺服器、1 個 AD FS Proxy、2 個網域控制站 |

**問：Azure AD Connect Health 是否支援 Azure 德國雲端？**

德國雲端不支援 Azure AD Connect Health，但[同步處理錯誤報告功能](how-to-connect-health-sync.md#object-level-synchronization-error-report-preview)除外。 

| 角色 | 特性 | 德國雲端中支援 |
| ------ | --------------- | --- |
| 適用於同步處理的 Connect Health | 監視/見解/警示/分析 | 否 |
|  | 同步處理錯誤報告 | 是 |
| 適用於 ADFS 的 Connect Health | 監視/見解/警示/分析 | 否 |
| 適用於 ADDS 的 Connect Health | 監視/見解/警示/分析 | 否 |

若要確保適用於同步處理的 Connect Health 的代理程式連線能力，請據此設定[安裝需求](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints)。   

## <a name="installation-questions"></a>安裝問題

**問：在個別的伺服器上安裝 Azure AD Connect Health 代理程式有什麼影響？**

安裝 Microsoft Azure AD Connect Health 代理程式、AD FS、Web 應用程式 Proxy 伺服器、Azure AD Connect (同步處理) 伺服器、網域控制站，對於 CPU、記憶體耗用量、網路頻寬和儲存體的影響非常小。

以下的數字是近似值：

* CPU 耗用量：增加約 1-5%。
* 記憶體耗用量：最多 10% 的系統總記憶體。

> [!NOTE]
> 如果代理程式無法與 Azure 通訊，則代理程式會在本機儲存資料，最多可達定義的上限。 代理程式會根據「最近最少服務」的準則來覆寫「快取」資料。
>
>

* Azure AD Connect Health 代理程式的本機緩衝區儲存體：約 20 MB。
* 對於 AD FS 伺服器，建議您為 AD FS 稽核通道佈建 1024 MB (1 GB) 的磁碟空間，Azure AD Connect Health 代理程式才能在所有稽核資料遭到覆寫前加以處理。

**問：在安裝 Azure AD Connect Health 代理程式期間，我是否需要重新啟動我的伺服器？**

否。 安裝代理程式不需要您重新啟動伺服器。 不過，安裝某些先決條件的步驟可能需要重新啟動伺服器。

例如，在 Windows Server 2008 R2 上安裝 .NET 4.5 Framework 需要重新啟動伺服器。

**問：Azure AD Connect Health 是否透過傳遞 Http Proxy 運作？**

是。 若是進行中的作業，您可以將 Health 代理程式設定為使用 HTTP Proxy 來轉送輸出 HTTP 要求。
深入了解[設定 Health 代理程式的 HTTP Proxy](how-to-connect-health-agent-install.md#configure-azure-ad-connect-health-agents-to-use-http-proxy)。

如果需要在代理程式註冊期間設定 Proxy，您可能需要預先修改 Internet Explorer 的 Proxy 設定。

1. 開啟 Internet Explorer > [設定]  >  [網際網路選項]  >  [連線]  >  [LAN 設定]。
2. 選取 [在您的區域網路使用 Proxy 伺服器]。
3. 如果您有不同的 Proxy 連接埠供 HTTP 和 HTTPS/安全使用，請選取 [進階]。

**問：連線到 HTTP Proxy 時，Azure AD Connect Health 是否支援基本驗證？**

否。 目前不支援為基本驗證指定任意使用者名稱和密碼的機制。

**問：我需要開放哪些防火牆連接埠，Azure AD Connect Health 代理程式才能運作？**

請參閱[需求](how-to-connect-health-agent-install.md#requirements)一節，以取得防火牆連接埠清單與其他的連線需求。

**問︰為什麼我會在 Azure AD Connect Health 入口網站中看到兩部名稱相同的伺服器？**

當您從伺服器移除代理程式時，系統不會自動從 Azure AD Connect Health 入口網站移除伺服器。 如果您以手動方式從伺服器移除代理程式，或移除伺服器本身，就需要以手動方式從 Azure AD Connect Health 入口網站中刪除伺服器項目。

您可能會重新安裝伺服器的映像，或以相同的細節 (例如電腦名稱) 建立新的伺服器。 如果您並未從 Azure AD Connect Health 入口網站移除已經註冊的伺服器，就在新伺服器上安裝代理程式，您可能會看到兩個同名的項目。

在此情況下，請手動刪除屬於舊伺服器的項目。 此伺服器的資料應該已過時。

## <a name="health-agent-registration-and-data-freshness"></a>Health 代理程式註冊和資料有效性

**問︰Health 代理程式註冊失敗的常見原因為何？該如何解決問題？**

Health 代理程式會因為下列可能原因而無法註冊：

* 因為防火牆封鎖流量，代理程式無法與所需端點通訊。 在 Web 應用程式 Proxy 伺服器上尤其常見。 請確定您已允許針對所需端點和連接埠的輸出通訊。 如需詳細資訊，請參閱[需求](how-to-connect-health-agent-install.md#requirements)一節。
* 輸出通訊會在網路層遇到 SSL 檢查。 這會導致代理程式所使用的憑證遭到檢查伺服器/實體所取代，完成代理程式註冊的步驟便會失敗。
* 使用者沒有執行代理程式註冊的存取權。 根據預設，全域系統管理員具有存取權。 您可以使用[角色型存取控制](how-to-connect-health-operations.md#manage-access-with-role-based-access-control)將存取權委派給其他使用者。

**問︰我收到有關「Health 服務資料不是最新狀態」的警示。我該如何進行解決這個問題？**

當 Azure AD Connect Health 在過去 2 小時沒有從伺服器收到所有資料點，就會產生此警示。 [閱讀更多資訊](how-to-connect-health-data-freshness.md)。

## <a name="operations-questions"></a>操作問題
**問︰是否需要在 Web 應用程式 Proxy 伺服器上啟用稽核？**

否，不需要在 Web 應用程式 Proxy 伺服器上啟用稽核。

**問：Azure AD Connect Health 警示如何獲得解決？**

Azure AD Connect Health 警示會在成功情況下獲得解決。 Azure AD Connect Health 代理程式會定期偵測成功情況，並向服務回報。 對於少數幾個警示，隱藏是以時間為基礎。 也就是說，如果在警示產生的 72 小時內未觀察到相同的錯誤狀況，就會自動解決警示。

**問：我收到下列警示：「測試驗證要求 (綜合交易) 無法取得權杖」。我該如何進行解決這個問題？**

在健康狀態代理程式所起始的綜合交易期間，AD FS 伺服器上安裝的健康狀態代理程式無法取得權杖時，Azure AD Connect Health for AD FS 會產生警示。 健康狀態代理程式使用本機系統內容，並嘗試取得自我信賴憑證者的權杖。 這是 catch-all 測試，可確保 AD FS 處於發行權杖狀態。

這項測試經常失敗，因為健康狀態代理程式無法解析 AD FS 伺服器陣列名稱。 如果 AD FS 伺服器受到網路負載平衡器保護，並且從受到負載平衡器保護的節點起始要求 (相較於不受負載平衡器保護的一般用戶端)，可能會發生此問題。 更新 "C:\Windows\System32\drivers\etc" 下的 "hosts" 檔案，使其包含 AD FS 伺服器的 IP 位址或 AD FS 伺服器陣列名稱 (例如 sts.contoso.com) 的迴圈 IP 位址 (127.0.0.1)，即可修正此問題。 新增主機檔案會讓網路呼叫短路，如此可讓健康狀態代理程式取得權杖。

**問：我收到一封電子郵件，指出未針對最近的勒索軟體攻擊修補我的電腦。我為什麼收到這封電子郵件？**

Azure AD Connect Health 服務已掃描所有已監視的電腦，確保已安裝的必要修補程式。 如果至少一部電腦沒有重大修補程式，則已將電子郵件傳送給租用戶系統管理員。 下列邏輯已用來進行這項決定。
1. 尋找電腦上安裝的所有 Hotfix。
2. 檢查定義的清單中是否有至少一個 Hotfix。
3. 如果有，則電腦受到保護。 如果沒有，則電腦會有受攻擊的風險。

您可以使用下列 PowerShell 指令碼手動執行這項檢查。 它會實作上述邏輯。

```
Function CheckForMS17-010 ()
{
    $hotfixes = "KB3205409", "KB3210720", "KB3210721", "KB3212646", "KB3213986", "KB4012212", "KB4012213", "KB4012214", "KB4012215", "KB4012216", "KB4012217", "KB4012218", "KB4012220", "KB4012598", "KB4012606", "KB4013198", "KB4013389", "KB4013429", "KB4015217", "KB4015438", "KB4015546", "KB4015547", "KB4015548", "KB4015549", "KB4015550", "KB4015551", "KB4015552", "KB4015553", "KB4015554", "KB4016635", "KB4019213", "KB4019214", "KB4019215", "KB4019216", "KB4019263", "KB4019264", "KB4019472", "KB4015221", "KB4019474", "KB4015219", "KB4019473"

    #checks the computer it's run on if any of the listed hotfixes are present
    $hotfix = Get-HotFix -ComputerName $env:computername | Where-Object {$hotfixes -contains $_.HotfixID} | Select-Object -property "HotFixID"

    #confirms whether hotfix is found or not
    if (Get-HotFix | Where-Object {$hotfixes -contains $_.HotfixID})
    {
        "Found HotFix: " + $hotfix.HotFixID
    } else {
        "Didn't Find HotFix"
    }
}

CheckForMS17-010

```

**問：為什麼 PowerShell Cmdlet <i>Get MsolDirSyncProvisioningError</i> 在結果中顯示的同步錯誤比較少？**

<i>Get MsolDirSyncProvisioningError</i> 只會傳回 DirSync 佈建錯誤。 除此之外，Connect Health 入口網站還會顯示其他同步錯誤類型，例如匯出錯誤。 這與 Azure AD Connect 差異結果一致。 深入瞭解 [Azure AD Connect 同步錯誤](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-troubleshoot-sync-errors)。

**問：為何系統並未產生我的 ADFS 稽核？**

請使用 PowerShell Cmdlet <i>Get-AdfsProperties -AuditLevel</i> 以確保稽核記錄不是處於停用狀態。 深入了解 [ADFS 稽核記錄](https://docs.microsoft.com/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server#auditing-levels-in-ad-fs-for-windows-server-2016)。 請注意，若系統將進階稽核設定推送至 ADFS 伺服器，將會覆寫 auditpol.exe 的任何變更 (即使並未設定產生的應用程式)。 在此情況下，請將本機本機安全性原則設定為記錄已產生應用程式的失敗和成功項目。 


## <a name="related-links"></a>相關連結
* [Azure AD Connect Health](whatis-hybrid-identity-health.md)
* [Azure AD Connect Health 代理程式安裝](how-to-connect-health-agent-install.md)
* [Azure AD Connect Health 操作](how-to-connect-health-operations.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](how-to-connect-health-adfs.md)
* [使用 Azure AD Connect Health 進行同步處理](how-to-connect-health-sync.md)
* [在 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)
* [Azure AD Connect Health 版本歷程記錄](reference-connect-health-version-history.md)
