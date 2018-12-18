---
title: 使用企業安全性套件的 Azure HDInsight 架構
description: 了解如何使用企業安全性套件規劃 HDInsight 安全性。
services: hdinsight
ms.service: hdinsight
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 975a4f7b15d1e1c13767cd7026e961e9d4227603
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998914"
---
# <a name="use-enterprise-security-package-in-hdinsight"></a>在 HDInsight 中使用企業安全性套件

標準的 Azure HDInsight 叢集是一個單一使用者叢集。 它適合大部分以小型應用程式團隊建置大型資料工作負載的公司。 每個使用者都可依需求建立專用的不同叢集，並且在不再需要時加以終結。 

許多企業都朝向一種模型發展，亦即由 IT 小組管理叢集，而由多個應用程式小組共用叢集。 這些較大的企業需要在 Azure HDInsight 中針對每個叢集的多使用者存取。

在管理方面，HDInsight 依賴廣受使用的識別提供者 – Active Directory。 藉由 HDInsight 與 [Azure Active Directory Domain Services (Azure AD DS)](../../active-directory-domain-services/active-directory-ds-overview.md) 的整合，您可以使用網域認證來存取叢集。 

HDInsight 中的虛擬機器 (VM) 會加入您提供的網域。 因此，經過驗證的使用者都能夠順暢地使用在 HDInsight 上執行的所有服務 (Ambari、Hive 伺服器、Ranger、Spark Thrift 伺服器等等)。 接著，系統管理員可使用 Apache Ranger 來建立強式授權原則，為叢集中的資源提供角色型存取控制。


## <a name="integrate-hdinsight-with-active-directory"></a>整合 HDInsight 與 Active Directory

開放原始碼 Hadoop 須依賴 Kerberos 來提供驗證和安全性。 因此，有企業安全性套件 (ESP) 的 HDInsight 叢集節點都會加入由 Azure AD DS 管理的網域。 針對叢集上的 Hadoop 元件，會設定 Kerberos 安全性。 

系統會自動為每個 Hadoop 元件建立一個服務主體。 此外，針對每部已加入網域的電腦也會建立對應的電腦主體。 若要儲存這些服務和電腦主體，您必須在網域控制站 (Azure AD DS) 內提供組織單位 (OU)，用來放置這些主體。 

總括而言，您必須設定一個具備下列條件的環境：

- Active Directory 網域 (由 Azure AD DS 管理)。
- 在 Azure AD DS 中啟用安全 LDAP (LDAPS)。
- Azure AD DS 虛擬網路和 HDInsight 虛擬網路兩者之間有適當的網路連線能力 (如果您選擇將兩者的虛擬網路分開)。 HDInsight 虛擬網路內的 VM 應可透過虛擬網路對等互連來看到 Azure AD DS。 如果 HDInsight 和 Azure AD DS 部署在相同的虛擬網路中，則會自動提供連線，而不需要進一步的動作。
- [建立在 Azure AD DS 上](../../active-directory-domain-services/active-directory-ds-admin-guide-create-ou.md)的 OU。
- 一個具有下列權限的服務帳戶：
    - 在 OU 中建立服務主體。
    - 將機器加入網域並在 OU 中建立機器主體。

以下螢幕擷取畫面顯示在 contoso.com 中建立的 OU。 它也會顯示一些服務主體和電腦主體。

![有 ESP 的 HDInsight 叢集所用的組織單位](./media/apache-domain-joined-architecture/hdinsight-domain-joined-ou.png).

## <a name="set-up-different-domain-controllers"></a>設定不同的網域控制站
HDInsight 目前僅支援以 Azure AD DS 作為主要網域控制站，讓叢集使用 Kerberos 通訊。 但您仍然可以使用其他複雜的 Active Directory 設定，只要這類設定能夠讓 Azure AD DS 進行 HDInsight 存取即可。

### <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services
[Azure AD DS](../../active-directory-domain-services/active-directory-ds-overview.md) 提供與 Windows Server Active Directory 完全相容的受控網域。 Microsoft 會負責管理、修補及監視高可用性 (HA) 設定中的網域。 您可以放心地部署叢集，不必擔心網域控制站的維護問題。 

使用者、群組和密碼都會從 Azure Active Directory (Azure AD) 中同步。 從您的 Azure AD 執行個體單向同步到 Azure AD DS 可讓使用者利用相同的公司認證登入叢集。 

如需詳細資訊，請參閱[使用 Azure AD DS 設定有 ESP 的 HDInsight 叢集](./apache-domain-joined-configure-using-azure-adds.md)。

### <a name="on-premises-active-directory-or-active-directory-on-iaas-vms"></a>內部部署 Active Directory 或在 IaaS VM 上的 Active Directory

如果您有內部部署的 Active Directory 執行個體或網域上有更複雜的 Active Directory 設定，則可以使用 Azure AD Connect 來將那些身分識別同步到 Azure AD。 之後，您可以在該 Active Directory 租用戶上啟用 Azure AD DS。 

因為 Kerberos 依賴密碼雜湊，您必須[對 Azure AD DS 啟用密碼雜湊同步](../../active-directory-domain-services/active-directory-ds-getting-started-password-sync.md)。 如果您使用與 Active Directory 同盟服務 (AD FS) 同盟，則可以視需要設定密碼雜湊同步來作為 AD FS 基礎結構失敗時的備用方式。 如需詳細資訊，請參閱[透過 Azure AD Connect 同步啟用密碼雜湊同步](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md)。 

對於有 ESP 的 HDInsight 叢集，不支援單獨使用內部部署 Active Directory 或 IaaS VM 上的 Active Directory，必須同時使用 Azure AD 和 Azure AD DS 的設定。

## <a name="next-steps"></a>後續步驟

* [設定有 ESP 的 HDInsight 叢集](apache-domain-joined-configure-using-azure-adds.md)
* [為有 ESP 的 HDInsight 叢集設定 Hive 原則](apache-domain-joined-run-hive.md)
* [管理有 ESP 的 HDInsight 叢集](apache-domain-joined-manage.md) 
