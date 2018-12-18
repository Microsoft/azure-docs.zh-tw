---
title: 升級 Azure Migrate 中的收集器設備 | Microsoft Docs
description: 提供有關 Azure Migrate 收集器設備升級的資訊。
author: musa-57
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/28/2018
ms.author: hamusa
services: azure-migrate
ms.openlocfilehash: 6080096194ec5b11f1d5593fcbb6732d1551667f
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47431387"
---
# <a name="collector-update-release-history"></a>收集器更新版本歷程記錄

此文章摘要說明 [Azure Migrate](migrate-overview.md) 中收集器設備的升級資訊。

「Azure Migrate 收集器」是一個輕量型設備，可用來探索內部部署 vCenter 環境，目的是要在移轉至 Azure 之前先進行評量。 [深入了解](concepts-collector.md)。


## <a name="one-time-discovery-upgrade-versions"></a>單次探索：升級版本

### <a name="version-10914"></a>1.0.9.14 版

升級[套件 1.0.9.14](https://aka.ms/migrate/col/upgrade_9_14) 的雜湊值

**演算法** | **雜湊值**
--- | ---
MD5 | c5bf029e9fac682c6b85078a61c5c79c
SHA1 | af66656951105e42680dfcc3ec3abd3f4da8fdec
SHA256 | 58b685b2707f273aa76f2e1d45f97b0543a8c4d017cd27f0bdb220e6984cc90e

### <a name="version-10913"></a>1.0.9.13 版

升級[套件 1.0.9.13](https://aka.ms/migrate/col/upgrade_9_13) 的雜湊值

**演算法** | **雜湊值**
--- | ---
MD5 | 739f588fe7fb95ce2a9b6b4d0bf9917e
SHA1 | 9b3365acad038eb1c62ca2b2de1467cb8eed37f6
SHA256 | 7a49fb8286595f39a29085534f29a623ec2edb12a3d76f90c9654b2f69eef87e

### <a name="version-10911"></a>1.0.9.11 版

升級[套件 1.0.9.11](https://aka.ms/migrate/col/upgrade_9_11) 的雜湊值

**演算法** | **雜湊值**
--- | ---
MD5 | 0e36129ac5383b204720df7a56b95a60
SHA1 | aa422ef6aa6b6f8bc88f27727e80272241de1bdf
SHA256 | 5f76dbbe40c5ccab3502cc1c5f074e4b4bcbf356d3721fd52fb7ff583ff2b68f

### <a name="version-1097"></a>1.0.9.7 版

升級[套件 1.0.9.7](https://aka.ms/migrate/col/upgrade_9_7) 的雜湊值

**演算法** | **雜湊值**
--- | ---
MD5 | 01ccd6bc0281f63f2a672952a2a25363
SHA1 | 3e6c57523a30d5610acdaa14b833c070bffddbff
SHA256 | e3ee031fb2d47b7881cc5b13750fc7df541028e0a1cc038c796789139aa8e1e6

## <a name="continuous-discovery-upgrade-versions"></a>連續探索：升級版本

目前針對連續探索設備尚未提供任何升級。

## <a name="run-an-upgrade"></a>執行升級

您無須再次下載 OVA，即可將收集器升級至最新版本。

1. 您需下載以下清單中的最新升級套件。
2. 若要確保下載的 Hotfix 是安全的，您需開啟系統管理員命令視窗，然後執行下列命令來產生 ZIP 檔案的雜湊。 產生的雜湊應該符合針對特定版本所述的雜湊：

    ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```

    範例：**C:\>CertUtil -HashFile C:\AzureMigrate\CollectorUpdate_release_1.0.9.14.zip SHA256)**
3. 將 ZIP 檔案複製到收集器設備 VM。
4. 在 ZIP 檔案上按一下滑鼠右鍵 > [解壓縮全部]。
5. 在 [Setup.ps1] 上按一下滑鼠右鍵 > [用 PowerShell 執行]，然後依照安裝指示進行操作。


## <a name="next-steps"></a>後續步驟

- [深入了解](concepts-collector.md)收集器設備。
- 為 VMware VM [執行評量](tutorial-assessment-vmware.md)。
