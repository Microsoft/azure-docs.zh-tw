---
title: 適用於 SAP 工作負載的 SQL Server Azure 虛擬機器 DBMS 部署 | Microsoft Docs
description: 適用於 SAP 工作負載的 SQL Server Azure 虛擬機器 DBMS 部署
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/26/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 3cefecdf0f87483a1fb544d1eb4e3e514e388259
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406908"
---
# <a name="sql-server-azure-virtual-machines-dbms-deployment-for-sap-netweaver"></a>適用於 SAP NetWeaver 的 SQL Server Azure 虛擬機器 DBMS 部署

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
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
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[azure-cli]:../../../cli-install-nodejs.md
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azureps-cmdlets-docs
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide_general.md 
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f 
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b 
[dbms-guide-5.6]:dbms_guide_sqlserver.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d 
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c 
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8
[dbms-guide-managed-disks]:dbms-guide.md#f42c6cb5-d563-484d-9667-b07ae51bce29

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]:deployment-guide.md 
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab 
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e 
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e 
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc 
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d 
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f 
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca 
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 

[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b 
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d 
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
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b

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

[msdn-set-azurermvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

[planning-guide]:planning-guide.md 
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff 
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058 
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77 
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f 
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803 
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a 
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358 
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560 
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665 
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 
[planning-guide-3.3.2]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53 
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c 
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef 
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7 
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79 
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a 
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1 
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d 
[planning-guide-7.1]:planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30 
[planning-guide-7]:planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14 
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3 
[planning-guide-azure-premium-storage]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92 

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
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd 
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f 

[powershell-install-configure]:https://docs.microsoft.com/powershell/azure/install-azurerm-ps
[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/resource-group-overview.md
[resource-groups-networking]:../../../networking/networking-overview.md
[sap-pam]:https://support.sap.com/pam 
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
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
[virtual-machines-azurerm-versus-azuresm]:../../../resource-manager-deployment-model.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md 
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md 
[virtual-machines-linux-agent-user-guide]:../../linux/agent-user-guide.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../linux/agent-user-guide.md#command-line-options
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
[virtual-machines-manage-availability-linux]:../../linux/manage-availability.md
[virtual-machines-manage-availability-windows]:../../windows/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:./../../windows/sql/virtual-machines-windows-sql-high-availability-dr.md
[virtual-machines-sql-server-infrastructure-services]:./../../windows/sql/virtual-machines-windows-sql-server-iaas-overview.md
[virtual-machines-sql-server-performance-best-practices]:./../../windows/sql/virtual-machines-windows-sql-performance.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/eresources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
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




本文介紹當我們在 Azure IaaS 部署 SAP 工作負載適用的 SQL Server 時，幾個要考量到的地方。 在閱讀本文件之前，您應該先參閱[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件，以及 [Azure 上的 SAP 工作負載文件](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)中的其他指南。 



> [!IMPORTANT]
> 本文件討論對象是 SQL Server 上的 Windows 版本。 SAP 不支援 Linux 版本的 SQL Server 安裝任何的 SAP 軟體。 木文件不討論 Microsoft Azure SQL Database，此為 Microsoft Azure 平台的「平台即服務」供應項目。 本白皮書中討論的是如何執行 SQL Server 產品 (已知適用於 Azure 虛擬機器中的內部部署)，以及如何運用 Azure 的「基礎架構即為服務」功能。 這兩個產品之間所提供的資料庫性能與功能並不相同，不應混用彼此。 另請參閱：<https://azure.microsoft.com/services/sql-database/>
> 
>

一般情況下，您應該考慮使用最新的 SQL Server 版本，以便在 Azure IaaS 中執行 SAP 工作負載。 最新的 SQL Server 版本能與部分的 Azure 服務和功能進行更好的整合。 或者功能上已有改進，能將 Azure IaaS 基礎結構的各項操作最佳化。

建議您先檢閱[這份][virtual-machines-sql-server-infrastructure-services]文件之後再繼續。

在下列章節中，會彙總並提及上述連結下方之文件的某些部分。 也會提到 SAP 專屬的詳細資料，並更深入說明一些概念。 不過，強烈建議您完整閱讀該文件，然後再閱讀 SQL Server 專屬的文件。

繼續之前，您應該先了解一些 IaaS 中 SQL Server 專屬的資訊：

* **SQL 版本支援**︰針對 SAP 客戶，Microsoft Azure 虛擬機器支援 SQL Server 2008 R2 和更新版本。 不支援舊版。 如需更多詳細資料，請檢閱這份通用的 [支援聲明](https://support.microsoft.com/kb/956893) 。 總體而言，Microsoft 也支援 SQL Server 2008。 不過，由於適用於 SAP 的重大功能是透過 SQL Server 2008 R2 所引進，因此 SQL Server 2008 R2 是適用於 SAP 的最低版本。 一般情況下，您應該考慮使用最新的 SQL Server 版本，以便在 Azure IaaS 中執行 SAP 工作負載。 最新的 SQL Server 版本能與部分的 Azure 服務和功能進行更好的整合。 或者功能上已有改進，能將 Azure IaaS 基礎結構的各項操作最佳化。 因此，本文僅適用於 SQL Server 2016 和 SQL Server 2017。
* **SQL 效能**：相較於其他公用雲端虛擬化供應項目，我們確信 Microsoft Azure 裝載的虛擬機器會執行得很順利，但產生的個別結果可能不同。 請瀏覽本文章：[Azure 虛擬機器中的 SQL Server 效能最佳做法](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance) \(機器翻譯\)。
* **使用來自 Azure Marketplace 的映像**︰部署新 Microsoft Azure VM 的最快方式就是使用來自 Azure Marketplace 的映像。 Azure Marketplace 中的映像包含最新的 SQL Server 版本。 已經安裝 SQL Server 的映像不能立即用於 SAP NetWeaver 應用程式。 原因是預設的 SQL Server 定序是安裝於這些映像內，而不是 SAP NetWeaver 系統所需的定序。 若要使用這類映像，請參閱[使用來自 Microsoft Azure Marketplace 的 SQL Server 映像][dbms-guide-5.6]一章中記載的步驟。 


## <a name="recommendations-on-vmvhd-structure-for-sap-related-sql-server-deployments"></a>適用於 SAP 相關之 SQL Server 部署的 VM/VHD 結構建議
根據一般的描述，SQL Server 可執行檔應該位於或安裝於 VM OS 磁碟的系統磁碟機 (磁碟機 C:\))。  一般而言，依照 SAP NetWeaver 工作負載，大部分的 SQL Server 系統資料庫都不會在高等級中使用。 因此，SQL Server 的系統資料庫 (master、msdb 和 model) 也可以保留於 C:\ 磁碟機。 有一個例外狀況應該是 tempdb，在某些 SAP 工作負載中，可能需要較高的資料量或 I/O 作業量。 I/O 工作負載，不應該套用至 OS VHD。 針對這類系統，應該執行下列步驟︰


* 排除 A 系列 VM 之外，所有 SAP 認證的 VM 類型 (請參閱 SAP 附註 [1928533])、tempdb 資料和記錄檔都可以儲存至非持續性 D:\ 磁碟機上。 
* 不過，建議使用多個 tempdb 資料檔。 請注意，D:\ 磁碟機磁碟區會根據 VM 類型而不同。 如需不同 VM 的 D:\ 磁碟機正確大小，請參閱 [Azure 中 Windows 虛擬機器的大小](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)文章 \(機器翻譯\)。

這些組態讓 tempdb 所耗用的空間比系統磁碟機能夠提供的還多。 非持續性 D:\ 磁碟機也會提供更好的 I/O 延遲和輸送量 (但 A 系列 VM 除外)。 若要判斷正確的 tempdb 大小，您可以在現有的系統上檢查 tempdb 大小。 

>[!NOTE]
> 如果您將 tempdb 資料檔案和記錄檔儲存至您在 D:\ 磁碟機上建立的資料夾，您必須確定 VM 重新啟動後，這個資料夾依然存在。 因為 VM 重新開機之後，D:\ 磁碟機會重新初始化，所有的檔案和目錄結構都會被會抹除。至於什麼情況下必須在 SQL Server 服務啟動前，先在 D:\ 磁碟機上重新建立目錄結構，請參閱[這篇文章](http://www.sqlserver.co.uk/index.php/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/)。

執行含有 SAP 資料庫的 SQL Server 且 tempdb 資料和 tempdb 記錄檔放置於 D:\ 磁碟機的 VM 組態應該像這樣︰

![SQL Server 簡單的 VM 磁碟設定圖表](./media/dbms_sqlserver_deployment_guide/Simple_disk_structure.PNG)

上圖顯示一個簡單的案例。 如同[針對 SAP 工作負載而部署 Azure 虛擬機器 DBMS 時的考量](dbms_guide_general.md)所述，「進階儲存體」磁碟的數量和大小，取決於不同的因素。 不過總體而言，我們建議：

- 您可以使用儲存體空間來形成一個或少數幾個包含 SQL Server 資料檔的磁碟區。 這個組態背後的原因是現實生活中，有數個 SAP 資料庫包含不同大小的資料庫檔案，而這檔案又包含不同的 I/O 工作負載。
- 使用儲存體空間來提供足夠的 IOPS 並用於 SQL Server 交易記錄檔。 潛在的 IOPS 工作負載通常是交易記錄磁碟區的大小調整依據，不是 SQL Server 交易磁碟區的潛在磁碟區。
- 只要效能夠好，tempdb 就使用 D:\磁碟機。 如果目前在磁碟機 D:\ 的 tmepdb，會讓整體工作負載的效能受到限制，那麼如同[這篇文章](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance)的建議，您可能需要考慮將 tempdb 移至個別的「進階儲存體」磁碟。


### <a name="special-for-m-series-vms"></a>M 系列 VM 的特殊情況
和 Azure 進階儲存體 相比之下，使用 Azure Write Accelerator 時，會有諸多因素造成寫入交易日誌的延遲時間被縮短。 因此，您應該為組成 SQL Server 交易記錄檔之磁碟區的 VHD，部署 Azure Write Accelerator。 如需詳細資訊，請參閱[寫入加速器](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-enable-write-accelerator)文件。
  

### <a name="formatting-the-disks"></a>將磁碟格式化
針對 SQL Server，適用於含有 SQL Server 資料和記錄檔之磁碟的 NTFS 區塊大小應該是 64 KB。 不需要將 D:\ 磁碟機格式化。 此磁碟機已預先格式化。

若要確定還原或建立資料庫不會藉由清空檔案的內容來初始化資料檔，您應該確定 SQL Server 服務執行所在的使用者內容具有特定的權限。 通常，Windows 系統管理員群組中的使用者會擁有這些權限。 在非 Windows 系統管理員使用者的使用者內容中執行 SQL Server 服務時，您需要為該使用者指派**執行磁碟區維護工作**的使用者權限。  詳細資料請參閱這篇 Microsoft 知識庫文章︰<https://support.microsoft.com/kb/2574695>

### <a name="impact-of-database-compression"></a>資料庫壓縮的影響
在 I/O 頻寬可能變成限制因素的組態中，減少 IOPS 的每個量值可能都有助於延展您可以在類似 Azure 的 IaaS 案例中執行的工作負載。 因此，如果尚未這麼做，SAP 和 Microsoft 建議您套用 SQL Server 頁面壓縮，然後再將現有的 SAP 資料庫上傳至 Azure。

在上傳至 Azure 之前執行資料庫壓縮的建議是出自下列兩項因素︰

* 降低要上傳的資料量。
* 假設您可以使用功能更強大的硬體，其在內部部署中使用更多 CPU 或更高 I/O 頻寬或更少 I/O 延遲，則執行壓縮的持續時間較短。
* 較小的資料庫大小可能會使磁碟配置的成本降低

資料庫壓縮也可以在 Azure 虛擬機器中運作，如同它在內部部署中的運作方式。 如需進一步了解如何壓縮現有的 SAP NetWeaver SQL Server 資料庫，請參閱[改善的 SAP 壓縮工具 MSSCOMPRESS](https://blogs.msdn.microsoft.com/saponsqlserver/2016/11/25/improved-sap-compression-tool-msscompress/) \(英文\) 文章。 

## <a name="sql-server-2014-and-more-recent---storing-database-files-directly-on-azure-blob-storage"></a>SQL Server 2014 以及最新的版本 - 將資料庫檔案直接儲存於 Azure Blog 儲存體上
SQL Server 2014 及更新版本可以直接在 Azure Blob Store上儲存資料檔案，無需檔案外面使用 VHD 的「包裝函式」。 特別是在使用標準 Azure 儲存體或較小的 VM 類型時，這種部署類型可以讓您在某些情況下克服某些限制，也就是指當數量有限的磁碟掛接至某些較小的 VM 類型時，不得不做出的 IOPS 限制。 不過，這種部署方式適用於 SQL Server 的使用者資料庫，而不適用於系統資料庫。 這也適用於 SQL server 的資料和記錄檔。 如果想您要使用此方式來部署 SAP SQL Server 資料庫，而不是將它「包裝」到 VHD，請記住下列資訊︰

* 所使用之儲存體帳戶所在的 Azure 區域必須與用來部署 SQL Server 執行所在之 VM 的相同。
* 稍早列出有關將 VHD 分散到不同 Azure 儲存體帳戶的考量，也適用於這種部署方法。 表示對 Azure 儲存體帳戶限制的 I/O 作業計數。
* 不要針對 VM 的儲存體 I/O 配額來結算，那些針對儲存體 blob (代表 SQL Server 資料和記錄檔) 所產生的流量，都將計入特定 VM 類型的 VM 網路頻寬。 針對特定的 VM 類型的網路和儲存頻寬，請參閱 [Azure 中 Windows 虛擬機器的大小](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)一文。
* 由於透過網路配額推送檔案 I/O 的關係，您大部分的儲存體配額都已耗損，因此 VM 的整體頻寬僅部分使用。
* Azure 進階儲存體針對不同磁碟大小所設定的 IOPS 和 I/O 輸送量效能目標，不會再套用。 即便您建立的 blob 是位於「Azure 進階儲存體」也是如此。 [VM 高效能進階儲存體與受控磁碟](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage#scalability-and-performance-targets) \(機器翻譯\) 文章有相關目標的記載。 由於將 SQL Server 資料檔案和記錄檔直接放在 Azure 進階儲存體所儲存的 blob 上，因此相較於 Azure 進階儲存體上的 VHD，效能特性會不一樣。
* 將 SQL Server 資料檔案直接放置在 Azure Blob 上時，適用於 Azure 進階儲存體磁碟的主機型快取將無法使用。
* 在 M 系列 VM 上，無法使用 Azure Write Accelerator 來支援 SQL Server 交易記錄檔的子毫秒寫入。 

如需這項功能的詳細資料，請參閱[Microsoft Azure 中的 SQL Server 資料檔案](https://docs.microsoft.com/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure?view=sql-server-2017) \(機器翻譯\)

針對生產環境系統提出的建議是：避免這種設定，寧願選擇將 SQL Server 資料檔和記錄檔放在 Azure 進階儲存體 VHD，也不要直接放在 Azure blob 上。


## <a name="sql-server-2014-buffer-pool-extension"></a>SQL Server 2014 緩衝集區延伸
SQL Server 2014 引進的新功能，稱為 [緩衝集區擴充](https://docs.microsoft.com/sql/database-engine/configure-windows/buffer-pool-extension?view=sql-server-2017) \(機器翻譯\)。 此功能會擴充 SQL Server 的緩衝集區，使用第二層快取將其保留於記憶體中，此快取是透過伺服器或 VM 的本機 SSD 來支援。 緩衝集區擴充模組可以將較大的資料工作集放入「記憶體內」。 相較於存取 Azure 標準儲存體，基於許多因素，存取儲存於 Azure VM 之本機 SSD 上的緩衝集區延伸的速度更快。 比較「緩衝集區擴充」和「Azure 進階儲存體讀取快取」後，和給出的 SQL Server 資料檔建議一樣，「緩衝集區擴充」沒有顯著的優勢。 原因在於這兩個快取 (SQL Server 緩衝集區延伸和進階儲存體讀取快取) 都是使用 Azure 計算節點的本機磁碟。

SQL Server 緩衝集區擴充搭配 SAP 工作負載時所得到的體驗有好有壞，而且仍然無法明確建議是否所有的情況下皆適用。 理想的情況是：SAP 應用程式所需的工作集，剛好能放入主記憶體中。 利用 Azure 再加上提供的 VM 又有高達 4 TB 的記憶體，這樣應該能將工作集放入記憶體中了。 因此緩衝集區擴充只能在某些罕見的情況下使用，而且不是主流案例。  

## <a name="backuprecovery-considerations-for-sql-server"></a>SQL Server 的備份/復原考量
將 SQL Server 部署至 Azure 時，必須檢閱您的備份方法。 即使系統不是生產系統，還是必須定期備份 SQL Server 所裝載的 SAP 資料庫。 由於 Azure 儲存體會保留三個映像，因此在補償儲存體損毀方面，備份現在已變得較不重要。 優先維護適當備份和復原方案的原因是，您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原回到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 

若要尋求 Azure 中各種不同的 SQL Server 備份可能性，請參閱[Azure 虛擬機器中的 SQL Server 備份和還原](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery) \(機器翻譯\) 文章。 本文章描述幾個不同的可能性。

### <a name="manual-backups"></a>手動備份
您有幾種可能性來執行「手動」備份：

1. 在直接連接的 Azure 磁碟上執行傳統的 SQL Server 備份。 這個方法的優點是您可以迅速擁有備份來重新整理系統，以及建立新系統做為現有 SAP 系統的複本
2.  SQL Server 2012 CU4 和更新版本可以將資料庫備份至 Azure 儲存體 URL。
3.  Azure Blob 儲存體中資料庫檔案的檔案快照集備份。 只有您的 SQL Server 資料檔和記錄檔都位於 Azure blob 儲存體，這個方法才管用。

第一種方法廣為人知，而且在內部部署環境的很案例中都曾用過。 不過，您要自己解決長期的備份位置。 因為您希望備份保留在本機外接 Azure 儲存體上的時間不要超過 30 天，因此您必須使用 Azure 備份服務或其他協力廠商備份/復原工具 (能管理備份的存取和保留)。 或者您可以使用 Windows 儲存體空間，在 Azure 中建立一個大型的檔案伺服器。

[SQL Server 備份至 URL](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-to-url?view=sql-server-2017) \(機器翻譯\) 文章會詳細描述第二種方法。 這個功能在不同的 SQL Server 版本中會出現變異。 因此，您應該參閱這個文件，檢查您的 SQL Server 究竟是哪一個版本。 請注意，這篇文章會列出很多限制。 您可能會針對以下各項來執行備份：

- 一個 Azure 頁面 blob，然後將備份大小限制為 1000 GB。 這同樣也會限制您可以達到的輸送量。
- 多個 (最多 64 個) Azure 區塊 blob，可以達到理論上的 12 TB 備份大小。 不過，測試客戶資料庫之後發現備份大小上限可能會小於理論上的限制。 在這種情況下，您要負責管理備份的保留和存取。


### <a name="automated-backup-for-sql-server"></a>適用於 SQL Server 2016+ 的自動備份
自動備份可為在 Windows Azure VM 中執行的 SQL Server Standard 與 Enterprise 版提供自動備份服務。 此服務是由 [SQL Server IaaS 代理程式延伸模組](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension)提供的，並會自動在 Azure 入口網站中安裝於 SQL Server Windows 虛擬機器映像上。 如果您部署自己的 OS 映像時，連帶安裝了 SQL Server，則必須另外再安裝 VM 延伸模組。 至於相關的步驟，請參閱這篇[文章](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension)。

如需進一步了解這個方法的功能，請參閱下列文章：

- SQL Server 2014：[SQL Server 2014 虛擬機器的自動備份 (Resource Manager)](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-automated-backup) \(機器翻譯\)
- SQL Server 2016/2017：[Azure 虛擬機器的自動備份 v2 (Resource Manager)](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-automated-backup-v2) \(機器翻譯\)

仔細看這份文件，您會發現 SQL Server 最新版本的功能已得到改善。 如需進一步了解 SQL Server 的自動備份功能，請參閱 [SQL Server Managed Backup to Microsoft Azure](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure?view=sql-server-2017) \(機器翻譯\) 這篇文章。 理論上的備份大小限制為 12 TB。  自動備份可以讓大小達到 12 TB，是一種好方法。 由於多個 blob 會以平行方式寫入，因此輸送量每秒會超過 100 MB。 
 

### <a name="azure-backup-for-sql-server-vms"></a>適用於 SQL Server VM 的 Azure 備份
這種新的 SQL Server 備份方法，自 2018 年 6 月起便成為 Azure 備份服務的公開預覽。 備份 SQL Server 的方法其他協力廠商工具相同，也就是透過 SQL Server VSS/VDI 介面，將 SQL Server 備份串流至目標位置。 這種情況下的目標位置是 Azure 復原服務保存庫。

這種備份方法多了很多的優點，例如集中備份設定、監視和管理，相關詳情請參閱[這裡](https://docs.microsoft.com/azure/backup/backup-azure-sql-database)。 


### <a name="third-party-backup-solutions"></a>第三方備份解決方案
有很多的 SAP 客戶，他們的 SAP 是在 Azure 上執行的，要讓他們重頭再來，改用全新的備份解決方案，這是不可能事。 如此一來，就必須使用現有的備份解決方案，並擴充至 Azure。 將現有的備份解決方案擴充至 Azure，大部分的主要供應商在這一邊通常做得不錯。 


## <a name="1b353e38-21b3-4310-aeb6-a77e7c8e81c8"></a>使用來自 Microsoft Azure Marketplace 的 SQL Server 映像
Microsoft 在 Azure Marketplace 中提供已經包含 SQL Server 版本的 VM。 至於需要 SQL Server 和 Windows 授權的 SAP 客戶，使用這些映像便可以利用已經安裝 SQL Server 來啟動 VM，這樣就不用取得授權了。 若要針對 SAP 使用這類映像，必須進行下列考量︰

* SQL Server 非評估版本的取得成本高於從 Azure Marketplace 部署的「僅限 Windows」VM。 請參閱下列文章以比較價格：<https://azure.microsoft.com/pricing/details/virtual-machines/windows/> 和 <https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-enterprise/>。 
* 您只能使用 SAP 支援的 SQL Server 版本。
* 安裝於 Azure Marketplace 中提供之 VM 上的 SQL Server 執行個體定序，並不是 SAP NetWeaver 要求執行之 SQL Server 執行個體的定序。 不過您可以利用下一節的指示來變更定序。

### <a name="changing-the-sql-server-collation-of-a-microsoft-windowssql-server-vm"></a>變更 Microsoft Windows/SQL Server VM 的 SQL Server 定序
因為 Azure Marketplace 中的 SQL Server 映像不是設定來使用 SAP NetWeaver 應用程式所需的定序，所以需要在部署之後立即變更。 至於 SQL Server，一旦部署 VM 且系統管理員能夠登入已部署的 VM 之後，就能夠使用下列步驟來完成定序的變更：

* 以系統管理員身分開啟 Windows 命令視窗。
* 將目錄變更為 C:\Program Files\Microsoft SQL Server\110\Setup Bootstrap\SQLServer2012。
* 執行命令︰Setup.exe /QUIET /ACTION=REBUILDDATABASE /INSTANCENAME=MSSQLSERVER /SQLSYSADMINACCOUNTS=`<local_admin_account_name`> /SQLCOLLATION=SQL_Latin1_General_Cp850_BIN2   
  * `<local_admin_account_name`> 是第一次透過資源庫部署 VM 時定義為系統管理員帳戶的帳戶。

此程序應該只需要幾分鐘的時間。 若要確定此步驟最終是否會有正確的結果，請執行下列步驟：

* 開啟 SQL Server Management Studio。
* 開啟查詢視窗。
* 在 SQL Server master 資料庫中，執行 sp_helpsort 命令。

所需的結果應該看起來如下：

    Latin1-General, binary code point comparison sort for Unicode Data, SQL Server Sort Order 40 on Code Page 850 for non-Unicode Data

如果這不是想要的結果，請停止部署 SAP，並調查為什麼安裝命令並未如預期般運作。 「不」  支援將 SAP NetWeaver 應用程式部署到 SQL Server 字碼頁與上述提及之字碼頁不同的 SQL Server 執行個體。

## <a name="sql-server-high-availability-for-sap-in-azure"></a>SQL Server 在 Azure 中適用於 SAP 的高可用性
您在適用於 SAP 的 Azure IaaS 部署中使用 SQL Server 時，是有幾種不同的可能性來部署高可用性的 DBMS 層。 如同[針對 SAP 工作負載而部署 Azure 虛擬機器 DBMS 時的考量](dbms_guide_general.md)所述，Azure 會針對單一 VM 以及 Azure 可用性集中部署的一對 VM，提供不同的運作時間 SLA。 我們假設您傾向為您的生產部署取得運作時間 SLA，以便在 Azure 可用性集中進行部署。 在這種情況下，您需要在這種可用性集中至少部署兩個 VM。 其中一個 VM 會執行運作中的 SQL Server 執行個體。 另一個 VM 會執行被動的執行個體

### <a name="sql-server-clustering-using-windows-scale-out-file-server"></a>SQL Server 叢集使用 Windows 向外延展檔案伺服器
在 Windows Server 2016 中，Microsoft 引進了[儲存空間直接存取](https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview)。 以儲存空間直接存取部署為基礎後，便能支援 SQL Server FCI 叢集。 如需詳細資訊，請參閱[在 Azure 虛擬機器上設定 SQL Server 容錯移轉叢集執行個體](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-create-failover-cluster) \(機器翻譯\)。 這個解決方案也需要 Azure 負載平衡器來處理叢集資源的虛擬 IP 位址。 SQL Server 資料庫檔案會儲存在「儲存空間」中。 因此，顯然您需要以 Azure 進階儲存體為基礎來建立 Windows 儲存空間。 因為這種解決方案受到支援的時間並不太長，所以尚未發現有 SAP 客戶在 SAP 生產案例中使用這個解決方案。  

### <a name="sql-server-log-shipping"></a>SQL Server 記錄傳送
高可用性 (HA) 的方法之一是 SQL Server 記錄傳送。 如果參與 HA 組態的 VM 具有運作中的名稱解析，就不會發生問題，而 Azure 中的設定與內部部署中完成的任何設定並無任何差別。 如需設定記錄傳送和記錄傳送原則的相關事宜。 請參閱[關於記錄傳送 (SQL Server)](https://docs.microsoft.com/sql/database-engine/log-shipping/about-log-shipping-sql-server?view=sql-server-2017) \(機器翻譯\) 文章。

SQL Server 記錄傳送功能根本很難用於 Azure 中來實現單一 Azure 區域中的高可用性。 不過在下列情況中，SAP 客戶已成功將記錄傳送與 Azure 搭配使用：

- 從一個 Azure 區域到另一個 Azure 區域的嚴重損壞修復案例
- 從內部部署到 Azure 區域的嚴重損壞修復設定
- 從內部部署至 Azure 的切換案例。 在這些情況下，記錄傳送是用來與進行中的生產系統內部部署，同步處理 Azure 中的新 DBMS 部署。 在切換時，會停止產生記錄並確定最後而且是最新的交易記錄備份已傳輸至 Azure DBMS 部署。 然後 Azure DBMS 部署會開啟來進行產生記錄。  



### <a name="database-mirroring"></a>資料庫鏡像
SAP 支援的「資料庫鏡像」(請參閱 SAP 附註 [965908]) 有賴於在 SAP 連接字串中定義容錯移轉夥伴。 針對跨單位案例，我們假設這兩個 VM 位於相同網域，且這兩個 SQL Server 執行個體執行所在的使用者內容也是網域使用者，而且在所涉及的這兩個 SQL Server 執行個體中具備足夠的權限。 因此，在 Azure 中設定資料庫鏡像，在一般內部部署設定/組態之間並無任何差別。

至於僅限雲端的部署，最簡單的方式是在 Azure 中設定另一個網域，以便讓這些 DBMS VM (最好專屬於 SAP VM) 位於一個網域。

如果網域不可行，您也可以針對資料庫鏡像端點使用憑證，相關說明請見︰<https://docs.microsoft.com/sql/database-engine/database-mirroring/use-certificates-for-a-database-mirroring-endpoint-transact-sql>

在 Azure 中設定資料庫鏡像的教學課程可在下列網址中找到︰<https://docs.microsoft.com/sql/database-engine/database-mirroring/database-mirroring-sql-server> 

### <a name="sql-server-always-on"></a>SQL Server Always On
因為 SAP 內部部署支援 Always On (請參閱 SAP 附註 [1772688])，因此與 Azure 中的 SAP 搭配使用時，同樣得到支援。 有一些關於 SQL Server 可用性群組接聽程式 (請勿與 Azure 可用性設定組混淆) 的特殊考量，因為 Azure 可能在內部部署中，所以目前不允許建立 AD/DNS 物件。 因此，需要有一些不同的安裝步驟來克服 Azure 的特定行為。

以下是使用可用性群組接聽程式的一些考量︰

* 使用可用性群組接聽程式，只能使用 Windows Server 2012 或更高版本作為 VM 的客體作業系統。 針對 Windows Server 2012，務必要套用此修補程式︰<https://support.microsoft.com/kb/2854082> 
* Windows Server 2008 R2 則沒有此修補程式，必須以和使用「資料庫鏡像」相同的方式使用 AlwaysOn，方法是在連接字串中指定容錯移轉夥伴 (透過 SAP default.pfl 參數 dbs/mss/server 來完成 - 請參閱 SAP 附註 [965908])。
* 使用可用性群組接聽程式時，資料庫 VM 需要連接到專用的負載平衡器。 為了避免 Azure 在這兩個 VM 意外關閉的情況下指派新的 IP 位址，您應該在 Always On 組態中為這些 VM 的網路介面指派靜態 IP 位址 (如需了解如何定義靜態 IP 位址，請參閱[這篇][virtual-networks-reserved-private-ip]文章)
* 建置叢集需要指派特定 IP 位址的 WSFC 叢集組態時需要特殊的步驟，因為具有其目前功能的 Azure 會為叢集名稱指派與叢集建立所在的節點相同的 IP 位址。 這表示必須執行手動步驟，為叢集指派不同的 IP 位址。
* 可用性群組接聽程式將建立於具備 TCP/IP 端點的 Azure 中，這些端點會指派給執行可用性群組之主要和次要複本的 VM。
* 可能需要使用 ACL 保護這些端點。

至於如何使用 Azure VM 中的 SQL Server 部署 Always On，請參閱以下文章：

- [Azure 虛擬機器上的 SQL Server Always On 可用性群組簡介](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview) \(機器翻譯\)。
- [在不同區域的 Azure 虛擬機器上設定 Always On 可用性群組](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-dr) \(機器翻譯\)。
- [在 Azure 中設定 Always On 可用性群組的負載平衡器](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener) \(機器翻譯\)。

>[!NOTE]
> 如果您要為可用性群組接聽程式的虛擬 IP 位址設定 Azure 負載平衡器，請務必設定 DirectServerReturn。 設定這個選項會減少 SAP 應用程式層與 DBMS 層之間的網路來回延遲。 

SQL Server Always On 是 Azure for SAP 工作負載部署中，最常使用的高可用性和嚴重損壞修復功能。 大部分的客戶使用 Always On 來實現單一 Azure 區域內的高可用性。 如果部署被限制為僅限兩個節點，您會有兩個連線選項：

- 使用可用性群組接聽程式。 使用可用性群組接聽程式時，您必須部署的 Azure 負載平衡器。 這通常是預設的部署方法。 SAP 應用程式應該設定為針對可用性群組接聽程式進行連線，而不是針對單一節點
- 使用 SQL Server 資料庫鏡像的連線參數。 在這種情況下，當設定 SAP 應用程式的連線時，這兩個節點名稱都要用到。 如需進一步了解如何設定這種的 SAP 端，請參閱 SAP 附註[編號 965908](https://launchpad.support.sap.com/#/notes/965908)。 使用這個選項時，您不需要設定可用性群組接聽程式。 而且，也不需要 Azure 負載平衡器來實現 SQL Server 高可用性。 如此一來，SAP 應用程式層與 DBMS 層之間的網路延遲較低，因為 SQL Server 執行個體的連入流量就不會經過 Azure 負載平衡器。 但是請回想一下，只有當將可用性群組限制只能跨越兩個執行個體的時候，這個選項才有效。 

很多客戶會將 SQL Server Always On 功能當作是 Azure 區域之間額外的嚴重損壞修復功能。 很多客戶也會利用這種功能，從次要複本執行備份。 

## <a name="sql-server-transparent-data-encryption"></a>SQL Server 透明資料加密
目前有一些客戶在 Azure 中部署 SAP SQL Server 資料庫時，使用 SQL Server [透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-2017)。 SAP 完全支援 SQL Server TDE 功能 (請參閱 SAP 附註[編號 1380493](https://launchpad.support.sap.com/#/notes/1380493))。 

### <a name="applying-sql-server-tde"></a>套用 SQL Server TDE
將另一個在內部部署上執行的 DBMS，往 Azure 中執行的 Windows/SQL Server，進行異質移轉時，您應該事先在 SQL Server 建立空白的目標資料庫。 下一個步驟中，您可以套用 SQL Server TDE 功能。 雖然您仍然是在執行內部部署的生產系統。 您之所以會按照這種順序來執行，原因是空資料庫的加密程序會消耗相當長的時間。 然後在停機階段，SAP 匯入程序會將資料匯入加密的資料庫。 一個是產生額外負荷來匯入至加密的資料庫，另一個是在停機階段的匯出階段之後進行資料庫的加密，前者的影響時間沒有後者來得長。 在嘗試將資料庫上執行的 SAP 工作負載套用至 TDE 時，產生負面的體驗。 因此，建議完成 TDE 的部署時，不需要特殊資料庫上的 SAP 工作負載。

將 SAP SQL Server 資料庫從內部部署移至 Azure 時，我們建議在您可以最快速套用加密的基礎結構進行測試。 請記住以下事實：

- 您無法定義當您將資料加密套用至資料庫時，會用到多少個執行緒。 執行緒數目主要是取決於 SQL Server 資料和記錄檔分佈的磁碟區數目。 也就是說不同的磁碟區越多 (磁碟機代號)，平行介入加密的緒行緒就越多。 這種組態與先前的磁碟組態建議衝突，也就是指為 Azure VM 中的 SQL Server 資料庫檔案，建立一或少數幾個儲存體空間。 具有少量磁碟區的組態，會導致執行加密的執行緒很少。 單一執行緒加密會讀取 64 KB 範圍，在加密之後並在交易記錄檔中寫入一筆記錄，告知已加密的範圍。 如此一來，交易記錄檔上的負載就比較適中。
- 在舊版的 SQL Server 版本中，當您加密自己的 SQL Server 資料庫時，備份壓縮的效率會變差。 當您的計劃是加密現場部署的 SQL Server 資料庫，然後將備份複製到 Azure，以便在 Azure 中還原資料庫時，這種行為會演變成一種問題。 SQL Server 備份壓縮通常可達到因子 4 的壓縮比。
- 從 SQL Server 2016 開始，SQL Server 引進了新的功能，可讓您以有效率的方式壓縮加密的資料庫。 如需詳細資訊，請參閱[這篇部落格文章](https://blogs.msdn.microsoft.com/sqlcat/2016/06/20/sqlsweet16-episode-1-backup-compression-for-tde-enabled-databases/) \(英文\)。
 
讓 TDE 加密套用無到少量 SAP 工作負載，您應該在自己的具體組態中進行測試，以便判斷最好是將 TDE 套用至您的內部部署 SAP 資料庫，還是在 Azure 中這麼做。 在 Azure 中，套用 TDE 之後，您當然有更多彈性決定是要過度佈建或縮減基礎結構。

### <a name="using-azure-key-vault"></a>編輯 Azure Key Vault
Azure 提供的 [Key Vault](https://azure.microsoft.com/services/key-vault/) 服務，可以儲存加密金鑰。 另一方面，SQL Server 提供連接器，可將 Azure Key Vault 當作是 TDE 憑證的存放區。

以下是 SQL Server TDE 詳細的 Azure Key Vault 用途：

- [使用 Azure Key Vault 進行可延伸金鑰管理 (SQL Server)](https://docs.microsoft.com/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server?view=sql-server-2017) \(機器翻譯\)。
- [使用 Azure Key Vault 進行 SQL Server TDE 可延伸金鑰管理 - 設定步驟](https://docs.microsoft.com/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault?view=sql-server-2017) \(機器翻譯\)。
- [SQL Server 連接器維護和疑難排解](https://docs.microsoft.com/sql/relational-databases/security/encryption/sql-server-connector-maintenance-troubleshooting?view=sql-server-2017) \(機器翻譯\)。
- [更多關於 SQL Server 透明資料加密的問題 – TDE + Azure Key Vault](https://blogs.msdn.microsoft.com/saponsqlserver/2017/04/04/more-questions-from-customers-about-sql-server-transparent-data-encryption-tde-azure-key-vault/) \(英文\)。


>[!IMPORTANT]
>使用 SQL Server TDE 時，尤其是 Azure 密鑰保管庫，建議使用 SQL Server 2014、SQL Server 2016 和 SQL Server 2017 的最新修補程序。 原因是根據客戶意見反應，會將程式碼最佳化並修正其中的錯誤。 相關範例，請參閱 [KBA 編號 4058175](https://support.microsoft.com/help/4058175/tde-enabled-backup-and-restore-slow-if-encryption-key-is-stored-in-ekm)。
>  

## <a name="9053f720-6f3b-4483-904d-15dc54141e30"></a>適用於 Azure 上 SAP 的一般 SQL Server 摘要
本指南中提供許多建議，而我們建議您在規劃 Azure 部署之前，多次閱讀本指南。 但是，一般而言，請務必遵循 Azure 具體建議前幾個一般 DBMS︰

1. 使用最新的 DBMS 版本 (例如 SQL Server 2017) ，其在 Azure 中最具優勢。 
2. 在 Azure 中仔細規劃您的 SAP 系統架構，以平衡資料檔案配置和 Azure 限制︰
   * 不需要有太多磁碟，但必須足以確保您可以連線到所需的 IOPS。
   * 如果您未使用受控磁碟，請記住每個「Azure 儲存體帳戶」也都有 IOPS 限制，而且「儲存體帳戶」在每個 Azure 訂用帳戶內是有限的 ([更多詳細資料][azure-subscription-service-limits])。 
   * 只有在您需要達到更高的輸送量時，才需在磁碟上劃分等量磁碟區。
3. 永遠不要在 D:\ 磁碟機上安裝軟體或放置任何需要永久保留的檔案，因為它不是永久性的，此磁碟機上的一切會在 Windows 重新開機時遺失。
4. 不要針對 Azure 標準儲存體使用磁碟快取。
5. 不要使用 Azure 異地複寫的 Azure 標準儲存體帳戶。  針對 DBMS 工作負載使用本機備援。
6. 使用 DBMS 廠商的 HA/DR 解決方案來複寫資料庫資料。
7. 一律使用名稱解析，不要依賴 IP 位址。
8. 使用 SQL Server TDE，套用最新的 SQL Server 修補程式。
9. 盡可能使用最高度的資料庫壓縮。 這是 SQL Server 的頁面壓縮。
10. 請務必謹慎使用來自 Azure Marketplace 的 SQL Server 映像。 如果您使用 SQL Server 的映像，就必須變更執行個體定序，才能在其上安裝任何 SAP NetWeaver 系統。
11. 依照[部署指南][deployment-guide]所述，安裝並設定適用於 Azure 的 SAP 主機監視功能。