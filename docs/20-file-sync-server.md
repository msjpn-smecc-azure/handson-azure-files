# Azure File Sync Server 構築

#### ⏳ 推定時間

- 分

#### 💡 学習概要


#### 🗒️ 目次


> [!NOTE]  
> Azure File Sync サーバーのデプロイ手順の最新情報は以下を参照
> - [Azure File Sync のデプロイ - MS Learn](https://learn.microsoft.com/ja-jp/azure/storage/file-sync/file-sync-deployment-guide)


## File Sync Service のデプロイ

1. [Azure ポータル](https://portal.azure.com/) を開き、上部検索窓にて `Azure File Sync` を検索

1. Azure File Sync のデプロイ

    1. 基本情報

        - サブスクリプション: (ハンズオン用のもの)
        - リソースグループ: (ハンズオン用のもの)
        - ストレージ同期サービス名: `handson-filesync` (任意)
        - リージョン: `Japan East` (リソースグループにあわせる)
    
    1. ネットワーク

        - ネットワーク接続: `プライベート エンドポイントのみ`
        - 「プライベートエンドポイントの追加」を選択
            - サブスクリプション: (ハンズオン用のもの)
            - リソースグループ: (ハンズオン用のもの)
            - 場所: `Japan East` (リソースグループにあわせる)
            - 名前: `handson-filesync-pe` (任意)
            - 仮想ネットワーク: `handson-cloud-vnet` (Files用のVNet)
            - サブネット: `main-snet` (Files用のサブネット)
            - プライべートDNSゾーンと統合: `はい`
            - プライベートDNS: `新規` (デフォルトまま)

    1. タグ

        指定なし

    1. レビューと作成

        内容を確認して「作成」

1. Azureポータルにて作成済みの Storage Sync Service を開く

1. [アクセス制御(IAM)] を開き、「ロールの割り当ての追加」を選択

1. ロールの割り当ての追加

    1. ロール
        
        - `Azure ファイル同期管理者 (Azure File Sync Administrator)`

    1. メンバー

        - アクセスの割当先: `ユーザー、グループ、またはサービスプリシンパル`
        - メンバー: (自分自身)

    1. レビューと割り当て

        「レビューと割り当て」

## Private DNS を On-premise 相当の VNet へリンク

On-premise相当のVNetから 作成した Files への名前解決ができるよう、 作成済み Private DNS を On-premise 相当の VNet へリンクします。

1. Azure ポータルを開き、デプロイ済みの Priavte DNS (=Storage Account 作成時に同時作成したもの) を開く

1. [DNSの管理]-[仮想ネットワークリンク] を開き、「追加」を選択

1. 仮想ネットワークリンクを追加

    - リンク名: `onpre-pdns-lnk` (任意)
    - サブスクリプション: (ハンズオン用のもの)
    - 仮想ネットワーク: `handson-onpremises-vnet` (on-premise相当のVNet)

仮想マシンをすべて再起動して、DNS情報が反映されるようにします。

1. Azure ポータルを開き、仮想マシン 一覧を開く

1. 作成済み仮想マシンをチェックして「再起動」を選択


## マネージドID による アクセス制御 の有効化

> [!NOTE]  
> オンプレミスサーバーの場合、Azure Arc を利用してオンプレマシンをAzureへ統合し、マネージドID を有効化します。

> [!NOTE]  
> 環境によっては最初から有効化されている場合があります。
> その場合、画面の設定箇所の確認のみ実施してください。

File Sync Service へアクセスする仮想マシンのマネージドID を有効化し、アクセス制御を行うことで、よりセキュアな構成にします。

1. Azure ポータルを開き、File Sync Agent 用 VMリソースを開く

1. [セキュリティ]-[ID] を開き、「システム割り当て」タブにある「状態」を「オン」にして「保存」


Storage Sync Service のマネージドIDによるアクセス制御を有効化します。

1. Azure ポータルを開き、File Sync Service のリソースを開く

1. [設定]-[マネージドID] を開き、「マネージドIDを有効にする」を選択



## File Sync サーバーに共有フォルダ を準備

[ファイルサーバーの構成](./00-init-infra.md#ファイルサーバー-の構成) を参考に、File Sync サーバー用の VM に共有フォルダを準備します。

- 準備する共有フォルダ:

    ```
    D:\share
    ```

- アクセス権: 
    - Principal: `Everyone`
    - Type: `Allow`
    - Applies to: `This folder, subfolders and files`
    - Basic permissions: `Full Control` にチェック(その他は自動ですべてチェック)


## File Sync サーバーに エージェント をインストール

1. Azureポータルを開き、File Sync サーバー用の VM に接続

1. VM内のブラウザを立ち上げ、以下のリンクから 「Azure File Sync エージェント」をダウンロード

    - [Microsoft ダウンロードセンター](https://www.microsoft.com/en-us/download/details.aspx?id=57159)

        `StorageSyncAgent_WS2025` (Windows Server 2025) をダウンロード

1. ダウンロードしたインストーラーを実行

    1. Welcome

        「Next」
    
    1. License Agreement

        - `I accept the terms in the License Agreement`: チェック

        「Next」
    
    1. Feature Selection

        - `Azure File Sync`: チェック
        - `Install Storage Sync Agent to:`: (デフォルトまま)

        「Next」
    
    1. Proxy Settings

        - `Use the existing proxy settings configurared on the server`: チェック

        「Next」
    
    1. Microsoft Update

        - `Use Microsoft Update`: 選択

        「Next」

    1. Ready to install Storage Sync Agent

        - `Automatically update when a new version becomes available`: チェックなし 
        - `Collect data necessary to identify and fix problems`: チェックなし

        「Install」

    1. Completed

        「Finish」


## File Sync サーバーを Azure File Sync Service へ登録

> [!IMPORTANT]  
> 環境によっては認証がうまくいかず、上記ウィザードによる登録がうまく動作しない場合があります。
> うまく登録できない場合、本節の後半にコマンドを利用した手動登録があるので、そちらを参照して実施します。

1. Azureポータルを開き、File Sync サーバー用の VM に接続 (直前からの続きであればスキップ)

1. 登録ウィザードの起動

    サーバー登録のウィザードは File Sync エージェントをインストールする後、自動起動するので、指示に従ってサーバーを Azure File Sync Service へ登録します。
    自動起動しない場合、以下のパスにあるファイルを直接起動します。

    ```
    C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe
    ```

1. サーバー登録

    1. Sing in and register this server

        - Azure Environment: `Azure Cloud`
        - I am signing in as a Cloud Solution Provider partner: `No`

        「Sing in」

        Sing in 画面が立ち上がるのでサインインする
    
    1. Choose a Storage Sync Service

        - Azure Subscription: (ハンズオン用のもの)
        - Resource Group: `handson-rg` (作成した Storage Sync Service の所属するリソースグループ名)
        - Storage Sync Service: `handson-filesync` (作成した Storage Sync Service の名前)

        「Register」

    1. Singin

        Azureへのサインインを実施


<details>
<summary>コマンドを使った サーバー登録 手順</summary>

Azure PowerShell のインストール

1. Azure PowerShell を以下のサイトへアクセスしてダウンロード

    https://github.com/Azure/azure-powershell/releases

1. ダウンロードした `Az-Cmdlets-xx.xx.xx.xx-x64.msi` を実行、インストール


サーバーの手動登録

1. PowerShell を「管理者」として起動

1. Azure へサインイン

    以下のいずれかの方法でサインイン

    方法1: 通常

    ```ps
    Connect-AzAccount -Subscription "<your-subscription-guid>" -Tenant "<your-tenant-guid>"
    ```

    方法2: devicecode利用

    ```ps
    Connect-AzAccount -Subscription "<your-subscription-guid>" -Tenant "<your-tenant-guid>" -UseDeviceAuthentication
    ```

    方法3: ClaimsChallenge が出てきた場合（MFAが通過できない場合）

    ```ps
    $claims = "<エラーで指示されたクレーム>"
    $subscription = "<your-subscription-guid>"
    $tenant = "<your-tenant-guid>"

    Connect-AzAccount -Subscription $subscription -Tenant $tenant -ClaimsChallenge $claims
    ```

1. Azure File Sync Service へサーバー登録

    ```ps
    Register-AzStorageSyncServer -ResourceGroupName "<your-resource-group-name>" -StorageSyncServiceName "<your-storage-sync-service-name>"
    ```

(*) 参考: [Azure File Sync エージェントのインストールとサーバー登録のトラブルシューティング - サーバー登録](https://learn.microsoft.com/ja-jp/troubleshoot/azure/azure-storage/files/file-sync/file-sync-troubleshoot-installation#server-registration)

<!--
     1. 以下のスクリプトを実行

        ```
        $ProgressPreference = 'SilentlyContinue'
        Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
        Start-Process msiexec.exe -Wait -ArgumentList '/I', 'AzureCLI.msi', '/quiet'
        Remove-Item .\AzureCLI.msi
        ```
-->

</details>

## 同期グループの作成

1. Azureポータルを開き、File Sync Service のリソースを開く

1. [同期]-[同期グループ] を開き、「同期グループの作成」を選択

1. 同期グループの作成

    以下を設定して「作成」

    - 同期グループ名: `test-sync-group` (任意)
    - クラウドエンドポイント
        - サブスクリプション: (ハンズオン用のもの)
        - ストレージアカウント: `handsonXXXXsa` (作成した Storage Sync Service と同じリソースグループにあるストレージアカウント)
        - Azure ファイル共有: `share` (作成したストレージアカウントにあるファイル共有)


## エンドポイントを追加

> [!IMPORTANT]  
> 同期グループ作成時に指定したクラウドエンドポイントが反映していない場合があります。
> しばらく待つか、もう一度「クラウドエンドポイントの追加」からクラウドエンドポイントを追加して、
> 同期グループにクラウドエンドポイントが追加されていることを確認してください。

1. Azureポータルを開き、File Sync Service のリソースを開く

1. [同期]-[同期グループ] を開き、先ほど作成した同期グループを選択

1. 「概要」にある「クラウドエンドポイントの追加」を選択

1. クラウドエンドポイントの追加

    以下を設定して「作成」

    - サブスクリプション: (ハンズオン用のもの)
    - ストレージアカウント: `handsonXXXXsa` (作成した Storage Sync Service と同じリソースグループにあるストレージアカウント)
    - Azure ファイル共有: `share` (作成したストレージアカウントにあるファイル共有)

1. 「概要」にある「サーバーエンドポイントの追加」を選択

1. サーバーエンドポイントの追加

    以下を設定して「作成」

    - 登録済みサーバー: `filesync-srv` (先ほど登録したサーバー)
    - パス: `D:\share` (共有フォルダとして準備したパス)
    - 初期同期: (デフォルトまま)

## File Sync Server の即時同期

1. Azureポータルを開き、File Sync Service のリソースを開く

1. PowerShell を管理者で開き、以下のコマンドを実行して、File Sync Server とクラウドエンドポイントの即時同期を開始します。

    1. Azure へログイン

        ```powershell
        Connect-AzAccount -Subscription "<YOUR_SUBSCRIPTION_GUID>" -Tenant "<YOUR_TENANT_GUID>" -UseDeviceAuthentication 
        ```

    1. 同期の開始

        ```powershell
        Invoke-AzStorageSyncChangeDetection -ResourceGroupName "<RESOURCE_GROUP_NAME>" -StorageSyncServiceName "<STORAGE_SYNC_SERVICE_NAME>" -SyncGroupName "<SYNC_GROUP_NAME>" -Name "CLOUD_ENDPOINT_NAME"
        ```

        (*) `CLOUD_ENDPOINT_NAME`: 同期グループにあるクラウドエンドポイントを開き、リソースIDに含まれる `.../cloudEndpoints/<CLOUD_ENDPOINT_GUID>` に続く GUID を指定


## File Sync Server の 共有フォルダ 動作確認

1. Azureポータルを開き、Client の VMリソース (`handson-client-vm`) を開く

1. [接続]-[Bastion] を選択して、Bastion 経由で接続

1. Explorer を開き、三点メニューから「Map network drive (ネットワークドライブの割り当て)」を選択

1. ネットワークドライブの割り当て

    - ドライブ: `Z` (任意)
    - フォルダー: `\\filesync-srv\share` (同期グループのサーバーエンドポイントで指定したファイル共有のパス)

    「Finish」

