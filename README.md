# Azure Files Hands-on


## 目的

オンプレファイルサーバーから Azure Files への移行を想定し、Azure Files の構成や管理方法を学ぶことを目的とするハンズオンです。

## 目標

このハンズオンでは以下の内容について学習します。

- Azure Files の構成方法 および 利用方法
- Azure File Sync を利用したオンプレファイルサーバーと Azure Files の同期構成方法
- Storage Mover を利用したオンプレファイルサーバーから Azure Files へのファイル移行方法


## 対象

以下のような方を対象として想定しています。

- クラウド管理者​
- クラウドアーキテクト​
- ネットワークエンジニア​


## 前提条件

構築する構成は Azure アーキテクチャセンター にある次の構成をベースとする。

- https://learn.microsoft.com/ja-jp/azure/architecture/example-scenario/hybrid/azure-files-on-premises-authentication


## ハンズオン 目次

1. [前提環境の構築](./docs/00-init-infra.md)
2. [Azure Files の基礎](./docs/10-files-basic.md)
3. [Azure File Sync Server の構築](./docs/20-file-sync-server.md)
4. [Storage Mover を利用したファイル移行](./docs/30-storage-mover.md)

