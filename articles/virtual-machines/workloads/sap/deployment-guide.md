---
title: 適用於 SAP NetWeaver 的 Azure 虛擬機器部署 | Microsoft Docs
description: 了解如何在 Azure 中的 Linux 虛擬機器上部署 SAP 軟體。
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 1c4f1951-3613-4a5a-a0af-36b85750c84e
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/26/2018
ms.author: sedusch
ms.openlocfilehash: 9208f2cb207daff2b122550fede48a8dda11d1db
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407921"
---
# <a name="azure-virtual-machines-deployment-for-sap-netweaver"></a>適用於 SAP NetWeaver 的 Azure 虛擬機器部署
[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1139904]:https://launchpad.support.sap.com/#/notes/1139904
[1173395]:https://launchpad.support.sap.com/#/notes/1173395
[1245200]:https://launchpad.support.sap.com/#/notes/1245200
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1558958]:https://launchpad.support.sap.com/#/notes/1558958
[1585981]:https://launchpad.support.sap.com/#/notes/1585981
[1588316]:https://launchpad.support.sap.com/#/notes/1588316
[1590719]:https://launchpad.support.sap.com/#/notes/1590719
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1605680]:https://launchpad.support.sap.com/#/notes/1605680
[1619720]:https://launchpad.support.sap.com/#/notes/1619720
[1619726]:https://launchpad.support.sap.com/#/notes/1619726
[1619967]:https://launchpad.support.sap.com/#/notes/1619967
[1750510]:https://launchpad.support.sap.com/#/notes/1750510
[1752266]:https://launchpad.support.sap.com/#/notes/1752266
[1757924]:https://launchpad.support.sap.com/#/notes/1757924
[1757928]:https://launchpad.support.sap.com/#/notes/1757928
[1758182]:https://launchpad.support.sap.com/#/notes/1758182
[1758496]:https://launchpad.support.sap.com/#/notes/1758496
[1772688]:https://launchpad.support.sap.com/#/notes/1772688
[1814258]:https://launchpad.support.sap.com/#/notes/1814258
[1882376]:https://launchpad.support.sap.com/#/notes/1882376
[1909114]:https://launchpad.support.sap.com/#/notes/1909114
[1922555]:https://launchpad.support.sap.com/#/notes/1922555
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1941500]:https://launchpad.support.sap.com/#/notes/1941500
[1956005]:https://launchpad.support.sap.com/#/notes/1956005
[1973241]:https://launchpad.support.sap.com/#/notes/1973241
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[2367194]:https://launchpad.support.sap.com/#/notes/2367194

[azure-cli]:../../../cli-install-nodejs.md
[azure-cli-2]:https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azureps-cmdlets-docs
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide.md (適用於 SAP 的 Azure 虛擬機器 DBMS 部署)
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f (VM 和 VHD 的快取)
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 (軟體 RAID)
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 (Microsoft Azure 儲存體)
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 (RDBMS 部署結構)
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 (Azure VM 相關的高可用性和災害復原)
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 (SQL Server 2012 SP1 CU4 和更新版本)
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b (SQL Server 2012 SP1 CU3 和舊版)
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 (使用來自 Azure Marketplace 的 SQL Server 映像)
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 (適用於 Azure 上 SAP 的一般 SQL Server 摘要)
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 (SQL Server RDBMS 的詳細規格)
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 (儲存體設定)
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d (備份與還原)
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c (備份與還原的效能考量)
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 (其他)
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]:deployment-guide.md (適用於 SAP 的 Azure 虛擬機器部署)
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 (SAP 資源)
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab (使用自訂映像部署 VM)
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 (案例 1：從 Azure Marketplace 為 SAP 部署 VM)
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 (案例 2：使用自訂映像為 SAP 部署 VM)
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 (案例 3：使用非一般化 Azure VHD 搭配 SAP 從內部部署移動 VM)
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e (適用於 Microsoft Azure 上 SAP 的 VM 部署案例)
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 (部署 Azure PowerShell Cmdlet)
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e (下載並匯入 SAP 相關的 PowerShell Cmdlet)
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc (將 VM 加入內部部署網域 - 僅限 Windows)
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 (Linux)
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d (下載、安裝和啟用 Azure VM 代理程式)
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 (Azure PowerShell)
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f (Azure CLI)
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca (設定 Azure Enhanced Monitoring Extension for SAP)
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 (Azure Enhanced Monitoring for SAP 的整備檢查)
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1 (Azure 監視基礎結構的健康情況檢查)
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 (對適用於 SAP 的 Azure 監視進行疑難排解)

[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b (設定監視)
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d (設定 Proxy)
[deployment-guide-figure-100]:media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]:media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]:deployment-guide.md#figure-11
[deployment-guide-figure-1100]:media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]:media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]:media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]:deployment-guide.md#figure-14
[deployment-guide-figure-1400]:media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]:media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]:media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]:deployment-guide.md#figure-5
[deployment-guide-figure-50]:media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]:media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]:deployment-guide.md#figure-6
[deployment-guide-figure-600]:media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]:deployment-guide.md#figure-7
[deployment-guide-figure-700]:media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]:media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]:media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]:deployment-guide.md#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]:deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]:deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b (針對設定端對端監視進行檢查及疑難排解)

[deploy-template-cli]:../../../resource-group-template-deploy-cli.md
[deploy-template-portal]:../../../resource-group-template-deploy-portal.md
[deploy-template-powershell]:../../../resource-group-template-deploy.md

[dr-guide-classic]:http://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[getting-started-dbms]:get-started.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]:get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]:get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]:../../virtual-machines-windows-classic-sap-get-started.md
[getting-started-windows-classic-dbms]:../../virtual-machines-windows-classic-sap-get-started.md#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]:../../virtual-machines-windows-classic-sap-get-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]:../../virtual-machines-windows-classic-sap-get-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]:../../virtual-machines-windows-classic-sap-get-started.md#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]:../../virtual-machines-windows-classic-sap-get-started.md#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]:http://go.microsoft.com/fwlink/?LinkId=613056

[install-extension-cli]:virtual-machines-linux-enable-aem.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-azurermvmaemextension]:https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmaemextension

[planning-guide]:planning-guide.md (適用於 SAP 的 Azure 虛擬機器規劃和實作)
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff (資源)
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058 (Azure 虛擬機器上執行之 SAP NetWeaver 的高可用性和災害復原)
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77 (SAP 應用程式伺服器的高可用性)
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f (對 SAP 執行個體使用 Autostart)
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803 (僅限雲端 - 將虛擬機器部署到 Azure，無須倚賴內部部署客戶網路)
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 (跨單位 - 將單一或多個 SAP VM 部署到 Azure，與內部部署網路完全整合)
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a (Azure 區域)
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358 (容錯網域)
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560 (升級網域)
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665 (Azure 可用性設定組)
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 (Microsoft Azure 虛擬機器概念)
[planning-guide-3.3.2]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 (Azure 進階儲存體)
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53 (使用非一般化磁碟將 VM 從內部部署移至 Azure)
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c (使用客戶特定的映像部署 VM)
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef (準備使用非一般化磁碟將 VM 從內部部署移至 Azure)
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 (準備使用客戶特定的映像為 SAP 部署 VM)
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7 (使用適用於 Azure 的 SAP 準備 VM)
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79 (Azure 磁碟和 Azure 映像之間的差異)
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a (將 VHD 從內部部署上傳至 Azure)
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1 (在 Azure 儲存體帳戶之間複製磁碟)
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 (SAP 部署的 VM/VHD 結構)
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d (為連接的磁碟設定自動掛接)
[planning-guide-7.1]:planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30 (搭配 SAP NetWeaver 之單一 VM 的示範/訓練案例)
[planning-guide-7]:planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14 (SAP 執行個體的僅限雲端部署概念)
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3 (適用於 SAP 的 Azure 監視解決方案)
[planning-guide-azure-premium-storage]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 (Azure 進階儲存體)
[planning-guide-managed-disks]:planning-guide.md#c55b2c6e-3ca1-4476-be16-16c81927550f (受控磁碟)
[planning-guide-figure-100]:media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]:media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]:media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]:media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]:media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]:media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]:media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]:media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]:media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]:media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]:media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd (Microsoft Azure 網路)
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f (儲存體：Microsoft Azure 儲存體和資料磁碟)

[powershell-install-configure]:https://docs.microsoft.com/powershell/azure/install-azurerm-ps
[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/resource-group-overview.md
[resource-groups-networking]:../../../networking/network-overview.md
[sap-pam]:https://support.sap.com/pam (SAP 產品可用性對照表)
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image-md%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-os-disk-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk-md%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-2-tier-user-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image-md%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-md%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image-md%2Fazuredeploy.json
[storage-azure-cli]:../../../storage/common/storage-azure-cli.md
[storage-azure-cli-copy-blobs]:../../../storage/common/storage-azure-cli.md#copy-blobs
[storage-introduction]:../../../storage/common/storage-introduction.md
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]:../../windows/premium-storage.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/storage-scalability-targets.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-azurerm-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md (使用 Azure Resource Manager 範本和 Azure CLI 部署和管理虛擬機器)
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md (使用 Azure Resource Manager 和 PowerShell 管理虛擬機器)
[virtual-machines-windows-agent-user-guide]:../../extensions/agent-windows.md
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-linux.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability]:../../linux/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:../../virtual-machines-windows-ps-create.md
[virtual-machines-sizes]:../../linux/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:./../../windows/sql/virtual-machines-windows-sql-high-availability-dr.md
[virtual-machines-sql-server-infrastructure-services]:./../../windows/sql/virtual-machines-windows-sql-server-iaas-overview.md
[virtual-machines-sql-server-performance-best-practices]:./../../windows/sql/virtual-machines-windows-sql-performance.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]:../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics]:../../../virtual-network/virtual-network-deploy-multinic-classic-ps.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-plan-design.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

對於需要在最短的時間內處理計算和儲存資源，而不需要漫長的採購週期的組織而言，Azure 虛擬機器是適用的解決方案。 您可以使用 Azure 虛擬機器在 Azure 中部署傳統應用程式，例如 SAP NetWeaver 架構應用程式。 擴充應用程式的可靠性與可用性，不需其他內部部署資源。 「Azure 虛擬機器」支援跨單位連線能力，貴公司可將「Azure 虛擬機器」整合到其內部部署網域、私人雲端，以及 SAP 系統環境中。

在本文中，我們將討論在 Azure 中的虛擬機器 (VM) 上部署 SAP 應用程式的步驟，包括替代部署選項和疑難排解。 本文是以 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]為基礎。 本文也可補充 SAP 安裝文件和 SAP Note 的不足，而這些是安裝及部署 SAP 軟體的主要資源。

## <a name="prerequisites"></a>必要條件
針對 SAP 軟體部署設定 Azure 虛擬機器，牽涉到多個步驟和資源。 開始之前，請確定您符合在 Azure 中的虛擬機器上安裝 SAP 軟體的必要條件。

### <a name="local-computer"></a>本機電腦
若要管理 Windows 或 Linux VM，您可以使用 PowerShell 指令碼和 Azure 入口網站。 對於這兩種工具，您需有執行 Windows 7 或更新 Windows 版本的本機電腦。 如果您只想管理 Linux VM，而且想要使用 Linux 電腦進行這項工作，您可以使用 Azure CLI。

### <a name="internet-connection"></a>網際網路連線
若要下載並執行 SAP 軟體部署所需的工具和指令碼，您必須連線到網際網路。 執行 Azure Enhanced Monitoring Extension for SAP 的 Azure VM 也必須能夠存取網際網路。 如果 Azure VM 是 Azure 虛擬網路或內部部署網域的一部分，請務必如[設定 Proxy][deployment-guide-configure-proxy] 所述，設定相關的 Proxy 設定。

### <a name="microsoft-azure-subscription"></a>Microsoft Azure 訂用帳戶
您需要使用中的 Azure 帳戶。

### <a name="topology-and-networking"></a>拓撲和網路
您需要在 Azure 中定義 SAP 部署的拓撲和架構：

* 要使用的 Azure 儲存體帳戶
* 您想要部署 SAP 系統的虛擬網路
* 您想要部署 SAP 系統的資源群組
* 您想要部署 SAP 系統的 Azure 區域
* SAP 組態 (兩層或三層)
* VM 大小以及要掛接到 VM 的額外資料磁碟數目
* SAP Correction and Transport System (CTS) 組態

開始 SAP 軟體部署程序之前，如有必要請先建立及設定 Azure 儲存體帳戶或 Azure 虛擬網路。 如需有關如何建立及設定這些資源的詳細資訊，請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]。

### <a name="sap-sizing"></a>SAP 大小調整
了解下列資訊，以便進行 SAP 大小調整︰

* 預估的 SAP 工作負載 (例如，使用 SAP Quick Sizer 工具) 和 SAP Application Performance Standard (SAPS) 數字
* SAP 系統所需的 CPU 資源和記憶體耗用量
* 每秒所需的輸入/輸出 (I/O) 作業
* Azure 中 VM 之間最終通訊所需的網路頻寬
* 內部部署資產與已部署 Azure 的 SAP 系統之間所需的網路頻寬

### <a name="resource-groups"></a>資源群組
在 Azure Resource Manager 中，您可以使用資源群組來管理 Azure 訂用帳戶中的所有應用程式資源。 如需詳細資訊，請參閱 [Azure Resource Manager概觀][resource-group-overview]。

## <a name="resources"></a>資源

### <a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>SAP 資源
當您設定 SAP 軟體部署時，您需要下列 SAP 資源︰

* SAP Note [1928533]，其中包含：
  * SAP 軟體部署支援的 Azure VM 大小清單
  * Azure VM 大小的重要容量資訊
  * 支援的 SAP 軟體，以及作業系統 (OS) 與資料庫組合
  * Microsoft Azure 上 Windows 和 Linux 所需的 SAP 核心版本

* SAP Note [2015553] 列出 Azure 中 SAP 支援的 SAP 軟體部署先決條件。
* SAP Note [2178632] 包含在 Azure 中針對 SAP 回報的所有監視計量詳細資訊。
* SAP Note [1409604] 包含 Azure 中 Windows 所需的 SAP Host Agent 版本。
* SAP Note [2191498] 包含 Azure 中 Linux 所需的 SAP Host Agent 版本。
* SAP Note [2243692] 包含 Azure 中 Linux 上的 SAP 授權相關資訊。
* SAP Note [1984787] 包含 SUSE LINUX Enterprise Server 12 的一般資訊。
* SAP Note [2002167] 包含 Red Hat Enterprise Linux 7.x 的一般資訊。
* SAP Note [2069760] 包含 Oracle Linux 7.x 的一般資訊。
* SAP Note [1999351] 包含 Azure Enhanced Monitoring Extension for SAP 的其他疑難排解資訊。
* SAP Note [1597355] 包含 Linux 交換空間的一般資訊。
* [Azure SCN 上的 SAP 頁面](https://wiki.scn.sap.com/wiki/x/Pia7Gg)包含新聞和實用資源的集合。
* [SAP Community WIKI](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) 包含 Linux 所需的所有 SAP Note。
* [Azure PowerShell][azure-ps] 所包含的 SAP 特定 PowerShell Cmdlet。
* [Azure CLI][azure-cli] 所包含的 SAP 特定 Azure CLI 命令。

### <a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>Windows 資源
這些 Microsoft 文章涵蓋 Azure 中的 SAP 部署：

* [SAP NetWeaver 的 Azure 虛擬機器規劃和實作指南][planning-guide]
* [適用於 SAP NetWeaver 的 Azure 虛擬機器部署 (本文)][deployment-guide]
* [SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]

## <a name="b3253ee3-d63b-4d74-a49b-185e76c4088e"></a>Azure VM 上的 SAP 軟體部署案例
您有多個選項可用於在 Azure 中部署 VM 和相關聯的磁碟。 一定要了解部署選項之間的差異，因為您可能必須根據您選擇的部署類型，採取不同的步驟來準備您的 VM 進行部署。

### <a name="db477013-9060-4602-9ad4-b0316f8bb281"></a>案例 1：從 Azure Marketplace 為 SAP 部署 VM
您可以使用 Azure Marketplace 中由 Microsoft 或協力廠商提供的映像來部署 VM。 Marketplace 會提供 Windows Server 的一些標準 OS 映像以及不同的 Linux 散發套件。 您也可以部署包含資料庫管理系統 (DBMS) SKU 的映像，例如，Microsoft SQL Server。 如需使用映像搭配 DBMS SKU 的詳細資訊，請參閱[適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]。

下列流程圖顯示從 Azure Marketplace 部署 VM 的 SAP 特定步驟順序：

![使用 Azure Marketplace 中的 VM 映像部署適用於 SAP 系統之 VM 的流程圖][deployment-guide-figure-100]

#### <a name="create-a-virtual-machine-by-using-the-azure-portal"></a>使用 Azure 入口網站建立虛擬機器
透過 Azure Marketplace 中的映像建立新虛擬機器的最簡單方式，就是使用 Azure 入口網站。

1.  移至 <https://portal.azure.com/#create/hub>。  或者，在 Azure 入口網站功能表中，選取 [+ 新增]。
1.  選取 [計算]，然後選取您想要部署的作業系統類型。 例如，Windows Server 2012 R2、SUSE Linux Enterprise Server 12 (SLES 12)、Red Hat Enterprise Linux 7.2 (RHEL 7.2) 或 Oracle Linux 7.2。 預設清單檢視不會顯示所有支援的作業系統。 選取 [查看全部] 以取得完整清單。 如需 SAP 軟體部署支援的作業系統詳細資訊，請參閱 SAP Note [1928533]。
1.  在下一頁上檢閱條款及條件。
1.  在 [選取部署模型] 方塊中，選取 [Resource Manager]。
1.  選取 [建立] 。

除了所有必要的資源 (例如網路介面或儲存體帳戶) 以外，此精靈會引導您完成必要參數設定以建立虛擬機器。 其中一些參數包括︰

1. **基本**：
 * **名稱**：資源名稱 (虛擬機器名稱)。
 * **VM 磁碟類型**：選取作業系統磁碟的磁碟類型。 如果您想要使用進階儲存體作為您的資料磁碟，建議您作業系統磁碟也使用進階儲存體。
 * **使用者名稱和密碼**或 **SSH 公用金鑰**︰輸入在佈建期間建立之使用者的使用者名稱和密碼。 對於 Linux 虛擬機器，您可以輸入要用來登入電腦的公開安全殼層 (SSH) 金鑰。
 * **訂用帳戶**：選取您想要用來佈建新虛擬機器的訂用帳戶。
 * **資源群組**：VM 的資源群組名稱。 您可以輸入新資源群組的名稱，或現有資源群組的名稱。
 * **位置**︰部署新虛擬機器的位置。 如果您想要將虛擬機器連線到內部部署網路，請務必選取將 Azure 連線到內部部署網路的虛擬網路位置。 如需詳細資訊，請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中的 [Microsoft Azure 網路][planning-guide-microsoft-azure-networking]。
1. **大小**：

     如需支援的 VM 類型清單，請參閱 SAP Note [1928533]。 如果您想要使用進階儲存體，請務必選取正確的 VM 類型。 並非所有 VM 類型都支援進階儲存體。 如需詳細資訊，請參閱[Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中的[儲存體：Microsoft Azure 儲存體和資料磁碟][planning-guide-storage-microsoft-azure-storage-and-data-disks]和 [Azure 進階儲存體][planning-guide-azure-premium-storage]。

1. **設定**：
  * **儲存體**
    * **磁碟類型**：選取作業系統磁碟的磁碟類型。 如果您想要使用進階儲存體作為您的資料磁碟，建議您作業系統磁碟也使用進階儲存體。
    * **使用受控磁碟**：如果您想要使用受控磁碟，請選取 [是]。 如需受控磁碟的詳細資訊，請參閱規劃指南中的[受管理磁碟][planning-guide-managed-disks]。
    * **儲存體帳戶**：選取現有的儲存體帳戶或建立新的儲存體帳戶。 並非所有的儲存體類型都適用於執行 SAP 應用程式。 如需儲存體類型的詳細資訊，請參閱[適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]中的 [Microsoft Azure 儲存體][dbms-guide-2.3]。
  * **網路**
    * **虛擬網路**和**子網路**：若要整合虛擬機器與內部網路，請選取連線到內部部署網路的虛擬網路。
    * **公用 IP 位址**︰選取您想要使用的公用 IP 位址，或輸入參數來建立新的公用 IP 位址。 您可以使用公用 IP 位址，透過網際網路存取您的虛擬機器。 請務必也建立網路安全性群組，以便對您的虛擬機器進行安全的存取。
    * **網路安全性群組**：如需詳細資訊，請參閱[使用網路安全性群組控制網路流量流程][virtual-networks-nsg]。
  * **擴充功能**：您可以將虛擬機器擴充功能新增至部署中，藉此安裝安裝。 您不需要在此步驟新增擴充功能。 稍後會安裝 SAP 支援所需的擴充功能。 請參閱本指南的[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5]。
  * **高可用性**：選取可用性設定組，或輸入參數來建立新的可用性設定組。 如需詳細資訊，請參閱 [Azure 可用性設定組][planning-guide-3.2.3]。
  * **監視**
    * **開機診斷**︰您可以選取 [停用] 開機診斷。
    * **客體 OS 診斷**︰您可以選取 [停用] 監視診斷。

1. **摘要**：

  檢閱您的選取項目，然後選取 [確定]。

您的虛擬機器會部署於您選取的資源群組中。

#### <a name="create-a-virtual-machine-by-using-a-template"></a>使用範本建立虛擬機器
您可以使用 [azure-quickstart-templates GitHub repository][azure-quickstart-templates-github] 中發佈的其中一個 SAP 範本來建立虛擬機器。 您可以也使用 [Azure 入口網站][virtual-machines-windows-tutorial]、[PowerShell][virtual-machines-ps-create-preconfigure-windows-resource-manager-vms] 或 [Azure CLI][virtual-machines-linux-tutorial] 來手動建立虛擬機器。

* [**兩層組態 (僅只一部虛擬機器) 範本** (sap-2-tier-marketplace-image)][sap-templates-2-tier-marketplace-image]

  若只要使用一部虛擬機器建立一個兩層系統，請使用此範本。
* [**兩層組態 (僅只一部虛擬機器) 範本 - 受管理磁碟** (sap-2-tier-marketplace-image-md)][sap-templates-2-tier-marketplace-image-md]

  若只要使用一部虛擬機器和受控磁碟建立一個兩層系統，請使用此範本。
* [**三層組態 (多部虛擬機器) 範本** (sap-3-tier-marketplace-image)][sap-templates-3-tier-marketplace-image]

  若要使用多部虛擬機器建立一個三層系統，請使用此範本。
* [**三層組態 (多部虛擬機器) 範本 - 受管理磁碟** (sap-3-tier-marketplace-image-md)][sap-templates-3-tier-marketplace-image-md]

  若要使用多部虛擬機器和受控磁碟建立一個三層系統，請使用此範本。

在 Azure 入口網站中，輸入範本的下列參數：

1. **基本**：
  * **訂用帳戶**：用來部署範本的訂用帳戶。
  * **資源群組**：用來部署範本的資源群組。 您可以建立新的資源群組，也可以選取訂用帳戶中的現有資源群組。
  * **位置**︰部署範本的位置。 如果您選取了現有的資源群組，則會使用該資源群組的位置。

1. **設定**：
  * **SAP 系統識別碼**：SAP 系統識別碼 (SID)。
  * **OS 類型**：您想要部署的作業系統，例如，Windows Server 2012 R2、SUSE Linux Enterprise Server 12 (SLES 12)、Red Hat Enterprise Linux 7.2 (RHEL 7.2) 或 Oracle Linux 7.2。

    清單檢視不會顯示所有支援的作業系統。 如需 SAP 軟體部署支援的作業系統詳細資訊，請參閱 SAP Note [1928533]。
  * **SAP 系統大小**：SAP 系統的大小。

    新系統所提供的 SAP 數量。 如果您不確定系統需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  * **系統可用性**：(僅限三層範本) 系統可用性。

    選取適合高可用性安裝組態的 **HA**。 為 ABAP SAP Central Services (ASCS) 建立兩部資料庫伺服器和兩部伺服器。
  * **儲存體類型**：(僅限兩層範本) 要使用的儲存體類型。

    對於大型系統，我們強烈建議使用 Azure 進階儲存體。 如需儲存體類型的詳細資訊，請參閱下列資源：
      * [針對 SAP DBMS 執行個體使用 Azure 進階 SSD 儲存體][2367194]
      * [適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]中的 [Microsoft Azure 儲存體][dbms-guide-2.3]
      * [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體][storage-premium-storage-preview-portal]
      * [Microsoft Azure 儲存體簡介][storage-introduction]
  * **管理員使用者名稱**和**管理員密碼**：使用者名稱和密碼。
    建立新的使用者，以便登入虛擬機器。
  * **新的或現有的子網路**︰決定要建立新的虛擬網路和子網路，還是使用現有的子網路。 如果您已經有連線到內部部署網路的虛擬網路，請選取 [現有]。
  * **子網路識別碼**：如果您想要將 VM 部署至您已定義應將 VM 指派到其中之目標子網路的現有 VNet，請提供該特定子網路的識別碼。 識別碼通常如下所示：/subscriptions/&lt;訂用帳戶識別碼>/resourceGroups/&lt;資源群組名稱>/providers/Microsoft.Network/virtualNetworks/&lt;虛擬網路名稱>/subnets/&lt;子網路名稱>

1. **條款及條件**：  
    檢閱並接受法律條款。

1.  選取 [購買]。

使用 Azure Marketplace 中的映像時，預設會部署 Azure VM 代理程式。

#### <a name="configure-proxy-settings"></a>進行 Proxy 設定
根據內部部署網路的設定方式，您可能需要在您的 VM 上設定 Proxy。 如果 VM 已透過 VPN 或 ExpressRoute 連線到內部部署網路，VM 可能無法存取網際網路，而且無法下載所需的擴充功能或收集監視資料。 如需詳細資訊，請參閱[設定 Proxy][deployment-guide-configure-proxy]。

#### <a name="join-a-domain-windows-only"></a>加入網域 (僅限 Windows)
如果 Azure 部署連線到內部部署 Active Directory 或 DNS 執行個體，透過 Azure 站對站 VPN 連線或 ExpressRoute (在 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中稱為「跨單位」)，則 VM 應該加入內部部署網域。 如需這項工作相關考量的詳細資訊，請參閱[將 VM 加入內部部署網域 (僅限 Windows)][deployment-guide-4.3]。

#### <a name="ec323ac3-1de9-4c3a-b770-4ff701def65b"></a>設定監視
若要確保 SAP 支援您的環境，請如[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 所述，設定 Azure Enhanced Monitoring Extension for SAP。 請從 [SAP 資源][deployment-guide-2.2]所列的資源中，檢查 SAP 監視的必要條件，以及所需的最低 SAP 核心和 SAP Host Agent 版本。

#### <a name="monitoring-check"></a>監視檢查
如[針對設定端對端監視進行檢查及疑難排解][deployment-guide-troubleshooting-chapter]所述，檢查監視是否正常運作。

#### <a name="post-deployment-steps"></a>部署後步驟
建立 VM 及部署 VM 之後，您需要在 VM 中安裝所需的軟體元件。 由於這類 VM 部署中的部署/軟體安裝順序，要安裝的軟體必須已可在 Azure 中或另一部 VM 上使用，或成為可以附加的磁碟。 否則，考慮使用可提供內部部署資產 (安裝共用) 連線的跨單位案例。

在 Azure 中部署您的 VM 之後，請遵循相同的指導方針和工具在 VM 上安裝 SAP 軟體，就像在內部部署環境中所做的一樣。 若要在 Azure VM 上安裝 SAP 軟體，SAP 和 Microsoft 建議將 SAP 安裝媒體上傳並儲存在 Azure VHD 或受控磁碟上，或建立作為檔案伺服器並包含所有必要 SAP 安裝媒體的 Azure VM。

### <a name="54a1fc6d-24fd-4feb-9c57-ac588a55dff2"></a>案例 2：使用自訂映像為 SAP 部署 VM
因為不同版本的作業系統或 DBMS 有不同的修補需求，所以您在 Azure Marketplace 中找到的映像可能無法滿足您的需求。 您可以改為使用自己的 OS/DBMS VM 映像來建立 VM，稍後再加以部署。
您可以使用不同的步驟來建立適用於 Linux 的私人映像，而不是建立適用於 Windows 的映像。

- - -
> ![Windows][Logo_Windows] Windows
>
> 若要準備可用來部署多部虛擬機器的 Windows 映像，必須在內部部署 VM 上將 Windows 設定 (如 Windows SID 和主機名稱) 抽象化或一般化。 您可以使用 [sysprep](https://msdn.microsoft.com/library/hh825084.aspx) 來執行此作業。
>
> ![Linux][Logo_Linux] Linux
>
> 若要準備可用來部署多部虛擬機器的 Linux 映像，必須在內部部署 VM 上抽象化或一般化一些 Linux 設定。 您可以使用 `waagent -deprovision` 來執行此作業。 如需詳細資訊，請參閱[擷取在 Azure 上執行的 Linux 虛擬機器][virtual-machines-linux-capture-image]和 [Azure Linux 代理程式使用者指南][virtual-machines-linux-agent-user-guide-command-line-options]。
>
>

- - -
您可以準備和建立自訂映像，然後使用它來建立多個新 VM。 如需詳細資訊，請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]。 藉由使用 SAP Software Provision Manager 來安裝新的 SAP 系統、從附加至虛擬機器的磁碟還原資料庫備份，或直接從 Azure 儲存體還原資料庫備份 (如果 DBMS 支援的話)，來設定您的資料庫內容。 如需詳細資訊，請參閱[適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]。 如果您已在內部部署 VM (特別針對二層系統) 中安裝 SAP 系統，您可以在部署 Azure VM 之後，使用 SAP Software Provisioning Manager 支援的「系統重新命名」程序來調整 SAP 系統設定 (SAP Note [1619720])。 否則，您可以在部署 Azure VM 之後安裝 SAP 軟體。

下列流程圖顯示從自訂映像部署 VM 的 SAP 特定步驟順序：

![使用私人 Marketplace 中的 VM 映像部署適用於 SAP 系統之 VM 的流程圖][deployment-guide-figure-300]

#### <a name="create-a-virtual-machine-by-using-the-azure-portal"></a>使用 Azure 入口網站建立虛擬機器
要從受控磁碟映像建立新的虛擬機器，最簡單方式就是使用 Azure 入口網站。 如需如何建立受控磁碟映像的詳細資訊，請參閱[在 Azure 中擷取一般化 VM 的受控映像](https://docs.microsoft.com/azure/virtual-machines/windows/capture-image-resource)。

1.  移至 <https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Compute%2Fimages>。 或者，在 Azure 入口網站的功能表中選取 [映像]。
1.  選取要部署的受控磁碟映像，然後按一下 [建立 VM]。

除了所有必要的資源 (例如網路介面或儲存體帳戶) 以外，此精靈會引導您完成必要參數設定以建立虛擬機器。 其中一些參數包括︰

1. **基本**：
 * **名稱**：資源名稱 (虛擬機器名稱)。
 * **VM 磁碟類型**：選取作業系統磁碟的磁碟類型。 如果您想要使用進階儲存體作為您的資料磁碟，建議您作業系統磁碟也使用進階儲存體。
 * **使用者名稱和密碼**或 **SSH 公用金鑰**︰輸入在佈建期間建立之使用者的使用者名稱和密碼。 對於 Linux 虛擬機器，您可以輸入要用來登入電腦的公開安全殼層 (SSH) 金鑰。
 * **訂用帳戶**：選取您想要用來佈建新虛擬機器的訂用帳戶。
 * **資源群組**：VM 的資源群組名稱。 您可以輸入新資源群組的名稱，或現有資源群組的名稱。
 * **位置**︰部署新虛擬機器的位置。 如果您想要將虛擬機器連線到內部部署網路，請務必選取將 Azure 連線到內部部署網路的虛擬網路位置。 如需詳細資訊，請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中的 [Microsoft Azure 網路][planning-guide-microsoft-azure-networking]。
1. **大小**：

     如需支援的 VM 類型清單，請參閱 SAP Note [1928533]。 如果您想要使用進階儲存體，請務必選取正確的 VM 類型。 並非所有 VM 類型都支援進階儲存體。 如需詳細資訊，請參閱[Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中的[儲存體：Microsoft Azure 儲存體和資料磁碟][planning-guide-storage-microsoft-azure-storage-and-data-disks]和 [Azure 進階儲存體][planning-guide-azure-premium-storage]。

1. **設定**：
  * **儲存體**
    * **磁碟類型**：選取作業系統磁碟的磁碟類型。 如果您想要使用進階儲存體作為您的資料磁碟，建議您作業系統磁碟也使用進階儲存體。
    * **使用受控磁碟**：如果您想要使用受控磁碟，請選取 [是]。 如需受控磁碟的詳細資訊，請參閱規劃指南中的[受管理磁碟][planning-guide-managed-disks]。
  * **網路**
    * **虛擬網路**和**子網路**：若要整合虛擬機器與內部網路，請選取連線到內部部署網路的虛擬網路。
    * **公用 IP 位址**︰選取您想要使用的公用 IP 位址，或輸入參數來建立新的公用 IP 位址。 您可以使用公用 IP 位址，透過網際網路存取您的虛擬機器。 請務必也建立網路安全性群組，以便對您的虛擬機器進行安全的存取。
    * **網路安全性群組**：如需詳細資訊，請參閱[使用網路安全性群組控制網路流量流程][virtual-networks-nsg]。
  * **擴充功能**：您可以將虛擬機器擴充功能新增至部署中，藉此安裝安裝。 您不需要在此步驟新增擴充功能。 稍後會安裝 SAP 支援所需的擴充功能。 請參閱本指南的[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5]。
  * **高可用性**：選取可用性設定組，或輸入參數來建立新的可用性設定組。 如需詳細資訊，請參閱 [Azure 可用性設定組][planning-guide-3.2.3]。
  * **監視**
    * **開機診斷**︰您可以選取 [停用] 開機診斷。
    * **客體 OS 診斷**︰您可以選取 [停用] 監視診斷。

1. **摘要**：

  檢閱您的選取項目，然後選取 [確定]。

您的虛擬機器會部署於您選取的資源群組中。
#### <a name="create-a-virtual-machine-by-using-a-template"></a>使用範本建立虛擬機器
若要透過 Azure 入口網站使用私人 OS 映像來建立部署，請使用下列其中一個 SAP 範本。 這些是 [azure-quickstart-templates GitHub 儲存機制][azure-quickstart-templates-github]中發佈的範本。 您可以也使用 [PowerShell][virtual-machines-upload-image-windows-resource-manager] 手動建立虛擬機器。

* [**兩層組態 (僅限一部虛擬機器) 範本** (sap-2-tier-user-image)][sap-templates-2-tier-user-image]

  若只要使用一部虛擬機器建立一個兩層系統，請使用此範本。
* [**兩層組態 (僅只一部虛擬機器) 範本 - 受管理磁碟映像** (sap-2-tier-user-image-md)][sap-templates-2-tier-user-image-md]

  若只要使用一部虛擬機器和受控磁碟映像建立一個兩層系統，請使用此範本。
* [**三層組態 (多部虛擬機器) 範本** (sap-3-tier-user-image)][sap-templates-3-tier-user-image]

  若要使用多部虛擬機器或自己的 OS 映像建立一個三層系統，請使用此範本。
* [**三層組態 (多部虛擬機器) 範本 - 受管理磁碟映像** (sap-3-tier-user-image-md)][sap-templates-3-tier-user-image-md]

  若要使用多部虛擬機器或自己的 OS 映像和受控磁碟映像建立一個三層系統，請使用此範本。

在 Azure 入口網站中，輸入範本的下列參數：

1. **基本**：
  * **訂用帳戶**：用來部署範本的訂用帳戶。
  * **資源群組**：用來部署範本的資源群組。 您可以建立新的資源群組，或選取訂用帳戶中的現有資源群組。
  * **位置**︰部署範本的位置。 如果您選取了現有的資源群組，則會使用該資源群組的位置。
1. **設定**：
  * **SAP 系統識別碼**：SAP 系統識別碼。
  * **OS 類型**：您想要部署的作業系統類型 (Windows 或 Linux)。
  * **SAP 系統大小**：SAP 系統的大小。

    新系統所提供的 SAP 數量。 如果您不確定系統需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  * **系統可用性**：(僅限三層範本) 系統可用性。

    選取適合高可用性安裝組態的 **HA**。 為 ASCS 建立兩部資料庫伺服器和兩部伺服器。
  * **儲存體類型**：(僅限兩層範本) 要使用的儲存體類型。

    對於大型系統，我們強烈建議使用 Azure 進階儲存體。 如需儲存體類型的詳細資訊，請參閱下列資源：
      * [針對 SAP DBMS 執行個體使用 Azure 進階 SSD 儲存體][2367194]
      * [適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]中的 [Microsoft Azure 儲存體][dbms-guide-2.3]
      * [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體][storage-premium-storage-preview-portal]
      * [Microsoft Azure 儲存體簡介][storage-introduction]
  * **使用者映像 VHD URI** (僅限非受控磁碟映像範本)：私人 OS 映像 VHD 的 URI，例如 https://&lt;accountname&gt;.blob.core.windows.net/vhds/userimage.vhd。
  * **使用者映像儲存體帳戶** (僅限非受控磁碟映像範本)：私人 OS 映像儲存所在的儲存體帳戶名稱，例如 &lt;accountname&gt; in https://&lt;accountname&gt;.blob.core.windows.net/vhds/userimage.vhd。
  * **userImageId** (僅限非受控磁碟映像範本)：您想要使用的受控磁碟映像的識別碼。
  * **管理員使用者名稱**和**管理員密碼**：使用者名稱和密碼。

    建立新的使用者，以便登入虛擬機器。
  * **新的或現有的子網路**︰決定要建立新的虛擬網路和子網路，還是使用現有的子網路。 如果您已經有連線到內部部署網路的虛擬網路，請選取 [現有]。
  * **子網路識別碼**：如果您想要將 VM 部署至您已定義應將 VM 指派到其中之目標子網路的現有 VNet，請提供該特定子網路的識別碼。 識別碼通常如下所示：/subscriptions/&lt;訂用帳戶識別碼>/resourceGroups/&lt;資源群組名稱>/providers/Microsoft.Network/virtualNetworks/&lt;虛擬網路名稱>/subnets/&lt;子網路名稱>

1. **條款及條件**：  
    檢閱並接受法律條款。

1.  選取 [購買]。

#### <a name="install-the-vm-agent-linux-only"></a>安裝 VM 代理程式 (僅限 Linux)
若要使用上一節所述的範本，必須已在使用者映像中安裝 Linux 代理程式，否則部署會失敗。 如[下載、安裝和啟用 Azure VM 代理程式][deployment-guide-4.4]所述，下載 VM 代理程式並安裝於使用者映像中。 如果您未使用範本，稍後也可以安裝 VM 代理程式。

#### <a name="join-a-domain-windows-only"></a>加入網域 (僅限 Windows)
如果 Azure 部署連線到內部部署 Active Directory 或 DNS 執行個體，透過 Azure 站對站 VPN 連線或 Azure ExpressRoute (在[Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中稱為「跨單位」)，則 VM 應該加入內部部署網域。 如需這個步驟相關考量的詳細資訊，請參閱[將 VM 加入內部部署網域 (僅限 Windows)][deployment-guide-4.3]。

#### <a name="configure-proxy-settings"></a>進行 Proxy 設定
根據內部部署網路的設定方式，您可能需要在您的 VM 上設定 Proxy。 如果 VM 已透過 VPN 或 ExpressRoute 連線到內部部署網路，VM 可能無法存取網際網路，而且無法下載所需的擴充功能或收集監視資料。 如需詳細資訊，請參閱[設定 Proxy][deployment-guide-configure-proxy]。

#### <a name="configure-monitoring"></a>設定監視
若要確保 SAP 支援您的環境，請如[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 所述，設定 Azure Enhanced Monitoring Extension for SAP。 請從 [SAP 資源][deployment-guide-2.2]所列的資源中，檢查 SAP 監視的必要條件，以及所需的最低 SAP 核心和 SAP Host Agent 版本。

#### <a name="monitoring-check"></a>監視檢查
如[針對設定端對端監視進行檢查及疑難排解][deployment-guide-troubleshooting-chapter]所述，檢查監視是否正常運作。


### <a name="a9a60133-a763-4de8-8986-ac0fa33aa8c1"></a>案例 3：使用非一般化 Azure VHD 搭配 SAP 來移動內部部署 VM
在此情況下，您會規劃將特定 SAP 系統從內部部署環境移至 Azure。 將包含 OS、SAP 二進位檔和最終 DBMS 二進位檔的 VHD，以及包含 DBMS 資料和記錄檔的 VHD 上傳至 Azure，即可達成。 不同於[案例 2：使用 SAP 適用的自訂映像部署 VM][deployment-guide-3.3] 中所述的案例，在此情況下，您可以在 Azure VM 中保留主機名稱、SAP SID 和 SAP 使用者帳戶，因為這些項目設定於內部部署環境中。 您不需要將 OS 一般化。 此案例最適用於跨單位案例，其中部分的 SAP 架構是在內部部署執行，部分則在 Azure 上執行。

在此案例中，**不會**在部署期間自動安裝 VM 代理程式。 由於 VM 代理程式和 Azure Enhanced Monitoring Extension for SAP 是在 Azure 上執行 SAP NetWeaver 的必要元件，因此在建立虛擬機器之後，您必須手動下載、安裝和啟用這兩個元件。

如需 Azure VM 代理程式的詳細資訊，請參閱下列資源。

- - -
> ![Windows][Logo_Windows] Windows
>
> [Azure 虛擬機器代理程式概觀][virtual-machines-windows-agent-user-guide]
>
> ![Linux][Logo_Linux] Linux
>
> [Azure Linux 代理程式使用者指南][virtual-machines-linux-agent-user-guide]
>
>

- - -

下列流程圖說明使用非一般化 Azure VHD 移動內部部署 VM 的步驟順序︰

![使用 VM 磁碟部署適用於 SAP 系統之 VM 的流程圖][deployment-guide-figure-400]

如果磁碟已上傳並在 Azure 中定義 (請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide])，請執行後面幾節所述的工作。

#### <a name="create-a-virtual-machine"></a>建立虛擬機器
若要透過 Azure 入口網站使用私人 OS 磁碟來建立部署，請使用 [azure-quickstart-templates Github 儲存機制][azure-quickstart-templates-github]中所發佈的 SAP 範本。 您可以也使用 PowerShell 手動建立虛擬機器。

* [**兩層組態 (僅限一部虛擬機器) 範本**  (sap-2-tier-user-disk)][sap-templates-2-tier-os-disk]

  若只要使用一部虛擬機器建立一個兩層系統，請使用此範本。
* [**兩層組態 (僅限一部虛擬機器) 範本 - 受管理磁碟** (sap-2-tier-user-disk-md)][sap-templates-2-tier-os-disk-md]

  若只要使用一部虛擬機器和受控磁碟建立一個兩層系統，請使用此範本。

在 Azure 入口網站中，輸入範本的下列參數：

1. **基本**：
  * **訂用帳戶**：用來部署範本的訂用帳戶。
  * **資源群組**：用來部署範本的資源群組。 您可以建立新的資源群組，或選取訂用帳戶中的現有資源群組。
  * **位置**︰部署範本的位置。 如果您選取了現有的資源群組，則會使用該資源群組的位置。
1. **設定**：
  * **SAP 系統識別碼**：SAP 系統識別碼。
  * **OS 類型**：您想要部署的作業系統類型 (Windows 或 Linux)。
  * **SAP 系統大小**：SAP 系統的大小。

    新系統所提供的 SAP 數量。 如果您不確定系統需要多少 SAP，請詢問您的 SAP 技術合作夥伴或系統整合者。
  * **儲存體類型**：(僅限兩層範本) 要使用的儲存體類型。

    對於大型系統，我們強烈建議使用 Azure 進階儲存體。 如需儲存體類型的詳細資訊，請參閱下列資源：
      * [針對 SAP DBMS 執行個體使用 Azure 進階 SSD 儲存體][2367194]
      * [適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署][dbms-guide]中的 [Microsoft Azure 儲存體][dbms-guide-2.3]
      * [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體][storage-premium-storage-preview-portal]
      * [Microsoft Azure 儲存體簡介][storage-introduction]
  * **OS 磁碟 VHD URI** (僅限非受控磁碟範本)：私人 OS 磁碟的 URI，例如 https://&lt;accountname&gt;.blob.core.windows.net/vhds/osdisk.vhd。
  * **作業系統磁碟受控磁碟識別碼** (僅限非受控磁碟範本)：作業系統磁碟 /subscriptions/92d102f7-81a5-4df7-9877-54987ba97dd9/resourceGroups/group/providers/Microsoft.Compute/disks/WIN 的受控磁碟識別碼
  * **新的或現有的子網路**︰決定要建立新的虛擬網路和子網路，還是使用現有的子網路。 如果您已經有連線到內部部署網路的虛擬網路，請選取 [現有]。
  * **子網路識別碼**：如果您想要將 VM 部署至您已定義應將 VM 指派到其中之目標子網路的現有 VNet，請提供該特定子網路的識別碼。 識別碼通常如下所示：/subscriptions/&lt;訂用帳戶識別碼>/resourceGroups/&lt;資源群組名稱>/providers/Microsoft.Network/virtualNetworks/&lt;虛擬網路名稱>/subnets/&lt;子網路名稱>

1. **條款及條件**：  
    檢閱並接受法律條款。

1.  選取 [購買]。

#### <a name="install-the-vm-agent"></a>安裝 VM 代理程式
若要使用上一節所述的範本，必須在 OS 磁碟上安裝 VM 代理程式，否則部署會失敗。 如[下載、安裝和啟用 Azure VM 代理程式][deployment-guide-4.4]所述，下載 VM 代理程式並安裝於 VM 中。

如果您未使用上一節所述的範本，則之後也可以安裝 VM 代理程式。

#### <a name="join-a-domain-windows-only"></a>加入網域 (僅限 Windows)
如果 Azure 部署連線到內部部署 Active Directory 或 DNS 執行個體，透過 Azure 站對站 VPN 連線或 ExpressRoute (在 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide]中稱為「跨單位」)，則 VM 應該加入內部部署網域。 如需這項工作相關考量的詳細資訊，請參閱[將 VM 加入內部部署網域 (僅限 Windows)][deployment-guide-4.3]。

#### <a name="configure-proxy-settings"></a>進行 Proxy 設定
根據內部部署網路的設定方式，您可能需要在您的 VM 上設定 Proxy。 如果 VM 已透過 VPN 或 ExpressRoute 連線到內部部署網路，VM 可能無法存取網際網路，而且無法下載所需的擴充功能或收集監視資料。 如需詳細資訊，請參閱[設定 Proxy][deployment-guide-configure-proxy]。

#### <a name="configure-monitoring"></a>設定監視
若要確保 SAP 支援您的環境，請如[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 所述，設定 Azure Enhanced Monitoring Extension for SAP。 請從 [SAP 資源][deployment-guide-2.2]所列的資源中，檢查 SAP 監視的必要條件，以及所需的最低 SAP 核心和 SAP Host Agent 版本。

#### <a name="monitoring-check"></a>監視檢查
如[針對設定端對端監視進行檢查及疑難排解][deployment-guide-troubleshooting-chapter]所述，檢查監視是否正常運作。

## <a name="update-the-monitoring-configuration-for-sap"></a>更新 SAP 的監視組態
在下列任何一個案例中，更新 SAP 監視組態︰
* 聯合 Microsoft/SAP 小組延伸監視功能，並要求增加或減少計數器。
* Microsoft 推出新版本的基礎 Azure 基礎結構來提供監視資料，而 Azure Enhanced Monitoring Extension for SAP 必須配合這些變更。
* 您將額外的資料磁碟掛接到 Azure VM 或是移除資料磁碟。 在此案例中，更新儲存體相關資料的集合。 藉由新增或刪除端點或將 IP 位址指派給 VM 來變更組態，就不會影響監視組態。
* 您可變更 Azure VM 的大小 (例如從大小 A5 變更成任何其他 VM 大小)。
* 您可以在 Azure VM 中新增網路介面。

若要更新監視設定，請依照[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 中的步驟來更新監視基礎結構。

## <a name="detailed-tasks-for-sap-software-deployment"></a>SAP 軟體部署的詳細工作
本節包含在設定和部署程序中執行特定工作的詳細步驟。

### <a name="604bcec2-8b6e-48d2-a944-61b0f5dee2f7"></a>部署 Azure PowerShell Cmdlet
1.  前往 [Microsoft Azure 下載](https://azure.microsoft.com/downloads/)。
1.  在**命令列工具**的 **PowerShell** 之下，選取 [Windows 安裝]。
1.  在 [Microsoft 下載管理員] 對話方塊中，針對已下載的檔案 (例如，WindowsAzurePowershellGet.3f.3f.3fnew.exe) 選取 [執行]。
1.  若要執行 Microsoft Web Platform Installer (Microsoft Web PI)，請選取 [是]。
1.  如下所示的頁面隨即出現︰

  ![Azure PowerShell Cmdlet 的安裝頁面][deployment-guide-figure-500]<a name="figure-5"></a>

1.  選取 [安裝]，然後接受 Microsoft 軟體授權條款。
1.  已安裝 PowerShell。 選取 [完成]以關閉安裝精靈。

經常檢查 PowerShell Cmdlet 的更新 (通常每個月更新一次)。 檢查更新的最簡單方式是繼續執行安裝步驟，直到步驟 5 所示的安裝頁面為止。 Cmdlet 的發行日期和版次號碼包含在步驟 5 所示的頁面上。 除非 SAP Note [1928533] 或 SAP Note [2015553] 另外指定，否則建議使用最新版的 Azure PowerShell Cmdlet。

若要檢查電腦上安裝的 Azure PowerShell Cmdlet 版本，請執行此 PowerShell 命令︰
```powershell
(Get-Module AzureRm.Compute).Version
```
結果如下所示：

![Azure PowerShell Cmdlet 版本檢查的結果][deployment-guide-figure-600]
<a name="figure-6"></a>

如果電腦上安裝的 Azure Cmdlet 版本是最新版本，則安裝精靈的第一頁會將 **(已安裝)** 新增至產品標題 (請參閱下列螢幕擷取畫面)。 您的 PowerShell Azure Cmdlet 都是最新的。 若要關閉安裝精靈，請選取 [結束]。

![Azure PowerShell Cmdlet 的安裝頁面，表示已安裝最新版的 Azure PowerShell Cmdlet][deployment-guide-figure-700]
<a name="figure-7"></a>

### <a name="1ded9453-1330-442a-86ea-e0fd8ae8cab3"></a>部署 Azure CLI
1.  前往 [Microsoft Azure 下載](https://azure.microsoft.com/downloads/)。
1.  在**命令列工具**的 **Azure 命令列介面**之下，選取您作業系統的 [安裝] 連結。
1.  在 [Microsoft 下載管理員] 對話方塊中，針對已下載的檔案 (例如，WindowsAzureXPlatCLI.3f.3f.3fnew.exe) 選取 [執行]。
1.  若要執行 Microsoft Web Platform Installer (Microsoft Web PI)，請選取 [是]。
1.  如下所示的頁面隨即出現︰

  ![Azure PowerShell Cmdlet 的安裝頁面][deployment-guide-figure-500]<a name="figure-5"></a>

1.  選取 [安裝]，然後接受 Microsoft 軟體授權條款。
1.  已安裝 Azure CLI。 選取 [完成]以關閉安裝精靈。

經常檢查 Azure CLI 的更新 (通常每個月更新一次)。 檢查更新的最簡單方式是繼續執行安裝步驟，直到步驟 5 所示的安裝頁面為止。

若要檢查電腦上安裝的 Azure CLI 版本，請執行此命令︰
```
azure --version
```

結果如下所示：

![Azure CLI 版本檢查的結果][deployment-guide-figure-760]
<a name="0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda"></a>

### <a name="31d9ecd6-b136-4c73-b61e-da4a29bbc9cc"></a>將 VM 加入內部部署網域 (僅限 Windows)
如果在跨單位案例 (於 Azure 擴充內部部署 Active Directory 和 DNS) 中部署 SAP VM，則必須將 VM 加入內部部署網域。 將 VM 加入內部部署網域的詳細步驟，以及成為內部部署網域成員所需的其他軟體，因客戶而有所不同。 通常要將 VM 加入內部部署網域，您必須安裝其他軟體 (例如反惡意程式碼軟體以及備份或監視軟體)。

在此案例中，您需要確定如果在 VM 加入您環境中的網域時強制使用網際網路 Proxy 設定，則客體 VM 中的 Windows 本機系統帳戶 (S-1-5-18) 會有相同的 Proxy 設定。 最簡單的選項是使用適用於網域內系統的網域群組原則來強制使用 Proxy。

### <a name="c7cbb0dc-52a4-49db-8e03-83e7edc2927d"></a>下載、安裝和啟用 Azure VM 代理程式
對於從未一般化的 OS 映像 (例如不是源自於 Windows 系統準備 (或 sysprep) 工具的映像) 部署的虛擬機器，您必須手動下載、安裝和啟用 Azure VM 代理程式。

如果您從 Azure Marketplace 部署 VM，就不需要執行此步驟。 Azure Marketplace 中的映像已經有 Azure VM 代理程式。

#### <a name="b2db5c9a-a076-42c6-9835-16945868e866"></a>Windows
1.  下載 Azure VM 代理程式︰
  1.  下載 [Azure VM 代理程式安裝程式套件](https://go.microsoft.com/fwlink/?LinkId=394789)。
  1.  在個人電腦或伺服器本機上儲存 VM 代理程式 MSI 套件。
1.  安裝 Azure VM 代理程式︰
  1.  使用遠端桌面通訊協定 (RDP) 連線到已部署的 Azure VM。
  1.  在 VM 上開啟「Windows 檔案總管」視窗，然後選取 VM 代理程式之 MSI 檔案的目標目錄。
  1.  將 Azure VM 代理程式安裝程式 MSI 檔案從本機電腦/伺服器拖放到 VM 上 VM 代理程式的目標目錄。
  1.  按兩下 VM 上的 MSI 檔案。
1.  對於加入內部部署網域的 VM，請確定最終網際網路 Proxy 設定也適用於 VM 中的 Windows 本機系統帳戶 (S-1-5-18)，如[設定 Proxy][deployment-guide-configure-proxy] 所述。 VM 代理程式會在此內容中執行，而且必須可以連線到 Azure。

更新 Azure VM 代理程式時，並不需要使用者介入。 VM 代理程式會自動更新，不需要重新啟動 VM。

#### <a name="6889ff12-eaaf-4f3c-97e1-7c9edc7f7542"></a>Linux
使用下列命令來安裝適用於 Linux 的 VM 代理程式：

* **SUSE Linux Enterprise Server (SLES)**

  ```
  sudo zypper install WALinuxAgent
  ```

* **Red Hat Enterprise Linux (RHEL) 或 Oracle Linux**

  ```
  sudo yum install WALinuxAgent
  ```

如果已安裝代理程式，若要更新 Azure Linux 代理程式，請執行[將 VM 上的 Azure Linux 代理程式更新為 GitHub 中的最新版本][virtual-machines-linux-update-agent]所述的步驟。

### <a name="baccae00-6f79-4307-ade4-40292ce4e02d"></a>設定 Proxy
在 Windows 中設定 Proxy 所採取的步驟，不同於您在 Linux 中設定 Proxy 的方式。

#### <a name="windows"></a>Windows
您也必須針對本機系統帳戶進行正確的 Proxy 設定，才能存取網際網路。 如果您的 Proxy 設定不是透過群組原則設定，您可以為本機系統帳戶進行這些設定。

1. 移至 [開始]，輸入 **gpedit.msc**，然後選取 **Enter**。
1. 選取 [電腦設定] > [系統管理範本] > [Windows 元件] > [Internet Explorer]。 請確定 [為每台電腦建立 Proxy 設定 - 而不是每個使用者] 設定已停用或未設定。
1. 在 [控制台] 中，移至 [網路和共用中心] > [網際網路選項]。
1. 在 [連線] 索引標籤上，選取 [區域網路設定] 按鈕。
1. 清除 [自動偵測設定] 核取方塊。
1. 選取 [為您的 LAN 使用 Proxy 伺服器] 核取方塊，然後輸入 Proxy 位址和連接埠
1. 選取 [進階] 按鈕。
1. 在 [例外狀況] 方塊中，輸入 IP 位址 **168.63.129.16**。 選取 [確定] 。

#### <a name="linux"></a>Linux
在 Microsoft Azure 客體代理程式的組態檔 (位於 \\etc\\waagent.conf) 中設定正確的 Proxy。

設定下列參數︰

1.  **HTTP Proxy 主機**。 例如，將它設為 **proxy.corp.local**。
  ```
  HttpProxy.Host=<proxy host>

  ```
1.  **HTTP Proxy 連接埠**。 例如，將它設定為 **80**。
  ```
  HttpProxy.Port=<port of the proxy host>

  ```
1.  重新啟動代理程式。

  ```
  sudo service waagent restart
  ```

\\etc\\waagent.conf 中的 Proxy 設定也適用於所需的 VM 擴充功能。 如果您想要使用 Azure 儲存機制，請確定這些儲存機制的流量不會經過內部部署內部網路。 如果您已建立使用者定義的路由來啟用強制通道，請務必新增路由，以將傳送給儲存機制的流量直接遞送到網際網路，而不透過站對站 VPN 連線。

* **SLES**

  您也需要新增 \\etc\\regionserverclnt.cfg 中所列 IP 位址的路由。 下圖顯示一個範例：

  ![強制通道][deployment-guide-figure-50]


* **RHEL**

  您還需要一併為 \\etc\\yum.repos.d\\rhui-load-balancers 中所列的主機 IP 位址新增路由。 如需範例，請參閱上圖。

* **Oracle Linux**

  Azure 上沒有任何 Oracle Linux 的存放庫。 您要為 Oracle Linux 設定您自己的存放庫，或使用公用存放庫。

如需有關使用者定義的路由詳細資訊，請參閱[使用者定義的路由和 IP 轉送][virtual-networks-udr-overview]。

### <a name="d98edcd3-f2a1-49f7-b26a-07448ceb60ca"></a>設定 Azure Enhanced Monitoring Extension for SAP
在您如 [SAP on Azure 的 VM 部署案例][deployment-guide-3]所述準備好 VM 之後，Azure VM 代理程式就會安裝於虛擬機器上。 下一個步驟是部署 Azure Enhanced Monitoring Extension for SAP (位於全球 Azure 資料中心的 Azure 擴充功能儲存機制)。 如需詳細資訊，請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南][planning-guide-9.1]。

您可以使用 PowerShell 或 Azure CLI 安裝和設定 Azure Enhanced Monitoring Extension for SAP。 若要使用 Windows 電腦在 Windows 或 Linux VM 上安裝該擴充功能，請參閱 [Azure PowerShell][deployment-guide-4.5.1]。 若要使用 Linux 桌上型電腦在 Linux VM 上安裝該擴充功能，請參閱 [Azure CLI][deployment-guide-4.5.2]。

#### <a name="987cf279-d713-4b4c-8143-6b11589bb9d4"></a>適用於 Linux 和 Windows VM 的 Azure PowerShell
若要使用 PowerShell 安裝 Azure Enhanced Monitoring Extension for SAP：

1. 確定您已安裝最新版的 Azure PowerShell Cmdlet。 如需詳細資訊，請參閱[部署 Azure PowerShell Cmdlet][deployment-guide-4.1]。  
1. 執行下列 PowerShell Cmdlet。
    如需可用環境的清單，請執行 `commandlet Get-AzureRmEnvironment`。 如果您想要使用全域 Azure，則您的環境是 **AzureCloud**。 若為中國的 Azure，請選取 **AzureChinaCloud**。

    ```powershell
    $env = Get-AzureRmEnvironment -Name <name of the environment>
    Connect-AzureRmAccount -Environment $env
    Set-AzureRmContext -SubscriptionName <subscription name>

    Set-AzureRmVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
    ```

輸入您的帳戶資料並識別 Azure 虛擬機器之後，指令碼會部署所需的擴充功能，並啟用所需的功能。 這可能需要數分鐘的時間。
如需有關 `Set-AzureRmVMAEMExtension` 的詳細資訊，請參閱 [Set-AzureRmVMAEMExtension][msdn-set-azurermvmaemextension]。

![成功執行 SAP 特定 Azure Cmdlet Set-AzureRmVMAEMExtension][deployment-guide-figure-900]

`Set-AzureRmVMAEMExtension` 設定可以進行設定 SAP 主機監視的所有步驟。

指令碼輸出包含下列資訊︰

* 確認已設定 OS 磁碟和所有額外資料磁碟的監視。
* 接下來的兩則訊息會確認特定儲存體帳戶的儲存體計量組態。
* 其中一行輸出會提供監視組態的實際更新狀態。
* 另一行輸出會確認已部署或已更新組態。
* 輸出的最後一行是參考資訊。 其顯示測試監視組態的選項。
* 若要確認 Azure Enhanced Monitoring 的所有步驟是否已成功執行，以及「Azure 基礎結構」是否提供必要的資料，請繼續進行 Azure Enhanced Monitoring Extension for SAP 的整備檢查 (如 [Azure Enhanced Monitoring for SAP 整備檢查][deployment-guide-5.1]所述)。
* 等候 15-30 分鐘的時間，讓 Azure 診斷收集相關的資料。

#### <a name="408f3779-f422-4413-82f8-c57a23b4fc2f"></a>適用於 Linux VM 的 Azure CLI
若要使用 Azure CLI 安裝 Azure Enhanced Monitoring Extension for SAP：

   1. 以[安裝 Azure 傳統 CLI][azure-cli] 中所述的方式安裝 Azure 傳統 CLI。
   1. 使用您的 Azure 帳戶進行登入：

      ```
      azure login
      ```

   1. 切換到 Azure Resource Manager 模式：

      ```
      azure config mode arm
      ```

   1. 啟用 Azure Enhanced Monitoring：

      ```
      azure vm enable-aem <resource-group-name> <vm-name>
      ```

1. 使用 Azure CLI 2.0 進行安裝

   1. 如[安裝 Azure CLI 2.0][azure-cli-2] 所述，安裝 Azure CLI 2.0。
   1. 使用您的 Azure 帳戶進行登入：

      ```
      az login
      ```

   1. 安裝 Azure CLI AEM 擴充功能
  
      ```
      az extension add --name aem
      ```
  
   1. 安裝擴充功能
  
      ```
      az vm aem set -g <resource-group-name> -n <vm name>
      ```

1. 確認 Azure Enhanced Monitoring Extension 在 Azure Linux VM 上為作用中狀態。 檢查 \\var\\lib\\AzureEnhancedMonitor\\PerfCounters 檔案是否存在。 如果存在，在命令提示字元中執行此命令，以顯示 Azure Enhanced Monitor 所收集的資訊︰

   ```
   cat /var/lib/AzureEnhancedMonitor/PerfCounters
   ```

   輸出如下所示：
   ```
   ...
   2;cpu;Current Hw Frequency;;0;2194.659;MHz;60;1444036656;saplnxmon;
   2;cpu;Max Hw Frequency;;0;2194.659;MHz;0;1444036656;saplnxmon;
   ...
   ```

## <a name="564adb4f-5c95-4041-9616-6635e83a810b"></a>針對端對端監視進行檢查及疑難排解
部署 Azure VM 並設定相關 Azure 監視基礎結構之後，請檢查 Azure Enhanced Monitoring Extension 的所有元件是否如預期般運作。

如 [Azure Enhanced Monitoring Extension for SAP 整備檢查][deployment-guide-5.1]所述，執行 Azure Enhanced Monitoring Extension for SAP 整備檢查。 如果所有整備檢查結果均為正向，而且所有相關效能計數器都呈現 OK 狀態，表示已成功設定 Azure 監視。 您可以繼續安裝 SAP Host Agent (如 [SAP 資源][deployment-guide-2.2]中的 SAP Note 所述)。 如果整備檢查指出遺失計數器，請執行 Azure 監視基礎結構的健康狀態檢查 (如 [Azure 監視基礎結構組態的健康狀態檢查][deployment-guide-5.2]所述)。 如需其他疑難排解選項，請參閱[針對適用於 SAP 的 Azure 監視進行疑難排解][deployment-guide-5.3]。

### <a name="bb61ce92-8c5c-461f-8c53-39f5e5ed91f2"></a>Azure Enhanced Monitoring Extension for SAP 整備檢查
這項檢查可確保基礎 Azure 監視基礎結構提供 SAP 應用程式內所顯示的所有效能計量。

#### <a name="run-the-readiness-check-on-a-windows-vm"></a>在 Windows VM 上執行整備檢查

1.  登入 Azure 虛擬機器 (不一定要使用管理帳戶)。
1.  開啟命令提示字元視窗。
1.  在命令提示字元，將目錄切換到 Azure Enhanced Monitoring Extension for SAP 的安裝資料夾：C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\&lt;版本>\\drop

  監視擴充功能路徑中的「版本」可能會有所不同。 如果您在安裝資料夾中看到多個監視擴充功能版本的資料夾，請檢查 AzureEnhancedMonitoring Windows 服務的組態，然後切換至「可執行檔的路徑」所表示的資料夾。

  ![執行 Azure Enhanced Monitoring Extension for SAP 之服務的屬性][deployment-guide-figure-1000]

1.  在命令視窗中，執行不含任何參數的 **azperflib.exe**。

  > [!NOTE]
  > Azperflib.exe 會以迴圈形式執行，並每隔 60 秒更新收集到的計數器。 若要完成迴圈，請關閉命令提示字元視窗。
  >
  >

如果未安裝 Azure Enhanced Monitoring Extension 或 AzureEnhancedMonitoring 服務未執行，即表示尚未正確設定擴充功能。 如需如何部署擴充功能的詳細資訊，請參閱[針對適用於 SAP 的 Azure 監視基礎結構進行疑難排解][deployment-guide-5.3]。

> [!NOTE]
> Azperflib.exe 為不能用於個人用途的元件。 它是為 SAP 主機代理程式提供與 VM 相關之 Azure 監視資料的元件。
> 

##### <a name="check-the-output-of-azperflibexe"></a>檢查 azperflib.exe 的輸出
Azperflib.exe 輸出會顯示適用於 SAP 的所有已填入 Azure 效能計數器。 在所收集計數器的清單底部，摘要和健康狀態指標會顯示 Azure 監視的狀態。

![執行 azperflib.exe 的健康狀態檢查輸出，其表示沒有任何問題][deployment-guide-figure-1100]
<a name="figure-11"></a>

請檢查針對 **Counters total** 傳回的結果 (其回報是空的)，以及針對 **Health status** 傳回的結果 (如上圖所示)。

解譯結果值，如下所示︰

| Azperflib.exe 結果值 | Azure 監視健康狀態 |
| --- | --- |
| **API 呼叫 - 無法使用** | 無法使用的計數器有可能不適用於虛擬機器組態，或發生錯誤。 請參閱 **Health status**。 |
| **Counters total - empty** |下列兩個 Azure 儲存體計數器可以空白： <ul><li>儲存體讀取 Op 延遲伺服器毫秒</li><li>儲存體讀取 Op 延遲 E2E 毫秒</li></ul>所有其他計數器都必須具有值。 |
| **Health status** |只有在傳回狀態顯示 [OK] 時，才算正常。 |
| **診斷** |健康狀態的相關詳細資訊。 |

如果 [Health status] 值不是 [OK]，請依照 [Azure 監視基礎結構組態的健康狀態檢查][deployment-guide-5.2]中的指示執行。

#### <a name="run-the-readiness-check-on-a-linux-vm"></a>在 Linux VM 上執行整備檢查

1.  使用 SSH 來連線到 Azure 虛擬機器。

1.  檢查 Azure Enhanced Monitoring Extension 的輸出。

  a.  執行 `more /var/lib/AzureEnhancedMonitor/PerfCounters`

   **預期的結果**：傳回效能計數器的清單。 此檔案不得是空的。

 b. 執行 `cat /var/lib/AzureEnhancedMonitor/PerfCounters | grep Error`

   **預期的結果**：傳回**無**錯誤的一行，例如 **3;config;Error;;0;0;none;0;1456416792;tst-servercs;**

  c. 執行 `more /var/lib/AzureEnhancedMonitor/LatestErrorRecord`

    **預期的結果**：傳回為空白或不存在。

如果上述檢查不成功，請執行下列額外的檢查︰

1.  確定已安裝並啟用 waagent。

  a.  執行 `sudo ls -al /var/lib/waagent/`

      **預期的結果**：列出 waagent 目錄的內容。

  b.  執行 `ps -ax | grep waagent`

   **預期的結果**：顯示類似下列一個項目：`python /usr/sbin/waagent -daemon`

1.   確定已安裝並啟用 Azure Enhanced Monitoring Extension。

  a.  執行 `sudo sh -c 'ls -al /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-*/'`

    **預期的結果**：列出 Azure Enhanced Monitoring Extension 目錄的內容。

  b. 執行 `ps -ax | grep AzureEnhanced`

     **預期的結果**：顯示類似下列一個項目：`python /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-2.0.0.2/handler.py daemon`

1. 安裝 SAP Host Agent (如 SAP Note [1031096] 所述) 並檢查 `saposcol` 的輸出。

  a.  執行 `/usr/sap/hostctrl/exe/saposcol -d`

  b.  執行 `dump ccm`

  c.  檢查 **Virtualization_Configuration\Enhanced Monitoring Access** 計量是否為 **true**。

如果您已安裝 SAP NetWeaver ABAP 應用程式伺服器，請開啟交易 ST06，並檢查是否已啟用增強監視。

如果上述任何檢查失敗，而且如需如何重新部署擴充功能的詳細資訊，請參閱[針對適用於 SAP 的 Azure 監視基礎結構進行疑難排解][deployment-guide-5.3]。

### <a name="e2d592ff-b4ea-4a53-a91a-e5521edb6cd1"></a>Azure 監視基礎結構組態的健康狀態檢查
如果未正確提供部分監視資料 (如上面 [Azure Enhanced Monitoring for SAP 整備檢查][deployment-guide-5.1]所述測試所指出)，請執行`Test-AzureRmVMAEMExtension` Cmdlet，以檢查適用於 SAP 的 Azure 監視基礎結構和監視」擴充功能是否設定正確。

1.  確定您已安裝最新版的 Azure PowerShell Cmdlet (如[部署 Azure PowerShell Cmdlet][deployment-guide-4.1]所述)。
1.  執行下列 PowerShell Cmdlet。 如需可用環境的清單，請執行 `Get-AzureRmEnvironment` Cmdlet。 若要使用公用 Azure，請選取 **AzureCloud** 環境。 若為中國的 Azure，請選取 **AzureChinaCloud**。
  ```powershell
  $env = Get-AzureRmEnvironment -Name <name of the environment>
  Connect-AzureRmAccount -Environment $env
  Set-AzureRmContext -SubscriptionName <subscription name>
  Test-AzureRmVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
  ```

1.  輸入您的帳戶資料並識別 Azure 虛擬機器。

  ![SAP 特定 Azure Cmdlet Test-VMConfigForSAP_GUI 的輸入頁面][deployment-guide-figure-1200]

1. 指令碼會測試您所選虛擬機器的組態。

  ![適用於 SAP 之 Azure 監視基礎結構的成功測試輸出][deployment-guide-figure-1300]

確定每個健康狀態檢查的結果都 [OK]。 如果部分檢查未顯示 [OK]，請執行更新 Cmdlet (如[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 所述)。 請等待 15 分鐘，然後重複 [Azure Enhanced Monitoring for SAP 整備檢查][deployment-guide-5.1]和 [Azure 監視基礎結構組態的健康狀態檢查][deployment-guide-5.2]所述的檢查。 如果檢查仍然指出部分或所有計數器有問題，請參閱[針對適用於 SAP 的 Azure 監視基礎結構進行疑難排解][deployment-guide-5.3]。

> [!Note]
> 在使用受控標準 Azure 磁碟的情況下，您可能會遇到一些警告。 測試將不會傳回「確定」，而會顯示警告。 這對於該磁碟類型而言為正常且預期中的情況。 請參閱[針對適用於 SAP 的 Azure 監視基礎結構進行疑難排解][deployment-guide-5.3]
> 

### <a name="fe25a7da-4e4e-4388-8907-8abc2d33cfd8"></a>針對適用於 SAP 的 Azure 監視基礎結構進行疑難排解

#### <a name="windowslogowindows-azure-performance-counters-do-not-show-up-at-all"></a>![Windows][Logo_Windows] 完全未顯示 Azure 效能計數器
AzureEnhancedMonitoring Windows 服務會收集 Azure 中的效能計量。 如果尚未正確安裝服務，或未在 VM 中執行服務，則不會收集任何效能計量。

##### <a name="the-installation-directory-of-the-azure-enhanced-monitoring-extension-is-empty"></a>Azure Enhanced Monitoring Extension 的安裝目錄是空的

###### <a name="issue"></a>問題
安裝目錄 \\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\&lt;版本>\\drop 是空的。

###### <a name="solution"></a>解決方法
未安裝此擴充功能。 判斷這是否為 Proxy 問題 (如先前所述)。 您可能需要重新啟動電腦，或重新執行 `Set-AzureRmVMAEMExtension` 組態指令碼。

##### <a name="service-for-azure-enhanced-monitoring-does-not-exist"></a>Azure Enhanced Monitoring 的服務不存在

###### <a name="issue"></a>問題
AzureEnhancedMonitoring Windows 服務不存在。

Azperflib.exe 輸出會擲回錯誤︰

![執行 azperflib.exe 即表示 Azure Enhanced Monitoring Extension for SAP 的服務未執行][deployment-guide-figure-1400]
<a name="figure-14"></a>

###### <a name="solution"></a>解決方法
如果服務不存在，即表示尚未正確安裝 Azure Enhanced Monitoring Extension for SAP。 請使用 [Azure 中適用於 SAP 的 VM 部署案例][deployment-guide-3]中針對您的部署案例所述的步驟，重新部署此擴充功能。

部署此擴充功能之後，請在一小時後再度檢查 Azure VM 中是否提供 Azure 效能計數器。

##### <a name="service-for-azure-enhanced-monitoring-exists-but-fails-to-start"></a>Azure Enhanced Monitoring 的服務存在，但無法啟動

###### <a name="issue"></a>問題
AzureEnhancedMonitoring Windows 服務存在並已啟用，但無法啟動。 如需詳細資訊，請檢查應用程式事件記錄檔。

###### <a name="solution"></a>解決方法
組態不正確。 如[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 所述，重新啟動 VM 的監視擴充功能。

#### <a name="windowslogowindows-some-azure-performance-counters-are-missing"></a>![Windows][Logo_Windows] 遺失部分 Azure 效能計數器
AzureEnhancedMonitoring Windows 服務會收集 Azure 中的效能計量。 此服務會從數個來源取得資料。 有些組態資料是在本機收集，而有些效能計量是從 Azure 診斷讀取而來。 您會從登入儲存體訂用帳戶的層級使用儲存體計數器。

如果使用 SAP Note [1999351] 進行疑難排解並未解決問題，請重新執行 `Set-AzureRmVMAEMExtension` 組態指令碼。 因為儲存體分析或診斷計數器在啟用後可能未立即建立，所以您可能必須等待一個小時的時間。 如果問題仍然存在，請在元件 BC-OP-NT-AZR (適用於 Windows) 或 BC-OP-LNX-AZR (適用於 Linux 虛擬機器) 上開啟 SAP 客戶支援訊息。

#### <a name="linuxlogolinux-azure-performance-counters-do-not-show-up-at-all"></a>![Linux][Logo_Linux] 完全未顯示 Azure 效能計數器
Daemon 會收集在 Azure 中的效能計量。 如果未執行 Daemon，則不會收集任何效能計量。

##### <a name="the-installation-directory-of-the-azure-enhanced-monitoring-extension-is-empty"></a>Azure Enhanced Monitoring Extension 的安裝目錄是空白

###### <a name="issue"></a>問題
\\var\\lib\\waagent\\ 目錄未包含 Azure Enhanced Monitoring Extension 的子目錄。

###### <a name="solution"></a>解決方法
未安裝此擴充功能。 判斷這是否為 Proxy 問題 (如先前所述)。 您可能需要重新啟動電腦及/或重新執行 `Set-AzureRmVMAEMExtension` 組態指令碼。

##### <a name="the-execution-of-set-azurermvmaemextension-and-test-azurermvmaemextension-show-warning-messages-stating-that-standard-managed-disks-are-not-supported"></a>執行 Set-AzureRmVMAEMExtension 與 Test-AzureRmVMAEMExtension 會顯示警告訊息，表示不支援標準受控磁碟

###### <a name="issue"></a>問題
執行 Set-AzureRmVMAEMExtension 或 Test-AzureRmVMAEMExtension 時，系統會顯示像這樣的訊息：

<pre><code>
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
</code></pre>

如先前所述，執行 azperfli.exe 可能會獲得表示健康情況不良狀態的結果。 

###### <a name="solution"></a>解決方法
導致這些訊息的原因，是因為標準受控磁碟沒有提供監視擴充功能用來檢查標準 Azure 儲存體帳戶之統計資料的 API。 使用者並不需要在意此問題。 針對標準磁碟儲存體帳戶導入監視功能的原因，是為了針對經常發生的 I/O 進行節流。 受控磁碟將會透過限制儲存體帳戶中的磁碟數量來避免此類節流。 因此，有沒有該類型的監視資料並不重要。


#### <a name="linuxlogolinux-some-azure-performance-counters-are-missing"></a>![Linux][Logo_Linux] 遺失部分 Azure 效能計數器
Azure 中的效能計量是由 Daemon 收集，而 Daemon 會從數個來源取得資料。 有些組態資料是在本機收集，而有些效能計量是從 Azure 診斷讀取而來。 儲存體計數器來自您儲存體訂用帳戶中的記錄檔。

如需完整且最新的已知問題清單，請參閱 SAP Note [1999351]，其中包含適用於 Enhanced Azure Monitoring for SAP 的其他疑難排解資訊。

如果使用 SAP Note [1999351] 進行排解疑難並未解決問題，請重新執行 `Set-AzureRmVMAEMExtension` 組態指令碼 (如[設定 Azure Enhanced Monitoring Extension for SAP][deployment-guide-4.5] 所述)。 因為儲存體分析或診斷計數器在啟用後可能未立即建立，所以您可能必須等待一個小時的時間。 如果問題仍然存在，請在元件 BC-OP-NT-AZR (適用於 Windows) 或 BC-OP-LNX-AZR (適用於 Linux 虛擬機器) 上開啟 SAP 客戶支援訊息。
