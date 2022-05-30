# Lab2 - 使用原生用戶端連線至 VM 並且利用 Kerberos 驗證

# 建立 Azure VM ADDS

- 進入虛擬機器設定 Active Directory 網域服務 <br> 
	- 進入 Azure Portal，選擇虛擬機器 ADDS，使用 Bastion 連線 (isadmin/isadmin@123) <br>
	  ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds2.png "adds2")<br>
	- 開啟伺服器管理員 (Server Manager)，點選 Promote this server to a domain controller<br>
	  ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds3.png "adds3")<br>
	- 點選 Add a new forest，並輸入 Root domain name，此範例先設定 brianhsing.club，後續再做 AAD Connect 時，就不需要再另外設定 UPN 尾碼。<br>
  	  ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds4.png "adds4")<br>

	> **Tips.請不要使用結尾為「.local」的網域，此網域無法在虛擬網路內路由** <br>
	
	
	- 自行輸入 Directory Services Restore Mode 密碼<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds5.png "adds5")<br>
	- DNS Option 直接選擇下一步<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds6.png "adds6")<br>
	- Additional Option 直接選擇下一步，稍後登入會使用 NETBIOS\isadmin 帳號格式<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds7.png "adds7")<br>
	- Path 直接選擇下一步。如果是正式環境，建議將這三個資料夾與系統磁區分開<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds8.png "adds8")<br>
	- Review Option 下一步後，Prerequisites Check 頁面選擇 Install，等待安裝結束後，會自動開機。<br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-Virtual-Desktop/blob/master/Lab2/adds9.png "adds9")<br>

# 將 srv01 加入網域

- 確認 Azure Bastion 計費層級為標準，並且有勾選 Kerberos 驗證<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/kerberos.png "kerberos")<br>
- 將 srv01-nic 的 DNS Servers 設定為 172.31.10.4<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/nicdnsstting.png "nicdnsstting")<br>
- 將 vnet-hub 的 DNS Services 設定為 172.31.10.4<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/vnetdnsstting.png "vnetdnsstting")<br>
- 先使用 Azure Bastion 登入，並且將 srv01 加入至 brianhsing.club 網域中<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/adjoin.png "adjoin")<br>

# 使用原生用戶端連線至 VM

- 確認 Azure Bastion 計費層級為標準，並且勾選原生用戶端支援<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/nativetool.png "nativetool")<br>
- 使用 Azure CLI 登入，並且選擇包含 Azure Bastion 的訂用帳戶<br>
  ````
	az login
	az account list
	az account set --subscription "<subscription ID>"
  ````
- 使用 Azure CLI 將對應的參數輸入，即可開啟本機原生工具連線至虛擬機器<br>
  ````
  az network bastion rdp --name “<BastionName>" --resource-group "<ResourceGroupName>" --target-resource-id "<VMResourceId>"

  ````
- 其中 target-resource-id，可以在虛擬機器，屬性頁面的資源識別碼找到<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/nativetoolcli.png "nativetoolcli")<br>
- 輸入後即可看到遠端桌面連線的視窗跳出<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/rdp1.png "rdp1")<br>
- 點選連線後，請輸入 UPN (brian@brianhsing) 與密碼<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/rdp2.png "rdp2")<br>
- 成功後，點選是<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/rdp3.png "rdp3")<br>
- 成功登入畫面，並且為網域管理員身分<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/rdp4.png "rdp4")<br>


前往 [Lab3 - 使用 RDP 連線至 Linux VM](https://github.com/BrianHsing/Azure-Bastion/blob/main/lab3.md)<br>