---
title: 在 Azure Active Directory 中啟用企業狀態漫遊 | Microsoft Docs
description: Windows 裝置中企業狀態漫遊設定的常見問題集。 企業狀態漫遊提供使用者跨 Windows 裝置的一致體驗，並且減少設定新的裝置所需的時間。
services: active-directory
keywords: 企業狀態漫遊, windows 雲端, 如何啟用企業狀態漫遊
documentationcenter: ''
author: tanning
manager: mtillman
editor: curtand
ms.component: devices
ms.assetid: f71d66fd-7f9e-45eb-9cfe-5d989870f8a4
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.author: markvi
ms.openlocfilehash: 71d8bfa116fd4d0e824d80da26df7271121a595c
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49311366"
---
# <a name="enable-enterprise-state-roaming-in-azure-active-directory"></a>在 Azure Active Directory 中啟用企業狀態漫遊
任何具有 Azure AD Premium 或 Enterprise Mobility + Security (EMS) 授權的組織都可以使用企業狀態漫遊。 如需有關如何取得 Azure AD 訂用帳戶的詳細資訊，請參閱 [Azure AD 產品頁面](https://azure.microsoft.com/services/active-directory)。

當您啟用企業狀態漫遊時，您的組織會自動從 Azure 資訊保護獲得 Azure Rights Management 保護的免費但有使用限制的授權。 此免費訂用帳戶只能加密和解密由企業狀態漫遊所同步的企業設定和應用程式資料。 您必須擁有[付費訂用帳戶](https://azure.microsoft.com/pricing/details/information-protection/)，才能使用 Azure Rights Management 服務的完整功能。

## <a name="to-enable-enterprise-state-roaming"></a>啟用企業狀態漫遊

1. 登入 [Azure AD 系統管理中心](https://aad.portal.azure.com/)。

1. 選取 [Azure Active Directory] &gt; [裝置] &gt; [企業狀態漫遊]。

1. 選取 [使用者可以在裝置間同步設定及應用程式資料]。 如需詳細資訊，請參閱[如何進行裝置設定](https://docs.microsoft.com/azure/active-directory/device-management-azure-portal)。
  
  ![標示為 [使用者可以在裝置間同步設定及應用程式資料] 的裝置設定影像](./media/active-directory-windows-enterprise-state-roaming-enable/device-settings.png)
  
若要讓 Windows 10 裝置使用企業狀態漫遊服務，裝置必須使用 Azure AD 身分識別進行驗證。 對於已加入 Azure AD 的裝置，使用者的主要登入身分識別就是其 Azure AD 身分識別，不需要額外設定。 對於使用內部部署 Active Directory 的裝置，IT 管理員必須[設定已加入混合式 Azure Active Directory 的裝置](https://docs.microsoft.com/azure/active-directory/devices/hybrid-azuread-join-manual-steps)。 

## <a name="data-storage"></a>資料儲存體
企業狀態漫遊資料裝載於一或多個 [Azure 區域](https://azure.microsoft.com/regions/)，這些區域最符合 Azure Active Directory 執行個體中設定的國家/區域值。 企業狀態漫遊的資料是根據三個主要地理區域來分割︰北美洲、EMEA 和 APAC。 適用於租用戶的企業狀態漫遊資料是位於本機的地理區域中，並不會跨區域複寫。  例如︰

國家/區域值 | 將其資料裝載於
---------------------|-------------------------
EMEA 國家/地區，例如，法國或尚比亞 | 歐洲內的一個或多個 Azure 區域 
北美洲國家/地區，例如美國或加拿大 | 美國內的一個或多個 Azure 區域
亞太地區國家/地區，例如澳洲或紐西蘭 | 亞洲內的一個或多個 Azure 區域
南美洲和南極洲區域 | 美國內的一個或多個 Azure 區域

國家/地區值是在 Azure AD 目錄建立程序期間所建立，之後無法修改。 如果您需要資料儲存體位置的詳細資料，請向 [Azure 支援](https://azure.microsoft.com/support/options/)提出票證。

## <a name="view-per-user-device-sync-status"></a>檢視每個使用者裝置同步處理狀態
請遵循下列步驟來檢視每個使用者裝置同步處理狀態報告。

1. 登入 [Azure AD 系統管理中心](https://aad.portal.azure.com/)。

1. 選取 [Azure Active Directory] &gt; [使用者] &gt; [所有使用者]。

1. 選取使用者，然後選取 [裝置]。

1. 在 [顯示] 底下，選取 [裝置同步設定和應用程式資料]，以顯示同步處理狀態。
  
  ![裝置同步處理資料設定的影像](./media/active-directory-windows-enterprise-state-roaming-enable/sync-status.png)
  
1. 如果此使用者有裝置在同步處理，您會看到裝置，如下所示。
  
  ![裝置同步處理單欄式資料的影像](./media/active-directory-windows-enterprise-state-roaming-enable/device-status-row.png)

## <a name="data-retention"></a>資料保留
使用企業狀態漫遊而同步處理至 Microsoft 雲端的資料會保留下來，直到手動刪除或確定該資料已過時為止。 

### <a name="explicit-deletion"></a>明確刪除
明確刪除是指 Azure 管理員刪除使用者或目錄，或是明確要求刪除資料。

* **使用者刪除**：在 Azure AD 中刪除使用者時，使用者帳戶漫遊資料會在 90 至 180 天後刪除。 
* **目錄刪除**：在 Azure AD 中刪除整個目錄是即時作業。 與該目錄相關聯的所有設定資料會在 90 至 180 天後刪除。 
* **依要求刪除**：如果 Azure AD 管理員想要手動刪除特定使用者的資料或設定資料，管理員可以向 [Azure 支援](https://azure.microsoft.com/support/)提出票證。 

### <a name="stale-data-deletion"></a>刪除過時資料
一年 (「保留期限」) 未存取的資料將視為過時，可能會從 Microsoft 雲端中刪除。 保留期限可能有所變更，但不會小於 90 天。 過時的資料可能是一組特定的 Windows/應用程式設定或使用者的所有設定。 例如︰

* 如果沒有任何裝置存取特定的設定集合 (例如，從裝置中移除應用程式，或針對使用者的所有裝置停用設定群組，例如「佈景主題」)，則該集合在保留期限之後就會變成過時，可能會被刪除。 
* 如果使用者在其所有裝置上已關閉設定同步處理，則不會存取任何設定資料，而且該使用者的所有設定資料將會變成過時，而且可能在保留期限之後刪除。 
* 如果 Azure AD 目錄管理員針對整個目錄關閉企業狀態漫遊，則該目錄中的所有使用者會停止同步處理設定，且所有使用者的所有設定資料會變成過時，並且可能在保留期限之後刪除。 

### <a name="deleted-data-recovery"></a>復原已刪除的資料
無法設定資料保留期原則。 一旦永久刪除資料，就無法復原。 但系統只會從 Microsoft 雲端刪除設定資料，而不會從使用者裝置中刪除。 如果任何裝置後來又重新連線至企業狀態漫遊服務，設定就會再次同步處理並儲存在 Microsoft 雲端中。

## <a name="related-topics"></a>相關主題
* [企業狀態漫遊概觀](active-directory-windows-enterprise-state-roaming-overview.md)
* [設定和資料漫遊常見問題集](active-directory-windows-enterprise-state-roaming-faqs.md)
* [設定同步處理的群組原則和 MDM 設定](active-directory-windows-enterprise-state-roaming-group-policy-settings.md)
* [Windows 10 漫遊設定參考](active-directory-windows-enterprise-state-roaming-windows-settings-reference.md)
* [疑難排解](active-directory-windows-enterprise-state-roaming-troubleshooting.md)
