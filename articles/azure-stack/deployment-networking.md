---
title: Azure Stack 部署網路流量 | Microsoft Docs
description: 本文說明 Azure Stack 部署網路程序的預期步驟。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2018
ms.author: jeffgilb
ms.reviewer: wamota
ms.openlocfilehash: aac8713f94482e0fc809f24786b96d29b23b076a
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43343383"
---
# <a name="about-deployment-network-traffic"></a>關於部署網路流量
要確保部署能夠成功，就必須了解 Azure Stack 部署期間的網路流量流動方式。 本文會帶您看一遍部署程序進行期間預期會有的網路流量，讓您對應該會有的情形有所了解。

下圖顯示部署程序中所涉及的所有元件和連線：

![Azure Stack 部署網路拓撲](media/deployment-networking/figure1.png)

> [!NOTE]
> 本文說明已連線部署的需求，若要了解其他部署方法，請參閱 [Azure Stack 部署連線模型](azure-stack-connection-models.md)。

### <a name="the-deployment-vm"></a>部署 VM
Azure Stack 解決方案中有一組伺服器，可用來裝載 Azure Stack 元件和額外的伺服器 (稱為「硬體生命週期主機 (HLH)」)。 此伺服器可用來部署及管理解決方案的生命週期，並在部署期間裝載部署 VM (DVM)。

## <a name="deployment-requirements"></a>部署需求
在開始部署之前，可由 OEM 驗證一些最低需求，以確保部署能順利完成。 了解這些需求將可協助您準備環境，並確定驗證會成功，這些需求如下：

-   [Certificates](azure-stack-pki-certs.md)
-   [Azure 訂用帳戶](https://azure.microsoft.com/free/?b=17.06)
-   網際網路存取
-   DNS
-   NTP

> [!NOTE]
> 本文重點放在最後三個需求。 如需前兩個需求的詳細資訊，請參閱上面的連結。

## <a name="deployment-network-traffic"></a>部署網路流量
DVM 在設定時所使用的是來自 BMC 網路的 IP，而且 DVM 必須能夠透過網路存取網際網路。 雖然不是所有的 BMC 網路元件都需要外部路由或存取網際網路，但某些利用此網路 IP 的 OEM 特有元件可能有需要。

在部署期間，DVM 會使用訂用帳戶的 Azure 帳戶對 Azure Active Directory (Azure AD) 進行驗證。 若要這樣做，DVM 必須透過網際網路存取特定連接埠和 URL 的清單。 您可以在[發佈端點](azure-stack-integrate-endpoints.md)文件中找到完整清單。 DVM 會利用 DNS 伺服器將內部元件所提出的 DNS 要求轉送至外部 URL。 內部 DNS 會將這些要求轉送至 DNS 轉寄站位址 (您會在部署之前對 OEM 提供此位址)。 NTP 伺服器也是如此，必須有可靠的時間伺服器才能讓所有 Azure Stack 元件維持一致性和時間同步。

DVM 在部署期間只需要對外的網際網路存取權，部署期間不會產生任何對內呼叫。 請注意，DVM 會使用其 IP 作為來源，而且 Azure Stack 不支援 Proxy 設定。 因此，如有必要，您必須提供透明 Proxy 或 NAT 才能存取網際網路。 在部署期間，某些內部元件會開始使用公用 VIP 的外部網路存取網際網路。 部署完成之後，Azure 和 Azure Stack 之間全都會使用公用 VIP 透過外部網路來通訊。

針對 Azure Stack 參數的網路設定中包含存取控制清單 (ACL)，其目的是要限制特定網路來源與目的地之間的流量。 DVM 是唯一沒有存取限制的元件；即使是 HLH 也會受到嚴格限制。 您可以向 OEM 了解自訂選項，以簡化從您網路進行管理和存取的作業。 由於有這些 ACL，請務必避免在部署期間變更 DNS 與 NTP 伺服器位址。 如果這樣做，就必須重新設定解決方案的所有參數。

部署完成後，所提供的 DNS 與 NTP 伺服器位址就會繼續供系統元件直接使用。 例如，如果您在部署完成後檢查 DNS 要求，來源會從 DVM IP 變成外部網路範圍的位址。

部署完成後，系統元件即可使用外部網路透過 SDN 繼續使用提供的 DNS 和 NTP 伺服器位址。 例如，如果您在部署完成後檢查 DNS 要求，來源會從 DVM IP 變成公用 VIP。

## <a name="next-steps"></a>後續步驟
[驗證 Azure 註冊](azure-stack-validate-registration.md)
