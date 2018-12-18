---
title: 包含檔案
description: 包含檔案
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 06/10/2018
ms.author: raynew
ms.custom: include file
ms.openlocfilehash: ade53ba29d165b3b33ef25dabda25c4e60022608
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "40133383"
---
**組態/處理序伺服器需求**

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
IIS | - 沒有預先存在的預設網站 <br> - 沒有預先存在的網站/應用程式接聽連接埠 443 <br>- 啟用[匿名驗證](https://technet.microsoft.com/library/cc731244(v=ws.10).aspx) <br> - 啟用 [FastCGI](https://technet.microsoft.com/library/cc753077(v=ws.10).aspx) 設定 
| 
**網路設定** | 
IP 位址類型 | 靜態 
網際網路存取 | 伺服器需要存取這些 URL (直接或透過 Proxy)： <br> - \*.accesscontrol.windows.net<br> - \*.backup.windowsazure.com <br>- \*.store.core.windows.net<br> - \*.blob.core.windows.net<br> - \*.hypervrecoverymanager.windowsazure.com  <br> - https:\//management.azure.com <br> - *.services.visualstudio.com <br> - time.nist.gov <br> - time.windows.com <br> OVF 也需要存取下列 URL： <br> - https:\//login.microsoftonline.com <br> - https:\//secure.aadcdn.microsoftonline-p.com <br> - https:\//login.live.com  <br> - https:\//auth.gfx.ms <br> - https:\//graph.windows.net <br> - https:\//login.windows.net <br> - https:\//www.live.com <br> - https:\//www.microsoft.com <br> - https:\//dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi 
連接埠 | 443 (控制通道協調流程)<br>9443 (資料傳輸) 
NIC 類型 | VMXNET3 (如果組態伺服器是 VMware 虛擬機器)
 | 
**要安裝的軟體** | 
VMware vSphere PowerCLI | 如果組態伺服器在 VMware 虛擬機器上執行，則應該安裝 [PowerCLI 6.0 版](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1)。
MySQL | 應該安裝 MySQL。 您可以手動安裝，或者 Site Recovery 可以安裝。

**組態/處理序伺服器調整大小需求**

**CPU** | **記憶體** | **快取磁碟** | **資料變更率** | **複寫的機器**
--- | --- | --- | --- | ---
8 個 vCPU<br/><br/> 2 個插槽 * 4 個核心 \@ 2.5 GHz | 16 GB | 300 GB | 500 GB 或更少 | < 100 部機器
12 個 vCPU<br/><br/> 2 個插槽 * 6 個核心 \@ 2.5 GHz | 18 GB | 600 GB | 500 GB-1 TB | 100 到 150 部機器
16 個 vCPU<br/><br/> 2 個插槽 * 8 個核心 \@ 2.5 GHz | 32 GB | 1 TB | 1-2 TB | 150 到 200 部機器

