* 在 [新增 vCenter] 中，指定 vSphere 主機或 vCenter 伺服器的易記名稱，然後指定伺服器的 IP 位址或 FQDN。 保留連接埠為 443，除非您的 VMware 伺服器設定為在不同連接埠上接聽要求。 選取將用於連接至 VMware vCenter 或 vSphere ESXi 伺服器的帳戶。 按一下 [確定]。

    ![VMware](./media/site-recovery-add-vcenter/vmware-server.png)

   > [!NOTE]
   > 如果您要以沒有 VMware vCenter 或主機伺服器系統管理員權限的帳戶新增 VMware vSphere 伺服器或 vSphere 主機，確定帳戶已啟用這些權限：資料中心、資料存放區、資料夾、主機、網路、資源、虛擬機器和 vSphere 分散式交換器。 此外，VMware vCenter 伺服器需要啟用儲存體檢視權限。
