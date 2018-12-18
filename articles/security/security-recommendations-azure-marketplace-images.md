---
title: Azure Marketplace 映像的安全性建議 | Microsoft Docs
description: 本文提供 Market Place 中所含映像的建議
services: security
documentationcenter: na
author: barclayn
manager: MBaldwin
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.date: 10/18/2017
ms.author: barclayn
ms.openlocfilehash: 4ae36f87c29975c82bb99f713893a9dc78a249e6
ms.sourcegitcommit: d6ad3203ecc54ab267f40649d3903584ac4db60b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/19/2017
ms.locfileid: "23465403"
---
# <a name="security-recommendations-for-azure-marketplace-images"></a>Azure Marketplace 映像的安全性建議

我們建議每個解決方案都要符合下列安全性設定建議。 這將有助於針對 Azure Marketplace 中的協力廠商解決方案映像維護高等級的安全性。

這些建議對於 Azure Marketplace 中沒有映像的組織而言也很有用。 您可以根據可在下列表格中找到的指導方針，來檢查貴公司的 Windows 和 Linux 映像設定。

## <a name="open-source-based-images"></a>開啟以來源為基礎的映像

|||
|--------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **類別**                                                 | **檢查**                                                                                                                                                                                                                                                                              |
| 安全性                                                     | 已針對 Linux 發行版本安裝所有最新的安全性修補程式。                                                                                                                                                                                                              |
| 安全性                                                     | 已遵循業界指導方針來保護特定 Linux 發行版本的 VM 映像。                                                                                                                                                                                     |
| 安全性                                                     | 只利用需要的 Windows 伺服器角色、功能、服務和網路連接埠來維持最少的使用量，藉以限制受攻擊面。                                                                                                                                               |
| 安全性                                                     | 掃描原始程式碼和產生的 VM 映像以偵測惡意程式碼。                                                                                                                                                                                                                                   |
| 安全性                                                     | VHD 映像只包含必要的鎖定帳戶，沒有預設的密碼可允許互動式登入；沒有後門。                                                                                                                                           |
| 安全性                                                     | 除非應用程式功能依賴防火牆規則 (例如防火牆應用裝置)，否則停用它們。                                                                                                                                                                             |
| 安全性                                                     | 已經從 VHD 映像中移除所有機密資訊，例如，測試 SSH 金鑰、已知的 Hosts 檔案、記錄檔及不必要的憑證。                                                                                                                                       |
| 安全性                                                     | 建議不要使用 LVM。                                                                                                                                                                                                                                            |
| 安全性                                                     | 必要程式庫的最新版本應該包含： </br> - OpenSSL v1.0 或更新版本 </br> - Python 2.5 或更新版本 (強烈建議使用 Python 2.6+) </br> - Python pyasn1 套件 (如果尚未安裝) </br> - d.OpenSSL v 1.0 或更新版本                                                                |
| 安全性                                                     | 必須清除 Bash/殼層歷程記錄項目                                                                                                                                                                                                                                             |
| 網路                                                   | 預設應包含 SSH 伺服器。 利用下列選項，將保持運作的 SSH 設為 sshd 設定：ClientAliveInterval 180                                                                                                                                                        |
| 網路                                                   | 映像不應包含任何自訂網路設定。 刪除 resolv.conf：`rm /etc/resolv.conf`                                                                                                                                                                                |
| 部署                                                   | 應該安裝最新的 Azure Linux 代理程式 </br> - 應該使用 RPM 或 Deb 套件來安裝代理程式。  </br> - 您也可以使用手動安裝程序，但建議使用且慣用安裝程式套件。 </br> - 如果是從 github 存放庫手動安裝代理程式，請先將 `waagent` 檔案複製到 `/usr/sbin` 並 (以 root 身分) 執行： </br>`# chmod 755 /usr/sbin/waagent` </br>`# /usr/sbin/waagent -install` </br>代理程式設定檔將會放在 `/etc/waagent.conf`。    |
| 部署                                                   | 確保 Azure 支援服務可以在需要時，為我們的合作夥伴提供序列主控台輸出，並針對從雲端儲存體掛接的 OS 磁碟提供適當的逾時。 映像必須已將下列參數加入至核心開機行：`console=ttyS0 earlyprintk=ttyS0 rootdelay=300` |
| 部署                                                   | OS 磁碟上沒有交換磁碟分割。 交換可透過 Linux 代理程式要求，以便在本機資源磁碟上建立。         |
| 部署                                                   | 建議您針對 OS 磁碟建立單一根磁碟分割。      |
| 部署                                                   | 僅限 64 位元作業系統。                                                                                                                                                                                                                                                          |

## <a name="windows-server-based-images"></a>以 Windows Server 為基礎的映像

|||
|-------------| -------------------------|
| **類別**                                                     | **檢查**                                                                                                                                                                |
| 安全性                                                         | 使用安全的 OS 基本映像。 針對以 Windows Server 為基礎之任何映像的來源使用的 VHD，必須來自透過 Microsoft Azure 提供的 Windows Server OS 映像。 |
| 安全性                                                         | 安裝所有最新的安全性更新。                                                                                                                                     |
| 安全性                                                         | 應用程式不應在限制的使用者名稱 (例如 Administrator、root 及 admin) 上有相依性。                                                                |
| 安全性                                                         | 作業系統硬碟上不支援 BitLocker 磁碟機加密。 可以在資料磁碟上使用 BitLocker。                                                            |
| 安全性                                                         | 只利用已啟用的必要 Windows Server 角色、功能、服務和網路連接埠來維持最少的使用量，以限制受攻擊面。                         |
| 安全性                                                         | 掃描原始程式碼和產生的 VM 映像以偵測惡意程式碼。                                                                                                                     |
| 安全性                                                         | 設定 Windows Server 映像安全性更新來自動更新。                                                                                                                |
| 安全性                                                         | VHD 映像只包含必要的鎖定帳戶，沒有預設的密碼可允許互動式登入；沒有後門。                             |
| 安全性                                                         | 除非應用程式功能依賴防火牆規則 (例如防火牆應用裝置)，否則停用它們。                                                               |
| 安全性                                                         | 已經從 VHD 映像中移除所有機密資訊。 例如，應移除 HOSTS 檔案、記錄檔和不必要的憑證。                                              |
| 部署                                                       | 僅限 64 位元作業系統。                            |
