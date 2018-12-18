---
title: Azure IoT Edge 安全性管理員 | Microsoft Docs
description: 管理 IoT Edge 裝置安全性立場和安全性服務的完整性。
services: iot-edge
keywords: security, element, enclave, IoT Edge, 安全性, 元素, 保護區
author: eustacea
manager: timlt
ms.author: eustacea
ms.date: 07/30/2018
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: 4594685291396b8b80e62abe57be109f0abbd81d
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46126338"
---
# <a name="azure-iot-edge-security-manager"></a>Azure IoT Edge 安全性管理員

Azure IoT Edge 安全性管理員是有完整界限的安全性核心，它透過將安全晶片硬體抽象化，來保護 IoT Edge 裝置與其所有元件。 它是安全性強化的焦點，並且向原始設備製造商 (OEM) 提供技術整合點。

![Azure IoT Edge 安全性管理員](media/edge-security-manager/iot-edge-security-manager.png)

IoT Edge 安全性管理員的目標是保護 IoT Edge 裝置與所有固有軟體作業的完整性。  透過將信任從底層硬體根信任 (如果有) 轉換到安全啟動 IoT Edge 執行階段，並繼續監視其作業的完整性。  IoT Edge 安全性管理員本質上包含搭配可用安全晶片硬體運作的軟體，有助於提供最高安全性保證。  

IoT Edge 安全性管理員的責任包括 (但不限於)：

* Azure IoT Edge 裝置的啟動受保護且受測量。
* 佈建裝置身分識別，以及在適用的情況下轉換信任。
* 裝載及保護雲端服務 (如裝置佈建服務) 的裝置元件。
* 使用模組唯一身分識別安全地佈建 IoT Edge 模組。
* 透過公證服務作為硬體根信任的閘道管理員。
* 在執行階段監視 IoT Edge 作業的完整性。

IoT Edge 安全性管理員包含三個主要元件：

* IoT Edge 安全性精靈。
* 硬體安全性模組平台抽象層 (HSM PAL)。
* 硬體晶片根信任或 HSM，雖然是選擇性，但強烈建議包含它。

## <a name="the-iot-edge-security-daemon"></a>IoT Edge 安全性精靈

IoT Edge 安全性精靈是負責 IoT Edge 安全性管理員邏輯作業的軟體。  它包含 IoT Edge 裝置信任運算基礎的重要部分。 

### <a name="design-principles"></a>設計原則

為了適當地且實際地執行其責任，IoT Edge 安全性精靈以符合以下核心原則為目標：

#### <a name="maximize-operational-integrity"></a>將運作完整性最大化

IoT Edge 安全性精靈的設計旨在以任何所指定根信任硬體的防禦功能內可得的最高完整性執行。 透過適當的整合，根信任硬體會靜態地以及在執行階段測量及監視安全性精靈，以防止竄改。

一般而言，實體存取對 IoT 裝置一向是個威脅。  因此硬體根信任在防禦 IoT Edge 安全性精靈的完整性中扮演重要角色。  硬體根信任分為兩種類別：

* 適用於敏感性資訊 (如祕密和密碼編譯金鑰) 保護的安全元素。
* 適用於祕密保護 (如金鑰) 與敏感性工作負載 (如計量和計費) 的安全性保護區。

這兩種利用硬體根信任的模型之存在，提供了兩種執行環境：

* 依靠使用安全元素來保護敏感性資訊的標準或豐富執行環境 (REE)。
* 依靠使用安全性保護區技術來保護敏感性資訊，並提供軟體執行保護的信任執行環境 (TEE)。

針對使用安全性保護區作為硬體根信任的裝置，IoT Edge 安全性精靈中的敏感性邏輯應該位於該保護區內。  安全性精靈的非敏感性部分可以位於 TEE 外部。  在任何情況下，原始設計製造商 (ODM) 和原始設備製造商 (OEM) 應從其 HSM 延伸信任，以測量及防禦 Edge 安全性精靈在啟動期間和執行階段的完整性。

#### <a name="minimize-bloat-and-churn"></a>將膨脹和變動最小化

Edge 安全性精靈的另一個核心原則是將變動最小化。  針對最高層級的信任，IoT Edge 安全性精靈可與裝置硬體根信任緊密耦合，並以原生程式碼的方式運作。  這些類型的實踐通常用於透過硬體根信任的安全性更新路徑 (相對於 OS 提供的更新機制) 來更新精靈軟體，視特定硬體和部署案例而定，此方法可能較困難。  雖然強烈建議 IoT 裝置進行安全性更新，但過多的更新需求或大型更新承載，是已知會擴展受威脅面的原因。  範例包括跳過更新以將作業可用性最大化，或根信任硬體太受限而無法處理大型更新承載。  因此，IoT Edge 安全性精靈被的設計旨在節省使用量，進而使信賴運算基礎維持小型，並將更新需求最小化。

### <a name="architecture-of-iot-edge-security-daemon"></a>IoT Edge 安全性精靈的架構

![Azure IoT Edge 安全性精靈](media/edge-security-manager/iot-edge-security-daemon.png)

IoT Edge 安全性精靈的架構是設計成利用任何可取得的硬體根信任技術來強化安全性。  更明確地說，它的架構是設計成允許在標準/豐富執行環境 (REE) 和信任執行環境 (TEE) 之間進行分割環境作業，以利用提供信任執行環境 (TEE) 的硬體技術。  IoT Edge 安全性精靈架構的核心是角色特定的介面，它能讓 Edge 的主要元件進行交互作用，以確保 IoT Edge 裝置及其作業的完整性。

#### <a name="cloud-interface"></a>雲端介面

雲端介面可讓 IoT Edge 安全性精靈存取雲端服務，例如安全性更新等裝置安全性的雲端互補項目。  例如，IoT Edge 安全性精靈目前使用此介面來存取 Azure IoT 中樞[裝置佈建服務 (DPS)](https://docs.microsoft.com/azure/iot-dps/)，以進行裝置身分識別生命週期管理。  

#### <a name="management-api"></a>管理 API

IoT Edge 安全性精靈提供管理 API，由 Edge 代理程式在建立/啟動/停止/移除 Edge 模組時呼叫。 IoT Edge 安全性精靈會儲存所有使用中模組的「註冊」。 這些註冊將模組的身分識別對應到模組的某些屬性。 這些屬性的一些範例為在容器中執行之處理序的處理序識別碼 (pid)，或是 Docker 容器內容的雜湊。

工作負載 API 會使用這些屬性來證明呼叫者有權限執行某個動作。

管理 API 是特殊權限 API，只能從 IoT Edge 代理程式呼叫。  由於 IoT Edge 安全性精靈會啟動 (Bootstrap) 及啟動 (Start) IoT Edge 代理程式，所以它能在證明 IoT Edge 代理程式沒有被竄改之後，為 IoT Edge 代理程式建立隱含的註冊。 工作負載 API 所使用的相同證明流程，也用於限制只有 IoT Edge 代理程式可以存取管理 API。

#### <a name="container-api"></a>容器 API

IoT Edge 安全性精靈提供用於和使用中的容器系統 (如 Moby 和 Docker) 互動的容器介面，以進行模組具現化。

#### <a name="workload-api"></a>工作負載 API

工作負載 API 是所有模組 (包括 IoT Edge 代理程式) 都可以存取的 IoT Edge 安全性精靈 API。 它會提供身分識別證明 (格式為 HSM 根簽署權杖或 X509 憑證) 與對應的信任組合給模組。 信任組合包含模組應該信任之所有其他伺服器的 CA 憑證。

IoT Edge 安全性精靈會使用證明流程來保護此 API。 當模組呼叫此 API 時，IoT Edge 安全性精靈會嘗試尋找身分識別的註冊。 如果成功，它會使用註冊的屬性來測量模組。 如果測量流程的結果符合註冊，會產生新的 HSM 根簽署權杖或 X509 憑證，而且對應的 CA 憑證 (信任組合) 會傳回到模組。  模組使用此憑證來連線到 IoT 中樞、其他模組或用來啟動伺服器。 當簽署的權杖或憑證快要過期時，是由模組來負責要求新的憑證。  其他考量也整合到設計中，讓身分識別更新流程儘可能順暢。

### <a name="integration-and-maintenance"></a>整合和維護

Microsoft [在 GitHub 上維護 IoT Edge 安全性精靈](https://github.com/Azure/iotedge/tree/master/edgelet) \(英文\) 的主要程式碼基底。

#### <a name="installation-and-updates"></a>安裝和更新

因為是原生處理序，所以 Azure IoT Edge 安全性精靈的安裝和更新是透過作業系統的套件管理系統來管理。  不過，IoT Edge 裝置通常會有硬體根信任，並且通常會透過裝置安全性開機和更新管理系統來為精靈的完整性提供額外的強化。  裝置製造商可根據其個別裝置功能來探索這些用途。

#### <a name="versioning"></a>版本控制

IoT Edge 執行階段會追蹤並回報 IoT Edge 安全性精靈的版本。 版本是回報為 Edge 代理程式模組回報屬性 (property) 的 *runtime.platform.version* 屬性 (attribute)。

### <a name="hardware-security-module-platform-abstraction-layer-hsm-pal"></a>硬體安全性模組平台抽象層 (HSM PAL)

HSM PAL 將所有根信任硬體抽象化，以將 IoT Edge 開發人員或使用者從其複雜性中分離。  其提供應用程式開發介面 (API) 和資訊傳送網域通訊程序的整合，例如標準執行環境和安全性保護區之間的通訊。  HSM PAL 的實際實作取決於使用中的特定安全硬體而定。  它的存在使幾乎任何安全晶片硬體都能在整個 IoT 生態系統使用。

## <a name="secure-silicon-root-of-trust-hardware"></a>安全晶片根信任硬體

必須有安全晶片才能在 IoT Edge 裝置硬體內固定信任。  安全晶片有各種類型，包括信賴平台模組 (TPM)、內嵌安全性元素 (eSM)、ARM 信任區、Intel SGX 及自訂安全晶片技術。  因為考慮到 IoT 裝置實體可存取性的相關威脅，所以強烈建議在裝置中使用安全晶片根信任。

## <a name="iot-edge-security-manager-integration-and-maintenance"></a>IoT Edge 安全性管理員整合和維護

IoT Edge 安全性管理員的其中一個主要目標是為自訂強化功能，識別及隔離負責抵禦 Azure IoT Edge 平台的安全性和完整性的元件。  因此裝置製造商等協力廠商，應該利用可和其裝置硬體搭配使用的自訂安全性功能。  請參閱下方的＜後續步驟＞一節，其中的連結包含之範例顯示如何在 Linux 和 Windows 平台上使用信賴平台模組 (TPM) 來強化 Azure IoT 安全性管理員。  這些範例是使用軟體或虛擬 TPM，但使用離散 TPM 裝置直接套用。  

## <a name="next-steps"></a>後續步驟

閱讀部落格文章[保護智慧邊緣](https://azure.microsoft.com/blog/securing-the-intelligent-edge/) \(英文\)。

[在 Linux 虛擬機器上使用虛擬 TPM](how-to-auto-provision-simulated-device-linux.md) 建立及佈建 Edge 裝置。

[在 Windows 上建立及佈建模擬 TPM Edge 裝置](how-to-auto-provision-simulated-device-windows.md)。

<!-- Links -->
[lnk-edge-blog]: https://azure.microsoft.com/blog/securing-the-intelligent-edge/
