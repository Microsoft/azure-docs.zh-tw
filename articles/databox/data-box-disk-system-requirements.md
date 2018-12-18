---
title: Microsoft Azure 資料箱磁碟的系統需求 | Microsoft Docs
description: 了解 Azure 資料箱磁碟的軟體和網路需求
services: databox
documentationcenter: NA
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: databox
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/06/2018
ms.author: alkohli
ms.openlocfilehash: aaa4e4bb24ca42adb9d283e6286dbef879bcb1ea
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299843"
---
# <a name="azure-data-box-disk-system-requirements-preview"></a>Azure 資料箱磁碟的系統需求 (預覽)

本文會針對 Microsoft Azure 資料箱磁碟解決方案以及連線至資料箱磁碟的用戶端，說明其各自的重要系統需求。 建議您先仔細檢閱此資訊再部署資料箱磁碟，之後在部署和後續作業期間若有必要，也請回頭查閱。

> [!IMPORTANT]
> 資料箱磁碟目前為預覽版。 部署此解決方案之前，請先檢閱[預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 

系統需求中包含可支援用戶端連線至磁碟的平台、支援的儲存體帳戶，以及儲存體類型。


## <a name="supported-operating-systems-for-clients"></a>支援用戶端的作業系統

以下是所支援作業系統的清單，這些系統可讓連線至資料箱磁碟的用戶端進行磁碟解除鎖定和資料複製作業。

| **作業系統** | **測試的版本** |
| --- | --- |
| Windows Server |2008 R2 SP1 <br> 2012 <br> 2012 R2 <br> 2016 |
| Windows |7, 8, 10 |
|Linux <br> <li> Ubuntu </li><li> Debian </li><li> Red Hat Enterprise Linux (RHEL) </li><li> CentOS| <br>14.04、16.04、18.04 <br> 8.11、9 <br> 7.0 <br> 6.5、6.9、7.0、7.5 |  

## <a name="other-required-software-for-windows-clients"></a>Windows 用戶端的其他必要軟體

針對 Windows 用戶端，也應安裝下列項目。

| **軟體**| **版本** |
| --- | --- |
| Windows PowerShell |5.0 |
| .NET Framework |4.5.1 |
| Windows Management Framework |5.0|
| BitLocker| - |

## <a name="other-required-software-for-linux-clients"></a>Linux 用戶端的其他必要軟體

針對 Linux 用戶端，資料箱磁碟工具組會安裝下列必要軟體：

- dislocker
- OpenSSL

## <a name="supported-storage-accounts"></a>支援的儲存體帳戶

以下是資料箱磁碟所支援的儲存體類型清單。

| **儲存體帳戶** | **注意事項** |
| --- | --- |
| 傳統 | 標準 |
| 一般用途  |標準；同時支援 V1 和 V2。 同時支援經常性儲存層和非經常性儲存層。 |


## <a name="supported-storage-types"></a>支援的儲存體類型

以下是資料箱磁碟所支援的儲存體類型清單。

| **檔案格式** | **注意事項** |
| --- | --- |
| Azure 區塊 Blob | |
| Azure 分頁 Blob  | |


## <a name="next-step"></a>後續步驟

* [部署 Azure 資料箱磁碟](data-box-disk-deploy-ordered.md)

