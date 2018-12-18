---
title: 進行從 VMware 到 Azure 的 Azure Site Recovery 疑難排解 | Microsoft Docs
description: 針對複寫 Azure 虛擬機器時的錯誤進行疑難排解。
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.author: ramamill
ms.date: 07/06/2018
ms.openlocfilehash: 8d5db03eeebb659414ea1f554e5b34c938fd2795
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37952904"
---
# <a name="troubleshoot-mobility-service-push-installation-issues"></a>針對行動服務推送安裝問題進行疑難排解

本文說明當您嘗試在來源伺服器上安裝 Azure Site Recovery 行動服務以啟用保護時，如何針對可能會面臨的常見問題進行疑難排解。

## <a name="error-78007---the-requested-operation-could-not-be-completed"></a>錯誤 78007：無法完成所要求的作業
有許多原因會使服務擲回此錯誤。 請選擇相對應的提供者錯誤以進行進一步的疑難排解。

* [錯誤 95103](#error-95103---protection-could-not-be-enabled-ep0854) 
* [錯誤 95105](#error-95105---protection-could-not-be-enabled-ep0856) 
* [錯誤 95107](#error-95107---protection-could-not-be-enabled-ep0858) 
* [錯誤 95108](#error-95108---protection-could-not-be-enabled-ep0859) 
* [錯誤 95117](#error-95117---protection-could-not-be-enabled-ep0865) 
* [錯誤 95213](#error-95213---protection-could-not-be-enabled-ep0874) 
* [錯誤 95224](#error-95224---protection-could-not-be-enabled-ep0883) 
* [錯誤 95265](#error-95265---protection-could-not-be-enabled-ep0902) 


## <a name="error-95105---protection-could-not-be-enabled-ep0856"></a>錯誤 95105：無法啟用保護 (EP0856)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95105 </br>**訊息：** 將行動服務推送安裝到來源機器失敗，錯誤碼為 **EP0856**。 <br> 可能是來源電腦不允許**檔案及印表機共用**，或是處理序伺服器與來源電腦之間有網路連線問題。| 未啟用 [檔案及印表機共用]。 | 在 Windows 防火牆中的來源電腦允許 [檔案及印表機共用]。 在來源電腦的 [Windows 防火牆]  >  [允許應用程式或功能通過防火牆]，為所有設定檔選取 [檔案及印表機共用]。 </br> 此外，請檢查下列必要條件，才能順利完成推送安裝。<br> 深入了解[針對 WMI 問題進行疑難排解](#troubleshoot-wmi-issues)。


## <a name="error-95107---protection-could-not-be-enabled-ep0858"></a>錯誤 95107：無法啟用保護 (EP0858)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95107 </br>**訊息：** 將行動服務推送安裝到來源機器失敗，錯誤碼為 **EP0858**。 <br> 可能是所提供用以安裝行動服務的認證不正確，或使用者帳戶的權限不足。 | 為了在來源機器上安裝行動服務所提供的使用者認證不正確。 | 請確定組態伺服器上針對來源機器提供的使用者認證正確無誤。 <br> 若要新增或編輯使用者認證，請前往組態伺服器，選取 [Cspsconfigtool] > [管理帳戶]。 </br> 此外，請檢查下列[必要條件](vmware-azure-install-mobility-service.md#install-mobility-service-by-push-installation-from-azure-site-recovery)，以順利完成推送安裝。


## <a name="error-95117---protection-could-not-be-enabled-ep0865"></a>錯誤 95117：無法啟用保護 (EP0865)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95117 </br>**訊息：** 將行動服務推送安裝到來源機器失敗，錯誤碼為 **EP0865**。 <br> 可能是來源電腦不在執行中，或是處理序伺服器與來源電腦之間有網路連線問題。 | 處理序伺服器與來源伺服器之間的網路連線問題。 | 請檢查處理序伺服器與來源伺服器之間的網路連線。 </br> 此外，請檢查下列[必要條件](vmware-azure-install-mobility-service.md#install-mobility-service-by-push-installation-from-azure-site-recovery)，以順利完成推送安裝。|

## <a name="error-95103---protection-could-not-be-enabled-ep0854"></a>錯誤 95103：無法啟用保護 (EP0854)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95103 </br>**訊息：** 將行動服務推送安裝到來源機器失敗，錯誤碼為 **EP0854**。 <br> 可能是來源電腦不允許 Windows Management Instrumentation (WMI) ，或是處理序伺服器與來源電腦之間有網路連線問題。| Windows 防火牆會封鎖 WMI。 | 在 Windows 防火牆允許 WMI。 在 [Windows 防火牆]  >  [允許應用程式或功能通過防火牆]，為所有設定檔選取 [WMI]。 </br> 此外，請檢查下列[必要條件](vmware-azure-install-mobility-service.md#install-mobility-service-by-push-installation-from-azure-site-recovery)，以順利完成推送安裝。|

## <a name="error-95213---protection-could-not-be-enabled-ep0874"></a>錯誤 95213：無法啟用保護 (EP0874)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95213 </br>**訊息：** 將行動服務安裝到來源機器 %SourceIP; 失敗，錯誤碼為 **EP0874**。 <br> | 不支援來源機器上的作業系統版本。 <br>| 請確定來源機器的 OS 版本為受支援的版本。 請參閱[支援矩陣](https://aka.ms/asr-os-support)。 </br> 此外，請檢查下列[必要條件](https://aka.ms/pushinstallerror)，以順利完成推送安裝。| 


## <a name="error-95108---protection-could-not-be-enabled-ep0859"></a>錯誤 95108：無法啟用保護 (EP0859)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95108 </br>**訊息：** 將行動服務推送安裝到來源機器失敗，錯誤碼為 **EP0859**。 <br>| 可能是所提供用以安裝行動服務的認證不正確，或使用者帳戶的權限不足。 <br>| 請確定所提供的認證為 **root** 帳戶的認證。 若要新增或編輯使用者認證，請移至組態伺服器，然後選取桌面上的 [Cspsconfigtool] 捷徑圖示。 按一下 [管理帳戶] 以新增或編輯認證。|

## <a name="error-95265---protection-could-not-be-enabled-ep0902"></a>錯誤 95265：無法啟用保護 (EP0902)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95265 </br>**訊息：** 將行動服務推送安裝到來源機器成功，但必須先重新啟動來源機器，部分系統變更才會生效。 <br>| 較舊版本的行動服務已經安裝於伺服器上。| 虛擬機器的複寫會順暢地繼續。<br> 在下一個維護時間窗口重新啟動伺服器，以取得由行動服務的新增強功能所帶來的好處。|


## <a name="error-95224---protection-could-not-be-enabled-ep0883"></a>錯誤 95224：無法啟用保護 (EP0883)

**錯誤碼** | **可能的原因** | **特定錯誤的建議**
--- | --- | ---
95224 </br>**訊息：** 將行動服務推送安裝到來源機器 %SourceIP; 失敗，錯誤碼為 **EP0883**。 有源自先前安裝或更新的系統重新啟動處於擱置狀態。| 系統在將較舊或不相容的行動服務版本解除安裝時，並沒有重新啟動。| 請確定伺服器上不存在任何版本的行動服務。 <br> 請將伺服器重新啟動，並重新執行啟用保護作業。|

## <a name="resource-to-troubleshoot-push-installation-problems"></a>針對推送安裝問題進行疑難排解的資源

#### <a name="troubleshoot-file-and-print-sharing-issues"></a>針對檔案與列印共用問題進行疑難排解
* [透過群組原則啟用或停用檔案共用](https://technet.microsoft.com/library/cc754359(v=ws.10).aspx)
* [如何透過 Windows 防火牆啟用檔案與列印共用](https://technet.microsoft.com/library/ff633412(v=ws.10).aspx)

#### <a name="troubleshoot-wmi-issues"></a>針對 WMI 問題進行疑難排解
* [基本 WMI 測試](https://blogs.technet.microsoft.com/askperf/2007/06/22/basic-wmi-testing/)
* [WMI 疑難排解](https://msdn.microsoft.com/library/aa394603(v=vs.85).aspx)
* [WMI 指令碼和 WMI 服務問題的疑難排解](https://technet.microsoft.com/library/ff406382.aspx#H22)

## <a name="next-steps"></a>後續步驟

[了解如何](vmware-azure-tutorial.md)為 VMware VM 設定災害復原。