### <a name="prepare-for-a-push-installation-on-a-linux-server"></a>準備在 Linux 伺服器上推送安裝

1. 確定 Linux 電腦與處理序伺服器之間有網路連線。
1. 建立可供處理序伺服器存取電腦的帳戶。 帳戶應該是來源 Linux 伺服器上的**根**使用者。 此帳戶僅適用於推送安裝和更新。
1. 檢查來源 Linux 伺服器上的 /etc/hosts 檔案，該檔案具有將本機主機名稱對應到所有網路介面卡相關聯之 IP 位址的項目。
1. 在您要複寫的電腦上安裝最新的 openssh、openssh-server 和 openssl 套件。
1. 請確定安全殼層 (SSH) 已啟用且正在連接埠 22 上執行。
1. 在 sshd_config 檔案中啟用 SFTP 子系統與密碼驗證。 請遵循下列步驟：

    a. 以 **root** 的身分登入。

    b. 在 **/etc/ssh/sshd_config** 檔案中，找到以 **PasswordAuthentication** 開頭這一行。

    c. 取消該行的註解，並將值變更為 **yes**。

    d. 找到以 **Subsystem** 開頭這一行，並取消其註解。

      ![Linux](./media/site-recovery-prepare-push-install-mob-svc-lin/mobility2.png)

    e. 重新啟動 **sshd** 服務。

1. 新增您在 CSPSConfigtool 中建立的帳戶。 請遵循下列步驟：

    a. 登入您的組態伺服器。

    b. 開啟 **cspsconfigtool.exe**。 它會是桌面上可用的捷徑，位於 %ProgramData%\home\svsystems\bin 資料夾中。

    c. 在 [管理帳戶] 索引標籤上，選取 [新增帳戶]。

    d. 加入您所建立的帳戶。

    d. 輸入您為電腦啟用複寫時使用的認證。
