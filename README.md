# Azure Bastion Lab
Azure Bastion 可讓您使用瀏覽器和 Azure 入口網站連線到虛擬機器，為了讓您快速的熟悉這個功能，本篇設計了一個實驗環境，透過利用 Terraform 佈署，可讓您在十分鐘左右將此環境佈署完成，並且開始您的功能驗證與測試。
## 環境架構
![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/architecture.png "architecture")
- 演練環境說明<br>
  - 模擬在同個訂用帳戶中，使用虛擬網路對等互連，連接東亞與東南亞的虛擬網路，並且依序完成 Kerberos 驗證、RDP 連線 Linux 虛擬機器、原生用戶端連線、NSG 管理規則等。<br>
  - 內部使用網段為 172.16.10.0/24。<br>
  - ADDS01 內部 IP 為 172.16.10.10。<br>
  - AP01 內部 IP 為 172.16.10.9，對外服務網址為 demo.brianhsing.fun/wordpress/。外部 IP 位址為 114.32.xxx.223<br>