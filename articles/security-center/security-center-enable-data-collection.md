---
title: Azure 資訊安全中心的資料收集 | Microsoft Docs
description: " 了解如何在 Azure 資訊安全中心啟用資料收集。 "
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 411d7bae-c9d4-4e83-be63-9f2f2312b075
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/23/2018
ms.author: rkarlin
ms.openlocfilehash: 9043c6583a15d3be9d0d468e83a4bf79b3121794
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304113"
---
# <a name="data-collection-in-azure-security-center"></a>Azure 資訊安全中心的資料收集
資訊安全中心會從您的 Azure 虛擬機器 (VM) 和非 Azure 電腦收集資料，以監視安全性漏洞與威脅。 資料是使用 Microsoft Monitoring Agent 收集而得，收集的方式是讀取機器的各種安全性相關組態和事件記錄檔，並將資料複製到工作區進行分析。 這類資料的範例包括︰作業系統類型和版本、作業系統記錄檔 (Windows 事件記錄檔)、執行中程序、電腦名稱、IP 位址和已登入的使用者。 Microsoft Monitoring Agent 也會將損毀傾印檔案複製到工作區。

必須收集資料，才能掌握遺漏的更新、設定錯誤的 OS 安全性設定、端點保護啟用，以及健康情況和威脅偵測。 

本文提供的指引有關於如何安裝 Microsoft Monitoring Agent，以及設定用於儲存所收集資料的 Log Analytics 工作區。 需要執行這兩項作業，才能啟用資料收集。 

> [!NOTE]
> - 計算資源 (VM 和非 Azure 電腦) 才需要收集資料。 即使您未佈建代理程式，也可受惠於 Azure 資訊安全中心；不過，您的安全性有限，而且不支援以上所列的功能。  
> - 如需支援的平台清單，請參閱 [Azure 資訊安全中心支援的平台](security-center-os-coverage.md)。
> - 目前不支援虛擬機器擴展集的資料收集。


## <a name="enable-automatic-provisioning-of-microsoft-monitoring-agent"></a>啟用 Microsoft Monitoring Agent 的自動佈建     
若要從電腦收集資料，您應該安裝 Microsoft Monitoring Agent。  代理程式可以自動安裝 (建議選項)，您也可以選擇手動安裝代理程式。  

>[!NOTE]
> 預設會關閉自動佈建。 若要將資訊安全中心設定為預設安裝自動佈建，請將它設定為 [開啟]。
>

開啟自動佈建時，資訊安全中心會在所有支援的 Azure VM 和任何新建立的 VM 上佈建 Microsoft Monitoring Agent。 強烈建議使用自動佈建，但是手動代理程式安裝也可行。 [深入了解如何安裝 Microsoft Monitoring Agent 擴充功能](#manualagent)。



若要啟用 Microsoft Monitoring Agent 的自動佈建：
1. 在 [資訊安全中心] 主功能表下，選取 [安全性原則]。
2. 選取訂用帳戶。

  ![選取訂用帳戶][7]

3. 在 [安全性原則] 下，選取 [資料收集]。
4. 在 [自動佈建] 下，選取 [開啟] 啟用自動佈建。
5. 選取 [ **儲存**]。

  ![啟用自動佈建][1]

>[!NOTE]
> - 如需佈建預先存在的安裝的相關指示，請參閱[在預先存在的代理程式安裝情況下自動佈建](#preexisting)。
> - 如需手動佈建的指示，請參閱[手動安裝 Microsoft Monitoring Agent 擴充功能](#manualagent)。
> - 如需有關關閉自動佈建的指示，請參閱[關閉自動佈建](#offprovisioning)。
>


## <a name="workspace-configuration"></a>工作區設定
資訊安全中心收集的資料會儲存在 Log Analytics 工作區。  您可以選擇將從 Azure VM 收集而得的資料儲存於資訊安全中心建立的工作區中，或儲存於您建立的現有工作區中。 

工作區是依據訂用帳戶進行設定，而且許多訂用帳戶可能會使用相同的工作區。

### <a name="using-a-workspace-created-by-security-center"></a>使用資訊安全中心所建立的工作區

資訊安全中心可以自動建立用來儲存資料的預設工作區。 

若要選取資訊安全中心所建立的工作區：

1.  在 [預設工作區組態] 下，選取 [使用資訊安全中心建立的工作區]。
   ![選取定價層][10] 

2. 按一下 [檔案] 。<br>
    資訊安全中心會在此地理位置建立新的資源群組和預設工作區，並將代理程式連線到該工作區。 工作區和資源群組的命名慣例如下：<br>
**工作區：DefaultWorkspace-[subscription-ID]-[geo]<br> 資源群組：DefaultResouceGroup-[geo]**

   如果訂用帳戶中包含多個地理位置上的虛擬機器，則資訊安全中心會建立多個工作區。 建立多個工作區是為了遵守資料隱私權規則。
-   資訊安全中心會依據為訂用帳戶設定的定價層，在工作區上自動啟用資訊安全中心解決方案。 

> [!NOTE]
> 資訊安全中心所建立的工作區不會產生 Log Analytics 費用。 資訊安全中心所建立工作區的 Log Analytics 定價層不會影響資訊安全中心的收費。 資訊安全中心的計費一律根據您的資訊安全中心的安全性原則，以及工作區安裝的解決方案。 針對免費層，資訊安全中心在預設工作區啟用 SecurityCenterFree 解決方案。 對於標準層，資訊安全中心在預設工作區啟用 Security 解決方案。

如需詳細資訊，請參閱[資訊安全中心價格](https://azure.microsoft.com/pricing/details/security-center/)。

如需現有 Log Analytics 帳戶的詳細資訊，請參閱[現有的 Log Analytics 客戶](security-center-faq.md#existingloganalyticscust)。

### <a name="using-an-existing-workspace"></a>使用現有工作區

如果您已經有現有的 Log Analytics 工作區，您可以使用相同的工作區。

若要使用現有的 Log Analytics 工作區，您必須具備工作區的讀取和寫入權限。

> [!NOTE]
> 在現有工作區上啟用的解決方案將套用至所連線的 Azure VM。 若為付費解決方案，這可能導致額外的費用。 對於資料隱私權考量，請確定您選取的工作區位於正確的地理區域中。
>

若要選取現有的 Log Analytics 工作區：

1. 在 [預設工作區組態] 下，選取 [使用其他工作區]。

   ![選取現有工作區][2]

2. 從下拉式功能表中，選取要儲存收集資料的工作區。

  > [!NOTE]
  > 在下拉式功能表中，所有訂用帳戶的所有工作區均可供使用。 如需詳細資訊，請參閱[跨訂用帳戶工作區選取](security-center-enable-data-collection.md#cross-subscription-workspace-selection)。 您必須具有此工作區的存取權。
  >
  >

3. 選取 [ **儲存**]。
4. 選取 [儲存] 之後，系統會詢問您是否要重新設定先前連線到預設工作區的受監控虛擬機器。

   - 如果您希望新的工作區設定僅套用在新的虛擬機器上，請選取 [否]。 新的工作區設定只會套用在新安裝的代理程式，以及新探索到的未安裝 Microsoft Monitoring Agent 之虛擬機器。
   - 如果您希望新的工作區設定套用在所有虛擬機器，請選取 [是]。 此外，每個連線到資訊安全中心建立之工作區的虛擬機器會重新連線到新的目標工作區。

   > [!NOTE]
   > 如果您選取 [是]，您必須刪除資訊安全中心建立的工作區，直到所有虛擬機器已重新連線至新的目標工作區。 如果過早刪除工作區，這項作業將會失敗。
   >
   >

   - 選取 [取消] 以取消作業。

     ![選取現有工作區][3]

5. 針對您要設定 Microsoft Monitoring agent 所需的工作區，選取定價層。 <br>若要使用現有的工作區，請設定工作區的定價層。 這會在工作區上安裝資訊安全中心解決方案 (如果不存在)。

    a.  在 [資訊安全中心] 主功能表中，選取 [安全性原則]。
     
    b.  選取您要連線代理程式所需的工作區。
        ![選取工作區][8] c. 設定定價層。
        ![選取定價層][9] 
   
   >[!NOTE]
   >如果工作區已經啟用 **Security** 或 **SecurityCenterFree** 解決方案，將會自動設定定價。 

## <a name="cross-subscription-workspace-selection"></a>跨訂用帳戶工作區選取
當您選取用來儲存資料的工作區時，所有訂用帳戶的所有工作區均可供使用。 跨訂用帳戶工作區選取可讓您從執行於不同訂用帳戶的虛擬機器收集資料，並將它儲存在您選擇的工作區中。 如果您在組織中使用集中式工作區，而且想要將它用於安全性資料收集，此選項會很有用。 如需有關如何管理工作區的詳細資訊，請參閱[管理工作區存取](https://docs.microsoft.com/azure/log-analytics/log-analytics-manage-access)。


## <a name="data-collection-tier"></a>資料收集層
資訊安全中心可以減少事件的數量，同時維持足夠的事件以供調查、稽核和威脅偵測之用。 對於訂用帳戶和工作區，您可以從代理程式將收集的四組事件中選擇合適的篩選原則。

- **所有事件** – 適合想要確定收集到所有事件的客戶。 這是預設值。
- **一般** – 這組事件能滿足大部分客戶的需求，而能讓這些客戶使用完整的稽核記錄。
- **最小** – 適合想要將事件數量最小化的客戶，是較小的一組事件。
- **無** – 停用安全性和 App Locker 記錄檔的安全性事件收集。 對於選擇此選項的客戶，其安全性儀表板上僅有 Windows 防火牆記錄檔和主動式評估，例如反惡意程式碼軟體、基準和更新。

> [!NOTE]
> 只能資訊安全中心的標準層會提供這些安全性事件集。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。
這些集合都是設計用來處理一般情況。 實作之前，請務必評估哪一個適合您的需求。
>
>

為了判定屬於**一般**和**最小**事件集合的事件，我們致力於與客戶合作並配合業界標準，以了解每個事件及其使用方式的未篩選頻率。 此流程中採取下列指導方針：

- **最小** - 確保這個集合所涵蓋的僅有可能代表發生成功入侵和重大事件的事件，數量很少。 例如，這個集合包含使用者成功和失敗的登入活動 (事件識別碼 4624、4625)，不過不包含對於稽核而言相當重要但對於偵測而言沒有意義而且數量相對多的登出活動。 這個集合的大部分資料量是登入事件和流程建立事件 (事件識別碼 4688)。
- **一般** - 提供此集合中的完整使用者稽核記錄。 例如，這個集合包含使用者登入和使用者登出活動 (事件識別碼 4634)。 納入了各種稽核動作，例如安全性群組變更、主要網域控制站 Kerberos 作業，以及業界組織建議的其他事件。

「一般」集合包含的事件相當少，因為選擇一般集合而非其他事件的主要動機在於降低數量，且不篩選出特定事件。

以下是對於每個集合的安全性和 App Locker 事件識別碼的完整分析：

| 資料層 | 收集的事件指標 |
| --- | --- |
| 有限 | 1102,4624,4625,4657,4663,4688,4700,4702,4719,4720,4722,4723,4724,4727,4728,4732,4735,4737,4739,4740,4754,4755, |
| | 4756,4767,4799,4825,4946,4948,4956,5024,5033,8001,8002,8003,8004,8005,8006,8007,8222 |
| 一般 | 1,299,300,324,340,403,404,410,411,412,413,431,500,501,1100,1102,1107,1108,4608,4610,4611,4614,461,4622, |
| |  4624,4625,4634,4647,4648,4649,4657,4661,4662,4663,4665,4666,4667,4688,4670,4672,4673,4674,4675,4689,4697, |
| | 4700,4702,4704,4705,4716,4717,4718,4719,4720,4722,4723,4724,4725,4726,4727,4728,4729,4733,4732,4735,4737, |
| | 4738,4739,4740,4742,4744,4745,4746,4750,4751,4752,4754,4755,4756,4757,4760,4761,4762,4764,4767,4768,4771, |
| | 4774,4778,4779,4781,4793,4797,4798,4799,4800,4801,4802,4803,4825,4826,4870,4886,4887,4888,4893,4898,4902, |
| | 4904,4905,4907,4931,4932,4933,4946,4948,4956,4985,5024,5033,5059,5136,5137,5140,5145,5632,6144,6145,6272, |
| | 6273,6278,6416,6423,6424,8001,8002,8003,8004,8005,8006,8007,8222,26401,30004 |

> [!NOTE]
> - 如果您使用群組原則物件 (GPO)，則建議您啟用稽核原則程序建立事件 4688 和事件 4688 內的 [CommandLine] 欄位。 如需更多程序建立事件 4688 的相關資訊，請參閱資訊安全中心的[常見問題集](security-center-faq.md#what-happens-when-data-collection-is-enabled)。 如需更多稽核原則的相關資訊，請參閱[稽核原則建議](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations)。
> -  若要啟用[自適性應用程式控制](security-center-adaptive-application.md)的資料收集，資訊安全中心會在稽核模式中設定本機 AppLocker 原則以允許所有的應用程式。 這會導致 AppLocker 產生事件，然後由資訊安全中心收集並利用。 請務必注意，在已經設定 AppLocker 原則的機器上不會設定此原則。 
> - 若要收集 Windows 篩選平台[事件識別碼 5156](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=5156)，您必須啟用[稽核篩選平台連線](https://docs.microsoft.com/windows/security/threat-protection/auditing/audit-filtering-platform-connection) (Auditpol /set /subcategory:"Filtering Platform Connection" /Success:Enable)
>

若要選擇篩選原則：
1. 在 [安全性原則資料收集] 刀鋒視窗的 [安全性事件] 下，選取您的篩選原則。
2. 選取 [ **儲存**]。

   ![選擇篩選原則][5]

### 在預先存在的代理程式安裝情況下自動佈建 <a name="preexisting"></a> 

下列使用案例會指定在已經安裝代理程式或擴充功能的情況下，自動佈建的運作方式。 

- Microsoft Monitoring Agent 已安裝在機器上，但不是擴充功能<br>
如果 Microsoft Monitoring Agent 已經直接安裝在虛擬機器上 (而非作為 Azure 擴充功能)，資訊安全中心不會再安裝 Microsoft Monitoring Agent。 您可以開啟自動佈建，並在資訊安全中心的自動佈建組態中選取相關的使用者工作區。 如果您選擇 VM 已經連線到的相同工作區，則會利用 Microsoft Monitoring Agent 擴充功能包裝現有的代理程式。 

> [!NOTE]
> 如果已安裝 SCOM 代理程式版本 2012，請**勿**開啟自動佈建。 

如需詳細資訊，請參閱[如果虛擬機器上已安裝 SCOM 或 OMS 直接代理程式，會發生什麼狀況？](security-center-faq.md#scomomsinstalled)

-   預先存在的 VM 擴充功能已存在<br>
    - 資訊安全中心支援現有的擴充功能安裝，而且不會覆寫現有的連線。 資訊安全中心會將虛擬機器送出的資料儲存到已經連線的工作區，並根據工作區上啟用的解決方案來提供保護。   
    - 若要查看現有擴充功能將資料傳送到哪個工作區，請執行測試以[驗證與 Azure 資訊安全中心的連線](https://blogs.technet.microsoft.com/yuridiogenes/2017/10/13/validating-connectivity-with-azure-security-center/)。 或者，您可以開啟 Log Analytics、選取工作區、選取 VM，以及查看 Microsoft Monitoring Agent 連線。 
    - 如果您環境中的用戶端工作站上已安裝 Microsoft Monitoring Agent 並會向現有的 Log Analytics 工作區報告，請檢閱 [Azure 資訊安全中心支援的作業系統](security-center-os-coverage.md)清單，確保您的作業系統受支援，並請參閱[現有的 Log Analytics 客戶](security-center-faq.md#existingloganalyticscust)以取得詳細資訊。
 
### 關閉自動佈建 <a name="offprovisioning"></a>
您可以在安全性原則中關閉從資源自動佈建，隨時關閉這項設定。 


1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您想要停用自動佈建的訂用帳戶。
3. 在 [安全性原則 - 資料收集] 刀鋒視窗的 [自動佈建] 下，選取 [關閉]。
4. 選取 [ **儲存**]。

  ![停用自動佈建][6]

自動佈建停用 (關閉) 後，就不會顯示預設的工作區組態區段。

如果您關閉先前開啟的自動佈建：
-   代理程式就不會佈建在新的 VM 上。
-   資訊安全中心會停止從預設工作區收集資料。
 
> [!NOTE]
>  停用自動佈建，並不會從已佈建代理程式的 Azure VM 中移除 Microsoft Monitoring Agent。 如需移除 OMS 擴充功能的相關資訊，請參閱[我要如何移除資訊安全中心安裝的 OMS 擴充功能](security-center-faq.md#remove-oms)。
>
    
## 手動佈建代理程式 <a name="manualagent"></a>
 
手動安裝 Microsoft Monitoring Agent 的方式有很多種。 手動安裝時，請務必停用自動佈建。

### <a name="operations-management-suite-vm-extension-deployment"></a>Operations Management Suite VM 擴充功能部署 

您可以手動安裝 Microsoft Monitoring Agent，讓資訊安全中心可以收集您虛擬機器所傳來的安全性資料，並提供建議和警示。
1.  選取自動佈建 – 關閉。
2.  針對您要設定 Microsoft Monitoring agent 的工作區，建立工作區及設定定價層：

    a.  在 [資訊安全中心] 主功能表中，選取 [安全性原則]。
     
    b.  選取您要在其中連線代理程式的工作區。 請確定工作區位於您在資訊安全中心使用的相同訂用帳戶中，而且您具備工作區的讀取/寫入權限。
        ![選取工作區][8]
3. 設定定價層。
   ![選取定價層][9] 
   >[!NOTE]
   >如果工作區已經啟用 **Security** 或 **SecurityCenterFree** 解決方案，將會自動設定定價。 
   > 

4.  如果您想要使用 Resource Manager 範本在新的 VM 上部署代理程式，請安裝 OMS 虛擬機器擴充功能：

    a.  [安裝適用於 Windows 的 OMS 虛擬機器擴充功能](../virtual-machines/extensions/oms-windows.md)
    
    b.  [安裝適用於 Linux 的 OMS 虛擬機器擴充功能](../virtual-machines/extensions/oms-linux.md)
5.  若要在現有的 VM 上部署擴充功能，請遵循[收集有關 Azure 虛擬機器的資料](../log-analytics/log-analytics-quick-collect-azurevm.md)中的指示。

  > [!NOTE]
  > **收集事件和效能資料**一節是選擇性的。
  >
6. 若要使用 PowerShell 來部署擴充功能，請使用下列 PowerShell 範例︰
    1.  移至 **Log Analytics** ，然後按一下 [進階設定]。
    
        ![設定 Log Analytics][11]

    2. 複製 [工作區識別碼] 和 [主索引鍵] 的值。
  
       ![複製值][12]

    3. 在公用組態和私用組態中填入下列值：
     
            $PublicConf = '{
                "workspaceId": "WorkspaceID value",
                "MultipleConnectistopOnons": true
            }' 
 
            $PrivateConf = '{
                "workspaceKey": "<Primary key value>”
            }' 

      - 在 Windows VM 上安裝時：
        
             Set-AzureRmVMExtension -ResourceGroupName $vm.ResourceGroupName -VMName $vm.Name -Name "MicrosoftMonitoringAgent" -Publisher "Microsoft.EnterpriseCloud.Monitoring" -ExtensionType "MicrosoftMonitoringAgent" -TypeHandlerVersion '1.0' -Location $vm.Location -Settingstring $PublicConf -ProtectedSettingString $PrivateConf -ForceRerun True 
    
       - 在 Linux VM 上安裝時：
        
             Set-AzureRmVMExtension -ResourceGroupName $vm1.ResourceGroupName -VMName $vm1.Name -Name "OmsAgentForLinux" -Publisher "Microsoft.EnterpriseCloud.Monitoring" -ExtensionType "OmsAgentForLinux" -TypeHandlerVersion '1.0' -Location $vm.Location -Settingstring $PublicConf -ProtectedSettingString $PrivateConf -ForceRerun True`




## <a name="troubleshooting"></a>疑難排解

-   若要找出自動佈建安裝問題，請參閱[監視代理程式健康情況問題](security-center-troubleshooting-guide.md#mon-agent)。

-  若要識別監視代理程式網路需求，請參閱[針對監視代理程式網路需求進行疑難排解](security-center-troubleshooting-guide.md#mon-network-req)。
-   若要識別手動上架問題，請參閱[如何針對 Operations Management Suite 上架問題進行疑難排解](https://support.microsoft.com/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues)

- 若要找出未受監視的 VM 和電腦問題，請參閱[未受監視的 VM 和電腦](security-center-virtual-machine-protection.md#unmonitored-vms-and-computers)

## <a name="next-steps"></a>後續步驟
本文說明資料收集和自動佈建如何在資訊安全中心運作。 如要深入了解資訊安全中心，請參閱下列主題：

* [Azure 資訊安全中心常見問題集](security-center-faq.md)-- 尋找有關使用服務的常見問題。
* [Azure 資訊安全中心的安全性健康狀態監視](security-center-monitoring.md)--了解如何監視 Azure 資源的健康狀態。



<!--Image references-->
[1]: ./media/security-center-enable-data-collection/enable-automatic-provisioning.png
[2]: ./media/security-center-enable-data-collection/use-another-workspace.png
[3]: ./media/security-center-enable-data-collection/reconfigure-monitored-vm.png
[5]: ./media/security-center-enable-data-collection/data-collection-tiers.png
[6]: ./media/security-center-enable-data-collection/disable-data-collection.png
[7]: ./media/security-center-enable-data-collection/select-subscription.png
[8]: ./media/security-center-enable-data-collection/manual-provision.png
[9]: ./media/security-center-enable-data-collection/pricing-tier.png
[10]: ./media/security-center-enable-data-collection/workspace-selection.png
[11]: ./media/security-center-enable-data-collection/log-analytics.png
[12]: ./media/security-center-enable-data-collection/log-analytics2.png
