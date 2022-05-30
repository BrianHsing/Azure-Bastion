# Lab3 - 使用 RDP 連線至 Linux VM

## 安裝 Ubuntu 桌面體驗

- 先透過 Azure Bastion 連線至 ubuntu1804<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/ssh1.png "ssh1")<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/ssh2.png "ssh2")<br>
- 依序更新 apt、安裝 tasksel、ubuntu-desktop、xrdp 與設定防火牆允許規則<br>
  ````
    sudo apt update
    sudo apt install -y tasksel 
    sudo tasksel install ubuntu-desktop
    sudo apt install -y xrdp 
    sudo systemctl enable --now xrdp
    sudo ufw allow from any to any port 3389 proto tcp
  ````
- 確認 Azure Bastion 計費層級為標準<br>
- 使用 RDP 3389 登入 Ubuntu VM<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/linuxrdp1.png "linuxrdp1")<br>
  ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/linuxrdp2.png "linuxrdp2")<br>

回到 [Azure Bastion Lab](https://github.com/BrianHsing/Azure-Bastion) <br>
