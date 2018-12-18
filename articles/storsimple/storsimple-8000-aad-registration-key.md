---
title: 在 Azure 中針對 StorSimple 8000 裝置管理員服務使用新驗證 | Microsoft Docs
description: 說明如何針對您的服務使用 AAD 型驗證、產生新的註冊金鑰，以及執行裝置的手動註冊。
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/23/2018
ms.author: alkohli
ms.openlocfilehash: b1ea195ab0b06c4ca0fab37fe7e5701229b34938
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387033"
---
# <a name="use-the-new-authentication-for-your-storsimple"></a>針對您的 StorSimple 使用新驗證

## <a name="overview"></a>概觀

StorSimple 裝置管理員服務在 Microsoft Azure 中執行，並連接至多個 StorSimple 裝置。 到目前為止，StorSimple 裝置管理員服務一直都是使用存取控制服務 (ACS) 來驗證針對您 StorSimple 裝置的服務。 ACS 機制即將被 Azure Active Directory (AAD) 驗證取代。 如需詳細資訊，請參閱下列有關 ACS 的淘汰及使用 AAD 驗證的公告。

- [Azure ACS 的未來為 Azure Active Directory](https://cloudblogs.microsoft.com/enterprisemobility/2015/02/12/the-future-of-azure-acs-is-azure-active-directory/) \(英文\)
- [即將對 Microsoft 存取控制服務進行的變更](https://azure.microsoft.com/blog/acs-access-control-service-namespace-creation-restriction/) \(英文\)

本文說明 AAD 驗證與相關新服務註冊金鑰的詳細資料，以及針對 StorSimple 裝置的防火牆規則所做出的修改。 本文所含資訊僅適用於 StorSimple 8000 系列裝置。

AAD 驗證會在執行 Update 5 或更新版本的 StorSimple 8000 系列裝置中發生。 由於 AAD 驗證的引進，下列項目將會發生變更：

- 防火牆規則的 URL 模式。
- 服務註冊金鑰。

下列各節將詳細討論這些變更。

## <a name="url-changes-for-aad-authentication"></a>AAD 驗證的 URL 變更

為了確保服務使用 AAD 型驗證，所有使用者都必須在其防火牆規則中包含新的驗證 URL。

如果使用的是 StorSimple 8000 系列，請務必在防火牆規則中包含下列 URL：

| URL 模式                         | 雲端 | 元件/功能         |
|------------------------------------|-------|----------------------------------|
| `https://login.windows.net`        | Azure 公用 |AAD 驗證服務      |
| `https://login.microsoftonline.us` | 美國政府 |AAD 驗證服務      |

如需適用於 StorSimple 8000 系列裝置之 URL 模式的完整清單，請移至[防火牆規則的 URL 模式](storsimple-8000-system-requirements.md#url-patterns-for-firewall-rules)。

如果在淘汰日期後仍未將驗證 URL 包含在防火牆規則內，且該裝置正在執行 Update 5，使用者將會看到 URL 警示。 使用者必須包含新的驗證 URL。 如果裝置正在執行 Update 5 之前的版本，則使用者會看到活動訊號警示。 在上述案例中，StorSimple 裝置都會無法向服務驗證，且服務將無法與裝置通訊。

## <a name="device-version-and-authentication-changes"></a>裝置版本和驗證變更

如果使用的是 StorSimple 8000 系列裝置，請使用下表以根據您正在執行的裝置軟體版本來判斷應採取的動作。

| 如果您的裝置正在執行| 請採取下列動作                                    |
|--------------------------|------------------------|--------------------|--------------------------------------------------------------|
| Update 5 或更新版本，且裝置已離線。 <br> 您會看到表示 URL 未列入允許清單的警示。|1.修改防火牆規則以包含驗證 URL。 請參閱[驗證 URL](#url-changes-for-aad-authentication)。<br>2.[從服務取得 AAD 註冊金鑰](#aad-based-registration-keys)。<br>3.[連線至 StorSimple 8000 系列裝置的 Windows PowerShell 介面](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console)。<br>4.使用 `Redo-DeviceRegistration` Cmdlet 以透過 Windows PowerShell 註冊裝置。 提供您在上一個步驟中取得的金鑰。|
| Update 5 或更新版本，且裝置已上線。| 不需採取任何動作。                                       |
| Update 4 或更早版本，且裝置已離線。 |1.修改防火牆規則以包含驗證 URL。<br>2.[透過目錄伺服器下載 Update 5](storsimple-8000-install-update-5.md#download-updates-for-your-device)。<br>3.[透過 Hotfix 方法套用 Update 5](storsimple-8000-install-update-5.md#install-update-5-as-a-hotfix)。<br>4.[從服務取得 AAD 註冊金鑰](#aad-based-registration-keys)。<br>5.[連線至 StorSimple 8000 系列裝置的 Windows PowerShell 介面](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console)。 <br>6.使用 `Redo-DeviceRegistration` Cmdlet 以透過 Windows PowerShell 註冊裝置。 提供您在上一個步驟中取得的金鑰。|
| Update 4 或更早版本，且裝置已上線。 |修改防火牆規則以包含驗證 URL。<br> 透過 Azure 入口網站安裝 Update 5。              |
| 恢復出廠預設值以重設至 Update 5 之前的版本。      |入口網站顯示 AAD 型註冊金鑰，而裝置執行的是舊版軟體。 當裝置執行的是 Update 4 或更早版本時，請遵循上述案例中的步驟。              |

## <a name="aad-based-registration-keys"></a>AAD 型註冊金鑰

針對 StorSimple 8000 系列裝置，從 Update 5 開始將會使用新的 AAD 型註冊金鑰。 您可以使用註冊金鑰來向裝置註冊您的 StorSimple 裝置管理員服務。

如果您使用的是執行 Update 4 或更早版本的 StorSimple 8000 系列裝置 (包含正在啟用的較舊裝置)，則您無法使用新的 AAD 服務註冊金鑰。
在此案例中，您需要重新產生服務註冊金鑰。 一旦您重新產生金鑰，將會使用新的金鑰來註冊所有後續的裝置。 舊的金鑰將不再有效。

- 新的 AAD 註冊金鑰會在 3 天後到期。
- AAD 註冊金鑰僅適用於執行 Update 5 或更新版本的 StorSimple 8000 系列裝置。
- AAD 註冊金鑰的長度超過對應的 ACS 註冊金鑰。

執行下列步驟以產生 AAD 服務註冊金鑰。

#### <a name="to-generate-the-aad-service-registration-key"></a>產生 AAD 服務註冊金鑰

1. 在 [StorSimple 裝置管理員] 中，移至 [管理] &gt; [金鑰]。 您也可以使用搜尋列搜尋_金鑰_。
    
2. 按一下 [產生金鑰]。

    ![按一下重新產生](./media/storsimple-8000-aad-registration-key/aad-click-generate-registration-key.png)

3. 複製新的金鑰。 舊的金鑰將無法再運作。

    ![確認重新產生](./media/storsimple-8000-aad-registration-key/aad-registration-key2.png)

    > [!NOTE] 
    > 如果您要在向 StorSimple 8000 系列裝置註冊的服務上建立 StorSimple 雲端設備，請勿在建立期間產生註冊金鑰。 請等候建立完成，然後再產生註冊金鑰。

## <a name="next-steps"></a>後續步驟

* 深入了解如何部署 [StorSimple 8000 系列裝置](storsimple-8000-deployment-walkthrough-u2.md)。

