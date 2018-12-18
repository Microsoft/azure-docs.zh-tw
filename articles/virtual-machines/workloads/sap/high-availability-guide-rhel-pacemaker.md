---
title: 在 Azure 中的 Red Hat Enterprise Linux 上設定 Pacemaker | Microsoft Docs
description: 在 Azure 中的 Red Hat Enterprise Linux 上設定 Pacemaker
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: mssedusch
manager: timlt
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/17/2018
ms.author: sedusch
ms.openlocfilehash: f8e16af629eaa18b49b054be9fc478d633263ddb
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45636794"
---
# <a name="setting-up-pacemaker-on-red-hat-enterprise-linux-in-azure"></a>在 Azure 中的 Red Hat Enterprise Linux 上設定 Pacemaker

[planning-guide]:planning-guide.md
[deployment-guide]:deployment-guide.md
[dbms-guide]:dbms-guide.md
[sap-hana-ha]:sap-hana-high-availability.md
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351

[virtual-machines-linux-maintenance]:../../linux/maintenance-and-updates.md#memory-preserving-maintenance

> [!NOTE]
> Red Hat Enterprise Linux 上的 Pacemaker 會使用 Azure Fence 代理程式隔離叢集節點 (如有必要)。 如果資源停止容錯或叢集節點無法再彼此進行通訊，則容錯移轉需要 15 分鐘。 如需詳細資訊，請閱讀[以 RHEL 高可用性叢集成員身分執行的 Azure VM 需要相當長時間完成隔離，或者 VM 關機前隔離失敗/逾時](https://access.redhat.com/solutions/3408711)

請先閱讀下列 SAP Note 和文件：

* SAP Note [1928533]，其中包含：
  * SAP 軟體部署支援的 Azure VM 大小清單。
  * Azure VM 大小的重要容量資訊。
  * 支援的 SAP 軟體，以及作業系統 (OS) 與資料庫組合。
  * Microsoft Azure 上 Windows 和 Linux 所需的 SAP 核心版本。
* SAP Note [2015553] 列出 Azure 中 SAP 支援的 SAP 軟體部署先決條件。
* SAP Note [2002167] 建議適用於 Red Hat Enterprise Linux 的作業系統設定
* SAP Note [2009879] 提供適用於 Red Hat Enterprise Linux 的 SAP HANA 方針
* SAP Note [2178632] 包含在 Azure 中針對 SAP 回報的所有監視計量詳細資訊。
* SAP Note [2191498] 包含 Azure 中 Linux 所需的 SAP Host Agent 版本。
* SAP Note [2243692] 包含 Azure 中 Linux 上的 SAP 授權相關資訊。
* SAP Note [1999351] 包含 Azure Enhanced Monitoring Extension for SAP 的其他疑難排解資訊。
* [SAP Community WIKI](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) 包含 Linux 所需的所有 SAP Note。
* [適用於 SAP on Linux 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP on Linux 的 Azure 虛擬機器部署 (本文)][deployment-guide]
* [適用於 SAP on Linux 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* [在 Pacemaker 叢集中的 SAP HANA 系統複寫](https://access.redhat.com/articles/3004101)
* 一般 RHEL 文件
  * [高可用性附加元件概觀](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [高可用性附加元件管理](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [高可用性附加元件參考](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
* Azure 專用 RHEL 文件：
  * [RHEL 高可用性叢集的支援原則：以 Microsoft Azure 虛擬機器作為叢集成員](https://access.redhat.com/articles/3131341)
  * [在 Microsoft Azure 上安裝和設定 Red Hat Enterprise Linux 7.4 (和更新版本) 高可用性叢集](https://access.redhat.com/articles/3252491)

## <a name="cluster-installation"></a>叢集安裝

![RHEL 的 Pacemaker 概觀](./media/high-availability-guide-rhel-pacemaker/pacemaker-rhel.png)

下列項目會加上下列其中一個前置詞：**[A]** - 適用於所有節點、**[1]** - 僅適用於節點 1 或 **[2]** - 僅適用於節點 2。

1. **[A]** 註冊

   註冊您的虛擬機器，並將其附加至含有適用於 RHEL 7 的存放庫集區。

   <pre><code>sudo subscription-manager register
   # List the available pools
   sudo subscription-manager list --available --matches '*SAP*'
   sudo subscription-manager attach --pool=&lt;pool id&gt;
   </code></pre>

1. **[A]** 啟用 RHEL for SAP 存放庫

   若要安裝必要的封裝，請啟用下列存放庫。

   <pre><code>sudo subscription-manager repos --disable "*"
   sudo subscription-manager repos --enable=rhel-7-server-rpms
   sudo subscription-manager repos --enable=rhel-ha-for-rhel-7-server-rpms
   sudo subscription-manager repos --enable="rhel-sap-for-rhel-7-server-rpms"
   </code></pre>

1. **[A]** 安裝 RHEL 高可用性附加元件

   <pre><code>sudo yum install -y pcs pacemaker fence-agents-azure-arm nmap-ncat
   </code></pre>

1. **[A]** 設定主機名稱解析

   您可以使用 DNS 伺服器，或修改所有節點上的 /etc/hosts。 這個範例示範如何使用 /etc/hosts 檔案。
   請取代下列命令中的 IP 位址和主機名稱。 使用 /etc/hosts 的好處在於，您的叢集會變成不受 DNS 影響，而 DNS 也可能是單一失敗點。

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   將下列幾行插入至 /etc/hosts。 變更 IP 位址和主機名稱以符合您的環境

   <pre><code># IP address of the first cluster node
   <b>10.0.0.6 prod-cl1-0</b>
   # IP address of the second cluster node
   <b>10.0.0.7 prod-cl1-1</b>
   </code></pre>

1. **[A]** 將 hacluster 密碼變更為同一個密碼

   <pre><code>sudo passwd hacluster
   </code></pre>

1. **[A]** 新增 Pacemaker 的防火牆規則

   新增下列防火牆規則至叢集節點間的所有叢集通訊。

   <pre><code>sudo firewall-cmd --add-service=high-availability --permanent
   sudo firewall-cmd --add-service=high-availability
   </code></pre>

1. **[A]** 啟用基本的叢集服務

   執行下列命令，以啟用 Pacemaker 服務並加以啟動。

   <pre><code>sudo systemctl start pcsd.service
   sudo systemctl enable pcsd.service
   </code></pre>

1. **[1]** 建立 Pacemaker 叢集

   執行下列命令以驗證節點，並建立叢集。 將權杖設為 30000，以允許記憶體保留維護。 如需詳細資訊，請參閱[這篇 Linux 文章][virtual-machines-linux-maintenance]。

   <pre><code>sudo pcs cluster auth <b>prod-cl1-0</b> <b>prod-cl1-1</b> -u hacluster
   sudo pcs cluster setup --name <b>nw1-azr</b> <b>prod-cl1-0</b> <b>prod-cl1-1</b> --token 30000
   sudo pcs cluster start --all
   
   # Run the following command until the status of both nodes is online
   sudo pcs status
   
   # Cluster name: nw1-azr
   # WARNING: no stonith devices and stonith-enabled is not false
   # Stack: corosync
   # Current DC: <b>prod-cl1-1</b> (version 1.1.18-11.el7_5.3-2b07d5c5a9) - partition with quorum
   # Last updated: Fri Aug 17 09:18:24 2018
   # Last change: Fri Aug 17 09:17:46 2018 by hacluster via crmd on <b>prod-cl1-1</b>
   #
   # 2 nodes configured
   # 0 resources configured
   #
   # Online: [ <b>prod-cl1-0</b> <b>prod-cl1-1</b> ]
   #
   # No resources
   #
   #
   # Daemon Status:
   #   corosync: active/disabled
   #   pacemaker: active/disabled
   #   pcsd: active/enabled
   </code></pre>

1. **[A]** 設定預期的投票

   <pre><code>sudo pcs quorum expected-votes 2
   </code></pre>

## <a name="create-stonith-device"></a>建立 STONITH 裝置

STONITH 裝置會使用服務主體來對 Microsoft Azure 授權。 請遵循下列步驟來建立服務主體。

1. 移至 <https://portal.azure.com>。
1. 開啟 [Azure Active Directory] 刀鋒視窗  
   移至 [屬性]，並記下目錄識別碼。 這是「租用戶識別碼」。
1. 按一下 [應用程式註冊]
1. 按一下 [新增]
1. 輸入名稱、選取應用程式類型 [Web 應用程式/API]、輸入登入 URL (例如 http://localhost))，然後按一下 [建立]
1. 登入 URL 並未使用，而且可以是任何有效的 URL
1. 選取新的應用程式，然後按一下 [設定] 索引標籤中的金鑰
1. 輸入新金鑰的說明、選取 [永不過期]，然後按一下 [儲存]
1. 記下值。 此值會用來做為服務主體的**密碼**
1. 記下應用程式識別碼。 此識別碼會用來做為服務主體的使用者名稱 (以下步驟中的「登入識別碼」)

### <a name="1-create-a-custom-role-for-the-fence-agent"></a>**[1]** 為柵欄代理程式建立自訂角色

服務主體預設沒有存取您 Azure 資源的權限。 您需要為服務主體提供權限來啟動和停止 (解除配置) 叢集的所有虛擬機器。 如果您尚未建立自訂角色，您可以使用 [PowerShell](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-powershell#create-a-custom-role) 或 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli#create-a-custom-role) 來建立

針對輸入檔使用下列內容。 您必須調整訂用帳戶的內容，也就是使用您訂用帳戶的識別碼取代 c276fc76-9cd4-44c9-99a7-4fd71546436e 和 e91d47c4-76f3-4271-a796-21b4ecfe3624。 如果您只有一個訂用帳戶，請在 AssignableScopes 中移除第二個項目。

```json
{
  "Name": "Linux Fence Agent Role",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows to deallocate and start virtual machines",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e",
    "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624"
  ]
}
```

### <a name="1-assign-the-custom-role-to-the-service-principal"></a>**[1]** 將自訂角色指派給服務主體

將在上一章中建立的自訂角色「Linux 柵欄代理程式角色」指派給服務主體。 不要再使用「擁有者」角色！

1. 移至 https://portal.azure.com。
1. 開啟 [所有資源] 刀鋒視窗
1. 選取第一個叢集節點的虛擬機器
1. 選取 [存取控制 (IAM)]
1. 按一下 [新增]
1. 選取「Linux 柵欄代理程式角色」角色
1. 輸入您先前建立的應用程式名稱
1. Click OK

針對第二個叢集節點重複上述步驟。

### <a name="1-create-the-stonith-devices"></a>**[1]** 建立 STONITH 裝置

當您編輯虛擬機器的權限之後，就可以在叢集中設定 STONITH 裝置。

<pre><code>
sudo pcs property set stonith-timeout=900
</code></pre>

使用下列命令來設定柵欄裝置。

> [!NOTE]
> 如果 RHEL 主機名稱和 Azure 節點名稱不相同，則命令中只需要選項「pcmk_host_map」。 請參閱命令中的粗體區段。

<pre><code>sudo pcs stonith create rsc_st_azure fence_azure_arm login="<b>login ID</b>" passwd="<b>password</b>" resourceGroup="<b>resource group</b>" tenantId="<b>tenant ID</b>" subscriptionId="<b>subscription id</b>" <b>pcmk_host_map="prod-cl1-0:10.0.0.6;prod-cl1-1:10.0.0.7"</b> power_timeout=240 pcmk_reboot_timeout=900</code></pre>

### <a name="1-enable-the-use-of-a-stonith-device"></a>**[1]** 啟用 STONITH 裝置的使用

<pre><code>sudo pcs property set stonith-enabled=true
</code></pre>

## <a name="next-steps"></a>後續步驟

* [適用於 SAP 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP 的 Azure 虛擬機器部署][deployment-guide]
* [適用於 SAP 的 Azure 虛擬機器 DBMS 部署][dbms-guide]
* 若要了解如何建立高可用性並為 Azure VM 上的 SAP HANA 規劃災害復原，請參閱 [Azure 虛擬機器 (VM) 上 SAP HANA 的高可用性][sap-hana-ha]
