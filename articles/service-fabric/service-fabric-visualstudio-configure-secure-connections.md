---
title: 設定安全的 Azure Service Fabric 叢集連線 | Microsoft Docs
description: 了解如何使用 Visual Studio 設定 Azure Service Fabric 叢集支援的安全連線。
services: service-fabric
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: tglee
ms.assetid: 80501867-dd7a-4648-8bd6-d4f26b68402d
ms.service: multiple
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 8/04/2017
ms.author: cawa
ms.openlocfilehash: 8d76a2144234591792359ed8dd4a0779e6a2fc5c
ms.sourcegitcommit: 17fe5fe119bdd82e011f8235283e599931fa671a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2018
ms.locfileid: "42141871"
---
# <a name="configure-secure-connections-to-a-service-fabric-cluster-from-visual-studio"></a>從 Visual Studio 設定對 Service Fabric 叢集的安全連線
了解如何使用 Visual Studio 安全地存取已設定存取控制原則的 Azure Service Fabric 叢集。

## <a name="cluster-connection-types"></a>叢集連線類型
Azure Service Fabric 叢集支援兩種連線：「非安全」連線和「x509 憑證型」安全連線。 (如果是裝載在內部部署環境的 Service Fabric 叢集，則也支援 **Windows** 和 **dSTS** 驗證。)建立叢集時，您必須設定叢集連線類型。 建立之後，即無法變更連線類型。

Visual Studio Service Fabric 工具支援所有用於連線到叢集來進行發佈的驗證類型。 如需有關如何設定安全 Service Fabric 叢集的說明，請參閱 [從 Azure 入口網站設定 Service Fabric 叢集](service-fabric-cluster-creation-via-portal.md) 。

## <a name="configure-cluster-connections-in-publish-profiles"></a>在發行設定檔中設定叢集連接
如果您從 Visual Studio 發佈 Service Fabric 專案，使用 [發佈 Service Fabric 應用程式] 對話方塊來選擇 Azure Service Fabric 叢集。 在 [連線端點] 之下，選取您訂用帳戶下的現有叢集。

![[發佈 Service Fabric 應用程式] 對話方塊用來設定 Service Fabric 連線。][publishdialog]

[發佈 Service Fabric 應用程式] 對話方塊會自動驗證叢集連線。 如果出現提示，登入您的 Azure 帳戶。 如果通過驗證，即表示您的系統已安裝正確的憑證，可安全地連線到叢集，否則即表示您的叢集不安全。 驗證失敗的原因可能是網路問題，或是您的系統尚未正確設定以連線到安全的叢集。

![[發佈 Service Fabric 應用程式] 對話方塊會驗證設定正確的現有 Service Fabric 叢集連線。][selectsfcluster]

### <a name="to-connect-to-a-secure-cluster"></a>連線到安全的叢集
1. 請確定您可以存取目的地叢集所信任的其中一個用戶端憑證。 憑證通常是以「個人資訊交換」(.pfx) 檔案的形式共用。 如需了解如何設定伺服器以授與用戶端存取權，請參閱 [從 Azure 入口網站設定 Service Fabric 叢集](service-fabric-cluster-creation-via-portal.md) 。
2. 安裝受信任的憑證。 若要這樣做，請按兩下.pfx 檔案，或使用 PowerShell 指令碼 Import-PfxCertificate 來匯入憑證。 將憑證安裝至 **Cert:\LocalMachine\My**。 匯入憑證時，可以接受所有預設設定。
3. 在專案的捷徑功能表上選擇 [發行...] 命令以開啟 [發行 Azure 應用程式] 對話方塊，然後選取目標叢集。 此工具會自動解析連線，並將安全連線參數儲存在發行設定檔中。
4. 選擇性︰您可以編輯發行設定檔來指定安全的叢集連線。
   
   由於您正手動編輯「發行設定檔」XML 檔案以指定憑證資訊，因此請務必記下憑證存放區名稱、存放區位置，以及憑證指紋。 您將必須為憑證的存放區名稱和存放區位置提供這些值。 如需詳細資訊，請參閱[做法：擷取憑證的指紋](https://msdn.microsoft.com/library/ms734695\(v=vs.110\).aspx)。
   
   您可以使用 *ClusterConnectionParameters* 參數指定連線到 Service Fabric 叢集時要使用的 PowerShell 參數。 有效的參數是 Connect-ServiceFabricCluster Cmdlet 所接受的任何參數。 如需可用參數的清單，請參閱 [Connect-ServiceFabricCluster](https://docs.microsoft.com/powershell/module/servicefabric/connect-servicefabriccluster) 。
   
   如果要發行至遠端叢集，您需要指定該特定叢集的適當參數。 以下是連線到非安全叢集的範例：
   
   `<ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000" />`
   
   以下是連線到 x509 憑證型安全叢集的範例：
   
   ```xml
   <ClusterConnectionParameters
   ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000"
   X509Credential="true"
   ServerCertThumbprint="0123456789012345678901234567890123456789"
   FindType="FindByThumbprint"
   FindValue="9876543210987654321098765432109876543210"
   StoreLocation="CurrentUser"
   StoreName="My" />
   ```
5. 編輯其他任何必要的設定 (例如升級參數和「應用程式參數」檔案位置)，然後從 Visual Studio 中的 [發佈 Service Fabric 應用程式]  對話方塊發佈您的應用程式。

## <a name="next-steps"></a>後續步驟
如需有關存取 Service Fabric 叢集的詳細資訊，請參閱 [使用 Service Fabric 總管視覺化叢集](service-fabric-visualizing-your-cluster.md)。

<!--Image references-->
[publishdialog]:./media/service-fabric-visualstudio-configure-secure-connections/publishdialog.png
[selectsfcluster]:./media/service-fabric-visualstudio-configure-secure-connections/selectsfcluster.png
