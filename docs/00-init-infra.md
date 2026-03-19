# 前提環境の構築

#### ⏳ 推定時間

- 分

#### 💡 学習概要


#### 🗒️ 目次

- [ARMテンプレート を使って デプロイ](#armテンプレート-を使って-デプロイ)

## ARMテンプレート を使って デプロイ

- Client VM
    - Windows Server 2025 Datacenter
- ADDS VM
    - Windows Server 2025 Datacenter
- File Sync Server VM
    - Windows Server 2025 Datacenter
    - D drive をつける(=File Sync の同期用ドライブ)


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

