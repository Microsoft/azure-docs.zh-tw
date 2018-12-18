---
title: 包含檔案
description: 包含檔案
services: backup
author: markgalioto
ms.service: backup
ms.topic: include
ms.date: 9/10/2018
ms.author: trinadhk;sogup
ms.custom: include file
ms.openlocfilehash: 64101ea5a3bbaac4a6b2e349a04d06ea84a87081
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44381071"
---
下列限制適用於 Azure 備份。

| 限制識別碼 | 預設限制 |
| --- | --- |
| 可針對每個保存庫註冊的伺服器/電腦數目 |如為 Windows Server/用戶端/SCDPM 則有 50 個， <br/> 如為 IaaS VM 則有 1000 個 |
| 在 Azure 保存庫儲存體中所儲存資料的資料來源大小 |最大 54400 GB<sup>1</sup> |
| 可以在每個 Azure 訂用帳戶中建立的備份保存庫的數目 |每個區域 500 個復原服務保存庫 |
| 每日可以排程的備份次數 |Windows 伺服器/用戶端每日 3 次 <br/> 對於 SCDPM 每日 2 次 <br/> IaaS VM 每日一次 |
| 連接到 Azure 虛擬機器進行備份的資料磁碟 |16 |
| 連接到 Azure 虛擬機器進行備份的個別資料磁碟大小| 4095 GB|

* <sup>1</sup>54400 GB 上限並不適用於 IaaS VM 備份。
 

