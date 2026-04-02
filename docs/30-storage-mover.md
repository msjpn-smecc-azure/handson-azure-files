# Azure Storage Mover を使った Azure Files への移行

#### ⏳ 推定時間

- 分

#### 💡 学習概要


#### 🗒️ 目次


## 移行作業のログ保管ストレージ作成

移行ログ保存用の Log Analytics ワークスペース を用意

1. [Azure ポータル](https://portal.azure.com/) を開き、上部検索窓にて `Log Analytics ワークスペース` を検索

1. 「作成」を選択

1. Log Analytics ワークスペース の作成

    1. 基本

        - サブスクリプション: (ハンズオン用のもの)
        - リソースグループ: (ハンズオン用のもの)
        - 名前: `handson-asm-log` (任意)
        - リージョン: `Japan East`
    
    1. タグ

        指定なし

    1. レビューと作成

        内容を確認して「作成」


## Azure Storage Mover リソースの作成

Azure Storage Mover のリソースを作成

1. [Azure ポータル](https://portal.azure.com/) を開き、上部検索窓にて `Azure Storage Mover` を検索

1. 「作成」を選択

1. Storage Mover の作成

    1. 基本

        - サブスクリプション: (ハンズオン用のもの)
        - リソースグループ: (ハンズオン用のもの)
        - 名前: `handson-asm` (任意)
        - リージョン: `West US3`
    
    1. 監視

        - コピーログを有効: ☑️
        - Log Analytics ワークスペース: `handson-asm-log` (先ほど作成した Log Analytics ワークスペース)

    1. タグ

        指定なし

    1. レビューと作成

        内容を確認して「作成」


## Storage Mover エージェント をデプロイ

> [!IMPORTANT]  
> Hyper-V が利用できる環境が必要です。
> Azure VM の場合、 **「入れ子になった仮想化 (Nested virtualization)」がサポート** されている SKU (例: Dasv5、Ev5 シリーズなど。D4as_v5, E4as_v5 など) を選択して VM を作成してください。
> また、あらかじめ **Hyper-V の役割を有効化** しておいてください。

> [!IMPORTANT]  
> Azure Storage Mover エージェントの最小要件は以下の通りです。
>
> アプリケーション： 
> - CPU: 4 コア以上
> - メモリ: 8 GiB 以上 (=ホストマシンは 8GiB より多くのメモリが必要)
> - ディスク: 20 GiB 以上の空き容量
>
> ネットワーク:
> - HTTPSのアウトバウンド通信を許可

> [!WARNING]  
> Azure Storage Mover エージェントを Azure VM 上で動かす構成は、Microsoft Learn 上では **未検証かつ非サポート** です。
> この手順はハンズオンや検証用途として実施してください。
> 本番利用では、ソースに近いオンプレミスまたはサポート対象の Hyper-V / VMware 環境へエージェントを配置する構成を推奨します。

1. Azure ポータル から On-premises File Server VM に接続

1. VM内のブラウザで Storage Mover エージェント のダウンロードページを開き、 Hyper-V のイメージをダウンロード

    - https://www.microsoft.com/en-us/download/details.aspx?id=104590

1. ダウンロードした zip ファイルを展開し、Storage Mover エージェント の VHD を任意のフォルダーに配置

    例： `C:\Hyper-V\storagemover-agent.vhd`


1. Hyper-V マネージャーで Hyper-Vサーバー(自分自身のVM) を右クリック、[New]-[Virtual Machine] を選択し、エージェント VM を作成

    1. Specify Name and Location
        - Name: `StorageMoverAgent` (任意)
        - Store the virtual machine in a different location: ☑️
        - Location: エージェントの VHD を配置したフォルダー (例: `C:\Hyper-V\agent`)
    1. Specify Generation
        - **Generation 1** (サポートは Gen 1 のみ)
    1. Assign Memory
        - Startup memory: `8 GiB (8192 MiB)` 以上を目安に設定
        - Use Dynamic Memory for this virtual machine: ☑️
    1. Configure Networking
        - Connection: `Internal Switch` (あらかじめ作成しておいた内部仮想スイッチを選択)
    1. Connect Virtual Hard Disk
        - Use an existing virtual hard disk:
            - zipファイルを展開して配置した Storage Mover エージェントの VHD を選択 (例: `C:\Hyper-V\storagemover-agent.vhd`)
    
1. 作成したエージェント を右クリック、[Settings] を開く

    以下の修正をして「OK」

    - Processor: プロセッサ数を最低 `4` コアに設定
    - Integration Services: `Guest services` を有効化


## Storage Mover エージェントを登録

> [!IMPORTANT]
> 登録に使用するアカウントには、対象のリソース グループおよび Storage Mover リソースに対する十分な権限が必要です。

1. エージェント VM を起動

1. コンテナ起動まで **10分程度** 待つ

1. Storage Mover Agent にログイン

    - 初期ユーザー名: `admin`
    - 初期パスワード: `admin`

    (*) 初回ログイン時にパスワードの変更が求められます。要件を満たす新しいパスワードを設定してください。

1. エージェントの管理シェルからネットワーク接続性を確認

    1. `2) Network configuration` を選択
    1. `3) Test network connectivity` を選択
    1. Region に `westus3`、 Azure Arc private link に `N` を指定して接続確認

        すべての接続が成功することを確認

    1. 問題ないことが確認できたら `6) Quit` で最初の画面に戻る

1. エージェントの管理シェルで `4) Register` を選択し、Azure Storage Mover リソースへ登録

    登録時に以下の情報を入力します。

    - Azure region: `westus3` (Storage Mover リソースを作成したリージョン)
    - Connect Agent using Azure Arc private link: `N`
    - Tenant ID: (ハンズオン用のもの)
    - Subscription ID: (ハンズオン用のもの)
    - Resource group name: (ハンズオン用のもの)
    - Storage mover resource name: `handson-asm` (作成済のもの)
    - Agent name: `handson-asm-agent` (任意)

    登録途中で表示される `https://microsoft.com/devicelogin` にアクセスし、表示されたコードでサインインします。

1. Azure ポータルで Storage Mover リソースを開き、[リソース管理]-[登録済みエージェント] に作成したエージェントが表示されることを確認


## キーコンテナーを作成

キーコンテナーの作成

1. Azure ポータル を開き、上部検索窓にて `キー コンテナー` (または `Key Vault`) を検索

1. 「作成」を選択

1. キーコンテナーの作成

    1. 基本

        - サブスクリプション: (ハンズオン用のもの)
        - リソースグループ: (ハンズオン用のもの)
        - 名前: `handson-kv` (任意)
        - リージョン: `Japan East`
        - 価格レベル: `Standard`
        - 回復オプション
            - 論理的な削除: ☑️
            - 削除されたキーコンテナーの保持期間: `7` 日
            - 消去保護: `消去保護を無効にする`
    
    1. アクセス構成

        - アクセス許可モデル: `Azure ロールベースのアクセス制御(Azure RBAC)`
        - リソースアクセス: (設定なし)
    
    1. ネットワーク

        - パブリックアクセスを有効にする: ☑️
        - パブリックアクセス:
            - 許可するアクセス元: `すべてのネットワーク`

    1. タグ

        指定なし

    1. レビューと作成

        内容を確認して「作成」

キーコンテナーに対するアクセス権を付与

1. Azure ポータルで作成したキーコンテナーを開き、[アクセス制御 (IAM)] を選択

1. 「ロールの割り当ての追加」を選択し、以下の情報を入力してエージェントにアクセス権を付与

    - ロール: `キー コンテナー シークレット責任者` (`Key Vault Secrets Officer`)
    - メンバー: `ユーザー、グループ、またはサービスプリンシパルを選択`
    - メンバーの選択: (自分自身)


ファイルサーバーへ接続するための資格情報をキーコンテナーに登録

1. Azure ポータルで作成したキーコンテナーを開き、[オブジェクト]-[シークレット] を選択

1. 「生成/インポート」を選択し、以下の情報を入力してユーザー名に相当するシークレットを作成

    - 方法: `手動`
    - 名前: `onpremise-fileserver-username` (任意)
    - シークレット値: `azureuser` (ファイル共有サーバーへ接続するためのユーザー名)
    - 有効: `はい`

    同様に、以下の情報でパスワード用のシークレットも作成

    - 方法: `手動`
    - 名前: `onpremise-fileserver-password` (任意)
    - シークレット値: (ファイル共有サーバーへ接続するためのパスワード)
    - 有効: `はい`


## ソースエンドポイントを作成

移行元のソースエンドポイントと、移行先のターゲットエンドポイントを作成します。

1. Azure ポータルで Storage Mover リソースを開き、[リソース管理]-[ストレージエンドポイント] を開く

1. 「ソースエンドポイント」タブになっていることを確認し、「エンドポイントの作成」を選択

1. ソースエンドポイントを作成

    - ソースの種類: `SMB`
    - ホスト名またはIP: `10.1.0.5` (ファイル共有サーバーのプライベートIPアドレス)
    - 共有名: `share` (ファイル共有サーバー 上の 共有フォルダー の名前)
    - キーコンテナー: `handson-kv` (事前に作成した Key Vault)
    - ユーザー名のシークレット:
        - `ユーザー名シークレットを選択する`
        - `onpremise-fileserver-username` (事前に作成したユーザー名のシークレット)
    - パスワードのシークレット:
        - `パスワードシークレットを選択する`
        - `onpremise-fileserver-password` (事前に作成したパスワードのシークレット)

## ターゲットエンドポイントを作成

1. Azure ポータルで Storage Mover リソースを開き、[リソース管理]-[ストレージエンドポイント] を開く

1. 「ターゲットエンドポイント」タブに移動し、「エンドポイントの作成」を選択

1. ターゲットエンドポイントを作成

    - サブスクリプション: (ハンズオン用のもの)
    - ストレージアカウント: (事前に作成した Azure Storage アカウント)
    - ターゲットの種類: `ファイル共有`
    - プロトコル: `SMB`
    - ファイル共有: `share` (事前に作成した Azure Files 共有)


## 移行プロジェクトを構成

1. Azure ポータルで Storage Mover リソースを開き、[移行の計画と実行]-[プロジェクト] を開く

1. 「プロジェクトの作成」を選択し、以下の情報を入力してプロジェクトを作成

    - 名前: `handson-asm-project` (任意)
    - 説明: (任意)

1. 作成したプロジェクトを開き、「ジョブの作成」を選択

1. ジョブを作成

    1. 基本

        - 名前: `handson-asm-migration` (任意)
        - 説明: (任意)
        - 移行の種類: `オンプレミスからクラウドへ`
        - 登録済みエージェント: `handson-asm-agent` (事前に登録したエージェント)

    1. ソース

        - ソースエンドポイント: `既存のエンドポイント参照を選択する`
        - 既存のソースエンドポイント: (事前に作成したソースエンドポイント)

    1. ターゲット

        - ターゲットエンドポイント: `既存のエンドポイント参照を選択する`
        - 既存のターゲットエンドポイント: (事前に作成したターゲットエンドポイント)

    1. 設定

        - コピーモード: `コンテンツをターゲットにマージする`

    1. レビューと作成

        内容を確認して「作成」

構成したジョブの実行

1. 作成したジョブを選択し、「ジョブの開始」を選択

1. ジョブの開始

    「開始」を選択してジョブを開始


## ファイル移行の確認

1. Azure ポータルで Storage Mover リソースを開き、[移行の計画と実行]-[ジョブ] を開く

1. ジョブのステータスを確認し、移行が正常に完了していることを確認

1. Azure ポータルで 作成した Azure Storage アカウントを開き、Azure Files 共有内にファイルが移行されていることを確認

> [!NOTE]  
> Storage Account へ直接移行した場合、 Storage Sync Service で検知がうまくできない場合があります。
> File Sync Server へ入って強制的に同期させることで解決する場合があります。

