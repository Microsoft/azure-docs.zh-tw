### <a name="open-tcp-ports-in-the-windows-firewall-for-the-default-instance-of-the-database-engine"></a>在 Windows 防火牆中為 Database Engine 的預設執行個體開啟 TCP 連接埠
1. 使用遠端桌面連線到虛擬機器。 如需連線到 VM 的詳細指示，請參閱[使用遠端桌面開啟 SQL VM](../articles/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision.md#remotedesktop)。
2. 登入後，在 [開始] 畫面中輸入 **WF.msc**，然後按 ENTER 鍵。
   
    ![啟動防火牆程式](./media/virtual-machines-sql-server-connection-steps/12Open-WF.png)
3. 在 [具有進階安全性的 Windows 防火牆] 的左窗格中，以滑鼠右鍵按一下 [輸入規則]，然後在動作窗格中按一下 [新增規則]。
   
    ![新增規則](./media/virtual-machines-sql-server-connection-steps/13New-FW-Rule.png)
4. 在 [新增輸入規則精靈] 對話方塊的 [規則類型] 下方， 選取 [連接埠]，然後按 [下一步]。
5. 在 [通訊協定及連接埠] 對話方塊中，使用預設的 [TCP]。 接著在 [特定本機連接埠] 方塊中，輸入 Database Engine 執行個體的連接埠號碼 (預設執行個體的連接埠號碼 **1433**，或您在端點步驟中選擇的私人連接埠號碼)。
   
    ![TCP 連接埠 1433](./media/virtual-machines-sql-server-connection-steps/14Port-1433.png)
6. 按 [下一步] 。
7. 在 [動作] 對話方塊中選取 [允許連線]，然後按 [下一步]。
   
    **安全性注意事項：** 選取 [僅允許安全連線] 可提供額外的安全性。 如果您想要在環境中設定額外的安全性選項，請選取此選項。
   
    ![允許連線](./media/virtual-machines-sql-server-connection-steps/15Allow-Connection.png)
8. 在 [設定檔] 對話方塊中，依序選取 [公用]、[私人]，然後 [網域]。 然後按 [下一步] 。
   
    **安全性注意事項：** 選取 [公用] 可允許透過網際網路存取。 請盡可能選取較具有限制性的設定檔。
   
    ![公用設定檔](./media/virtual-machines-sql-server-connection-steps/16Public-Private-Domain-Profile.png)
9. 在 [名稱] 對話方塊中輸入此規則的名稱和描述，然後按一下 [完成]。
   
    ![規則名稱](./media/virtual-machines-sql-server-connection-steps/17Rule-Name.png)

視需要為其他元件開啟額外的連接埠。 如需詳細資訊，請參閱 [設定 Windows 防火牆以允許 SQL Server 存取](http://msdn.microsoft.com/library/cc646023.aspx)。

### <a name="configure-sql-server-to-listen-on-the-tcp-protocol"></a>設定 SQL Server 以接聽 TCP 通訊協定

[!INCLUDE [Enable TCP](virtual-machines-sql-server-connection-tcp-protocol.md)]

### <a name="configure-sql-server-for-mixed-mode-authentication"></a>設定 SQL Server 以進行混合模式驗證
SQL Server Database Engine 須有網域環境才能使用 Windows 驗證。 若要從另一部電腦連接 Database Engine，請設定 SQL Server 以進行混合模式驗證。 混合模式驗證可允許 SQL Server 驗證和 Windows 驗證。

> [!NOTE]
> 如果您已經設定具有已設定網域環境的 Azure 虛擬網路，可能就不需要設定混合模式驗證。
> 
> 

1. 連接到虛擬機器時，在 [開始] 頁面上輸入 **SQL Server Management Studio** ，然後按一下選取的圖示。
   
    當您首次開啟 Management Studio 時，它必須建立使用者 Management Studio 環境。 這可能需要花費幾分鐘的時間。
2. Management Studio 會出現 [連接到伺服器] 對話方塊。 在 [伺服器名稱] 方塊中，輸入虛擬機器的名稱，以使用 [物件總管] 連接到 Database Engine (除了虛擬機器名稱之外，您也可以使用 **(local)** 或單一句點做為 [伺服器名稱])。 選取 [Windows 驗證]，並保留 [使用者名稱] 方塊中的 ***your_VM_name*\your_local_administrator**。 按一下 [ **連接**]。
   
    ![連線到伺服器](./media/virtual-machines-sql-server-connection-steps/19Connect-to-Server.png)
3. 在 SQL Server Management Studio 物件總管中，以滑鼠右鍵按一下 SQL Server 執行個體的名稱 (虛擬機器名稱)，然後按一下 [屬性] 。
   
    ![伺服器屬性](./media/virtual-machines-sql-server-connection-steps/20Server-Properties.png)
4. 在 [安全性] 頁面的 [伺服器驗證] 下方，選取 [SQL Server 及 Windows 驗證模式]，然後按一下 [確定]。
   
    ![選取驗證模式](./media/virtual-machines-sql-server-connection-steps/21Mixed-Mode.png)
5. 在 [SQL Server Management Studio] 對話方塊中，按一下 [確定]  以確認重新啟動 SQL Server 的需求。
6. 在物件總管中，以滑鼠右鍵按一下伺服器，然後按一下 [重新啟動]。 (如果 SQL Server Agent 處於執行狀態，您也必須將其重新啟動。)
   
    ![重新啟動](./media/virtual-machines-sql-server-connection-steps/22Restart2.png)
7. 在 [SQL Server Management Studio] 對話方塊中，按一下 [ **是** ] 以同意重新啟動 SQL Server。

### <a name="create-sql-server-authentication-logins"></a>建立 SQL Server 驗證登入
若要從另一部電腦連接 Database Engine，您至少必須建立一個 SQL Server 驗證登入。

1. 在 SQL Server Management Studio 物件總管中，展開要建立新登入之伺服器執行個體的資料夾。
2. 以滑鼠右鍵按一下 [安全性] 資料夾，指向 [新增]，然後選取 [登入...]。
   
    ![新增登入](./media/virtual-machines-sql-server-connection-steps/23New-Login.png)
3. 在 [登入 - 新增] 對話方塊的 [一般] 頁面中，於 [登入名稱] 方塊輸入新使用者的名稱。
4. 選取 [SQL Server 驗證] 。
5. 在 [密碼]  方塊中，輸入新使用者的密碼。 在 [確認密碼]  方塊中再次輸入密碼。
6. 選取所需的密碼強制選項 ([強制執行密碼原則]、[強制執行密碼逾期]，以及 [使用者必須在下次登入時變更密碼])。 如果您是自己使用此登入，就不需要在下次登入時變更密碼。
7. 在 [預設資料庫] 清單中選取登入的預設資料庫。 [master] 是此選項的預設值。 如果您尚未建立使用者資料庫，請保留 [master] 的設定。
   
    ![登入屬性](./media/virtual-machines-sql-server-connection-steps/24Test-Login.png)
8. 如果您正在建立第一個登入，也許會想要將此登入指定為 SQL Server 系統管理員。 如果是的話，請在 [伺服器角色] 頁面中勾選 [系統管理員 (sysadmin)]。
   
   > [!NOTE]
   > 系統管理員 (sysadmin) 固定伺服器角色的成員擁有 Database Engine 的完整控制權限。 因此請謹慎限制此角色的成員資格。
   > 
   > 
   
   ![系統管理員 (sysadmin)](./media/virtual-machines-sql-server-connection-steps/25sysadmin.png)
9. 按一下 [確定]。

如需 SQL Server 登入的詳細資訊，請參閱 [建立登入](http://msdn.microsoft.com/library/aa337562.aspx)。

