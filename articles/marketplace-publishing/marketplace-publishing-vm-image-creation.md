---
title: 建立 Azure Marketplace 的虛擬機器映像 | Microsoft Docs
description: 如何建立 Azure Marketplace 的虛擬機器映像以讓其他人購買的詳細指示。
services: Azure Marketplace
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 5c937b8e-e28d-4007-9fef-624046bca2ae
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 01/05/2017
ms.author: hascipio; v-divte
ms.openlocfilehash: 2a3c317dc9abdb861a007be9aaed714089e9f453
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388189"
---
# <a name="guide-to-create-a-virtual-machine-image-for-the-azure-marketplace"></a>建立 Azure Marketplace 的虛擬機器映像的指南
本文的 **步驟 2**會逐步引導您準備您將部署到 Azure Marketplace 的虛擬硬碟 (VHD)。 您的 VHD 是 SKU 的基礎。 這個程序會因為您是否提供以 Linux 或 Windows 為基礎的 SKU 而有所不同。 本文將探討這兩種狀況。 這個程序可與[帳戶建立和註冊][link-acct-creation]同步執行。

## <a name="1-define-offers-and-skus"></a>1.定義供應項目和 SKU
在本節中，您會了解如何定義供應項目與其相關聯的 SKU。

供應項目是其所有 SKU 的「上層」。 您可以擁有多個供應項目。 您可以隨意決定供應項目的建構方式。 當供應項目進入預備環境，它的所有 SKU 也會一起進入。 細心考量您的 SKU 識別碼，因為這些識別碼會顯示於 URL 中：

* Azure.com： http://azure.microsoft.com/marketplace/partners/{PartnerNamespace}/{OfferIdentifier}-{SKUidentifier}
* Azure 預覽入口網站： https://portal.azure.com/#gallery/{PublisherNamespace}.{OfferIdentifier}{SKUIDdentifier}  

SKU 是 VM 映像的商務名稱。 VM 映像包含一個作業系統磁碟以及零或多個資料磁碟。 它在本質上是虛擬機器的完整儲存體設定檔。 每個磁碟都需要一個 VHD。 即使空的資料磁碟也需要建立一個 VHD。

無論您使用哪一種作業系統，您只能加入 SKU 所需的最少數目資料磁碟。 客戶無法在部署時移除屬於映像一部分的磁碟，但是可隨時在必要時於部署期間或之後加入磁碟。

> [!IMPORTANT]
> **請勿變更新映像版本中的磁碟計數。** 如果您必須重新設定映像中的資料磁碟，請定義新的 SKU。 對於自動調整大小、透過 ARM 範本自動部署解決方案和其他案例來說，發行磁碟計數相異的新映像版本有可能會毀損以新映像版本為基礎的新部署。
>
>

### <a name="11-add-an-offer"></a>1.1 加入供應項目
1. 使用您的賣方帳戶登入[發佈入口網站][link-pubportal]。
2. 選取發佈入口網站的 [虛擬機器]  索引標籤。 在提示的輸入欄位中輸入您的供應項目名稱。 供應項目名稱通常是您計劃在 Azure Marketplace 中銷售的產品或服務的名稱。
3. 選取 [建立] 。

### <a name="12-define-a-sku"></a>1.2 定義 SKU
在您加入供應項目之後，您必須定義及識別您的 SKU。 您可以有多個供應項目，每個供應項目在其下可以有多個 SKU。 當供應項目進入預備環境，它的所有 SKU 也會一起進入。

1. **加入 SKU。** SKU 必須具備用於 URL 中的識別碼。 此識別碼必須是發行設定檔中的唯一識別碼，但是若與其他發行者發生識別碼衝突，並不會有任何風險。

   > [!NOTE]
   > 供應項目和 SKU 識別項會顯示在 Marketplace 中的供應項目 URL。
   >
   >
2. **為 SKU 加入摘要描述。** 客戶可以看見摘要說明，所以您應讓說明可容易閱讀。 此資訊在「進入預備環境」階段之前都不需要鎖定。 在那之前，您都可以任意編輯。
3. 如果您正在使用以 Windows 為基礎的 SKU，請遵循建議的連結，要求使用核准的 Windows Server 版本。

## <a name="2-create-an-azure-compatible-vhd-linux-based"></a>2.建立與 Azure 相容的 VHD (以 Linux 為基礎)
本節的焦點在於為 Azure Marketplace 建立以 Linux 為基礎的 VM 映像所採行的最佳作法。 如需逐步解說，請參閱下列文件：[建立自訂 Linux VM 映像](../virtual-machines/linux/create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="3-create-an-azure-compatible-vhd-windows-based"></a>3.建立與 Azure 相容的 VHD (以 Windows 為基礎)
本節的焦點在於為 Azure Marketplace 建立以 Windows Server 為基礎的 SKU 所採行的步驟。

### <a name="31-ensure-that-you-are-using-the-correct-base-vhds"></a>3.1 確認您使用的是正確的基底 VHD
VM 映像的作業系統 VHD 必須以獲得 Azure 核准的基底映像為基礎，包含 Windows Server 或 SQL Server。

若要開始，請從下列其中一個映像建立 VM，位於 [Microsoft Azure 入口網站][link-azure-portal]：

* Windows Server ([2012 R2 Datacenter][link-datactr-2012-r2]、[2012 Datacenter][link-datactr-2012]、[2008 R2 SP1][link-datactr-2008-r2])
* SQL Server 2014 
* SQL Server 2012 SP2 

這些連結也可以在 SKU 頁面下的發佈入口網站中找到。

> [!TIP]
> 如果您使用的是最新的 Azure 入口網站或 PowerShell，2014 年 9 月 8 日之後發行的 Windows Server 映像都會獲得核准。
>
>

### <a name="32-create-your-windows-based-vm"></a>3.2 建立您的 Windows 型 VM
從 Microsoft Azure 入口網站，您可以利用幾個簡單的步驟，根據核准的基底映像建立您的 VM。 下列內容是程序概觀：

1. 從基底映像頁面，選取 [建立虛擬機器] 以導向新的 [Microsoft Azure 入口網站][link-azure-portal]。

    ![繪圖][img-acom-1]
2. 利用您想使用的 Azure 訂用帳戶之 Microsoft 帳戶和密碼登入入口網站。
3. 遵循提示以使用您已選取的基底映像建立 VM。 您必須提供主機名稱 (電腦的名稱)、使用者名稱 (以系統管理員身分註冊) 和 VM 的密碼。

    ![繪圖][img-portal-vm-create]
4. 選取要部署之 VM 的大小：

    a.    如果您打算開發 VHD 內部部署，大小不會造成影響。 請考慮使用其中一個較小的 VM。

    b.    如果您打算在 Azure 中開發映像，請考慮使用其中一個建議的 VM 大小做為選取的映像。

    c.    關於價格資訊，請參閱入口網站上所顯示的**建議定價層**選取器。 (在這個案例中，發行者為 Microsoft。 )

    ![繪圖][img-portal-vm-size]
5. 設定屬性：

    a.    若要快速部署，您可以保留 [選用設定] 與 [資源群組] 下的預設值。

    b.    在 [儲存體帳戶] 之下，您可以視情況選取要儲存作業系統 VHD 的儲存體帳戶。

    c.    在 [資源群組] 之下，您可以視情況選取要放置 VM 的邏輯群組。
6. 選取部署的 [位置]  ：

    a.    如果您打算開發 VHD 內部部署，位置不會造成影響，因為您稍後會將映像上傳至 Azure。

    b.    如果您打算在 Azure 中開發映像，請考慮從一開始就使用其中一個位於美國的 Microsoft Azure 區域。 這樣可以在您提交映像以進行認證時，加速 Microsoft 代表您執行的 VHD 複製程序。

    ![繪圖][img-portal-vm-location]
7. 按一下頁面底部的 [新增] 。 開始部署 VM。 在數分鐘內，您將成功完成部署，並且可以開始為您的 SKU 建立映像。

### <a name="33-develop-your-vhd-in-the-cloud"></a>3.3 在雲端開發您的 VHD
強烈建議您在使用遠端桌面通訊協定 (RDP)，在雲端開發您的 VHD。 您會利用在佈建期間指定的使用者名稱和密碼連接到 RDP。

> [!IMPORTANT]
> **請勿使用受控磁碟。** 因為目前不支援從受控磁碟中建立映像，所以用以針對雲端開發 VHD 的虛擬機器不應以其為基礎。
> 以選擇性功能建立虛擬機器會變更受控磁碟的預設值。

> 如果您開發 VHD 內部部署 (不建議)，請參閱 [建立虛擬機器映像內部部署](marketplace-publishing-vm-image-creation-on-premise.md)。 如果您在雲端中開發，就不必下載您的 VHD。
>
>

**使用 [Microsoft Azure 入口網站][link-azure-portal]，透過 RDP 進行連接**

1. 選取 [所有服務] > [VM]。
2. [虛擬機器] 刀鋒視窗隨即開啟。 確認您要連接的 VM 正在運作，然後從已部署的 VM 清單中選取該 VM。
3. 描述選取之 VM 的刀鋒視窗隨即開啟。 在頂端，按一下 [連接] 。
4. 系統會提示您輸入在佈建期間指定的使用者名稱和密碼。

**使用 PowerShell，透過 RDP 連接**

若要將遠端桌面檔案下載至本機電腦，請使用 [Get-AzureRemoteDesktopFile Cmdlet][link-technet-2]。 為了使用這個 Cmdlet，您必須知道服務名稱和 VM 的名稱。 如果您已從 [Microsoft Azure 入口網站][link-azure-portal]建立 VM，您可以在 VM 屬性下找到此資訊：

1. 在 Microsoft Azure 入口網站中，選取 [所有服務] > [VM]。
2. [虛擬機器] 刀鋒視窗隨即開啟。 選取您部署的 VM。
3. 描述選取之 VM 的刀鋒視窗隨即開啟。
4. 按一下 [內容] 。
5. 網域名稱的第一個部份是服務名稱。 主機名稱是 VM 名稱。

    ![繪圖][img-portal-vm-rdp]
6. 要將已建立之 VM 的 RDP 檔案下載至系統管理員的本機桌面之 Cmdlet 如下所示。

        Get‐AzureRemoteDesktopFile ‐ServiceName “baseimagevm‐6820cq00” ‐Name “BaseImageVM” –LocalPath “C:\Users\Administrator\Desktop\BaseImageVM.rdp”

在 MSDN 的文章＜ [透過 RDP 或 SSH 連接至 Azure VM](http://msdn.microsoft.com/library/azure/dn535788.aspx)＞中可找到 RDP 的相關詳細資訊。

**設定 VM 並建立您的 SKU**

下載作業系統 VHD 之後，請使用 Hyper-V 並將 VM 設定為開始建立您的 SKU。 詳細步驟可在下列 TechNet 連結中找到： [安裝 Hyper-V 和設定 VM](http://technet.microsoft.com/library/hh846766.aspx)。

### <a name="34-choose-the-correct-vhd-size"></a>3.4 選擇正確的 VHD 大小
您 VM 映像中的 Windows 作業系統 VHD 應建立為 128 GB 固定格式 VHD。  

如果實體大小小於 128 GB，此 VHD 應為疏鬆 VHD。 所提供的基底 Windows 和 SQL Server 映像皆符合這些需求；所以請勿變更所取得之 VHD 的格式或大小。  

資料磁碟的大小可高達 1 TB。 決定磁碟大小時，請記得客戶無法於部署時在映像中調整 VHD 大小。 資料磁碟 VHD 應建立為固定格式 VHD。 也可以是疏鬆 VHD。 資料磁碟可以空白或包含資料。

### <a name="35-install-the-latest-windows-patches"></a>3.5 安裝最新的 Windows 補充程式
基底映像包含至其發佈日期為止的最新補充程式。 在您發行您已建立的作業系統 VHD 之前，請確認 Windows Update 已執行，且已安裝最新的「關鍵」和「重要」安全性更新。

### <a name="36-perform-additional-configuration-and-schedule-tasks-as-necessary"></a>3.6 必要時執行其他設定並排程工作
如果需要進行其他設定，請在部署 VM 後，考慮使用在啟動時執行的已排程工作，進行最終的 VM 變更：

* 讓工作在成功執行時將自己刪除是最佳作法。
* 所有設定都應該依靠磁碟機 C 或 D，因為它們是唯二保證隨時都存在的磁碟機。 磁碟機 C 是作業系統磁碟，而磁碟機 D 是暫存本機磁碟。

### <a name="37-generalize-the-image"></a>3.7 一般化映像
Azure Marketplace 中的所有映像通常都必須能夠重複使用。 也就是說，作業系統 VHD 必須一般化：

* 對 Windows 而言，此映像應該是 "sysprepped"，而且所有設定都應該在支援 **sysprep** 命令的狀態下進行。
* 您可以從目錄 %windir%\System32\Sysprep 執行以下命令。

        sysprep.exe /generalize /oobe /shutdown

  下列 MSDN 文章的「步驟」提供如何 sysprep 作業系統的指導：[建立 Windows Server VHD 並將其上傳至 Azure](../virtual-machines/windows/classic/createupload-vhd.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

## <a name="4-deploy-a-vm-from-your-vhds"></a>4.從您的 VHD 部署 VM
在您將 VHD (一般化作業系統 VHD 和零或更多個資料磁碟 VHD) 上傳至 Azure 儲存體帳戶之後，您就可以將它們註冊為使用者 VM 映像。 您可以接著測試該映像。 請注意，因為您的作業系統 VHD 已一般化，所以您無法藉由提供 VHD URL 來直接部署 VM。

若要深入了解 VM 映像，請檢閱下列部落格文章：

* [VM 映像](https://azure.microsoft.com/blog/vm-image-blog-post/)
* [VM 映像 PowerShell 如何](https://azure.microsoft.com/blog/vm-image-powershell-how-to-blog-post/)
* [關於 Azure 中的 VM 映像](https://msdn.microsoft.com/library/azure/dn790290.aspx)

### <a name="set-up-the-necessary-tools-powershell-and-azure-classic-cli"></a>設定必要工具 (PowerShell 與 Azure 傳統 CLI)
* [如何設定 PowerShell](/powershell/azure/overview)
* [如何設定 Azure 傳統 CLI](../cli-install-nodejs.md)

### <a name="41-create-a-user-vm-image"></a>4.1 建立使用者 VM 映像
#### <a name="capture-vm"></a>擷取 VM
如需使用 API/PowerShell/Azure CLI 擷取 VM 的相關指引，請閱讀下面指定的連結。

* [API](https://msdn.microsoft.com/library/mt163560.aspx)
* [PowerShell](../virtual-machines/windows/capture-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Azure CLI](../virtual-machines/linux/capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="generalize-image"></a>一般化映像
如需使用 API/PowerShell/Azure CLI 擷取 VM 的相關指引，請閱讀下面指定的連結。

* [API](https://msdn.microsoft.com/library/mt269439.aspx)
* [PowerShell](../virtual-machines/windows/capture-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Azure CLI](../virtual-machines/linux/capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="42-deploy-a-vm-from-a-user-vm-image"></a>4.2 從使用者 VM 映像部署 VM
若要從使用者 VM 映像部署 VM，您可以使用最新的 [Azure 入口網站](https://manage.windowsazure.com) 或 PowerShell。

**從最新的 Azure 入口網站部署 VM**

1. 移至 [新增] > [計算] > [虛擬機器] > [從資源庫]。

2. 移至 [我的映像] ，然後選取要從其中部署 VM 的 VM 映像：

   1. 密切注意您選取的映像，因為 [我的映像]  檢視會同時列出作業系統映像和 VM 映像。
   2. 查看磁碟數目有助於判斷您正在部署的映像類型，因為大部分的 VM 映像都具有一個以上的磁碟。 然而，VM 映像還是可能只有一個作業系統磁碟，然後會將 [磁碟數目]  設定為 1。

      ![繪圖][img-manage-vm-select]
3. 遵循 VM 建立精靈，並指定 VM 名稱、VM 大小、位置、使用者名稱和密碼。

**從 PowerShell 部署 VM**

若要從剛才建立的一般化 VM 映像部署大型 VM，您可使用下列 Cmdlet。

    $img = Get-AzureVMImage -ImageName "myVMImage"
    $user = "user123"
    $pass = "adminPassword123"
    $myVM = New-AzureVMConfig -Name "VMImageVM" -InstanceSize "Large" -ImageName $img.ImageName | Add-AzureProvisioningConfig -Windows -AdminUsername $user -Password $pass
    New-AzureVM -ServiceName "VMImageCloudService" -VMs $myVM -Location "West US" -WaitForBoot

> [!IMPORTANT]
> 如需其他協助，請參閱 [針對 VHD 建立常見問題進行疑難排解]。
>
>

## <a name="5-obtain-certification-for-your-vm-image"></a>5.取得 VM 映像的認證
為您的 VM 映像做好進入 Azure Marketplace 之準備的下一個步驟是為它取得認證。

這個程序包含執行特殊的認證工具、將認證結果上傳至您的 VHD 所在的 Azure 容器、加入供應項目、定義您的 SKU、以及提交您的 VM 映像以取得認證。

### <a name="51-download-and-run-the-certification-test-tool-for-azure-certified"></a>5.1 下載並執行適用於 Azure 認證的認證測試工具
此認證工具會在運作中的 VM (從您的使用者 VM 映像佈建) 上執行，以確保 VM 映像與 Microsoft Azure 相容。 它將驗證是否已符合為 VHD 做準備的指導和需求。 工具的輸出是相容性報告，應該在要求憑證時於發佈入口網站上傳。

認證工具可使用於 Windows 和 Linux VM。 它會透過 PowerShell 連接至 Windows 型 VM，並透過 SSH.Net 連接至 Linux VM：

1. 首先，請在 [Microsoft 下載網站][link-msft-download]下載認證工具。
2. 開啟認證工具，然後按一下 [啟動新測試]  按鈕。
3. 從 [測試資訊]  畫面，輸入執行測試的名稱。
4. 選擇您的 VM 位於 Linux 或 Windows。 根據您的選擇，選取後續選項。

### <a name="connect-the-certification-tool-to-a-linux-vm-image"></a>**將認證工具連接到 Linux VM 映像**
1. 選取 SSH 驗證模式：密碼或金鑰檔案。
2. 如果使用以密碼為基礎的驗證，請輸入網域名稱系統 (DNS) 名稱、使用者名稱和密碼。
3. 如果使用金鑰檔案驗證，請輸入 DNS 名稱、使用者名稱和私密金鑰位置。

   ![Linux VM 映像的密碼驗證][img-cert-vm-pswd-lnx]

   ![Linux VM 映像的金鑰檔案驗證][img-cert-vm-key-lnx]

### <a name="connect-the-certification-tool-to-a-windows-based-vm-image"></a>**將認證工具連接到 Windows 型 VM 映像**
1. 輸入完整 VM DNS 名稱 (例如 MyVMName.Cloudapp.net)。
2. 輸入使用者名稱和密碼。

   ![Windows VM 映像的密碼驗證][img-cert-vm-pswd-win]

在您選取了 Linux 型或 Windows 型 VM 映像的正確選項之後，請選取 [測試連接]  以確認 SSH.Net 或 PowerShell 是否具有有效的連接以供測試用途。 建立連接之後，選取 [下一步]  啟動測試。

測試完成時，您將收到每個測試項目的結果 (通過/失敗/警告)。

![Linux VM 映像的測試案例][img-cert-vm-test-lnx]

![Windows VM 映像的測試案例][img-cert-vm-test-win]

如果任何測試失敗，您的映像就不會獲得認證。 如果發生這種情況，請檢閱需求並進行任何必要的變更。

在自動化測試之後，系統會要求您透過問卷螢幕提供您的 VM 映像的額外輸入。  完成問題，然後選取 [下一步] 。

![認證工具問卷][img-cert-vm-questionnaire]

![認證工具問卷][img-cert-vm-questionnaire-2]

在您完成問卷調查之後，您可以提供其他資訊，例如 Linux VM 映像的 SSH 存取資訊以及失敗評估的說明。 除了問卷調查的答案以外，您可以下載已執行測試案例的測試結果和記錄檔。 將結果儲存在與您的 VHD 相同的容器中。

![儲存認證測試結果][img-cert-vm-results]

### <a name="52-get-the-shared-access-signature-uri-for-your-vm-images"></a>5.2 取得 VM 映像的共用存取簽章 URI
在發行期間，您會指定統一資源識別項 (URI)，這些 URI 會指向您為 SKU 所建立的每個 VHD。 Microsoft 需要在認證程序期間存取這些 VHD。 因此，您必須建立每個 VHD 的共用存取簽章 URI。 此為要在發佈入口網站的 [映像]  標籤中輸入的 URI。

建立的共用存取簽章 URI 應符合下列需求：

注意：以下指示只適用於非受控磁碟，也是唯一支援的一種。

* 為 VHD 產生共用存取簽章 URI 時，必須有足夠的列出和讀取權限。 不提供寫入或刪除存取權。
* 存取期間應為建立共用存取簽章 URI 起至少三個 (3) 星期。
* 為了確保使用 UTC 時間，請選取目前日期之前的日期。 例如，如果目前日期為 2014 年 10 月 6 日，則選取 10/5/2014。

有多種方式可產生 SAS URL，以便將您的 VHD 分享到 Azure Marketplace。
以下是 3 個建議的工具︰

1.  Azure 儲存體總管
2.  Microsoft 儲存體總管
3.  Azure CLI

**Azure 儲存體總管 (建議 Windows 使用者採用)**

以下是關於使用 Azure 儲存體總管產生 SAS URL 的步驟

1. 從 CodePlex 下載 [Azure 儲存體總管 6 預覽 3](https://azurestorageexplorer.codeplex.com/)。 移至 [Azure 儲存體總管 6 預覽](https://azurestorageexplorer.codeplex.com/)，然後按 [下載]。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_01.png)

2. 下載 [AzureStorageExplorer6Preview3.zip](https://azurestorageexplorer.codeplex.com/downloads/get/891668)，解壓縮之後安裝。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_02.png)

3. 安裝之後，請開啟應用程式。
4. 按一下 [加入帳戶] 。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_03.png)

5. 指定儲存體帳戶名稱、儲存體帳戶金鑰和儲存體端點網域。 這是您的 Azure 訂用帳戶中的儲存體帳戶，其中保存您在 Azure 入口網站上的 VHD。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_04.png)

6. Azure 儲存體總管連接至特定的儲存體帳戶之後，就會開始顯示所有儲存體帳戶內的容器。 選取您已複製作業系統磁碟 VHD 檔案 (如果適用於您的案例，同時也複製資料磁碟) 所在的容器。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_05.png)

7. 選取 Blob 容器之後，Azure 儲存體總管會開始顯示容器內的檔案。 選取需要提交的影像檔案 (.vhd)。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_06.png)

8.  選取容器中的 .vhd 檔案之後，按一下 [安全性]  索引標籤。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_07.png)

9.  在 [Blob 容器安全性] 對話方塊中，保留 [存取層級] 索引標籤上的預設值，然後按一下 [共用存取簽章] 索引標籤。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_08.png)

10. 遵循下列步驟來產生 .vhd 映像的共用存取簽章 URI：

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_09.png)

    a. **允許存取開始日期**：為了確保使用 UTC 時間，請選取目前日期之前的日期。 例如，如果目前日期為 2014 年 10 月 6 日，則選取 10/5/2014。

    b. **允許存取結束日期**：選取至少在 [允許存取開始日期] 之後 3 個星期的日期。

    c. **允許動作**：選取 [列出] 和 [讀取] 權限。

    d. 如果您已正確選取 .vhd 檔案，則您的檔案會出現在 [要存取的 Blob 名稱] 中且副檔名為 .vhd。

    e. 按一下 [產生簽章]。

    f. 在 [此容器產生的共用存取簽章 URI] 中，在下列項目中檢查上方反白顯示的項目：

       - 確定您的映像檔案名稱和 **".vhd"** 位於 URI 中。
       - 確定 **"= rl"** 出現在簽章的結尾。 這表明已成功提供 [讀取] 和 [列出] 存取權。
       - 確定 **"sr=c"** 出現在簽章的中間。 這示範您具有容器層級存取

11. 若要確認產生的共用存取簽章 URI 有效，請按一下 [在瀏覽器中測試]。 應該會啟動下載程序。

12. 複製共用存取簽章 URI。 此為要貼入發佈入口網站的 URI。

13. 針對 SKU 中的每個 VHD 重複步驟 6-10。

**Microsoft Azure 儲存體總管 (Windows/MAC/Linux)**

以下是關於使用 Microsoft Azure 儲存體總管產生 SAS URL 的步驟

1.  從 [http://storageexplorer.com/](http://storageexplorer.com/) 網站下載 Microsoft Azure 儲存體總管。 移至 [Microsoft Azure 儲存體總管](http://storageexplorer.com/releasenotes.html)，然後按 [下載 Windows 版]。

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_10.png)

2.  安裝之後，請開啟應用程式。

3.  按一下 [加入帳戶] 。

4.  登入帳戶將 Microsoft Azure 儲存體總管設定給您的訂用帳戶

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_11.png)

5.  移至儲存體帳戶，選取 [容器]

6.  選取 [取得共用存取簽章...] \(以滑鼠右鍵按一下 [容器])

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_12.png)

7.  根據下列指示，更新 [開始時間]、[到期時間] 和 [權限]

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_13.png)

    a.  **開始時間：** 為了確保使用 UTC 時間，請選取目前日期之前的日期。 例如，如果目前日期為 2014 年 10 月 6 日，則選取 10/5/2014。

    b.  **到期時間︰** 選取至少在 [開始時間] 日期之後 3 個星期的日期。

    c.  **權限：**：選取 [列出] 和 [讀取] 權限

8.  複製容器共用存取簽章 URI

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_14.png)

    產生的 SAS URL 是容器層級，我們現在需要在其中新增 VHD 名稱。

    容器層級 SAS URL 的格式︰`https://testrg009.blob.core.windows.net/vhds?st=2016-04-22T23%3A05%3A00Z&se=2016-04-30T23%3A05%3A00Z&sp=rl&sv=2015-04-05&sr=c&sig=J3twCQZv4L4EurvugRW2klE2l2EFB9XyM6K9FkuVB58%3D`

    在 SAS URL 中的容器名稱之後插入 VHD 名稱，如下所示 `https://testrg009.blob.core.windows.net/vhds/<VHD NAME>?st=2016-04-22T23%3A05%3A00Z&se=2016-04-30T23%3A05%3A00Z&sp=rl&sv=2015-04-05&sr=c&sig=J3twCQZv4L4EurvugRW2klE2l2EFB9XyM6K9FkuVB58%3D`

    範例：

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_15.png)

    TestRGVM201631920152.vhd 是 VHD 的名稱，所以 VHD SAS URL 會是 `https://testrg009.blob.core.windows.net/vhds/TestRGVM201631920152.vhd?st=2016-04-22T23%3A05%3A00Z&se=2016-04-30T23%3A05%3A00Z&sp=rl&sv=2015-04-05&sr=c&sig=J3twCQZv4L4EurvugRW2klE2l2EFB9XyM6K9FkuVB58%3D`

    - 確定您的映像檔案名稱和 **".vhd"** 位於 URI 中。
    - 確定 **"sp=rl"** 出現在簽章的中間。 這表明已成功提供 [讀取] 和 [列出] 存取權。
    - 確定 **"sr=c"** 出現在簽章的中間。 這示範您具有容器層級存取

9.  若要確認產生的共用存取簽章 URI 有效，請在瀏覽器中測試。 應該會啟動下載程序

10. 複製共用存取簽章 URI。 此為要貼入發佈入口網站的 URI。

11. 為 SKU 中的每個 VHD 重複這些步驟。

**Azure 傳統 CLI (建議用於非 Windows 與持續整合)**

以下是關於使用 Azure 傳統 CLI 產生 SAS URL 的步驟

[!INCLUDE [outdated-cli-content](../../includes/contains-classic-cli-content.md)]

1.  從[這裡](https://azure.microsoft.com/documentation/articles/xplat-cli-install/)下載 Azure 傳統 CLI。 您也可以找到適用於 **[Windows](http://aka.ms/webpi-azure-cli)** 和 **[MAC OS](http://aka.ms/mac-azure-cli)** 的不同連結。

2.  下載之後，請安裝

3.  使用下列程式碼建立 PowerShell (或其他指令碼可執行檔) 檔案，並將它儲存在本機

          $conn="DefaultEndpointsProtocol=https;AccountName=<StorageAccountName>;AccountKey=<Storage Account Key>"
          azure storage container list vhds -c $conn
          azure storage container sas create vhds rl <Permission End Date> -c $conn --start <Permission Start Date>  

    在上面的程式碼中，更新下列參數

    a. **`<StorageAccountName>`**：提供您的儲存體帳戶名稱

    b. **`<Storage Account Key>`**：提供您的儲存體帳戶金鑰

    c. **`<Permission Start Date>`**：為了確保使用 UTC 時間，請選取目前日期之前的日期。 例如，如果目前日期為 2016 年 10 月 26 日，則值應該為 10/25/2016。 如果使用 Azure CLI 2.0 版或更高版本，請在 [開始日期] 和 [結束日期] 中提供日期和時間，例如：10-25-2016T00:00:00Z。

    d. **`<Permission End Date>`**：選取至少在 [開始日期] 之後 3 個星期的日期。 此值應該是 **11/02/2016**。 如果使用 Azure CLI 2.0 版或更高版本，請在 [開始日期] 和 [結束日期] 中提供日期和時間，例如：11-02-2016T00:00:00Z。

    以下是更新適當參數之後的範例程式碼

          $conn="DefaultEndpointsProtocol=https;AccountName=st20151;AccountKey=TIQE5QWMKHpT5q2VnF1bb+NUV7NVMY2xmzVx1rdgIVsw7h0pcI5nMM6+DVFO65i4bQevx21dmrflA91r0Vh2Yw=="
          azure storage container list vhds -c $conn
          azure storage container sas create vhds rl 11/02/2016 -c $conn --start 10/25/2016  

4.  使用「以系統管理員身分執行」模式下開啟 Powershell 編輯器，並在步驟 #3 中開啟檔案。 您可以使用作業系統上可取得的任何指令碼編輯器。

5.  執行指令碼，它會提供容器層級存取的 SAS URL 給您

    以下是 SAS 簽章的輸出，請將醒目提示的部分複製到記事本

    ![繪圖](media/marketplace-publishing-vm-image-creation/img5.2_16.png)

6.  現在您會取得容器層級 SAS URL，您需要在其中新增 VHD 名稱。

    容器等級 SAS URL #

    `https://st20151.blob.core.windows.net/vhds?st=2016-10-25T07%3A00%3A00Z&se=2016-11-02T07%3A00%3A00Z&sp=rl&sv=2015-12-11&sr=c&sig=wnEw9RfVKeSmVgqDfsDvC9IHhis4x0fc9Hu%2FW4yvBxk%3D`

7.  在 SAS URL 中的容器名稱之後插入 VHD 名稱，如下所示 `https://st20151.blob.core.windows.net/vhds/<VHDName>?st=2016-10-25T07%3A00%3A00Z&se=2016-11-02T07%3A00%3A00Z&sp=rl&sv=2015-12-11&sr=c&sig=wnEw9RfVKeSmVgqDfsDvC9IHhis4x0fc9Hu%2FW4yvBxk%3D`

    範例：

    TestRGVM201631920152.vhd 是 VHD 的名稱，所以 VHD SAS URL 會是

    `https://st20151.blob.core.windows.net/vhds/ TestRGVM201631920152.vhd?st=2016-10-25T07%3A00%3A00Z&se=2016-11-02T07%3A00%3A00Z&sp=rl&sv=2015-12-11&sr=c&sig=wnEw9RfVKeSmVgqDfsDvC9IHhis4x0fc9Hu%2FW4yvBxk%3D`

    - 確定您的映像檔案名稱和 ".vhd" 位於 URI 中。
    -   確定 "sp=rl" 出現在簽章的中間。 這表明已成功提供 [讀取] 和 [列出] 存取權。
    -   確定 "sr=c" 出現在簽章的中間。 這示範您具有容器層級存取

8.  若要確認產生的共用存取簽章 URI 有效，請在瀏覽器中測試。 應該會啟動下載程序

9.  複製共用存取簽章 URI。 此為要貼入發佈入口網站的 URI。

10. 為 SKU 中的每個 VHD 重複這些步驟。


### <a name="53-provide-information-about-the-vm-image-and-request-certification-in-the-publishing-portal"></a>5.3 在發佈入口網站中提供 VM 映像和要求認證的相關資訊。
在您已建立供應項目和 SKU 之後，您應該輸入和該 SKU 相關聯的映像詳細資料：

1. 移至[發佈入口網站][link-pubportal]，然後以您的賣方帳戶登入。
2. 選取 [VM 映像]  索引標籤。
3. 列在頁面頂端的識別碼其實是供應項目識別碼，而不是 SKU 識別碼。
4. 在 [SKU]  區段中填入屬性。
5. 在 [作業系統系列] 下，選取與作業系統 VHD 相關聯的作業系統類型。
6. 在 [作業系統]  方塊中，描述作業系統。 請考慮使用作業系統系列、類型、版本和更新等格式。 其中一個範例為 "Windows Server Datacenter 2014 R2"。
7. 最多選取六個建議的虛擬機器大小。 當客戶決定購買與部署您的映像時，這些大小是在 Azure 入口網站的 [定價層] 刀鋒視窗中對其顯示的建議大小。 **這些只是建議大小。客戶可以選取任何可容納您映像中指定之磁碟的 VM 大小。**
8. 輸入版本。 [版本] 欄位會封裝語意版本來識別產品及其更新：
   * 版本格式應該是 X.Y.Z，其中 X、Y 和 Z 是整數。
   * 不同 SKU 中的映像可以有不同的主要和次要版本。
   * SKU 中的版本應該是累加變更，增加修補程式版本 (X.Y.Z 中的 Z)。
9. 在 [OS VHD URL]  方塊中輸入為作業系統 VHD 建立的共用存取簽章 URI。
10. 如果有和這個 SKU 相關聯的資料磁碟，請選取您希望在部署時於其中裝載這個資料磁碟的邏輯單元編號 (LUN)。
11. 在 [LUN X VHD URL]  方塊中輸入為第一個資料 VHD 建立的共用存取簽章 URI。

    ![繪圖](media/marketplace-publishing-vm-image-creation/vm-image-pubportal-skus-3.png)


## <a name="common-sas-url-issues--fixes"></a>常見的 SAS URL 問題和修正

|問題|失敗訊息|修正|文件連結|
|---|---|---|---|
|複製映像失敗 - 在 SAS url 中找不到 "?"|失敗︰複製映像。 無法使用提供的 SAS Uri 下載 blob。|使用建議的工具更新 SAS Url|[https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)|
|映像複製失敗 - SAS url 中未設定 “st” 和 “se” 參數|失敗︰複製映像。 無法使用提供的 SAS Uri 下載 blob。|更新 SAS Url，必須包含開始和結束日期|[https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)|
|複製映像失敗 - SAS url 中沒有 “sp=rl”|失敗︰複製映像。 無法使用提供的 SAS Uri 下載 blob|更新 SAS Url，將權限設定為「讀取」和「列出」|[https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)|
|複製映像失敗 - SAS url 中的 vhd 名稱含有空格|失敗︰複製映像。 無法使用提供的 SAS Uri 下載 blob。|更新 SAS Url，不能含有空格|[https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)|
|複製映像失敗 – SAS Url 授權錯誤的|失敗︰複製映像。 因為發生授權錯誤，無法下載 blob|重新產生 SAS Url|[https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)|
|複製映像失敗 – SAS Url "st" 和 "se" 參數沒有完整的日期時間規格|失敗︰複製映像。 因為 SAS Url 不正確，無法下載 blob |SAS Url 的開始和結束日期參數 ("st"、"se") 必須有完整的日期時間規格，例如 11-02-2017T00:00:00Z，而不能只有日期或縮短的時間版本。 使用 Azure CLI 2.0 版或更高版本，可能會遇到此狀況。 請務必提供完整的日期時間規格，並重新產生 SAS Url。|[https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)|

## <a name="next-step"></a>後續步驟
完成 SKU 詳細資料之後，您可以移至 [Azure Marketplace 行銷內容指南][link-pushstaging]。 在發佈程序的該步驟中，您會在 **步驟 3：在預備環境中測試您的 VM 供應項目**之前提供行銷內容、價格和其他必要資訊，而您會在該步驟中測試各種使用案例，然後再將供應項目部署到 Azure Marketplace 以供公開查看和購買。  

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

[img-acom-1]:media/marketplace-publishing-vm-image-creation/vm-image-acom-datacenter.png
[img-portal-vm-size]:media/marketplace-publishing-vm-image-creation/vm-image-portal-size.png
[img-portal-vm-create]:media/marketplace-publishing-vm-image-creation/vm-image-portal-create-vm.png
[img-portal-vm-location]:media/marketplace-publishing-vm-image-creation/vm-image-portal-location.png
[img-portal-vm-rdp]:media/marketplace-publishing-vm-image-creation/vm-image-portal-rdp.png
[img-azstg-add]:media/marketplace-publishing-vm-image-creation/vm-image-storage-add.png
[img-manage-vm-new]:media/marketplace-publishing-vm-image-creation/vm-image-manage-new.png
[img-manage-vm-select]:media/marketplace-publishing-vm-image-creation/vm-image-manage-select.png
[img-cert-vm-key-lnx]:media/marketplace-publishing-vm-image-creation/vm-image-certification-keyfile-linux.png
[img-cert-vm-pswd-lnx]:media/marketplace-publishing-vm-image-creation/vm-image-certification-password-linux.png
[img-cert-vm-pswd-win]:media/marketplace-publishing-vm-image-creation/vm-image-certification-password-win.png
[img-cert-vm-test-lnx]:media/marketplace-publishing-vm-image-creation/vm-image-certification-test-sample-linux.png
[img-cert-vm-test-win]:media/marketplace-publishing-vm-image-creation/vm-image-certification-test-sample-win.png
[img-cert-vm-results]:media/marketplace-publishing-vm-image-creation/vm-image-certification-results.png
[img-cert-vm-questionnaire]:media/marketplace-publishing-vm-image-creation/vm-image-certification-questionnaire.png
[img-cert-vm-questionnaire-2]:media/marketplace-publishing-vm-image-creation/vm-image-certification-questionnaire-2.png
[img-pubportal-vm-skus]:media/marketplace-publishing-vm-image-creation/vm-image-pubportal-skus.png
[img-pubportal-vm-skus-2]:media/marketplace-publishing-vm-image-creation/vm-image-pubportal-skus-2.png

[link-pushstaging]:marketplace-publishing-push-to-staging.md
[link-github-waagent]:https://github.com/Azure/WALinuxAgent
[link-azure-codeplex]:https://azurestorageexplorer.codeplex.com/
[link-azure-2]:../storage/blobs/storage-dotnet-shared-access-signature-part-2.md
[link-azure-1]:../storage/common/storage-dotnet-shared-access-signature-part-1.md
[link-msft-download]:http://www.microsoft.com/download/details.aspx?id=44299
[link-technet-3]:https://technet.microsoft.com/library/hh846766.aspx
[link-technet-2]:https://msdn.microsoft.com/library/dn495261.aspx
[link-azure-portal]:https://portal.azure.com
[link-pubportal]:https://publish.windowsazure.com
[link-acct-creation]:marketplace-publishing-accounts-creation-registration.md
[link-technet-1]:https://technet.microsoft.com/library/hh848454.aspx
[link-azure-vm-2]:./virtual-machines-linux-agent-user-guide/
[link-openssl]:https://www.openssl.org/
[link-intsvc]:http://www.microsoft.com/download/details.aspx?id=41554
[link-python]:https://www.python.org/
