---
title: Azure Active Directory Domain Services：部署案例 | Microsoft Docs
description: Azure AD 網域服務的部署案例
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: c5216ec9-4c4f-4b7e-830b-9d70cf176b20
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/21/2017
ms.author: maheshu
ms.openlocfilehash: db2bd855300d93d832a3dd7ca0ce526478824ccc
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502564"
---
# <a name="deployment-scenarios-and-use-cases"></a>部署案例和使用案例
在本節中，我們會看一些受益於 Azure Active Directory (AD) 網域服務的部署案例和使用案例。

## <a name="secure-easy-administration-of-azure-virtual-machines"></a>安全、輕鬆管理 Azure 虛擬機器
您可以使用 Azure Active Directory 網域服務有效率地管理您的 Azure 虛擬機器。 Azure 虛擬機器可以加入至受控網域，如此一來，您便可以使用您公司的 AD 認證來登入。 這種方法有助於避免認證管理的麻煩，例如維護每個 Azure 虛擬機器上的本機系統管理員帳戶。

已加入至受控網域的伺服器虛擬機器也可以使用群組原則來管理和保護其安全。 您可以將所需的安全性基準套用到您的 Azure 虛擬機器，並依據公司的安全性指導方針加以鎖定。 例如，您可以使用群組原則管理功能，來限制在這些虛擬機器上可以啟動的應用程式類型。

![流暢的管理 Azure 虛擬機器](./media/active-directory-domain-services-scenarios/streamlined-vm-administration.png)

當伺服器和其他基礎結構到達生命週期結束時，Contoso 將目前裝載於內部部署的許多應用程式移動至雲端。 其目前的 IT 標準會強制代管公司應用程式的伺服器必須加入網域，並使用群組原則進行管理。 Contoso 的 IT 系統管理員偏好將在 Azure 中部署的虛擬機器加入網域，讓管理更容易。 這麼一來，系統管理員和使用者可以使用他們的公司認證來登入網域。 同時，可以使用群組原則將機器設定為符合所需的安全性基準。 Contoso 偏好不要在 Azure 中部署、監視、管理網域控制站，以便保護 Azure 虛擬機器。 因此，Azure AD 網域服務十分適合這個使用案例。

**部署注意事項**

此部署案例請考慮下列重點︰

* Azure Active Directory Domain Services 所提供的受控網域依預設會提供單一個一般組織單位 (OU) 結構。 位於單一一般 OU 中所有加入網域的機器。 不過，您可以選擇建立自訂的 OU。
* Azure AD 網域服務以每個使用者和電腦容器內建 GPO 的形式支援簡單群組原則。 您可以建立自訂 GPO，並將它們的目標設為自訂 OU。
* Azure AD 網域服務支援基礎 AD 電腦物件結構描述。 您無法延伸電腦物件的結構描述。

## <a name="lift-and-shift-an-on-premises-application-that-uses-ldap-bind-authentication-to-azure-infrastructure-services"></a>將使用 LDAP 繫結驗證的內部部署應用程式「隨即轉移」(lift-and-shift) 至 Azure 基礎結構服務
![LDAP 繫結](./media/active-directory-domain-services-scenarios/ldap-bind.png)

Contoso 有多年前購自 ISV 的一個內部部署應用程式。 ISV 將應用程式目前設為維護模式，並且要求對應用程式進行變更對 Contoso 來說所費不貲。 這個應用程式具有 Web 式前端，它會使用 Web 表單收集使用者認證，然後向公司的 Active Directory 執行 LDAP 繫結來驗證使用者。 Contoso 想要將此應用程式移轉至 Azure 基礎結構服務。 最好的情況是應用程式如常運作，而不需要任何變更。 此外，使用者應該能夠使用其現有的公司認證進行驗證，而不必重新訓練使用者以不同方式執行作業。 換句話說，一般使用者應該不知道應用程式執行所在的位置，而且不會察覺到移轉。

**部署注意事項**

此部署案例請考慮下列重點︰

* 確定應用程式不需要對目錄進行修改/寫入。 不支援 Azure Active Directory Domain Services 所提供的受控網域的 LDAP 寫入存取權。
* 您無法直接對受控網域變更密碼。 一般使用者可以使用 Azure AD 的自助密碼變更機制或對內部部署目錄變更其密碼。 這些變更會自動同步處理，而且可在受控網域中取得。

## <a name="lift-and-shift-an-on-premises-application-that-uses-ldap-read-to-access-the-directory-to-azure-infrastructure-services"></a>將使用 LDAP 讀取存取目錄的內部部署應用程式「隨即轉移」(lift-and-shift) 至 Azure 基礎結構服務
Contoso 有大約十年前開發的內部部署企業營運 (LOB) 應用程式。 此應用程式是目錄感知而且其設計是要搭配 Windows Server AD 使用。 應用程式使用 LDAP (輕量型目錄存取通訊協定) 以從 Active Directory 讀取使用者的相關資訊/屬性。 應用程式不會修改屬性，也不會寫入目錄。 Contoso 想要將此應用程式移轉至 Azure 基礎結構服務，並淘汰目前裝載這個應用程式的過時內部部署硬體。 應用程式無法重新編寫以使用現代目錄 API (例如以 REST 為基礎的 Azure AD 圖形 API)。 因此，在移轉應用程式以在雲端中執行而不修改程式碼或重寫應用程式的情況下，需要「隨即轉移」(lift-and-shift) 選項。

**部署注意事項**

此部署案例請考慮下列重點︰

* 確定應用程式不需要對目錄進行修改/寫入。 不支援 Azure Active Directory Domain Services 所提供的受控網域的 LDAP 寫入存取權。
* 確定應用程式不需要自訂/延伸的 Active Directory 結構描述。 Azure AD 網域服務中不支援結構描述延伸模組。

## <a name="migrate-an-on-premises-service-or-daemon-application-to-azure-infrastructure-services"></a>將內部部署服務或精靈應用程式移轉至 Azure 基礎結構服務
某些應用程式包含多個階層，其中一層必須對後端層 (如資料庫層) 執行經驗證的呼叫。 Active Directory 服務帳戶通常會用於這些使用案例。 您可以隨即轉移這些應用程式到 Azure 基礎結構服務，並將 Azure AD 網域服務用於這類應用程式的身分識別需求。 您可以選擇使用從內部部署目錄同步至 Azure AD 的同一服務帳戶。 或者，您可以先建立自訂 OU，然後在該 OU 中建立另外的服務帳戶，來部署這類應用程式。

![使用 WIA 的服務帳戶](./media/active-directory-domain-services-scenarios/wia-service-account.png)

Contoso 有自訂內建的軟體保存庫應用程式，其中包含 Web 前端、SQL 伺服器和後端 FTP 伺服器。 他們使用服務帳戶的 Windows 整合式驗證來向 FTP 伺服器驗證 Web 前端。 並設定 Web 前端以服務帳戶的身分執行。 後端伺服器則設定為從服務帳戶授權對 Web 前端的存取權。 Contoso 偏好不要在雲端中部署網域控制站虛擬機器來將此應用程式移至 Azure 基礎結構服務。 Contoso 的 IT 系統管理員可以將裝載 Web 前端、SQL 伺服器和 FTP 伺服器的伺服器部署到 Azure 中的虛擬機器。 接著將這些機器加入 Azure AD 網域服務的受控網域。 然後，他們可以在他們的內部部署目錄中使用同一服務帳戶進行應用程式的驗證用途。 此服務帳戶會同步處理至 Azure Active Directory Domain Services 受控的網域，而且可供使用。

**部署注意事項**

此部署案例請考慮下列重點︰

* 確定應用程式使用使用者名稱/密碼進行驗證。 Azure AD 網域服務不支援憑證/智慧卡式驗證。
* 您無法直接對受控網域變更密碼。 一般使用者可以使用 Azure AD 的自助密碼變更機制或對內部部署目錄變更其密碼。 這些變更會自動同步處理，而且可在受控網域中取得。

## <a name="windows-server-remote-desktop-services-deployments-in-azure"></a>Azure 中的 Windows Server 遠端桌面服務部署
您可以使用 Azure AD Domain Services，將受控 AD 網域服務提供給 Azure 中部署的遠端桌面伺服器。

如需此部署案例的詳細資訊，請參閱如何[整合 Azure AD Domain Services 與 RDS 部署](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-azure-adds)。


## <a name="domain-joined-hdinsight-clusters-preview"></a>已加入網域的 HDInsight 叢集 (預覽)
您可以設定 Azure HDInsight 叢集，該叢集已加入 Azure AD Domain Services 受控網域並啟用 Apache Ranger。 透過 Apache Ranger 建立和套用 Hive 原則，並允許使用者 (如資料科學家) 使用 ODBC 型工具 (如 Excel、Tableau 等) 連線到 Hive。Microsoft 正努力將其他工作負載 (例如 HBase、Spark、Storm) 盡快新增至已加入網域的 HDInsight。

如需此部署案例的詳細資訊，請參閱如何[設定已加入網域的 HDInsight 叢集](../hdinsight/domain-joined/apache-domain-joined-configure.md)
