---
title: 針對 Windows 無法完成系統設定進行疑難排解
titlesuffix: Azure Virtual Machines
description: 本文提供的步驟可解決 Sysprep 程式無法啟動 Azure VM 的問題。
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 5fc57a8f-5a0c-4b5f-beef-75eecb4d310d
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/09/2020
ms.author: v-miegge
ms.openlocfilehash: 6cb3467fec99bd12810ed058a61de1be7b39cdd0
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98629584"
---
# <a name="troubleshoot-windows-could-not-finish-configuring-the-system"></a>針對 Windows 無法完成系統設定進行疑難排解

本文提供的步驟可解決 Sysprep 程式無法將 Azure 虛擬機器 (VM) 啟動的問題。

## <a name="symptom"></a>徵狀

當您使用 [開機診斷](./boot-diagnostics.md) 來查看 VM 的螢幕擷取畫面時，您會在 Windows 安裝程式啟動服務時看到螢幕擷取畫面顯示安裝 windows 錯誤。 錯誤會顯示下列訊息：

`Windows could not finish configuring the system. To attempt to resume configuration, restart the computer. Setup is starting services`

  ![[圖 1] 顯示的是安裝 Windows 錯誤訊息：「Windows 無法完成設定系統。 若要嘗試繼續設定，請重新開機電腦。 安裝程式正在啟動服務」](./media/windows-could-not-configure-system/1-windows-error-configure.png)

## <a name="cause"></a>原因

當作業系統 (OS) 無法完成 [Sysprep](/windows-hardware/manufacture/desktop/sysprep-process-overview)程式時，就會導致此錯誤。 當您嘗試進行一般化 VM 的初始開機時，將會發生此錯誤。 如果您遇到此問題，請重新建立一般化映射，因為映射處於無法部署的狀態，且無法復原。

## <a name="solution"></a>解決方案

> [!TIP]
> 如果您有最新的 VM 備份，您可以嘗試 [從備份還原 vm](../../backup/backup-azure-arm-restore-vms.md) 以修正開機問題。

若要修正此問題，請遵循 [Azure 有關準備/捕獲映射的指引](../windows/upload-generalized-managed.md) ，並準備新的一般化映射。