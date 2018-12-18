---
title: Azure AD Connect：啟用裝置回寫 | Microsoft Docs
description: 本文詳細說明如何使用 Azure AD Connect 啟用裝置回寫
services: active-directory
documentationcenter: ''
author: billmath
manager: femila
editor: curtand
ms.assetid: c0ff679c-7ed5-4d6e-ac6c-b2b6392e7892
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/08/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: b98b9e0a10f142cb0177e7e5ea831b61f79a4b7a
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304657"
---
# <a name="azure-ad-connect-enabling-device-writeback"></a>Azure AD Connect：啟用裝置回寫
> [!NOTE]
> 需要 Azure AD Premium 的訂用帳戶才能使用裝置回寫功能。
> 
> 

以下文件提供有關在 Azure AD Connect 中啟用裝置回寫功能的資訊。 裝置回寫用於下列案例：

* 對 ADFS (2012 R2 或更高版本) 保護的應用程式 (信賴憑證者信任)，根據裝置啟用條件式存取。

這提供額外的安全性，確保只授權信任的裝置才能存取應用程式。 如需條件式存取的詳細資訊，請參閱[使用條件式存取管理風險](../active-directory-conditional-access-azure-portal.md)和[使用 Azure Active Directory 裝置註冊設定內部部署條件式存取](../active-directory-conditional-access-automatic-device-registration-setup.md)。

> [!IMPORTANT]
> <li>裝置必須位於使用者所在的樹系中。 由於裝置必須回寫到單一樹系中，因此這項功能目前並不支援利用多重使用者樹系的部署。</li>
> <li>只有一個裝置註冊設定物件可以新增至內部部署 Active Directory 樹系。 這項功能與會將內部部署 Active Directory 同步至多個 Azure AD 目錄的拓撲不相容。</li>

## <a name="part-1-install-azure-ad-connect"></a>第 1 部分：安裝 Azure AD Connect
使用自訂或快速設定安裝 Azure AD Connect。 Microsoft 建議您在啟用裝置回寫之前，首先讓所有使用者和群組成功完成同步處理。

## <a name="part-2-enable-device-writeback-in-azure-ad-connect"></a>第 2 部分：在 Azure AD Connect 中啟用裝置回寫功能
1. 再次執行安裝精靈。 選取 [其他工作] 頁面中的 [設定裝置選項]，然後按 [下一步]。 

    ![設定裝置選項](./media/how-to-connect-device-writeback/deviceoptions.png)

    >[!NOTE]
    > 新的設定裝置選項僅適用於 1.1.819.0 版和更新版本。

2. 在裝置選項頁面上，選取 [設定裝置回寫]。 直到啟用裝置回寫以後，才可使用 [停用裝置回寫] 選項。 按 [下一步] 移至精靈的下一頁。
    ![選擇裝置作業](./media/how-to-connect-device-writeback/configuredevicewriteback1.png)

3. 在 [回寫] 頁面中，您會看到提供的網域已成為預設的裝置回寫樹系。
   ![自訂安全裝置回寫樹系](./media/how-to-connect-device-writeback/writebackforest.png)

4. [裝置容器] 頁面使用下列其中一個可用選項來提供準備 Active Directory 的選項：

    a. **提供企業系統管理員認證**：如果針對裝置需要回寫的樹系提供企業系統管理員認證，則 Azure AD Connect 會在裝置回寫的設定期間自動準備樹系。

    b. **下載 PowerShell 指令碼**：Azure AD Connect 會自動產生 PowerShell 指令碼，以便針對裝置回寫準備 Active Directory。 為了避免無法在 Azure AD Connect 中提供企業系統管理員認證，建議下載 PowerShell 指令碼。 將下載的 PowerShell 指令碼 **CreateDeviceContainer.psq** 提供給要回寫裝置之樹系的企業系統管理員。
    ![準備 Active Diretory 樹系](./media/how-to-connect-device-writeback/devicecontainercreds.png)
    
    執行下列作業以準備 Active Directory 樹系：
    * 如果尚未存在，請在 CN=Device Registration Configuration,CN=Services,CN=Configuration,[forest-dn] 下方建立並設定新的容器和物件。
    * 如果尚未存在，請在 CN=RegisteredDevices,[domain-dn] 下方建立並設定新的容器和物件。 將會在此容器中建立裝置物件。
    * 請在 Azure AD Connector 帳戶上設定必要的權限，以便管理 Active Directory 上的裝置。
    * 即使 Azure AD Connect 安裝在多個樹系上，也只需要在一個樹系上執行。

## <a name="verify-devices-are-synchronized-to-active-directory"></a>確認裝置已同步處理至 Active Directory
裝置回寫現在應該正常運作。 請注意，裝置物件寫回至 Active Directory 可能需要 3 小時。  若要確認您的裝置已正確同步化，請在同步化規則完成之後執行下列作業：

1. 啟動 Active Directory 管理中心。
2. 在同盟的網域中，展開 RegisteredDevices。

   ![Active Directory 管理中心已註冊的裝置](./media/how-to-connect-device-writeback/devicewriteback5.png)

3. 此處會列出目前已註冊的裝置。

   ![Active Directory 管理中心已註冊的裝置清單](./media/how-to-connect-device-writeback/devicewriteback6.png)

## <a name="enable-conditional-access"></a>啟用條件式存取
如需如何啟用此案例的詳細指示，請參閱 [使用 Azure Active Directory 裝置註冊設定內部部署條件式存取](../active-directory-conditional-access-automatic-device-registration-setup.md)。

## <a name="troubleshooting"></a>疑難排解
### <a name="the-writeback-checkbox-is-still-disabled"></a>回寫核取方塊仍然停用
如果未啟用裝置回寫的核取方塊，即使您已遵循上述步驟，下列步驟還是會在啟用此方塊之前引導您完成安裝精靈正在驗證的程序。

首先：

* 確定至少有一個樹系具備 Windows Server 2012R2。 裝置物件類型必須存在。
* 如果安裝精靈已經在執行中，則將不會偵測到任何變更。 在此情況下，請先完成安裝精靈，然後再執行一次。
* 確定您在初始化指令碼中提供的帳戶是 Active Directory 連接器實際使用的正確使用者。 若要確認，請依照下列步驟執行：
  * 從 [開始] 功能表開啟 [同步處理服務] 。
  * 開啟 [連接器]  索引標籤。
  * 尋找類型為 Active Directory 網域服務的連接器並加以選取。
  * 選取 [動作] 下方的 [屬性]。
  * 前往 [連線至 Active Directory 樹系] 。 請確認此畫面上所指定的網域和使用者名稱，與提供給指令碼的帳戶相符。
    ![Sync Service Manager 中的連接器帳戶](./media/how-to-connect-device-writeback/connectoraccount.png)

確認 Active Directory 中的組態：

* 確認裝置註冊服務位於組態命名內容下方 (CN=DeviceRegistrationService,CN=Device Registration Services,CN=Device Registration Configuration,CN=Services,CN=Configuration) 底下的位置。

![疑難排解，組態命名空間中的 DeviceRegistrationService](./media/how-to-connect-device-writeback/troubleshoot1.png)

* 搜尋組態命名空間來確認只有一個組態物件。 如果有一個以上的物件，請刪除重複項目。

![疑難排解，搜尋重複的物件](./media/how-to-connect-device-writeback/troubleshoot2.png)

* 在「裝置註冊服務」物件上，確定 msDS-DeviceLocation 屬性存在且具有值。 查閱此位置，並確定它存在且 objectType 為  msDS-DeviceContainer。

![疑難排解，msDS-DeviceLocation](./media/how-to-connect-device-writeback/troubleshoot3.png)

![疑難排解，RegisteredDevices 物件類別](./media/how-to-connect-device-writeback/troubleshoot4.png)

* 確認 Active Directory 連接器所使用的帳戶，具備上一個步驟所找到之「註冊的裝置」容器的必要權限。 這是此容器的預期權限：

![疑難排解，驗證容器上的權限](./media/how-to-connect-device-writeback/troubleshoot5.png)

* 確認 Active Directory 帳戶具備 CN=Device Registration Configuration,CN=Services,CN=Configuration 物件的權限。

![疑難排解，，驗證裝置註冊組態的權限](./media/how-to-connect-device-writeback/troubleshoot6.png)

## <a name="additional-information"></a>其他資訊
* [使用條件式存取管理風險](../active-directory-conditional-access-azure-portal.md)
* [使用 Azure Active Directory 裝置註冊設定內部部署條件式存取](../active-directory-device-registration-on-premises-setup.md)

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。

