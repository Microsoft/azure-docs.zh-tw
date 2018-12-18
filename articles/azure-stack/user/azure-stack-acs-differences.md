---
title: Azure Stack 儲存體差異與考量 | Microsoft Docs
description: 了解「Azure Stack 儲存體」與「Azure 儲存體」之間的差異，以及 Azure Stack 部署考量。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/05/2018
ms.author: mabrigg
ms.reviwer: xiaofmao
ms.openlocfilehash: 14e32bdfcde6969b820c0950d59bd5cf946a51e6
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48802316"
---
# <a name="azure-stack-storage-differences-and-considerations"></a>Azure Stack 儲存體：差異與考量

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

「Azure Stack 儲存體」是 Microsoft Azure Stack 中的一組儲存體雲端服務。 「Azure Stack 儲存體」使用與 Azure 一致的語意來提供 Blob、資料表、佇列及帳戶管理功能。

此文章摘要說明 Azure Stack 儲存體與 Azure 儲存體服務的已知差異。 也會列出部署 Azure Stack 時所要考量的事項。 若要了解全域 Azure 與 Azure Stack 之間的大致差異，請參閱[主要考量](azure-stack-considerations.md)一文。

## <a name="cheat-sheet-storage-differences"></a>速查表：儲存體差異

| 功能 | Azure (全域) | Azure Stack |
| --- | --- | --- |
|檔案儲存體|支援雲端式 SMB 檔案共用|尚不支援
|待用資料的 Azure 儲存體服務加密|256 位元 AES 加密|BitLocker 128 位元 AES 加密
|儲存體帳戶類型|一般用途和 Azure Blob 儲存體帳戶|僅限一般用途。
|複寫選項|本地備援儲存體、異地備援儲存體、讀取權限異地備援儲存體，以及區域備援儲存體|本地備援儲存體。
|進階儲存體|完全支援|可佈建，但無效能限制或保證。
|受控磁碟|支援進階和標準|支援 1808 版或更新版本。
|Blob 名稱|1,024 個字元 (2,048 個位元組)|880 個字元 (1,760 個位元組)
|區塊 Blob 大小上限|4.75 TB (100 MB X 50,000 個區塊)|適用於 1802 更新或更新版本的 4.75 TB (100 MB x 50,000 個區塊)。 適用於舊版的 50,000 X 4 MB (大約 195 GB)。
|分頁 Blob 增量快照複製|支援進階和標準 Azure 分頁 Blob|尚不支援。
|Blob 儲存體的儲存層|經常性存取、非經常性存取和封存儲存層。|尚不支援。
Blob 儲存體的虛刪除|預覽|尚不支援。
|分頁 Blob 大小上限|8 TB|1 TB
|分頁 Blob 分頁大小|512 個位元組|4 KB
|資料表分割區索引鍵和資料列索引鍵大小|1,024 個字元 (2,048 個位元組)|400 個字元 (800 個位元組)
|Blob 快照集|一個 blob 的快照集數目沒有上限。|一個 blob 的快照集數目上限為 1,000。|

儲存體計量也有些差異：

* 儲存體度量中的交易資料並不區分內部或外部網路頻寬。
* 儲存體度量中的交易資料並不包含虛擬機器對所掛接磁碟的存取。

## <a name="api-version"></a>API 版本

使用「Azure Stack 儲存體」時支援下列版本：

Azure 儲存體服務 API：

1802 更新或更新版本：

 - [2017-04-17](https://docs.microsoft.com/rest/api/storageservices/version-2017-04-17)
 - [2016-05-31](https://docs.microsoft.com/rest/api/storageservices/version-2016-05-31)
 - [2015-12-11](https://docs.microsoft.com/rest/api/storageservices/version-2015-12-11)
 - [2015-07-08](https://docs.microsoft.com/rest/api/storageservices/version-2015-07-08)
 - [2015-04-05](https://docs.microsoft.com/rest/api/storageservices/version-2015-04-05)

舊版：

 - [2015-04-05](https://docs.microsoft.com/rest/api/storageservices/version-2015-04-05)

Azure 儲存體服務管理 API：

 - [2015-05-01-preview](https://docs.microsoft.com/rest/api/storagerp/?redirectedfrom=MSDN)
 - [2015-06-15](https://docs.microsoft.com/rest/api/storagerp/?redirectedfrom=MSDN)
 - [2016-01-01](https://docs.microsoft.com/rest/api/storagerp/?redirectedfrom=MSDN)

## <a name="sdk-versions"></a>SDK 版本

Azure Stack 儲存體支援下列用戶端程式庫：

| 用戶端程式庫 | Azure Stack 支援的版本 | 連結                                                                                                                                                                                                                                                                                                                                     | 端點規格       |
|----------------|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| .NET           | 從 6.2.0 到 8.7.0.          | Nuget 套件：<br>https://www.nuget.org/packages/WindowsAzure.Storage/<br> <br>GitHub 版本：<br>https://github.com/Azure/azure-storage-net/releases                                                                                                                                                                                    | app.config 檔案              |
| Java           | 從 4.1.0 到 6.1.0           | Maven 套件：<br>http://mvnrepository.com/artifact/com.microsoft.azure/azure-storage<br> <br>GitHub 版本：<br>https://github.com/Azure/azure-storage-java/releases                                                                                                                                                                    | 連接字串設定      |
| Node.js        | 從 1.1.0 到 2.7.0           | NPM 連結：<br>https://www.npmjs.com/package/azure-storage<br>(例如：執行 "npm install azure-storage@2.7.0")<br> <br>GitHub 版本：<br>https://github.com/Azure/azure-storage-node/releases                                                                                                                                         | 服務執行個體宣告 |
| C++            | 從 2.4.0 到 3.1.0           | Nuget 套件：<br>https://www.nuget.org/packages/wastorage.v140/<br> <br>GitHub 版本：<br>https://github.com/Azure/azure-storage-cpp/releases                                                                                                                                                                                          | 連接字串設定      |
| PHP            | 從 0.15.0 到 1.0.0          | GitHub 版本：<br>https://github.com/Azure/azure-storage-php/releases<br> <br>透過編輯器安裝 (請參閱下面的詳細資料)                                                                                                                                                                                                                  | 連接字串設定      |
| Python         | 從 0.30.0 到 1.0.0          | GitHub 版本：<br>https://github.com/Azure/azure-storage-python/releases                                                                                                                                                                                                                                                                | 服務執行個體宣告 |
| Ruby           | 從 0.12.1 從 1.0.1          | RubyGems 套件：<br>一般：<br>https://rubygems.org/gems/azure-storage-common/<br>Blob： https://rubygems.org/gems/azure-storage-blob/<br>佇列： https://rubygems.org/gems/azure-storage-queue/<br>資料表： https://rubygems.org/gems/azure-storage-table/<br> <br>GitHub 版本：<br>https://github.com/Azure/azure-storage-ruby/releases | 連接字串設定      |

## <a name="next-steps"></a>後續步驟

* [開始使用 Azure Stack 儲存體開發工具](azure-stack-storage-dev.md)
* [Azure Stack 儲存體簡介](azure-stack-storage-overview.md)
