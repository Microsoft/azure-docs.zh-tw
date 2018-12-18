---
title: 重新執行 Azure AD Connect 安裝精靈 | Microsoft Docs
description: 說明安裝精靈在第二次執行時的運作方式。
keywords: Azure AD Connect 安裝精靈可讓您在第二次執行時設定維護設定
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: d800214e-e591-4297-b9b5-d0b1581cc36a
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: c5f5f59329e8f4e34a4c08a5819d0e4d96fe2f5c
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310637"
---
# <a name="azure-ad-connect-sync-running-the-installation-wizard-a-second-time"></a>Azure AD Connect 同步處理︰第二次執行安裝精靈
第一次執行 Azure AD Connect 安裝精靈時，它會逐步引導您設定安裝。 如果您再次執行安裝精靈，它則會提供維護選項。

您可以在 [開始] 功能表中找到名為 **Azure AD Connect**的安裝精靈。

![[開始] 功能表](./media/how-to-connect-installation-wizard/startmenu.png)

當您啟動安裝精靈時，您會看到具有下列選項的頁面︰

![含有其他工作清單的頁面](./media/how-to-connect-installation-wizard/additionaltasks.png)

如果您已使用 Azure AD Connect 安裝 ADFS，您會有更多選項。 [ADFS 管理](how-to-connect-fed-management.md#manage-ad-fs)中記載了您擁有的其他 ADFS 選項。

請選取其中一項工作，然後按 [下一步]  繼續。

> [!IMPORTANT]
> 當您開啟安裝精靈時，同步處理引擎中的所有作業都會暫停。 請確定您有在完成組態變更後，就馬上關閉安裝精靈。
>
>

## <a name="view-current-configuration"></a>檢視目前的組態
此選項可讓您快速檢視目前設定的選項。

![含有所有選項和其狀態之清單的頁面](./media/how-to-connect-installation-wizard/viewconfig.png)

按 [上一步]  返回。 如果您選取 [結束] ，您會關閉安裝精靈。

## <a name="customize-synchronization-options"></a>自訂同步處理選項
此選項可用來變更同步處理組態。 您會在自訂組態安裝路徑中看到選項子集。 即使您一開始是使用快速安裝也會看到此選項。

* [新增其他目錄](how-to-connect-install-custom.md#connect-your-directories)。 若要移除目錄，請參閱 [刪除連接器](how-to-connect-sync-service-manager-ui-connectors.md#delete)。
* [變更網域和 OU 篩選](how-to-connect-install-custom.md#domain-and-ou-filtering)。
* 移除群組篩選。
* [變更選用功能](how-to-connect-install-custom.md#optional-features)。

初始安裝中的其他選項既無法變更，也不能使用。 這些選項包括：

* 變更用於 userPrincipalName 和 sourceAnchor 的屬性。
* 變更不同樹系之物件的聯結方法。
* 啟用群組式篩選。

## <a name="refresh-directory-schema"></a>重新整理目錄結構描述
如果您已變更其中一個內部部署 AD DS 樹系中的結構描述，則會使用這個選項。 例如，您可能已安裝 Exchange，或升級至 Windows Server 2012 結構描述 (含裝置物件)。 在此情況下，您必須指示 Azure AD Connect，讓其從 AD DS 再次讀取結構描述並更新其快取。 此動作也會重新產生同步處理規則。 舉例來說，如果您新增 Exchange 結構描述，組態中就會新增 Exchange 的同步處理規則。

當您選取此選項時，便會列出組態中的所有目錄。 您可以保留預設設定，並重新整理所有樹系或取消選取其中某些樹系。

![含有環境中所有目錄清單的頁面](./media/how-to-connect-installation-wizard/refreshschema.png)

## <a name="configure-staging-mode"></a>設定預備模式
此選項可讓您啟用及停用伺服器上的預備模式。 預備模式和其使用方式的詳細資訊可在 [作業](how-to-connect-sync-operations.md#staging-mode)中找到。

此選項會顯示預備模式目前是已啟用還是已停用︰  
![也會顯示預備模式目前狀態的選項](./media/how-to-connect-installation-wizard/stagingmodecurrentstate.png)

若要變更狀態，請選取此選項，並選取或取消選取核取方塊。  
![也會顯示預備模式目前狀態的選項](./media/how-to-connect-installation-wizard/stagingmodeenable.png)

## <a name="change-user-sign-in"></a>變更使用者登入
此選項可讓您將使用者登入方法變更為密碼雜湊同步處理、傳遞驗證或同盟。 您無法變更為 [不設定] 。

如需此選項的詳細資訊，請參閱 [使用者登入](plan-connect-user-signin.md#changing-the-user-sign-in-method)。

## <a name="next-steps"></a>後續步驟
* 如需 Azure AD Connect 同步處理所用的組態模型的詳細資訊，請參閱 [了解宣告式佈建](concept-azure-ad-connect-sync-declarative-provisioning.md)。

**概觀主題**

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
