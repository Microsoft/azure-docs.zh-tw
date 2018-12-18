---
title: Red Hat Update Infrastructure | Microsoft Docs
description: 了解適用於 Microsoft Azure 中隨選 Red Hat Enterprise Linux 執行個體的 Red Hat Update Infrastructure
services: virtual-machines-linux
documentationcenter: ''
author: BorisB2015
manager: jeconnoc
editor: ''
ms.assetid: f495f1b4-ae24-46b9-8d26-c617ce3daf3a
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/02/2018
ms.author: borisb
ms.openlocfilehash: 4a8bc45b253def1130e5a02dfcd6d359f0e74506
ms.sourcegitcommit: 76797c962fa04d8af9a7b9153eaa042cf74b2699
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42141932"
---
# <a name="red-hat-update-infrastructure-for-on-demand-red-hat-enterprise-linux-vms-in-azure"></a>適用於 Azure 中隨選 Red Hat Enterprise Linux VM 的 Red Hat Update Infrastructure
 [Red Hat Update Infrastructure](https://access.redhat.com/products/red-hat-update-infrastructure) (RHUI) 允許雲端提供者 (例如 Azure) 鏡像 Red Hat 代管的存放庫內容、建立具有 Azure 特定內容的自訂存放庫，以及讓它可供使用者 VM 使用。

Red Hat Enterprise Linux (RHEL) 預付型方案 (PAYG) 映像預先設定為存取 Azure RHUI。 不需要任何其他設定。 若要取得最新的更新，請在 RHEL 執行個體備妥之後執行 `sudo yum update`。 此服務包含在 RHEL PAYG 軟體費用中。

## <a name="important-information-about-azure-rhui"></a>Azure RHUI 的重要資訊
* Azure RHUI 目前僅支援每個 RHEL 系列 (RHEL6 或 RHEL7) 中的最新次要版本。 若要將連線至 RHUI 的 RHEL VM 執行個體會升級為最新的次要版本，請執行 `sudo yum update`。

    例如，如果您從 RHEL 7.2 PAYG 映像佈建 VM 並執行 `sudo yum update`，最終將會得到 RHEL 7.5 VM (RHEL7 系列中的最新次要版本)。

    若要避免此行為，您必須如[建立與上傳適用於 Azure 的 Red Hat 型虛擬機器](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)一文中所述，建置自己的映像。 之後，您需要將它連線至不同的更新基礎結構，像是[直接連線至 Red Hat 內容傳遞伺服器](https://access.redhat.com/solutions/253273) \(英文\) 或 [Red Hat 附屬伺服器](https://access.redhat.com/products/red-hat-satellite) \(英文\) 。

* 對 Azure 代管之 RHUI 的存取，包含在 RHEL PAYG 映像價格中。 如果您將 PAYG RHEL VM 從 Azure 代管的 RHUI 取消註冊，這樣並不會將虛擬機器轉換成自備授權 (BYOL) 類型的虛擬機器。 如果您以另一個更新來源註冊相同的 VM，可能會產生「間接」雙重費用。 您需要支付 Azure RHEL 軟體費用， 而且還需要支付先前已購買的 Red Hat 訂用帳戶費用。 如果您持續需要使用非 Azure 代管 RHUI 的更新基礎結構，請考慮建立及部署您自己的 (BYOL 類型) 映像。 [建立與上傳適用於 Azure 的 Red Hat 型虛擬機器](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)一文中有描述此流程。

* Azure 中兩個類別的 RHEL PAYG 映像 (RHEL for SAP Hana 和 RHEL for SAP Business Applications) 會連線至專用 RHUI 通道，以維持在 SAP 認證所需的特定 RHEL 次要版本。 

* 只有 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653) \(英文\) 中的 VM 能夠存取 Azure 代管的 RHUI。 如果您透過內部部署網路基礎結構為所有 VM 流量設定 Proxy，則可能需要為 RHEL PAYG VM 設定使用者定義的路由，以便存取 Azure RHUI。

### <a name="the-ips-for-the-rhui-content-delivery-servers"></a>RHUI 內容傳遞伺服器的 IP

在所有可使用 RHEL 隨選映像的地區，皆可使用 RHUI。 目前包含 [Azure 狀態儀表板](https://azure.microsoft.com/status/)頁面上所列的所有公用區域、Azure 美國政府和 Microsoft Azure 德國區域。 

如果您使用網路組態來進一步限制來自 RHEL PAYG VM 的存取，請確定已允許下列 IP，如此 `yum update` 才能依據您所在的環境運作： 

```
# Azure Global
13.91.47.76
40.85.190.91
52.187.75.218
52.174.163.213
52.237.203.198

# Azure US Government
13.72.186.193

# Azure Germany
51.5.243.77
51.4.228.145
```

## <a name="rhui-azure-infrastructure-update"></a>RHUI Azure 基礎結構更新

在 2016 年 9 月時，我們部署了更新的 Azure RHUI。 在 2017 年 4 月，我們關閉了舊版 Azure RHUI。 如果您在 2016 年 9 月或之後的時間使用 RHEL PAYG 映像 (或其快照集)，便會自動連線至新的 Azure RHUI。 不過，如果您的 VM 上有舊版快照集，則需要如下節所述手動更新其設定，才能存取 Azure RHUI。

新的 Azure RHUI 伺服器會透過 [Azure 流量管理員](https://azure.microsoft.com/services/traffic-manager/)來部署。 在流量管理員中，不論所在區域為何，任何 VM 皆可使用單一端點 (rhui-1.microsoft.com)。 

### <a name="troubleshoot-connection-problems-to-azure-rhui"></a>針對 Azure RHUI 連線問題進行疑難排解
從 Azure RHEL PAYG VM 連線至 RHUI 時若發生問題，請依照下列步驟操作：

1. 檢查 Azure RHUI 端點的 VM 組態：

    a. 檢查 `/etc/yum.repos.d/rh-cloud.repo` 檔案 `[rhui-microsoft-azure-rhel*]` 區段的 `baseurl` 中是否包含對 `rhui-[1-3].microsoft.com` 的參考。 如果有此參考，您使用的便是新版 Azure RHUI。

    b. 如果它指向具有 `mirrorlist.*cds[1-4].cloudapp.net` 模式的位置，則需要進行組態更新。 您正在使用舊版 VM 快照集，請更新它以指向新版 Azure RHUI。

1. 只有 [Azure 資料中心 IP 範圍] (https://www.microsoft.com/download/details.aspx?id=41653) \(英文\) 中的 VM 能夠存取 Azure 代管的 RHUI。
 
1. 如果您使用的是新設定，且已驗證 VM 是從 Azure IP 範圍連線，但仍無法連接至 Azure RHUI，請向 Microsoft 或 Red Hat 提出支援案例。

### <a name="manual-update-procedure-to-use-the-azure-rhui-servers"></a>使用 Azure RHUI 伺服器的手動更新程序
此程序僅供參考。 RHEL PAYG 映像已設定成能夠連線至 Azure RHUI。 若要手動更新設定以使用 Azure RHUI 伺服器，請完成下列步驟：

1. 透過 curl 下載公開金鑰簽章。

   ```bash
   curl -o RPM-GPG-KEY-microsoft-azure-release https://download.microsoft.com/download/9/D/9/9d945f05-541d-494f-9977-289b3ce8e774/microsoft-sign-public.asc 
   ```

1. 驗證已下載金鑰的有效性。

   ```bash
   gpg --list-packets --verbose < RPM-GPG-KEY-microsoft-azure-release
   ```

1. 檢查輸出，然後驗證 `keyid` 和 `user ID packet`。

   ```bash
   Version: GnuPG v1.4.7 (GNU/Linux)
   :public key packet:
           version 4, algo 1, created 1446074508, expires 0
           pkey[0]: [2048 bits]
           pkey[1]: [17 bits]
           keyid: EB3E94ADBE1229CF
   :user ID packet: "Microsoft (Release signing) <gpgsecurity@microsoft.com>"
   :signature packet: algo 1, keyid EB3E94ADBE1229CF
           version 4, created 1446074508, md5len 0, sigclass 0x13
           digest algo 2, begin of digest 1a 9b
           hashed subpkt 2 len 4 (sig created 2015-10-28)
           hashed subpkt 27 len 1 (key flags: 03)
           hashed subpkt 11 len 5 (pref-sym-algos: 9 8 7 3 2)
           hashed subpkt 21 len 3 (pref-hash-algos: 2 8 3)
           hashed subpkt 22 len 2 (pref-zip-algos: 2 1)
           hashed subpkt 30 len 1 (features: 01)
           hashed subpkt 23 len 1 (key server preferences: 80)
           subpkt 16 len 8 (issuer key ID EB3E94ADBE1229CF)
           data: [2047 bits]
   ```

1. 安裝公開金鑰。

   ```bash
   sudo install -o root -g root -m 644 RPM-GPG-KEY-microsoft-azure-release /etc/pki/rpm-gpg
   sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-microsoft-azure-release
   ```

1. 下載、驗證並安裝用戶端 RPM 套件管理員 (RPM)。
    
    >[!NOTE]
    >套件版本變更。 如果是手動連線至 Azure RHUI，藉由佈建資源庫最新映像，便可找到每個 RHEL 系列的最新版本用戶端套件。
  
   a. 下載。 
   
    - RHEL 6：
        ```bash
        curl -o azureclient.rpm https://rhui-1.microsoft.com/pulp/repos/microsoft-azure-rhel6/rhui-azure-rhel6-2.1-32.noarch.rpm 
        ```
    
    - RHEL 7：
        ```bash
        curl -o azureclient.rpm https://rhui-1.microsoft.com/pulp/repos/microsoft-azure-rhel7/rhui-azure-rhel7-2.1-19.noarch.rpm  
        ```

   b. 驗證。

   ```bash
   rpm -Kv azureclient.rpm
   ```

   c. 檢查輸出，以確認套件簽章無誤。

   ```bash
   azureclient.rpm:
       Header V3 RSA/SHA256 Signature, key ID be1229cf: OK
       Header SHA1 digest: OK (927a3b548146c95a3f6c1a5d5ae52258a8859ab3)
       V3 RSA/SHA256 Signature, key ID be1229cf: OK
       MD5 digest: OK (c04ff605f82f4be8c96020bf5c23b86c)
   ```

   d. 安裝 RPM。

    ```bash
    sudo rpm -U azureclient.rpm
    ```

1. 完成之後，請確認您可以從 VM 存取 Azure RHUI。

## <a name="next-steps"></a>後續步驟
若要從 Azure Marketplace PAYG 映像建立 Red Hat Enterprise Linux VM 並使用 Azure 代管的 RHUI，請移至 [Azure Marketplace](https://azure.microsoft.com/marketplace/partners/redhat/)。 
