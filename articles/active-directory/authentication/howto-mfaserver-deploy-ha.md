---
title: 針對高可用性設定 Azure MFA Server | Microsoft Docs
description: 在設定中部署多個 Azure Multi-Factor Authentication Server 執行個體以提供高可用性。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 5d3833d3218a4b6252c9591bb67686ddc1c3cdf9
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298569"
---
# <a name="configure-azure-multi-factor-authentication-server-for-high-availability"></a>針對高可用性設定 Azure Multi-Factor Authentication Server

若要透過 Azure Server MFA 部署達到高可用性，您必須部署多部 MFA Server。 本節提供負載平衡設計的相關資訊，該設計可達到 Azure MFS Server 部署中的高可用性目標。

## <a name="mfa-server-overview"></a>MFA Server 概觀

Azure MFA Server 服務架構包含數個元件，如下圖所示：

 ![MFA Server 架構](./media/howto-mfaserver-deploy-ha/mfa-ha-architecture.png)

MFA Server 是已安裝 Azure Multi-Factor Authentication 軟體的 Windows Server。 MFA Server 執行個體必須透過 Azure MFA 服務啟用才能運作。 您可以在內部部署安裝多部 MFA Server。

第一部安裝的 MFA Server 是主要 MFA Server，Azure MFA 服務預設會啟用此伺服器。 主要 MFA Server 具有 PhoneFactor.pfdata 資料庫的可寫入複本。 MFA Server 執行個體的後續安裝稱為從屬項目。 MFA 從屬項目具有 PhoneFactor.pfdata 資料庫的唯讀複寫複本。 MFA Server 使用遠端程序呼叫 (RPC) 複寫資訊。 所有 MFA Server 必須共同加入網域或必須是獨立，才能複寫資訊。

需要雙因素驗證時，MFA 的主要和從屬 MFA Server 會與 MFA 服務通訊。 例如，當使用者嘗試存取需要雙因素驗證的應用程式，使用者會先經過識別提供者驗證，例如 Active Directory (AD)。

向 AD 成功驗證之後，MFA Server 將會與 MFA 服務通訊。 MFA Server 會等候來自 MFA 服務的通知，以允許或拒絕使用者存取應用程式。

如果 MFA 主要伺服器離線，仍可處理驗證，但無法處理需要變更 MFA 資料庫的作業。 (範例包括：新增使用者、自助式 PIN 變更及變更使用者資訊)。

## <a name="deployment"></a>部署

請考慮對 Azure MFA Server 及其相關元件進行負載平衡的下列重點。

* **使用 RADIUS 標準以達到高可用性**。 如果您使用 Azure MFA Server 作為 RADIUS 伺服器，您可能會將一部 MFA Server 設定為主要 RADIUS 驗證目標，並將其他 Azure MFA Server 設定為次要驗證目標。 不過，透過此方法可能無法達到高可用性，因為您必須等候主要驗證目標上的驗證失敗時所發生的逾時結束，才能對次要驗證目標進行驗證。 對 RADIUS 用戶端與 RADIUS 伺服器 (在本例中是以 Azure MFA Server 作為 RADIUS 伺服器) 之間的 RADIUS 流量進行負載平衡會更有效率，因此您可以為 RADIUS 用戶端設定其可指向的單一 URL。
* **需要手動升階 MFA 從屬項目**。 如果主要 Azure MFA Server 離線，次要 Azure MFA Server 會繼續處理 MFA 要求。 不過，在主要 MFA Server 可供使用之前，系統管理員無法新增使用者或修改 MFA 設定，而且使用者無法透過使用者入口網站進行變更。 將 MFA 從屬項目升階為主要角色一律是手動程序。
* **元件的可分性**。 Azure MFA Server 包含數個元件，可安裝在相同的 Windows Server 執行個體上或不同的執行個體上。 這些元件包含使用者入口網站、行動裝置應用程式 Web 服務和 ADFS 配接器 (代理程式)。 此可分性讓您能夠使用 Web 應用程式 Proxy，從周邊網路發佈使用者入口網站和行動裝置應用程式網頁伺服器。 您可以將這類設定新增至設計的整體安全性，如下圖所示。 您也可以在 HA 負載平衡設定中部署 MFA 使用者入口網站和行動裝置應用程式網頁伺服器。

   ![具有周邊網路的 MFA Server](./media/howto-mfaserver-deploy-ha/mfasecurity.png)

* **透過簡訊傳送單次密碼 (OTP) (又稱為單向簡訊) 需要在平衡流量負載時使用黏性工作階段**。 單向簡訊是一個驗證選項，可讓 MFA Server 傳送含有 OTP 的簡訊給使用者。 使用者會在提示字元視窗中輸入 OTP 以完成 MFA 挑戰。 如果您對 Azure MFA Server 進行負載平衡，從使用者收到 OTP 訊息的伺服器必須是服務初始驗證要求的相同伺服器；如果是另一部 MFA Server 收到 OTP 回覆，則驗證挑戰會失敗。 如需詳細資訊，請參閱 [One Time Password over SMS Added to Azure MFA Server](https://blogs.technet.microsoft.com/enterprisemobility/2015/03/02/one-time-password-over-sms-added-to-azure-mfa-server) (透過簡訊傳送單次密碼功能已新增至 Azure MFA Server)。
* **使用者入口網站和行動裝置應用程式 Web 服務的負載平衡部署需要黏性工作階段**。 如果您想要對 MFA 使用者入口網站和行動裝置應用程式 Web 服務進行負載平衡，每個工作階段必須保留在相同的伺服器上。

## <a name="high-availability-deployment"></a>高可用性部署

下圖顯示 Azure MFA 及其元件的完整 HA 負載平衡實作，以及用於參考的 ADFS。

 ![Azure MFA Server HA 實作](./media/howto-mfaserver-deploy-ha/mfa-ha-deployment.png)

注意上圖中對應至編號區域的下列項目。

1. 這兩部 Azure MFA Server (MFA1 和 MFA2) 已經過負載平衡 (mfaapp.contoso.com)，並設定為使用靜態連接埠 (4443) 以複寫 PhoneFactor.pfdata 資料庫。 Web 服務 SDK 會安裝在每部 MFA Server 上，以允許透過 TCP 連接埠 443 與 ADFS 伺服器通訊。 MFA Server 會部署在無狀態負載平衡設定中。 不過，如果您想要透過簡訊使用 OTP，您必須使用具狀態負載平衡。
   ![Azure MFA Server - 應用程式伺服器 HA](./media/howto-mfaserver-deploy-ha/mfaapp.png)

   > [!NOTE]
   > 因為 RPC 使用動態連接埠，所以不建議在 RPC 可能使用的動態連接埠範圍內開啟防火牆。 如果您在 MFA 應用程式伺服器**之間**有防火牆，您應該針對從屬與主要伺服器之間的複寫流量設定要在靜態連接埠上通訊的 MFA Server，並在防火牆上開啟該連接埠。 您可以在 ```HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Positive Networks\PhoneFactor``` 建立稱為 ```Pfsvc_ncan_ip_tcp_port``` 的 DWORD 登錄值，並將該值設定為可用的靜態連接埠，來強制使用靜態連接埠。 一律會由從屬 MFA Server 起始對主要的連線，只有主要才需要靜態連接埠，但由於您可以隨時將從屬項目升階為主要，因此您應該在所有 MFA Server 上設定靜態連接埠。

2. 這兩部使用者入口網站/MFA 行動裝置應用程式伺服器 (MFA-UP-MAS1 和 MFA-UP-MAS2) 會在**具狀態**設定 (mfa.contoso.com) 中進行負載平衡。 如前所述，對 MFA 使用者入口網站和行動裝置應用程式服務進行負載平衡需要黏性工作階段。
   ![Azure MFA Server - 使用者入口網站和行動裝置應用程式服務 HA](./media/howto-mfaserver-deploy-ha/mfaportal.png)
3. ADFS 伺服器陣列已經過負載平衡，並透過周邊網路中的負載平衡 ADFS Proxy 發佈至網際網路。 每部 ADFS 伺服器使用 ADFS 代理程式，利用 TCP 連接埠 443 上的單一負載平衡 URL (mfaapp.contoso.com) 與 Azure MFA Server 通訊。

## <a name="next-steps"></a>後續步驟

* [安裝和設定 Azure MFA Server](howto-mfaserver-deploy.md)
