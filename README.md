# Azure Bastion Lab
Azure Bastion 可讓您使用瀏覽器和 Azure 入口網站連線到虛擬機器，為了讓您快速的熟悉這個功能，本篇設計了一個實驗環境，透過利用 Terraform 佈署，可讓您在十分鐘左右將此環境佈署完成，並且開始您的功能驗證與測試。
## 環境架構
![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/architecture-1.png "architecture")
- 演練環境說明<br>
  - 模擬在同個訂用帳戶中，使用虛擬網路對等互連，連接東亞與東南亞的虛擬網路，並且依序完成 Kerberos 驗證、RDP 連線 Linux 虛擬機器、原生用戶端連線、NSG 管理規則等。主要會建立兩個虛擬網路、三台虛擬機器、一個 Azure Bastion 服務<br>
  - vnet-hub 網路位址為 172.31.10.0/23，包含兩個子網路 Azure Bastion (172.31.11.0/26)、hub-subnet (172.31.10.0/24)，建立區域在東亞<br>
  - vnet-spoke1 網路位址為 172.31.20.0/24，包含一個子網路 hub-subnet，建立區域在東南亞<br>
  - vnet-hub 與 vnet-spoke1 有建立對等互連，並且每個子網路都有套用網路安全性群組<br>
  - 三台虛擬機器分別為 win2k22-adds (172.31.10.4)、<br>