---
title: 包含檔案
description: 包含檔案
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 09/03/2018
ms.author: raynew
ms.custom: include file
ms.openlocfilehash: c32d1bbe5c2f735333a312638c553d7a2c434c0b
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702658"
---
**實體伺服器複寫的組態/處理序伺服器需求**

**元件** | **需求** 
--- | ---
**硬體設定** | 
CPU 核心 | 8 
RAM | 16 GB
磁碟數量 | 3，包括作業系統磁碟、處理序伺服器快取磁碟和用於容錯回復的保留磁碟機 
可用磁碟空間 (處理序伺服器快取) | 600 GB
可用磁碟空間 (保留磁碟) | 600 GB
 | 
**軟體設定** | 
作業系統 | Windows Server 2012 R2 <br> Windows Server 2016
作業系統地區設定 | 英文 (en-us)
Windows Server 角色 | 請勿啟用這些角色： <br> - Active Directory Domain Services <br>- 網際網路資訊服務 <br> - Hyper-V 
群組原則 | 請勿啟用這些群組原則： <br> - 防止存取命令提示字元。 <br> - 防止存取登錄編輯工具。 <br> - 檔案附件的信任邏輯。 <br> - 開啟指令碼執行。 <br> [深入了解](https://technet.microsoft.com/library/gg176671(v=ws.10).aspx)
IIS | - 沒有預先存在的預設網站 <br> - 沒有預先存在的網站/應用程式接聽連接埠 443 <br>- 啟用[匿名驗證](https://technet.microsoft.com/library/cc731244(v=ws.10).aspx) <br> - 啟用 [FastCGI](https://technet.microsoft.com/library/cc753077(v=ws.10).aspx) 設定。
IP 位址類型 | 靜態 
| 
**存取設定** | 
MySQL | 應該在組態伺服器上安裝 MySQL。 您可以手動安裝，或由 Site Recovery 在部署期間安裝。 若要讓 Site Recovery 安裝，請確認機器可以連線到 http://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.37-win32.msi。
URL | 組態伺服器需要存取這些 URL (直接或透過 Proxy)：<br/><br/> Azure AD：``login.microsoftonline.com``；``login.microsoftonline.us``；``*.accesscontrol.windows.net``<br/><br/> 複寫資料傳輸：``*.backup.windowsazure.com``；``*.backup.windowsazure.us``<br/><br/> 複寫管理：``*.hypervrecoverymanager.windowsazure.com``；``*.hypervrecoverymanager.windowsazure.us``；``https://management.azure.com``；``*.services.visualstudio.com``<br/><br/> 儲存體存取：``*.blob.core.windows.net``；``*.blob.core.usgovcloudapi.net``<br/><br/> 時間同步處理：``time.nist.gov``；``time.windows.com<br/><br/> Telemetry (optional): ``dc.services.visualstudio.com``
防火牆 | 以 IP 位址為基礎的防火牆規則都應該允許對 Azure URL 的通訊。 為簡化並限制 IP 範圍，建議您使用 URL 篩選。<br/><br/>**針對商業 IP：**<br/><br/>- 允許 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/confirmation.aspx?id=41653)和 HTTPS (443) 連接埠。<br/><br/> 允許美國西部使用 IP 位址範圍 (用於存取控制和身分識別管理)。<br/><br/> - 允許訂用帳戶 Azure 區域的 IP 位址範圍，以支援 Azure Active Directory、備份、複寫和儲存體所需的 URL。<br/><br/> **針對政府 IP：**<br/><br/> - 允許 Azure Government 資料中心 IP 範圍和 HTTPS (443) 連接埠。<br/><br/> - 允許所有 US Gov 區域 (維吉尼亞州、德州、亞歷桑那州和愛荷華州) 的 IP 位址範圍，以支援 Azure Active Directory、備份、複寫和儲存體所需的 URL。
連接埠 | 允許 443 (控制通道協調流程)<br/><br/> 允許 9443 (資料傳輸) 


**組態/處理序伺服器調整大小需求**

**CPU** | **記憶體** | **快取磁碟** | **資料變更率** | **複寫的機器**
--- | --- | --- | --- | ---
8 個 vCPU<br/><br/> 2 個插槽 * 4 個核心 \@ 2.5 GHz | 16 GB | 300 GB | 500 GB 或更少 | < 100 部機器
12 個 vCPU<br/><br/> 2 個插槽 * 6 個核心 \@ 2.5 GHz | 18 GB | 600 GB | 500 GB-1 TB | 100 到 150 部機器
16 個 vCPU<br/><br/> 2 個插槽 * 8 個核心 \@ 2.5 GHz | 32 GB | 1 TB | 1-2 TB | 150 到 200 部機器

