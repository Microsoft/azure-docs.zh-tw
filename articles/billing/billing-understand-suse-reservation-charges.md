---
title: 了解 SUSE 方案折扣和使用方式 - Azure Reservations | Microsoft Docs
description: 了解 SUSE 方案折扣如何套用至虛擬機器上的 SUSE 軟體。
services: billing
documentationcenter: ''
author: yashesvi
manager: yashar
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/28/2018
ms.author: yashar
ms.openlocfilehash: 20fae715923da93799b2d0ac8adc2083d726a95a
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43310282"
---
# <a name="understand-how-the-suse-linux-enterprise-software-plan-discount-is-applied"></a>了解如何套用 SUSE Linux Enterprise 軟體方案折扣

在您購買 SUSE Linux 方案之後，折扣就會自動套用到符合保留的已部署 SUSE 虛擬機器 (VM)。 SUSE Linux 方案涵蓋在 Azure VM 上執行 SUSE 軟體的成本。

若要購買適當的 SUSE Linux 方案，您需要了解您執行哪些 SUSE VM，以及這些 VM 上的 vCPU 數目。 使用下列各節，可協助您從使用量 CSV 檔案中找出所要購買的方案。

## <a name="discount-applies-to-different-vm-sizes-with-instance-size-flexibility"></a>折扣適用於具有執行個體大小彈性的各種 VM 大小

如同保留的 VM 執行個體，購買 SUSE 方案時具有執行個體大小彈性。 這表示您的折扣甚至適用於當您部署具有不同 vCPU 計數的 VM 時。 折扣適用於軟體方案內的各種 VM 大小。

折扣金額取決於下列表格中所列的比例。 此比例會比較該群組中每個計量的相對使用量。 此比例取決於 VM vCPU。 使用比例值來計算有多少 VM 執行個體取得 SUSE Linux 方案折扣。

例如，如果您針對具有 3 或 4 個 vCPU 的 VM，購買適用於 SUSE Linux Enterprise Server for HPC Priority 的方案，則該保留的比例為 2。 此折扣涵蓋以下的 SUSE 軟體成本：

- 2 部已部署的 VM (具有 1 或 2 個 vCPU)、
- 1 部已部署的 VM (具有 3 或 4 個 vCPU)、
- 或 0.77 或大約 77% 的 VM (具有 5 個以上 vCPU)。

5 個以上 vCPU 的比例為 2.6。 因此，對於具有 5 個以上 vCPU 的 VM，其 SUSE 保留只涵蓋軟體成本的一部分 (大約是 77%)。

## <a name="understand-your-suse-vm-usage-before-buying-a-suse-linux-plan"></a>購買 SUSE Linux 方案前先了解您的 SUSE VM 使用量

下表會顯示您可以購買保留的軟體方案、其相關聯的使用量計量，以及每個計量的比例。

### <a name="suse-linux-enterprise-server-for-hpc-priority"></a>SUSE Linux Enterprise Server for HPC Priority

Azure 入口網站市集名稱：

- SLES 12 SP3 for HPC (Priority)

|SUSE VM | 計量識別碼| 比例| VM 大小範例|
| -------| ------------------------| --- |--- |
|SLES for HPC (1-2 個 vCPU)|e275a668-ce79-44e2-a659-f43443265e98|1|D2s_v3|
|SLES for HPC (3-4 個 vCPU)|e531e1c0-09c9-4d83-b7d0-a2c6741faa22|2|D4s_v3|
|SLES for HPC (5 個以上 vCPU)|4edcd5a5-8510-49a8-a9fc-c9721f501913|2.6|D8s_v3|

### <a name="suse-linux-enterprise-server-for-hpc-standard"></a>SUSE Linux Enterprise Server for HPC Standard

Azure 入口網站市集名稱：

- SLES 12 SP3 for HPC

|SUSE VM | 計量識別碼 | 比例|VM 大小範例|
| ------- | --- | ------------------------| --- | --- |
|SLES for HPC (1-2 個 vCPU) |8c94ad45-b93b-4772-aab1-ff92fcec6610|1|D2s_v3|
|SLES for HPC (3-4 個 vCPU)|4ed70d2d-e2bb-4dcd-b6fa-42da71861a1c|1.92308|D4s_v3|
|SLES for HPC (5 個以上 vCPU) |907a85de-024f-4dd6-969c-347d47a1bdff|2.92308|D8s_v3|

### <a name="suse-linux-enterprise-server-for-sap-priority"></a>SUSE Linux Enterprise Server for SAP Priority

Azure 入口網站市集名稱：

- SLES for SAP 15 (Priority)
- SLES for SAP 12 SP3 (Priority)
- SLES for SAP 12 SP2 (Priority)

|SUSE VM | 計量識別碼 | 比例|VM 大小範例|
| ------- |------------------------| --- | --- |
|SLES for SAP Priority (1-2 個 vCPU)|497fe0b6-fa3c-4e3d-a66b-836097244142|1|D2s_v3|
|SLES for SAP Priority (3-4 個 vCPU) |847887de-68ce-4adc-8a33-7a3f4133312f|2|D4s_v3|
|SLES for SAP Priority (5 個以上 vCPU) |18ae79cd-dfce-48c9-897b-ebd3053c6058|2.41176|D8s_v3|

### <a name="suse-linux-enterprise-server-priority"></a>SUSE Linux Enterprise Server Priority

Azure 入口網站市集名稱：

- SLES 15 (PRIORITY)
- SLES 12 SP3 (Priority)
- SLES 11 SP4 (Priority)

|SUSE VM | 計量識別碼 | 比例|VM 大小範例|
| ------- |------------------------| --- |--- |
|SLES 1 vCPU|462cd632-ec6b-4663-b79f-39715f4e8b38|1|B1ms|
|SLES 2-4 個 vCPU |924bee71-5eb8-424f-83ed-a58823c33908|2|D4s_v3|
|SLES 2-4 個 vCPU |60b3ae9d-e77a-46b2-9cdf-92fa87407969|2|D4s_v3|
|SLES 6 個 vCPU |e8862232-6131-4dbe-bde4-e2ae383afc6f|3||
|SLES 8 個 vCPU |e11331a8-fd32-4e71-b60e-4de2a818c67a|3.2|D8s_v3|
|SLES 12 個核心 vCPU |a5afd00d-d3ef-4bcd-8b42-f158b2799782|3.2||
|SLES 16 個 vCPU |bb21066f-fe46-46d3-8006-b326b1663e52|3.2| D16s_v3|
|SLES 20 個 vCPU |c5228804-1de6-4bd4-a61c-501d9003acc8|3.2| |
|SLES 24 個核心 vCPU |-005d-4075-ac11-822ccde9e8f6|3.2| ND24s|
|SLES 32 個 vCPU |180c1a0a-b0a5-4de3-a032-f92925a4bf90|3.2| D32s_v3|
|SLES 40 個核心 vCPU |a161d3d3-0592-4956-9b64-6829678b6506|3.2||
|SLES 64 個 vCPU |7f5a36ed-d5b5-4732-b6bb-837dbf0fb9d8|3.2| D64s_v3|
|SLES 72 個核心 vCPU |93329a72-24d7-4faa-93d9-203f367ed334|3.2|F72s_v2|
|SLES 96 個核心 vCPU |2018c3a8-ff13-41f8-b64d-9558c5206547|3.2||
|SLES 128 個核心 vCPU |ac27e4d7-44b5-4fee-bc1a-78ac5b4abaf7|3.2| M128ms|

### <a name="suse-linux-enterprise-server-standard"></a>SUSE Linux Enterprise Server Standard

Azure 入口網站市集名稱：

- SLES 15
- SLES 15 (Standard)
- SLES 12 SP3 (Standard)

|SUSE VM | 計量識別碼 | 比例|VM 大小範例|
| ------- |------------------------| --- |--- |
|SLES 1-2 個核心 vCPU |4b2fecfc-b110-4312-8f9d-807db1cb79ae|1|D2s_v3|
|SLES 3-4 個核心 vCPU |0c3ebb4c-db7d-4125-b45a-0534764d4bda|1.92308|D4s_v3|
|SLES 5 個以上 vCPU |7b349b65-d906-42e5-833f-b2af38513468|2.30769| D8s_v3|

## <a name="next-steps"></a>後續步驟

若要深入了解保留，請參閱下列文章：

- [什麼是 Azure Reservations？](billing-save-compute-costs-reservations.md)
- [使用 Azure Reservations 預付 SUSE 軟體方案](../virtual-machines/linux/prepay-suse-software-charges.md)
- [預付具有 Azure 保留 VM 執行個體的虛擬機器](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [管理 Azure 保留項目](billing-manage-reserved-vm-instance.md)
- [了解預付型方案訂用帳戶的保留項目使用量](billing-understand-reserved-instance-usage.md)
- [了解 Enterprise 註冊的保留項目使用量](billing-understand-reserved-instance-usage-ea.md)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。