---
title: Azure Migrate 中的 VMware 評估支援
description: 了解使用 Azure Migrate 伺服器評估來評估 VMware VM 的支援。
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 11/10/2020
ms.openlocfilehash: 02757477ca4a07f4526df0339291122eda57c9b5
ms.sourcegitcommit: ca215fa220b924f19f56513fc810c8c728dff420
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2021
ms.locfileid: "98567422"
---
# <a name="support-matrix-for-vmware-assessment"></a>VMware 評估的支援矩陣 

本文摘要說明當您使用 [Azure Migrate：伺服器評估](migrate-services-overview.md#azure-migrate-server-assessment-tool) 工具探索和評估 VMware vm 以遷移至 Azure 時的必要條件和支援需求。 若要評估 VMware Vm，請建立 Azure Migrate 專案，並將伺服器評定工具新增至專案。 新增工具之後，請部署 Azure Migrate 設備。 設備會持續探索內部部署機器，並將機器的中繼資料和效能資料傳送至 Azure。 探索完成之後，請將探索到的機器收集到群組中，然後為群組執行評估。 

如果您想要將 VMware VM 遷移至 Azure，請檢閱[遷移支援矩陣](migrate-support-matrix-vmware-migration.md)。



## <a name="limitations"></a>限制

**需求** | **詳細資料**
--- | ---
**專案限制** | 您可以在 Azure 訂用帳戶中建立多個專案。<br/><br/> 您可以在單一[專案](migrate-support-matrix.md#azure-migrate-projects)中探索及評估最多 35,000 個 VMware VM。 專案中也可以包含實體伺服器和 Hyper-V VM，每個最多都可以達到評估限制。
**探索** | Azure Migrate 設備可以在 vCenter Server 上探索最多 10,000 個 VMware VM。
**評量** | 您最多可以在單一群組中新增 35,000 部機器。<br/><br/> 您最多可以在單一評估中評估 35,000 個 VM。

[深入了解](concepts-assessment-calculation.md)評估。


## <a name="vmware-requirements"></a>VMware 需求

**VMware** | **詳細資料**
--- | ---
**vCenter Server** | 您想要探索和評估的機器必須由 vCenter Server 版本5.5、6.0、6.5、6.7 或7.0 來管理。<br/><br/> 目前不支援在設備中提供 ESXi 主機詳細資料來探索 VMware Vm。
**權限** | 伺服器評定需要 vCenter Server 唯讀帳戶才能進行探索和評量。<br/><br/> 如果您想要進行應用程式探索或相依性視覺效果，則帳戶必須啟用 **虛擬機器**  >  **來賓作業** 的許可權。

## <a name="vm-requirements"></a>VM 需求
**VMware** | **詳細資料**
--- | ---
**VMware VM** | 所有作業系統都可以進行評估以進行遷移。 
**儲存體** | 支援連接至 SCSI、IDE 和以 SATA 為基礎之控制器的磁片。


## <a name="azure-migrate-appliance-requirements"></a>Azure Migrate 設備需求

Azure Migrate 會使用 [Azure Migrate 設備](migrate-appliance.md)來進行探索和評估。 您可以使用 OVA 範本將設備部署為 VMware VM，並匯入 vCenter Server 或使用 [PowerShell 腳本](deploy-appliance-script.md)。

- 了解 VMware 的[設備需求](migrate-appliance.md#appliance---vmware)。
- 在 Azure Government 中，您必須[使用指令碼](deploy-appliance-script-government.md)來部署設備。
- 查看設備在 [公用](migrate-appliance.md#public-cloud-urls) 和 [政府](migrate-appliance.md#government-cloud-urls) 雲端中需要存取的 url。


## <a name="port-access-requirements"></a>埠存取需求

**裝置** | **[連接]**
--- | ---
**設備** | TCP 連接埠 3389上的輸入連線，以允許遠端桌面對設備連線。<br/><br/> 連接埠 44368 上的輸入連線，以使用 URL 從遠端存取設備管理應用程式：```https://<appliance-ip-or-name>:44368``` <br/><br/>連接埠 443 (HTTPS) 上的輸出連線，以將探索和效能中繼資料傳送至 Azure Migrate。
**vCenter 伺服器** | TCP 連接埠 443 上的輸入連線，以允許設備收集設定和效能中繼資料以進行評估。 <br/><br/> 根據預設，設備會連線到連接埠 443 上的 vCenter。 如果 vCenter 伺服器接聽不同的連接埠，您可以在設定探索時修改連接埠。
**ESXi 主機** | 如果您想要執行 [應用程式探索](how-to-discover-applications.md)或 [無代理](concepts-dependency-visualization.md#agentless-analysis)程式相依性分析，則設備會連接到 TCP 埠443上的 ESXi 主機，以探索應用程式，並在 vm 上執行無代理程式相依性視覺效果。

## <a name="application-discovery-requirements"></a>應用程式探索需求

除了探索機器之外，伺服器評量還可以探索機器上執行的應用程式、角色和功能。 探索應用程式詳細目錄可讓您找出並規劃專為內部部署工作負載量身打造的遷移路徑。 

**支援** | **詳細資料**
--- | ---
**支援的機器** | 目前僅支援 VMware VM。 您可以從每個 Azure Migrate 設備探索最多可在 10000 VMware Vm 上安裝的應用程式。
**作業系統** | 支援執行所有 Windows 和 Linux 版本的 Vm。
**VM 需求** | 您必須在要探索應用程式的 Vm 上安裝並執行 VMware 工具。 <br/><br/> VMware 工具版本必須為 10.2.0 之後。<br/><br/> VM 必須安裝 PowerShell 2.0 版或更新版本。
**探索** | 在 vm 上安裝之應用程式的相關資訊，會使用安裝在 VM 上的 VMware 工具從 vCenter Server 收集。 設備會使用 vSphere Api，從 vCenter Server 收集應用程式資訊。 應用程式探索不需要代理程式。 Vm 上未安裝任何專案，且設備不會直接連線至 Vm。 WMI/SSH 應在 Vm 上啟用及使用。
**vCenter** | 用於評量的 vCenter Server 唯讀帳戶，需要 **虛擬機器**  >  **來賓作業** 啟用的許可權，才能與 VM 互動以進行應用程式探索。
**VM 存取** | 應用程式探索需要 VM 上的本機使用者帳戶，才能進行應用程式探索。<br/><br/> Azure Migrate 目前支援在所有 Windows 伺服器上使用一個認證，以及為所有 Linux 伺服器使用一個認證。<br/><br/> 請為 Windows VM 建立來賓使用者帳戶，以及為所有 Linux VM 建立常規/一般使用者帳戶 (非 sudo 存取權)。
**連接埠存取** | Azure Migrate 設備必須能夠連線到執行您要探索應用程式之 Vm 的 ESXi 主機上的 TCP 埠443。 VCenter Server 會傳回 ESXI 主機連接，以下載包含應用程式資訊的檔案。



## <a name="dependency-analysis-requirements-agentless"></a> (無代理程式) 的相依性分析需求

[相依性分析](concepts-dependency-visualization.md)可協助您識別所要評估並遷移至 Azure 的內部部署機器之間的相依性。 下表摘要說明用於設定無代理程式相依性分析的需求。 

**支援** | **詳細資料**
--- | --- 
**支援的機器** | 目前僅支援 VMware VM。
**Windows VM** | Windows Server 2016<br/> Windows Server 2012 R2<br/> Windows Server 2012<br/> Windows Server 2008 R2 (64 位元)。<br/>Microsoft Windows Server 2008 (32 位) 。 
**Linux VM** | Red Hat Enterprise Linux 7、6、5<br/> Ubuntu Linux 14.04、16.04<br/> Debian 7、8<br/> Oracle Linux 6、7<br/> CentOS 5、6、7。<br/> SUSE Linux Enterprise Server 11 和更新版本。
**VM 需求** | VMware Tools (晚于 10.2.0) 必須安裝並在您想要分析的 Vm 上執行。<br/><br/> VM 必須安裝 PowerShell 2.0 版或更新版本。
**探索方法** |  您可以使用安裝在 VM 上的 VMware 工具，從 vCenter Server 收集 Vm 之間的相依性資訊。 設備會使用 vSphere Api，從 vCenter Server 收集資訊。 探索為無代理程式。 VM 上未安裝任何專案，且設備不會直接連線至 Vm。 WMI/SSH 應在 Vm 上啟用及使用。
**vCenter 帳戶** | Azure Migrate 用於評量的唯讀帳戶，必須啟用 **虛擬機器 > 來賓作業** 的許可權。
**Windows VM 許可權** |  帳戶 (本機系統管理員或網域) 具有 Vm 的本機系統管理員許可權。
**Linux 帳戶** | 根使用者帳戶，或在/bin/netstat 和/bin/ls 檔案上具有這些許可權的帳戶： CAP_DAC_READ_SEARCH 和 CAP_SYS_PTRACE。<br/><br/> 使用下列命令來設定這些功能： <br/><br/> sudo setcap CAP_DAC_READ_SEARCH、CAP_SYS_PTRACE = ep/bin/ls<br/><br/> sudo setcap CAP_DAC_READ_SEARCH、CAP_SYS_PTRACE = ep/bin/netstat
**連接埠存取** | Azure Migrate 設備必須能夠連線到執行您要探索其相依性之 Vm 的 ESXI 主機上的 TCP 埠443。 VCenter Server 會傳回 ESXI 主機連接，以下載包含相依性資訊的檔案。


## <a name="dependency-analysis-requirements-agent-based"></a>以代理程式為基礎的)  (相依性分析需求

[相依性分析](concepts-dependency-visualization.md)可協助您識別所要評估並遷移至 Azure 的內部部署機器之間的相依性。 下表摘要說明用於設定代理程式型相依性分析的需求。 

**需求** | **詳細資料** 
--- | --- 
**部署之前** | 請先備妥 Azure Migrate 專案，並將「Azure Migrate：伺服器評估」工具新增至專案中。<br/><br/>  請在設定 Azure Migrate 設備以探索內部部署機器之後，部署相依性視覺效果。<br/><br/> [了解如何](create-manage-projects.md)第一次建立專案。<br/> [了解如何](how-to-assess.md)將評估工具新增至現有專案。<br/> 了解如何設定 Azure Migrate 設備，以評估 [Hyper-V](how-to-set-up-appliance-hyper-v.md)、[VMware](how-to-set-up-appliance-vmware.md) 或實體伺服器。
**支援的機器** | 支援所有機器。
**Azure Government** | 在 Azure Government 中無法使用相依性視覺效果。
**Log Analytics** | Azure Migrate 會使用 [Azure 監視器記錄](../azure-monitor/log-query/log-query-overview.md)中的[服務對應](../azure-monitor/insights/service-map.md)解決方案來實現相依性視覺效果。<br/><br/> 請將新的或現有的 Log Analytics 工作區與 Azure Migrate 專案建立關聯。 Azure Migrate 專案的工作區在新增之後就無法進行修改。 <br/><br/> 此工作區必須位於和 Azure Migrate 專案相同的訂用帳戶中。<br/><br/> 此工作區必須位於「美國東部」、「東南亞」或「西歐」區域。 其他區域中的工作區則無法與專案相關聯。<br/><br/> 此工作區必須位於[支援服務對應](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions)的區域中。<br/><br/> 在 Log Analytics 中，與 Azure Migrate 相關聯的工作區會標記上遷移專案金鑰和專案名稱。
**所需的代理程式** | 在您要分析的每部機器上安裝下列代理程式：<br/><br/> [Microsoft Monitoring agent (MMA)](../azure-monitor/platform/agent-windows.md)。<br/> [相依性代理程式](../azure-monitor/platform/agents-overview.md#dependency-agent)。<br/><br/> 如果內部部署機器未連線到網際網路，則需要下載 Log Analytics 閘道並安裝到機器上。<br/><br/> 深入了解如何安裝 [Dependency Agent](how-to-create-group-machine-dependencies.md#install-the-dependency-agent) 和 [MMA](how-to-create-group-machine-dependencies.md#install-the-mma)。
**Log Analytics 工作區** | 此工作區必須位於和 Azure Migrate 專案相同的訂用帳戶中。<br/><br/> Azure Migrate 可支援位於美國東部、東南亞和西歐區域的工作區。<br/><br/>  此工作區必須位於[支援服務對應](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions)的區域中。<br/><br/> Azure Migrate 專案的工作區在新增之後就無法進行修改。
**成本** | 服務對應解決方案在使用的前 180 天 (從您將 Log Analytics 工作區關聯至 Azure Migrate 專案的那一天算起) 不會產生任何費用。<br/><br/> 經過 180 天後，會套用標準 Log Analytics 費用。<br/><br/> 在相關聯的 Log Analytics 工作區中使用非服務對應的解決方案則會產生 Log Analytics 的[標準費用](https://azure.microsoft.com/pricing/details/log-analytics/)。<br/><br/> 刪除 Azure Migrate 專案時，工作區不會隨之刪除。 在專案刪除後，服務對應就無法免費使用，系統會根據 Log Analytics 工作區的付費層向每個節點收取費用。<br/><br/>如果您的專案是在 Azure Migrate 正式發行 (GA - 2018 年 2 月 28 日) 之前建立的，則可能會產生額外的服務對應費用。 為確保在 180 天後才要付費，建議您建立新的專案，因為 GA 之前的既有工作區仍要付費。
**管理** | 當您向工作區註冊代理程式時，請使用 Azure Migrate 專案所提供的識別碼和金鑰。<br/><br/> 您可以使用 Azure Migrate 外部的 Log Analytics 工作區。<br/><br/> 如果您刪除相關聯的 Azure Migrate 專案，工作區並不會自動刪除。 [請手動將其刪除](../azure-monitor/platform/manage-access.md)。<br/><br/> 除非您刪除 Azure Migrate 專案，否則請勿刪除 Azure Migrate 建立的工作區。 如果這樣做，相依性視覺效果功能不會如預期般運作。
**網際網路連線** | 如果機器未連線到網際網路，則需要在機器上安裝 Log Analytics 閘道。
**Azure Government** | 不支援代理程式型的相依性分析。


## <a name="next-steps"></a>後續步驟

- [檢閱](best-practices-assessment.md)適用於建立評估的最佳做法。
- [準備 VMware](./tutorial-discover-vmware.md) 評估。