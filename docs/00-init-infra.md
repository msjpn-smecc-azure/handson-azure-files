# 前提環境の構築

#### ⏳ 推定時間

- 分

#### 💡 学習概要


#### 🗒️ 目次

- [ARMテンプレート を使って デプロイ](#armテンプレート-を使って-デプロイ)

## ARMテンプレート を使って デプロイ

- Client VM
    - Windows Server 2025 Datacenter
    - ファイルサーバーへ接続するクライアント相当。
<!--
 - ADDS VM
    - Windows Server 2025 Datacenter
    - ファイル共有の移行元想定
    - Storage Mover Agent の Hyper-V を動作させる
    - 仮想化機能が使えるSKUを選択する必要がある (例: Dv5、Ev5 シリーズなど)
-->
- On-premises File Server VM
    - Windows Server 2025 Datacenter
    - オンプレミスのファイルサーバー相当。
      以下の２機能を同梱。
        - ファイルサーバー
        - Hyper-V (Storage Mover Agent) 
    - 仮想化機能が使えるSKUを選択する必要がある (例: Dv5、Ev5 シリーズなど)
- File Sync Server VM
    - Windows Server 2025 Datacenter
    - オンプレミスに配置する File Sync Server 相当。
      以下の2機能を同梱。
        - ファイルサーバー
        - File Sync Agent
    - Dドライブ など 追加ドライブ をつける(=File Sync の同期用ドライブ。OSドライブ以外が必要)

<!-- 
## ADDSの構成

初期設定

1. 初回起動の Server Manager ポップアップ

    1. `Don't show this message again` にチェックを入れて閉じる

1. Server Manager 画面

    1. 右上メニュー [Manage]-[Server Manager Properties] を開く

    1. `Do not start Server Manager automatically at logon` にチェックを入れて「OK」

    1. 左メニュー「Local Server」を開き、「IE Enhanced Security Configuration」を選択、「Administrators: `Off`」「Users： `Off`」にして「OK」


ADDS機能の追加

1. Server Manager を開き、右上 [Manage]-[Add Roles and Features] を開く

1. 「Add Roles and Features Wizard」画面

    1. 「Before You Begin」は `Skip this page by default` にチェックを入れて「Next」

    1. 「Installation Type」は `Role-based or feature-based installation` のまま「Next」

    1. 「Server Selection」は自分自身のサーバーが選択されいていることを確認して「Next」

    1. 「Server Roles」にて `Active Directory Domain Services` をチェックして「Next」

    1. 「Features」は変更せずそのまま「Next」

    1. 「ADDS」はそのまま「Next」

    1. 「Install」


ADDSの初期設定

1. Server Manager を開き、右上 「Notifications（アラートが出ている）」を選択、「Post-deployment Configuration」の通知にある `Promote this server to a domain controller` リンクを開く

1. 「Active Directory Domain Services Configuration Wizard」画面

    1. 「Deployment Configuration」

        - Select the deployment operation: `Add a new forest`
        - Root domain name: `test.local`

    1. 「Domain Controller Options」

        - Password & Confirm password: (任意)
    
    1. 「DNS Options」

        警告は無視してそのまま「Next」

    1. 「Additional Options」, 「Paths」, 「Review Options」

        デフォルトまま
    
    1. 「Prerequisites Check」

        内容確認して「Install」

1. 自動再起動


ユーザー作成

1. Server Manager を開き、右上 [Tools]-[Active Directory Users and Computers] を開く

1. [test.local]-[Users] を 右クリック、[New]-[User] を選択

1. 「New Object - User」

    - First name, Last name, User logon name: (任意)

    - Password, Confirm password: (任意)
    - User must change password at next logon: `チェックなし` ←デフォルトから変更
    - User cannot change password: `チェックなし`
    - Password never expires: `チェック` ←デフォルトから変更
    - Account is disabled: `チェックなし`
-->

## ファイルサーバー の構成

### ファイルサーバー の役割を有効化

1. Server Manager を開き、右上 [Manage]-[Add Roles and Features] を開く

1. 「Add Roles and Features Wizard」画面

    1. 「Before You Begin」は「Next」

    1. 「Installation Type」は `Role-based or feature-based installation` のまま「Next」

    1. 「Server Selection」は自分自身のサーバーが選択されいていることを確認して「Next」

    1. 「Server Roles」にて以下の役割をチェックして「Next」

        - ☑️ `File and Storage Services`
            - ☑️ `File and iSCSI Services`
                - ☑️ `File Server`
                - ☑️ `File Server Resource Manager`

    1. 「Confirmation」は以下の設定をして「Install」
    
        - **Restart** the destination server automatically if required: ☑️

### 共有フォルダの作成

1. 任意の場所に共有用のフォルダを作成、フォルダ内に適当なファイルを作成しておく

    例：
    ```
    D:\share
        └─ sample_from_onpre.txt
    ```

1. Server Manager を開き、左メニュー [File and Storage Services]-[Shares] を開く

1. 左側「Shares」の右にあるメニューから [Tasks]-[New Share] を選択

1. New Share Wizard 画面

    1. Select the profile for this share

        - `SMB Share - Advanced` を選択して「Next」

    1. Specify the share location

        - `Type a custom path` を選択して、先ほど作成した共有用のフォルダを指定 (例: `D:\share`)

    1. Specify share name

        - Share name: `share` (任意)

    1. Configure share settings

        そのまま「Next」

        - [ ] Enable access-based enumeration
        - [x] Allow caching of share
        - [ ] Encrypt data access

    1. Permissions

        例として、Everyone に対してフルコントロールを与える設定を行う

        1. [Customize permissions] を選択
        1. Advanced Secuirty Setting 画面
            1. [Add] を選択、 Everyone に対してフルコントロールを与える設定を行う
                1. Permission Entry 画面

                    以下の設定を行って「OK」

                    - Principal: `Everyone`
                    - Type: `Allow`
                    - Applies to: `This folder, subfolders and files`
                    - Basic permissions: `Full Control` にチェック(その他は自動ですべてチェック)

            1. 設定確認して「OK」

    1. Management Properties

        そのまま「Next」

    1. Quota
        
        設定せず、そのまま「Next」
    
    1. Confirmation

        内容確認して「Create」


## Hyper-V の構成

### Hyper-V, DHCP, DNS の役割を有効化

1. Server Manager を開き、右上 [Manage]-[Add Roles and Features] を開く

1. 「Add Roles and Features Wizard」画面

    1. 「Before You Begin」は「Next」

    1. 「Installation Type」は `Role-based or feature-based installation` のまま「Next」

    1. 「Server Selection」は自分自身のサーバーが選択されいていることを確認して「Next」

    1. 「Server Roles」にて以下の役割をチェックして「Next」

        - `Hyper-V`
        - `DHCP Server`
        - `DNS Server`

    1. 「Features」は変更せずそのまま「Next」

    1. 「Hyper-V」
        
        1. Create Virtual Switches

            デフォルトまま「Next」

        1. Virtual Machine Migration

            デフォルトまま「Next」

        1. Default Stores

            - disk files: `C:\Hyper-V`
            - configurations files: `C:\Hyper-V`
            
            デフォルトから変更して「Next」

    1. 「DHCP Server」

        そのまま「Next」

    1. 「DNS Server」

        そのまま「Next」

    1. 「Confirmation」は以下の設定をして「Install」
    
        - **Restart** the destination server automatically if required: ☑️

再起動したら「Server Manager」を開き、DHCP サーバーの Post-Install configuration を完了させる

1. Server Manager を開き、右上 「Notifications（アラートが出ている）」を選択、「Complete DHCP configuration」の通知にある `Complete DHCP configuration` リンクを開く

1. 「DHCP Post-Install configuration wizard」画面

    以下のセキュリティグループ作成がされる

    - `DHCP Administrators`
    - `DHCP Users`

    内容確認して「Commit」


### Virtual Switch の作成
<!-- 
1. VM内で **Hyper-V マネージャー** を開き、右側パネルの「Virtual Switch Manager (仮想スイッチ マネージャー)」を開く

1. 「New virtual network switch (新しい仮想ネットワーク スイッチ)」を選択し、「External (外部)」を選択して、「Create Virtual Switch (仮想スイッチの作成)」を選択

    以下の設定をして「OK」を選択

    - Name: `External Virtual Switch` (任意)
    - Connection type: `External network`
        - `Microsoft Hyper-V Network Adapter`
        - `Allow management operating system to share this network adapter`: ☑️ 
-->

1. PowerShell を管理者で起動

1. 内部仮想スイッチを作成

    ```powershell
    New-VMSwitch -Name "Internal Switch" -SwitchType Internal
    ```

1. 内部仮想スイッチにIPアドレスを割り当て

    ```powershell
    Get-NetAdapter -Name "vEthernet (Internal Switch)" | New-NetIPAddress -IPAddress "172.16.0.1" -PrefixLength 24
    ```

1. ホスト側にNAT構成を作成

    ```powershell
    New-NetNat -Name "Internal Switch NAT" -InternalIPInterfaceAddressPrefix "172.16.0.0/24"
    ```

### DHCP の構成

1. DHCP を起動

1. [DHCP]-[<YOUR_SERVER_NAME>]-[IPv4] を右クリック、[New Scope] を選択

1. New Scope Wizard 画面

    1. Welcome to the New Scope Wizard

        「Next」

    1. Scope Name

        - Name: `Hyper-V Internal Switch Scope`

    1. IP Address Range

        - Start IP address: `172.16.0.2`
        - End IP address: `172.16.0.254`
        - Length: `24`

    1. Add Exclusions and Delay

        そのまま「Next」

    1. Lease Duration

        そのまま「Next」

    1. Configure DHCP Options

        `Yes, I want to configure these options now` を選択して「Next」
    
    1. Router (Default Gateway)

        `172.16.0.1` (Internal Switch に設定したIPアドレス) を追加して「Next」

    1. Domain Name and DNS Servers

         Azure DNS のIPアドレス が入っていることを確認して「Next」

        - IP address: `172.16.0.1` (Internal Switch に設定したIPアドレス)

    1. WINS Servers

        そのまま「Next」

    1. Activate Scope

        `Yes, I want to activate this scope now` を選択して「Next」

    1. Summary

        内容確認して「Finish」


### DNS の構成

1. DNS を起動

1. [DNS]-[<YOUR_SERVER_NAME>] を右クリック、[Properties] を選択

1. [Interfaces] タブを開き、以下のIPアドレスのみを設定

    - `Only the following IP addresses` を選択
        - `172.16.0.1` (Internal Switch に設定したIPアドレス) のみ選択して、他のIPアドレスは選択解除

1. [Forwarders] タブを開き、以下のIPアドレスを追加して「OK」

    - `168.63.129.16` (Azure DNS のIPアドレス)


### Storage Mover エージェント の vhd取得

1. VM内のブラウザを開く

1. Storage Mover エージェント のダウンロードページを開き、 Hyper-V のイメージをダウンロード

    - https://www.microsoft.com/en-us/download/details.aspx?id=104590

1. 7zip をダウンロードおよびインストール

    - https://7-zip.org/download.html  
        └ `.exe` 形式, `64-bit Windows x64` インストーラーをダウンロードしてインストール

> [!IMPORTANT]  
> Hyper-V のイメージを解凍する際、 Windows 標準の解答機能だと正しく解凍できない場合があります。
> 正しく解凍できない場合は、7zip などのサードパーティ製の解凍ツールを利用して解凍してください。
