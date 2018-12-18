---
title: 適用於 SAP 工作負載的 Oracle Azure 虛擬機器 DBMS 部署 | Microsoft Docs
description: 適用於 SAP 工作負載的 Oracle Azure 虛擬機器 DBMS 部署
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
ms.date: 07/12/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: bda1c4074db123294ffde39e5c4cd5494474ca38
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39075190"
---
# <a name="oracle-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>適用於 SAP 工作負載的 Oracle Azure 虛擬機器 DBMS 部署

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



## <a name="specifics-to-oracle-database"></a>Oracle Database 專屬的詳細資料
Oracle 支援 Oracle 軟體在 Microsoft Azure 上執行。 如需關於 Windows Hyper-V 和 Azure 一般支援的詳細資訊，請查看︰<https://blogs.oracle.com/cloud/entry/oracle_and_microsoft_join_forces> 

遵循一般支援，也支援利用 Oracle 資料庫的 SAP 應用程式特定案例。 詳細資料已詳述於文件中。 在閱讀本文件之前，您應該先參閱[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件，以及 [Azure 上的 SAP 工作負載文件](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)中的其他指南。 

在 SAP 附註 [2039619] 中，可找到 Azure 虛擬機器上的 Oracle 執行 SAP 時所支援的 Oracle 版本和對應的 OS 版本。

如需有關在 Oracle 上執行 SAP 商務套件的一般資訊，請參閱 <https://www.sap.com/community/topic/oracle.html> \(英文\)

與 Oracle、SAP 和 Azure 清單相關的 SAP 附註如下：

下列的 SAP 附註，在本文所討論的領域上與 Azure 上的 SAP 有關：

| 附註編號 | 標題 |
| --- | --- |
| [1928533] |Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型 |
| [2015553] |Microsoft Azure 上的 SAP：支援的必要條件 |
| [1999351] |疑難排解適用於 SAP 且已強化的 Azure 監視功能 |
| [2178632] |Microsoft Azure 上的 SAP 主要監視度量 |
| [2191498] |Linux 和 Azure 上的 SAP：增強型監視功能 |
| [2039619] |Microsoft Azure 上使用 Oracle 資料庫的 SAP 應用程式︰支援的產品和版本 |
| [2243692] |Microsoft Azure (IaaS) VM 上的 Linux：SAP 授權問題 |
| [2069760] |Oracle Linux 7.x SAP 安裝和升級 |
| [1597355] |適用於 Linux 的交換空間建議 |
| [2171857] |Oracle Database 12c - Linux 上的檔案系統支援 |
| [1114181] |Oracle Database 11g - Linux 上的檔案系統支援 |

Oracle 及 Azure 上的 SAP 所支援的確切設定和功能，已記載於 SAP 附註 [#2039619](https://launchpad.support.sap.com/#/notes/2039619) \(英文\) 之中

如您所見，僅支援 Windows 和 Oracle Linux 這兩個客體作業系統。 廣泛使用的 SLES 和 RHEL Linux 並無法在 Azure 中部署 Oracle 元件。 Oracle 元件確實也有包含 Oracle 資料庫用戶端，SAP 應用程式會使用此用戶端來針對 Oracle DBMS 進行連線。 根據 SAP 附註 [#2039619](https://launchpad.support.sap.com/#/notes/2039619) \(英文\) 所述，SAP 元件是唯一的例外，這些元件不會使用 Oracle 用戶端，因為它們可能不需要連線至 Oracle DBMS。 這類的 SAP 元件是 SAP 的獨立加入佇列、訊息伺服器，以及加入佇列複寫服務。 這代表就算是在 Oracle Linux 上執行 Oracle DBMS 和 SAP 應用程式執行個體，您仍舊無法在 SLES 或 RHEL 上執行 SAP 中央服務，並使用以 Pacemaker 為基礎的叢集來保護它。 Oracle Linux 上不支援 HA 設定。

## <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 Oracle 組態指導方針
### <a name="storage-configuration"></a>儲存體組態
只支援 Oracle 使用 NTFS 格式化磁碟的單一執行個體。 所有的資料庫檔案都必須儲存於以 VHD 或受控磁碟為基礎的 NTFS 檔案系統上。 這些磁碟會掛接到 Azure VM，並且以 Azure 分頁 Blob 儲存體 (<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) 或受控磁碟 (<https://docs.microsoft.com/azure/storage/storage-managed-disks-overview>) 為基礎。 任何類型的網路磁碟機或遠端共用 (例如 Azure 檔案服務)：

* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx> 
* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx>

就 Oracle 資料庫檔案而言，都「不」  支援使用！

使用以 Azure 分頁 BLOB 儲存體或受控磁碟為基礎的磁碟時，[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中的描述同樣適用於使用 Oracle Database 所做的部署。

如[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中所述，Azure 磁碟的 IOPS 輸送量具有配額上的限制。 確切的配額會根據使用的 VM 類型而定。 如需 VM 類型及其配額的清單，請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows]。

若要找出支援的 Azure VM 類型，請參閱 SAP 附註 [1928533]。

只要每個磁碟上目前的 IOPS 配額能滿足需求，就可以將所有 DB 資料檔案儲存於單一已掛接的磁碟上。 且兩個重做記錄檔會位於另外兩個磁碟上。

如果需要更多 IOPS ，建議使用 Windows 儲存集區 (僅適用於 Windows Server 2012 和更新版本) ，在多個已掛接的磁碟上建立一個大型邏輯裝置。 這種方法可以簡化系統管理負荷來管理磁碟空間，並避免將檔案手動分散到多個已掛接的磁碟。

根據客戶體驗而做出的建議為：

- 針對重做記錄檔和其鏡像使用不同的磁碟區
- 在考量 IOPS 因素的必要情況下，僅將一個等量組套用至包含重做記錄檔和其鏡像的磁碟區
- 針對 Azure M 系列的 VM，寫入重做記錄檔的延遲可以透過使用 Azure 寫入加速器來大幅降低 (和 Azure 進階儲存體相比之下)。 因此，您應該針對組成 Oracle 重做記錄檔磁碟區的 VHD 來部署 Azure 寫入加速器。 如需詳細資訊，請參閱[寫入加速器](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-enable-write-accelerator)文件。

### <a name="backup--restore"></a>備份 / 還原
針對備份 / 還原功能，適用於 Oracle 的 SAP BR*Tools 的支援方式，與標準的 Windows Server 作業系統相同。 Oracle Recovery Manager (RMAN) 也支援備份至磁碟，以及從磁碟還原。

您也可以使用 Azure 備份服務來執行應用程式一致的 VM 備份。 [在 Azure 中規劃 VM 備份基礎結構](https://docs.microsoft.com/azure/backup/backup-azure-vms-introduction)一文說明 Azure 備份服務會使用 Windows VSS 功能來執行應用程式一致的備份。 Azure 與 SAP 上所支援的 Oracle 資料庫版本，能夠利用 VSS 功能進行備份。 如需詳細資料，請參閱 Oracle 文件[搭配 VSS 進行資料庫備份與復原的基本概念](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/ntqrf/basic-concepts-of-database-backup-and-recovery-with-vss.html#GUID-C085101B-237F-4773-A2BF-1C8FD040C701) \(英文\)。


### <a name="high-availability"></a>高可用性
基於高可用性和災害復原目的支援 Oracle Data Guard。 

Azure 中適用於 Oracle 資料庫的災害復原層面，已詳述於[Azure 環境中 Oracle Database 12c 資料庫的災害復原](https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/oracle-disaster-recovery)中。

### <a name="accelerated-networking"></a>加速網路
針對 Windows 上的 Oracle 部署，強烈建議使用 Azure 的加速網路功能，如 [Azure 加速網路](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) \(英文\) 中所述。 另請考慮在[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中所提出的建議。 

### <a name="other"></a>其他
所有其他一般領域 (例如 Azure 可用性設定組或 SAP 監視) 也同樣適用於使用 Oracle Database 部署 VM 的情況，如[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件中所述。

## <a name="specifics-to-oracle-database-on-oracle-linux"></a>Oracle Linux 上 Oracle 資料庫專屬的詳細資料
Oracle 支援 Oracle 軟體在以 Oracle Linux 為客體 OS 的 Microsoft Azure 上執行。 如需關於 Windows Hyper-V 和 Azure 一般支援的詳細資訊，請查看︰<https://blogs.oracle.com/cloud/entry/oracle_and_microsoft_join_forces> 

遵循一般支援，也支援利用 Oracle 資料庫的 SAP 應用程式特定案例。 詳細資料已在文件的這個部分中命名。

### <a name="oracle-version-support"></a>Oracle 版本支援
在 SAP 附註 [2039619] 中，可找到 Azure 虛擬機器上的 Oracle 執行 SAP 時所支援的 Oracle 版本和對應的 OS 版本。

如需有關在 Oracle 上執行 SAP 商務套件的一般資訊，請參閱 <https://www.sap.com/community/topic/oracle.html> \(英文\)

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 Oracle 組態指導方針
#### <a name="storage-configuration"></a>儲存體組態
只支援 Oracle 使用 ext3、ext4 和 xfs 格式化磁碟的單一執行個體。 所有的資料庫檔案都必須儲存於以 VHD 或受控磁碟為基礎的這些檔案系統上。 這些磁碟會掛接到 Azure VM，並且以 Azure 分頁 Blob 儲存體 (<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) 或受控磁碟 (<https://docs.microsoft.com/azure/storage/storage-managed-disks-overview>) 為基礎。 任何類型的網路磁碟機或遠端共用 (例如 Azure 檔案服務)：

* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx> 
* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx>

就 Oracle 資料庫檔案而言，都「不」  支援使用！

使用以 Azure 分頁 BLOB 儲存體或受控磁碟為基礎的磁碟時，[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中的描述同樣適用於使用 Oracle Database 所做的部署。

如[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件中所述，Azure 磁碟的 IOPS 輸送量具有配額上的限制。 確切的配額會根據使用的 VM 類型而定。 如需 VM 類型及其配額的清單，請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows]。

若要找出支援的 Azure VM 類型，請參閱 SAP 附註 [1928533]。

只要每個磁碟上目前的 IOPS 配額能滿足需求，就可以將所有 DB 檔案儲存於單一已掛接的磁碟上。 且兩個重做記錄檔會位於另外兩個磁碟上。

如果需要更多 IOPS，建議使用 LVM (邏輯磁碟區管理員) 或 MDADM 透過多個已掛接的磁碟建立一個大型邏輯磁碟區。 另請參閱[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件，以取得使用 LVM 或 MDADM 的指導方針和指示。 這種方法可以簡化系統管理負荷來管理磁碟空間，並避免將檔案手動分散到多個已掛接的磁碟。

根據客戶體驗而做出的建議為：

- 針對重做記錄檔和其鏡像使用不同的磁碟區
- 在考量 IOPS 因素的必要情況下，僅將一個等量組套用至包含重做記錄檔和其鏡像的磁碟區
- 針對 Azure M 系列的 VM，寫入重做記錄檔的延遲可以透過使用 Azure 寫入加速器來大幅降低 (和 Azure 進階儲存體相比之下)。 因此，您應該針對組成 Oracle 重做記錄檔磁碟區的 VHD 來部署 Azure 寫入加速器。 如需詳細資訊，請參閱[寫入加速器](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-enable-write-accelerator)文件。


#### <a name="backup--restore"></a>備份 / 還原
針對備份 / 還原功能，利用與裸機和 Hyper-V 上所做的相同方式來支援 SAP BR*Tools for Oracle。 Oracle Recovery Manager (RMAN) 也支援備份至磁碟，以及從磁碟還原。

如需如何使用 Azure 備份與還原服務來備份和還原 Oracle 資料庫的詳細資料，請參閱[在 Azure Linux 虛擬機器上備份及復原 Oracle Database 12c 資料庫](https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/oracle-backup-recovery)文章。

#### <a name="high-availability"></a>高可用性
基於高可用性和災害復原目的支援 Oracle Data Guard。 如需詳細資料，請參閱[在 Azure Linux 虛擬機器上實作 Oracle Data Guard](https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard) 文章。


Azure 中適用於 Oracle 資料庫的災害復原層面，已詳述於[Azure 環境中 Oracle Database 12c 資料庫的災害復原](https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/oracle-disaster-recovery)中。

#### <a name="accelerated-networking"></a>加速網路
Oracle Linux 7 Update 5 (Oracle Linux 7.5) 已針對 Oracle Linux 提供 Azure 加速網路的相關支援。 如果您無法升級至最新的 Oracle Linux 7.5 版，可能的因應措施為使用 RHEL 核心，而非使用 Oracle UEK 核心。 根據 SAP 附註 [#1565179](https://launchpad.support.sap.com/#/notes/1565179) \(英文\)，支援在 Oracle Linux 內使用 RHEL 核心。 不過，請留意最小的 RHEL 核心版本必須是 3.10.0-862.el7.x86_64，Azure 加速網路才能正常執行。


#### <a name="other"></a>其他
所有其他一般領域 (例如「Azure 可用性設定組」或 SAP 監視) 也適用於使用 Oracle 資料庫來部署 VM 的情況，如本文件的前三章中所述。
