## <a name="scenario"></a>案例
建立具有單一 NIC 的 VM，並連接至虛擬網路。 VM 需要三個不同的「私人」IP 位址和兩個「公用」IP 位址。 IP 位址會指派給下列 IP 組態︰

* **IPConfig-1：** 指派「靜態」私人 IP 位址和「靜態」公用 IP 位址。
* **IPConfig-2：** 指派「靜態」私人 IP 位址和「靜態」公用 IP 位址。
* **IPConfig-3：** 指派「靜態」私人 IP 位址，不指派任何公用 IP 位址。
  
    ![多個 IP 位址](./media/virtual-network-multiple-ip-addresses-scenario/multiple-ipconfigs.png)

建立 NIC 時，IP 組態會與 NIC 產生關聯，而建立 VM 時，NIC 會連結至 VM。 此案例使用的 IP 位址類型只是舉例說明。 您可以指派您需要的任何 IP 位址和作業類型。

> [!NOTE]
> 雖然本文中的步驟會將所有 IP 組態指派給單一 NIC，您也可以指派多個 IP 組態給多個 NIC VM 中的任何 NIC。 閱讀[建立具有多個 NIC 的 VM](../articles/virtual-machines/windows/multiple-nics.md) 文章以了解如何建立具有多個 NIC 的 VM。