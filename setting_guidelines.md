# LiNEA40 ZMK設定の責務分離ガイド

## 概要

ZMKのキーボード設定は複数のファイルにまたがっており、それぞれのファイルが担うべき責務を明確にすることで、設定の見通しがよくなり、変更時の影響範囲を把握しやすくなる。

---

## ファイル構成と責務

### ハードウェア定義層

#### `LiNEA40.dtsi`
**責務：ハードウェアの構造定義**

キーボードの物理的な構造（キーレイアウト、マトリクス変換、GPIOピン割り当て）を定義する。左右共通の定義をここに集約し、右手・左手それぞれのoverlayから参照される。

- キーの物理配置（`physical_layout`）
- マトリクス変換（`keymap_transform`）
- KSCANのGPIO設定
- エンコーダーの定義
- `trackball_listener` の箱だけを定義（`status = "disabled"`）

**ここに書くべきでないもの：** レイヤー番号、タイムアウト値、動作設定

---

#### `LiNEA40_right.overlay`
**責務：右手側のハードウェア固有設定**

右手側のSPI、トラックボールセンサーの接続、GPIOピンの割り当てを定義する。センサーとZMKのリスナーを繋ぐ役割。

- SPIのピン設定（SCLK、SDIO、NCS）
- PMW3610センサーの接続設定
- `trackball_listener` の有効化とデバイス紐付け
- オートマウス・スクロール・スナイプの**レイヤー番号指定**

**現状の問題点：**

```c
trackball: trackball@0 {
    automouse-layer = <1>;  // Layer 1（NAV）を指定 ← keymapの定義と不一致
    snipe-layers = <2>;
    scroll-layers = <5>;
};
```

`automouse-layer = <1>` がハードコードされており、キーマップ側でレイヤー番号を変えても自動では追従しない。

**改善案：**

```c
trackball: trackball@0 {
    automouse-layer = <4>;  // MOUSE layer
    snipe-layers = <2>;     // SYM layer
    scroll-layers = <5>;    // WIRELESS layer → 将来的に専用レイヤーに
};
```

---

#### `LiNEA40_right.conf`
**責務：ハードウェアドライバーのパラメータ設定**

センサーの動作特性（CPI、ポーリングレート、省電力設定）やBluetooth・バッテリー関連の設定を行う。ここはハードウェアの「チューニング」層であり、キーマップの論理には関与しない。

- センサーのCPI・ポーリングレート
- 省電力（ダウンシフト）設定
- Y/X軸の反転
- オートマウスのタイムアウト（ms単位）
- Bluetooth・バッテリー設定

**現状の問題点：**

```
CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS=600
```

オートマウスのタイムアウトがここで設定されているが、キーマップ側でも `zip_temp_layer` の第2引数（2000ms）として別のタイムアウトが設定されており、二重管理になっている。

**改善案：**

オートマウスをドライバーに任せるなら `.conf` のタイムアウトを正とし、キーマップ側の `zip_temp_layer` は削除する。

```
CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS=600  # ここで一元管理
```

---

### キーマップ・動作層

#### `LiNEA40.keymap`
**責務：キーの論理的な動作定義**

レイヤー構成、キーバインド、センサー動作、input processorの設定を行う。ハードウェアの詳細には関与せず、「このキーを押したらこう動く」という論理だけを定義する。

- レイヤー番号の定義（`#define`）
- `&mmv`、`&msc` の速度・加速度チューニング
- input processorの設定（スクロール速度スケーリングなど）
- 各レイヤーのキーバインド
- センサーバインド（エンコーダー）

**ここに書くべきでないもの：** GPIOピン番号、SPI設定、センサーのCPI

**現状の問題点：**

```c
&trackball_listener {
    input-processors = <&zip_temp_layer MOUSE 2000>;
    move {
        layers = <MOUSE>;
        input-processors = <&zip_xy_scaler 1 1>;
        process-next;
    };
};
```

ドライバー側（`automouse-layer`）とキーマップ側（`zip_temp_layer`）の両方でオートマウスが設定されており、どちらが実際に動いているか不明瞭。

**改善案：**

オートマウスの制御はドライバー側に一本化し、キーマップ側からは削除する。

```
// &trackball_listenerへのオートマウス設定は不要
// ドライバー側(LiNEA40_right.overlay)のautomouse-layerで管理
```

---

## 現状の問題まとめ

| 問題 | 場所 | 内容 |
|---|---|---|
| オートマウスの二重管理 | `.overlay` + `.keymap` | `automouse-layer` と `zip_temp_layer` が競合 |
| レイヤー番号の不一致 | `.overlay` | `automouse-layer = <1>` だがkeymapではLayer 4をMOUSEとして定義 |
| タイムアウトの二重管理 | `.conf` + `.keymap` | 600msと2000msが別々に設定 |

---

## 改善後の責務マップ
```
LiNEA40.dtsi
  └─ ハードウェア構造（キー配置、マトリクス、GPIO）

LiNEA40_right.overlay
  └─ センサー接続 + レイヤー番号の指定
       - automouse-layer = <4>  // MOUSEレイヤー
       - snipe-layers = <2>     // SYMレイヤー
       - scroll-layers = <4>    // MOUSEレイヤー（または専用）

LiNEA40_right.conf
  └─ センサーのチューニングパラメータ
       - CPI、ポーリングレート
       - タイムアウト（AUTOMOUSE_TIMEOUT_MS）
       - 省電力設定

LiNEA40.keymap
  └─ キーの論理定義
       - レイヤー構成（#define）
       - キーバインド
       - mmv/mscの速度設定
       - input processor（スクロール倍速など）
```

---

## 変更時のチェックリスト

レイヤー番号を変更する場合：
- [ ] `LiNEA40.keymap` の `#define` を更新
- [ ] `LiNEA40_right.overlay` の `automouse-layer`、`snipe-layers`、`scroll-layers` を更新

オートマウスのタイムアウトを変更する場合：
- [ ] `LiNEA40_right.conf` の `CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS` のみを変更

センサーの感度を変更する場合：
- [ ] `LiNEA40_right.conf` の `CONFIG_PMW3610_CPI` を変更