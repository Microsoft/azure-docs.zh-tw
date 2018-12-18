---
title: 適用於 SAP 工作負載的 SAP ASE Azure 虛擬機器 DBMS 部署 | Microsoft Docs
description: 適用於 SAP 工作負載的 SAP ASE Azure 虛擬機器 DBMS 部署
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
ms.date: 07/1/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 243aecbe3627a6cc72de1bc98c301e8fa632ec36
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39075221"
---
# <a name="sap-ase-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>適用於 SAP 工作負載的 SAP ASE Azure 虛擬機器 DBMS 部署

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

[dbms-guide]:dbms-guide.md 
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f 
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b 
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 
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
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/resources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
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



本文介紹在 Azure IaaS 中部署 SAP ASE 時，幾個要考量到的地方。 在閱讀本文件之前，您應該先參閱[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件，以及 [Azure 上的 SAP 工作負載文件](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)中的其他指南。 

## <a name="specifics-to-sap-ase-on-windows"></a>Windows 上 SAP ASE 專屬的詳細資料
從 Microsoft Azure 開始，您即可將現有的 SAP ASE 應用程式遷移至 Azure 虛擬機器。 Azure 虛擬機器中的 SAP ASE 可讓您輕鬆地將這些應用程式移轉到 Microsoft Azure，藉以減少部署、管理和維護企業級應用程式的擁有權總成本。 透過 Azure 虛擬機器中的 SAP ASE，系統管理員和開發人員仍然可以使用可在內部部署使用的相同開發和管理工具。

Azure 虛擬機器的 SLA 可在此找到：<https://azure.microsoft.com/support/legal/sla/virtual-machines>

Microsoft Azure 提供許多不同的虛擬機器類型，可讓您執行最小的 SAP 系統和架構，以至具有數千個使用者的大型 SAP 系統和架構。 如需不同 SAP 認證的 VM SKU SAP 大小調整 SAPS 數目，請參閱 SAP 附註 [1928533]。

[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中關於 Azure 儲存體使用方式、部署 SAP VM 或 SAP 監視的陳述與建議也適用於部署 SAP ASE。

### <a name="sap-ase-version-support"></a>SAP ASE 版本支援
SAP 目前支援 SAP ASE 版本 16.0，可與 SAP 商務套件產品搭配使用。 不論是適用於 SAP ASE 伺服器的所有更新，還是要與「SAP 商務套件」產品搭配使用的 JDBC 和 ODBC 驅動程式，都只會透過 SAP Service Marketplace 來提供，網址是：<https://support.sap.com/swdc>。

不要直接從 Sybase 網站下載適用於 SAP ASE 伺服器或適用於 JDBC 和 ODBC 驅動程式的更新。 如需在內部部署和 Azure 虛擬機器中支援與 SAP 產品搭配使用的修補程式詳細資訊，請參閱下列 SAP 附註：

* [1590719]
* [1973241]

如需有關在 SAP ASE 上執行「SAP 商務套件」的一般資訊，請參閱 [SCN](https://www.sap.com/community/topic/ase.html)

### <a name="sap-ase-configuration-guidelines-for-sap-related-sap-ase-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 相關之 SAP ASE 的 SAP ASE 組態指導方針
#### <a name="structure-of-the-sap-ase-deployment"></a>SAP ASE 部署結構
SAP ASE 可執行檔應該位於或安裝於 VM OS 磁碟的系統磁碟機 (磁碟機 c:\)。 一般而言，大部分的 SAP ASE 系統和工具資料庫都不會遭遇高工作負載。 因此，系統和工具資料庫 (master、model、saptools、sybmgmtdb、sybsystemdb) 可以保留於 C:\ 磁碟機上。 

有一個例外狀況就是暫存資料庫在某些 SAP ERP 和所有 BW 工作負載中，可能需要較高的資料量或 I/O 作業量。 VM 的 OS 磁碟無法提供的磁碟區或 IOPS (磁碟機 C:\)。

根據用來安裝的 SAPInst/SWPM 版本而言，SAP ASE 執行個體組態可能如下所示︰

* 安裝 SAP ASE 時建立單一的 SAP ASE tempdb
* 安裝 SAP ASE 時建立的 SAP ASE tempdb，以及 SAP 安裝常式所建立的其他 saptempdb
* 透過安裝 SAP ASE 來建立的 SAP ASE tempdb，以及手動建立 (例如依照 SAP 附註 [1752266]) 來符合 ERP/BW 特定 tempdb 需求的其他 tempdb

基於特定 ERP 或所有 BW 工作負載的效能因素，在 C:\ 以外的磁碟機上儲存另外建立的 tempdb 裝置很合理。 如果沒有其他 tempdb 存在，則建議您建立一個 (SAP 附註 [1752266])。

針對這類系統，應該針對另外建立的 tempdb 執行下列步驟︰

* 將第一個 tempdb 裝置移至 SAP 資料庫的第一個裝置
* 將 tempdb 裝置新增至每個包含 SAP 資料庫裝置的 VHD

這個組態讓 tempdb 所耗用的空間比系統磁碟機能夠提供的還多。 作為參考，您可以在內部部署執行的現有系統上檢查 tempdb 裝置的大小。 或者，這類組態可針對無法使用系統磁碟機來提供的 tempdb 啟用 IOPS 數目。 內部部署執行的系統可以用來監視 tempdb 的 I/O 工作負載。

永遠不要將任何 SAP ASE 裝置放入 VM 的 D:\ 磁碟機。 對於 SAP ASE，此裝置也適用於 tempdb，即使 tempdb 中保留的物件只是暫時性的。

如需資料和交易記錄檔部署，請參閱在[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中提出的陳述和建議。 在以 Windows 為主的部署案例中，建議將 Windows 儲存空間用來建置具有足夠 IOPS、輸送量和磁碟區的等量磁碟區組。  

#### <a name="impact-of-database-compression"></a>資料庫壓縮的影響
在 I/O 頻寬可能變成限制因素的組態中，減少 IOPS 的每個量值可能都有助於延展您可以在類似 Azure 的 IaaS 案例中執行的工作負載。 因此，建議您在將現有的 SAP 資料庫上傳至 Azure 之前，確定已使用 SAP ASE 壓縮。

在上傳至 Azure 之前套用壓縮的建議是出自下列數個因素︰

* 降低要上傳至 Azure 的資料量
* 假設您可以使用功能更強大的硬體，其在內部部署中使用更多 CPU 或更高 I/O 頻寬或更少 I/O 延遲，則執行壓縮的持續時間較短
* 較小的資料庫大小可能會使磁碟配置的成本降低

資料壓縮和 LOB 壓縮可以在 Azure 虛擬機器上裝載的 VM 中運作，如同它在內部部署中的運作方式。 如需有關如何檢查在現有 SAP ASE 資料庫中是否已經使用壓縮的更多詳細資料，請參閱 SAP 附註 [1750510]。

#### <a name="using-dbacockpit-to-monitor-database-instances"></a>使用 DBACockpit 監視資料庫執行個體
針對使用 SAP ASE 作為資料庫平台的 SAP 系統，可以存取 DBACockpit 作為交易 DBACockpit 中內嵌的瀏覽器視窗或作為 Webdynpro。 不過，監視和管理資料庫的完整功能只能在 DBACockpit 的 Webdynpro 實作中取得。

做為內部部署系統，需要使用數個步驟，才能啟用 DBACockpit 之 Webdynpro 實作所使用的所有 SAP NetWeaver 功能。 請依照 SAP 附註 [1245200] 來啟用 Webdynpro 的使用，以及產生所需的 Webdynpro。 遵循上述附註中的指示時，您也可設定網際網路通訊管理員 (icm) 以及針對 http 和 https 連線所使用的連接埠。 Http 的預設設定看起來如下︰

> icm/server_port_0 = PROT=HTTP,PORT=8000,PROCTIMEOUT=600,TIMEOUT=600
> 
> icm/server_port_1 = PROT=HTTPS,PORT=443$$,PROCTIMEOUT=600,TIMEOUT=600
> 
> 

此外，在交易 DBACockpit 中產生的連結看起來如下︰

> https://`<fullyqualifiedhostname`>:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http://`<fullyqualifiedhostname`>:8000/sap/bc/webdynpro/sap/dba_cockpit
> 
> 

取決於裝載 SAP 系統的 Azure 虛擬機器是否連線到您的 AD 和 DNS，您需要確定 ICM 會使用完整的主機名稱，此名稱可在您嘗試從中開啟 DBACockpit 的電腦上加以解析。 請參閱 SAP 附註 [773830] 以了解 ICM 如何根據設定檔參數決定完整的主機名稱，並於必要時明確設定 icm/host_name_full 參數。

如果您在僅限雲端的案例中部署 VM，而不需要在內部部署與 Azure 之間跨單位的連線能力，您需要定義一個公用 IP 位址和一個 domainlabel。 然後 VM 的公用 DNS 名稱格式如下所示︰

> `<custom domainlabel`>.`<azure region`>.cloudapp.azure.com
> 
> 

如需有關 DNS 名稱的更多詳細資料，請參閱[這裡][virtual-machines-azurerm-versus-azuresm]。

將 SAP 設定檔參數 icm/host_name_full 設定為 Azure VM 的 DNS 名稱，其連結看起來可能如下︰

> https://mydomainlabel.westeurope.cloudapp.net:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http://mydomainlabel.westeurope.cloudapp.net:8000/sap/bc/webdynpro/sap/dba_cockpit
> 
> 

在此情況下，您需要確定︰

* 在 Azure 入口網站中，針對用來與 ICM 通訊的 TCP/IP 連接埠，將輸入規則新增至網路安全性群組
* 針對用來與 ICM 通訊的 TCP/IP 連接埠，將輸入規則新增至 Windows 防火牆組態

針對自動匯入所有可用修正的功能，建議定期套用適用於您 SAP 版本的修正集合 SAP 附註︰

* [1558958]
* [1619967]
* [1882376]

您可以在下列 SAP 附註中找到適用於 SAP ASE 之 DBA Cockpit 的進一步資訊︰

* [1605680]
* [1757924]
* [1757928]
* [1758182]
* [1758496]    
* [1814258]
* [1922555]
* [1956005]

#### <a name="backuprecovery-considerations-for-sap-ase"></a>SAP ASE 的備份/復原考量
將 SAP ASE 部署至 Azure 時，必須檢閱您的備份方法。 即使在非生產系統中，SAP 資料庫都必須定期備份。 由於 Azure 儲存體會保留三個映像，因此在補償儲存體損毀方面，備份已變得較不重要。 維護適當備份和還原方案的主要原因是，您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原回到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 

在 Azure 中備份和還原資料庫的運作方式，與內部部署中的運作方式一樣。 請參閱 SAP 附註︰

* [1588316]
* [1585981]

以取得建立傾印組態和排程備份的詳細資料。 根據您的策略和需求，您可以在其中一個現有的磁碟上將資料庫和記錄傾印設定至磁碟，或者新增其他磁碟以供備份使用。 為了減少在發生錯誤時產生資料遺失的風險，建議使用其上沒有任何資料庫檔案的磁碟。

除了資料壓縮與 LOB 壓縮之外，SAP ASE 也會提供備份壓縮。 若要讓資料庫和記錄傾印取用較少的空間，建議使用備份壓縮。 如需詳細資訊，請參閱 SAP 附註 [1588316]。 如果您打算從 Azure 虛擬機器將備份或包含備份傾印的 VHD 下載至內部部署，則壓縮備份對於降低要傳輸的資料量來說也很重要。

請勿使用磁碟機 D:\ 做為資料庫或記錄傾印目的地。

#### <a name="performance-considerations-for-backupsrestores"></a>備份/還原的效能考量
如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 請記住，備份壓縮會取用 CPU 資源。 備份壓縮的 CPU 耗用量在具有少量 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，讀取的整體輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (等量目錄、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用/相結合：

* 在多個掛接的磁碟上等量配置備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 在 SAP ASE 層級上建立傾印組態，該層級會使用一個以上的目標目錄來寫入傾印

在多個掛接的磁碟上等量配置磁碟區的方式，已在[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中討論過。 如需有關在 SAP ASE 傾印組態中使用多個目錄的詳細資訊，請參閱 sp_config_dump 預存程序的相關文件，此預存程序可用來在 [Sybase Infocenter](http://infocenter.sybase.com/help/index.jsp) 上建立傾印組態。

### <a name="disaster-recovery-with-azure-vms"></a>使用 Azure VM 的災害復原
#### <a name="data-replication-with-sap-sybase-replication-server"></a>利用 SAP Sybase 複寫伺服器的資料複寫
利用 SAP Sybase 複寫伺服器 (SRS)，SAP ASE 可提供暖待命的解決方案，以非同步方式將資料庫交易傳輸到遠端位置。 

安裝及操作 SRS 也可以在 Azure 虛擬機器中裝載的 VM 上運作，就像它在內部部署的運作方式。

SAP ASE HADR 不需要使用 Azure Internal Load Balancer，對 OS 層級叢集也沒有相依性。 它可在 Azure Windows 和 Linux VM 上運作。 如需 SAP ASE HADR 的詳細資訊，請參閱 [SAP ASE HADR 使用者指南](https://help.sap.com/viewer/efe56ad3cad0467d837c8ff1ac6ba75c/16.0.3.3/en-US/a6645e28bc2b1014b54b8815a64b87ba.html)。

## <a name="specifics-to-sap-ase-on-linux"></a>Linux 上 SAP ASE 專屬的詳細資料
從 Microsoft Azure 開始，您就能輕易地將現有的 SAP ASE 應用程式移轉至 Azure 虛擬機器。 虛擬機器中的 SAP ASE 可讓您輕鬆地將這些應用程式移轉到 Microsoft Azure，藉以減少部署、管理和維護企業級應用程式的擁有權總成本。 透過 Azure 虛擬機器中的 SAP ASE，系統管理員和開發人員仍然可以使用可在內部部署使用的相同開發和管理工具。

若要部署 Azure VM，請務必了解官方 SLA，請參閱：<https://azure.microsoft.com/support/legal/sla>

SAP 附註 [1928533] 會提供 SAP 大小調整資訊和 SAP 認證的 VM SKU 清單。 Azure 虛擬機器的其他 SAP 大小調整文件可在 <http://blogs.msdn.com/b/saponsqlserver/archive/2015/06/19/how-to-size-sap-systems-running-on-azure-vms.aspx> 和 <http://blogs.msdn.com/b/saponsqlserver/archive/2015/12/01/new-white-paper-on-sizing-sap-solutions-on-azure-public-cloud.aspx> 找到

關於 Azure 儲存體使用方式、部署 SAP VM 或 SAP 監視的陳述與建議適用於搭配 SAP 應用程式來部署 SAP ASE，如本文件前四章所述。

下列這兩個 SAP 附註會提供有關 Linux 上的 ASE 以及雲端中 ASE 的通用資訊：

* [2134316]
* [1941500]

### <a name="sap-ase-version-support"></a>SAP ASE 版本支援
SAP 目前支援 SAP ASE 版本 16.0，可與 SAP 商務套件產品搭配使用。 不論是適用於 SAP ASE 伺服器的所有更新，還是要與「SAP 商務套件」產品搭配使用的 JDBC 和 ODBC 驅動程式，都只會透過 SAP Service Marketplace 來提供，網址是：<https://support.sap.com/swdc>。

如同內部部署安裝，不要直接從 Sybase 網站下載適用於 SAP ASE 伺服器或適用於 JDBC 和 ODBC 驅動程式的更新。 如需在內部部署和 Azure 虛擬機器中支援與 SAP 商務套件產品搭配使用之修補程式的詳細資訊，請參閱下列 SAP 附註：

* [1590719]
* [1973241]

如需有關在 SAP ASE 上執行「SAP 商務套件」的一般資訊，請參閱 [SCN](https://www.sap.com/community/topic/ase.html)

### <a name="sap-ase-configuration-guidelines-for-sap-related-sap-ase-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 相關之 SAP ASE 的 SAP ASE 組態指導方針
#### <a name="structure-of-the-sap-ase-deployment"></a>SAP ASE 部署結構
SAP ASE 可執行檔應該位於或安裝於 VM 的根檔案系統 ( /sybase )。 一般而言，大部分的 SAP ASE 系統和工具資料庫都不會遭遇高工作負載。 因此，系統和工具資料庫 (master、model、saptools、sybmgmtdb、sybsystemdb) 可以儲存在根檔案系統上。 

有一個例外狀況就是暫存資料庫在某些 SAP ERP 和所有 BW 工作負載中，可能需要較高的資料量或 I/O 作業量。 VM 的 OS 磁碟無法提供的磁碟區或 IOPS 

根據用來安裝系統的 SAPInst/SWPM 版本，資料庫可能包含︰

* 安裝 SAP ASE 時建立單一的 SAP ASE tempdb
* 安裝 SAP ASE 時建立的 SAP ASE tempdb，以及 SAP 安裝常式所建立的其他 saptempdb
* 透過安裝 SAP ASE 來建立的 SAP ASE tempdb，以及手動建立 (例如依照 SAP 附註 [1752266]) 來符合 ERP/BW 特定 tempdb 需求的其他 tempdb

基於特定 ERP 或所有 BW 工作負載的效能因素，在個別檔案系統上儲存另外建立的 tempdb (透過 SWPM 或手動) tempdb 裝置很合理，該檔案系統可以利用單一 Azure 資料磁碟或橫跨多個 Azure 資料磁碟的 Linux RAID 來表示。 如果沒有其他 tempdb 存在，則建議您建立一個 (SAP 附註 [1752266])。

針對這類系統，應該針對另外建立的 tempdb 執行下列步驟︰

* 將第一個 tempdb 目錄移至 SAP 資料庫的第一個檔案系統
* 將 tempdb 目錄新增至每個包含 SAP 資料庫檔案系統的磁碟

這個組態讓 tempdb 所耗用的空間比系統磁碟機能夠提供的還多。 作為參考，您可以在內部部署執行的現有系統上檢查 tempdb 裝置的大小。 或者，這類組態可針對無法使用系統磁碟機來提供的 tempdb 啟用 IOPS 數目。 內部部署執行的系統可以用來監視 tempdb 的 I/O 工作負載。

永遠不要將任何 SAP ASE 目錄放置於 VM 的 /mnt 或 /mnt/resource 上。 對於 SAP ASE，此裝置也適用於 tempdb，即使 tempdb 中保留的物件只是暫時性的。 因為 /mnt 或 /mnt/resource 是預設的 Azure VM 暫存空間，而不是永續性空間。 如需有關 Azure VM 暫存空間的更多詳細資料，請參閱[這篇文章][virtual-machines-linux-how-to-attach-disk]

如需資料和交易記錄檔部署，請參閱在[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中提出的陳述和建議。 在以 Linux 為主的部署案例中，建議將 LVM 或 MDADM 的使用量用來建置具有足夠 IOPS、輸送量和磁碟區的等量磁碟區組。 

#### <a name="impact-of-database-compression"></a>資料庫壓縮的影響
在 I/O 頻寬可能變成限制因素的組態中，減少 IOPS 的每個量值可能都有助於延展您可以在類似 Azure 的 IaaS 案例中執行的工作負載。 因此，建議您在將現有的 SAP 資料庫上傳至 Azure 之前，確定已使用 SAP ASE 壓縮。

在上傳至 Azure 之前套用壓縮的建議是出自下列數個因素︰

* 降低要上傳至 Azure 的資料量
* 假設您可以使用功能更強大的硬體，其在內部部署中使用更多 CPU 或更高 I/O 頻寬或更少 I/O 延遲，則執行壓縮的持續時間較短
* 較小的資料庫大小可能會使磁碟配置的成本降低

資料壓縮和 LOB 壓縮可以在 Azure 虛擬機器上裝載的 VM 中運作，如同它在內部部署中的運作方式。 如需有關如何檢查在現有 SAP ASE 資料庫中是否已經使用壓縮的更多詳細資料，請參閱 SAP 附註 [1750510]。 如需有關資料庫壓縮的詳細資訊，請參閱 SAP 附註 [2121797]。

#### <a name="using-dbacockpit-to-monitor-database-instances"></a>使用 DBACockpit 監視資料庫執行個體
針對使用 SAP ASE 作為資料庫平台的 SAP 系統，可以存取 DBACockpit 作為交易 DBACockpit 中內嵌的瀏覽器視窗或作為 Webdynpro。 不過，監視和管理資料庫的完整功能只能在 DBACockpit 的 Webdynpro 實作中取得。

做為內部部署系統，需要使用數個步驟，才能啟用 DBACockpit 之 Webdynpro 實作所使用的所有 SAP NetWeaver 功能。 請依照 SAP 附註 [1245200] 來啟用 Webdynpro 的使用，以及產生所需的 Webdynpro。 遵循上述附註中的指示時，您也可設定網際網路通訊管理員 (icm) 以及針對 http 和 https 連線所使用的連接埠。 Http 的預設設定看起來如下︰

> icm/server_port_0 = PROT=HTTP,PORT=8000,PROCTIMEOUT=600,TIMEOUT=600
> 
> icm/server_port_1 = PROT=HTTPS,PORT=443$$,PROCTIMEOUT=600,TIMEOUT=600
> 
> 

而且在交易 DBACockpit 中產生的連結看起來如下︰

> https://`<fullyqualifiedhostname`>:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http://`<fullyqualifiedhostname`>:8000/sap/bc/webdynpro/sap/dba_cockpit
> 
> 

取決於裝載 SAP 系統的 Azure 虛擬機器是否連線到您的 AD 和 DNS，您需要確定 ICM 會使用完整的主機名稱，此名稱可在您嘗試從中開啟 DBACockpit 的電腦上加以解析。 請參閱 SAP 附註 [773830] 以了解 ICM 如何根據設定檔參數決定完整的主機名稱，並於必要時明確設定 icm/host_name_full 參數。

如果您在僅限雲端的案例中部署 VM，而不需要在內部部署與 Azure 之間跨單位的連線能力，您需要定義一個公用 IP 位址和一個 domainlabel。 然後 VM 的公用 DNS 名稱格式如下所示︰

> `<custom domainlabel`>.`<azure region`>.cloudapp.azure.com
> 
> 

如需有關 DNS 名稱的更多詳細資料，請參閱[這裡][virtual-machines-azurerm-versus-azuresm]。

將 SAP 設定檔參數 icm/host_name_full 設定為 Azure VM 的 DNS 名稱，其連結看起來可能如下︰

> https://mydomainlabel.westeurope.cloudapp.net:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http://mydomainlabel.westeurope.cloudapp.net:8000/sap/bc/webdynpro/sap/dba_cockpit
> 
> 

在此情況下，您需要確定︰

* 在 Azure 入口網站中，針對用來與 ICM 通訊的 TCP/IP 連接埠，將輸入規則新增至網路安全性群組
* 針對用來與 ICM 通訊的 TCP/IP 連接埠，將輸入規則新增至 Windows 防火牆組態

針對自動匯入所有可用修正的功能，建議定期套用適用於您 SAP 版本的修正集合 SAP 附註︰

* [1558958]
* [1619967]
* [1882376]

您可以在下列 SAP 附註中找到適用於 SAP ASE 之 DBA Cockpit 的進一步資訊︰

* [1605680]
* [1757924]
* [1757928]
* [1758182]
* [1758496]    
* [1814258]
* [1922555]
* [1956005]

#### <a name="backuprecovery-considerations-for-sap-ase"></a>SAP ASE 的備份/復原考量
將 SAP ASE 部署至 Azure 時，必須檢閱您的備份方法。 即使在非生產系統中，SAP 資料庫都必須定期備份。 由於 Azure 儲存體會保留三個映像，因此在補償儲存體損毀方面，備份已變得較不重要。 維護適當備份和還原方案的主要原因是，您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原回到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 

在 Azure 中備份和還原資料庫的運作方式，與內部部署中的運作方式一樣。 請參閱 SAP 附註︰

* [1588316]
* [1585981]

以取得建立傾印組態和排程備份的詳細資料。 根據您的策略和需求，您可以在其中一個現有的磁碟上將資料庫和記錄傾印設定至磁碟，或者新增其他磁碟以供備份使用。 為了減少在發生錯誤時產生資料遺失的風險，建議使用磁碟 (不會有任何資料庫目錄/檔案位於其中)。

除了資料壓縮與 LOB 壓縮之外，SAP ASE 也會提供備份壓縮。 若要讓資料庫和記錄傾印取用較少的空間，建議使用備份壓縮。 如需詳細資訊，請參閱 SAP 附註 [1588316]。 如果您打算從 Azure 虛擬機器將備份或包含備份傾印的 VHD 下載至內部部署，則壓縮備份對於降低要傳輸的資料量來說也很重要。

不要使用 Azure VM 暫存空間 /mnt 或 /mnt/resource 做為資料庫或記錄傾印目的地。

#### <a name="performance-considerations-for-backupsrestores"></a>備份/還原的效能考量
如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 請記住，備份壓縮會取用 CPU 資源。 備份壓縮的 CPU 耗用量在具有少量 CPU 執行緒的 VM 上扮演重要的角色。  因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，讀取的整體輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (Linux 軟體 RAID、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用/相結合：

* 在多個掛接的磁碟上等量配置備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 在 SAP ASE 層級上建立傾印組態，該層級會使用一個以上的目標目錄來寫入傾印

在多個掛接的磁碟上等量配置磁碟區的方式，已在[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中討論過。 如需有關在 SAP ASE 傾印組態中使用多個目錄的詳細資訊，請參閱 sp_config_dump 預存程序的相關文件，此預存程序可用來在 [Sybase Infocenter](http://infocenter.sybase.com/help/index.jsp) 上建立傾印組態。

### <a name="disaster-recovery-with-azure-vms"></a>使用 Azure VM 的災害復原
#### <a name="data-replication-with-sap-sybase-replication-server"></a>利用 SAP Sybase 複寫伺服器的資料複寫
利用 SAP Sybase 複寫伺服器 (SRS)，SAP ASE 可提供暖待命的解決方案，以非同步方式將資料庫交易傳輸到遠端位置。 

安裝及操作 SRS 也可以在 Azure 虛擬機器中裝載的 VM 上運作，就像它在內部部署的運作方式。

支援透過 SAP 複寫伺服器的 ASE HADR。 強烈建議使用 SAP ASE 16.03 來嘗試這種組態。 如需安裝這類組態的更詳細指示，請參閱這個[部落格](https://blogs.msdn.microsoft.com/saponsqlserver/2018/06/18/installation-procedure-for-sybase-16-3-patch-level-3-always-on-dr-on-suse-12-3-recent-customer-proof-of-concept/)。