---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: jimdial
ms.service: virtual-network
ms.topic: include
ms.date: 04/09/2018
ms.author: jdial
ms.custom: include file
ms.openlocfilehash: ec1727926f6dbfeead9932004715a8bb1dfbb0cd
ms.sourcegitcommit: 0408c7d1b6dd7ffd376a2241936167cc95cfe10f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2018
ms.locfileid: "36964530"
---
## <a name="os-config"></a>將 IP 位址新增至 VM 作業系統

連線並登入您使用多個私人 IP 位址建立的 VM。 您必須手動新增您新增至 VM 的所有私人 IP 位址 (包括主要位址)。 對您的 VM 作業系統完成後續步驟。

### <a name="windows"></a>Windows

1. 從命令提示字元輸入 *ipconfig /all*。  您只會看到 *Primary* 私人 IP 位址 (透過 DHCP)。
2. 在命令提示字元中輸入 ncpa.cpl，以開啟 [網路連線]。
3. 開啟適當介面卡的屬性：**區域連線**。
4. 按兩下 [網際網路通訊協定第 4 版 (IPv4)]。
5. 選取 [使用下列 IP 位址]  並輸入下列值︰

    * **IP 位址**︰輸入 *Primary* 私人 IP 位址
    * **子網路遮罩**︰根據您的子網路設定。 例如，如果子網路為 /24 子網路，則子網路遮罩為 255.255.255.0。
    * **預設閘道**︰子網路中的第一個 IP 位址。 如果您的子網路為 10.0.0.0/24，則閘道 IP 位址為 10.0.0.1。
    * 選取 [使用下列 DNS 伺服器位址]  並輸入下列值︰
        * **慣用 DNS 伺服器**︰如果您不是使用自己的 DNS 伺服器，請輸入 168.63.129.16。  如果您是使用自己的 DNS 伺服器，請輸入您的伺服器 IP 位址。
    * 選取 [進階] 按鈕，然後新增其他 IP 位址。 將您在上一個步驟中新增至 Azure 網路介面的每個次要私人 IP 位址新增至 Windows 網路介面，而該 Windows 網路介面已獲取指派給 Azure 網路介面的主要 IP 位址。

        您絕對不能手動指派在虛擬機器的作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。 當您手動設定作業系統內的 IP 位址時，請確保它的位址與指派給 Azure [網路介面](../articles/virtual-network/virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](../articles/virtual-network/virtual-network-network-interface-addresses.md#private)設定。 您絕對不能指派作業系統內的 Azure 公用 IP 位址。

    * 按一下 [確定] 關閉 TCP/IP 設定，然後再按一次 [確定] 關閉介面卡設定。 您的 RDP 連接已重建。

6. 從命令提示字元輸入 *ipconfig /all*。 此時會顯示您加入的所有 IP 位址，而 DHCP 是關閉的。
7. 設定 Windows 在 Azure 中使用主要 IP 設定的私人 IP 位址，作為適用於 Windows 的主要 IP 位址。 如需詳細資訊，請參閱[從具有多個 IP 位址的 Azure Windows VM 無權存取網際網路](https://support.microsoft.com/help/4040882/no-internet-access-from-azure-windows-vm-that-has-multiple-ip-addresse)。 

### <a name="validation-windows"></a>驗證 (Windows)

若要確保您能夠透過相關聯的公用 IP 從第二個 IP 組態連接到網際網路，在使用上述步驟正確地新增該 IP 後，請使用下列命令︰

```bash
ping -S 10.0.0.5 hotmail.com
```
>[!NOTE]
>對於次要 IP 組態，如果組態有與其相關聯的公用 IP 位址，您只可以 Ping 網際網路。 對於主要 IP 組態，公用 IP 位址不需要 Ping 網際網路。

### <a name="linux-ubuntu"></a>Linux (Ubuntu)

1. 開啟終端機視窗。
2. 請確定您是 root 使用者。 如果不是，請輸入下列命令：

    ```bash
    sudo -i
    ```

3. 更新網路介面 (假設為 'eth0') 的組態檔。

    * 保留針對 dhcp 的現有行。 主要 IP 位址的設定仍然與先前一樣。
    * 使用下列命令，新增其他靜態 IP 位址的組態︰

        ```bash
        cd /etc/network/interfaces.d/
        ls
        ```

    您應該會看到一個 .cfg 檔案。
4. 開啟 檔案。 您應該會在檔案結尾看到下列這幾行：

    ```bash
    auto eth0
    iface eth0 inet dhcp
    ```

5. 在此檔案已有的幾行後面加入下列這幾行︰

    ```bash
    iface eth0 inet static
    address <your private IP address here>
    netmask <your subnet mask>
    ```

6. 使用下列命令儲存檔案︰

    ```bash
    :wq
    ```

7. 使用下列命令重設網路介面︰

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

    > [!IMPORTANT]
    > 如果使用遠端連線，請在同一行中同時執行 ifdown 和 ifup。
    >

8. 使用下列命令確認 IP 位址已加入網路介面︰

    ```bash
    ip addr list eth0
    ```

    您應該會在清單中看到您加入的 IP 位址。

### <a name="linux-red-hat-centos-and-others"></a>Linux (Red Hat、CentOS 以及其他)

1. 開啟終端機視窗。
2. 請確定您是 root 使用者。 如果不是，請輸入下列命令：

    ```bash
    sudo -i
    ```

3. 輸入您的密碼，並且依照提示的指示。 成為 root 使用者之後，使用下列命令瀏覽至網路指令碼資料夾︰

    ```bash
    cd /etc/sysconfig/network-scripts
    ```

4. 使用下列命令列出相關的 ifcfg 檔案︰

    ```bash
    ls ifcfg-*
    ```

    您應該會看到其中一個檔案是 *ifcfg-eth0* 。

5. 若要新增 IP 位址，如下所示，為其建立組態檔。 請注意，必須針對每個 IP 組態建立一個檔案。

    ```bash
    touch ifcfg-eth0:0
    ```

6. 使用下列命令開啟 *ifcfg-eth0:0* 檔案︰

    ```bash
    vi ifcfg-eth0:0
    ```

7. 使用下列命令，新增檔案內容，在此案例中為 eth0:0。 請務必根據您的 IP 位址更新資訊。

    ```bash
    DEVICE=eth0:0
    BOOTPROTO=static
    ONBOOT=yes
    IPADDR=192.168.101.101
    NETMASK=255.255.255.0
    ```

8. 使用下列命令儲存檔案︰

    ```bash
    :wq
    ```

9. 執行下列命令重新啟動網路服務，並確定所做的變更都成功︰

    ```bash
    /etc/init.d/network restart
    ifconfig
    ```

    您應該會在傳回的清單中看到您加入的 IP 位址 *eth0:0*。

### <a name="validation-linux"></a>驗證 (Linux)

若要確保您能夠透過相關聯的公用 IP 從第二個 IP 組態連接到網際網路，請使用下列命令︰

```bash
ping -I 10.0.0.5 hotmail.com
```
>[!NOTE]
>對於次要 IP 組態，如果組態有與其相關聯的公用 IP 位址，您只可以 Ping 網際網路。 對於主要 IP 組態，公用 IP 位址不需要 Ping 網際網路。

對於 Linux VM，在嘗試驗證來自次要 NIC 的輸出連線能力時，您可能需要新增適當的路由。 有許多方法可以執行這項作業。 請參閱您的 Linux 散發套件相關文件。 以下是完成這項作業的其中一種方法︰

```bash
echo 150 custom >> /etc/iproute2/rt_tables 

ip rule add from 10.0.0.5 lookup custom
ip route add default via 10.0.0.1 dev eth2 table custom

```
- 請務必︰
    - 將 **10.0.0.5** 替換成有相關聯公用 IP 位址的私人 IP 位址
    - 將 **10.0.0.1** 替換為您的預設閘道
    - 將 **eth2** 替換為您的次要 NIC 名稱
