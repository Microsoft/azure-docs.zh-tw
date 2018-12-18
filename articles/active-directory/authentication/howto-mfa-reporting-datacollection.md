---
title: Azure Multi-Factor Authentication 使用者資料收集
description: Azure Multi-Factor Authentication 會使用哪些資訊協助進行使用者驗證？
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 1b380bc20c9f80710ca62672b99649ce3498a8e8
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39223757"
---
# <a name="azure-multi-factor-authentication-user-data-collection"></a>Azure Multi-Factor Authentication 使用者資料收集

本文件說明當您想要移除使用者資訊時，該如何找出 Azure Multi-Factor Authentication Server (MFA Server) 和 Azure MFA (雲端式) 所收集的資料。

[!INCLUDE [gdpr-hybrid-note](../../../includes/gdpr-hybrid-note.md)]

## <a name="information-collected"></a>收集的資訊

MFA Server、NPS 擴充功能和 Windows Server 2016 Azure MFA AD FS Adapter 會收集下列資訊並儲存 90 天。

驗證嘗試 (用於報告和疑難排解)：

- Timestamp
- 使用者名稱
- 名字
- 姓氏
- 電子郵件地址
- 使用者群組
- 驗證方法 (通話、簡訊、行動應用程式、OATH 權杖)
- 通話模式 (標準、PIN)
- 簡訊方向 (單向、雙向)
- 簡訊模式 (OTP、OTP + PIN)
- 行動應用程式模式 (標準、PIN)
- OATH 權杖模式 (標準、PIN)
- 驗證類型
- 應用程式名稱
- 主要通話國碼 (地區碼)
- 主要通話電話號碼
- 主要通話分機
- 已驗證主要通話
- 主要通話結果
- 備用通話國碼 (地區碼)
- 備用通話電話號碼
- 備用通話分機
- 已驗證備用通話
- 備用通話結果
- 已整體驗證
- 整體結果
- 結果
- 已驗證
- 結果
- 正在起始 IP 位址
- 裝置
- 裝置權杖
- 裝置類型
- 行動應用程式版本
- 作業系統版本
- 結果
- 檢查是否有通知 (已使用)

啟用 (嘗試啟用 Microsoft Authenticator 行動應用程式中的帳戶)：
- 使用者名稱
- 帳戶名稱
- Timestamp
- 取得啟用代碼結果
- 啟用成功
- 啟用錯誤
- 啟用狀態結果
- 裝置名稱
- 裝置類型
- 應用程式版本
- 啟用的 OATH 權杖

封鎖 (用來判斷已封鎖的狀態並用於報告)：

- 封鎖時間戳記
- 依使用者名稱封鎖
- 使用者名稱
- 國碼 (地區碼)
- 電話號碼
- 設有格式的電話號碼
- 分機
- 清除分機
- Blocked
- 封鎖原因
- 完成時間戳記
- 完成原因
- 帳戶鎖定
- 詐騙警示
- 不封鎖詐騙警示
- 語言

略過 (用於報告)：

- 略過時間戳記
- 略過秒數
- 依使用者名稱略過
- 使用者名稱
- 國碼 (地區碼)
- 電話號碼
- 設有格式的電話號碼
- 分機
- 清除分機
- 略過原因
- 完成時間戳記
- 完成原因
- 已使用的略過

變更 (用來同步處理 MFA Server 或 AAD 的 使用者變更)：

- 變更時間戳記
- 使用者名稱
- 新國碼 (地區碼)
- 新電話號碼
- 新分機
- 新備用國碼 (地區碼)
- 新備用電話號碼
- 新備用分機
- 新 PIN
- 需要變更 PIN
- 舊裝置權杖
- 新裝置權杖

## <a name="gather-data-from-mfa-server"></a>從 MFA Server 收集資料

若為 MFA Server 8.0 版或更新版本，系統管理員可按照以下流程匯出使用者的所有資料：

- 登入 MFA Server，瀏覽至 [使用者] 索引標籤，選取所需的使用者，然後按一下 [編輯] 按鈕。 建立每個索引標籤的螢幕擷取畫面 (Alt-PrtScn)，以便將目前的 MFA 設定提供給使用者。
- 以 MFA Server 的命令列執行下列命令，根據您的安裝 `C:\Program Files\Multi-Factor Authentication Server\MultiFactorAuthGdpr.exe export <username>` 來變更路徑，以產生 JSON 格式的檔案。
- 系統管理員也可以選擇使用 Web 服務 SDK GetUserGdpr 作業，匯出針對特定使用者收集的所有 MFA 雲端服務資訊，或合併到較大的報告解決方案之中。
- 在 `C:\Program Files\Multi-Factor Authentication Server\Logs\MultiFactorAuthSvc.log` 與任何備份之中搜尋 “<username>” (搜尋時加入引號)，即可找出所有新增或變更的使用者記錄執行個體。
   - 在 MFA Server UX [記錄] 區段的 [記錄檔] 索引標籤中，取消勾選 [記錄使用者變更]，即可限制這些記錄 (而非刪除)。
   - 如果已設定 syslog，而且已在 MFA Server UX [記錄] 區段的 [Syslog] 索引標籤中勾選 [記錄使用者變更]，則會改從 syslog 收集記錄項目。
- 其他存在於 MultiFactorAuthSvc.log 使用者名稱以及其他與驗證企圖有關的 MFA Server 記錄檔，則視為仍在使用中，且與使用 MultiFactorAuthGdpr.exe export 或 Web Service SDK GetUserGdpr 所提供的資訊重複。

## <a name="delete-data-from-mfa-server"></a>刪除 MFA Server 中的資料

以 MFA Server 的命令列執行下列命令，根據您的安裝 `C:\Program Files\Multi-Factor Authentication Server\MultiFactorAuthGdpr.exe delete <username>` 來變更路徑，以刪除為該位使用者收集的所有 MFA 雲端服務資訊。

- 即時刪除匯出中所含的資料，但是可能需要長達 30 天才會生效或完全移除重複的資料。
- 系統管理員也可以選擇使用 Web 服務 SDK DeleteUserGdpr 作業，刪除針對特定使用者收集的所有 MFA 雲端服務資訊，或合併到較大的報告解決方案之中。

## <a name="gather-data-from-nps-extension"></a>從 NPS 延伸模組收集資料

使用 [Microsoft 隱私權入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview)提出匯出要求。

- MFA 資訊會包含在匯出中，可能需要數小時或數天才能完成。
- AzureMfa/AuthN/AuthNOptCh、AzureMfa/AuthZ/AuthZAdminCh 和 AzureMfa/AuthZ/AuthZOptCh 事件記錄中出現的使用者名稱可視為仍在使用中，而且與匯出中提供的資訊相重複。

## <a name="delete-data-from-nps-extension"></a>刪除 NPS 擴充功能中的資料

使用 [Microsoft 隱私權入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview)提出帳戶關閉的要求，刪除為該位使用者收集的所有 MFA 雲端服務資訊。

- 可能需要 30 天才能完全移除資料。

## <a name="gather-data-from-windows-server-2016-azure-mfa-ad-fs-adapter"></a>從 Windows Server 2016 Azure MFA AD FS Adapter 收集資料

使用 [Microsoft 隱私權入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview)提出匯出要求。 

- MFA 資訊會包含在匯出中，可能需要數小時或數天才能完成。
- AD FS 追蹤/偵錯事件記錄檔 (如果已啟用) 中出現的使用者名稱會視為仍在使域中，而且與匯出中提供的資訊相重複。

## <a name="delete-data-from-windows-server-2016-azure-mfa-ad-fs-adapter"></a>刪除 Windows Server 2016 Azure MFA AD FS Adapter 中的資料

使用 [Microsoft 隱私權入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview)提出帳戶關閉的要求，刪除為該位使用者收集的所有 MFA 雲端服務資訊。

- 可能需要 30 天才能完全移除資料。

## <a name="gather-data-for-azure-mfa"></a>收集 Azure MFA 的資料

使用 [Microsoft 隱私權入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview)提出匯出要求。

- MFA 資訊會包含在匯出中，可能需要數小時或數天才能完成。

## <a name="delete-data-for-azure-mfa"></a>刪除 Azure MFA 的資料

使用 [Microsoft 隱私權入口網站](https://portal.azure.com/#blade/Microsoft_Azure_Policy/UserPrivacyMenuBlade/Overview)提出帳戶關閉的要求，刪除為該位使用者收集的所有 MFA 雲端服務資訊。

- 可能需要 30 天才能完全移除資料。

## <a name="next-steps"></a>後續步驟

[MFA Server 報告](howto-mfa-reporting.md)
