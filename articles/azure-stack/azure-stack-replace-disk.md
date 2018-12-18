---
title: 更換 Azure Stack 中的實體磁碟 | Microsoft Docs
description: 概述如何更換 Azure Stack 中實體磁碟的程序。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 449ae53e-b951-401a-b2c9-17fee2f491f1
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/30/2018
ms.author: mabrigg
ms.openlocfilehash: f168c005c729ae75a5369b80b3dc5eab03ee0243
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30311316"
---
# <a name="replace-a-physical-disk-in-azure-stack"></a>更換 Azure Stack 中的實體磁碟

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

本文說明更換 Azure Stack 中實體磁碟的一般程序。 如果實體磁碟故障，您應該儘快更換它。

您可將此程序用於整合系統，以及用於具有熱插拔磁碟的開發套件部署。

實際的磁碟更換步驟會因原始設備製造商 (OEM) 硬體廠商而異。 如需您系統專屬的詳細步驟，請參閱廠商的現場可更換單元 (FRU) 文件。 

## <a name="review-disk-alert-information"></a>檢閱磁碟警示資訊
當磁碟故障時，您會收到通知您實體磁碟連線中斷的警示。 

 ![顯示實體磁碟連線中斷的警示](media/azure-stack-replace-disk/DiskAlert.png)

如果您開啟該警示，警示描述會包含您必須更換之磁碟的縮放單位節點和確切實體插槽位置。 Azure Stack 會使用 LED 指示器功能來進一步協助您識別故障的磁碟。

 ## <a name="replace-the-disk"></a>更換磁碟

請依照您 OEM 硬體廠商的 FRU 指示來實際更換磁碟。

> [!note]
> 請一次只更換一個縮放單位節點的磁碟。 等候虛擬磁碟修復作業完成之後，再移至下一個縮放單位節點

為了避免在整合系統中使用不支援的磁碟，系統會封鎖您廠商不支援的磁碟。 如果您嘗試使用不支援的磁碟，將會有新警示通知您因為不支援該磁碟的型號或韌體，所以已將該磁碟隔離。

在您更換磁碟之後，Azure Stack 會自動探索新磁碟，並啟動虛擬磁碟修復程序。  
 
 ## <a name="check-the-status-of-virtual-disk-repair"></a>檢查虛擬磁碟修復狀態
 
 在您更換磁碟之後，您可以使用具特殊權限的端點，來監視虛擬磁碟健康狀態和修復作業進度。 請從任何能夠透過網路連線到具特殊權限端點的電腦，依照下列步驟操作。

1. 開啟 Windows PowerShell 工作階段，然後連線到具特殊權限的端點。
    ````PowerShell
        $cred = Get-Credential
        Enter-PSSession -ComputerName <IP_address_of_ERCS>`
          -ConfigurationName PrivilegedEndpoint -Credential $cred
    ```` 
  
2. 執行下列命令來檢視虛擬磁碟健康情況：
    ````PowerShell
        Get-VirtualDisk -CimSession s-cluster
    ````
   ![Get-VirtualDisk 命令的 PowerShell 輸出](media/azure-stack-replace-disk/GetVirtualDiskOutput.png)

3. 執行下列命令來檢視目前的儲存體作業狀態：
    ```PowerShell
        Get-VirtualDisk -CimSession s-cluster | Get-StorageJob
    ````
      ![Get-StorageJob 命令的 PowerShell 輸出](media/azure-stack-replace-disk/GetStorageJobOutput.png)

## <a name="troubleshoot-virtual-disk-repair"></a>針對虛擬磁碟修復問題進行疑難排解

如果虛擬磁碟修復作業出現停滯，請執行下列命令來重新啟動作業：
  ````PowerShell
        Get-VirtualDisk -CimSession s-cluster | Repair-VirtualDisk
  ```` 
