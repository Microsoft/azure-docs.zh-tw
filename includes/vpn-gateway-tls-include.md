---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 07/27/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 1bbb56f819bccaa032f25e43b197e31a6a0730b7
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "43426418"
---
1. 以滑鼠右鍵按一下 [命令提示字元]，然後選取 [以系統管理員身分執行]，以使用較高的權限開啟命令提示字元。
2. 在命令提示字元中執行下列命令：

  ```
  reg add HKLM\SYSTEM\CurrentControlSet\Services\RasMan\PPP\EAP\13 /v TlsVersion /t REG_DWORD /d 0xfc0
  reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp" /v DefaultSecureProtocols /t REG_DWORD /d 0xaa0
  if %PROCESSOR_ARCHITECTURE% EQU AMD64 reg add "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp" /v DefaultSecureProtocols /t REG_DWORD /d 0xaa0
  ```

3. 安裝下列更新：
  
  * [KB3140245](https://www.catalog.update.microsoft.com/search.aspx?q=kb3140245)
  * [KB2977292](https://www.microsoft.com/en-us/download/details.aspx?id=44342)

4. 重新啟動電腦。
5. 連線至 VPN。
