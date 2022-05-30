# Azure Bastion Lab
Azure Bastion 可讓您使用瀏覽器和 Azure 入口網站連線到虛擬機器，為了讓您快速的熟悉這個功能，本篇設計了一個實驗環境，透過利用 Terraform 佈署，可讓您在十分鐘左右將此環境佈署完成，並且開始您的功能驗證與測試。在本篇文件您將練習:<br>
- 使用 Terraform 佈署 Azure 資源<br>
- Azure Bastion 功能操作:<br>
  - 使用瀏覽器在 Azure 入口網站中透過 Azure Bastion 登入虛擬機器<br>
  - 使用現有的 Windows AD 驗證登入虛擬機器<br>
  - 使用 RDP 連線存取 Linux GUI<br>
  - 使用本機原生用戶端經由 Azure Bastion 連線至虛擬機器<br>
  - 了解如何透過網路安全性群組控管您的輸入與輸出連線<br>
## 環境架構
![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/architecture-1.png "architecture")<br>
- 演練環境說明<br>
  - 模擬在同個訂用帳戶中，使用虛擬網路對等互連，連接東亞與東南亞的虛擬網路，並且依序完成 Kerberos 驗證、RDP 連線 Linux 虛擬機器、原生用戶端連線、NSG 管理規則等。主要會建立兩個虛擬網路、三台虛擬機器、一個 Azure Bastion 服務<br>
  - vnet-hub 網路位址為 172.31.10.0/23，包含兩個子網路 Azure Bastion (172.31.11.0/26)、hub-subnet (172.31.10.0/24)，建立區域在東亞<br>
  - vnet-spoke1 網路位址為 172.31.20.0/24，包含一個子網路 hub-subnet，建立區域在東南亞<br>
  - vnet-hub 與 vnet-spoke1 有建立對等互連，並且每個子網路都有套用網路安全性群組<br>
  - 三台虛擬機器分別為 win2k22-adds (172.31.10.4)、ubuntu1804 (172.31.20.4)、srv01 (172.31.20.5)<br>
    - win2k22-adds 作為 Windows AD，srv01 作為網域內的電腦，在進行 Kerberos 驗證練習時會使用<br>
    - ubuntu1804 會用於 RDP 連線 Linux 虛擬機器的練習<br>
  - 建立一個 Azure Bastion 標準計費層，用於提供安全並且快速地連線到虛擬機器<br>
## 演練流程 <br>

- [Lab1 - 透過 Terraform on Azure 快速建立環境](https://github.com/BrianHsing/Azure-Bastion/blob/main/lab1.md)<br>
- [Lab2 - 使用原生用戶端連線至 VM 並且利用 Kerberos 驗證](https://github.com/BrianHsing/Azure-Bastion/blob/main/lab2.md)<br>
- [Lab3 - 使用 RDP 連線至 Linux VM](https://github.com/BrianHsing/Azure-Bastion/blob/main/lab3.md)<br>
