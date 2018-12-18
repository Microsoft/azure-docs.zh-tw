---
title: 設定 Linux VM 的 DHCPv6 | Microsoft Docs
description: 如何設定 Linux VM 的 DHCPv6。
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
editor: ''
keywords: ipv6, azure load balancer, 雙重堆疊, 公用 ip, 原生 ipv6, 行動, iot
ms.assetid: b32719b6-00e8-4cd0-ba7f-e60e8146084b
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: 6248ed2f55fb5bbcc2061af6ce1dedf2bd31ccad
ms.sourcegitcommit: c3d53d8901622f93efcd13a31863161019325216
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/29/2018
ms.locfileid: "30261842"
---
# <a name="configure-dhcpv6-for-linux-vms"></a>設定 Linux VM 的 DHCPv6


Azure Marketplace 中的一些 Linux 虛擬機器映像沒有預設的動態主機設定通訊協定版本 6 (DHCPv6) 設定。 若要支援 IPv6，在您使用的 Linux OS 散發套件中必須設定 DHCPv6。 各種 Linux 發行套件會以各種不同的方式設定 DHCPv6，因為它們使用不同的套件。

> [!NOTE]
> Azure Marketplace 中最新的 SUSE Linux 和 CoreOS 映像已有預先設定 DHCPv6。 使用這些映像不需要再進行額外的變更。

本文件說明如何啟用 DHCPv6 使您的 Linux 虛擬機器取得 IPv6 位址。

> [!WARNING]
> 不當編輯網路組態檔可能會使您失去 VM 的網路存取權。 我們建議您先在非生產系統上測試組態變更。 本文中的指示已經過在 Azure Marketplace 中最新版 Linux 映像上測試過。 如需更詳細的指示，請參閱您所用 Linux 版本的文件。

## <a name="ubuntu"></a>Ubuntu

1. 編輯 */etc/dhcp/dhclient6.conf* 檔案，並新增下行：

        timeout 10;

2. 編輯下列組態的 eth0 介面網路組態︰

   * 在 **Ubuntu 12.04 和 14.04** 上，編輯 */etc/network/interfaces.d/eth0.cfg* 檔案。 
   * 在 **Ubuntu 16.04** 上，編輯 */etc/network/interfaces.d/50-cloud-init.cfg* 檔案。

         iface eth0 inet6 auto
             up sleep 5
             up dhclient -1 -6 -cf /etc/dhcp/dhclient6.conf -lf /var/lib/dhcp/dhclient6.eth0.leases -v eth0 || true

3. 更新 IPv6 位址︰

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="debian"></a>Debian

1. 編輯 */etc/dhcp/dhclient6.conf* 檔案，並新增下行：

        timeout 10;

2. 編輯 */etc/network/interfaces* 檔案，並新增下列組態：

        iface eth0 inet6 auto
            up sleep 5
            up dhclient -1 -6 -cf /etc/dhcp/dhclient6.conf -lf /var/lib/dhcp/dhclient6.eth0.leases -v eth0 || true

3. 更新 IPv6 位址︰

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="rhel-centos-and-oracle-linux"></a>RHEL、CentOS 和 Oracle Linux

1. 編輯 */etc/sysconfig/network* 檔案，並新增下列參數：

        NETWORKING_IPV6=yes

2. 編輯 */etc/sysconfig/network-scripts/ifcfg-eth0* 檔案，並新增下列兩個參數：

        IPV6INIT=yes
        DHCPV6C=yes

3. 更新 IPv6 位址︰

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="sles-11-and-opensuse-13"></a>SLES 11 和 openSUSE 13

在 Azure 中最新的 SUSE Linux Enterprise Server (SLES) 和 openSUSE 映像已預先設定 DHCPv6。 使用這些映像不需要再進行額外的變更。 如果您的 VM 是以較舊或自訂 SUSE 映像建置而成，請執行下列步驟︰

1. 如有需要，安裝 `dhcp-client` 套件：

    ```bash
    sudo zypper install dhcp-client
    ```

2. 編輯 */etc/sysconfig/network/ifcfg-eth0* 檔案，並新增下列參數：

        DHCLIENT6_MODE='managed'

3. 更新 IPv6 位址︰

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="sles-12-and-opensuse-leap"></a>SLES 12 和 openSUSE Leap

在 Azure 中最新的 SLES 和 openSUSE 映像已預先設定 DHCPv6。 使用這些映像不需要再進行額外的變更。 如果您的 VM 是以較舊或自訂 SUSE 映像建置而成，請執行下列步驟︰

1. 編輯 */etc/sysconfig/network/ifcfg-eth0* 檔案，並以下列值取代 `#BOOTPROTO='dhcp4'` 參數：

        BOOTPROTO='dhcp'

2. 對 */etc/sysconfig/network/ifcfg-eth0* 檔案，新增下列參數：

        DHCLIENT6_MODE='managed'

3. 更新 IPv6 位址︰

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="coreos"></a>CoreOS

在 Azure 中最新的 SLES 映像已預先設定 DHCPv6。 使用這些映像不需要再進行額外的變更。 如果您的 VM 是以較舊或自訂 CoreOS 映像建置而成，請執行下列步驟︰

1. 編輯 */etc/systemd/network/10_dhcp.network* 檔案：

        [Match]
        eth0

        [Network]
        DHCP=ipv6

2. 更新 IPv6 位址︰

    ```bash
    sudo systemctl restart systemd-networkd
    ```
