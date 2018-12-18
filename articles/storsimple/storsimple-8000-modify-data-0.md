---
title: 修改 StorSimple 8000 系列裝置上的 DATA 0 設定 | Microsoft Docs
description: 了解如何使用 Windows PowerShell for StorSimple 重新設定 StorSimple 裝置上的 DATA 0 網路介面。
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
ms.date: 03/27/2017
ms.author: alkohli
ms.openlocfilehash: 90df43e22f17fd32fe642514df098b72700e77af
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108296"
---
# <a name="modify-the-data-0-network-interface-settings-on-your-storsimple-8000-series-device"></a>修改 StorSimple 8000 系列裝置上的 DATA 0 網路介面設定

## <a name="overview"></a>概觀

Microsoft Azure StorSimple 裝置有 6 個網路介面，從 DATA 0 至 DATA 5。 DATA 0 介面一律透過 Windows PowerShell 介面或序列主控台設定，且已自動啟用雲端功能。 請注意，您無法透過 Azure 入口網站設定 DATA 0 網路介面。

DATA 0 介面會在初始部署 StorSimple 裝置期間，透過安裝精靈進行第一次設定。 當裝置處於操作模式時，您可能需要重新進行 DATA 0 設定。 本教學課程提供兩種修改 DATA 0 網路設定的方法，皆需透過 Windows PowerShell for StorSimple 進行。

閱讀本教學課程之後，您將能夠：

* 透過安裝精靈修改 DATA 0 網路設定
* 透過 `Set-HcsNetInterface` Cmdlet 修改 DATA 0 網路設定

## <a name="modify-data-0-network-settings-through-setup-wizard"></a>透過安裝精靈修改 DATA 0 網路設定
您可以連接至 StorSimple 裝置的 Windows PowerShell 介面並啟動安裝精靈工作階段以重新設定 DATA 0 網路設定。 執行下列步驟以修改 DATA 0 設定：

#### <a name="to-modify-data-0-network-settings-through-setup-wizard"></a>透過安裝精靈修改 DATA 0 網路設定
1. 在序列主控台功能表中，選擇選項 1 [使用完整存取權登入] 。 出現提示時，提供**裝置系統管理員密碼**。 預設密碼為 `Password1`。
2. 在命令提示字元中，輸入：
   
    `Invoke-HcsSetupWizard`
3. 安裝精靈隨即出現，以協助您設定裝置的 DATA 0 介面。 提供 IP 位址、閘道器和網路遮罩的新值。

> [!NOTE]
> 固定控制器 IP 必須在 Azure 入口網站中透過 StorSimple 裝置的 [網路設定] 刀鋒視窗重新設定。 如需詳細資訊，請移至 [修改網路介面](storsimple-8000-modify-device-config.md#modify-network-interfaces)。

## <a name="modify-data-0-network-settings-through-set-hcsnetinterface-cmdlet"></a>透過 Set-HcsNetInterface cmdlet 修改 DATA 0 網路設定
重新設定 DATA 0 網路介面的替代方式為透過使用 `Set-HcsNetInterface` cmdlet。 cmdlet 是從 StorSimple 裝置的 Windows PowerShell 介面執行。 使用此程序時，控制器固定 IP 也可以在此設定。 執行下列步驟以修改 DATA 0 設定： 

#### <a name="to-modify-data-0-network-settings-through-the-set-hcsnetinterface-cmdlet"></a>透過 Set-HcsNetInterface cmdlet 修改 DATA 0 網路設定
1. 在序列主控台功能表中，選擇選項 1 [使用完整存取權登入] 。 出現提示時，提供裝置系統管理員密碼。 預設密碼為 `Password1`。
2. 在命令提示字元中，輸入：
   
    `Set-HCSNetInterface -InterfaceAlias Data0 -IPv4Address <> -IPv4Netmask <> -IPv4Gateway <> -Controller0IPv4Address <> -Controller1IPv4Address <> -IsiScsiEnabled 1 -IsCloudEnabled 1`
   
    在角括弧內鍵入下列 DATA 0 的值：
   
   * IPv4 位址
   * IPv4 閘道
   * IPv4 子網路遮罩
   * 控制器 0 的固定的 IPv4 位址
   * 控制器 1 的固定的 IPv4 位址
     
     如需使用此 Cmdlet 的詳細資訊，請移至 [Windows PowerShell for StorSimple Cmdlet 參考](https://technet.microsoft.com/library/dn688161.aspx)。

## <a name="next-steps"></a>後續步驟
* 若要設定 DATA 0 以外的網路介面，您可以使用[在 Azure 入口網站中設定網路設定](storsimple-8000-modify-device-config.md)。 
* 如果您在設定您的網路介面時遇到任何問題，請參閱 [疑難排解部署問題](storsimple-troubleshoot-deployment.md)。

