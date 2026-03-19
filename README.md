# Azure Files Hands-on


## 目的



## 目標



## 対象



## 前提条件

構築する構成は Azure アーキテクチャセンター にある次の構成をベースとする。

- https://learn.microsoft.com/ja-jp/azure/architecture/example-scenario/hybrid/azure-files-on-premises-authentication



## ハンズオン 目次

-----------------------
環境準備
1. ARM/Bicep 展開開始
1. VM 接続確認

ハンズオン
1. Azure Files 作成
1. Azure Files マウント・読み書き
1. Azure File Sync の Storage Sync Service / Sync Group 作成
1. Agent インストール / サーバー登録 / 初回同期開始
1. 同期待ちの間に Share 分割演習
1. Storage Mover 演習
1. アクセス制御演習
1. File Sync 結果確認
1. Defender for Storage
1. Backup / Immutable

-----------------------
1. ARM/Bicep 展開開始
1. VM 接続確認
1. AD DS 構築 & ドメイン参加

1. Azure Files 作成
1. Azure Files の AD DS 連携（IDベース認証）
1. Azure Files マウント（ドメインユーザー）
1. ACL 設定・動作確認 ← ★重要
1. Azure File Sync 構築開始（同期開始）
1. 同期待ち中に Share 分割
1. Storage Mover
1. File Sync 確認
1. Defender / Backup
