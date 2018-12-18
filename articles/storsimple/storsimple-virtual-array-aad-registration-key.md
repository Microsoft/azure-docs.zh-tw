---
title: 適用於 StorSimple Virtual Array 的新驗證 |Microsoft 文件
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
ms.date: 06/28/2018
ms.author: alkohli
ms.openlocfilehash: e33a3f843017ec24f3a79701fac9a62e15b4f9ba
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37109183"
---
# <a name="use-the-new-authentication-for-your-storsimple"></a>針對您的 StorSimple 使用新驗證

## <a name="overview"></a>概觀

StorSimple Manager 裝置管理員服務在 Microsoft Azure 中執行，並連接至多個 StorSimple Virtual Array。 到目前為止，StorSimple 裝置管理員服務一直都是使用存取控制服務 (ACS) 來驗證針對您 StorSimple 裝置的服務。 ACS 機制即將被 Azure Active Directory (AAD) 驗證取代。

本文中所包含的資訊僅適用於 StorSimple 1200 系列虛擬陣列。 本文說明 AAD 驗證與相關新服務註冊金鑰的詳細資料，以及針對 StorSimple 裝置的防火牆規則所做出的修改。

AAD 驗證會在執行 Update 1 或更新版本的 StorSimple Virtual Array (1200 型) 中發生。

由於 AAD 驗證的引進，下列項目將會發生變更：

- 防火牆規則的 URL 模式。
- 服務註冊金鑰。

下列各節將詳細討論這些變更。

## <a name="url-changes-for-aad-authentication"></a>AAD 驗證的 URL 變更

為了確保服務使用 AAD 型驗證，所有使用者都必須在其防火牆規則中包含新的驗證 URL。

如果使用的是 StorSimple Virtual Array，請務必在防火牆規則中包含下列 URL：

| URL 模式                         | 雲端 | 元件/功能         |
|------------------------------------|-------|---------------------------------|
| `https://login.windows.net`        | Azure 公用 |AAD 驗證服務      |
| `https://login.microsoftonline.us` | 美國政府 |AAD 驗證服務      |

如需適用於 StorSimple Virtual Array 之 URL 模式的完整清單，請移至[防火牆規則的 URL 模式](storsimple-ova-system-requirements.md#url-patterns-for-firewall-rules)。

如果在淘汰日期後仍未將驗證 URL 包含在防火牆規則內，使用者將會看到重要警示，指出其 StorSimple 裝置無法向服務驗證。 服務將無法與裝置通訊。 如果使用者看到此警示，就必須包含新的驗證 URL。 如需有關警示的詳細資訊，請移至[使用警示來監視您的 StorSimple 裝置](storsimple-virtual-array-manage-alerts.md#networking-alerts)。

## <a name="device-version-and-authentication-changes"></a>裝置版本和驗證變更

如果使用的是 StorSimple Virtual Array，請使用下表以根據您正在執行的裝置軟體版本來判斷應採取的動作。

| 如果您的裝置正在執行  | 請採取下列動作                                    |
|----------------------------|--------------------------------------------------------------|
| Update 1.0 或更新版本，且裝置已離線。 <br> 您會看到表示 URL 未列入允許清單的警示。| 1.修改防火牆規則以包含驗證 URL。 請參閱[驗證 URL](#url-changes-for-aad-authentication)。 <br> 2.[從服務取得 AAD 註冊金鑰](#aad-based-registration-keys)。 <br> 3.執行步驟 1-5 以[連線到虛擬陣列的 Windows PowerShell 介面](storsimple-virtual-array-deploy2-provision-hyperv.md#step-2-provision-a-virtual-array-in-hypervisor)。<br> 4.使用 `Invoke-HcsReRegister` Cmdlet 以透過 Windows PowerShell 註冊裝置。 提供您在上一個步驟中取得的金鑰。|
| Update 1.0 或更新版本，且裝置已上線。| 不需採取任何動作。                                       |
| Update 0.6 或更早版本，且裝置已離線。 | 1.[透過目錄伺服器下載 Update 1.0](storsimple-virtual-array-install-update-1.md#download-the-update-or-the-hotfix)。<br>2.[透過本機 Web UI 套用 Update 1.0](storsimple-virtual-array-install-update-1.md#install-the-update-or-the-hotfix)。<br>3.[從服務取得 AAD 註冊金鑰](#aad-based-registration-keys)。 <br>4.執行步驟 1-5 以[連線到虛擬陣列的 Windows PowerShell 介面](storsimple-virtual-array-deploy2-provision-hyperv.md#step-2-provision-a-virtual-array-in-hypervisor)。<br>5.使用 `Invoke-HcsReRegister` Cmdlet 以透過 Windows PowerShell 註冊裝置。 提供您在上一個步驟中取得的金鑰。|
| Update 0.6 或更早版本，且裝置已上線 | 修改防火牆規則以包含驗證 URL。<br> 透過 Azure 入口網站來安裝 Update 1.0。 |

## <a name="aad-based-registration-keys"></a>AAD 型註冊金鑰

針對 StorSimple Virtual Array，從 Update 1.0 開始將會使用新的 AAD 型註冊金鑰。 您可以使用註冊金鑰來向裝置註冊您的 StorSimple 裝置管理員服務。

如果您是使用執行 Update 0.6 或更早版本的 StorSimple Virtual Array，則您無法使用新的 AAD 服務註冊金鑰。 您必須重新產生服務註冊金鑰。 一旦您重新產生金鑰，將會使用新的金鑰來註冊所有後續的裝置。 舊的金鑰將不再有效。

- 新的 AAD 註冊金鑰會在 3 天後到期。
- AAD 註冊金鑰僅適用於執行 Update 1 或更新版本的 StorSimple 1200 系列虛擬陣列。 來自 StorSimple 8000 系列裝置的 AAD 註冊金鑰將無法運作。
- AAD 註冊金鑰的長度超過對應的 ACS 註冊金鑰。

執行下列步驟以產生 AAD 服務註冊金鑰。

#### <a name="to-generate-the-aad-service-registration-key"></a>產生 AAD 服務註冊金鑰

1. 在 [StorSimple 裝置管理員] 中，移至 [管理] &gt; [金鑰]。
    
    ![移至 [金鑰]](./media/storsimple-virtual-array-aad-registration-key/aad-registration-key1.png)

2. 按一下 [產生金鑰]。

    ![按一下重新產生](./media/storsimple-virtual-array-aad-registration-key/aad-click-generate-registration-key.png)

3. 複製新的金鑰。 舊的金鑰無法再運作。

    ![確認重新產生](./media/storsimple-virtual-array-aad-registration-key/aad-registration-key2.png)

## <a name="next-steps"></a>後續步驟

* 深入了解如何部署 [StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md)
