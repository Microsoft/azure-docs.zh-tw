---
title: 對 Azure Active Directory 活動記錄內容套件錯誤進行疑難排解 | Microsoft Docs
description: 為您提供 Azure Active Directory 活動內容套件的錯誤訊息清單以及修正它們的步驟。
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: ffce7eb1-99da-4ea7-9c4d-2322b755c8ce
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 01/15/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: bf50dbf942dc7a82afbb60455be45b6c4b287ccd
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43782159"
---
# <a name="troubleshooting-azure-active-directory-activity-logs-content-pack-errors"></a>對 Azure Active Directory 活動記錄內容套件錯誤進行疑難排解 

|  |
|--|
|目前，Azure AD Power BI 內容套件會使用 Azure AD Graph API 從您的 Azure AD 租用戶擷取資料。 因此，您可能會發現內容套件中的可用資料與使用 [Microsoft Graph API](concept-reporting-api.md) 擷取以用於報告的資料之間有些許差異。 |
|  |

使用適用於 Azure Active Directory 預覽版的 Power BI 內容套件時，可能會遇到下列錯誤： 

- [重新整理失敗](troubleshoot-content-pack.md#refresh-failed) 
- [無法更新資料來源認證](troubleshoot-content-pack.md#failed-to-update-data-source-credentials) 
- [匯入資料的時間太長](troubleshoot-content-pack.md#importing-of-data-is-taking-too-long) 
 
本文為您提供可能的原因以及如何修正這些錯誤的相關資訊。
 
## <a name="refresh-failed"></a>重新整理失敗 
 
**此錯誤的呈現方式**：從 Power BI 寄送電子郵件，或是重新整理記錄中的失敗狀態。 


| 原因 | 修正方式 |
| ---   | ---        |
| 若連線到內容套件的使用者認證已重設，但卻未在內容套件的連線設定中加以更新，即會導致重新整理失敗錯誤。 | 在 Power BI 中，找出對應到 Azure Active Directory 活動記錄儀表板 (Azure Active Directory 活動記錄) 的資料集、選擇 [排程重新整理]，然後輸入您的 Azure AD 認證。 |
| 重新整理可能會因為基礎內容套件中的資料問題而失敗。 | 提出支援票證。 如需詳細資訊，請參閱[如何取得 Azure Active Directory 支援](../fundamentals/active-directory-troubleshooting-support-howto.md)。|
 
 
## <a name="failed-to-update-data-source-credentials"></a>無法更新資料來源認證 
 
**此錯誤的呈現方式**：在 Power BI 中，當您連線到 Azure Active Directory 活動記錄 (預覽) 內容套件時。 

| 原因 | 修正方式 |
| ---   | ---        |
| 連線的使用者既不是全域系統管理員，也不是安全性讀取者或安全性系統管理員。 | 使用非全域系統管理員或安全性讀取者或安全性系統管理員的帳戶來存取內容套件。 |
| 您的租用戶不是 Premium 租用戶，或者沒有任何具備 Premium 授權檔案的使用者。 | 提出支援票證。 如需詳細資訊，請參閱[如何取得 Azure Active Directory 支援](../fundamentals/active-directory-troubleshooting-support-howto.md)。|
 

 

## <a name="importing-of-data-is-taking-too-long"></a>匯入資料的時間太長 
 
**此錯誤的呈現方式**：在 Power BI 中，一旦連線到內容套件之後，資料匯入程序就會開始準備您的儀表板以用於 Azure Active Directory 活動記錄。 您會看見下列訊息：「正在匯入資料...」  

| 原因 | 修正方式 |
| ---   | ---        |
| 根據您的租用戶大小而定，此步驟所需的時間可能從數分鐘到 30 分鐘。 | 請耐心等候。 如果訊息未在一小時內變更以顯示您的儀表板，請提出支援票證。 如需詳細資訊，請參閱[如何取得 Azure Active Directory 支援](../fundamentals/active-directory-troubleshooting-support-howto.md)。|

## <a name="next-steps"></a>後續步驟

若要安裝適用於 Azure Active Directory 預覽版的 Power BI 內容套件，請按一下[這裡](https://powerbi.microsoft.com/blog/azure-active-directory-meets-power-bi/) \(英文\)。


