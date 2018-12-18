---
title: 教學課程：在 AWS 上建立 Service Fabric 叢集的基礎結構 - Azure Service Fabric |Microsoft Docs
description: 在本教學課程中，您將了解如何設定用來執行 Service Fabric 叢集的 AWS 基礎結構。
services: service-fabric
documentationcenter: .net
author: david-stanford
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 05/11/2018
ms.author: dastanfo
ms.custom: mvc
ms.openlocfilehash: 6b7d2223d33abb429ab5f59b14c80d43c70598dc
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34209645"
---
# <a name="tutorial-create-aws-infrastructure-to-host-a-service-fabric-cluster"></a>教學課程：建立用來裝載 Service Fabric 叢集的 AWS 基礎結構

Service Fabric 獨立叢集讓您能夠選擇自己的環境，並且在 Service Fabric 所採用的「任何 OS、任何雲端」方法中建立叢集。 在此教學課程系列中，您會建立裝載於 AWS 的獨立叢集，並在其中安裝應用程式。

本教學課程是一個系列的第一部分。 在本文中，您會產生要裝載 Service Fabric 的獨立叢集所需的 AWS 資源。 在後續的文章中，您將必須安裝 Service Fabric 獨立套件、將範例應用程式安裝到您的叢集中，最後清除您的叢集。

在系列的第一部分中，您將了解如何：

> [!div class="checklist"]
> * 建立一組 EC2 執行個體
> * 修改安全性群組
> * 登入其中一個執行個體
> * 準備 Service Fabric 的執行個體

## <a name="prerequisites"></a>先決條件

若要完成此教學課程，您需要 AWS 帳戶。  如果您還沒有帳戶，請移至 [AWS 主控台](https://aws.amazon.com/)建立一個。

## <a name="create-ec2-instances"></a>建立 EC2 執行個體

登入 AWS 主控台 > 在搜尋方塊中輸入 **EC2** > **雲端中的 EC2 虛擬伺服器**

![AWS 主控台搜尋][aws-console]

選取 [啟動執行個體]，然後在下一個畫面上，選擇 Microsoft Windows Server 2016 Base 旁的 [選取]。

![EC2 執行個體選取][aws-ec2instance]

依序選取 [t2.medium]、[下一步：設定執行個體詳細資料]，然後在下一個畫面上將執行個體數目變更為 `3`，再選取 [進階詳細資料] 以展開該區段。

若要一併連接 Service Fabric 中的虛擬機器，裝載基礎結構的 VM 必須具有相同的認證。  有兩個常見的方法可用來取得一致的認證：將虛擬機器全部加入相同的網域，或在每個 VM 上設定相同的系統管理員密碼。  在本教學課程中，您會透過使用者資料指令碼來設定 EC2 執行個體，使其全部具有相同的密碼。  在生產環境中，將主機加入 Windows 網域，會更加安全。

在主控台上，將下列指令碼輸入使用者資料欄位中：

```powershell
<powershell>
$user = [adsi]"WinNT://localhost/Administrator,user"
$user.SetPassword("serv1ceF@bricP@ssword")
$user.SetInfo()
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
New-NetFirewallRule -DisplayName "Service Fabric Ports" -Direction Inbound -Action Allow -RemoteAddress LocalSubnet -Protocol TCP -LocalPort 135, 137-139, 445
</powershell>
```

輸入 PowerShell 指令碼後，請選取 [檢閱並啟動]

![EC2 檢閱並啟動][aws-ec2configure2]

在檢閱畫面上，選取 [啟動]。  接著，將下拉式清單變更為 [繼續作業而不使用金鑰組]，然後選取指出您知道密碼的核取方塊。

![AWS 金鑰組選取][aws-keypair]

最後，選取 [啟動執行個體]，然後選取 [檢視執行個體]。  您所建立的 Service Fabric 叢集已具有基礎，現在，您必須將一些最終的組態新增至執行個體本身，使其做好設定 Service Fabric 的準備。

## <a name="modify-the-security-group"></a>修改安全性群組

Service Fabric 需要在叢集中的主機之間開啟多個連接埠。 若要在 AWS 基礎結構中開啟這些連接埠，請選取您所建立的其中一個執行個體。 然後，選取安全性群組的名稱，例如 **launch-wizard-1**。 現在，選取 [輸入] 索引標籤。

若要避免對外界開啟這些連接埠，您應改為僅針對相同安全性群組中的主機開啟連接埠。 記下安全性群組識別碼 (在此範例中為 **sg-c4fb1eba**)。  然後，選取 [編輯]。

接下來，將四項用於服務相依性的規則新增至安全性群組，然後再為 Service Fabric 本身新增三項規則。 第一項規則是允許 ICMP 流量，以進行基本連線能力檢查。 其他規則會開啟必要的連接埠，以啟用 SMB 與遠端登錄。

針對第一項規則選取 [新增規則]，然後從下拉式功能表中選取 [所有 ICMP - IPv4]。 選取自訂旁的輸入方塊，然後輸入前述的安全性群組識別碼。

針對後三項相依性，您必須執行類似的程序。  選取 [新增規則]，從下拉式清單中選取 [自訂 TCP 規則]，然後在連接埠範圍中為每項規則輸入 `135`、`137-139` 或 `445`。 最後，在 [來源] 方塊中輸入您的安全性群組識別碼。

![安全性群組連接埠][aws-ec2securityports]

現在，相依性所需的連接埠已開啟，接著您必須對 Service Fabric 本身用來進行通訊的連接埠執行相同的動作。 選取 [新增規則]，從下拉式清單中選取 [自訂 TCP 規則]，在連接埠範圍中輸入 `20001-20031`，然後在 [來源] 方塊中輸入安全性群組。

接著，新增暫時連接埠範圍的規則。  選取 [新增規則]，從下拉式清單中選取 [自訂 TCP 規則]，然後在連接埠範圍中輸入 `20606-20861`。 最後，在 [來源] 方塊中輸入您的安全性群組識別碼。

針對 Service Fabric 的最後兩項規則，請對外界開啟連接埠，以便您從個人電腦管理 Service Fabric 叢集。 選取 [新增規則]，從下拉式清單中選取 [自訂 TCP 規則]，然後在連接埠範圍中輸入 `19000-19003` 或 `19080-19081`，然後將 [來源] 下拉式清單變更為 [任何位置]。

最後，我們只需要開啟連接埠 8080，以便您在應用程式部署後加以檢視。 選取 [新增規則]，從下拉式清單中選取 [自訂 TCP 規則]，然後在連接埠範圍中輸入 `8080`，然後將 [來源] 下拉式清單變更為 [任何位置]。

所有規則現在都已輸入。 選取 [ **儲存**]。

## <a name="connect-to-an-instance-and-validate-connectivity"></a>連線至執行個體並驗證連線能力

在 [安全性群組] 索引標籤中，從左側功能表選取 [執行個體]。  選取您已建立的每個執行個體，並記下其私人 IP 位址 (在下方的範例中會 `172.31.21.141` 和 `172.31.20.163`)。

在記下所有的 IP 位址後，請選取其中一個要連線的執行個體、以滑鼠右鍵按一下該執行個體，然後選取 [連接]。  在此處，您可以為此執行個體下載 RDP 檔案。  選取 [下載遠端桌面檔案]，然後開啟已下載的檔案，以建立對此執行個體的遠端桌面連線 (RDP)。  在出現提示時，請輸入您的密碼 `serv1ceF@bricP@ssword`。

![下載遠端桌面檔案][aws-rdp]

在成功連線至您的執行個體後，請驗證您可以在這些執行個體之間連線和共用檔案。  您已收集所有執行個體的 IP 位址，請選取一個您目前未連線的執行個體。 移至 [開始]、輸入 `cmd`，然後選取 [命令提示字元]。

在前述範例中，我們已建立下列 IP 位址的 RDP 連線：172.31.21.141。 接著會對另一個 IP 位址執行所有連線能力測試：172.31.20.163。

若要驗證基本連線可運作，請使用 ping 命令。

```
ping 172.31.20.163
```

如果您的輸出看起來像是 `Reply from 172.31.20.163: bytes=32 time<1ms TTL=128` 重複四次，表示執行個體之間的連線運作正常。  此時，請使用下列命令驗證您的 SMB 共用可運作：

```
net use * \\172.31.20.163\c$
```

它應傳回 `Drive Z: is now connected to \\172.31.20.163\c$.` 這樣的輸出。

## <a name="prep-instances-for-service-fabric"></a>準備 Service Fabric 的執行個體

如果您是從頭建立的，則必須採取一些額外的步驟。  也就是說，您必須驗證遠端登錄已執行、啟用 SMB，並開啟為 SMB 和遠端登錄必要的連接埠。

為了簡化作業，您在透過使用者資料指令碼進行執行個體的啟動程序時，內嵌了所有相關工作。

為了啟用 SMB，您使用了下列 PowerShell 命令：

```powershell
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
```

若要在防火牆中開啟連接埠，應使用下列 PowerShell 命令：

```powershell
New-NetFirewallRule -DisplayName "Service Fabric Ports" -Direction Inbound -Action Allow -RemoteAddress LocalSubnet -Protocol TCP -LocalPort 135, 137-139, 445
```

## <a name="next-steps"></a>後續步驟

在此系列的第一個單元中，您已了解如何啟動三個 EC2 執行個體並加以設定，以進行 Service Fabric 安裝：

> [!div class="checklist"]
> * 建立一組 EC2 執行個體
> * 修改安全性群組
> * 登入其中一個執行個體
> * 準備 Service Fabric 的執行個體

請繼續進行此系列的第二個部分，以在叢集上設定 Service Fabric。

> [!div class="nextstepaction"]
> [安裝 Service Fabric](service-fabric-tutorial-standalone-create-service-fabric-cluster.md)

<!-- IMAGES -->
[aws-console]: ./media/service-fabric-tutorial-standalone-cluster/aws-console.png
[aws-ec2instance]: ./media/service-fabric-tutorial-standalone-cluster/aws-ec2instance.png
[aws-ec2configure2]: ./media/service-fabric-tutorial-standalone-cluster/aws-ec2configure2.png
[aws-rdp]: ./media/service-fabric-tutorial-standalone-cluster/aws-rdp.png
[aws-ec2securityports]: ./media/service-fabric-tutorial-standalone-cluster/aws-ec2securityports.png
[aws-keypair]: ./media/service-fabric-tutorial-standalone-cluster/aws-keypair.png