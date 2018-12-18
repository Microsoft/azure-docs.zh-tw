---
title: 開始在 Azure VM 上使用 SAP | Microsoft Docs
description: 了解在 Microsoft Azure 中的虛擬機器 (VM) 上執行的 SAP 解決方案
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ad8e5c75-0cf6-4564-ae62-ea1246b4e5f2
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/27/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 27fa69043797860c0a4acb1a128b45fe9e2b01c1
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43090689"
---
# <a name="using-azure-for-hosting-and-running-sap-workload-scenarios"></a>使用 Azure 來裝載和執行 SAP 工作負載案例
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
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
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
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md#subscription

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
[deploy-template-cli]:../../../resource-group-template-deploy.md
[deploy-template-portal]:../../../resource-group-template-deploy.md
[deploy-template-powershell]:../../../resource-group-template-deploy.md

[dr-guide-classic]:http://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[getting-started-dbms]:get-started.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]:get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]:get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c


[ha-guide-classic]:http://go.microsoft.com/fwlink/?LinkId=613056
[ha-guide]:sap-high-availability-guide.md
[ha-guide-get-started]:sap-high-availability-guide-start.md

[install-extension-cli]:virtual-machines-linux-enable-aem.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-azurermvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

[planning-guide]:planning-guide.md
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
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
[sap-pam]:https://support.sap.com/pam (SAP 產品可用性對照表)
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
[virtual-machines-azurerm-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md
[virtual-machines-linux-agent-user-guide]:../../linux/agent-user-guide.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../linux/agent-user-guide.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#capture-the-vm
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
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
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

藉由選擇 Microsoft Azure 作為符合 SAP 需求的雲端合作夥伴，您將能夠在可調整、符合規範且經企業證明的平台上，可靠地執行您的任務關鍵性 SAP 工作負載和案例。  Azure 不僅具備調整規模的能力，而且彈性十足，可以為您節省成本。 Microsoft 與 SAP 的合作讓您可以在 Azure 的多種開發/測試及生產案例中執行 SAP 應用程式，最重要是能夠獲得完整的支援。 從 SAP NetWeaver 到 SAP S4/HANA、SAP BI、Linux 到 Windows，乃至於 SAP HANA 到 SQL，全部含括在內。

除了在 Azure 上裝載具有不同 DBMS 的 SAP NetWeaver 案例外，您還可以在 Azure 上裝載其他不同的 SAP 工作負載案例，例如 SAP BI。 關於在 Azure 原生虛擬機器上部署 SAP NetWeaver 的文件，可於＜Azure 虛擬機器上的 SAP NetWeaver＞一節中找到。

Azure 擁有原生的 Azure 虛擬機器供應項目，這些供應項目的 CPU 和記憶體資源大小會日益增長，以應付運用 SAP HANA 的 SAP 工作負載。 如需這方面的詳細資訊，請查閱文件中＜Azure 虛擬機器上的 SAP HANA＞一節底下的內容。

Azure for SAP HANA 是獨一無二的供應項目，可讓 Azure 在競爭中顯得與眾不同。 為了讓涉及 SAP HANA 並裝載更嚴苛之記憶體和 CPU 資源的 SAP 案例能夠成真，Azure 可讓您使用客戶專用的裸機硬體來執行 SAP HANA 部署，以便應付最多需要 20 TB (可相應放大為 60 TB) 記憶體的 S/4HANA 或其他 SAP HANA 工作負載。 SAP HANA on Azure (大型執行個體) 的此一獨特 Azure 解決方案可讓您在專用的裸機硬體 (在原生 Azure 虛擬機器上裝載了 SAP 應用程式層或工作負載中介軟體層) 上執行 SAP HANA。 此解決方案記載於＜SAP HANA on Azure (大型執行個體)＞一節中的多份文件。   

在 Azure 中裝載 SAP 工作負載案例也會讓不同的 SAP 元件與 SAP SaaS (或 PaaS) 供應項目需要身分識別整合和單一登入 (使用 Azure Active Directory)。 這類使用 Azure Active Directory (AAD) 和 SAP 實體之整合和單一登入案例的清單會在＜AAD SAP 身分識別整合及單一登入＞一節中說明並加以記載。

## <a name="latest-changes"></a>最新變更

有關 Azure VM M128s 上 SAP HANA 相應放大的記載已新增到：

- [SAP HANA 在 Azure 上的基礎結構組態和作業](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)
- [單一 Azure 區域中的 SAP HANA 可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-one-region)


## <a name="sap-hana-on-sap-hana-on-azure-large-instances"></a>SAP HANA on Azure (大型執行個體) 上的 SAP HANA

### <a name="overview-and-architecture-of-sap-hana-on-azure-large-instances"></a>SAP HANA on Azure (大型執行個體) 的概觀與架構
標題：SAP HANA on Azure (大型執行個體) 的概觀與架構

摘要：本《架構與技術部署指南》提供可協助您在 Azure 中的新 SAP HANA on Azure (大型執行個體) 上部署 SAP 的資訊。 它不是涵蓋 SAP 解決方案特定設定的完整指南，但是是您在初始部署和持續進行的作業中相當實用的資訊。 不應該取代與 SAP HANA 安裝相關的 SAP 文件 (或是涵蓋該方面的許多＜SAP 支援附註＞)。 它提供概觀，並提供安裝 SAP HANA on Azure (大型執行個體) 的其他詳細資料。

更新日期：2017 年 10 月

[這裡可以找到本指南](hana-overview-architecture.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="infrastructure-and-connectivity-to-sap-hana-on-azure-large-instances"></a>SAP HANA on Azure (大型執行個體) 的基礎結構和連接
標題：SAP HANA on Azure (大型執行個體) 的基礎結構和連接

摘要︰SAP HANA on Azure (大型執行個體) 的購買在您與 Microsoft 企業帳戶小組之間完成之後，需要各種不同的網路組態以確保適當的連線。  本文概述必須與下列必要資訊共用的資訊。 本文概述必須收集哪些資訊以及必須執行哪些組態指令碼。

更新日期：2017 年 10 月

[這裡可以找到本指南](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="install-sap-hana-in-sap-hana-on-azure-large-instances"></a>在 SAP HANA on Azure (大型執行個體) 中安裝 SAP HANA
標題：在 SAP HANA on Azure (大型執行個體) 上安裝 SAP HANA

摘要︰本文件概述在您的 Azure 大型執行個體上安裝 SAP HANA 的安裝程序。

更新日期：2017 年 7 月

[這裡可以找到本指南](hana-installation.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="high-availability-and-disaster-recovery-of-sap-hana-on-azure-large-instances"></a>SAP HANA on Azure (大型執行個體) 的高可用性和災害復原
標題：SAP HANA on Azure (大型執行個體) 的高可用性和災害復原

摘要：高可用性 (HA) 和災害復原 (DR) 對執行具任務關鍵性的 SAP HANA on Azure (大型執行個體) 伺服器來說，是相當重要的層面。 與 SAP、您的系統整合者和/或 Microsoft 合作，以正確地架構並實作合適的 HA/DR 策略，是非常重要的。 您必須考量您的環境特定的復原點目標 (RPO) 和復原時間目標 (RTO) 等重要考量。  本文件說明啟用您慣用層級 HA 和 DR 的選項。

更新日期：2017 年 10 月

[可以在這裡找到這份文件](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="troubleshooting-and-monitoring-of-sap-hana-on-azure-large-instances"></a>針對 SAP HANA on Azure (大型執行個體) 進行疑難排解和監視
標題：針對 SAP HANA on Azure (大型執行個體) 進行疑難排解和監視

摘要︰本指南提供建立監視 SAP HANA on Azure 環境的有用資訊，以及其他疑難排解資訊。

更新日期：2017 年 10 月

[可以在這裡找到這份文件](troubleshooting-monitoring.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="sap-hana-on-azure-virtual-machines"></a>Azure 虛擬機器上的 SAP HANA

### <a name="getting-started-with-sap-hana-on-azure"></a>在 Azure 上開始使用 SAP HANA
標題：在 Azure VM 上手動安裝 SAP HANA 的快速入門指南

摘要：此快速入門指南可協助藉由手動安裝 SAP NetWeaver 7.5 和 SAP HANA SP12，在 Azure VM 上設定單一執行個體 SAP HANA 系統。 本指南假設讀者已熟悉 Azure IaaS 基本概念，像是如何透過 Azure 入口網站或 Powershell/CLI 部署虛擬機器或虛擬網路，包括使用 json 範本的選項。 此外還預期讀者已熟悉 SAP HANA、SAP NetWeaver 以及在內部部署進行安裝的方式。

更新日期：2017 年 6 月

[這裡可以找到本指南](hana-get-started.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="s4hana-sap-cal-deployment-on-azure"></a>Azure 上的 S/4HANA SAP CAL 部署
標題：在 Azure 上部署 SAP S/4HANA 或 BW/4HANA

摘要：此指南可協助示範如何使用 SAP 雲端設備庫，在 Azure 上部署 SAP S/4HANA。 SAP 雲端設備庫是 SAP 提供的服務，讓使用者能夠在 Azure 上部署 SAP 應用程式。 此指南會逐步說明部署。

更新日期：2017 年 6 月

[這裡可以找到本指南](cal-s4h.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="high-availability-of-sap-hana-in-azure-virtual-machines"></a>Azure 虛擬機器中 SAP HANA 的高可用性
標題：Azure 虛擬機器上 SAP HANA 的高可用性

摘要：此指南將引導您完成 SUSE 12 OS 與 SAP HANA 的高可用性組態，以容納具有自動容錯移轉的 HANA 系統複寫。 此指南僅適用 SUSE 和 Azure 虛擬機器。 此指南尚不適用 Red Hat 或裸機或私人雲端或其他非 Azure 公用雲端部署。

更新日期：2017 年 6 月

[這裡可以找到本指南](sap-hana-high-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="sap-hana-backup-overview-on-azure-vms"></a>Azure VM 上的 SAP HANA 備份概觀
標題：Azure 虛擬機器上的 SAP HANA 備份指南

摘要：此指南提供關於在 Azure 虛擬機器上執行 SAP HANA 之備份可能性的基本資訊。

更新日期：2017 年 3 月

[這裡可以找到本指南](sap-hana-backup-guide.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="sap-hana-file-level-backup-on-azure-vms"></a>Azure VM 上的 SAP HANA 檔案層級備份
標題：以儲存體快照集為基礎的 SAP HANA 備份

摘要：此指南提供在 Azure 虛擬機器上執行 SAP HANA 時，於 Azure VM 上使用以快照集為基礎的備份相關資訊。

更新日期：2017 年 3 月

[這裡可以找到本指南](sap-hana-backup-file-level.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)


### <a name="sap-hana-snapshot-based-backups-on-azure-vms"></a>在 Azure VM 上以 SAP HANA 快照集為基礎的備份
標題：檔案層級的 SAP HANA Azure 備份

摘要：此指南提供在 Azure 虛擬機器上執行 SAP HANA 時，使用 SAP HANA 檔案層級備份的相關資訊

更新日期：2017 年 3 月

[這裡可以找到本指南](sap-hana-backup-storage-snapshots.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)


## <a name="sap-netweaver-deployed-on-azure-virtual-machines"></a>在 Azure 虛擬機器上開發的 SAP NetWeaver

### <a name="deploy-sap-ides-system-on-windows-and-sql-server-through-sap-cal-on-azure"></a>透過 SAP CAL on Azure，在 Windows 和 SQL Server 上部署 SAP IDES 系統
標題：在 Microsoft Azure SUSE Linux VM 上測試 SAP NetWeaver

摘要：此文件說明如何在 Azure 上，使用 SAP 雲端設備庫，根據 Windows 和 SQL Server 來部署 SAP IDES 系統。 SAP 雲端設備庫是一種 SAP 服務，讓使用者能夠在 Azure 上部署 SAP 產品。 此文件會逐步說明 SAP IDES 系統的部署。 IDES 系統只是其他數十個可在 Microsoft Azure 上透過 SAP 雲端設備進行部署之應用程式的一個範例。

更新日期：2017 年 6 月

[這裡可以找到本指南](cal-ides-erp6-erp7-sp3-sql.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)


### <a name="quickstart-guide-for-netweaver-on-suse-linux-on-azure"></a>Azure 之 SUSE Linux 上的 NetWeaver 快速入門指南
標題：在 Microsoft Azure SUSE Linux VM 上測試 SAP NetWeaver

摘要：本文描述在 Microsoft Azure SUSE Linux 虛擬機器 (VM) 上執行 SAP NetWeaver 時應考量的各種事項。 在 Azure 的 SUSE Linux VM 上已正式支援 SAP NetWeaver。 如需有關 Linux 版本、SAP 核心版本的所有詳細資料及其他詳細資料，請參閱 SAP 附註 1928533＜Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型＞。

更新時間：2016 年 9 月

[這裡可以找到本指南](suse-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="3da0389e-708b-4e82-b2a2-e92f132df89c"></a>規劃和實作
標題：SAP NetWeaver 的 Azure 虛擬機器規劃和實作

摘要：如果您考慮在 Azure 虛擬機器中執行 SAP NetWeaver，那麼請將此文件當作入門指南。 此規劃和實作指南會協助您評估是否可將現有或已規劃的 SAP NetWeaver 架構系統部署到 Azure 虛擬機器環境。 其內容涵蓋多個 SAP NetWeaver 部署案例，並包含 Azure 特有的 SAP 組態。 本白皮書列出並描述若要執行混合式 SAP 環境，於 SAP/Azure 端所需的所有必要設定資訊。 另外也會說明為確保 IaaS 上的 SAP NetWeaver 架構系統擁有高可用性，您所能採取的措施。

更新日期：2017 年 6 月

[這裡可以找到本指南][planning-guide]

### <a name="high-availability-configurations-of-sap-netweaver-in-azure-vms"></a>在 Azure VM 中 SAP NetWeaver 的高可用性組態
標題：SAP NetWeaver 的 Azure 虛擬機器高可用性

摘要：在此文件中，我們會探討使用 Azure Resource Manager 部署模型，在 Azure 中部署高可用性 SAP 系統時可採取的步驟。 我們會引導您完成這些主要工作。 在文件中，我們會說明在 Azure VM 中執行時，如何能夠保護單一失敗點元件，例如進階商業應用程式程式設計 (ABAP) SAP Central Services (ASCS)/SAP Central Services (SCS) 和資料庫管理系統 (DBMS)，以及備援元件，例如 SAP 應用程式伺服器。 此文件中會示範及顯示在 Azure 的 Windows Server 容錯移轉叢集的叢集和 SUSE Linux Enterprise Server Cluster Framework 中安裝及設定高可用性 SAP 系統的逐步說明範例。

更新日期：2017 年 10 月

[這裡可以找到本指南][ha-guide-get-started]

### <a name="realizing-multi-sid-deployments-of-sap-netweaver-in-azure-vms"></a>在 Azure VM 中實現 SAP NetWeaver 的多 SID 部署
標題：建立 SAP NetWeaver 多 SID 組態

摘要：此文件是＜Azure VM 上的 SAP NetWeaver 高可用性＞一文的補充說明。 由於 Azure 中已在 2016 年 9 月引進的新功能，因此，能夠在一組 Azure VM 中部署多個 SAP NetWeaver ASCS/SCS 執行個體。 使用這類設定時，您可以減少部署來實現高可用性 SAP NetWeaver 組態所需的 VM 數目。 此指南說明這類多 SID 組態的設定。

更新日期：2016 年 12 月

[這裡可以找到本指南](high-availability-multi-sid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

### <a name="6aadadd2-76b5-46d8-8713-e8d63630e955"></a>在 Azure VM 中部署 SAP NetWeaver
標題：SAP NetWeaver 的 Azure 虛擬機器部署

摘要：本文件提供用來將 SAP NetWeaver 軟體部署到 Azure 虛擬機器的程序指引。 本白皮書將重點放在三個特定的部署案例，強調要啟用 Azure Monitoring Extensions for SAP，並包括 Azure Monitoring Extensions for SAP 的疑難排解建議。 本白皮書假設您已閱讀過規劃和實作指南。

更新日期：2017 年 6 月

[這裡可以找到本指南][deployment-guide]

### <a name="1343ffe1-8021-4ce6-a08d-3a1553a4db82"></a>DBMS 部署指南
標題：SAP NetWeaver 的 Azure 虛擬機器 DBMS 部署

摘要：本白皮書涵蓋應該與 SAP 搭配執行之 DBMS 系統的規劃和實作考量。 第一個部分會列出和提出一般考量。 本白皮書的後續部分與在 Azure 中部署 SAP 所支援的不同 DBMS 有關。 所提出的不同 DBMS 分別是 SQL Server、SAP ASE 和 Oracle。 在這幾個特定部分中，會探討當您在 Azure 上搭配這些 DBMS 來執行 SAP 系統時，所必須納入的考量。 本白皮書也會提出在搭配 SAP 應用程式使用時，Azure 上不同 DBMS 所支援的備份和高可用性方法等主題。

更新日期：2017 年 6 月

[這裡可以找到本指南][dbms-guide]

### <a name="using-azure-site-recovery-for-sap-workload"></a>針對 SAP 工作負載使用 Azure Site Recovery
標題：SAP NetWeaver：使用 Azure Site Recovery 建置災害復原解決方案

摘要：本文說明 Azure Site Recovery 能如何用以應對災害復原案例。 案例中使用 Azure 作為災害復原位置，應用於使用 Azure Site Recovery 服務的內部部屬 SAP 環境。 本文另又介紹 Aure 至 Azure (A2A) 災難復原案例，以及該案例如何以 Azure Site Recovery 進行管理。  

更新日期：2017 年 8 月

[這裡可以找到本指南](http://aka.ms/asr-sap)
