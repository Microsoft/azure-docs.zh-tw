---
title: 包含檔案
description: 包含檔案
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 09/12/2018
ms.author: raynew
ms.openlocfilehash: ae8eebf9667f2bc03fdc1082fb38c19c5c645c10
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47060999"
---
**名稱** | **商業 URL**  | **政府 URL** | **說明** |
--- | --- | --- | ---
Azure AD | ``login.microsoftonline.com`` | ``login.microsoftonline.us`` | 用於使用 AAD 的存取控制和身分識別管理 
Backup  | ``*.backup.windowsazure.com`` | ``*.backup.windowsazure.us`` | 用於複寫資料轉送和協調 
複寫 | ``*.hypervrecoverymanager.windowsazure.com`` | ``*.hypervrecoverymanager.windowsazure.us``  | 用於複寫管理作業和協調 
儲存體 | ``*.blob.core.windows.net`` | ``*.blob.core.usgovcloudapi.net``  | 用於存取儲存體帳戶來儲存複寫的資料 
遙測 (選用) | ``dc.services.visualstudio.com`` | ``dc.services.visualstudio.com`` | 用於遙測 
時間同步處理 | ``time.windows.com`` | ``time.nist.gov`` | 用於檢查所有部署中系統時間與通用時間之間的時間同步處理。


