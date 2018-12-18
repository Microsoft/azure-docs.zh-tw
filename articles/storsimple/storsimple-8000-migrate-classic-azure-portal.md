---
title: 移轉您的 StorSimple 裝置管理員服務的儲存體帳戶、訂用帳戶 | Microsoft Docs
description: 了解如何移轉您的 StorSimple 裝置管理員服務 8000 的訂用帳戶、儲存體帳戶。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/08/2018
ms.author: alkohli
ms.openlocfilehash: 5a0da47b854e625f2f4a2fcf4c95ec566ba63093
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633974"
---
# <a name="migrate-subscriptions-and-storage-accounts-associated-with-storsimple-device-manager-service"></a>移轉與 StorSimple 裝置管理員服務相關聯的訂用帳戶和儲存體帳戶

您可能需要將 StorSimple 服務移至新的註冊或新的訂用帳戶。 這些移轉案例是帳戶變更或資料中心變更。 使用下表來了解支援哪些案例，包括移動的詳細步驟。

## <a name="account-changes"></a>帳戶變更

| 您可以移動...| 支援| 停機| Azure 支援流程| 方法|
|-----|-----|-----|-----|-----|
| 整個訂用帳戶 (包括 StorSimple 服務和儲存體帳戶) 到另一個註冊嗎？ | 是       | 否       | **註冊轉移**<br>使用：<li>當您購買新合約底下的新 Azure 承諾用量。</li><li>您想要將所有帳戶和訂用帳戶從舊的註冊移轉到新的註冊。 這包括舊訂用帳戶底下的所有 Azure 服務。</li> | **步驟 1：開啟 Azure 企業作業支援票證。**<li>移至 [http://aka.ms/AzureEntSupport](http://aka.ms/AzureEntSupport)。</li><li> 選取 [註冊管理]，然後選取 [從一個註冊轉移到新的註冊]。<br>**步驟 2：提供要求的資訊**<br>包含：<li>來源註冊號碼</li><li> 目的地註冊號碼</li><li>轉移生效日期|
| StorSimple 服務從現有的帳戶到新的註冊嗎？    | 是       | 否       | **帳戶轉移**<br>使用︰<li>當您不想要完整註冊轉移時。</li><li>您只想要將特定帳戶移至新的註冊。</li>| **步驟 1：開啟 Azure 企業作業支援票證。**<li>移至 [http://aka.ms/AzureEntSupport](http://aka.ms/AzureEntSupport)。</li><li>選取 [註冊管理]，然後選取 [將 EA 帳戶轉移到新的註冊]。<br>**步驟 2：提供要求的資訊**<br>包含：<li>來源註冊號碼</li><li> 目的地註冊號碼</li><li>轉移生效日期|
| StorSimple 服務從一個訂用帳戶到其他訂用帳戶嗎？      | 否        |    是         | 否，手動程序|<li>移轉 StorSimple 裝置的資料。</li><li>執行將裝置重設成出廠預設值，這會刪除裝置上的任何本機資料。</li><li>為裝置向 StorSimple 裝置管理員服務註冊新的訂用帳戶。</li><li>將資料移轉回裝置。|
  |可以將 Azure 訂用帳戶的擁有權轉移給另一個目錄嗎？ | 是       | 否       | 將現有的訂用帳戶關聯至您的 Azure AD 目錄 | 請參閱[將現有的訂用帳戶關聯至您的 Azure AD 目錄](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)。 最多可能需要 10 分鐘的時間才能正確顯示所有項目。|
| StorSimple 裝置從一個 StorSimple 裝置管理員服務到不同區域中的另一個服務嗎？      | 否        | 是            | 否，手動程序 |同上。|
| 新訂用帳戶或資源群組的儲存體帳戶？     | 是        | 否             |將儲存體帳戶移至不同的訂用帳戶或資源群組 |移動後，如果儲存體帳戶存取金鑰遭到更新，使用者必須透過 StorSimple Device Manager 服務，為移轉後的儲存體帳戶手動設定存取金鑰。|
| 傳統儲存體帳戶或 Azure Resource Manager 儲存體帳戶      | 是        | 否             |從傳統移轉至 Azure Resource Manager |<li>如需如何將儲存體帳戶從傳統移轉至 Azure Resource Manager 的詳細指示，請移至[移轉傳統儲存體帳戶](../virtual-machines/windows/migration-classic-resource-manager-ps.md#step-62-migrate-a-storage-account)。</li><li> 如果儲存體帳戶存取金鑰在移動後遭到更新，使用者需要透過 StorSimple Device Manager 服務，為移轉後的儲存體帳戶同步處理存取金鑰。 這是為了確保 StorSimple 裝置繼續正常運作，而且能夠將主要/備份資料層疊至 Azure。 如需同步處理存取金鑰的詳細指示，請移至[替換工作流程](storsimple-8000-manage-storage-accounts.md#key-rotation-of-storage-accounts)。</li><li> 若是 StorSimple 雲端設備，當移轉傳統儲存體帳戶，但基礎虛擬機器仍停留在傳統時，設備應該會正常運作。 如果移轉雲端設備的基礎虛擬機器，則停用及刪除功能將無法運作。</li><li> 您必須在 Azure 入口網站建立新的 StorSimple 雲端設備，然後從較舊的雲端設備容錯移轉。 您無法在新的 Azure 入口網站，使用傳統儲存體帳戶建立 StorSimple 雲端設備，它們必須有 Azure Resource Manager 儲存體帳戶。 如需詳細資訊，請移至[部署和管理 StorSimple 雲端設備](storsimple-8000-cloud-appliance-u2.md)。</li>|同上。|

## <a name="datacenter-changes"></a>資料中心變更

| 您可以移動...| 支援|停機| Azure 支援流程| 方法|
|-----|-----|-----|-----|-----|
| StorSimple 服務從一個 Azure 資料中心到另一個嗎？ | 否 | 是 |否，手動程序  |<li>移轉 StorSimple 裝置的資料。</li><li>執行將裝置重設成出廠預設值，這會刪除裝置上的任何本機資料。</li><li>為裝置向新的 StorSimple 裝置管理員服務註冊新的訂用帳戶。</li><li>將資料移轉回裝置。|
| 儲存體帳戶從一個 Azure 資料中心到另一個嗎？ | 否 |是  |否，手動程序  | 同上。|

## <a name="next-steps"></a>後續步驟

* [部署 StorSimple 裝置管理員服務](storsimple-8000-manage-service.md)
* [在 Azure 入口網站中部署 StorSimple 8000 系列裝置](storsimple-8000-deployment-walkthrough-u2.md)
