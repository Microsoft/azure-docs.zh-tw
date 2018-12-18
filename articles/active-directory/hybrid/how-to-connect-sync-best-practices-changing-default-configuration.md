---
title: Azure AD Connect 同步：變更預設組態 | Microsoft Docs
description: 提供變更 Azure AD Connect 同步處理預設組態的最佳作法。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 7638a031-1635-4942-94c3-fce8f09eed5e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/29/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: ad19916dc49c2bee870eef77611d120cc66b44cd
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306237"
---
# <a name="azure-ad-connect-sync-best-practices-for-changing-the-default-configuration"></a>Azure AD Connect 同步處理：變更預設組態的最佳作法
本主題的目的旨在說明支援及不支援的 Azure AD Connect 同步處理變更。

Azure AD Connect 所建立的組態適用於大部分同步內部部署 Active Directory 與 Azure AD 的「現狀」環境。 不過，在某些情況下，組態必須套用某些變更以滿足特定需要或需求。

## <a name="changes-to-the-service-account"></a>服務帳戶的變更
Azure AD Connect 同步處理會使用安裝精靈所建立的服務帳戶執行。 這個服務帳戶會存放同步處理所使用的資料庫加密金鑰。它是使用 127 個字元長的密碼所建立的，而且密碼已設定為永不到期。

* 它 **不支援** 變更或重設服務帳戶的密碼。 這麼做會損毀加密金鑰，而服務無法存取資料庫且無法啟動。

## <a name="changes-to-the-scheduler"></a>排程器的變更
從組建 1.1 (2016 年 2 月) 的版本開始，您可以將 [排程器](how-to-connect-sync-feature-scheduler.md) 的同步處理週期設定為預設值 30 分鐘以外的值。

## <a name="changes-to-synchronization-rules"></a>同步處理規則的變更
安裝精靈所提供的組態應該適用於最常見的案例。 萬一您需要對組態進行變更，則您必須遵循這些規則，以便仍能具備支援的組態。

> [!WARNING]
> 若您變更預設的同步處理規則，則系統在下次更新 Azure AD Connect 時會覆寫這些變更，導致非預期且可能不想要的同步處理結果。

* 如果預設的直接屬性流程不適合您的組織使用，您可以 [變更屬性流程](how-to-connect-sync-change-the-configuration.md#other-common-attribute-flow-changes) 。
* 如果您想要 [不傳送屬性](how-to-connect-sync-change-the-configuration.md#do-not-flow-an-attribute) 並移除 Azure AD 中任何現有的屬性值，則必須為此案例建立規則。
* [停用不必要的同步處理規則](#disable-an-unwanted-sync-rule) 而非加以刪除。 升級期間會重新建立已刪除的規則。
* 若要 [變更現成可用的規則](#change-an-out-of-box-rule)，您應該複製原始規則並停用現成可用的規則。 同步處理規則編輯器會提示並協助您。
* 使用同步處理規則編輯器，匯出您的自訂同步處理規則。 此編輯器會提供可用來在災害復原情況下輕鬆重建它們的 PowerShell 指令碼。

> [!WARNING]
> 現成可用的同步處理規則具有憑證指紋。 如果您變更這些規則，將不再符合憑證指紋。 未來當您嘗試套用新版的 Azure AD Connect 時可能會遇到問題。 僅利用本文所述的方式進行變更。

### <a name="disable-an-unwanted-sync-rule"></a>停用不必要的同步處理規則
請勿刪除現成可用的同步處理規則。 升級期間會重新建立此規則。

在某些情況下，安裝精靈所產生的組態不適用於您的拓撲。 例如，如果您具備帳戶與資源樹系拓撲，但已在具備 Exchange 結構描述的帳戶樹系中擴充該結構描述，則系統會針對帳戶樹系和資源樹系建立適用於 Exchange 的規則。 在此情況下，您需要停用適用於 Exchange 的同步處理規則。

![已停用同步處理規則](./media/how-to-connect-sync-best-practices-changing-default-configuration/exchangedisabledrule.png)

在上圖中，安裝精靈已在帳戶樹系中找到舊的 Exchange 2003 結構描述。 此結構描述擴充是在 Fabrikam 的環境中引進資源樹系之前新增的。 若要確保不會同步處理任何來自舊的 Exchange 實作的屬性，就必須以所述的方式停用同步處理規則。

### <a name="change-an-out-of-box-rule"></a>變更現成可用的規則
只有當您需要變更聯結規則時，才應該變更內建規則。 如果您需要變更屬性流程，則您應該建立其優先順序高於內建規則的同步處理規則。 您實際上唯一需要複製的規則是 **In from AD - User Join** 規則。 您可以使用具有較高優先順序的規則來覆寫所有其他規則。

如果您需要對現成可用的規則進行變更，則您應該複製該現成可用的規則，然後停用原始的規則。 接著對複製的規則進行變更。 同步處理規則編輯器會協助您完成這些步驟。 當您開啟現成可用的規則時，即會顯示此對話方塊：  
![對現成可用的規則發出警告](./media/how-to-connect-sync-best-practices-changing-default-configuration/warningoutofboxrule.png)

選取 [是]  來建立規則的複本。 隨即會開啟複製的規則。  
![複製的規則](./media/how-to-connect-sync-best-practices-changing-default-configuration/clonedrule.png)

在這個複製的規則上，對範圍、聯結和轉換進行任何必要變更。

## <a name="next-steps"></a>後續步驟
**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
