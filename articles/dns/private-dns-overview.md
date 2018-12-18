---
title: 將 Azure DNS 用於私人網域 | Microsoft Docs
description: Microsoft Azure 上的私人 DNS 主機服務概觀。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2018
ms.author: victorh
ms.openlocfilehash: 2ab7070a4cf46dae543af8d3e1d688e12ec1eb2a
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173637"
---
# <a name="use-azure-dns-for-private-domains"></a>將 Azure DNS 用於私人網域
「網域名稱系統」(DNS) 會負責將服務名稱轉譯 (或解析) 為其 IP 位址。 Azure DNS 是 DNS 網域的主機服務，採用 Microsoft Azure 基礎結構來提供名稱解析。 除了支援網際網路對向 DNS 網域之外，Azure DNS 現在也支援私人 DNS 網域作為預覽版功能。 
 
Azure DNS 提供一個可靠、安全的 DNS 服務，讓您不必新增自訂 DNS 解決方案，就能管理及解析虛擬網路中的網域名稱。 藉由使用私人 DNS 區域，您就可以使用自己的自訂網域名稱，而不是現今可用的 Azure 提供名稱。 使用自訂網域名稱可協助您量身打造虛擬網路架構，來充分滿足您的組織需求。 它可以為虛擬網路內的虛擬機器 (VM) 及虛擬網路之間提供名稱解析。 此外，您還可以利用水平分割檢視來設定區域名稱，使私人與公用 DNS 區域能夠共用相同名稱。

![DNS 概觀](./media/private-dns-overview/scenario.png)

[!INCLUDE [private-dns-public-preview-notice](../../includes/private-dns-public-preview-notice.md)]

## <a name="benefits"></a>優點

Azure DNS 提供下列優點：

* **不再需要自訂 DNS 解決方案**。 以前，許多客戶建立自訂 DNS 解決方案來管理其虛擬網路中的 DNS 區域。 您現在可以使用原生 Azure 基礎結構來執行 DNS 區域管理，擺脫了建立和管理自訂 DNS 解決方案的負擔。

* **使用所有常見的 DNS 記錄類型**。 Azure DNS 支援 A、AAAA、CNAME、MX、NS、PTR、SOA、SRV 及 TXT 記錄。

* **自動主機名稱記錄管理**。 除了裝載您的自訂 DNS 記錄之外，Azure 還會自動維護所指定虛擬網路中 VM 的主機名稱記錄。 在此案例中，您無須建立自訂 DNS 解決方案或修改應用程式，即可將您使用的網域名稱最佳化。

* **虛擬網路之間的主機名稱解析**。 不同於 Azure 提供的主機名稱，私人 DNS 區域可以在虛擬網路之間共用。 此功能簡化了跨網路及服務探索案例，例如虛擬網路對等互連。

* **熟悉的工具和使用者體驗**。 為了縮短學習曲線，這個新供應項目使用已經成熟的 Azure DNS 工具 (PowerShell、Azure Resource Manager 範本和 REST API)。

* **水平分割 DNS 支援**。 Azure DNS 可讓您建立具有相同名稱，但從虛擬網路內與從公用網際網路會解析成不同答案的區域。 典型的水平分割 DNS 案例是提供一個專用的服務版本以供在您的虛擬網路內使用。

* **所有 Azure 區域均可使用**。 Azure DNS 私人區域已在 Azure 公用雲端中的所有 Azure 區域內正式運作。 


## <a name="capabilities"></a>功能

Azure DNS 提供以下功能：
 
* **對於連結至私人區域成為註冊虛擬網路的單一虛擬網路，為此虛擬網路中的虛擬機器進行自動化註冊**。 虛擬機器將會在私人區域中註冊 (新增) 為指向其私人 IP 位址的 A 記錄。 當註冊虛擬網路中的虛擬機器遭到刪除時，Azure 也會自動從連結的私人區域中移除對應的 DNS 記錄。 

  > [!NOTE]
  > 根據預設，註冊虛擬網路預設也會作為解析虛擬網路，其中區域的 DNS 解析可從註冊虛擬網路中的任何虛擬機器運作。 

* **針對已連結到私人區域成為解析虛擬網路的多個虛擬網路，支援正向 DNS 解析**。 針對跨虛擬網路的 DNS 解析，並沒有虛擬網路彼此間對等互連的明確相依性。 不過，客戶可能會想在其他案例中 (例如 HTTP 流量) 讓虛擬網路對等互連。

* **在虛擬網路範圍內支援反向 DNS 查閱**。 針對指派到私人區域的虛擬網路內私人 IP 位址的反向 DNS 查閱，將會傳回包含主機/記錄名稱以及用區域名稱作為尾碼的 FQDN。 


## <a name="limitations"></a>限制

Azure DNS 受到下列限制：

* 每個私人區域只能有 1 個註冊虛擬網路。
* 每個私人區域最多可以有 10 個解析虛擬網路。
* 特定虛擬網路只可以連結到單一私人區域作為註冊虛擬網路。
* 特定虛擬網路可以連結到最多 10 個私人區域作為解析虛擬網路。
* 如果指定了註冊虛擬網路，在已註冊至私人區域的該虛擬網路中，將無法從 Azure PowerShell 和 AzureCLI API 檢視或擷取來自該虛擬網路虛擬機器的 DNS 記錄，但虛擬機器記錄實際上已註冊，且將會成功解析。
* 反向 DNS 只適用於註冊虛擬網路中的私人 IP 空間。
* 未在私人區域中註冊的私人 IP (例如：在作為解析虛擬網路連結至私人區域的虛擬網路中，虛擬機器的私人 IP) 反向 DNS，將會傳回 internal.cloudapp.net 作為 DNS 尾碼。 但是此尾碼將無法解析。 
* 虛擬網路一開始 (也就是第一次) 連結到私人區域作為註冊或解析虛擬網路時，必須是空的 (也就是沒有任何虛擬機器記錄存在)。 不過，對於未來要作為註冊或解析虛擬網路連結到其他私人區域的虛擬網路，則可為非空白。 
* 此階段不支援條件式轉送 (例如，啟用 Azure 和 OnPrem 網路之間的解析)。 如需客戶可如何透過其他機制實現此案例的詳細資訊，請參閱[虛擬機器與角色執行個體的名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

如需 Azure DNS 中私人區域的常見問題和解答，包括針對特定作業類型可預期的特定 DNS 註冊和解析行為，請參閱[常見問題集](./dns-faq.md#private-dns)。  


## <a name="pricing"></a>價格

私人 DNS 區域在公開預覽期間為免費。 在正式運作期間，此功能將與現有的 Azure DNS 供應項目類似，採用以使用量型計價的模型。 


## <a name="next-steps"></a>後續步驟

了解如何使用 [Azure PowerShell](./private-dns-getstarted-powershell.md) 或 [Azure CLI](./private-dns-getstarted-cli.md)，在 Azure DNS 中建立私人區域。

請參閱一些常見[私人區域案例](./private-dns-scenarios.md)，這些案例可在 Azure DNS 中透過私人區域實現。

如需 Azure DNS 中私人區域的常見問題和解答，包括特定作業類型預期會有的特定行為，請參閱[常見問題集](./dns-faq.md#private-dns)。 

如需了解 DNS 區域和記錄，請瀏覽：[DNS 區域和記錄概觀](dns-zones-records.md)。

了解 Azure 的一些其他重要[網路功能](../networking/networking-overview.md)。 

