# ZMK Studio 有効化計画

ZMK Studioを有効にするために、以下の変更を行います。

## 背景
ZMK Studioは、キーマップをリアルタイムで変更できるGUIツールです。
現状、`build.yaml`で右側（right）のみに設定がありますが、通常USB接続する側（Central、一般的に左側）で有効にする必要があります。また、ZMKのバージョンを最新の`main`ブランチに向けることで、確実に対応させます。

## ユーザーレビューが必要な事項
- **USB接続**: ZMK Studioは基本的にUSB接続で使用します。
- **左手側への接続**: 通常、左手側が「Central（親機）」として動作するため、左手側をUSBでPCに接続してください。

## 変更内容

### 1. ZMKファームウェアのバージョン更新
#### [MODIFY] [west.yml](file:///home/lindq/develop/zmk-config-LiNEA40/config/west.yml)
- `zmk`の`revision`を`v0.2.1`から`main`に変更し、最新のZMK Studio対応コードを利用可能にします。

### 2. ビルド設定の修正
#### [MODIFY] [build.yaml](file:///home/lindq/develop/zmk-config-LiNEA40/build.yaml)
- 左側（left）のビルド設定にも `snippet: studio-rpc-usb-uart` と `cmake-args: -DCONFIG_ZMK_STUDIO=y` を追加します。これにより、左側をUSB接続した際にZMK Studioが利用可能になります。

### 3. キーマップの確認
- すでに `&studio_unlock` が `WIRELESS` レイヤー（Layer 5）に配置されていることを確認済みです（左手側の `bootloader` の隣）。これを使用してロックを解除します。

## 開放的な質問
- 現在、主に右側をPCに接続して使用されていますか？もしそうであれば、現在の `build.yaml` の設定で合っていますが、一般的には左側を接続することが多いため、両方または左側での有効化を推奨します。

## 検証計画
### 手動確認
- GitHub Actionsでビルドが成功することを確認。
- ファームウェアを書き込み、[ZMK Studio (web)](https://zmk.studio/) に接続できるか確認。
- レイヤー5の `studio_unlock` キーを押して、設定変更が可能になるか確認。
