---
title: Azure VM 上的 SAP MaxDB、liveCache 與內容伺服器部署 | Microsoft Docs
description: Azure 上的 SAP MaxDB、liveCache 與內容伺服器部署
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
ms.openlocfilehash: 408d43f07179f9f18c05f22fdd4ea36a3a90cb49
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39075163"
---
# <a name="sap-maxdb-livecache-and-content-server-deployment-on-azure-vms"></a>Azure VM 上的 SAP MaxDB、liveCache 與內容伺服器部署

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
[dbms-guide-figure-900]:media/dbms_maxdb_deployment_guide/900-sap-cache-server-on-premises.png

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




本文涵蓋在 Azure IaaS 中部署 MaxDB、liveCache 與內容伺服器時需考量的幾個領域。 在閱讀本文件之前，您應該先參閱[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)文件，以及 [Azure 上的 SAP 工作負載文件](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)中的其他指南。 

## <a name="specifics-for-the-sap-maxdb-deployments-on-windows"></a>Windows 上 SAP MaxDB 部署專屬的詳細資料
### <a name="sap-maxdb-version-support-on-azure"></a>Azure 上的 SAP MaxDB 版本支援
SAP 目前支援 SAP MaxDB 7.9 版或更新版本，以便與 Azure 中 SAP NetWeaver 架構的產品搭配使用。 不論是適用於 SAP MaxDB 伺服器的所有更新，還是要與 SAP NetWeaver 架構的產品搭配使用的 JDBC 和 ODBC 驅動程式，都只會透過 SAP Service Marketplace 來提供，網址是：<https://support.sap.com/swdc>。
如需關於在 SAP MaxDB 上執行 SAP NetWeaver 的一般資訊，請參閱 <https://www.sap.com/community/topic/maxdb.html>。

### <a name="supported-microsoft-windows-versions-and-azure-vm-types-for-sap-maxdb-dbms"></a>針對 SAP MaxDB DBMS 支援的 Microsoft Windows 版本和 Azure VM 類型
若要尋找在 Azure 上支援 SAP MaxDB DBMS 的 Microsoft Windows 版本，請參閱︰

* [SAP 產品可用性對照表 (PAM)][sap-pam]
* SAP 附註 [1928533]

強烈建議使用最新版本的 Microsoft Windows 作業系統，也就是 Microsoft Windows 2016。

### <a name="available-sap-maxdb-documentation-for-maxdb"></a>適用於 MaxDB 的可用 SAP MaxDB 文件
您可以在下列 SAP 附註中找到已更新的 SAP MaxDB 文件清單： [767598]

### <a name="sap-maxdb-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 SAP MaxDB 組態指導方針
#### <a name="b48cfe3b-48e9-4f5b-a783-1d29155bd573"></a>儲存體組態
適用於 SAP MaxDB 的 Azure 儲存體最佳做法是依照 [RDBMS 部署結構][dbms-guide-2]一章中所提到的一般建議。

> [!IMPORTANT]
> 如同其他資料庫，SAP MaxDB 也有資料和記錄檔。 不過，在 SAP MaxDB 術語中，正確的詞彙是「磁碟區」(不是「檔案」)。 例如，有 SAP MaxDB 資料磁碟區和記錄磁碟區。 請勿與作業系統磁碟區混淆。 
> 
> 

簡單地說，您必須︰

* 如果您使用 Azure 儲存體帳戶，請依[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中的指示，將存放 SAP MaxDB 資料和記錄磁碟區 (資料和記錄檔) 的 Azure 儲存體帳戶設定為 [本地備援儲存體 (LRS)]。
* 將 SAP MaxDB 資料磁碟區 (資料檔) 的 IO 路徑，與記錄磁碟區 (記錄檔) 的 IO 路徑分隔開來。 這表示 SAP MaxDB 資料磁碟區 (資料檔) 必須安裝於一個邏輯磁碟機上，而 SAP MaxDB 記錄磁碟區 (記錄檔) 則須安裝於另一個邏輯磁碟機上。
* 請依[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中的指示，根據您是否要針對 SAP MaxDB 資料或記錄磁碟區 (資料和記錄檔) 使用磁碟，以及是使用 Azure 標準儲存體或 Azure 進階儲存體，為每個磁碟設定適當的快取類型。
* 只要每個磁碟上目前的 IOPS 配額可滿足需求，就能夠將所有資料磁碟區儲存於單一掛接的磁碟上，而且也會將所有資料庫記錄磁碟區儲存於另一個單一掛接的磁碟上。
* 如果需要更多 IOPS 和/或空間，建議使用 Microsoft Windows 儲存集區 (僅適用於 Microsoft Windows Server 2012 和更新版本) ，在多個已掛接的磁碟上建立一個大型邏輯裝置。 如需詳細資料，也請參閱[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)。 這種方法可以簡化系統管理負荷來管理磁碟空間，並避免將檔案手動分散到多個掛接的磁碟。
* 強烈建議針對 MaxDB 部署使用 Azure 進階儲存體。 

![適用於 SAP MaxDB DBMS 之 Azure IaaS VM 的參考組態](./media/dbms_maxdb_deployment_guide/Simple_disk_structure_maxdb.PNG)


#### <a name="23c78d3b-ca5a-4e72-8a24-645d141a3f5d"></a>備份與還原
將 SAP MaxDB 部署至 Azure 時，您必須檢閱備份方法。 即使系統不是生產系統，還是必須定期備份 SAP MaxDB 所裝載的 SAP 資料庫。 由於 Azure 儲存體會保留三個映像，因此在保護系統以免發生儲存體失敗以及更重要的操作或系統管理失敗方面，備份現在已變得較不重要。 維護適當備份和還原方案的主要原因是，讓您可以藉由提供時間點復原功能來補償邏輯/手動錯誤。 因此，目標是使用備份來將資料庫還原到某個時間點，或者藉由複製現有的資料庫，在 Azure 中使用備份來植入另一個系統。 

在 Azure 中備份和還原資料庫的運作方式與針對內部部署系統所做的一樣，因此您可以使用標準的 SAP MaxDB 備份/還原工具，這些工具的說明位於 SAP 附註 [767598]中所列的其中一份 SAP MaxDB 文件內。 

#### <a name="77cd2fbb-307e-4cbf-a65f-745553f72d2c"></a>備份與還原的效能考量
如同裸機部署，備份和還原效能取決於可以平行讀取的磁碟區數目，以及那些磁碟區的輸送量。 因此，您可以假設︰

* 用來儲存資料庫裝置的磁碟數目越少，整體的讀取輸送量就越小
* 要寫入備份的目標 (等量目錄、磁碟) 越少，輸送量就越低

若要增加要寫入的目標數目，根據您的需求，有兩個選項可以使用 (可能是在組合中)：

* 專門用來備份的個別磁碟區
* 在多個掛接的磁碟上等量劃分備份目標磁碟區，以改善該等量磁碟區上的 IOPS 輸送量
* 有適用於下列各項的個別專用邏輯磁碟裝置︰
  * SAP MaxDB 備份磁碟區 (也就是檔案)
  * SAP MaxDB 資料磁碟區 (也就是檔案)
  * SAP MaxDB 記錄磁碟區 (也就是檔案)

在多個已掛接的磁碟上劃分磁碟區的方式，已於稍早在[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中討論。 

#### <a name="f77c1436-9ad8-44fb-a331-8671342de818"></a>其他考量
所有其他一般領域 (例如 Azure 可用性設定組或 SAP 監視) 也同樣適用於使用 SAP MaxDB 資料庫部署 VM 的情況，如[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中所述  。
其他 SAP MaxDB 特定的設定針對 Azure VM 是透明的，並已詳述於 SAP 附註 [767598] 及下列 SAP 附註中所列的不同文件中︰

* [826037] 
* [1139904]
* [1173395]

## <a name="specifics-for-sap-livecache-deployments-on-windows"></a>Windows 上 SAP liveCache 部署專屬的詳細資料
### <a name="sap-livecache-version-support"></a>SAP liveCache 版本支援
Azure 虛擬機器中支援的 SAP liveCache 最低版本為針對 **EhP 2 for SAP SCM 7.0** 和更新版本所發行的 **SAP LC/LCAPPS 10.0 SP 25** (包括 **liveCache 7.9.08.31** 和 **LCA-Build 25**)。

### <a name="supported-microsoft-windows-versions-and-azure-vm-types-for-sap-livecache-dbms"></a>針對 SAP liveCache DBMS 支援的 Microsoft Windows 版本和 Azure VM 類型
若要尋找在 Azure 上支援 SAP liveCache 的 Microsoft Windows 版本，請參閱︰

* [SAP 產品可用性對照表 (PAM)][sap-pam]
* SAP 附註 [1928533]

強烈建議使用最新版本的作業系統 Microsoft Windows Server。 

### <a name="sap-livecache-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 SAP liveCache 組態指導方針
#### <a name="recommended-azure-vm-types-for-livecache"></a>適用於 liveCache 的建議 Azure VM 類型
因為 SAP liveCache 是執行大量計算的應用程式，所以 RAM 與 CPU 的數量和速度會對 SAP liveCache 效能產生主要影響。 

以 SAP 所支援的 Azure VM 類型 (SAP 附註 [1928533]) 來說，配置給 VM 的所有虛擬 CPU 資源都是以 Hypervisor 的專用實體 CPU 資源為後盾。 不會產生過度佈建 (因此不會產生 CPU 資源競爭)。

同樣地，針對 SAP 支援的所有 Azure VM 執行個體類型，VM 記憶體會完全對應到實體記憶體 (例如，不會使用過度佈建 (過度承諾))。

從這個觀點來看，強烈建議使用最新的 Dv2、Dv3、Ev3 及 M 系列 VM。 不同 VM 類型的選擇，取決於您針對 liveCache 所需的記憶體數量，以及所需的 CPU 資源。 和所有其他 DBMS 部署相同，建議您針對要求效能的磁碟區使用 Azure 進階儲存體。

#### <a name="storage-configuration-for-livecache-in-azure"></a>Azure 中針對 liveCache 的儲存體設定
由於 SAP liveCache 是以 SAP MaxDB 技術為基礎，因此本文件中針對 SAP MaxDB 所描述的所有 Azure 儲存體最佳做法建議，也都適用於 SAP liveCache。 

#### <a name="dedicated-azure-vm-for-livecache-scenario"></a>針對 liveCache 案例的專用 Azure VM
因為 SAP liveCache 會密集使用運算能力，所以針對可提高生產力的使用方式，強烈建議部署於專用的 Azure 虛擬機器上。 

![適用於具生產力使用案例的 liveCache 專用的 Azure VM](./media/dbms_maxdb_deployment_guide/700-livecach-prod.PNG)


#### <a name="backup-and-restore-for-livecache-in-azure"></a>Azure 中針對 liveCache 的備份與還原
備份與還原 (包括效能考量) 已詳述於本文中相關的 SAP MaxDB 章節。 

#### <a name="other-considerations"></a>其他考量
所有其他一般領域已詳述於相關的 SAP MaxDB 章節。 

## <a name="specifics-for-the-sap-content-server-deployment-on-windows-in-azure"></a>於 Windows 上在 Azure 中部署 SAP 內容伺服器專屬的詳細資料
SAP 內容伺服器是個別的伺服器架構元件，可以不同格式儲存內容，例如電子文件。 SAP 內容伺服器是透過技術開發來提供，可針對任何 SAP 應用程式跨應用程式加以使用。 它會安裝於不同的系統上。 典型的內容是來自 Knowledge Warehouse 的訓練材料和文件，或者源自 mySAP PLM 文件管理系統的技術繪圖。 

### <a name="sap-content-server-version-support-for-azure-vms"></a>適用於 Azure VM 的 SAP 內容伺服器版本支援
SAP 目前支援：

* **SAP 內容伺服器**版本 **6.50 (和更新版本)**
* **SAP MaxDB 版本 7.9**
* **Microsoft IIS (網際網路資訊伺服器) 版本 8.0 (和更新版本)**

強烈建議使用最新版的 SAP 內容伺服器，以及最新版的 **Microsoft IIS**。 

請從 [SAP 產品可用性對照表 (PAM)][sap-pam] 中，查看支援的 SAP Content Server 與 Microsoft IIS 最新版本。

### <a name="supported-microsoft-windows-and-azure-vm-types-for-sap-content-server"></a>針對 SAP 內容伺服器支援的 Microsoft Windows 和 Azure VM 類型
若要找出在 Azure 上支援 SAP 內容伺服器的 Windows 版本，請參閱︰

* [SAP 產品可用性對照表 (PAM)][sap-pam]
* SAP 附註 [1928533]

強烈建議使用最新版本的 Microsoft Windows Server。

### <a name="sap-content-server-configuration-guidelines-for-sap-installations-in-azure-vms"></a>在 Azure VM 中安裝 SAP 的 SAP 內容伺服器組態指導方針
#### <a name="storage-configuration-foir-content-server-in-azure"></a>Azure 中適用於內容伺服器的儲存體設定
如果您將 SAP 內容伺服器設定成將檔案儲存在 SAP MaxDB 資料庫中，則本文件中針對 SAP MaxDB 所描述的所有 Azure 儲存體最佳做法建議，也適用於 SAP 內容伺服器案例。 

如果您設定 SAP 內容伺服器來將檔案儲存於檔案系統中，建議使用專用的邏輯磁碟機。 使用 Windows 儲存體空間也可讓您增加邏輯磁碟大小和 IOPS 輸送量，如[適用於 SAP 工作負載的 Azure 虛擬機器 DBMS 部署考量](dbms_guide_general.md)中所述。 

#### <a name="sap-content-server-location"></a>SAP 內容伺服器位置
SAP 內容伺服器必須部署於部署 SAP 系統的相同 Azure 區域和 Azure VNET 中。 您可以自行決定是否要在專用的 Azure VM 上或 SAP 系統執行所在的相同 VM 上部署 SAP 內容伺服器元件。 

![SAP 內容伺服器專用的 Azure VM](./media/dbms_maxdb_deployment_guide/800-azure-vm-sap-content-server.png)


#### <a name="sap-cache-server-location"></a>SAP 快取伺服器位置
SAP 快取伺服器是一個額外的伺服器架構元件，可提供在本機存取 (快取) 文件的權限。 SAP 快取伺服器會快取 SAP 內容伺服器的文件。 如果必須從不同位置擷取文件一次以上，這樣可將網路流量最佳化。 一般規則是 SAP 快取伺服器實際上必須接近可存取 SAP 快取伺服器的用戶端。 

您有兩個選擇：

1. **用戶端是後端 SAP 系統** - 如果已設定後端 SAP 系統來存取「SAP 內容伺服器」，則該 SAP 系統就是用戶端。 由於 SAP 系統和 SAP 內容伺服器都會部署於同一個 Azure 區域 (在相同的 Azure 資料中心)，所以它們實際上是彼此接近的。 因此，不需要有專用的 SAP 快取伺服器。 SAP UI 用戶端 (SAP GUI 或 Web 瀏覽器) 可直接存取 SAP 系統，而 SAP 系統會從 SAP 內容伺服器擷取文件。
2. **用戶端是內部部署的 Web 瀏覽器** - 可以將「SAP 內容伺服器」設定成供 Web 瀏覽器直接存取。 在此情況下，在內部部署執行的 Web 瀏覽器就是 SAP 內容伺服器的用戶端。 內部部署的資料中心與 Azure 資料中心位於不同的實體位置 (最好彼此接近)。 您的內部部署資料中心是透過 Azure 站對站 VPN 或 ExpressRoute 連接到 Azure。 雖然這兩個選項提供安全的 VPN 網路連線至 Azure，但站對站網路連接不會在內部部署的資料中心與 Azure 資料中心之間提供網路頻寬和延遲 SLA。 若要加快文件的存取，您可以執行下列其中一項︰
   1. 安裝內部部署的「SAP 快取伺服器」，使其靠近內部部署的 Web 瀏覽器 ([這張][dbms-guide-900-sap-cache-server-on-premises]圖中的選項)
   2. 設定 Azure ExpressRoute，提供內部部署的資料中心與 Azure 資料中心之間高速且低延遲的專用網路連接。

![安裝內部部署 SAP 快取伺服器的選項](./media/dbms_maxdb_deployment_guide/900-sap-cache-server-on-premises.png)
<a name="642f746c-e4d4-489d-bf63-73e80177a0a8"></a>

#### <a name="backup--restore"></a>備份 / 還原
如果您將 SAP 內容伺服器設定成將檔案儲存在 SAP MaxDB 資料庫中，其備份/還原程序和效能考量已詳述於本文的 SAP MaxDB 章節。 

如果您設定 SAP 內容伺服器來將檔案儲存於檔案系統中，有一個選項是針對文件所在的整個檔案結構執行手動備份/還原。 與 SAP MaxDB 備份/還原類似，基於備份目的，建議要有專用的磁碟區。 

#### <a name="other"></a>其他
其他 SAP 內容伺服器特定的設定對於 Azure VM 是顯而易見的，相關說明請見各種文件和 SAP 附註︰

* <https://service.sap.com/contentserver> 
* SAP 附註 [1619726]  
