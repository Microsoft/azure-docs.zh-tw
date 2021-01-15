---
title: 檢視與修改主機名稱 | Microsoft Docs
description: 如何檢視和變更 Azure 虛擬機器的主機名稱、Web 和背景工作角色以進行名稱解析
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
ms.assetid: c668cd8e-4e43-4d05-acc3-db64fa78d828
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/30/2018
ms.author: genli
ms.openlocfilehash: ed250e3f32965fc450102fb14b93b93d6753ab3e
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98222780"
---
# <a name="viewing-and-modifying-hostnames"></a>檢視與修改主機名稱
若要允許主機名稱參考您的角色執行個體，您必須在各個角色的服務組態檔中設定主機名稱的值。 請將所需的主機名稱加入至 [角色] 元素的 [vmName] 屬性。 [vmName] 屬性的值可做為每個角色執行個體的主機名稱基底。 例如，如果 [vmName] 是 [webrole]，而該角色有三個執行個體，執行個體的主機名稱會是 *webrole0*、*webrole1* 和 *webrole2*。 您不需要指定組態檔中虛擬機器的主機名稱，因為虛擬機器的主機名稱會根據虛擬機器名稱填入。 如需設定 Microsoft Azure 服務的詳細資訊，請參閱 [Azure 服務組態結構描述 (.cscfg 檔)](/previous-versions/azure/reference/ee758710(v=azure.100))

## <a name="viewing-hostnames"></a>檢視主機名稱
您可以使用下列任何工具，在雲端服務中檢視虛擬機器和角色執行個體的主機名稱。

### <a name="service-configuration-file"></a>服務組態檔
您可以從 Azure 入口網站中服務的 [設定] 刀鋒視窗，下載已部署服務的服務組態檔。 然後，您可以尋找 [角色名稱]元素的 [vmName] 屬性，以查看主機名稱。 請記住，這個主機名稱是做為各個角色執行個體之主機名稱的基底。 例如，如果 [vmName] 是 [webrole]，而該角色有三個執行個體，執行個體的主機名稱會是 *webrole0*、*webrole1* 和 *webrole2*。

### <a name="remote-desktop"></a>遠端桌面
啟用您虛擬機器或角色執行個體的遠端桌面 (Windows)、Windows PowerShell 遠端執行功能 (Windows) 或 SSH (Linux 和 Windows) 連線之後，您可以利用多種方式檢視使用中遠端桌面連線的主機名稱：

* 在命令提示字元或 SSH 終端機中輸入主機名稱。
* 在命令提示字元中輸入 ipconfig /all (僅限 Windows)。
* 在系統設定中檢視電腦名稱 (僅限 Windows)。

### <a name="azure-service-management-rest-api"></a>Azure 服務管理 REST API
從 REST 用戶端，請遵循下列指示：

1. 確定您有連線到 Azure 入口網站的用戶端憑證 若要取得用戶端憑證，請遵循[做法：下載與匯入發行設定與訂用帳戶資訊](/previous-versions/dynamicsnav-2013/dn385850(v=nav.70))中的步驟。 
2. 設定名稱為 x-ms-version，值為 2013-11-01 的標頭項目。
3. 傳送下列格式的要求： HTTPs： \/ /management.core.windows.net/ \<subscrition-id\> /services/hostedservices/ \<service-name\> ？ embed-detail = true
4. 尋找 每個 **RoleInstance** 元素的 **HostName** 元素。

> [!WARNING]
> 您也可以透過檢查 **InternalDnsSuffix** 項目、從遠端桌面工作階段中的命令提示字元執行 ipconfig /all (Windows)，或從 SSH 終端機執行 cat /etc/resolv.conf (Linux)，檢視 REST 呼叫回應中您雲端服務的內部網域尾碼。
> 
> 

## <a name="modifying-a-hostname"></a>修改主機名稱
您可以透過上傳已修改的服務組態檔，或是從遠端桌面工作階段重新命名電腦，來修改任何虛擬機器或角色執行個體的主機名稱。

## <a name="next-steps"></a>後續步驟
[名稱解析 (DNS)](virtual-networks-name-resolution-for-vms-and-role-instances.md)

[Azure 服務組態結構描述 (.cscfg)](/previous-versions/azure/reference/ee758710(v=azure.100))

[Azure 虛擬網路組態結構描述](/previous-versions/azure/reference/jj157100(v=azure.100))

[使用網路組態檔指定 DNS 設定](/previous-versions/azure/virtual-network/virtual-networks-specifying-a-dns-settings-in-a-virtual-network-configuration-file)