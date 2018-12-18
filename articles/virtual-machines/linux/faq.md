---
title: Azure 中 Linux VM 的常見問題集 | Microsoft Docs
description: 針對以 Resource Manager 模型建立的 Linux 虛擬機器，提供一些相關常見問題的解答。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-management
ms.assetid: 3648e09c-1115-4818-93c6-688d7a54a353
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/22/2018
ms.author: cynthn
ms.openlocfilehash: ffc724c52fdab01bfbf2846ff06a35f38a57c8eb
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42141076"
---
# <a name="frequently-asked-question-about-linux-virtual-machines"></a>Linux 虛擬機器的常見問題
本文可解決在 Azure 中使用 Resource Manager 部署模型建立之 Linux 虛擬機器的一些常見問題。 如需本主題的 Windows 版本，請參閱 [Windows 虛擬機器的常見問題](../windows/faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="what-can-i-run-on-an-azure-vm"></a>我可以在 Azure VM 上執行什麼？
所有的訂閱者都可以在 Azure 虛擬機器上執行伺服器軟體。 如需詳細資訊，請參閱 [經 Azure 背書之配送映像上的 Linux](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>我可以使用多少的儲存體搭配虛擬機器？
每個資料磁碟最多可達 4 TB (4,095 GB)。 可使用的資料磁碟數量取決於虛擬機器的大小。 如需詳細資訊，請參閱 [虛擬機器的大小](sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

Azure 受控磁碟是受歡迎磁碟儲存體供應項目，可用在 Azure 虛擬機器當作資料的用續性儲存體。 每部虛擬機器可使用多部受控磁碟。 受控磁碟提供兩種耐久的儲存體選項：進階與標準受控磁碟。 如需定價資訊，請參閱[受控磁碟定價](https://azure.microsoft.com/pricing/details/managed-disks)。

Azure 儲存體帳戶也提供作業系統磁碟和任何資料磁碟的儲存空間。 每個磁碟是以分頁 Blob 方式儲存的 .vhd 檔案。 如需定價的詳細資料，請參閱 [儲存體定價詳細資料](https://azure.microsoft.com/pricing/details/storage/)。

## <a name="how-can-i-access-my-virtual-machine"></a>如何存取我的虛擬機器？
使用安全殼層 (SSH) 建立遠端連線來登入虛擬機器。 請參閱如何[從 Windows](ssh-from-windows.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 或[從 Linux 及 Mac](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 連線的指示。 根據預設，SSH 允許最多 10 個並行連線。 您可以編輯組態檔以增加這個數字。

如果您遇到問題，請參閱 [疑難排解以 Linux 為基礎之 Azure 虛擬機器的安全殼層 (SSH) 連線](troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="can-i-use-the-temporary-disk-devsdb1-to-store-data"></a>我可以使用暫存磁碟 (/dev/sdb1) 儲存資料嗎？
請勿使用暫存磁碟 (/dev/sdb1) 來儲存資料。 它只是用於暫時儲存。 您可能會遺失資料且無法復原。

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>我是否可以複製或再製現有的 Azure VM？
是。 如需相關指示，請參閱 [如何在 Resource Manager 部署模型中建立 Linux 虛擬機器的複本](copy-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="why-am-i-not-seeing-canada-central-and-canada-east-regions-through-azure-resource-manager"></a>為什麼我透過 Azure Resource Manager 沒看到加拿大中部和加拿大東部區域？
針對現有 Azure 訂用帳戶所建立的虛擬機器，不會自動註冊加拿大中部和加拿大東部這兩個新的區域。 當虛擬機器透過 Azure 入口網站使用 Azure Resource Manager 部署到任何其他區域時，就會自動完成註冊。 將虛擬機器部署到任何其他 Azure 區域之後，新的區域即可供後續的虛擬機器使用。

## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>我可以在建立 VM 之後將 NIC 新增至此 VM 嗎？
是的，目前可行。 您必須先停止解除配置 VM。 然後您可以新增或移除 NIC (除非它是 VM 上的最後一個 NIC)。 

## <a name="are-there-any-computer-name-requirements"></a>是否有任何電腦名稱需求？
是。 電腦名稱的長度最多可以有 64 個字元。 如需命名資源相關詳細資訊，請參閱[命名慣例規則與限制](/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="are-there-any-resource-group-name-requirements"></a>是否有任何資源群組名稱需求？
是。 資源群組名稱長度最多可以有 90 個字元。 如需資源群組相關詳細資訊，請參閱[命名慣例規則與限制](/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>建立 VM 時的使用者名稱需求為何？

使用者名稱的長度應介於 1 - 32 個字元之間。

不允許下列使用者名稱︰

<table>
    <tr>
        <td style="text-align:center">administrator </td><td style="text-align:center"> admin </td><td style="text-align:center"> user </td><td style="text-align:center"> user1</td>
    </tr>
    <tr>
        <td style="text-align:center">test </td><td style="text-align:center"> user2 </td><td style="text-align:center"> test1 </td><td style="text-align:center"> user3</td>
    </tr>
    <tr>
        <td style="text-align:center">admin1 </td><td style="text-align:center"> 1 </td><td style="text-align:center"> 123 </td><td style="text-align:center"> a</td>
    </tr>
    <tr>
        <td style="text-align:center">actuser  </td><td style="text-align:center"> adm </td><td style="text-align:center"> admin2 </td><td style="text-align:center"> aspnet</td>
    </tr>
    <tr>
        <td style="text-align:center">backup </td><td style="text-align:center"> console </td><td style="text-align:center"> david </td><td style="text-align:center"> guest</td>
    </tr>
    <tr>
        <td style="text-align:center">john </td><td style="text-align:center"> owner </td><td style="text-align:center"> root </td><td style="text-align:center"> 伺服器</td>
    </tr>
    <tr>
        <td style="text-align:center">sql </td><td style="text-align:center"> 支援 </td><td style="text-align:center"> support_388945a0 </td><td style="text-align:center"> sys</td>
    </tr>
    <tr>
        <td style="text-align:center">test2 </td><td style="text-align:center"> test3 </td><td style="text-align:center"> user4 </td><td style="text-align:center"> user5</td>
    </tr>
</table>


## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>建立 VM 時的密碼需求為何？
密碼必須是長度在 6 - 72 之間的字元，且符合下列 4 個複雜性需求的其中 3 個：

* 包含小寫字元
* 包含大小字元
* 包含數字
* 包含特殊字元 (Regex match [\W_])

不允許下列密碼︰

<table>
    <tr>
        <td style="text-align:center">abc@123</td>
        <td style="text-align:center">P@$$w0rd</td>
        <td style="text-align:center">P@ssw0rd</td>
        <td style="text-align:center">P@ssword123</td>
        <td style="text-align:center">Pa$$word</td>
    </tr>
    <tr>
        <td style="text-align:center">pass@word1</td>
        <td style="text-align:center">Password!</td>
        <td style="text-align:center">Password1</td>
        <td style="text-align:center">Password22</td>
        <td style="text-align:center">iloveyou!</td>
    </tr>
</table>
