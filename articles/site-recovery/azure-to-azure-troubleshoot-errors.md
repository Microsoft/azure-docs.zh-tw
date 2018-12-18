---
title: Azure 至 Azure 複寫問題和錯誤的 Azure Site Recovery 疑難排解 | Microsoft Docs
description: 對於複寫 Azure 虛擬機器進行嚴重損壞修復時發生的錯誤和問題進行疑難排解
services: site-recovery
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.date: 08/09/2018
ms.author: sujayt
ms.openlocfilehash: 86d6c77dab817cf755c34bdd699ee1158e852f37
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42141023"
---
# <a name="troubleshoot-azure-to-azure-vm-replication-issues"></a>Azure 至 Azure VM 複寫問題的疑難排解

本文說明從一個區域將 Azure 虛擬機器複寫和復原到另一個區域時，關於 Azure Site Recovery 的常見問題，並解說如何進行疑難排解。 如需受支援組態的詳細資訊，請參閱[複寫 Azure VM 的支援矩陣](site-recovery-support-matrix-azure-to-azure.md)。

## <a name="azure-resource-quota-issues-error-code-150097"></a>Azure 資源配額問題 (錯誤碼 150097)
您的訂用帳戶應該啟用，才能在要做為災害復原區域的目標區域中建立 Azure VM。 此外，您的訂用帳戶應該已啟用足夠的配額，才能建立特定大小的 VM。 根據預設，Site Recovery 會取用相同大小的目標 VM 做為來源 VM。 如果沒有相符大小，系統會自動挑選最接近的大小。 如果沒有支援來源 VM 組態的相符大小，則會出現下列錯誤訊息：

**錯誤碼** | **可能的原因** | **建議**
--- | --- | ---
150097<br></br>**訊息**：虛擬機器 VmName 無法啟用複寫。 | - 可能未啟用您的訂用帳戶 ID 在目標區域位置中建立任何 VM。</br></br>- 可能未啟用您的訂用帳戶 ID 或沒有足夠的配額可在目標區域位置中建立特定 VM 大小。</br></br>- 對於目標區域位置中的訂用帳戶 ID，找不到符合來源 VM NIC 計數 (2) 的適當 VM 大小。| 對於訂用帳戶的目標位置之中的所需 VM 大小，請連絡 [Azure 計費支援](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request)啟用 VM。 啟用後，請重試失敗的作業。

### <a name="fix-the-problem"></a>修正問題
您可以連絡 [Azure 計費支援](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request)啟用訂用帳戶，在目標位置中建立所需大小的 VM。

如果目標位置有容量限制，請停用複寫，並對於訂用帳戶有足夠配額的不同位置啟用複寫，以建立所需大小的 VM。

## <a name="trusted-root-certificates-error-code-151066"></a>受信任的根憑證 (錯誤碼 151066)

如果 VM 沒有所有最新的受信任根憑證，「啟用複寫」工作可能會失敗。 如果沒有憑證，對於從 VM 的 Site Recovery 服務呼叫進行的驗證和授權會失敗。 失敗的「啟用複寫」Site Recovery 工作的錯誤訊息會出現：

**錯誤碼** | **可能的原因** | **建議**
--- | --- | ---
151066<br></br>**訊息**：Site Recovery 組態失敗。 | 機器上不存在用於授權和驗證的受信任根憑證。 | - 對於執行 Windows 作業系統的 VM，請確定機器上存在受信任根憑證。 如需資訊，請參閱[設定受信任根目錄和不允許的憑證](https://technet.microsoft.com/library/dn265983.aspx)。<br></br>- 對於執行 Linux 作業系統的 VM，請依照 Linux 作業系統版本散發者發行的受信任根憑證適用的指引。

### <a name="fix-the-problem"></a>修正問題
**Windows**

在 VM 上安裝所有最新的 Windows 更新，使機器上存在所有的受信任根憑證。 如果您是在中斷連線的環境，請遵循您組織的標準 Windows 更新程序來取得憑證。 如果 VM 上不存在所需的憑證，對於 Site Recovery 進行的呼叫將而於安全性因素而失敗。

遵循您組織的一般 Windows 更新管理或憑證更新管理程序，取得所有最新的根憑證以及 VM 上更新的憑證撤銷清單。

若要確認問題已解決，請在 VM 中從瀏覽器移至 login.microsoftonline.com。

**Linux**

請遵循 Linux 散發者提供的指引，取得最新的受信任根憑證以及 VM 上最新的憑證撤銷清單。

因為 SuSE Linux 使用符號連結來維護憑證清單，因此請遵循下列步驟：

1.  以根使用者身分登入。

2.  執行這個命令來變更目錄。

      ``# cd /etc/ssl/certs``

3. 檢查是否有 Symantec 根 CA 憑證。

      ``# ls VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

4. 如果找不到 Symantec 根 CA 憑證，請執行下列命令來下載檔案。 檢查是否有任何錯誤，並遵循適用於網路失敗的建議動作。

      ``# wget https://www.symantec.com/content/dam/symantec/docs/other-resources/verisign-class-3-public-primary-certification-authority-g5-en.pem -O VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

5. 檢查是否有 Baltimore 根 CA 憑證。

      ``# ls Baltimore_CyberTrust_Root.pem``

6. 如果找不到 Baltimore 根 CA 憑證，請下載憑證。  

    ``# wget http://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem -O Baltimore_CyberTrust_Root.pem``

7. 檢查是否有 DigiCert_Global_Root_CA 憑證。

    ``# ls DigiCert_Global_Root_CA.pem``

8. 如果找不到 DigiCert_Global_Root_CA，請執行下列命令來下載憑證。

    ``# wget http://www.digicert.com/CACerts/DigiCertGlobalRootCA.crt``

    ``# openssl x509 -in DigiCertGlobalRootCA.crt -inform der -outform pem -out DigiCert_Global_Root_CA.pem``

9. 執行 rehash 指令碼，來更新適用於最新下載憑證的憑證主體雜湊。

    ``# c_rehash``

10. 檢查是否已針對憑證建立主體雜湊作為符號連結。

    - 命令

      ``# ls -l | grep Baltimore``

    - 輸出

      ``lrwxrwxrwx 1 root root   29 Jan  8 09:48 3ad48a91.0 -> Baltimore_CyberTrust_Root.pem
      -rw-r--r-- 1 root root 1303 Jun  5  2014 Baltimore_CyberTrust_Root.pem``

    - 命令

      ``# ls -l | grep VeriSign_Class_3_Public_Primary_Certification_Authority_G5``

    - 輸出

      ``-rw-r--r-- 1 root root 1774 Jun  5  2014 VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem
      lrwxrwxrwx 1 root root   62 Jan  8 09:48 facacbc6.0 -> VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem``

    - 命令

      ``# ls -l | grep DigiCert_Global_Root``

    - 輸出

      ``lrwxrwxrwx 1 root root   27 Jan  8 09:48 399e7759.0 -> DigiCert_Global_Root_CA.pem
      -rw-r--r-- 1 root root 1380 Jun  5  2014 DigiCert_Global_Root_CA.pem``

11. 使用檔案名稱 b204d74a.0 建立 VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem 檔案的複本

    ``# cp VeriSign_Class_3_Public_Primary_Certification_Authority_G5.pem b204d74a.0``

12. 使用檔案名稱 653b494a.0 建立 Baltimore_CyberTrust_Root.pem 檔案的複本

    ``# cp Baltimore_CyberTrust_Root.pem 653b494a.0``

13. 使用檔案名稱 3513523f.0 建立 DigiCert_Global_Root_CA.pem 檔案的複本

    ``# cp DigiCert_Global_Root_CA.pem 3513523f.0``  


14. 檢查這些檔案是否存在。  

    - 命令

      ``# ls -l 653b494a.0 b204d74a.0 3513523f.0``

    - 輸出

      ``-rw-r--r-- 1 root root 1774 Jan  8 09:52 3513523f.0
      -rw-r--r-- 1 root root 1303 Jan  8 09:52 653b494a.0
      -rw-r--r-- 1 root root 1774 Jan  8 09:52 b204d74a.0``


## <a name="outbound-connectivity-for-site-recovery-urls-or-ip-ranges-error-code-151037-or-151072"></a>Site Recovery URL 或 IP 範圍的輸出連線能力 (錯誤碼 151037 或 151072)

若要使 Site Recovery 複寫正常運作，VM 需要特定 URL 或 IP 範圍的輸出連線能力。 如果您的 VM 位於防火牆後方，或使用網路安全性群組 (NSG) 規則控制輸出連線能力，您可能會遇到下列其中一個問題。

### <a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151037-br"></a>問題 1：無法向 Site Recovery 註冊 Azure 虛擬機器 (151037) </br>
- **可能的原因** </br>
  - 您使用 NSG 控制傳出 VM 的存取，而且所需的 IP 範圍並未列在傳出存取的允許清單中。
  - 您使用第三方防火牆工具，而且所需的 IP 範圍/URL 並未列在允許清單中。


- **解決方案**
   - 如果您使用防火牆 Proxy 控制 VM 上的傳出網路連線能力，請確定必要條件 URL 或資料中心 IP 範圍列在允許清單中。 如需資訊，請參閱[防火牆 Proxy 指引](https://aka.ms/a2a-firewall-proxy-guidance)。
   - 如果您使用 NSG 規則控制 VM 上的傳出網路連線能力，請確定必要條件資料中心 IP 範圍列在允許清單中。 如需資訊，請參閱[網路安全性群組指引](https://aka.ms/a2a-nsg-guidance)。
   - 若要將[所需的 URL](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) 或[所需的 IP 範圍](azure-to-azure-about-networking.md#outbound-connectivity-for-ip-address-ranges)列入允許清單中，請依照[網路指引文件](site-recovery-azure-to-azure-networking-guidance.md)中的步驟進行。

### <a name="issue-2-site-recovery-configuration-failed-151072"></a>問題 2：Site Recovery 組態失敗 (151072)
- **可能的原因** </br>
  - 無法在連線至 Site Recovery 服務端點


- **解決方案**
   - 如果您使用防火牆 Proxy 控制 VM 上的傳出網路連線能力，請確定必要條件 URL 或資料中心 IP 範圍列在允許清單中。 如需資訊，請參閱[防火牆 Proxy 指引](https://aka.ms/a2a-firewall-proxy-guidance)。
   - 如果您使用 NSG 規則控制 VM 上的傳出網路連線能力，請確定必要條件資料中心 IP 範圍列在允許清單中。 如需資訊，請參閱[網路安全性群組指引](https://aka.ms/a2a-nsg-guidance)。
   - 若要將[所需的 URL](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) 或[所需的 IP 範圍](azure-to-azure-about-networking.md#outbound-connectivity-for-ip-address-ranges)列入允許清單中，請依照[網路指引文件](site-recovery-azure-to-azure-networking-guidance.md)中的步驟進行。

### <a name="issue-3-a2a-replication-failed-when-the-network-traffic-goes-through-on-premise-proxy-server-151072"></a>問題 3：當網路流量通過內部部署 Proxy 伺服器時，A2A 複寫失敗 (151072)
 - **可能的原因** </br>
   - 自訂 Proxy 設定無效，且 ASR 行動服務代理程式未自動偵測到 IE 的 Proxy 設定


 - **解決方案**
  1.    行動服務代理程式偵測到 IE 的 Proxy 設定 (在 Windows 上) 和 /etc/environment (在 Linux 上)。
  2.  如果您只想要為 ASR 行動服務設定 Proxy，則可在 ProxyInfo.conf 中提供 Proxy 詳細資料，該檔案位於：</br>
      - ``/usr/local/InMage/config/`` (在 ***Linux*** 上)
      - ``C:\ProgramData\Microsoft Azure Site Recovery\Config`` (在 ***Windows*** 上)
  3.    ProxyInfo.conf 應該要有下列 INI 格式的 Proxy 設定。 </br>
                   [proxy]</br>
                   Address=http://1.2.3.4</br>
                   Port=567</br>
  4. ASR 行動服務代理程式僅支援***未驗證的 Proxy***。

### <a name="fix-the-problem"></a>修正問題
若要將[所需的 URL](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) 或[所需的 IP 範圍](azure-to-azure-about-networking.md#outbound-connectivity-for-ip-address-ranges)列入白名單中，請依照[網路指引文件](site-recovery-azure-to-azure-networking-guidance.md)中的步驟進行。

## <a name="disk-not-found-in-the-machine-error-code-150039"></a>在機器中找不到磁碟 (錯誤碼 150039)

必須先將連接到 VM 的新磁碟初始化。

**錯誤碼** | **可能的原因** | **建議**
--- | --- | ---
150039<br></br>**訊息**：邏輯單元編號 (LUN) 為 (LUNValue) 的 Azure 資料磁碟 (DiskName) (DiskURI) 未對應到從 LUN 值相同的 VM 內回報的對應磁碟。 | - 新的資料磁碟連接到 VM，但是未初始化。</br></br>- VM 內的資料磁碟不當回報 LUN 值，該磁碟已連接到 VM。| 請確定資料磁碟都已初始化，然後再次嘗試操作。</br></br>對於 Windows：[連接並初始化新的磁碟](https://docs.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal)。</br></br>對於 Linux：[在 Linux 中初始化新的資料磁碟](https://docs.microsoft.com/azure/virtual-machines/linux/add-disk)。

### <a name="fix-the-problem"></a>修正問題
請確定資料磁碟都已初始化，然後再次嘗試操作：

- 對於 Windows：[連接並初始化新的磁碟](https://docs.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal)。
- 對於 Linux：[在 Linux 中新增資料磁碟](https://docs.microsoft.com/azure/virtual-machines/linux/add-disk)。

若問題持續發生，請連絡支援服務。


## <a name="unable-to-see-the-azure-vm-for-selection-in-enable-replication"></a>看不見 Azure VM，無法在「啟用複寫」中選取

 **原因 1：資源群組和來源虛擬機器位於不同的位置** <br>
Azure Site Recovery 目前要求來源區域資源群組和虛擬機器應該位在相同的位置。 如果情況不是這樣，那麼在保護期間您就無法尋找虛擬機器。

**原因 2：資源群組不在選取的訂用帳戶中** <br>
如果資源群組不是指定的訂用帳戶的一部分，則可能無法在保護時尋找該資源組。 請確定資源群組屬於正在使用的訂用帳戶。

 **原因 3：過時的設定** <br>
如果您未看見想要啟用來進行複寫的 VM，可能是因為 Azure VM 上保留了過時的 Site Recovery 設定。 在下列情況中，Azure VM 可能保留過時的組態：

- 您使用 Site Recovery 啟用 Azure VM 的複寫，然後刪除 Site Recovery 保存庫，但是並未明確停用 VM 上的複寫。
- 您使用 Site Recovery 啟用 Azure VM 的複寫，然後刪除包含 Site Recovery 保存庫的資源群組，但是並未明確停用 VM 上的複寫。

### <a name="fix-the-problem"></a>修正問題

您可以使用[移除過時 ASR 組態指令碼](https://gallery.technet.microsoft.com/Azure-Recovery-ASR-script-3a93f412)，並移除 Azure VM 上的過時 Site Recovery 組態。 在移除過時的設定之後，您應該就能看到該 VM。

## <a name="unable-to-select-virtual-machine-for-protection"></a>無法選取虛擬機器進行保護 
 **原因 1：虛擬機器在失敗或沒有回應的狀態下安裝了某些延伸模組** <br>
 移至 [虛擬機器] > [設定] > [延伸模組]，並檢查是否有任何延伸模組處於失敗狀態。 解除安裝失敗的延伸模組，並重試保護虛擬機器。<br>
 **原因 2：[VM 的佈建狀態無效](#vms-provisioning-state-is-not-valid-error-code-150019)**

## <a name="vms-provisioning-state-is-not-valid-error-code-150019"></a>VM 的佈建狀態無效 (錯誤碼 150019)

若要在 VM 上啟用複寫，佈建狀態應該是「成功」。 您可以藉由遵循下列步驟來檢查 VM 狀態。

1.  在 Azure 入口網站中從 [所有服務] 選取 [資源總管]。
2.  展開 [訂用帳戶] 清單然後選取您的訂用帳戶。
3.  展開 [ResourceGroups] 清單然後選取 VM 的資源群組。
4.  展開 [資源] 清單然後選取您的虛擬機器
5.  勾選右側 [執行個體] 檢視中的 [provisioningState] 欄位。

### <a name="fix-the-problem"></a>修正問題

- 如果 [provisioningState] 是「失敗」，請連絡支援人員以取得疑難排解的詳細資訊。
- 如果 [provisioningState] 是「正在更新」，則可能有其他擴充功能正在部署。 檢查 VM 上是否有任何進行中的作業，等候它們完成然後重試失敗的 Site Recovery **啟用複寫**作業。

## <a name="unable-to-select-target-virtual-network---network-selection-tab-is-grayed-out"></a>無法選取目標虛擬網路 - 網路選取索引標籤呈現灰色。

**原因 1：如果 VM 所連結的網路已對應至「目標網路」。**
- 如果來源 VM 屬於虛擬網路，且來自相同虛擬網路的另一個 VM 已對應至目標資源群組中的網路，則根據預設，系統會停用網路選取下拉式清單。

![Network_Selection_greyed_out](./media/site-recovery-azure-to-azure-troubleshoot/unabletoselectnw.png)

**原因 2︰如果您先前使用 Azure Site Recovery 保護 VM 並停用複寫。**
 - 停用 VM 的複寫不會刪除網路對應。 您必須從受保護 VM 所在的復原服務保存庫中將其刪除。 </br>
 瀏覽至 [復原服務保存庫] > [Site Recovery 基礎結構] > [網路對應]。 </br>
 ![Delete_NW_Mapping](./media/site-recovery-azure-to-azure-troubleshoot/delete_nw_mapping.png)
 - 完成初始設定後，您可在 VM 受到保護後，對災害復原設定期間所設定的目標網路進行變更。 </br>
 ![Modify_NW_mapping](./media/site-recovery-azure-to-azure-troubleshoot/modify_nw_mapping.png)
 - 請注意，變更網路對應會影響所有使用該特定網路對應的受保護 VM。


## <a name="comvolume-shadow-copy-service-error-error-code-151025"></a>COM+/磁碟區陰影複製服務錯誤 (錯誤碼 151025)
**錯誤碼** | **可能的原因** | **建議**
--- | --- | ---
151025<br></br>**訊息**：無法安裝站台復原擴充功能 | - 停用「COM+ 系統應用程式」服務。</br></br>- 停用「磁碟區陰影複製」服務。| 將「COM+ 系統應用程式」和「磁碟區陰影複製」服務設為自動或手動啟動模式。

### <a name="fix-the-problem"></a>修正問題

您可以開啟 [服務] 主控台，確定「COM+ 系統應用程式」和「磁碟區陰影複製」的 [啟動類型] 未設定為 [停用]。
  ![com-error](./media/azure-to-azure-troubleshoot-errors/com-error.png)

## <a name="next-steps"></a>後續步驟
[複寫 Azure 虛擬機器](site-recovery-replicate-azure-to-azure.md)
