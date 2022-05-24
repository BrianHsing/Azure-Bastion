# 透過 Terraform on Azure 快速建立環境

## 環境與安裝必要條件

- 電腦環境
  - OS : Windows 11 專業版<br>
  - IDE : Visual Studio Code<br>
    https://code.visualstudio.com/download<br>
- 安裝 Azure CLI<br>
  https://docs.microsoft.com/zh-TW/cli/azure/install-azure-cli-windows?tabs=azure-cli<br>
  完成後，可以直接在 Terminal 輸入以下指令來驗證是否安裝成功：<br>
    ```
    az version
    ```
    成功後即可看到以下回覆的目前版本資訊：<br>
    ```
    {
    "azure-cli": "2.36.0",
    "azure-cli-core": "2.36.0",
    "azure-cli-telemetry": "1.0.6",
    "extensions": {}
    }
    ```
- 在 Windows 上安裝 Terraform<br>
  - 下載 Terraform<br>
    https://www.terraform.io/downloads.html<br>
  - 將下載下來的 terraform.exe 放在 C:\terraform 資料夾中，並將此路徑加入至系統環境變數中<br>
    ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/systempath.png "systempath")<br>
  - 開啟命令提示字元<br>
  - 輸入命令 ` terraform -version`<br>

# 使用 Azure CLI 登入

在這個部分可以持續使用 Terminal 繼續進行操作，但本篇會透過 Vscode 所內建的 Terminal 介面直接進行操作，這部分可以自行選擇要使用哪種方式。<br>

登入之前您必須確認幾件事情：<br>
  1. 確保自己的帳號與訂用帳戶有進行關聯，您必須是`參與者`才能建立 Azure 服務資源<br>
  2. 後續在建立服務主體時確保自己帳號的權限為`擁有者`或`使用者存取系統管理員`的權限，如果您沒有這兩個其中之一的權限，就會收到錯誤訊息<br>
  3. 登入 Azure CLI 後，務必要確認 TenantId、SubscriptionId 是不是您進行部署或管理所提供的<br>

確認後即可開始執行以下操作：<br>

- 首先在 Terminal 介面中執行 `az login`，之後會自動開啟網頁<br>
- 選擇您具有上述權限的帳號進行登入，並且能在您的 Terminal 中看到帳務資訊<br>
  ```
  [
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "您的TenantId",
    "id": "您的訂用帳戶id",
    "isDefault": true,
    "managedByTenants": [],
    "name": "您的訂用帳戶名稱",
    "state": "Enabled",
    "tenantId": "您的目錄ID",
    "user": {
      "name": "isadmin@brianhsing.club",
      "type": "user"
    }
  }
  ]
  ```
# 使用 Azure CLI 建立服務主體

請輸入下方命令列建立服務主體，主要是會在 Azure AD 註冊應用程式，並且給予此應用程式參與者的角色，才能透過 Terraform 進行部署與維運，其中的 `<service_principal_name>` 您可以自訂名稱：<br>
```
az ad sp create-for-rbac --name <service_principal_name> --role Contributor --scopes "/subscriptions/<subscription id>"
```

完成後您將可以得到`appId`、`password`、`tenant`、`displayname`等數值，請務必保留這些數值，稍後會需要用到：<br>
```
{
  "appId": "您的appId",
  "displayName": "您的displayName",
  "password": "您的password",
  "tenant": "您的tenant"
}
```
接下來就是將上述的資訊新增至環境變數，當然如果您只是要簡單進行測試，也可以略過這個步驟，將相關資訊加入到 Terraform 的 mail.tf 中：<br>
```
export ARM_SUBSCRIPTION_ID="<您的訂用帳戶id>"
export ARM_TENANT_ID="<您的tenant>"
export ARM_CLIENT_ID="<您的appId>"
export ARM_CLIENT_SECRET="<您的password>"
```