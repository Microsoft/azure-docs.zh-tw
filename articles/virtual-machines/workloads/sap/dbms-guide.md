---
title: 適用於 SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署 | Microsoft Docs
description: SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5654dac7-4204-4387-b312-3d8b2898eb3a
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/26/2018
ms.author: sedusch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2caa9a5137edd4e012adf704c01dc5c470e1bb51
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38972439"
---
# <a name="azure-virtual-machines-dbms-deployment-for-sap-netweaver"></a>SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署
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

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

本指南是在 Microsoft Azure 上實作和部署 SAP 軟體之文件的一部分。 閱讀本指南之前，請先閱讀[規劃和實作指南][planning-guide]。 本文說明如何在 Microsoft Azure 虛擬機器 (VM) 上，使用 Azure 基礎結構即服務 (IaaS) 功能搭配 SAP，來部署各種關聯式資料庫管理系統 (RDBMS) 及相關產品。

本白皮書會對「SAP 安裝文件」和「SAP 附註」進行補充說明，指出用來在指定平台上安裝和部署 SAP 軟體的主要資源。

## <a name="general-considerations"></a>一般考量
本章將說明在 Azure VM 中執行 SAP 相關 DBMS 系統的考量。 有數個對於本章中特定 DBMS 系統的參考資料。 在本白皮書內，改為在本章之後討論特定的 DBMS 系統。

### <a name="definitions-upfront"></a>預先定義
我們在本文件中使用下列詞彙︰

* IaaS：基礎結構即服務。
* PaaS：平台即服務。
* SaaS：軟體即服務。
* SAP 元件︰個別 SAP 應用程式，例如 ECC、BW、Solution Manager 或 EP。  SAP 元件可以傳統 ABAP 或 Java 技術為基礎，或以非 NetWeaver 應用程式 (例如商務物件) 為基礎。
* SAP 環境 (SAP Environment)︰一或多個以邏輯方式分組的 SAP 元件，可執行像是開發、QAS、訓練、DR 或生產等商務功能。
* SAP 架構 (SAP Landscape)︰這是指客戶 IP 環境中的整個 SAP 資產。 SAP 環境包含所有生產和非生產環境。
* SAP 系統 (SAP System)︰DBMS 層與應用程式層的組合，例如 SAP ERP 開發系統、SAP BW 測試系統、SAP CRM 生產系統等。在 Azure 部署中，不支援在內部部署與 Azure 之間分割這兩個層級。 這表示 SAP 系統可以在內部部署或在 Azure 部署。 不過，您可以將具 SAP 結構的不同系統部署於 Azure 或內部部署。 例如，您可以在 Azure 部署 SAP CRM 開發和測試系統，但在內部部署 SAP CRM 生產系統。
* 僅限雲端的部署︰在此部署中，Azure 訂用帳戶並未透過站對站或 ExpressRoute 連線，連接到內部部署網路基礎結構。 在一般 Azure 文件中，這類部署也會描述為「僅限雲端」的部署。 以此方法部署的虛擬機器可透過網際網路和指派給 Azure 中 VM 的公用網際網路端點來存取。 內部部署 Active Directory (AD) 和 DNS 不會在這些部署類型中擴充到 Azure。 因此，VM 不是內部部署 Active Directory 的一部分。 附註：僅限雲端的部署在本文中定義為在 Azure 中以獨佔方式執行的完整 SAP 架構，不會將 Active Directory 或名稱解析從內部部署擴充到公用雲端。 SAP 生產系統或組態不支援僅限雲端組態，在此組態中，裝載於 Azure 的 SAP 系統和位於內部部署的資源之間，必須使用 SAP STMS 或其他內部部署資源。
* 跨單位：描述將 VM 部署到 Azure 訂用帳戶的案例，該訂用帳戶在內部部署資料中心與 Azure 之間具有站對站、多站台或 ExpressRoute 連線能力。 在一般 Azure 文件中，這類部署也會描述為跨單位案例。 連線的原因是為了將內部部署網域、內部部署 Active Directory 和內部部署 DNS 擴充到 Azure。 內部部署的架構會擴充到訂用帳戶的 Azure 資產。 在此擴充下，VM 可以是內部部署網域的一部分。 內部部署網域的網域使用者可以存取伺服器，並且可在這些 VM 上執行服務 (例如 DBMS 服務)。 您可以在內部部署的 VM 與 Azure 中部署的 VM 之間進行通訊與名稱解析。 我們預期這是在 Azure 上部署 SAP 資產最常見的案例。 如需詳細資訊，請參閱[這篇文章][vpn-gateway-cross-premises-options]和[這篇文章][vpn-gateway-site-to-site-create]。

> [!NOTE]
> SAP 生產系統支援跨單位部署 SAP 系統，其中執行 SAP 系統的 Azure 虛擬機器是內部部署網域的成員。 支援跨單位組態，以便將部分或完整的 SAP 架構部署到 Azure。 即使在 Azure 中執行完整的 SAP 架構，也會要求這些 VM 隸屬於內部部署網域和 ADS。 在本文件的先前版本中，我們曾談到混合式 IT 案例，其中「混合式」一詞基本上是指內部部署與 Azure 之間有跨單位連線能力。 在此案例中，*混合式*也表示 Azure 中的 VM 是內部部署 Active Directory 的一部分。
> 
> 

有些 Microsoft 文件在描述跨單位案例時稍有不同，特別是針對 DBMS HA 組態。 在 SAP 相關的文件中，跨單位案例會縮減為站對站或私人 (ExpressRoute) 連線能力，以及實際上 SAP 架構會分散至內部部署與 Azure 之間。

### <a name="resources"></a>資源
以下是 Azure 上的 SAP 部署適用的指南︰

* [SAP NetWeaver 的 Azure 虛擬機器規劃和實作指南][planning-guide]
* [SAP NetWeaver 的 Azure 虛擬機器部署][deployment-guide]
* [SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署 (本文件)][dbms-guide]

以下是與 Azure 上的 SAP 有關的 SAP 附註︰

| 附註編號 | 標題 |
| --- | --- |
| [1928533] |Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型 |
| [2015553] |Microsoft Azure 上的 SAP：支援的必要條件 |
| [1999351] |疑難排解適用於 SAP 且已強化的 Azure 監視功能 |
| [2178632] |Microsoft Azure 上的 SAP 主要監視度量 |
| [1409604] |Windows 上的虛擬化︰增強型監視功能 |
| [2191498] |Linux 和 Azure 上的 SAP：增強型監視功能 |
| [2039619] |Microsoft Azure 上使用 Oracle 資料庫的 SAP 應用程式︰支援的產品和版本 |
| [2233094] |DB6︰Azure 上使用 IBM DB2 for Linux, UNIX, and Windows 的應用程式 - 其他資訊 |
| [2243692] |Microsoft Azure (IaaS) VM 上的 Linux：SAP 授權問題 |
| [1984787] |SUSE LINUX Enterprise Server 12：安裝注意事項 |
| [2002167] |Red Hat Enterprise Linux 7.x：安裝和升級 |
| [2069760] |Oracle Linux 7.x SAP 安裝和升級 |
| [1597355] |適用於 Linux 的交換空間建議 |
| [2171857] |Oracle Database 12c - Linux 上的檔案系統支援 |
| [1114181] |Oracle Database 11g - Linux 上的檔案系統支援 |


另請參閱 [SCN Wiki](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes)，其中包含適用於 Linux 的所有 SAP 附註。

您應該具備有關 Microsoft Azure 架構以及如何部署和操作 Microsoft Azure 虛擬機器的有效知識。 您可以在 <https://azure.microsoft.com/documentation/> 找到詳細資訊

> [!NOTE]
> 我們將「不」討論「Microsoft Azure 平台」的「Microsoft Azure 平台即服務」(PaaS) 供應項目。 本白皮書的內容是關於如何在 Microsoft Azure 虛擬機器 (IaaS) 上執行資料庫管理系統 (DBMS)，就像在內部部署環境中執行 DBMS 一樣。 這兩個產品之間所提供的資料庫性能與功能差異極大，不應混用彼此。 另請參閱：<https://azure.microsoft.com/services/sql-database/>
> 
> 

由於我們所討論的是 IaaS，因此，一般而言，Windows、Linux 和 DBMS 的安裝和組態基本上與您要安裝內部部署的任何虛擬機器或裸機機器相同。 不過，有一些架構和系統管理實作決策會與使用 IaaS 時的不同。 本文件的目的是說明您在使用 IaaS 時必須備妥的特定架構和系統管理差異。

一般而言，本白皮書討論的整體差異範圍如下：

* 為 SAP 系統規劃適當的 VM/磁碟配置，以確保您會有適當的資料檔案配置，而且可達到足夠的 IOPS 供您的工作負載使用。
* 使用 IaaS 時的網路功能考量。
* 用來最佳化資料庫配置的特定資料庫功能。
* IaaS 的備份和還原考量。
* 使用不同類型的映像進行部署。
* Azure IaaS 中的高可用性。

## <a name="65fa79d6-a85f-47ee-890b-22e794f51a64"></a>RDBMS 部署結構
為了依循本章內容，必須了解[部署指南][deployment-guide]的[這個][deployment-guide-3]章節所提供的內容。 閱讀本章之前，必須先了解並熟悉有關不同的 VM 系列及其差異，以及 Azure 標準和進階儲存體之差異的知識。

在 2015 年 3 月之前，包含作業系統的磁碟大小限制為 127 GB。 這項限制已在 2015 年 3 月取消 (如需詳細資訊請參閱 <https://azure.microsoft.com/blog/2015/03/25/azure-vm-os-drive-limit-octupled/>)。 從那時起，在包含作業系統的磁碟上，其大小就與任何其他磁碟一樣。 不過，我們仍然偏好部署的結構如下：作業系統、DBMS 和最終的 SAP 二進位檔會與資料庫檔案分隔開來。 因此，我們預期在 Azure 虛擬機器上執行的 SAP 系統會使用作業系統、資料庫管理系統可執行檔和 SAP 可執行檔來安裝基底 VM (或磁碟)。 DBMS 資料和記錄檔會儲存於個別磁碟中的 Azure 儲存體 (標準或進階儲存體)，並以邏輯磁碟形式連接到原始的 Azure 作業系統映像 VM。 

根據利用的是「Azure 標準儲存體」還是「進階儲存體」(例如，使用 DS 系列或 GS 系列的 VM) 而定，Azure 中還有其他的配額 (請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows] 的記載)。 規劃磁碟配置時，您必須找到適用於下列各項的配額最佳平衡︰

* 資料檔案數量。
* 包含檔案的磁碟數目。
* 單一磁碟的 IOPS 配額。
* 每個磁碟的資料輸送量。
* 每個 VM 大小可能的額外資料磁碟數目。
* VM 可提供的整體儲存體輸送量。

Azure 會強制執行每個資料磁碟的 IOPS 配額。 這些配額與 Azure 標準儲存體和進階儲存體上裝載的磁碟不同。 在這兩個進階儲存體傳遞因素比 I/O 延遲更好的儲存體類型之間，I/O 延遲也很不一樣。 不同 VM 類型的每一個可提供來讓您連接的資料磁碟數目有限。 另一個限制是只有特定的 VM 類型可以利用 Azure 進階儲存體。 這表示適用於特定 VM 類型的決策可能不只受到 CPU 和記憶體需求影響，而且也會受到 IOPS、延遲及磁碟輸送量需求影響，這些需求通常是利用磁碟數目或進階儲存體磁碟的類型來調整。 特別是進階儲存體，磁碟的大小可能也會受到每個磁碟需要到達的 IOPS 數目和輸送量所支配。

實際將整體 IOPS 速率、掛接的磁碟數目及 VM 大小全都綁在一起，可能導致 SAP 系統的 Azure 組態與其內部部署產生差異。 每個 LUN 的 IOPS 限制通常可在內部部署中加以設定。 然而使用 Azure 儲存體時，根據磁碟類型而定，這些限制可以是固定的，或者與進階儲存體相同。 因此在內部部署中，我們看到資料庫伺服器的客戶組態會針對特定的可執行檔 (例如 SAP 和 DBMS) 使用許多不同的磁碟區，或者針對暫存資料庫或資料表空間使用特定的磁碟區。 將這類內部部署系統移至 Azure 時，可能會因為針對未執行任何或沒有很多 IOPS 的可執行檔或資料庫浪費了一個磁碟，因而可能產生 IOPS 頻寬的浪費。 所以，在 Azure VM 中，我們建議盡可能將 DBMS 和 SAP 的可執行檔安裝於作業系統磁碟上。

資料庫檔案和記錄檔的位置以及所使用的 Azure 儲存體類型應該依據 IOPS、延遲和輸送量需求來定義。 為了有足夠的 IOPS 可供交易記錄使用，您可能被迫針對交易記錄檔利用多個磁碟或使用更大的進階儲存體磁碟。 在這種情況下，只需使用將包含交易記錄的磁碟來建置軟體 RAID (例如，如果是 Windows，即為「Windows 儲存集區」，或如果是 Linux，則為 MDADM 和 LVM (邏輯磁碟區管理員)) 即可。

- - -
> ![Windows][Logo_Windows] Windows
> 
> Azure VM 中的磁碟機 D:\ 不是永續性磁碟機，而是由 Azure 計算節點上的一些本機磁碟所組成。 由於它不是永續性的，這表示當 VM 重新開機時，即會遺失對 D:\ 磁碟機的內容所做的任何變更。 「任何變更」的意思是已儲存的檔案、已建立的目錄、已安裝的應用程式等。
> 
> ![Linux][Logo_Linux] Linux
> 
> Linux Azure VM 會在 /mnt/resource 上自動掛接磁碟機，這個非永續性磁碟機會利用 Azure 計算節點上的本機磁碟來備份。 由於它不是永續性的，這表示當 VM 重新開機時，會遺失對 /mnt/resource 的內容所做的任何變更。 「任何變更」的意思是已儲存的檔案、已建立的目錄、已安裝的應用程式等。
> 
> 

- - -
根據 Azure VM 系列而定，計算節點上的本機磁碟會顯示可以如下方式分類的不同效能：

* A0-A7︰非常有限的效能。 無法使用 Windows 分頁檔以外的項目
* A8-A11︰效能特性非常良好，有數萬個 IOPS 和 > 1 GB/秒的輸送量
* D 系列︰效能特性非常良好，有數萬個 IOPS 和 > 1 GB/秒的輸送量
* DS 系列︰效能特性非常良好，有數萬個 IOPS 和 > 1 GB/秒的輸送量
* G 系列︰效能特性非常良好，有數萬個 IOPS 和 > 1 GB/秒的輸送量
* GS 系列︰效能特性非常良好，有數萬個 IOPS 和 > 1 GB/秒的輸送量

以上陳述均適用於通過 SAP 認證的 VM 類型。 具有絕佳 IOPS 和輸送量的 VM 系列符合某些 DBMS 功能的使用資格，例如，tempdb 或暫存資料表空間。

### <a name="c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f"></a>VM 和資料磁碟的快取
當我們透過入口網站建立資料磁碟，或在將上傳的磁碟掛接到 VM 時，可以選擇是否要快取在 VM 和位於 Azure 儲存體之磁碟間的 I/O 流量。 Azure 標準和進階儲存體會針對這種快取類型使用兩種不同技術。 在這兩個案例中，快取本身會在 VM 的暫存磁碟 (Windows 上的 D:\ 或 Linux 上的 /mnt/resource) 所使用的同一個磁碟機上進行磁碟備份。

針對 Azure 標準儲存體，可能的快取類型如下︰

* 無快取
* 讀取快取
* 讀取和寫入快取

為了取得一致且具決定性的效能，您應該在「Azure 標準儲存體」上，針對包含 **DBMS 相關資料檔、記錄檔和資料表空間的所有磁碟，將快取設定為「無」**。 VM 的快取可以保留預設值。

針對 Azure 進階儲存體，提供下列快取選項︰

* 無快取
* 讀取快取

對於「Azure 進階儲存體」的建議是利用 SAP 資料庫的「資料檔案讀取快取」，並選擇「不對記錄檔的磁碟進行快取」。

### <a name="c8e566f9-21b7-4457-9f7f-126036971a91"></a>軟體 RAID
如上所述，您需要在可設定磁碟數目，以及 Azure VM 將針對每個磁碟或進階儲存體磁碟類型提供的最大 IOPS 數目之間，平衡資料檔所需的 IOPS 數目。 處理磁碟上 IOPS 負載的最簡單方式是在不同的磁碟上建置軟體 RAID。 然後在劃分出軟體 RAID 的 LUN 上放置多個 SAP DBMS 的資料檔。 根據需求，因為這三個不同進階儲存體磁碟的其中兩個會提供比以標準儲存體為基礎的磁碟更高的 IOPS 配額，所以您可能也要考慮使用進階儲存體。 除了明顯變好之外，Azure 進階儲存體還提供 I/O 延遲。 

同樣適用於不同 DBMS 系統的交易記錄檔。 對其中許多系統而言，新增更多 Tlog 檔案是沒有幫助的，因為 DBMS 系統一次只會寫入其中一個檔案。 如果需要的 IOPS 速率比以單一標準儲存體為基礎的磁碟可提供的速率更高，您可以等量劃分多個標準儲存體磁碟，或者可以使用遠超過 IOPS 速率的更大型進階儲存體磁碟類型，其也可以將降低寫入 I/O 延遲的因素傳遞至交易記錄。

以下是在 Azure 部署中使用優先軟體 RAID 時所遇到的情況：

* 交易記錄/重做記錄需要的 IOPS 數目比 Azure 針對單一磁碟所提供的還多。 如前所述，解決此問題的方式是在多個磁碟上使用軟體 RAID 來建置 LUN。
* 在 SAP 資料庫的不同資料檔之間 I/O 工作負載分配不平均。 在這種情況下，會遇到某個資料檔到達配額的頻率比平常更高的狀況。 而其他資料檔甚至還未接近單一磁碟的 IOPS 配額。 在這種情況下，最簡單的解決方案是在多個磁碟上使用軟體 RAID 來建置一個 LUN。 
* 您不知道每個資料檔確切的 I/O 工作負載，大約只知道以 DBMS 為根據的整體 IOPS 工作負載。 執行此動作的最簡單方式是透過軟體 RAID 的協助來建置一個 LUN。 之後，位於這個 LUN 後方的多個磁碟配額總和應可滿足已知的 IOPS 速率。

- - -
> ![Windows][Logo_Windows] Windows
> 
> 如果您在 Windows Server 2012 或更高版本上執行，則建議使用 Windows 儲存體空間。 它比舊版 Windows 的 Windows 等量更有效率。 當您使用 Windows Server 2012 作為作業系統時，可能需要透過 PowerShell 命令來建立 Windows 儲存體集區和儲存空間。 PowerShell 命令可以在這裡找到：<https://technet.microsoft.com/library/jj851254.aspx>
> 
> ![Linux][Logo_Linux] Linux
> 
> 只支援使用 MDADM 和 LVM (邏輯磁碟區管理員) 在 Linux 上建立軟體 RAID。 如需詳細資訊，請參閱下列文章：
> 
> * [在 Linux 上設定軟體 RAID][virtual-machines-linux-configure-raid] (適用於 MDADM)
> * [在 Azure 中的 Linux VM 上設定 LVM][virtual-machines-linux-configure-lvm]
> 
> 

- - -
利用 VM 系列 (這些系列通常能夠與 Azure 進階儲存體搭配使用) 的考量如下︰

* 要接近 SAN/NAS 裝置所傳遞的 I/O 延遲需求。
* 比 Azure 標準儲存體可傳遞的 I/O 延遲更好的因素需求。
* 對於可根據特定 VM 類型使用多個標準儲存體磁碟來達成的每個 VM 提供更高的 IOPS。

由於基礎的 Azure 儲存體會將每個磁碟複寫到至少三個儲存體節點，因此可以使用簡單的 RAID 0 等量。 不需要實作 RAID5 或 RAID1。

### <a name="10b041ef-c177-498a-93ed-44b3441ab152"></a>Microsoft Azure 儲存體
Microsoft Azure 儲存體會將基底 VM (含 OS) 以及磁碟或 BLOB 儲存到至少 3 個不同的儲存體節點。 建立儲存體帳戶或受控磁碟時，有一個保護選項，如下所示：

![針對 Azure 儲存體帳戶啟用異地複寫][dbms-guide-figure-100]

Azure 儲存體本機複寫 (本地備援) 會根據因為少數客戶提供來部署的基礎結構失敗而導致的資料遺失來提供保護層級。 如上所示，有 4 個不同的選項，第 5 個則是前三個其中之一的變化。 仔細查看我們區分它們的方式︰

* **進階本地備援儲存體 (LRS)**：針對執行需要大量 I/O 之工作負載的虛擬機器，「Azure 進階儲存體」提供高效能、低延遲的磁碟支援。 在 Azure 區域的同一個 Azure 資料中心內有 3 個資料複本。 這些複本位於不同的「容錯網域」和「升級網域」中 (如需相關概念，請參閱[規劃指南][planning-guide]中的[這個][planning-guide-3.2]章節)。 萬一資料複本因為儲存體節點失敗或磁碟失敗而停止服務，即會自動產生新的複本。
* **本地備援儲存體 (LRS)**：在此案例中，在 Azure 區域的同一個 Azure 資料中心內會有 3 個資料複本。 這些複本位於不同的「容錯網域」和「升級網域」中 (如需相關概念，請參閱[規劃指南][planning-guide]中的[這個][planning-guide-3.2]章節)。 萬一資料複本因為儲存體節點失敗或磁碟失敗而停止服務，即會自動產生新的複本。 
* **異地備援儲存體 (GRS)**︰在此案例中，會有一個非同步複寫，此複寫會在另一個 Azure 區域中饋送額外 3 個資料複本，而在大部分情況下，此區域會在同一個地理區域內 (例如，北歐和西歐)。 這會產生 3 個額外的複本，因此總共有 6 個複本。 這個變化讓您能夠基於讀取目的來使用異地複寫之 Azure 區域中的資料 (讀取權限異地備援)。
* **區域備援儲存體 (ZRS)**︰在此案例中，3 個資料複本會保留於同一個 Azure 區域中。 如[規劃指南][planning-guide]的[這個][planning-guide-3.1]章節所述，一個 Azure 區域可以是相近的一些資料中心。 在 LRS 的案例中，複本可以分散至組成一個 Azure 區域的不同資料中心。

如需詳細資訊，請參閱[這裡][storage-redundancy]。

> [!NOTE]
> 針對 DBMS 部署，不建議使用異地備援儲存體
> 
> Azure 儲存體異地複寫是非同步的。 複寫已掛接到單一 VM 的個別磁碟，不會在鎖定步驟中進行同步處理。 因此，不適合複寫已分散至不同磁碟的 DBMS 檔案，或者根據以多個磁碟為基礎的軟體 RAID 來部署的 DBMS 檔案。 DBMS 軟體要求永續性磁碟儲存體在不同的 LUN 和基礎磁碟/磁針上準確地進行同步處理。 DBMS 軟體會使用各種適用於 IO 寫入活動順序的機制，即使有幾毫秒的變化，DBMS 都會報告複寫的目標磁碟儲存體已損毀。 因此，如果您想要讓資料庫組態中的其中一個資料庫延展到多個異地複寫的磁碟，則需要使用資料庫方法和功能來執行這類複寫。 您不應該依賴 Azure 儲存體異地複寫來執行此工作。 
> 
> 最簡單的方式是使用範例系統來說明此問題。 假設您已將 SAP 系統上傳至 Azure，其中有 8 個磁碟包含 DBMS 的資料檔，加上一個包含交易記錄檔的磁碟。 在這 9 個磁碟中，不論資料是否要寫入資料檔或交易記錄檔，每一個 VHD 都會根據 DBMS，以一致性方式將資料寫入其中。
> 
> 若要針對資料正確地進行異地複寫並維護一致的資料庫映像，這九個磁碟的內容全都必須以 I/O 作業根據這九個不同磁碟執行的正確順序進行異地複寫。 不過，Azure 儲存體異地複寫不允許宣告磁碟之間的相依性。 這表示 Microsoft Azure 儲存體異地複寫不會知道這九個不同磁碟中的內容是彼此相關的，而且只有在複寫是以 I/O 作業在這 9 個磁碟上發生的順序進行時，資料變更才會是一致的。
> 
> 除了案例中的異地複寫映像不會提供一致的資料庫映像的可能性很高，還會有效能負面影響，這是透過可嚴重影響效能的異地備援儲存體來顯示。 總之，請勿針對 DBMS 類型的工作負載使用這個類型的儲存體備援。
> 
> 

#### <a name="mapping-vhds-into-azure-virtual-machine-service-storage-accounts"></a>將 VHD 對應至 Azure 虛擬機器服務儲存體帳戶
此章節只適用於 Azure 儲存體帳戶。 如果您打算使用受控磁碟，則不適用本章節提到的限制。 如需受控磁碟的詳細資訊，請參閱本指南的[受控磁碟][dbms-guide-managed-disks]章節。

Azure 儲存體帳戶不只是一個系統管理的建構，而且還是限制的緣由。 然而，限制會根據我們討論的是 Azure 標準儲存體或 Azure 進階儲存體帳戶而有所不同。 如需了解確切的功能和限制，請參閱[這裡][storage-scalability-targets]

因此，就「Azure 標準儲存體」而言，要特別注意每個儲存體帳戶都有 IOPS 限制 ([這篇文章][storage-scalability-targets]中包含**總要求率**的資料列)。 此外，每個 Azure 訂用帳戶的「儲存體帳戶」初始限制為 100 個 (自 2015 年 7 月起)。 因此，建議在使用 Azure 標準儲存體時，平衡多個儲存體帳戶之間 VM 的 IOPS。 另一方面，單一 VM 最好盡可能使用一個儲存體帳戶。 因此，如果我們談到 DBMS 部署，其中每個裝載於 Azure 標準儲存體的 VHD 會到達其配額限制時，您應該只針對每個使用 Azure 標準儲存體的 Azure 儲存體帳戶部署 30-40 個 VHD。 相反地，如果您利用 Azure 進階儲存體，並且想要儲存大型資料庫磁碟區，您在 IOPS 方面可能不會遇到問題。 但是，Azure 進階儲存體帳戶在資料磁碟區方面會比 Azure 標準儲存體帳戶更嚴格。 因此，在到達資料磁碟區限制之前，您只能在 Azure 進階儲存體帳戶內部署有限的 VHD 數目。 最後，將 Azure 儲存體帳戶想像為「虛擬 SAN」，其在 IOPS 和/或容量中的功能有限。 因此，工作將會保留 (如同在內部部署中)，以便在不同的「虛構 SAN 裝置」或 Azure 儲存體帳戶上定義不同 SAP 系統的 VHD 配置。

就「Azure 標準儲存體」而言，建議儘量不要對單一 VM 呈現來自不同儲存體帳戶的儲存體。

使用 DS 或 GS 系列的 Azure VM 時，可以掛接來自「Azure 標準儲存體帳戶」和「進階儲存體帳戶」的 VHD。 像是將備份寫入標準儲存體備份的 VHD，使其在進階儲存體上具有 DBMS 資料和記錄檔的使用案例，會在腦海中浮現可在何處利用這類異質儲存體的想法。 

根據客戶部署和測試，大約可在單一 Azure 標準儲存體帳戶上，利用可接受的效能來佈建 30 到 40 個含有資料庫資料檔和記錄檔的 VHD。 如先前所述，Azure 進階儲存體帳戶的限制很可能是它可持有的資料容量，而不是 IOPS。

使用內部部署的 SAN 裝置，共用需要一些監視，以便最終能夠偵測到 Azure 儲存體帳戶上的瓶頸。 適用於 SAP 的 Azure 監視擴充功能和 Azure 入口網站等工具可用來偵測可能提供次佳 IO 效能的忙碌 Azure 儲存體帳戶。  如果偵測到這種情況，則建議將忙碌的 VM 移到另一個 Azure 儲存體帳戶。 如需有關如何啟用 SAP 主機監視功能的詳細資訊，請參閱[部署指南][deployment-guide]。

您可以在下列網址中，找到另一篇摘要說明「Azure 標準儲存體」和「Azure 標準儲存體帳戶」最佳做法的文章：<https://blogs.msdn.com/b/mast/archive/2014/10/14/configuring-azure-virtual-machines-for-optimal-storage-performance.aspx>

#### <a name="f42c6cb5-d563-484d-9667-b07ae51bce29"></a>受控磁碟
受控磁碟是 Azure Resource Manager 的新資源類型，可用來取代儲存在 Azure 儲存體帳戶的 VHD。 受控磁碟會針對所連接的虛擬機器，自動配合其可用性設定組，因此提高了虛擬機器和虛擬機器上所執行服務的可用性。 若要深入了解，請閱讀[概觀](https://docs.microsoft.com/azure/storage/storage-managed-disks-overview)一文。

SAP 目前僅支援進階受控磁碟。 如需詳細資訊，請讀取 SAP 附註 [1928533]。

#### <a name="moving-deployed-dbms-vms-from-azure-standard-storage-to-azure-premium-storage"></a>將部署的 DBMS VM 從 Azure 標準儲存體移至 Azure 進階儲存體
我們遇到了相當多客戶想要將部署的 VM 從 Azure 標準儲存體移至 Azure 進階儲存體的案例。 如果您的磁碟儲存在 Azure 儲存體帳戶中，這不可能在沒有實際移動資料的情況下完成。 有數種方法能夠達成此目標：

* 您可以將所有的 VHD、基底 VHD 以及資料 VHD 複製到新的 Azure 進階儲存體帳戶。 很多時候，您在 Azure 標準儲存體中選擇 VHD 數目不是因為您需要資料磁碟區。 但是，您需要這麼多 VHD 是因為 IOPS 的緣故。 現在，您移至 Azure 進階儲存體，您可以採用較少 VHD 數目的方式來達到相同 IOPS 輸送量。 事實上，在 Azure 標準儲存體中，您是針對使用的資料而不是名義上的磁碟大小來付費，因此，VHD 的數目與成本無關。 不過，使用 Azure 進階儲存體，您就要支付名義上的磁碟大小。 因此，大部分的客戶嘗試將進階儲存體中 Azure VHD 的數目保留為達到 IOPS 輸送量所需的數目。 所以，大多數客戶會根據簡單的 1:1 複製方式進行決策。
* 如果尚未掛接，您要掛接單一 VHD，其中可包含 SAP 資料庫的資料庫備份。 備份之後，您要取消掛接所有包括含有備份之 VHD 的 VHD，以及將基底 VHD 和含有備份的 VHD 複製到 Azure 進階儲存體帳戶。 然後根據基底 VHD 部署 VM，並掛接含有備份的 VHD。 現在您可以針對用來還原資料庫的目標 VM，建立額外的空白進階儲存體磁碟。 這會假設 DBMS 可讓您變更資料和記錄檔的路徑，以做為還原程序的一部分。
* 另一個可能性是先前程序的變化：您將備份 VHD 複製到 Azure 進階儲存體，然後根據您最新部署與安裝的 VM 來連接它。
* 當您需要變更資料庫的資料檔數目時，會選擇第四個可能性。 在這種情況下，您會使用匯出/匯入來執行 SAP 同質性系統複製。 將這些匯出檔案放到 VHD (已複製到 Azure 進階儲存體帳戶)，然後將它連接到您用來執行匯入程序的 VM。 主要是在客戶想要減少資料檔數目時，才會使用這個可能性。

如果您使用受控磁碟，您可以移轉到進階儲存體，方法如下：

1. 解除配置虛擬機器
1. 如有需要，將虛擬機器的大小調整為支援進階儲存體 (例如 DS 或 GS) 的大小
1. 將受控磁碟帳戶類型變更為進階 (SSD)
1. 如 [VM 和資料磁碟快取][dbms-guide-2.1]一章的建議，變更資料磁碟的快取
1. 啟動虛擬機器

### <a name="deployment-of-vms-for-sap-in-azure"></a>在 Azure 中部署適用於 SAP 的 VM
Microsoft Azure 提供多種方法來部署 VM 和相關聯的磁碟。 因此，請務必了解這些差異，因為 VM 的準備工作可能會因部署方法而異。 我們通常會查看下列章節中所描述的案例。

#### <a name="deploying-a-vm-from-the-azure-marketplace"></a>從 Azure Marketplace 部署 VM
您想要從 Azure Marketplace 使用 Microsoft 或第三方提供的映像來部署 VM。 在 Azure 中部署您的 VM 之後，您可以遵循相同的指導方針和工具在 VM 中安裝 SAP 軟體，就像在內部部署環境中所做的一樣。 若要在 Azure VM 中安裝 SAP 軟體，SAP 和 Microsoft 建議將 SAP 安裝媒體上傳並儲存到磁碟，或建立作為「檔案伺服器」並包含所有必要 SAP 安裝媒體的 Azure VM。

#### <a name="deploying-a-vm-with-a-customer-specific-generalized-image"></a>使用客戶特定的一般化映像部署 VM
基於您的 OS 或 DBMS 版本的特定修補程式需求，Azure Marketplace 所提供的映像可能不符合您的需求。 因此，您可能必須使用自己「私人」的 OS/DBMS VM 映像來建立 VM，之後可多次部署此映像。 若要準備這類「私人」映像以供複製，您必須在內部部署 VM 上將作業系統一般化。 如需有關如何將 VM 一般化的詳細資訊，請參閱[部署指南][deployment-guide]。

如果您已在內部部署 VM (特別是針對 2 層系統) 中安裝 SAP 內容，您可以在部署 Azure VM 之後，透過 SAP Software Provisioning Manager 支援的執行個體重新命名程序來調整 SAP 系統設定 (SAP 附註 [1619720])。 否則，您稍後可以在部署 Azure VM 之後安裝 SAP 軟體。

正如 SAP 應用程式所使用的資料庫內容，您可以透過 SAP 安裝產生全新的內容，或者可以使用 VHD 搭配 DBMS 資料庫備份，或是利用 DBMS 的功能直接備份至 Microsoft Azure 儲存體，來將內容匯入 Azure。 在此情況下，您可以也備妥 VHD 以及內部部署的 DBMS 資料檔和記錄檔，然後將它們當成磁碟匯入 Azure。 但是，傳輸從內部部署載入 Azure 的 DBMS 資料，會在需要於內部部署中備妥的 VHD 磁碟上運作。

#### <a name="moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a>使用非一般化磁碟將 VM 從內部部署移至 Azure
您打算將特定的 SAP 系統從內部部署移至 Azure (提升與轉變)。 這可藉由將包含作業系統、SAP 二進位檔和最終 DBMS 二進位檔的磁碟，以及包含 DBMS 資料和記錄檔的磁碟上傳至 Azure 來完成。 相對於上述的案例 2，您可以在 Azure VM 中保留主機名稱、SAP SID 和 SAP 使用者帳戶，因為這些項目已設定於內部部署環境中。 因此，不需要將映像一般化。 此情況最適用於跨單位案例，其中 SAP 架構的一部分是在內部部署執行，其餘部分則在 Azure 上執行。

## <a name="871dfc27-e509-4222-9370-ab1de77021c3"></a>Azure VM 的相關高可用性和災害復原
Azure 提供下列高可用性 (HA) 和災害復原 (DR) 功能，這些功能可以套用至我們用來進行 SAP 和 DBMS 部署的不同元件

### <a name="vms-deployed-on-azure-nodes"></a>部署於 Azure 節點上的 VM
Azure 平台不會針對部署的 VM 提供像是即時移轉等功能。 這表示，如果部署 VM 的伺服器叢集上需要維護，該 VM 就必須停止並重新啟動。 Azure 中的維護作業是在伺服器叢集內使用升級網域來執行。 一次只會維護一個升級網域。 在執行這類重新啟動期間，當 VM 關閉、執行維護，以及 VM 重新啟動時，服務會中斷。 不過，大部分的 DBMS 廠商都不會提供高可用性和災害復原功能，此功能會在主要節點無法使用時，在另一個節點上快速重新啟動 DBMS 服務。 Azure 平台提供將 VM、儲存體及其他 Azure 服務分散到升級網域的功能，以確保預計的維護或基礎結構故障只會影響一小部分的 VM 或服務。  透過謹慎的規劃，就可以達到相當於內部部署基礎結構的可用性層級。

「Microsoft Azure 可用性設定組」是 VM 或「服務」的邏輯群組，可確保 VM 和其他服務會分散到叢集內不同的「容錯網域」和「升級網域」，使得在任何時間點都只有一個節點會被關閉 (如需更多詳細資料，請參閱[這篇 (Linux)][virtual-machines-manage-availability-linux] 或[這篇 (Windows)][virtual-machines-manage-availability-windows] 文章)。

推出 VM 時，必須依用途來設定，如下所示：

![適用於 DBMS HA 組態的可用性設定組定義][dbms-guide-figure-200]

如果想要建立 DBMS 部署的高可用性組態 (而不是個別使用 DBMS HA 功能)，則 DBMS VM 需要︰

* 將 VM 新增至相同的 Azure 虛擬網路 (<https://azure.microsoft.com/documentation/services/virtual-network/>)
* HA 組態的 VM 也應該位於相同的子網路中。 不同子網路之間的名稱解析無法在僅限雲端的部署中運作，只有 IP 解析才能正常運作。 針對跨單位部署使用站對站或 ExpressRoute 連線能力，就已經建立了至少含有一個子網路的網路。 名稱解析是根據內部部署 AD 原則和網路基礎結構來完成。 



#### <a name="ip-addresses"></a>IP 位址
強烈建議使用彈性方式來設定 HA 組態的 VM。 除非使用靜態 IP 位址，否則，在 Azure 中依賴 IP 位址來處理 HA 組態內的 HA 夥伴並不可靠。 Azure 中有兩種「關閉」概念︰

* 透過 Azure 入口網站或 Azure PowerShell Cmdlet Stop-AzureRmVM 來關閉︰在此情況下，會關閉虛擬機器並取消配置。 之後不會再針對此 VM 向您的 Azure 帳戶收取費用，因此，只有在使用儲存體時才會向您收費。 不過，如果網路介面的私人 IP 位址不是靜態的，即會發行該 IP 位址，但不保證在重新啟動 VM 之後，會再次將舊的 IP 位址指派給網路介面。 透過 Azure 入口網站或透過呼叫 Stop-AzureRmVM 來執行關閉，會自動取消配置。 如果您不希望使用 Stop-AzureRmVM -StayProvisioned 取消配置該機器 
* 如果您從作業系統層級關閉 VM，該 VM 即會關閉且不會取消配置。 不過，在此情況下，仍會向您的 Azure 帳戶收取 VM 的費用，儘管它已經關閉也一樣。 在這種情況下，將 IP 位址指派給停止運作的 VM 仍會維持不變。 從中關閉 VM 不會自動強制取消配置。

實際上對於跨內部部署案例而言，關閉和取消配置預設都表示要從 VM 中取消 IP 位址的指派，即使 DHCP 設定中的內部部署原則不同也一樣。 

* 有一個例外狀況，就是如果您如[這裡][virtual-networks-reserved-private-ip]所述，將靜態 IP 位址指派給網路介面。
* 在這種情況下，只要網路介面未遭到刪除，IP 位址就會保持固定。

> [!IMPORTANT]
> 為了讓整個部署保持簡單且容易管理，明確的建議是在 Azure 的 DBMS HA 或 DR 組態中設定 VM 合夥，方法是在所涉及的不同 VM 之間提供正常運作的名稱解析。
> 
> 

## <a name="deployment-of-host-monitoring"></a>部署主機監視功能
若要使 Azure 虛擬機器中的 SAP 應用程式以具生產力的方式運作，SAP 需要能夠從執行 Azure 虛擬機器的實際主機取得主機監視資料。 需要有特定的 SAP HostAgent 修補程式等級，才能在 SAPOSCOL 和 SAP HostAgent 中啟用此功能。 確切的修補程式等級記載於 SAP 附註 [1409604]。

如需了解如何部署將主機資料傳遞給 SAPOSCOL 和 SAPHostAgent 的元件及這些元件的生命週期管理，請參閱[部署指南][deployment-guide]

## <a name="3264829e-075e-4d25-966e-a49dad878737"></a>Microsoft SQL Server 專屬的詳細資料
### <a name="sql-server-iaas"></a>SQL Server IaaS
從 Microsoft Azure 開始，您就能輕易地將建置於 Windows Server 平台上現有的 SQL Server 應用程式移轉至 Azure 虛擬機器。 虛擬機器中的 SQL Server 可讓您輕鬆地將這些應用程式移轉到 Microsoft Azure，藉以減少部署、管理和維護企業級應用程式的擁有權總成本。 透過 Azure 虛擬機器中的 SQL Server，系統管理員和開發人員仍然可以使用可在內部部署使用的相同開發和管理工具。 

> [!IMPORTANT]
> 我們不討論 Microsoft Azure SQL Database，此為 Microsoft Azure 平台的「平台即服務」供應項目。 本白皮書中討論的是如何執行 SQL Server 產品 (已知適用於 Azure 虛擬機器中的內部部署)，以及如何運用 Azure 的「基礎架構即為服務」功能。 這兩個產品之間所提供的資料庫性能與功能並不相同，不應混用彼此。 另請參閱：<https://azure.microsoft.com/services/sql-database/>
> 
> 

建議您先檢閱[這份][virtual-machines-sql-server-infrastructure-services]文件之後再繼續。

在下列章節中，會彙總並提及上述連結下方之文件的某些部分。 也會提到 SAP 專屬的詳細資料，並更深入說明一些概念。 不過，強烈建議您完整閱讀該文件，然後再閱讀 SQL Server 專屬的文件。

繼續之前，您應該先了解一些 IaaS 中 SQL Server 專屬的資訊：

* **虛擬機器 SLA**：如需在 Azure 中執行的虛擬機器適用的 SLA，請參閱：<https://azure.microsoft.com/support/legal/sla/>  
* **SQL 版本支援**︰針對 SAP 客戶，我們在「Microsoft Azure 虛擬機器」上支援 SQL Server 2008 R2 和更新版本。 不支援舊版。 如需更多詳細資料，請檢閱這份通用的 [支援聲明](https://support.microsoft.com/kb/956893) 。 請注意，Microsoft 通常也支援 SQL Server 2008。 不過，由於適用於 SAP 的重大功能是透過 SQL Server 2008 R2 所引進，因此 SQL Server 2008 R2 是適用於 SAP 的最低版本。 請記住，SQL Server 2012 和 2014 已擴充來與 IaaS 案例進行更深入整合 (例如，直接對 Azure 儲存體進行備份)。 因此，我們將本白皮書範圍限制為 SQL Server 2012 和 2014 及其適用於 Azure 的最新修補程式等級。
* **SQL 功能支援**︰「Microsoft Azure 虛擬機器」上支援大部分的 SQL Server 功能，但有一些例外。 **不支援使用共用磁碟的 SQL Server 容錯移轉叢集**。  單一 Azure 區域內支援分散式技術 (例如，資料庫鏡像、AlwaysOn 可用性群組、複寫、記錄傳送，以及 Service Broker)。 在不同的 Azure 區域之間也可支援 SQL Server AlwaysOn，相關資訊請見：<https://blogs.technet.com/b/dataplatforminsider/archive/2014/06/19/sql-server-alwayson-availability-groups-supported-between-microsoft-azure-regions.aspx>。  如需更多詳細資料，請檢閱這份 [支援聲明](https://support.microsoft.com/kb/956893) 。 如需有關如何部署 AlwaysOn 組態的範例，請參閱[這篇][virtual-machines-workload-template-sql-alwayson]文章。 此外，也請參閱[這裡][virtual-machines-sql-server-infrastructure-services]所記載的最佳做法。 
* 
  **SQL 效能**︰相較於其他公用雲端虛擬化供應項目，我們確信 Microsoft Azure 裝載的「虛擬機器」會有非常好的表現，但個別結果可能有所不同。 請參閱[這篇][virtual-machines-sql-server-performance-best-practices]文章。
* **使用來自 Azure Marketplace 的映像**︰部署新 Microsoft Azure VM 的最快方式就是使用來自 Azure Marketplace 的映像。 Azure Marketplace 中提供包含 SQL Server 的映像。 已經安裝 SQL Server 的映像不能立即用於 SAP NetWeaver 應用程式。 原因是預設的 SQL Server 定序是安裝於這些映像內，而不是 SAP NetWeaver 系統所需的定序。 若要使用這類映像，請參閱[使用來自 Microsoft Azure Marketplace 的 SQL Server 映像][dbms-guide-5.6]一章中記載的步驟。 
* 如需詳細資訊，請參閱 [定價詳細資料](https://azure.microsoft.com/pricing/) 。 [SQL Server 2012 授權指南](https://download.microsoft.com/download/7/3/C/73CAD4E0-D0B5-4BE5-AB49-D5B886A5AE00/SQL_Server_2012_Licensing_Reference_Guide.pdf)和 [SQL Server 2014 授權指南](https://download.microsoft.com/download/B/4/E/B4E604D9-9D38-4BBA-A927-56E4C872E41C/SQL_Server_2014_Licensing_Guide.pdf)也是相當重要的資源。

### <a name="sql-server-configuration-guidelines-for-sap-related-sql-server-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 相關之 SQL Server 的 SQL Server 組態指導方針
#### <a name="recommendations-on-vmvhd-structure-for-sap-related-sql-server-deployments"></a>適用於 SAP 相關之 SQL Server 部署的 VM/VHD 結構建議
根據一般的描述，SQL Server 可執行檔應該位於或安裝於 VM OS 磁碟的系統磁碟機 (磁碟機 C:\))。  一般而言，依照 SAP NetWeaver 工作負載，大部分的 SQL Server 系統資料庫都不會在高等級中使用。 因此，SQL Server 的系統資料庫 (master、msdb 和 model) 也可以保留於 C:\ 磁碟機。 有一個例外狀況是 tempdb，在某些 SAP ERP 和所有 BW 工作負載中，可能需要較高的資料量或 I/O 作業量，以致於不適合放在原始 VM 中。 針對這類系統，應該執行下列步驟︰

* 將主要 tempdb 資料檔移至與 SAP 資料庫的主要資料檔相同的邏輯磁碟機。
* 將任何其他 tempdb 資料檔新增到其他包含 SAP 使用者資料庫之資料檔的邏輯磁碟區。
* 將 tempdb 記錄檔新增到包含使用者資料庫記錄檔的邏輯磁碟機。
* **(僅適用於計算節點上使用本機 SSD 的 VM 類型)** tempdb 資料檔和記錄檔可以放在 D:\ 磁碟機上。 不過，可能會建議使用多個 tempdb 資料檔。 請注意，D:\ 磁碟機磁碟區會根據 VM 類型而不同。

這些組態讓 tempdb 所耗用的空間比系統磁碟機能夠提供的還多。 若要判斷正確的 tempdb 大小，使用者可以在內部部署中執行的現有系統上檢查 tempdb 大小。 此外，這類組態可針對無法使用系統磁碟機來提供的 tempdb 啟用 IOPS 數目。 同樣地，在內部部署執行的系統可用於監視 tempdb 的 I/O 工作負載，讓您可以衍生出預期要在 tempdb 上看到的 IOPS 數目。

執行含有 SAP 資料庫的 SQL Server 且 tempdb 資料和 tempdb 記錄檔放置於 D:\ 磁碟機的 VM 組態應該像這樣︰

![適用於 SAP 之 Azure IaaS VM 的參考組態][dbms-guide-figure-300]

請注意，根據 VM 類型而定，D:\ 磁碟機會有不同的大小。 根據 tempdb 的大小需求而定，萬一 D:\ 磁碟機太小，可能要強制您將 tempdb 資料和記錄檔與 SAP 資料庫資料和記錄檔進行配對。

#### <a name="formatting-the-disks"></a>將磁碟格式化
針對 SQL Server，適用於含有 SQL Server 資料和記錄檔之磁碟的 NTFS 區塊大小應該是 64 K。 不需要將 D:\ 磁碟機格式化。 此磁碟機已預先格式化。

若要確定還原或建立資料庫不會藉由清空檔案的內容來初始化資料檔，您應該確定 SQL Server 服務執行所在的使用者內容具有特定的權限。 通常，Windows 系統管理員群組中的使用者會擁有這些權限。 在非 Windows 系統管理員使用者的使用者內容中執行 SQL Server 服務時，您需要為該使用者指派**執行磁碟區維護工作**的使用者權限。  詳細資料請參閱這篇 Microsoft 知識庫文章︰<https://support.microsoft.com/kb/2574695>

#### <a name="impact-of-database-compression"></a>資料庫壓縮的影響
在 I/O 頻寬可能變成限制因素的組態中，減少 IOPS 的每個量值可能都有助於延展您可以在類似 Azure 的 IaaS 案例中執行的工作負載。 因此，如果尚未這麼做，SAP 和 Microsoft 建議您套用 SQL Server 頁面壓縮，然後再將現有的 SAP 資料庫上傳至 Azure。

在上傳至 Azure 之前執行資料庫壓縮的建議是出自下列兩項因素︰

* 降低要上傳的資料量。
* 假設您可以使用功能更強大的硬體，其在內部部署中使用更多 CPU 或更高 I/O 頻寬或更少 I/O 延遲，則執行壓縮的持續時間較短。
* 較小的資料庫大小可能會使磁碟配置的成本降低

資料庫壓縮也可以在 Azure 虛擬機器中運作，如同它在內部部署中的運作方式。 如需關於如何壓縮現有 SAP SQL Server 資料庫的詳細資訊，請參閱：<https://blogs.msdn.com/b/saponsqlserver/archive/2010/10/08/compressing-an-sap-database-using-report-msscompress.aspx>

### <a name="sql-server-2014---storing-database-files-directly-on-azure-blob-storage"></a>SQL Server 2014 – 將資料庫檔案直接儲存於 Azure Blog 儲存體上
SQL Server 2014 開放將資料庫檔案直接儲存於 Azure Blob 儲存體上的可能性，而不需在其周圍使用 VHD 的「包裝函式」。 特別是在使用標準 Azure 儲存體或較小的 VM 類型時，這讓您可以在其中克服 IOPS 的限制，此限制是透過可以掛接到某些較小型 VM 類型的有限磁碟數目來強制執行。 不過，這適用於 SQL Server 的使用者資料庫，而不適用於系統資料庫。 這也適用於 SQL server 的資料和記錄檔。 如果您想要使用此方式部署 SAP SQL Server 資料庫，而不是將它「包裝」到 VHD，請記住下列資訊︰

* 所使用之儲存體帳戶所在的 Azure 區域必須與用來部署 SQL Server 執行所在之 VM 的相同。
* 稍早列出有關將 VHD 分散到不同 Azure 儲存體帳戶的考量，也適用於這種部署方法。 表示對 Azure 儲存體帳戶限制的 I/O 作業計數。

[comment]: <> (MSSedusch TODO 但這將會使用網路頻寬而非儲存體頻寬，不是嗎？)

這種部署類型的詳細資料列載於︰<https://docs.microsoft.com/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure>

若要將 SQL Server 資料檔直接儲存在「Azure 進階儲存體」上，您必須具備下列網址所記載的最基本 SQL Server 2014 修補程式版本：<https://support.microsoft.com/kb/3063054>。 將 SQL Server 資料檔儲存於 Azure 標準儲存體上，利用 SQL Server 2014 的發行版本來運作。 不過，相同的修補程式包含另一個系列的修正程式，其可以更可靠的方式針對 SQL Server 資料檔和備份使用 Azure Blob 儲存體。 因此，我們通常建議使用這些修補程式。

### <a name="sql-server-2014-buffer-pool-extension"></a>SQL Server 2014 緩衝集區延伸
SQL Server 2014 引進的新功能，稱為「緩衝集區延伸」。 此功能會擴充 SQL Server 的緩衝集區，使用第二層快取將其保留於記憶體中，此快取是透過伺服器或 VM 的本機 SSD 來支援。 這可在「記憶體內部」保留更大型的資料工作集。 相較於存取 Azure 標準儲存體，基於許多因素，存取儲存於 Azure VM 之本機 SSD 上的緩衝集區延伸的速度更快。  因此，利用具有絕佳 IOPS 和輸送量之 VM 類型的本機 D:\ 磁碟機可能是一個非常合理的方式，可降低對於 Azure 儲存體的 IOPS 負載，並大幅提升查詢的回應時間。 這特別適用於未使用進階儲存體時。 在進階儲存體以及使用計算節點上的進階 Azure 讀取快取中，對於資料檔的建議，和預期的沒有太大差異。 原因在於這兩個快取 (SQL Server 緩衝集區延伸和進階儲存體讀取快取) 都是使用計算節點的本機磁碟。
如需關於此功能的詳細資料，請參閱這份文件︰<https://docs.microsoft.com/sql/database-engine/configure-windows/buffer-pool-extension> 

### <a name="backuprecovery-considerations-for-sql-server"></a>SQL Server 的備份/復原考量
將 SQL Server 部署至 Azure 時，必須檢閱您的備份方法。 即使系統不是生產系統，還是必須定期備份 SQL Server 所裝載的 SAP 資料庫。 由於 Azure 儲存體會保留三個映像，因此在補償儲存體損毀方面，備份現在已變得較不重要。 優先維護適當備份和復原方案的原因是，您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原回到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 例如，您可以藉由還原備份，從 2 層 SAP 組態轉移到同一個系統的 3 層系統設定。

有三種不同方式可將 SQL Server 備份至 Azure 儲存體︰

1. SQL Server 2012 CU4 和更新版本可以原生方式將資料庫備份至 URL。 下列部落格有詳細說明： [New functionality in SQL Server 2014 - Part 5 - Backup/Restore Enhancements (SQL Server 2014 的新功能 – 第 5 部分 – 備份/還原增強功能)](https://blogs.msdn.com/b/saponsqlserver/archive/2014/02/15/new-functionality-in-sql-server-2014-part-5-backup-restore-enhancements.aspx)。 請參閱 [SQL Server 2012 SP1 CU4 和更新版本][dbms-guide-5.5.1]一章。
2. SQL 2012 CU4 之前的 SQL Server 版本可以使用重新導向功能來備份到 VHD，而且寫入串流基本上會流向已設定的 Azure 儲存體位置。 請參閱 [SQL Server 2012 SP1 CU3 和舊版][dbms-guide-5.5.2]一章。
3. 最後一個方法是在磁碟裝置上執行傳統 SQL Server 備份到磁碟的命令。 這與內部部署模式完全相同，但不會在本文中詳細討論。

#### <a name="0fef0e79-d3fe-4ae2-85af-73666a6f7268"></a>SQL Server 2012 SP1 CU4 和更新版本
此功能可讓您直接備份到 Azure BLOB 儲存體。 如果沒有這個方法，您必須備份到其他此磁碟，因而會耗用磁碟和 IOPS 容量。 以下為基本概念：

 ![使用 SQL Server 2012 備份至 Microsoft Azure 儲存體 BLOB][dbms-guide-figure-400]

此案例的優點是您不需要耗用磁碟來儲存 SQL Server 備份。 因此，所配置的磁碟較少，而且可以針對資料和記錄檔使用磁碟 IOPS 的整個頻寬。 請注意，如下列文章的**限制**一節所記載，備份的大小上限是 1 TB︰<https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-to-url#limitations>。 如果在使用「SQL Server 備份」壓縮的情況下，備份大小仍然會超過 1 TB，就需要使用本文件的 [SQL Server 2012 SP1 CU3 和舊版][dbms-guide-5.5.2]一章中所述的功能。

描述從「Azure Blob 存放區」之備份還原資料庫的[相關文件](https://docs.microsoft.com/sql/relational-databases/backup-restore/restoring-from-backups-stored-in-microsoft-azure)建議在備份大於 25 GB 的情況下，不要直接從 Azure BLOB 存放區還原。 本文中的建議以效能考量為依據，並未將功能限制納入考量。 因此，隨著案例的不同，可能會發生不同的狀況。

如需有關如何設定和利用此備份類型的說明，請參閱 [這個](https://docs.microsoft.com/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016) 教學課程

您可以在 [這裡](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-to-url)找到一系列步驟的範例。

自動備份，最重要的就是確定每個備份的 BLOB 都是以不同方式命名。 否則，它們會遭到覆寫且還原鏈會中斷。

為了避免混用這 3 種不同的備份類型，建議您在用來備份的儲存體帳戶底下建立不同的容器。 容器可以僅供 VM 使用，也可以供 VM 和備份類型使用。 結構描述看起來如下︰

 ![使用 SQL Server 2012 備份至 Microsoft Azure 儲存體 BLOB - 個別儲存體帳戶中的不同容器][dbms-guide-figure-500]

在上述範例中，備份不會執行到部署 VM 的相同儲存體帳戶。 有專用於備份的新儲存體帳戶。 在儲存體帳戶內，會有使用備份類型和 VM 名稱的組合所建立的不同容器。 這種細分方式可讓您輕鬆地管理不同 VM 的備份。

直接寫入備份的 BLOB，不會納入 VM 的資料磁碟計數。 因此，您可以將適用於資料和交易記錄檔之特定 VM SKU 且已掛接的資料磁碟數目上限最大化，並且仍能針對儲存體容器執行備份。 

#### <a name="f9071eff-9d72-4f47-9da4-1852d782087b"></a>SQL Server 2012 SP1 CU3 和舊版
若要直接封存「Azure 儲存體」的備份，您必須執行的第一個步驟就是下載連結到[這篇](https://www.microsoft.com/download/details.aspx?id=40740) KBA 文章的 msi。

下載 x64 安裝檔案和文件。 此檔案會安裝下列程式：**Microsoft SQL Server Backup to Microsoft Azure Tool**。 請仔細閱讀產品文件。  此工具基本上會以下列方式運作︰

* 從 SQL Server 端，定義 SQL Server 備份的磁碟位置 (請以 D:\ 磁碟機作為位置)。
* 此工具可讓您定義可用來將不同備份類型導向至不同 Azure 儲存體容器的規則。
* 一旦規則準備就緒，此工具就會將備份的寫入串流重新導向至稍早定義之 Azure 儲存體位置的其中一個 VHD/磁碟。
* 此工具會在針對 SQL Server 備份定義的 VHD/磁碟上，保留數個 KB 大小的小型 stub 檔案。 **此檔案應該保留在儲存體位置上，因為它需要再次從 Azure 儲存體還原。**
  * 如果您遺失了 stub 檔案 (例如，因為遺失了包含 stub 檔案的儲存媒體)，而您選擇了備份到 Microsoft Azure 儲存體帳戶，則您可能會透過 Microsoft Azure 儲存體，從放置 stub 檔案的儲存體容器中下載它，藉以復原該檔案。 如果加密會與原始規則搭配使用，請將 stub 檔案放置於本機電腦上的資料夾中，並已在其中設定此工具，使用相同的加密密碼來偵測並上傳至同一個容器。 

這表示上述適用於較新版 SQL Server 的結構描述也適用於不允許直接處理 Azure 儲存體位置的 SQL Server 版本。

此方法不應與較新版的 SQL Server 搭配使用，這些版本支援以原生方式備份 Azure 儲存體。 例外狀況是對 Azure 的原生備份限制會封鎖對 Azure 執行原生備份。

#### <a name="other-possibilities-to-back-up-sql-server-databases"></a>備份 SQL Server 資料庫的其他可能性
備份資料庫的其他可能性是將其他資料磁碟連結到您用來儲存備份的 VM。 在這種情況下，您必須確認磁碟並未完整執行。 如果是這種情況，您必須取消掛接磁碟，也就是所謂的「封存」它，然後使用新的空白磁碟來取代它。 如果您執行此動作，就會想要將這些 VHD 保留於個別的 Azure 儲存體帳戶中 (不同於具有含資料庫檔案之 VHD 的帳戶)。

第二種可能性是使用可以連接許多磁碟的大型 VM，例如具有 32 個 VHD 的 D14。 使用儲存空間來建立有彈性的環境，您可以在其中建置共用，之後用來做為不同 DBMS 伺服器的備份目標。

[這裡](https://blogs.msdn.com/b/sqlcat/archive/2015/02/26/large-sql-server-database-backup-on-an-azure-vm-and-archiving.aspx) 也記載了一些最佳做法。 

#### <a name="performance-considerations-for-backupsrestores"></a>備份/還原的效能考量
如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 此外，備份壓縮所使用的 CPU 耗用量在最多有 8 個 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料檔的磁碟數目越少，讀取的整體輸送量就越小。
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重。
* 要寫入備份的目標 (BLOB、VHD 或磁碟) 越少，輸送量就越低。
* VM 大小越小，從 Azure 儲存體寫入和讀取的儲存體輸送量配額就越小。 無關於是否要將備份直接儲存於 Azure Blob 上，或者是否要將它們儲存在再次儲存於 Azure Blob 的 VHD 上。

在較新版本中使用 Microsoft Azure 儲存體 BLOB 做為備份目標時，您就只能針對每個特定備份指定一個 URL 目標。

但是，在舊版中使用 "Microsoft SQL Server backup to Microsoft Azure Tool" 時，您可以定義多個檔案目標。 使用多個目標，就可以調整備份且備份的輸送量會更高。 這接著也會在 Azure 儲存體帳戶中產生多個檔案。 在此測試中，使用多個檔案目的地，絕對可以達到輸送量，而您可以藉由從 SQL Server 2012 SP1 CU4 實作備份延伸模組來達成。 您也不會遭到 1 TB 限制所封鎖，就像在 Azure 的原生備份一樣。

不過，請記住，輸送量也會取決於您用來備份的 Azure 儲存體帳戶位置。 最好是將儲存體帳戶放置於執行 VM 以外的不同區域中。 例如，您在西歐執行 VM 組態，但將您用來備份的儲存體帳戶放在北歐。 這當然會對備份輸送量產生影響，而且很可能不會產生每秒 150MB 的輸送量，因為看似會發生目標儲存體和 VM在同一個區域資料中心內執行的情況。

#### <a name="managing-backup-blobs"></a>管理備份 BLOB
若要自行管理備份，有一個需求。 由於經常執行交易記錄備份會建立多個 Blob 是預期狀況，所以，這些 Blob 的系統管理員很容易就會使 Azure 入口網站過度負載。 因此，建議使用 Azure 儲存體總管。 有數個好工具可用來協助管理 Azure 儲存體帳戶

* 安裝 Azure SDK 的 Microsoft Visual Studio (<https://azure.microsoft.com/downloads/>)
* Microsoft Azure 儲存體總管 (<https://azure.microsoft.com/downloads/>)
* 第三方工具

如需有關備份和 Azure 上的 SAP 的更完整討論，請參閱 [SAP 備份指南](sap-hana-backup-guide.md)以取得詳細資訊。

### <a name="1b353e38-21b3-4310-aeb6-a77e7c8e81c8"></a>使用來自 Microsoft Azure Marketplace 的 SQL Server 映像
Microsoft 在 Azure Marketplace 中提供已經包含 SQL Server 版本的 VM。 對於需要 SQL Server 和 Windows 授權的 SAP 客戶，這可能是透過組織已安裝 SQL Server 的 VM，大致涵蓋授權需求的機會。 若要針對 SAP 使用這類映像，必須進行下列考量︰

* SQL Server 非評估版本的取得成本高於從 Azure Marketplace 部署的「僅限 Windows」VM。 請參閱下列文章以比較價格：<https://azure.microsoft.com/pricing/details/virtual-machines/windows/> 和 <https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-enterprise/>。 
* 您只能使用 SAP 支援的 SQL Server 版本，例如 SQL Server 2012。
* 安裝於 Azure Marketplace 中提供之 VM 上的 SQL Server 執行個體定序，並不是 SAP NetWeaver 要求執行之 SQL Server 執行個體的定序。 不過您可以利用下一節的指示來變更定序。

#### <a name="changing-the-sql-server-collation-of-a-microsoft-windowssql-server-vm"></a>變更 Microsoft Windows/SQL Server VM 的 SQL Server 定序
因為 Azure Marketplace 中的 SQL Server 映像不是設定來使用 SAP NetWeaver 應用程式所需的定序，所以需要在部署之後立即變更。 針對 SQL Server 2012，一旦部署 VM 且系統管理員能夠登入已部署的 VM 之後，就能夠使用下列步驟來完成此動作：

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

如果這不是結果，請停止部署 SAP，並調查為什麼安裝命令並未如預期般運作。 「不」  支援將 SAP NetWeaver 應用程式部署到 SQL Server 字碼頁與上述提及之字碼頁不同的 SQL Server 執行個體。

### <a name="sql-server-high-availability-for-sap-in-azure"></a>SQL Server 在 Azure 中適用於 SAP 的高可用性
如本文稍早所述，您無法建立使用最舊 SQL Server 高可用性功能所需的共用儲存體。 此功能會在 Windows Server 容錯移轉叢集 (WSFC) 中針對使用者資料庫 (以及最終的 tempdb) 使用共用磁碟，來安裝兩個以上的 SQL Server 執行個體。 這是 SAP 也支援的長期標準高可用性方法。 由於 Azure 不支援共用儲存體，所以無法實現具有共用磁碟叢集組態的 SQL Server 高可用性組態。 不過，仍有許多其他高可用性方法，如下列各節所述。

#### <a name="sql-server-log-shipping"></a>SQL Server 記錄傳送
高可用性 (HA) 的方法之一是 SQL Server 記錄傳送。 如果參與 HA 組態的 VM 具有運作中的名稱解析，就不會發生問題，而 Azure 中的設定與內部部署中完成的任何設定並無任何差別。 不建議只依賴 IP 解析。 如需設定記錄傳送和記錄傳送原則的相關事宜，請參閱這份文件︰

<https://docs.microsoft.com/sql/database-engine/log-shipping/about-log-shipping-sql-server>

為了達成任何高可用性，您需要部署位於這類記錄傳送組態內的 VM，使其位於相同的 Azure 可用性設定組內。

#### <a name="database-mirroring"></a>資料庫鏡像
SAP 支援的「資料庫鏡像」(請參閱 SAP 附註 [965908]) 有賴於在 SAP 連接字串中定義容錯移轉夥伴。 針對跨單位案例，我們假設這兩個 VM 位於相同網域，且這兩個 SQL Server 執行個體執行所在的使用者內容也是網域使用者，而且在所涉及的這兩個 SQL Server 執行個體中具備足夠的權限。 因此，在 Azure 中設定資料庫鏡像，在一般內部部署設定/組態之間並無任何差別。

至於僅限雲端的部署，最簡單的方式是在 Azure 中設定另一個網域，以便讓這些 DBMS VM (最好專屬於 SAP VM) 位於一個網域。

如果網域不可行，您也可以針對資料庫鏡像端點使用憑證，相關說明請見︰<https://docs.microsoft.com/sql/database-engine/database-mirroring/use-certificates-for-a-database-mirroring-endpoint-transact-sql>

在 Azure 中設定資料庫鏡像的教學課程可在下列網址中找到︰<https://docs.microsoft.com/sql/database-engine/database-mirroring/database-mirroring-sql-server> 

#### <a name="sql-server-always-on"></a>SQL Server Always On
由於針對內部部署的 SAP 支援 AlwaysOn (請參閱 SAP 附註 [1772688])，因此支援將它與 Azure 中的 SAP 搭配使用。 事實上，您無法在 Azure 中建立共用磁碟，但這不表示您無法在不同 VM 之間建立 AlwaysOn Windows Server 容錯移轉叢集 (WSFC) 組態。 這只表示您不能在叢集組態中使用共用磁碟做為仲裁。 因此您可以在 Azure 中建置 AlwaysOn WSFC 組態，而不選取利用共用磁碟的仲裁類型。 部署這些 VM 的 Azure 環境應該會依名稱解析 VM，而 VM 應該位於同一個網域。 這適用於僅限 Azure 和跨單位部署。 有一些關於 SQL Server 可用性群組接聽程式 (請勿與 Azure 可用性設定組混淆) 的特殊考量，因為 Azure 可能在內部部署中，所以目前不允許建立 AD/DNS 物件。 因此，需要有一些不同的安裝步驟來克服 Azure 的特定行為。

以下是使用可用性群組接聽程式的一些考量︰

* 使用可用性群組接聽程式，只能使用 Windows Server 2012 或更高版本作為 VM 的客體作業系統。 針對 Windows Server 2012，務必要套用此修補程式︰<https://support.microsoft.com/kb/2854082> 
* Windows Server 2008 R2 則沒有此修補程式，必須以和使用「資料庫鏡像」相同的方式使用 AlwaysOn，方法是在連接字串中指定容錯移轉夥伴 (透過 SAP default.pfl 參數 dbs/mss/server 來完成 - 請參閱 SAP 附註 [965908])。
* 使用可用性群組接聽程式時，資料庫 VM 需要連接到專用的負載平衡器。 僅限雲端部署中的名稱解析可能會要求 SAP 系統的所有 VM (應用程式伺服器、DBMS 伺服器及 (A)SCS 伺服器) 位於同一個虛擬網路，或者從 SAP 應用程式層要求維護 etc\host 檔案，以解析 SQL Server VM 的 VM 名稱。 為了避免 Azure 在這兩個 VM 意外關閉的情況下指派新的 IP 位址，您應該在 AlwaysOn 組態中為這些 VM 的網路介面指派靜態 IP 位址 (如需了解如何定義靜態 IP 位址，請參閱[這篇][virtual-networks-reserved-private-ip]文章)

[comment]: <> (舊部落格)
[comment]: <> (<https://blogs.msdn.com/b/alwaysonpro/archive/2014/08/29/recommendations-and-best-practices-when-deploying-sql-server-alwayson-availability-groups-in-windows-azure-iaas.aspx>, <https://blogs.technet.com/b/rmilne/archive/2015/07/27/how-to-set-static-ip-on-azure-vm.aspx>) 
* 建置叢集需要指派特定 IP 位址的 WSFC 叢集組態時需要特殊的步驟，因為具有其目前功能的 Azure 會為叢集名稱指派與叢集建立所在的節點相同的 IP 位址。 這表示必須執行手動步驟，為叢集指派不同的 IP 位址。
* 可用性群組接聽程式將建立於具備 TCP/IP 端點的 Azure 中，這些端點會指派給執行可用性群組之主要和次要複本的 VM。
* 可能需要使用 ACL 保護這些端點。

[comment]: <> (TODO 舊部落格)
[comment]: <> (若要了解在 Azure 上安裝 AlwaysOn 組態的詳細步驟和必要條件，最佳體驗方式就是逐步完成[這裡][virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]提供的教學課程)
[comment]: <> (透過 Azure 映像庫預先設定的 AlwaysOn 安裝程式 <https://blogs.technet.com/b/dataplatforminsider/archive/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery.aspx>)
[comment]: <> ([這個][virtual-machines-windows-classic-ps-sql-int-listener] 教學課程提供建立「可用性群組接聽程式」的最佳說明)
[comment]: <> (以下提供使用 ACL 來保護網路端點的最佳說明：)
[comment]: <> (*    <https://michaelwasham.com/windows-azure-powershell-reference-guide/network-access-control-list-capability-in-windows-azure-powershell/>)
[comment]: <> (*    <https://blogs.technet.com/b/heyscriptingguy/archive/2013/08/31/weekend-scripter-creating-acls-for-windows-azure-endpoints-part-1-of-2.aspx> )
[comment]: <> (*    <https://blogs.technet.com/b/heyscriptingguy/archive/2013/09/01/weekend-scripter-creating-acls-for-windows-azure-endpoints-part-2-of-2.aspx>)  
[comment]: <> (*    <https://blogs.technet.com/b/heyscriptingguy/archive/2013/09/18/creating-acls-for-windows-azure-endpoints.aspx>) 

您也可以在不同的 Azure 區域上部署 SQL Server AlwaysOn 可用性群組。 此功能會利用 Azure VNet 對 Vnet 連線 ([更多詳細資料][virtual-networks-configure-vnet-to-vnet-connection])。

[comment]: <> (TODO 舊部落格)
[comment]: <> (這裡描述了在這種情況下 SQL Server AlwaysOn 可用性群組的設定：<https://blogs.technet.com/b/dataplatforminsider/archive/2014/06/19/sql-server-alwayson-availability-groups-supported-between-microsoft-azure-regions.aspx>。) 

#### <a name="summary-on-sql-server-high-availability-in-azure"></a>Azure 中 SQL Server 高可用性的摘要
由於 Azure 儲存體會保護內容，因此沒有理由堅持要有熱待命映像。 這表示您的高可用性案例只需要在下列情況中提供保護：

* 因為在 Azure 的伺服器叢集上進行維護或其他理由而導致 VM 完全無法使用
* SQL Server 執行個體中的軟體問題
* 防止發生資料遭到刪除而需要進行時間點復原的手動錯誤

尋找相符的技術，您可能認為前兩個案例可以使用資料庫鏡像或 AlwaysOn 來解決，但第三個案例僅能使用記錄傳送來解決。

相較於資料庫鏡像，您必須利用 AlwaysOn 來平衡更複雜的 AlwaysOn 設定。 以下列出這些優點︰

* 可讀取的次要複本。
* 來自次要複本的備份。
* 延展性較佳。
* 一個以上的次要複本。

### <a name="9053f720-6f3b-4483-904d-15dc54141e30"></a>適用於 Azure 上 SAP 的一般 SQL Server 摘要
本指南中提供許多建議，而我們建議您在規劃 Azure 部署之前，多次閱讀本指南。 但是，一般而言，請務必在 Azure 特定點上遵循前十個一般的 DBMS︰

[comment]: <> (2.3 輸送量比什麼更高？比一個 VHD 高？)
1. 使用最新的 DBMS 版本 (例如 SQL Server 2014) ，其在 Azure 中最具優勢。 針對 SQL Server，這是 SQL Server 2012 SP1 CU4，其中包含對 Azure 儲存體進行備份的功能。 不過，若要與 SAP 合併，建議至少要使用 SQL Server 2014 SP1 CU1 或 SQL Server 2012 SP2 和最新的 CU。
2. 在 Azure 中仔細規劃您的 SAP 系統架構，以平衡資料檔案配置和 Azure 限制︰
   * 不需要有太多磁碟，但必須足以確保您可以連線到所需的 IOPS。
   * 如果您未使用受控磁碟，請記住每個「Azure 儲存體帳戶」也都有 IOPS 限制，而且「儲存體帳戶」在每個 Azure 訂用帳戶內是有限的 ([更多詳細資料][azure-subscription-service-limits])。 
   * 只有在您需要達到更高的輸送量時，才需在磁碟上劃分等量磁碟區。
3. 永遠不要在 D:\ 磁碟機上安裝軟體或放置任何需要永久保留的檔案，因為它不是永久性的，此磁碟機上的一切會在 Windows 重新開機時遺失。
4. 不要針對 Azure 標準儲存體使用磁碟快取。
5. 不要使用 Azure 異地備援的儲存體帳戶。  針對 DBMS 工作負載使用本機備援。
6. 使用 DBMS 廠商的 HA/DR 解決方案來複寫資料庫資料。
7. 一律使用名稱解析，不要依賴 IP 位址。
8. 盡可能使用最高度的資料庫壓縮。 這是 SQL Server 的頁面壓縮。
9. 請務必謹慎使用來自 Azure Marketplace 的 SQL Server 映像。 如果您使用 SQL Server 的映像，就必須變更執行個體定序，才能在其上安裝任何 SAP NetWeaver 系統。
10. 依照[部署指南][deployment-guide]所述，安裝並設定適用於 Azure 的 SAP 主機監視功能。

## <a name="specifics-to-sap-ase-on-windows"></a>Windows 上 SAP ASE 專屬的詳細資料
從 Microsoft Azure 開始，您就能輕易地將現有的 SAP ASE 應用程式移轉至 Azure 虛擬機器。 虛擬機器中的 SAP ASE 可讓您輕鬆地將這些應用程式移轉到 Microsoft Azure，藉以減少部署、管理和維護企業級應用程式的擁有權總成本。 透過 Azure 虛擬機器中的 SAP ASE，系統管理員和開發人員仍然可以使用可在內部部署使用的相同開發和管理工具。

Azure 虛擬機器有 SLA，可以在下列位置中找到：<https://azure.microsoft.com/support/legal/sla/virtual-machines>

相較於其他公用雲端虛擬化供應項目，我們確信 Microsoft Azure 裝載的虛擬機器會執行得很順利，但產生的個別結果可能不同。 如需不同 SAP 認證之 VM SKU 的 SAP 大小調整 SAPS 數目，請參閱個別的 SAP 附註 [1928533]。

關於 Azure 儲存體使用方式、部署 SAP VM 或 SAP 監視的陳述與建議適用於搭配 SAP 應用程式來部署 SAP ASE，如本文件前四章所述。

### <a name="sap-ase-version-support"></a>SAP ASE 版本支援
SAP 目前支援 SAP ASE 版本 16.0，可與 SAP 商務套件產品搭配使用。 不論是適用於 SAP ASE 伺服器的所有更新，還是要與「SAP 商務套件」產品搭配使用的 JDBC 和 ODBC 驅動程式，都只會透過 SAP Service Marketplace 來提供，網址是：<https://support.sap.com/swdc>。

如同內部部署安裝，不要直接從 Sybase 網站下載適用於 SAP ASE 伺服器或適用於 JDBC 和 ODBC 驅動程式的更新。 如需在內部部署和 Azure 虛擬機器中支援與 SAP 商務套件產品搭配使用之修補程式的詳細資訊，請參閱下列 SAP 附註：

* [1590719]
* [1973241]

如需有關在 SAP ASE 上執行「SAP 商務套件」的一般資訊，請參閱 [SCN](https://www.sap.com/community/topic/ase.html)

### <a name="sap-ase-configuration-guidelines-for-sap-related-sap-ase-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 相關之 SAP ASE 的 SAP ASE 組態指導方針
#### <a name="structure-of-the-sap-ase-deployment"></a>SAP ASE 部署結構
根據一般的描述，SAP ASE 可執行檔應該位於或安裝於 VM OS 磁碟的系統磁碟機 (磁碟機 c:\))。 一般而言，大部分的 SAP ASE 系統和工具資料庫，SAP NetWeaver 工作負載都不常使用。 因此，系統和工具資料庫 (master、model、saptools、sybmgmtdb、sybsystemdb) 也可以保留於 C:\ 磁碟機上。 

有一個例外狀況可能是暫存資料庫包含 SAP ASE 建立的所有工作資料表和暫存資料表，萬一某些 SAP ERP 和所有 BW 工作負載可能要求更高的資料量或 I/O 作業量，而其無法符合原始 VM 的 OS 磁碟 (磁碟機 c:\))，就會發生此情況。

根據用來安裝系統的 SAPInst/SWPM 版本，資料庫可能包含︰

* 安裝 SAP ASE 時建立單一的 SAP ASE tempdb
* 安裝 SAP ASE 時建立的 SAP ASE tempdb，以及 SAP 安裝常式所建立的其他 saptempdb
* 透過安裝 SAP ASE 來建立的 SAP ASE tempdb，以及手動建立 (例如依照 SAP 附註 [1752266]) 來符合 ERP/BW 特定 tempdb 需求的其他 tempdb

萬一發生特定的 ERP 或所有 BW 工作負載，就能基於效能因素，在 C:\ 以外的磁碟機上保留另外建立之 tempdb (透過 SWPM 或手動) 的 tempdb 裝置。 如果沒有其他 tempdb 存在，則建議您建立一個 (SAP 附註 [1752266])。

針對這類系統，應該針對另外建立的 tempdb 執行下列步驟︰

* 將第一個 tempdb 裝置移至 SAP 資料庫的第一個裝置
* 將 tempdb 裝置新增至每個包含 SAP 資料庫裝置的 VHD

這個組態讓 tempdb 所耗用的空間比系統磁碟機能夠提供的還多。 作為參考，您可以在內部部署執行的現有系統上檢查 tempdb 裝置的大小。 或者，這類組態可針對無法使用系統磁碟機來提供的 tempdb 啟用 IOPS 數目。 再次提醒，內部部署執行的系統可以用來監視 tempdb 的 I/O 工作負載。

永遠不要將任何 SAP ASE 裝置放入 VM 的 D:\ 磁碟機。 這也適用於 tempdb，即使 tempdb 中保留的物件只是暫時性的。

#### <a name="impact-of-database-compression"></a>資料庫壓縮的影響
在 I/O 頻寬可能變成限制因素的組態中，減少 IOPS 的每個量值可能都有助於延展您可以在類似 Azure 的 IaaS 案例中執行的工作負載。 因此，建議您在將現有的 SAP 資料庫上傳至 Azure 之前，確定已使用 SAP ASE 壓縮。

建議在上傳至 Azure 之前執行壓縮，如果尚未實作，可能是因為下列數種因素所導致：

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

取決於裝載 SAP 系統的 Azure 虛擬機器是否以及如何透過站對站、多重網站或 ExpressRoute (跨單位部署) 來連接，您需要確定 ICM 會使用完整的主機名稱，此名稱可在您嘗試從中開啟 DBACockpit 的電腦上加以解析。 請參閱 SAP 附註 [773830] 以了解 ICM 如何根據設定檔參數決定完整的主機名稱，並於必要時明確設定 icm/host_name_full 參數。

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
將 SAP ASE 部署至 Azure 時，必須檢閱您的備份方法。 即使系統不是生產系統，還是必須定期備份 SAP ASE 所裝載的 SAP 資料庫。 由於 Azure 儲存體會保留三個映像，因此在補償儲存體損毀方面，備份現在已變得較不重要。 維護適當備份和還原方案的主要原因是，您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原回到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 例如，您可以藉由還原備份，從 2 層 SAP 組態轉移到同一個系統的 3 層系統設定。

在 Azure 中備份和還原資料庫的運作方式，與內部部署中的運作方式一樣。 請參閱 SAP 附註︰

* [1588316]
* [1585981]

以取得建立傾印組態和排程備份的詳細資料。 根據您的策略和需求，您可以在其中一個現有的磁碟上將資料庫和記錄傾印設定至磁碟，或者新增其他磁碟以供備份使用。 為了減少在發生錯誤時產生資料遺失的風險，建議使用磁碟 (不會有任何資料庫裝置位於其中)。

除了資料壓縮與 LOB 壓縮之外，SAP ASE 也會提供備份壓縮。 若要讓資料庫和記錄傾印佔用較少的空間，建議使用備份壓縮。 如需詳細資訊，請參閱 SAP 附註 [1588316]。 如果您打算從 Azure 虛擬機器將備份或包含備份傾印的 VHD 下載至內部部署，則壓縮備份對於降低要傳輸的資料量來說也很重要。

請勿使用磁碟機 D:\ 做為資料庫或記錄傾印目的地。

#### <a name="performance-considerations-for-backupsrestores"></a>備份/還原的效能考量
如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 此外，備份壓縮所使用的 CPU 耗用量在最多有 8 個 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，讀取的整體輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (等量目錄、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用/相結合：

* 在多個掛接的磁碟上等量劃分備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 在 SAP ASE 層級上建立傾印組態，該層級會使用一個以上的目標目錄來寫入傾印

本指南的先前內容中已討論過在多個掛接的磁碟上等量劃分磁碟區的方式。 如需有關在 SAP ASE 傾印組態中使用多個目錄的詳細資訊，請參閱 sp_config_dump 預存程序的相關文件，此預存程序可用來在 [Sybase Infocenter](http://infocenter.sybase.com/help/index.jsp) 上建立傾印組態。

### <a name="disaster-recovery-with-azure-vms"></a>使用 Azure VM 的災害復原
#### <a name="data-replication-with-sap-sybase-replication-server"></a>利用 SAP Sybase 複寫伺服器的資料複寫
利用 SAP Sybase 複寫伺服器 (SRS)，SAP ASE 可提供暖待命的解決方案，以非同步方式將資料庫交易傳輸到遠端位置。 

安裝及操作 SRS 也可以在 Azure 虛擬機器中裝載的 VM 上運作，就像它在內部部署的運作方式。

SAP ASE HADR 不需要使用 Azure 內部負載平衡器，對 OS 層級叢集也沒有相依性，並且可在 Azure Windows 和 Linux VM 上運作。 如需 SAP ASE HADR 的詳細資訊，請參閱 [SAP ASE HADR 使用者指南](https://help.sap.com/viewer/efe56ad3cad0467d837c8ff1ac6ba75c/16.0.3.3/en-US/a6645e28bc2b1014b54b8815a64b87ba.html)。

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
根據一般的描述，SAP ASE 可執行檔應該位於或安裝於 VM 的根檔案系統 ( /sybase ) 上。 一般而言，大部分的 SAP ASE 系統和工具資料庫，SAP NetWeaver 工作負載都不常使用。 因此，系統和工具資料庫 (master、model、saptools、sybmgmtdb、sybsystemdb) 也可以保留於根檔案系統上。 

有一個例外狀況可能是暫存資料庫包含 SAP ASE 建立的所有工作資料表和暫存資料表，萬一某些 SAP ERP 和所有 BW 工作負載可能要求更高的資料量或 I/O 作業量，而其無法符合原始 VM 的 OS 磁碟，就會發生此情況。

根據用來安裝系統的 SAPInst/SWPM 版本，資料庫可能包含︰

* 安裝 SAP ASE 時建立單一的 SAP ASE tempdb
* 安裝 SAP ASE 時建立的 SAP ASE tempdb，以及 SAP 安裝常式所建立的其他 saptempdb
* 透過安裝 SAP ASE 來建立的 SAP ASE tempdb，以及手動建立 (例如依照 SAP 附註 [1752266]) 來符合 ERP/BW 特定 tempdb 需求的其他 tempdb

萬一發生特定的 ERP 或所有 BW 工作負載，就能基於效能因素，在個別的檔案系統上保留另外建立之 tempdb (透過 SWPM 或手動) 的 tempdb 裝置，該檔案系統可以利用單一 Azure 資料磁碟或橫跨多個 Azure 資料磁碟的 Linux RAID 來表示。 如果沒有其他 tempdb 存在，則建議您建立一個 (SAP 附註 [1752266])。

針對這類系統，應該針對另外建立的 tempdb 執行下列步驟︰

* 將第一個 tempdb 目錄移至 SAP 資料庫的第一個檔案系統
* 將 tempdb 目錄新增至每個包含 SAP 資料庫檔案系統的磁碟

這個組態讓 tempdb 所耗用的空間比系統磁碟機能夠提供的還多。 作為參考，您可以在內部部署執行的現有系統上檢查 tempdb 目錄的大小。 或者，這類組態可針對無法使用系統磁碟機來提供的 tempdb 啟用 IOPS 數目。 再次提醒，內部部署執行的系統可以用來監視 tempdb 的 I/O 工作負載。

永遠不要將任何 SAP ASE 目錄放置於 VM 的 /mnt 或 /mnt/resource 上。 這也適用於 tempdb，即使保留於 tempdb 中的物件只是暫時性的也一樣，因為 /mnt 或 /mnt/resource 是非永續性的預設 Azure VM 暫存空間。 如需有關 Azure VM 暫存空間的更多詳細資料，請參閱[這篇文章][virtual-machines-linux-how-to-attach-disk]

#### <a name="impact-of-database-compression"></a>資料庫壓縮的影響
在 I/O 頻寬可能變成限制因素的組態中，減少 IOPS 的每個量值可能都有助於延展您可以在類似 Azure 的 IaaS 案例中執行的工作負載。 因此，建議您在將現有的 SAP 資料庫上傳至 Azure 之前，確定已使用 SAP ASE 壓縮。

建議在上傳至 Azure 之前執行壓縮，如果尚未實作，可能是因為下列數種因素所導致：

* 降低要上傳至 Azure 的資料量
* 假設您可以使用功能更強大的硬體，其在內部部署中使用更多 CPU 或更高 I/O 頻寬或更少 I/O 延遲，則執行壓縮的持續時間較短
* 較小的資料庫大小可能會使磁碟配置的成本降低

資料壓縮和 LOB 壓縮可以在 Azure 虛擬機器上裝載的 VM 中運作，如同它在內部部署中的運作方式。 如需有關如何檢查在現有 SAP ASE 資料庫中是否已經使用壓縮的更多詳細資料，請參閱 SAP 附註 [1750510]。 如需有關資料庫壓縮的其他資訊，請參閱 SAP 附註 [2121797]。

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

取決於裝載 SAP 系統的 Azure 虛擬機器是否以及如何透過站對站、多重網站或 ExpressRoute (跨單位部署) 來連接，您需要確定 ICM 會使用完整的主機名稱，此名稱可在您嘗試從中開啟 DBACockpit 的電腦上加以解析。 請參閱 SAP 附註 [773830] 以了解 ICM 如何根據設定檔參數決定完整的主機名稱，並於必要時明確設定 icm/host_name_full 參數。

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
將 SAP ASE 部署至 Azure 時，必須檢閱您的備份方法。 即使系統不是生產系統，還是必須定期備份 SAP ASE 所裝載的 SAP 資料庫。 由於 Azure 儲存體會保留三個映像，因此在補償儲存體損毀方面，備份現在已變得較不重要。 維護適當備份和還原方案的主要原因是，您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原回到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 例如，您可以藉由還原備份，從 2 層 SAP 組態轉移到同一個系統的 3 層系統設定。

在 Azure 中備份和還原資料庫的運作方式，與內部部署中的運作方式一樣。 請參閱 SAP 附註︰

* [1588316]
* [1585981]

以取得建立傾印組態和排程備份的詳細資料。 根據您的策略和需求，您可以在其中一個現有的磁碟上將資料庫和記錄傾印設定至磁碟，或者新增其他磁碟以供備份使用。 為了減少在發生錯誤時產生資料遺失的風險，建議使用磁碟 (不會有任何資料庫目錄/檔案位於其中)。

除了資料壓縮與 LOB 壓縮之外，SAP ASE 也會提供備份壓縮。 若要讓資料庫和記錄傾印佔用較少的空間，建議使用備份壓縮。 如需詳細資訊，請參閱 SAP 附註 [1588316]。 如果您打算從 Azure 虛擬機器將備份或包含備份傾印的 VHD 下載至內部部署，則壓縮備份對於降低要傳輸的資料量來說也很重要。

不要使用 Azure VM 暫存空間 /mnt 或 /mnt/resource 做為資料庫或記錄傾印目的地。

#### <a name="performance-considerations-for-backupsrestores"></a>備份/還原的效能考量
如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 此外，備份壓縮所使用的 CPU 耗用量在最多有 8 個 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，讀取的整體輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (Linux 軟體 RAID、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用/相結合：

* 在多個掛接的磁碟上等量劃分備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 在 SAP ASE 層級上建立傾印組態，該層級會使用一個以上的目標目錄來寫入傾印

本指南的先前內容中已討論過在多個掛接的磁碟上等量劃分磁碟區的方式。 如需有關在 SAP ASE 傾印組態中使用多個目錄的詳細資訊，請參閱 sp_config_dump 預存程序的相關文件，此預存程序可用來在 [Sybase Infocenter](http://infocenter.sybase.com/help/index.jsp) 上建立傾印組態。

### <a name="disaster-recovery-with-azure-vms"></a>使用 Azure VM 的災害復原
#### <a name="data-replication-with-sap-sybase-replication-server"></a>利用 SAP Sybase 複寫伺服器的資料複寫
利用 SAP Sybase 複寫伺服器 (SRS)，SAP ASE 可提供暖待命的解決方案，以非同步方式將資料庫交易傳輸到遠端位置。 

安裝及操作 SRS 也可以在 Azure 虛擬機器中裝載的 VM 上運作，就像它在內部部署的運作方式。

目前不支援透過 SAP 複寫伺服器的 ASE HADR。 未來可能會進行測試並針對 Microsoft Azure 平台加以發行。

## <a name="specifics-to-oracle-database-on-windows"></a>Windows 上 Oracle 資料庫專屬的詳細資料
Oracle 支援 Oracle 軟體可在 Microsoft Windows HYPER-V 和 Azure 上執行。 如需關於 Windows Hyper-V 和 Azure 一般支援的詳細資訊，請查看︰<https://blogs.oracle.com/cloud/entry/oracle_and_microsoft_join_forces> 

遵循一般支援，也支援利用 Oracle 資料庫的 SAP 應用程式特定案例。 詳細資料已在文件的這個部分中命名。

### <a name="oracle-version-support"></a>Oracle 版本支援
在 SAP 附註 [2039619] 中，可找到 Azure 虛擬機器上的 Oracle 執行 SAP 時所支援的 Oracle 版本和對應的 OS 版本。

如需關於在 SAP ASE 上執行「SAP 商務套件」的一般資訊，請參閱 1DX：<https://www.sap.com/community/topic/oracle.html>

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 Oracle 組態指導方針
#### <a name="storage-configuration"></a>儲存體組態
只支援 Oracle 使用 NTFS 格式化磁碟的單一執行個體。 所有的資料庫檔案都必須儲存於以 VHD 或受控磁碟為基礎的 NTFS 檔案系統上。 這些磁碟會掛接到 Azure VM，並且以 Azure 分頁 Blob 儲存體 (<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) 或受控磁碟 (<https://docs.microsoft.com/azure/storage/storage-managed-disks-overview>) 為基礎。 任何類型的網路磁碟機或遠端共用 (例如 Azure 檔案服務)：

* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx> 
* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx>

就 Oracle 資料庫檔案而言，都「不」  支援使用！

使用以 Azure 分頁 BLOB 儲存體或受控磁碟為基礎的磁碟時，本文件的 [VM 和資料磁碟的快取][dbms-guide-2.1]及 [Microsoft Azure 儲存體][dbms-guide-2.3]章節中所做的陳述也適用於利用 Oracle 資料庫所做的部署。

如同先前在文件通用部分中所述，適用於 Azure 磁碟的 IOPS 輸送量上有配額存在。 確切的配額會根據使用的 VM 類型而定。 如需 VM 類型及其配額的清單，請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows]。

若要找出支援的 Azure VM 類型，請參閱 SAP 附註 [1928533]。

只要每個磁碟上目前的 IOPS 配額能滿足需求，就可以將所有 DB 檔案儲存於單一已掛接的磁碟上。 

如果需要更多 IOPS，建議使用 Windows 儲存集區 (僅適用於 Windows Server 2012 和更新版本) 或適用於 Windows 2008 R2 的 Windows 等量，透過多個已掛接的磁碟來建立一個大型邏輯裝置 (另請參閱本文件的[軟體 RAID][dbms-guide-2.2] 章節)。 這種方法可以簡化系統管理負荷來管理磁碟空間，並避免將檔案手動分散到多個已掛接的磁碟。

#### <a name="backup--restore"></a>備份 / 還原
針對備份 / 還原功能，利用與標準 Windows Server 作業系統和 Hyper-V 上所做的相同方式來支援 SAP BR*Tools for Oracle。 Oracle 復原管理員 (RMAN) 也支援備份至磁碟，以及從磁碟還原。

#### <a name="high-availability"></a>高可用性
基於高可用性和災害復原目的支援 Oracle Data Guard。 如需詳細資料，請參閱[這份][virtual-machines-windows-classic-configure-oracle-data-guard]文件。

#### <a name="other"></a>其他
所有其他一般領域 (例如「Azure 可用性設定組」或 SAP 監視) 也適用於使用 Oracle 資料庫來部署 VM 的情況，如本文件的前三章中所述。

## <a name="specifics-to-oracle-database-on-oracle-linux"></a>Oracle Linux 上 Oracle 資料庫專屬的詳細資料
Oracle 支援 Oracle 軟體可在 Microsoft Windows HYPER-V 和 Azure 上執行。 如需關於 Windows Hyper-V 和 Azure 一般支援的詳細資訊，請查看︰<https://blogs.oracle.com/cloud/entry/oracle_and_microsoft_join_forces> 

遵循一般支援，也支援利用 Oracle 資料庫的 SAP 應用程式特定案例。 詳細資料已在文件的這個部分中命名。

### <a name="oracle-version-support"></a>Oracle 版本支援
在 SAP 附註 [2039619] 中，可找到 Azure 虛擬機器上的 Oracle 執行 SAP 時所支援的 Oracle 版本和對應的 OS 版本。

如需關於在 SAP ASE 上執行「SAP 商務套件」的一般資訊，請參閱 1DX：<https://www.sap.com/community/topic/oracle.html>

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 Oracle 組態指導方針
#### <a name="storage-configuration"></a>儲存體組態
只支援 Oracle 使用 ext3、ext4 和 xfs 格式化磁碟的單一執行個體。 所有的資料庫檔案都必須儲存於以 VHD 或受控磁碟為基礎的這些檔案系統上。 這些磁碟會掛接到 Azure VM，並且以 Azure 分頁 Blob 儲存體 (<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) 或受控磁碟 (<https://docs.microsoft.com/azure/storage/storage-managed-disks-overview>) 為基礎。 任何類型的網路磁碟機或遠端共用 (例如 Azure 檔案服務)：

* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx> 
* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx>

就 Oracle 資料庫檔案而言，都「不」  支援使用！

使用以 Azure 分頁 BLOB 儲存體或受控磁碟為基礎的磁碟時，本文件的 [VM 和資料磁碟的快取][dbms-guide-2.1]及 [Microsoft Azure 儲存體][dbms-guide-2.3]章節中所做的陳述也適用於利用 Oracle 資料庫所做的部署。

如同先前在文件通用部分中所述，適用於 Azure 磁碟的 IOPS 輸送量上有配額存在。 確切的配額會根據使用的 VM 類型而定。 如需 VM 類型及其配額的清單，請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows]。

若要找出支援的 Azure VM 類型，請參閱 SAP 附註 [1928533]。

只要每個磁碟上目前的 IOPS 配額能滿足需求，就可以將所有 DB 檔案儲存於單一已掛接的磁碟上。 

如果需要更多 IOPS，建議使用 LVM (邏輯磁碟區管理員) 或 MDADM 透過多個已掛接的磁碟建立一個大型邏輯磁碟區。 另請參閱本文件的[軟體 RAID][dbms-guide-2.2] 一章。 這種方法可以簡化系統管理負荷來管理磁碟空間，並避免將檔案手動分散到多個已掛接的磁碟。

#### <a name="backup--restore"></a>備份 / 還原
針對備份 / 還原功能，利用與裸機和 Hyper-V 上所做的相同方式來支援 SAP BR*Tools for Oracle。 Oracle 復原管理員 (RMAN) 也支援備份至磁碟，以及從磁碟還原。

#### <a name="high-availability"></a>高可用性
基於高可用性和災害復原目的支援 Oracle Data Guard。 如需詳細資料，請參閱[這份][virtual-machines-windows-classic-configure-oracle-data-guard]文件。

#### <a name="other"></a>其他
所有其他一般領域 (例如「Azure 可用性設定組」或 SAP 監視) 也適用於使用 Oracle 資料庫來部署 VM 的情況，如本文件的前三章中所述。

## <a name="specifics-for-the-sap-maxdb-database-on-windows"></a>Windows 上 SAP MaxDB 資料庫專屬的詳細資料
### <a name="sap-maxdb-version-support"></a>SAP MaxDB 版本支援
SAP 目前支援 SAP MaxDB 版本 7.9，以便與 Azure 中 SAP NetWeaver 架構的產品搭配使用。 不論是適用於 SAP MaxDB 伺服器的所有更新，還是要與 SAP NetWeaver 架構的產品搭配使用的 JDBC 和 ODBC 驅動程式，都只會透過 SAP Service Marketplace 來提供，網址是：<https://support.sap.com/swdc>。
如需關於在 SAP MaxDB 上執行 SAP NetWeaver 的一般資訊，請參閱 <https://www.sap.com/community/topic/maxdb.html>。

### <a name="supported-microsoft-windows-versions-and-azure-vm-types-for-sap-maxdb-dbms"></a>針對 SAP MaxDB DBMS 支援的 Microsoft Windows 版本和 Azure VM 類型
若要尋找在 Azure 上支援 SAP MaxDB DBMS 的 Microsoft Windows 版本，請參閱︰

* [SAP 產品可用性對照表 (PAM)][sap-pam]
* SAP 附註 [1928533]

強烈建議使用最新版本的作業系統 Microsoft Windows，也就是 Microsoft Windows 2012 R2。

### <a name="available-sap-maxdb-documentation"></a>可用的 SAP MaxDB 文件
您可以在下列 SAP 附註中找到已更新的 SAP MaxDB 文件清單： [767598]

### <a name="sap-maxdb-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 SAP MaxDB 組態指導方針
#### <a name="b48cfe3b-48e9-4f5b-a783-1d29155bd573"></a>儲存體組態
適用於 SAP MaxDB 的 Azure 儲存體最佳做法是依照 [RDBMS 部署結構][dbms-guide-2]一章中所提到的一般建議。

> [!IMPORTANT]
> 如同其他資料庫，SAP MaxDB 也有資料和記錄檔。 不過，在 SAP MaxDB 術語中，正確的詞彙是「磁碟區」(不是「檔案」)。 例如，有 SAP MaxDB 資料磁碟區和記錄磁碟區。 請勿與作業系統磁碟區混淆。 
> 
> 

簡單地說，您必須︰

* 如果您使用 Azure 儲存體帳戶，如 [Microsoft Azure 儲存體][dbms-guide-2.3]一章所指定，將保有 SAP MaxDB 資料和記錄磁碟區 (也就是檔案) 的 Azure 儲存體帳戶設定為 [本機備援儲存體 (LRS)]。
* 將 SAP MaxDB 資料磁碟區 (也就是檔案) 的 IO 路徑，與記錄磁碟區 (也就是檔案) 的 IO 路徑分隔開來。 這表示 SAP MaxDB 資料磁碟區 (也就是檔案) 必須安裝於一個邏輯磁碟機上，而 SAP MaxDB 記錄磁碟區 (也就是檔案) 必須安裝於另一個邏輯磁碟機上。
* 如 [VM 和資料磁碟的快取][dbms-guide-2.1]一章所述，根據您是要針對 SAP MaxDB 資料還是記錄磁碟區 (也就是檔案) 使用檔案快取，以及您使用的是「Azure 標準儲存體」還是「Azure 進階儲存體」而定，為每個磁碟設定適當的快取類型。
* 只要每個磁碟上目前的 IOPS 配額可滿足需求，就能夠將所有資料磁碟區儲存於單一掛接的磁碟上，而且也會將所有資料庫記錄磁碟區儲存於另一個單一掛接的磁碟上。
* 如果需要更多 IOPS 和/或空間，建議使用 Microsoft Windows 儲存集區 (僅適用於 Microsoft Windows Server 2012 和更新版本) 或適用於 Microsoft Windows 2008 R2 的 Microsoft Windows 等量，透過多個已掛接的磁碟來建立一個大型邏輯裝置。 另請參閱本文件的[軟體 RAID][dbms-guide-2.2] 一章。 這種方法可以簡化系統管理負荷來管理磁碟空間，並避免將檔案手動分散到多個掛接的磁碟。
* 針對最高的 IOPS 需求，您可以使用 Azure 進階儲存體，這可以在 DS 系列和 GS 系列 VM 上使用。

![適用於 SAP MaxDB DBMS 之 Azure IaaS VM 的參考組態][dbms-guide-figure-600]

#### <a name="23c78d3b-ca5a-4e72-8a24-645d141a3f5d"></a>備份與還原
將 SAP MaxDB 部署至 Azure 時，您必須檢閱備份方法。 即使系統不是生產系統，還是必須定期備份 SAP MaxDB 所裝載的 SAP 資料庫。 由於 Azure 儲存體會保留三個映像，因此在保護系統以免發生儲存體失敗以及更重要的操作或系統管理失敗方面，備份現在已變得較不重要。 維護適當備份和還原方案的主要原因是，讓您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 例如，您可以藉由還原備份，從 2 層 SAP 組態轉移到同一個系統的 3 層系統設定。

在 Azure 中備份和還原資料庫的運作方式與針對內部部署系統所做的一樣，因此您可以使用標準的 SAP MaxDB 備份/還原工具，這些工具的說明位於 SAP 附註 [767598]中所列的其中一份 SAP MaxDB 文件內。 

#### <a name="77cd2fbb-307e-4cbf-a65f-745553f72d2c"></a>備份與還原的效能考量
如同裸機部署，備份和還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區的輸送量。 此外，備份壓縮所使用的 CPU 耗用量在最多有 8 個 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，整體的讀取輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (等量目錄、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用 (可能是在組合中)：

* 專門用來備份的個別磁碟區
* 在多個掛接的磁碟上等量劃分備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 有適用於下列各項的個別專用邏輯磁碟裝置︰
  * SAP MaxDB 備份磁碟區 (也就是檔案)
  * SAP MaxDB 資料磁碟區 (也就是檔案)
  * SAP MaxDB 記錄磁碟區 (也就是檔案)

關於在多個掛接的磁碟上等量劃分磁碟區，先前在本文件的[軟體 RAID][dbms-guide-2.2] 一章中已討論過。 

#### <a name="f77c1436-9ad8-44fb-a331-8671342de818"></a>其他
所有其他一般領域 (例如「Azure 可用性設定組」或 SAP 監視) 也適用於使用 SAP MaxDB 資料庫來部署 VM 的情況，如本文件的前三章中所述。
其他 SAP MaxDB 特定設定對 Azure VM 而言是在背景運作的，在 SAP 附註 [767598] 及下列 SAP 附註所列的不同文件中，都有提供這些設定的說明︰

* [826037] 
* [1139904]
* [1173395]

## <a name="specifics-for-sap-livecache-on-windows"></a>Windows 上 SAP liveCache 專屬的詳細資料
### <a name="sap-livecache-version-support"></a>SAP liveCache 版本支援
「Azure 虛擬機器」中支援的 SAP liveCache 最低版本是針對 **EhP 2 for SAP SCM 7.0** 和更新版本發行的 **SAP LC/LCAPPS 10.0 SP 25** (包括 **liveCache 7.9.08.31** 和 **LCA-Build 25**)。

### <a name="supported-microsoft-windows-versions-and-azure-vm-types-for-sap-livecache-dbms"></a>針對 SAP liveCache DBMS 支援的 Microsoft Windows 版本和 Azure VM 類型
若要尋找在 Azure 上支援 SAP liveCache 的 Microsoft Windows 版本，請參閱︰

* [SAP 產品可用性對照表 (PAM)][sap-pam]
* SAP 附註 [1928533]

強烈建議使用最新版本的作業系統 Microsoft Windows Server。 

### <a name="sap-livecache-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 SAP liveCache 組態指導方針
#### <a name="recommended-azure-vm-types"></a>建議的 Azure VM 類型
因為 SAP liveCache 是執行大量計算的應用程式，所以 RAM 與 CPU 的數量和速度會對 SAP liveCache 效能產生主要影響。 

以 SAP 所支援的 Azure VM 類型 (SAP 附註 [1928533]) 來說，配置給 VM 的所有虛擬 CPU 資源都是以 Hypervisor 的專用實體 CPU 資源為後盾。 不會產生過度佈建 (因此不會產生 CPU 資源競爭)。

同樣地，針對 SAP 支援的所有 Azure VM 執行個體類型，VM 記憶體會完全對應到實體記憶體 - 例如，不會使用過度佈建 (過度使用)。

從這個觀點來看，強烈建議使用新的 D 系列或 DS 系列 (搭配 Azure 進階儲存體) 的 Azure VM 類型，因為它們所擁有的處理器速度比 A 系列快上 60%。 針對最高的 RAM 與 CPU 負載，您可以使用 G 系列和 GS 系列 (搭配 Azure 進階儲存體) 的 VM，其中配備最新的 Intel?? Xeon?? 處理器 E5 v3 系列，具有兩倍的記憶體以及四倍的 D/DS 系列固態硬碟儲存體 (SSD)。

#### <a name="storage-configuration"></a>儲存體組態
由於 SAP liveCache 是以 SAP MaxDB 技術為基礎，因此[儲存體組態][dbms-guide-8.4.1]一章中針對 SAP MaxDB 提到的所有 Azure 儲存體最佳做法建議也適用於 SAP liveCache。 

#### <a name="dedicated-azure-vm-for-livecache"></a>liveCache 專用的 Azure VM
因為 SAP liveCache 會密集使用運算能力，所以針對可提高生產力的使用方式，強烈建議部署於專用的 Azure 虛擬機器上。 

![適用於具生產力使用案例的 liveCache 專用的 Azure VM][dbms-guide-figure-700]

#### <a name="backup-and-restore"></a>備份與還原
備份與還原 (包括效能考量) 已經在相關的 SAP MaxDB 章節[備份與還原][dbms-guide-8.4.2]和[備份與還原的效能考量][dbms-guide-8.4.3]中做過說明。 

#### <a name="other"></a>其他
所有其他一般領域已經在[這個][dbms-guide-8.4.4]相關的 SAP MaxDB 章節中做過說明。 

## <a name="specifics-for-the-sap-content-server-on-windows"></a>Windows 上 SAP 內容伺服器專用的詳細資料
SAP 內容伺服器是個別的伺服器架構元件，可以不同格式儲存內容，例如電子文件。 SAP 內容伺服器是透過技術開發來提供，可針對任何 SAP 應用程式跨應用程式加以使用。 它會安裝於不同的系統上。 典型的內容是來自 Knowledge Warehouse 的訓練材料和文件，或者源自 mySAP PLM 文件管理系統的技術繪圖。 

### <a name="sap-content-server-version-support"></a>SAP 內容伺服器版本支援
SAP 目前支援：

* **SAP 內容伺服器**版本 **6.50 (和更新版本)**
* **SAP MaxDB 版本 7.9**
* **Microsoft IIS (網際網路資訊伺服器) 版本 8.0 (和更新版本)**

強烈建議使用最新版的「SAP 內容伺服器」(在撰寫本文件時為 **6.50 SP4**)，以及最新版的 **Microsoft IIS 8.5**。 

請從 [SAP 產品可用性對照表 (PAM)][sap-pam] 中，查看支援的 SAP Content Server 與 Microsoft IIS 最新版本。

### <a name="supported-microsoft-windows-and-azure-vm-types-for-sap-content-server"></a>針對 SAP 內容伺服器支援的 Microsoft Windows 和 Azure VM 類型
若要找出在 Azure 上支援 SAP 內容伺服器的 Windows 版本，請參閱︰

* [SAP 產品可用性對照表 (PAM)][sap-pam]
* SAP 附註 [1928533]

強烈建議使用最新版本的 Microsoft Windows Server。

### <a name="sap-content-server-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 SAP 內容伺服器組態指導方針
#### <a name="storage-configuration"></a>儲存體組態
如果您將「SAP 內容伺服器」設定成將檔案儲存在 SAP MaxDB 資料庫中，則[儲存體組態][dbms-guide-8.4.1]一章中針對 SAP MaxDB 提到的所有 Azure 儲存體最佳做法建議也適用於「SAP 內容伺服器」案例。 

如果您設定 SAP 內容伺服器來將檔案儲存於檔案系統中，建議使用專用的邏輯磁碟機。 使用 Windows 儲存體空間可讓您一併增加邏輯磁碟大小和 IOPS 輸送量，如[軟體 RAID][dbms-guide-2.2] 一章所述。 

#### <a name="sap-content-server-location"></a>SAP 內容伺服器位置
SAP 內容伺服器必須部署於部署 SAP 系統的相同 Azure 區域和 Azure VNET 中。 您可以自行決定是否要在專用的 Azure VM 上或 SAP 系統執行所在的相同 VM 上部署 SAP 內容伺服器元件。 

![SAP 內容伺服器專用的 Azure VM][dbms-guide-figure-800]

#### <a name="sap-cache-server-location"></a>SAP 快取伺服器位置
SAP 快取伺服器是一個額外的伺服器架構元件，可提供在本機存取 (快取) 文件的權限。 SAP 快取伺服器會快取 SAP 內容伺服器的文件。 如果必須從不同位置擷取文件一次以上，這樣可將網路流量最佳化。 一般規則是 SAP 快取伺服器實際上必須接近可存取 SAP 快取伺服器的用戶端。 

您有兩個選擇：

1. **用戶端是後端 SAP 系統** - 如果已設定後端 SAP 系統來存取「SAP 內容伺服器」，則該 SAP 系統就是用戶端。 由於 SAP 系統和 SAP 內容伺服器都會部署於同一個 Azure 區域 (在相同的 Azure 資料中心)，所以它們實際上是彼此接近的。 因此，不需要有專用的 SAP 快取伺服器。 SAP UI 用戶端 (SAP GUI 或 Web 瀏覽器) 可直接存取 SAP 系統，而 SAP 系統會從 SAP 內容伺服器擷取文件。
2. **用戶端是內部部署的 Web 瀏覽器** - 可以將「SAP 內容伺服器」設定成供 Web 瀏覽器直接存取。 在此情況下，在內部部署執行的 Web 瀏覽器就是 SAP 內容伺服器的用戶端。 內部部署的資料中心與 Azure 資料中心位於不同的實體位置 (最好彼此接近)。 您的內部部署資料中心是透過 Azure 站對站 VPN 或 ExpressRoute 連接到 Azure。 雖然這兩個選項提供安全的 VPN 網路連線至 Azure，但站對站網路連接不會在內部部署的資料中心與 Azure 資料中心之間提供網路頻寬和延遲 SLA。 若要加快文件的存取，您可以執行下列其中一項︰
   1. 安裝內部部署的「SAP 快取伺服器」，使其靠近內部部署的 Web 瀏覽器 ([這張][dbms-guide-900-sap-cache-server-on-premises]圖中的選項)
   2. 設定 Azure ExpressRoute，提供內部部署的資料中心與 Azure 資料中心之間高速且低延遲的專用網路連接。

![安裝內部部署 SAP 快取伺服器的選項][dbms-guide-figure-900]
<a name="642f746c-e4d4-489d-bf63-73e80177a0a8"></a>

#### <a name="backup--restore"></a>備份 / 還原
如果您將「SAP 內容伺服器」設定成將檔案儲存在 SAP MaxDB 資料庫中，請參閱 SAP MaxDB 的[備份與還原][dbms-guide-8.4.2]一章及[備份與還原的效能考量][dbms-guide-8.4.3]一章中已提供的備份/還原程序和效能考量。 

如果您設定 SAP 內容伺服器來將檔案儲存於檔案系統中，有一個選項是針對文件所在的整個檔案結構執行手動備份/還原。 與 SAP MaxDB 備份/還原類似，基於備份目的，建議要有專用的磁碟區。 

#### <a name="other"></a>其他
其他 SAP 內容伺服器特定的設定對於 Azure VM 是顯而易見的，相關說明請見各種文件和 SAP 附註︰

* <https://service.sap.com/contentserver> 
* SAP 附註 [1619726]  

## <a name="specifics-to-ibm-db2-for-luw-on-windows"></a>Windows 上 IBM DB2 for LUW 專屬的詳細資料
使用 Microsoft Azure，您可以輕鬆地將目前在 IBM DB2 for Linux、UNIX 及 Windows (LUW) 上執行的 SAP 應用程式移轉至 Azure 虛擬機器。 透過 IBM DB2 for LUW 上的 SAP，系統管理員和開發人員仍然可以使用可在內部部署使用的相同開發和管理工具。
如需關於在 IBM DB2 for LUW 上執行「SAP 商務套件」的一般資訊，請參閱「SAP 社群網路」(SCN)，網址是：<https://www.sap.com/community/topic/db2-for-linux-unix-and-windows.html>。

如需有關 Azure 中 DB2 for LUW 上 SAP 的其他資訊和更新，請參閱 SAP 附註 [2233094]。 

### <a name="ibm-db2-for-linux-unix-and-windows-version-support"></a>IBM DB2 for Linux, UNIX, and Windows 支援版本
當 DB2 版本為 10.5 時，支援 Microsoft Azure 虛擬機器中 IBM DB2 for LUW 上的 SAP。

如需有關支援之 SAP 產品和 Azure VM 類型的資訊，請參閱 SAP 附註 [1928533]。

### <a name="ibm-db2-for-linux-unix-and-windows-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 IBM DB2 for Linux, UNIX, and Windows 組態指導方針
#### <a name="storage-configuration"></a>儲存體組態
所有的資料庫檔案都必須儲存於以直接連接磁碟為基礎的 NTFS 檔案系統上。 這些磁碟會掛接到 Azure VM，並且以 Azure 分頁 Blob 儲存體 (<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) 或受控磁碟 (<https://docs.microsoft.com/azure/storage/storage-managed-disks-overview>) 為基礎。 針對資料庫檔案，「不」  支援任何類型的網路磁碟機或遠端共用 (例如下列的 Azure 檔案服務)︰ 

* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx>
* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx>

如果您使用以 Azure 分頁 BLOB 儲存體或受控磁碟為基礎的磁碟，則本文件的 [RDBMS 部署結構][dbms-guide-2]一章中所做的陳述也適用於利用「IBM DB2 for LUW 資料庫」所做的部署。 

如同先前在文件通用部分中所述，適用於磁碟的 IOPS 輸送量上有配額存在。 確切的配額會根據使用的 VM 類型而定。 如需 VM 類型及其配額的清單，請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows]。

只要每個磁碟上目前有足夠的 IOPS 配額，就可以將所有資料庫檔案儲存於單一已掛接的磁碟上。 

針對效能考量，也請參閱 SAP 安裝指南中的＜適用於資料庫目錄的資料安全性和效能考量＞章節。

或者，您也可以使用「Windows 儲存集區」(僅適用於 Windows Server 2012 和更新版本) 或適用於 Windows 2008 R2 的 Windows 等量分割 (如本文件的[軟體 RAID][dbms-guide-2.2] 一章所述)，在多個磁碟上建立一個大型邏輯裝置。
如果磁碟包含適用於您 sapdata 和 saptmp 目錄的 DB2 儲存路徑，您必須將實體磁碟磁區大小指定為 512 KB。 使用 Windows 儲存集區時，您必須透過命令列介面，使用參數 `-LogicalSectorSizeDefault`，以手動方式建立儲存集區。 如需詳細資訊，請參閱 <https://technet.microsoft.com/itpro/powershell/windows/storage/new-storagepool>。

#### <a name="backuprestore"></a>備份/還原
針對 IBM DB2 for LUW 的備份/還原功能，是利用與標準 Windows Server 作業系統和 Hyper-V 上所做的相同方式來支援。

您必須確定您擁有恰當且有效的資料庫備份策略。 

如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 此外，備份壓縮所使用的 CPU 耗用量在最多有 8 個 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，讀取的整體輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (等量目錄、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用/相結合：

* 在多個磁碟上等量劃分備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 使用一個以上的目標目錄來寫入備份

#### <a name="high-availability-and-disaster-recovery"></a>高可用性和災害復原
不支援 Microsoft Cluster Server (MSCS)。

支援 DB2 高可用性災害復原 (HADR)。 如果 HA 組態的虛擬機器具有運作中的名稱解析，則 Azure 中的設定與內部部署中完成的任何設定並無任何差別。 不建議只依賴 IP 解析。

請勿將異地複寫使用於可儲存資料庫磁碟的儲存體帳戶。 如需詳細資訊，請參閱 [Microsoft Azure 儲存體][dbms-guide-2.3]一章和 [Azure VM 的相關高可用性和災害復原][dbms-guide-3]一章。

#### <a name="other"></a>其他
所有其他一般領域 (例如「Azure 可用性設定組」或 SAP 監視) 也適用於使用 IBM DB2 for LUW 來部署 VM 的情況，如本文件的前三章中所述。 

另請參閱[適用於 Azure 上 SAP 的一般 SQL Server 摘要][dbms-guide-5.8]。

## <a name="specifics-to-ibm-db2-for-luw-on-linux"></a>Linux 上 IBM DB2 for LUW 專屬的詳細資料
使用 Microsoft Azure，您可以輕鬆地將目前在 IBM DB2 for Linux、UNIX 及 Windows (LUW) 上執行的 SAP 應用程式移轉至 Azure 虛擬機器。 透過 IBM DB2 for LUW 上的 SAP，系統管理員和開發人員仍然可以使用可在內部部署使用的相同開發和管理工具。 如需關於在 IBM DB2 for LUW 上執行「SAP 商務套件」的一般資訊，請參閱「SAP 社群網路」(SCN)，網址是：<https://www.sap.com/community/topic/db2-for-linux-unix-and-windows.html>。

如需有關 Azure 中 DB2 for LUW 上 SAP 的其他資訊和更新，請參閱 SAP 附註 [2233094]。

### <a name="ibm-db2-for-linux-unix-and-windows-version-support"></a>IBM DB2 for Linux, UNIX, and Windows 支援版本
當 DB2 版本為 10.5 時，支援 Microsoft Azure 虛擬機器中 IBM DB2 for LUW 上的 SAP。

如需有關支援之 SAP 產品和 Azure VM 類型的資訊，請參閱 SAP 附註 [1928533]。

### <a name="ibm-db2-for-linux-unix-and-windows-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 IBM DB2 for Linux, UNIX, and Windows 組態指導方針
#### <a name="storage-configuration"></a>儲存體組態
所有的資料庫檔案都必須儲存於以直接連接磁碟為基礎的檔案系統上。 這些磁碟會掛接到 Azure VM，並且以 Azure 分頁 Blob 儲存體 (<https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>) 或受控磁碟 (<https://docs.microsoft.com/azure/storage/storage-managed-disks-overview>) 為基礎。 針對資料庫檔案，「不」  支援任何類型的網路磁碟機或遠端共用 (例如下列的 Azure 檔案服務)︰

* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx>
* <https://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx>

如果您使用以 Azure 分頁 BLOB 儲存體為基礎的磁碟，則本文件的 [RDBMS 部署結構][dbms-guide-2]一章中所做的陳述也適用於利用 IBM DB2 for LUW 資料庫所做的部署。

如同先前在文件通用部分中所述，適用於磁碟的 IOPS 輸送量上有配額存在。 確切的配額會根據使用的 VM 類型而定。 如需 VM 類型及其配額的清單，請參閱[這裡 (Linux)][virtual-machines-sizes-linux] 和[這裡 (Windows)][virtual-machines-sizes-windows]。

只要每個磁碟上目前有足夠的 IOPS 配額，就可以將所有資料庫檔案儲存於單一磁碟上。

針對效能考量，也請參閱 SAP 安裝指南中的＜適用於資料庫目錄的資料安全性和效能考量＞章節。

或者，您也可以使用 LVM (邏輯磁碟區管理員) 或 MDADM (如本文件的[軟體 RAID][dbms-guide-2.2] 一章所述)，在多個磁碟上建立一個大型邏輯裝置。
如果磁碟包含適用於您 sapdata 和 saptmp 目錄的 DB2 儲存路徑，您必須將實體磁碟磁區大小指定為 512 KB。

#### <a name="backuprestore"></a>備份/還原
對 IBM DB2 for LUW 的備份/還原功能的支援方式，與內部部署環境中標準 Linux 安裝上提供的支援方式相同。

您必須確定您擁有恰當且有效的資料庫備份策略。

如同裸機部署，備份/還原效能取決於可以平行讀取的磁碟區數目，以及這些磁碟區可能的輸送量。 此外，備份壓縮所使用的 CPU 耗用量在最多有 8 個 CPU 執行緒的 VM 上扮演重要的角色。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，讀取的整體輸送量就越小
* VM 中 CPU 執行緒數目越小，備份壓縮的影響就越嚴重
* 要寫入備份的目標 (等量目錄、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用/相結合：

* 在多個磁碟上等量劃分備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 使用一個以上的目標目錄來寫入備份

#### <a name="high-availability-and-disaster-recovery"></a>高可用性和災害復原
支援 DB2 高可用性災害復原 (HADR)。 如果 HA 組態的虛擬機器具有運作中的名稱解析，則 Azure 中的設定與內部部署中完成的任何設定並無任何差別。 不建議只依賴 IP 解析。

請勿將異地複寫使用於可儲存資料庫磁碟的儲存體帳戶。 如需詳細資訊，請參閱 [Microsoft Azure 儲存體][dbms-guide-2.3]一章和 [Azure VM 的相關高可用性和災害復原][dbms-guide-3]一章。

#### <a name="other"></a>其他
所有其他一般主題 (例如「Azure 可用性設定組」或 SAP 監視) 也適用於使用 IBM DB2 for LUW 來部署 VM 的情況，如本文件的前三章中所述。

另請參閱[適用於 Azure 上 SAP 的一般 SQL Server 摘要][dbms-guide-5.8]。

