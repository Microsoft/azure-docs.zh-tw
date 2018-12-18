### <a name="prepare-for-a-push-installation-on-a-windows-computer"></a>準備在 Windows 電腦上推送安裝

1. 確定 Windows 電腦與處理序伺服器之間有網路連線。
1. 建立可供處理序伺服器存取電腦的帳戶。 帳戶應該具有本機或網域系統管理員權限。 此帳戶僅適用於推送安裝和代理程式更新。

   > [!NOTE]
   > 如果您未使用網域帳戶，請停用本機電腦上的遠端使用者存取控制。 若要停用遠端使用者存取控制，請在 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System 登錄機碼之下，新增 DWORD：**LocalAccountTokenFilterPolicy**。 將值設定為 **1**。 此工作的做法是在命令提示字元執行下列命令︰  
   `REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1`
   >
   >
1. 在您要保護之電腦的 Windows 防火牆中，選取 [允許應用程式或功能通過防火牆]。 啟用 [檔案及印表機共用] 和 [Windows Management Instrumentation (WMI)]。 針對隸屬於網域的電腦，您可以利用群組原則物件 (GPO) 進行防火牆設定。

   ![防火牆設定](./media/site-recovery-prepare-push-install-mob-svc-win/mobility1.png)

1. 新增您在 CSPSConfigtool 中建立的帳戶。 請遵循下列步驟：

    a. 登入您的組態伺服器。

    b. 開啟 **cspsconfigtool.exe**。 它會是桌面上可用的捷徑，位於 %ProgramData%\home\svsystems\bin 資料夾中。

    c. 在 [管理帳戶] 索引標籤上，選取 [新增帳戶]。

    d. 加入您所建立的帳戶。

    e. 輸入您為電腦啟用複寫時使用的認證。
