---
title: Microsoft Azure 資料箱的系統需求 | Microsoft Docs
description: 了解 Azure 資料箱的軟體和網路需求
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: 957100e042da1d41fb7c8fa27a5663cf99a41c20
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093930"
---
# <a name="azure-data-box-system-requirements"></a>Azure 資料箱的系統需求

本文會針對 Microsoft Azure 資料箱以及連線至資料箱的用戶端，說明其各自的重要系統需求。 建議您先仔細檢閱此資訊再部署資料箱，之後在部署和後續作業期間若有必要，也請回頭查閱。

系統需求包括：

* **連線到資料箱的主機軟體需求** - 說明支援的平台、本機 Web UI 的瀏覽器、SMB 用戶端，以及可以連線到資料箱的主機其他需求。
* **資料箱的網路需求** - 提供資料箱最佳作業的網路需求相關資訊。


## <a name="software-requirements"></a>軟體需求

軟體需求包括支援的作業系統、支援的本機 Web UI 瀏覽器和 SMB 用戶端相關資訊。

### <a name="supported-operating-systems-for-clients"></a>支援用戶端的作業系統

以下是所支援的作業系統清單，這些系統可讓連線至資料箱的用戶端進行資料複製作業。

| **作業系統** | **版本** | 
| --- | --- | 
| Windows Server |2008 R2 SP1 <br> 2012 <br> 2012 R2 <br> 2016 | 
| Windows |7, 8, 10 | 
|Linux    |         |

### <a name="supported-file-systems-for-linux-clients"></a>Linux 用戶端的支援檔案系統

| **通訊協定** | **版本** | 
| --- | --- | 
| SMB |2.X 和更新版本 |
| NFS | 4.1 版 (含) 之前的所有版本|

### <a name="supported-storage-accounts"></a>支援的儲存體帳戶

以下是資料箱裝置所支援的儲存體類型清單。

| **儲存體帳戶** | **注意事項** |
| --- | --- |
| 傳統 | 標準 |
| 一般用途  |標準；同時支援 V1 和 V2。 |
| Blob |經常性存取和非經常性存取均受支援。 |


### <a name="supported-storage-types"></a>支援的儲存體類型

以下是資料箱裝置所支援的儲存體類型清單。

| **檔案格式** | **注意事項** |
| --- | --- |
| Azure 區塊 Blob | |
| Azure 分頁 Blob  | 資料應該是 512 位元組。|
| Azure 檔案 | |


### <a name="supported-web-browsers"></a>支援的網頁瀏覽器

以下是本機 Web UI 的支援網頁瀏覽器清單。

| **[瀏覽器]** | **版本** | **其他需求/注意事項** |
| --- | --- | --- |
| Google Chrome |最新版本 |通過 Chrome 測試|
| Microsoft Edge |最新版本 | |
| FireFox | 最新版本 | 通過 FireFox 測試|
| Internet Explorer |最新版本 |如果您無法登入，請檢查是否已啟用 cookie 和 Javascript。 若要啟用 UI 存取，請將裝置 IP 新增至 [隱私權動作]，讓裝置可以存取 cookie。 |


## <a name="networking-requirements"></a>網路需求

您的資料中心必須有高速網路。 我們強烈建議您具有至少一個 10 GbE 的連線。 如果無法使用 10 GbE 連線，也可以使用 1 GbE 資料連結來複製資料，但是複製速度會受到影響。

## <a name="next-step"></a>後續步驟

* [部署 Azure 資料箱](data-box-deploy-ordered.md)

