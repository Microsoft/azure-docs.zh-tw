---
title: 在 Azure 入口網站中建立 Service Fabric 叢集 | Microsoft Docs
description: 本文說明如何使用 Azure 入口網站和 Azure 金鑰保存庫在 Azure 中建立安全的 Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: vturecek
ms.assetid: 426c3d13-127a-49eb-a54c-6bde7c87a83b
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/09/2018
ms.author: aljo
ms.openlocfilehash: 5d8f1d2634fd2efd624d1000f2fbc0400af4af11
ms.sourcegitcommit: dc646da9fbefcc06c0e11c6a358724b42abb1438
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39136803"
---
# <a name="create-a-service-fabric-cluster-in-azure-using-the-azure-portal"></a>使用 Azure 入口網站在 Azure 中建立 Service Fabric 叢集
> [!div class="op_single_selector"]
> * [Azure Resource Manager](service-fabric-cluster-creation-via-arm.md)
> * [Azure 入口網站](service-fabric-cluster-creation-via-portal.md)
> 
> 

本逐步指南可逐步引導您使用 Azure 入口網站於 Azure 設定 Service Fabric 叢集 (Linux 或 Windows)。 本指南將逐步引導您完成下列步驟：

* 透過 Azure 入口網站在 Azure 中建立叢集。
* 使用憑證驗證系統管理員。

> [!NOTE]
> 如需更進階的安全性選項 (例如使用 Azure Active Directory 進行使用者驗證及設定應用程式安全性的憑證)，請參閱[使用 Azure Resource Manager 建立您的叢集][create-cluster-arm]。
> 
> 

## <a name="cluster-security"></a>叢集安全性 
憑證是在 Service Fabric 中用來提供驗證與加密，以保護叢集和其應用程式的各個層面。 如需如何在 Service Fabric 中使用憑證的詳細資訊，請參閱 [Service Fabric 叢集安全性案例][service-fabric-cluster-security]。

如果這是您第一次為測試工作負載而建立 Service Fabric 叢集或部署叢集，您可以跳到下一節 (**在 Azure 入口網站中建立叢集**)，並讓系統產生執行測試工作負載之叢集所需的憑證。 如果您準備為生產工作負載設定叢集，請繼續閱讀。

#### <a name="cluster-and-server-certificate-required"></a>叢集和伺服器憑證 (必要)
需要此憑證來保護叢集安全及防止未經授權存取叢集。 它會透過幾種方式提供叢集安全性：

* **叢集驗證：** 驗證叢集同盟的節點對節點通訊。 只有可使用此憑證提供其身分識別的節點可以加入叢集。
* **伺服器驗證：** 向管理用戶端驗證叢集管理端點，管理用戶端就能知道它正在交談的對象是真正的叢集。 此憑證也會為 HTTPS 管理 API，以及為透過 HTTPS 使用的 Service Fabric Explorer 提供 SSL。

為用於這些用途，憑證必須符合下列要求：

* 憑證必須包含私密金鑰。
* 憑證必須是為了進行金鑰交換而建立，且可匯出成個人資訊交換檔 (.pfx)。
* 憑證的**主體名稱必須符合**用來存取 Service Fabric 叢集的網域。 這是必要的，以便為叢集的 HTTPS 管理端點和 Service Fabric Explorer 提供 SSL。 您無法向憑證授權單位 (CA) 取得 `.cloudapp.azure.com` 網域的 SSL 憑證。 為您的叢集取得自訂網域名稱。 當您向 CA 要求憑證時，憑證的主體名稱必須符合用於您叢集的自訂網域名稱。

#### <a name="client-authentication-certificates"></a>用戶端驗證憑證
其他用戶端憑證會驗證系統管理員以執行叢集管理工作。 Service Fabric 有兩個存取層級：[系統管理員] 和 [唯讀使用者]。 您至少應使用一個單一憑證以用於進行系統管理存取。 若要進行其他使用者層級存取，則必須提供個別憑證。 如需存取角色的詳細資訊，請參閱[角色型存取控制 (適用於 Service Fabric 用戶端)][service-fabric-cluster-security-roles]。

若要使用 Service Fabric，您並不需要將用戶端驗證憑證上傳至金鑰保存庫。 這些憑證只需要提供給獲得授權來管理叢集的使用者。 

> [!NOTE]
> 建議使用 Azure Active Directory 驗證用戶端以執行叢集管理作業。 若要使用 Azure Active Directory，您必須[使用 Azure Resource Manager 建立叢集][create-cluster-arm]。
> 
> 

#### <a name="application-certificates-optional"></a>應用程式憑證 (選用)
您可以針對應用程式安全性目的，在叢集上安裝任何數目的其他憑證。 在建立您的叢集之前，請考量需要在節點上安裝憑證的應用程式安全性案例，例如：

* 加密和解密應用程式組態值
* 在複寫期間跨節點加密資料 

[透過 Azure 入口網站建立叢集](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/service-fabric/service-fabric-cluster-creation-via-portal.md)時無法設定應用程式憑證。 若要在建立叢集時設定應用程式憑證，您必須[使用 Azure Resource Manager 建立叢集][create-cluster-arm]。 您也可以在建立叢集之後將應用程式憑證新增到叢集。

## <a name="create-cluster-in-the-azure-portal"></a>在 Azure 入口網站中建立叢集

若想建立生產叢集以滿足應用程式需求，您需要先行規劃，為了幫助您解決這個問題，我們強烈建議您閱讀並了解 [Service Fabric 叢集規劃考量][service-fabric-cluster-capacity]文件。 

### <a name="search-for-the-service-fabric-cluster-resource"></a>搜尋 Service Fabric 叢集資源
![在 Azure 入口網站上搜尋 Service Fabric 叢集範本。][SearchforServiceFabricClusterTemplate]

1. 登入 [Azure 入口網站][azure-portal]。
2. 按一下 [建立資源] 以新增新的資源範本。 在 [全部內容] 下方的 [Marketplace] 中搜尋 Service Fabric 叢集範本。
3. 選取清單中的 [Service Fabric 叢集]  。
4. 瀏覽至 [Service Fabric 叢集] 刀鋒視窗，按一下 [建立]，
5. [建立 Service Fabric 叢集] 刀鋒視窗包含下列四個步驟：

#### <a name="1-basics"></a>1.基本概念
![建立新資源群組的螢幕擷取畫面。][CreateRG]

您必須在 [基本] 刀鋒視窗中提供您叢集的基本詳細資料。

1. 輸入您的叢集名稱。
2. 輸入 VM 遠端桌面的 [使用者名稱] 和 [密碼]。
3. 請務必選取您要部署叢集的 [訂用帳戶]  ，尤其是在您擁有多個訂用帳戶時。
4. 建立 **新的資源群組**。 最好讓它與叢集同名，因為這有助於稍後尋找它們，尤其是當您嘗試變更您的部署及刪除您的叢集時，特別有用。
   
   > [!NOTE]
   > 雖然您可以決定使用現有的資源群組，但最好還是建立新的資源群組。 這可讓您輕鬆地刪除叢集及其所使用的所有資源。
   > 
   > 
5. 選取您要在其中建立叢集的 [區域]  。 如果您打算使用您已上傳至金鑰保存庫的現有憑證，您必須使用與金鑰保存庫所在位置相同的區域。 

#### <a name="2-cluster-configuration"></a>2.叢集組態
![建立節點類型][CreateNodeType]

設定您的叢集節點。 可用來定義定義 VM 的大小、VM 的數目，以及 VM 的屬性。 您的叢集可以有多個節點類型，但主要節點類型 (您在入口網站定義的第一個節點類型) 必須至少有 5 個 VM。這是 Service Fabric 系統服務放置所在的節點類型。 請勿設定 [放置屬性]，因為會自動新增 "NodeTypeName" 預設放置屬性。

> [!NOTE]
> 多個節點類型的常見案例是包含前端服務和後端服務的應用程式。 您想要將「前端」服務放在連接埠對網際網路開放的較小型 VM (D2_V2 等 VM 大小) 上，並將「後端」服務放在連接埠不對網際網路開放的較大型 VM (D3_V2、D6_V2、D15_V2 等 VM 大小) 上。
> 
> 

1. 選擇節點類型的名稱 (1 到 12 個字元，只能包含字母和數字)。
2. 主要節點類型的 VM **大小**下限取決於您為叢集選擇的**持久性**層級。 持久性層級的預設值為 Bronze。 如需關於持久性的詳細資訊，請參閱[如何選擇 Service Fabric 叢集持久性][service-fabric-cluster-durability]。
3. 選取 VM 大小。 D 系列 VM 擁有 SSD 磁碟機，且強烈建議用於具狀態應用程式。 請勿使用任何只有部分核心或可用磁碟容量少於 10 GB 的 VM SKU。 如需有關選取 VM 大小的說明，請參閱 [Service Fabric 叢集規劃考量文件][service-fabric-cluster-capacity]。
4. 選擇節點類型的 VM 數目。 您可以在稍後將節點類型中的 VM 數目相應增加或相應減少，但在主要節點類型上，生產工作負載的數目下限是五個。 其他節點類型最少可以有一個 VM。主要節點類型之 VM 的**數目**下限可決定叢集的**可靠性**。  
5. **單一節點叢集與三個節點叢集**：僅供測試使用。 這些節點叢集不支援任何執行中的生產工作負載。
6. 設定自訂端點。 此欄位可讓您輸入以逗號區隔的連接埠清單，您可以透過 Azure Load Balancer 針對您的應用程式向公用網際網路公開這些連接埠。 例如，如果您計劃對您的叢集部署 Web 應用程式，請在這裡輸入「80」來允許連接埠 80 的流量進入您的叢集。 如需端點的詳細資訊，請參閱[與應用程式通訊][service-fabric-connect-and-communicate-with-services]
7. 設定叢集**診斷**。 預設會在您的叢集上啟用診斷功能，以協助排解疑難問題。 如果您要停用診斷，請將其 [狀態] 切換至 [關閉]。 **不**建議將診斷關閉。 如果您已經建立了 Application Insights 專案，那麼請提供其密鑰，以便將應用程式追蹤透過路由方式傳送至該專案。
8. 選取您想要為叢集設定的 Fabric 升級模式。 如果您要讓系統自動挑選最新可用的版本，並嘗試將叢集升級到此版本，請選取 [自動] 。 如果您想要選擇支援的版本，將模式設定為 [手動] 。 如需 Fabric 升級模式的詳細資訊，請參閱 [service-fabric-cluster-upgrade 文件。][service-fabric-cluster-upgrade]

> [!NOTE]
> 我們支援的叢集限於執行支援的 Service Fabric 版本。 如果選取 [手動]  模式，您必須負責將叢集升級到支援的版本。 > 
> 

#### <a name="3-security"></a>3.安全性
![Azure 入口網站中安全性組態的螢幕擷取畫面。][BasicSecurityConfigs]

為了讓您輕鬆設定安全的測試叢集，我們已提供 [基本] 選項。 如果您已擁有憑證，並且已將憑證上傳至金鑰保存庫 (且已啟用金鑰保存庫以供部署)，則請使用 [自訂] 選項

##### <a name="basic-option"></a>基本選項
請遵循畫面指示新增或重複使用現有的金鑰保存庫，並新增憑證。 新增憑證為同步的程序，所以您必須等待憑證建立完成。


請不要離開畫面，直到前述程序完成為止。

![CreateKeyVault]

現在憑證已新增至金鑰保存庫中，您可能會看到下列畫面，提示您編輯金鑰保存庫的存取原則。 按一下 [編輯存取原則]。 按鈕。

![CreateKeyVault2]

按一下進階存取原則，並啟用虛擬機器的存取權以進行部署。 建議您也啟用範本部署。 選取完畢時，請務必按一下 [儲存] 按鈕，並關閉 [存取原則] 窗格。

![CreateKeyVault3]

現在，您已準備好繼續進行其餘的叢集建立程序。

![CreateKeyVault4]

##### <a name="custom-option"></a>自訂選項
如果您已在 [基本] 選項中執行步驟，請跳過本節。

![SecurityCustomOption]

您需要 CertificateThumbprint、SourceVault 和 CertificateURL 資訊，以完成安全性頁面。 如果您不方便取得，請在另一個瀏覽器視窗中開啟，並執行下列動作


1. 瀏覽至您的金鑰保存庫，並選取憑證。 
2. 選取 [屬性] 索引標籤，並將「資源 ID」複製到其他瀏覽器視窗上的「來源金鑰保存庫」 

    ![CertInfo0]

3. 現在，請選取 [憑證] 索引標籤。
4. 按一下憑證指紋，這將帶您前往 [版本] 頁面。
5. 按一下您在目前版本下看到的 Guid。

    ![CertInfo1]

6. 您現在應該在如下所示的畫面中。 將「指紋」複製到其他瀏覽器視窗上的「憑證指紋」
7. 將「祕密識別碼」資訊複製到其他瀏覽器視窗上的「憑證 URL」。


![CertInfo2]


選取 [設定進階設定] 核取方塊來輸入**系統管理用戶端**和**唯讀用戶端**的用戶端憑證。 在這些欄位中，輸入系統管理用戶端憑證的指紋和唯讀使用者用戶端憑證的指紋 (如果適用)。 當系統管理員嘗試連線叢集時，只有在他們的憑證指紋和這裡輸入的指紋值相符時，才會被授與存取權。  

#### <a name="4-summary"></a>4.總結

現在您已經準備好部署叢集。 在進行作業前，請先下載憑證，並在大型藍色資訊方塊中查看連結。 請務必將憑證保存在安全的地方。 您需要將憑證連線到叢集。 由於您下載的憑證沒有密碼，建議您新增密碼。

若要完成叢集建立作業，請按一下 [建立]。 您也可以選擇性下載此範本。 

![總結]

您可以在通知功能中看到叢集的建立進度。 (請按一下畫面右上角狀態列附近的鈴噹圖示。)如果您在建立叢集時按了 [釘選到「開始面板」]，您會看到 [部署 Service Fabric 叢集] 已釘選到 [開始] 面板。

若要使用 Powershell 或 CLI 在您的叢集上執行管理作業，您需要連接至叢集，請參閱如何[連接至您的叢集](service-fabric-connect-to-secure-cluster.md)，了解更多資訊。

### <a name="view-your-cluster-status"></a>檢視叢集狀態
![顯示叢集詳細資料的儀表板螢幕擷取畫面。][ClusterDashboard]

建立叢集之後，您就可以在入口網站檢查您的叢集：

1. 移至 [瀏覽]，然後按一下 [Service Fabric 叢集]。
2. 找出您的叢集，然後按一下它。
3. 現在儀表板會顯示叢集的詳細資料，包括叢集的公用端點和 Service Fabric Explorer 的連結。

叢集之儀表板刀鋒視窗上的 [節點監視器]  區段會指出健康狀態良好和不良的 VM 數目。 如需進一步了解叢集健康狀態，請參閱 [Service Fabric 健康狀態模型簡介][service-fabric-health-introduction]。

> [!NOTE]
> Service Fabric 叢集需要有一定數量的節點保持運作中，以維護可用性並維持狀態 - 稱為「維持仲裁」。 因此，除非您已先執行[狀態的完整備份][service-fabric-reliable-services-backup-restore]，否則關閉叢集中的所有電腦通常並不安全。
> 
> 

## <a name="remote-connect-to-a-virtual-machine-scale-set-instance-or-a-cluster-node"></a>遠端連接到虛擬機器擴展集執行個體或叢集節點
您在叢集中指定的每個 NodeTypes 都會形成虛擬機器擴展集。 <!--See [Remote connect to a Virtual Machine Scale Set instance][remote-connect-to-a-vm-scale-set] for details. -->

## <a name="next-steps"></a>後續步驟
此時，您擁有一個使用憑證來管理驗證的安全叢集。 接下來，請[連線到您的叢集](service-fabric-connect-to-secure-cluster.md)並了解如何[管理應用程式密碼](service-fabric-application-secret-management.md)。  同時，了解 [Service Fabric 支援選項](service-fabric-support.md)。

<!-- Links -->
[azure-powershell]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[service-fabric-rp-helpers]: https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers
[azure-portal]: https://portal.azure.com/
[key-vault-get-started]: ../key-vault/key-vault-get-started.md
[create-cluster-arm]: service-fabric-cluster-creation-via-arm.md
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-cluster-security-roles]: service-fabric-cluster-security-roles.md
[service-fabric-cluster-capacity]: service-fabric-cluster-capacity.md
[service-fabric-cluster-durability]: service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster
[service-fabric-connect-and-communicate-with-services]: service-fabric-connect-and-communicate-with-services.md
[service-fabric-health-introduction]: service-fabric-health-introduction.md
[service-fabric-reliable-services-backup-restore]: service-fabric-reliable-services-backup-restore.md
<!--[remote-connect-to-a-vm-scale-set]: service-fabric-cluster-nodetypes.md#remote-connect-to-a-virtual-machine-scale-set-instance-or-a-cluster-node --> [remote-connect-to-a-vm-scale-set]: service-fabric-cluster-nodetypes.md [service-fabric-cluster-upgrade]: service-fabric-cluster-upgrade.mdd

<!--Image references-->
[SearchforServiceFabricClusterTemplate]: ./media/service-fabric-cluster-creation-via-portal/SearchforServiceFabricClusterTemplate.png
[CreateRG]: ./media/service-fabric-cluster-creation-via-portal/CreateRG.png
[CreateNodeType]: ./media/service-fabric-cluster-creation-via-portal/NodeType.png
[BasicSecurityConfigs]: ./media/service-fabric-cluster-creation-via-portal/BasicSecurityConfigs.PNG
[CreateKeyVault]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault.PNG
[CreateKeyVault2]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault2.PNG
[CreateKeyVault3]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault3.PNG
[CreateKeyVault4]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault4.PNG
[CertInfo0]: ./media/service-fabric-cluster-creation-via-portal/CertInfo0.PNG
[CertInfo1]: ./media/service-fabric-cluster-creation-via-portal/CertInfo1.PNG
[CertInfo2]: ./media/service-fabric-cluster-creation-via-portal/CertInfo2.PNG
[SecurityCustomOption]: ./media/service-fabric-cluster-creation-via-portal/SecurityCustomOption.PNG
[DownloadCert]: ./media/service-fabric-cluster-creation-via-portal/DownloadCert.PNG
[總結]: ./media/service-fabric-cluster-creation-via-portal/Summary.PNG
[SecurityConfigs]: ./media/service-fabric-cluster-creation-via-portal/SecurityConfigs.png
[Notifications]: ./media/service-fabric-cluster-creation-via-portal/notifications.png
[ClusterDashboard]: ./media/service-fabric-cluster-creation-via-portal/ClusterDashboard.png
[cluster-security-cert-installation]: ./media/service-fabric-cluster-creation-via-arm/cluster-security-cert-installation.png
