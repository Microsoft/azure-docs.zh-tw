---
author: manish-shukla01
ms.author: manshuk
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 08-03-2018
ms.openlocfilehash: 41216fe12e10f72f76043f1a8bc361b538259ac1
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39720737"
---
# <a name="virtual-machine-size-flexibility-with-reserved-vm-instances"></a>利用保留 VM 執行個體獲得虛擬機器大小彈性

利用已針對執行個體大小彈性進行最佳化的保留虛擬機器執行個體，您所購買的保留項目可適用於相同大小系列群組中的虛擬機器 (VM) 大小。 例如，如果您購買 DSv2 系列資料表 (例如，Standard_DS5_v2) 中所列 VM 大小的保留項目，則保留項目折扣可適用於該相同資料表中所列的其他四個大小：

- Standard_DS1_v2
- Standard_DS2_v2
- Standard_DS3_v2
- Standard_DS4_v2

但該保留項目折扣不適用於不同資料表中所列的 VM 大小，例如 DSv2 系列高記憶體資料表中的列出項目：Standard_DS11_v2、Standard_DS12_v2，依此類推。

在大小系列群組內，保留項目折扣所適用的 VM 數目取決於您購買保留項目時所挑選的 VM 大小。 此外，也取決於您執行中 VM 的大小。 下列資料表所列的比例資料行，會比較每個 VM 大小在該群組中的相對使用量。 請使用比例值來計算保留項目折扣適用於您執行中 VM 的程度。

## <a name="examples"></a>範例

下列範例使用 DSv2 系列資料表中的大小和比例。

 您購買大小為 Standard_DS4_v2 的保留 VM 執行個體，其相較於該系列中其他大小的比例 (相對使用量) 為 8。

- 案例 1：以 1 的比例執行八個 Standard_DS1_v2 大小的 VM。 保留項目折扣適用於這八個 VM 全部。
- 案例 2：以各為 2 的比例執行兩個 Standard_DS2_v2 大小的 VM。 此外，以 4 的比例執行一個 Standard_DS3_v2 大小的 VM。 總使用量是 2+2+4=8。 因此，保留項目折扣適用於這三個 VM 全部。
- 案例 3：以 16 的比例執行一個 Standard_DS5_v2。 保留項目折扣適用於該 VM 計算費用的一半。

下列各節說明在購買已針對執行個體大小彈性進行最佳化的保留 VM 執行個體時，相同大小系列群組中有哪些大小。

## <a name="b-series"></a>B 系列

| 大小 | 比例|
|---|---|
| Standard_B1s | 1 |
|Standard_B2s|4|

如需詳細資訊，請參閱 [B 系列高載虛擬機器大小](../articles/virtual-machines/windows/b-series-burstable.md)。

## <a name="b-series-high-memory"></a>B 系列高記憶體

| 大小 | 比例|
|---|---|
| Standard_B1ms |1|
|Standard_B2ms|4|
|Standard_B4ms|8|
|Standard_B8ms|16|

如需詳細資訊，請參閱 [B 系列高載虛擬機器大小](../articles/virtual-machines/windows/b-series-burstable.md)。

## <a name="d-series"></a>D 系列

| 大小 | 比例|
|---|---|
| 標準_D1|1|
|標準_D2|2|
|Standard_D3|4|
|標準_D4|8|

如需詳細資訊，請參閱[前幾代的虛擬機器大小](../articles/virtual-machines/windows/sizes-previous-gen.md)。

## <a name="d-series-high-memory"></a>D 系列高記憶體

| 大小 | 比例|
|---|---|
| 標準_D11|1|
|標準_D12|2|
|標準_D13|4|
|標準_D14|8|

如需詳細資訊，請參閱[前幾代的虛擬機器大小](../articles/virtual-machines/windows/sizes-previous-gen.md)。

## <a name="ds-series"></a>Ds 系列

| 大小 | 比例|
|---|---|
|Standard_DS1|1|
|Standard_DS2|2|
|Standard_DS3|4|
|Standard_DS4|8|

如需詳細資訊，請參閱[前幾代的虛擬機器大小](../articles/virtual-machines/windows/sizes-previous-gen.md)。

## <a name="ds-series-high-memory"></a>Ds 系列高記憶體

| 大小 | 比例|
|---|---|
|Standard_DS11|1|
|Standard_DS12|2|
|Standard_DS13|4|
|Standard_DS14|8|

如需詳細資訊，請參閱[前幾代的虛擬機器大小](../articles/virtual-machines/windows/sizes-previous-gen.md)。

## <a name="dsv2-series"></a>DSv2 系列

| 大小 | 比例|
|---|---|
|Standard_DS1_v2|1|
|Standard_DS2_v2|2|
|Standard_DS3_v2|4|
|Standard_DS4_v2|8|
|Standard_DS5_v2|16|

如需詳細資訊，請參閱[一般用途的虛擬機器大小](../articles/virtual-machines/windows/sizes-general.md#dv2-series)。

## <a name="dsv2-series-high-memory"></a>DSv2 系列高記憶體

| 大小 | 比例|
|---|---|
|Standard_DS11_v2|1|
|Standard_DS12_v2|2|
|Standard_DS13_v2|4|
|Standard_DS14_v2|8|
|Standard_DS15_v2|10|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#dsv2-series-11-15)。

## <a name="dsv3-series"></a>DSv3 系列

| 大小 | 比例|
|---|---|
|Standard_D2s_v3|1|
|Standard_D4s_v3|2|
|Standard_D8s_v3|4|
|Standard_D16s_v3|8|
|Standard_D32s_v3|16|
|Standard_D64s_v3|32|

如需詳細資訊，請參閱[一般用途的虛擬機器大小](../articles/virtual-machines/windows/sizes-general.md#dsv3-series-sup1sup)。

## <a name="dv2-series"></a>Dv2 系列

| 大小 | 比例|
|---|---|
|Standard_D1_v2|1|
|Standard_D2_v2|2|
|Standard_D3_v2|4|
|Standard_D4_v2|8|
|Standard_D5_v2|16|

如需詳細資訊，請參閱[一般用途的虛擬機器大小](../articles/virtual-machines/windows/sizes-general.md#dv2-series)。

## <a name="dv2-series-high-memory"></a>Dv2 系列高記憶體

| 大小 | 比例|
|---|---|
|Standard_D11_v2|1|
|Standard_D12_v2|2|
|Standard_D13_v2|4|
|Standard_D14_v2|8|
|Standard_D15_v2|10|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#dv2-series-11-15)。

## <a name="dv3-series"></a>Dv3 系列

| 大小 | 比例|
|---|---|
| Standard_D2_v3|1|
|Standard_D4_v3|2|
|Standard_D8_v3|4|
|Standard_D16_v3|8|
|Standard_D32_v3|16|
|Standard_D64_v3|32|

如需詳細資訊，請參閱[一般用途的虛擬機器大小](../articles/virtual-machines/windows/sizes-general.md#dv3-series-sup1sup)。

## <a name="esv3-series"></a>ESv3 系列

| 大小 | 比例|
|---|---|
| Standard_E2s_v3|1|
|Standard_E4s_v3|2|
|Standard_E8s_v3|4|
|Standard_E16s_v3|8|
|Standard_E32s_v3|16|
|Standard_E64s_v3|32|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#esv3-series)。

## <a name="ev3-series"></a>Ev3 系列

| 大小 | 比例|
|---|---|
| Standard_E2_v3|1|
|Standard_E4_v3|2|
|Standard_E8_v3|4|
|Standard_E16_v3|8|
|Standard_E32_v3|16|
|Standard_E64_v3|32|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#ev3-series)。

## <a name="f-series"></a>F 系列

| 大小 | 比例|
|---|---|
| Standard_F1|1|
|Standard_F2|2|
|Standard_F4|4|
|Standard_F8|8|
Standard_F16|16|

如需詳細資訊，請參閱[計算最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-compute.md#f-series)。

## <a name="fs-series"></a>FS 系列

| 大小 | 比例|
|---|---|
| Standard_F1s|1|
|Standard_F2s|2|
|Standard_F4s|4|
|Standard_F8s|8|
|Standard_F16s|16|

如需詳細資訊，請參閱[計算最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-compute.md#fs-series-sup1sup)。

## <a name="fsv2-series"></a>Fsv2 系列

| 大小 | 比例|
|---|---|
|Standard_F2s_v2|1|
|Standard_F4s_v2|2|
|Standard_F8s_v2|4|
|Standard_F16s_v2|8|
|Standard_F32s_v2|16|
|Standard_F64s_v2|32|
|Standard_F72s_v2|36|

如需詳細資訊，請參閱[計算最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-compute.md#fsv2-series-sup1sup)。

## <a name="h-series"></a>H 系列

| 大小 | 比例|
|---|---|
| Standard_H8|1|
|Standard_H16|2|

如需詳細資訊，請參閱[高效能運算 VM 大小](../articles/virtual-machines/windows/sizes-hpc.md)。

## <a name="h-series-high-memory"></a>H 系列高記憶體

| 大小 | 比例|
|---|---|
| Standard_H8m|1|
|Standard_H16m|2|

如需詳細資訊，請參閱[高效能運算 VM 大小](../articles/virtual-machines/windows/sizes-hpc.md)。

## <a name="ls-series"></a>Ls 系列

| 大小 | 比例|
|---|---|
| Standard_L4s|1|
|Standard_L8s|2|
|Standard_L16s|4|
|Standard_L32s|8|

如需詳細資訊，請參閱[儲存體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-storage.md)。

## <a name="m-series"></a>M 系列

| 大小 | 比例|
|---|---|
| Standard_M64s|1|
|Standard_M128s|2|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#m-series)。

## <a name="m-series-fractional"></a>M 系列小數

| 大小 | 比例|
|---|---|
| Standard_M16s|1|
|Standard_M32s|2|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#m-series)。

## <a name="m-series-fractional-high-memory"></a>M 系列小數高記憶體

| 大小 | 比例|
|---|---|
| Standard_M8ms|1|
|Standard_M16ms|2|
|Standard_M32ms|4|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#m-series)。

## <a name="m-series-fractional-large"></a>M 系列小數大型

| 大小 | 比例|
|---|---|
| Standard_M32ls|1|
|Standard_M64ls|2|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#m-series)。

## <a name="m-series-high-memory"></a>M 系列高記憶體

| 大小 | 比例|
|---|---|
| Standard_M64ms|1|
|Standard_M128ms|2|

如需詳細資訊，請參閱[記憶體最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-memory.md#m-series)。

## <a name="nc-series"></a>NC 系列

| 大小 | 比例|
|---|---|
| Standard_NC6|1|
|Standard_NC12|2|
|Standard_NC24|4|

如需詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../articles/virtual-machines/windows/sizes-gpu.md)。

## <a name="ncv2-series"></a>NCv2 系列

| 大小 | 比例|
|---|---|
| Standard_NC6s_v2|1|
|Standard_NC12s_v2|2|
|Standard_NC24s_v2|4|

如需詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../articles/virtual-machines/windows//sizes-gpu.md#ncv2-series)。

## <a name="ncv3-series"></a>NCv3 系列

| 大小 | 比例|
|---|---|
| Standard_NC6s_v3|1|
|Standard_NC12s_v3|2|
|Standard_NC24s_v3|4|

如需詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../articles/virtual-machines/windows//sizes-gpu.md#ncv3-series)。

## <a name="nd-series"></a>ND 系列

| 大小 | 比例|
|---|---|
| Standard_ND6s|1|
|Standard_ND12s|2|
|Standard_ND24s|4|

如需詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../articles/virtual-machines/windows//sizes-gpu.md#nd-series)。

## <a name="nv-series"></a>NV 系列

| 大小 | 比例|
|---|---|
| Standard_NV6|1|
|Standard_NV12|2|
|Standard_NV24|4|

如需詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../articles/virtual-machines/windows//sizes-gpu.md#nv-series)。


