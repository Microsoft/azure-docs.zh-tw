---
title: 包含檔案
description: 包含檔案
services: active-directory
author: curtand
ms.service: active-directory
ms.topic: include
ms.date: 07/30/2018
ms.author: curtand
ms.custom: include file
ms.openlocfilehash: a2055c3f00306fbfdad3028a16bb49d919e1e251
ms.sourcegitcommit: 99a6a439886568c7ff65b9f73245d96a80a26d68
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39361386"
---
以下是 Azure Active Directory (Azure AD) 服務的使用條件約束和其他服務限制。

| 類別 | 限制 |
| --- | --- |
| 目錄 | 單一使用者能以成員或來賓的身分，隸屬於最多 500 個 Azure AD 目錄。<br/>單一使用者最多可以建立 20 個目錄。 |
| 網域 | 您可以新增 900 個以內的受控網域名稱。 如果您要設定所有網域與內部部署 Active Directory 建立同盟，則可以在每個目錄中新增 450 個以內的網域名稱。 |
| 物件 |<ul><li>在免費版的 Azure Active Directory 中，最多可以在單一目錄中建立 500,000 個物件。</li><li>非系統管理員的使用者最多可以建立 250 個物件。</li></ul> |
| 結構描述延伸模組 |<ul><li>字串類型延伸模組最多可有 256 個字元。 </li><li>二進位類型延伸模組受限於 256 個位元組。</li><li>任何單一物件均可寫入 100 個延伸模組值 (所有類型和所有應用程式皆可)。</li><li>只能使用「字串」類型或「二進位」類型單一值屬性來擴充 “User”、“Group”、“TenantDetail”、“Device”、“Application” 和 “ServicePrincipal” 實體。</li><li>結構描述延伸模組僅適用於圖形 API 版本 1.21 預覽。 應用程式必須取得寫入權限才能登錄延伸模組。</li></ul> |
| [應用程式] |最多 100 個使用者可以成為單一應用程式的擁有者。 |
| 群組 |<ul><li>最多 100 個使用者可以成為單一群組的擁有者。</li><li>任何數目的物件都可以是 Azure Active Directory 中的單一群組的成員。</li><li>可以使用 Azure AD Connect 從您的內部部署 Active Directory 同步至 Azure Active Directory 群組中的成員數目限制為 5 萬個成員。</li></ul> |
| 存取面板 |<ul><li>對於 Azure AD Premium 或 Enterprise Mobility Suite 使用者已指派授權，每位使用者在存取面板中可以看到的應用程式數目沒有限制。</li><li>對於 Azure Active Directory 免費版或 Azure AD Basic 版的使用者已指派授權，每位使用者在存取面板中最多可以看到 10 個應用程式磚 (例如：Box、Salesforce 或 Dropbox)。 此限制不適用於系統管理員帳戶。</li></ul> |
| 報告 | 在任何報告中，最多可以檢視或下載 1000 個資料列。 任何其他資料會遭到截斷。 |
| 管理單位 | 物件可以是有不超過 30 個管理單位的成員。 |
