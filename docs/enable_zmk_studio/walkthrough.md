# ZMK Studio 有効化 修正内容の確認

ZMK Studioを有効にするための設定変更が完了しました。

## 変更内容

### 1. ZMKファームウェアのリビジョン変更
- [west.yml](file:///home/lindq/develop/zmk-config-LiNEA40/config/west.yml) の `zmk` リビジョンを `v0.2.1` から `main` に変更しました。これにより、ZMK Studio の最新機能が利用可能になります。

### 2. ビルド設定の更新
- [build.yaml](file:///home/lindq/develop/zmk-config-LiNEA40/build.yaml) において、左手（left）側のビルド構成でも ZMK Studio を有効化しました（`snippet: studio-rpc-usb-uart`, `cmake-args: -DCONFIG_ZMK_STUDIO=y`）。これにより、左手側をPCにUSB接続した際に ZMK Studio から設定を変更できるようになります。

### 3. キーマップの確認（既存）
- すでにレイヤー5（WIRELESS）に `&studio_unlock` が配置されていることを確認しています。

## 使用方法

1. **ビルドと書き込み**: GitHub Actions で新しくビルドされたファームウェアをダウンロードし、キーボード（左手側）に書き込んでください。
2. **接続**: 左手側を USB で PC に接続します。
3. **ロック解除**: キーボードをレイヤー5（WIRELESS）に切り替え、`studio_unlock` キー（左手側の上段、`bootloader` キーの右）を押します。
4. **ZMK Studio を開く**: ブラウザで [https://zmk.studio/](https://zmk.studio/) にアクセスし、デバイスを接続してください。

## 注意事項
- ZMK Studioでキーマップを変更した場合、リポジトリの `.keymap` ファイルの内容より、Studioでの設定が優先されます。元の状態に戻したい場合は、Studio内の「Restore Stock Settings」を実行してください。
