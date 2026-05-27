# GeaconPolaris

**GeaconPolaris is not merely a keyboard firmware repository.**

It is a ZMK-powered split ergonomic input device, built around the idea that typing, pointing, scrolling, navigating, and experimenting with input hardware should live in the same constellation.

Polaris is the North Star of the Geacon lineage: a route marker for modular input, a working keyboard, and a platform for trying trackballs, touchpads, rotary encoders, joystick-style controls, and sensor-driven interaction without treating them as afterthoughts.

![Physical keymap preview](https://github.com/te9no/zmk-config-GeaconPolaris/blob/main/keymap-svg/Polaris.svg)

## Concept

GeaconPolaris is a split ergonomic keyboard configuration for [ZMK Firmware](https://zmk.dev/), targeting `seeeduino_xiao_ble`-based shield combinations.

The design is centered on a simple premise: a keyboard can be more than a grid of switches. It can be a navigation instrument. The left-hand side is expressed as a set of module variants, while the right-hand side also has pointing-module variants. The firmware keeps those routes explicit through separate shields, a shared physical layout, and a keymap that treats pointing and scrolling as first-class operations.

This repository is therefore both:

- a usable ZMK user config for building Polaris firmware;
- a record of an input-device experiment at the edge of keyboard, pointer, and controller.

## Features

- **Split ergonomic ZMK configuration** for `seeeduino_xiao_ble`.
- **GeaconPolaris shield set** under `boards/shields/GeaconPolaris/`.
- **Modular left-hand variants**:
  - `Polaris_L_TB`: trackball module.
  - `Polaris_L_JOY`: joystick module.
  - `Polaris_L_ENC`: rotary encoder module.
  - `Polaris_L_TPD`: touchpad module.
- **Right-hand pointing variants**:
  - `Polaris_R_TB`: trackball module.
  - `Polaris_R_TPD`: touchpad module.
  - `Polaris_R_IQS`: IQS-based module.
- **Shared physical layout definition** in `boards/shields/GeaconPolaris/Polaris.dtsi`.
- **Seven ZMK layers** defined in `config/Polaris.keymap`: `DEF`, `FUNC`, `NUM`, `SNIPE`, `BT`, `SCROLL`, and `SSNIPE`.
- **Combos** for language switching: `COMBO_LANG1` and `COMBO_LANG2`.
- **Runtime sensor rotate behaviors** for scrolling and volume-style sensor bindings.
- **layout-shift key press behavior** via `layout_shift.dtsi` and `zmk,behavior-layout-shift-key-press`.
- **Generated visual previews**:
  - `keymap-svg/Polaris.svg`: physical-layout-based keymap SVG with combo overlays.
  - `keymap-drawer/Polaris.svg`: keymap-drawer output.
- **GitHub Actions builds** through ZMK's reusable `build-user-config.yml` workflow.

## Modular Input System

Polaris is designed as a modular input platform, not just as a typing board.

The left-hand side can be built with different module shields depending on what kind of input experiment you want to run:

| Module | Shield | Role |
| --- | --- | --- |
| Trackball | `Polaris_L_TB` | Pointer control from the left-hand module position. |
| Joystick | `Polaris_L_JOY` | Directional or analog-style input experiments. |
| Rotary Encoder | `Polaris_L_ENC` | Rotational input, scrolling, volume, or mode control. |
| Touchpad | `Polaris_L_TPD` | Surface-based pointing or gesture-oriented input. |

The right-hand side also has pointing-module variants:

| Module | Shield | Role |
| --- | --- | --- |
| Trackball | `Polaris_R_TB` | Right-hand trackball variant. |
| Touchpad | `Polaris_R_TPD` | Right-hand touchpad variant. |
| IQS | `Polaris_R_IQS` | IQS-based right-hand input module. |

The modules are represented in firmware as separate shield overlays and build matrix entries rather than hidden conditionals. That makes each route visible: choose the module, build the matching artifact, flash the matching half.

TODO: Document the physical attachment mechanism and whether modules are hot-swappable at the hardware level.

## Keymap

The keymap lives in [`config/Polaris.keymap`](config/Polaris.keymap).

Layer constants are defined as:

| Constant | Value | Layer node | Label |
| --- | ---: | --- | --- |
| `DEF` | `0` | `default_layer` | `Def` |
| `FUNC` | `1` | `function_layer` | `Fnc` |
| `NUM` | `2` | `num_layer` | `Num` |
| `SNIPE` | `3` | `snipe_layer` | `Snipe` |
| `BT` | `4` | `bt_layer` | `BT` |
| `SCROLL` | `5` | `scroll_layer` | `SCROLL` |
| `SSNIPE` | `6` | `SSNIPE_layer` | `SSNIPE` |

### Layer Notes

- **`DEF` / `default_layer`**: the main typing layer. It includes layer-tap access to `BT`, `NUM`, `FUNC`, and `SNIPE`, mouse buttons on the right-hand side, and momentary access to `SCROLL` / `SSNIPE`.
- **`FUNC` / `function_layer`**: navigation, function keys, and layout-shift toggle via `&tog_ls`.
- **`NUM` / `num_layer`**: number row, symbols, enter/backspace/delete, and navigation keys.
- **`SNIPE` / `snipe_layer`**: a precision-oriented layer with mostly transparent bindings and navigation controls.
- **`BT` / `bt_layer`**: Bluetooth clear/select bindings, including `BT_CLR`, `BT_SEL 0` through `BT_SEL 4`, and `BT_CLR_ALL`.
- **`SCROLL` / `scroll_layer`**: scroll-oriented layer using sensor bindings.
- **`SSNIPE` / `SSNIPE_layer`**: a second precision/scroll-related layer using the same sensor-binding style.

### Combos

Two combos are currently defined:

| Combo | Binding | Key positions |
| --- | --- | --- |
| `COMBO_LANG1` | `&kp LANGUAGE_1` | `<1 2>` |
| `COMBO_LANG2` | `&kp LANG2` | `<2 3>` |

The physical-layout SVG preview draws these combos as overlays connecting their `key-positions`, so the keymap is visible as geometry, not just text.

### Behaviors and Sensors

`config/Polaris.keymap` defines and uses several non-trivial behaviors:

- `original_key_press`: keeps the original `zmk,behavior-key-press` available.
- `kp`: overrides key press through `zmk,behavior-layout-shift-key-press`.
- `rsr_msch`: runtime sensor rotate behavior for vertical scroll bindings.
- `rsr_mscv`: runtime sensor rotate behavior for horizontal scroll bindings.
- `rsr_vol`: runtime sensor rotate behavior for volume-style bindings.

TODO: Explain the intended user-facing behavior of `layout_shift.dtsi` in more detail.

## Build

Firmware builds are defined by [`build.yaml`](build.yaml) and executed by [`.github/workflows/build.yml`](.github/workflows/build.yml).

The build workflow calls:

```yaml
uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3
```

It runs on:

- manual `workflow_dispatch`;
- pushes that change `config/**`, `boards/**`, or `build.yaml`.

The workflow also downloads produced artifacts into `firmware/<branch>/` and commits them back to the repository.

### Build Matrix

| Artifact | Board | Shields | Snippet |
| --- | --- | --- | --- |
| `Polaris_L_MODULE_TB` | `seeeduino_xiao_ble` | `Polaris_L_Base Polaris_L_TB rgbled_adapter nice_oled` | `zmk-usb-logging studio-rpc-usb-uart` |
| `Polaris_L_MODULE_JOY` | `seeeduino_xiao_ble` | `Polaris_L_Base Polaris_L_JOY rgbled_adapter nice_oled` | `zmk-usb-logging studio-rpc-usb-uart` |
| `Polaris_L_MODULE_ENC` | `seeeduino_xiao_ble` | `Polaris_L_Base Polaris_L_ENC rgbled_adapter nice_oled` | `zmk-usb-logging studio-rpc-usb-uart` |
| `Polaris_L_MODULE_TPD` | `seeeduino_xiao_ble` | `Polaris_L_Base Polaris_L_TPD rgbled_adapter nice_oled` | `zmk-usb-logging studio-rpc-usb-uart` |
| `Polaris_R_MODULE_TB` | `seeeduino_xiao_ble` | `Polaris_R_Base Polaris_R_TB rgbled_adapter` | `zmk-usb-logging studio-rpc-usb-uart` |
| `Polaris_R_MODULE_TPD` | `seeeduino_xiao_ble` | `Polaris_R_Base Polaris_R_TPD rgbled_adapter` | `zmk-usb-logging studio-rpc-usb-uart` |
| `Polaris_R_MODULE_IQS` | `seeeduino_xiao_ble` | `Polaris_R_Base Polaris_R_IQS rgbled_adapter` | `zmk-usb-logging studio-rpc-usb-uart` |
| settings reset | `seeeduino_xiao_ble` | `settings_reset` | none |

### Local Build Example

Use the same board and shield combinations as `build.yaml`.

Example: left-hand trackball module.

```sh
west build -b seeeduino_xiao_ble -- \
  -DSHIELD="Polaris_L_Base Polaris_L_TB rgbled_adapter nice_oled"
```

Example: right-hand IQS module.

```sh
west build -b seeeduino_xiao_ble -- \
  -DSHIELD="Polaris_R_Base Polaris_R_IQS rgbled_adapter"
```

TODO: Document the exact local west setup expected for this repository, including module dependencies from `config/west.yml`.

## Keymap SVG Workflow

The keymap SVG is generated by [`.github/workflows/draw-keymap.yml`](.github/workflows/draw-keymap.yml).

This workflow calls a reusable workflow from `te9no/zmk-workspace`:

```yaml
jobs:
  draw-keymap-svg:
    uses: te9no/zmk-workspace/.github/workflows/draw-keymap-svg.yml@main
    permissions:
      contents: write
    with:
      commit_message: "[Draw keymap-svg] ${{ github.event.head_commit.message || 'manual run' }}"
      amend_commit: false
      keymap_patterns: "config/*.keymap"
      output_folder: "keymap-svg"
      destination: "both"
      artifact_name: "keymap-svg"
```

It runs on:

- manual `workflow_dispatch`;
- pushes that change keymap, layout, JSON, keymap-drawer config, or the workflow file.

The output is written to [`keymap-svg/Polaris.svg`](keymap-svg/Polaris.svg) and uploaded as an artifact.

## Repository Structure

| Path | Purpose |
| --- | --- |
| `config/Polaris.keymap` | ZMK keymap, layers, combos, and behavior definitions. |
| `config/Polaris.json` | Layout JSON used by drawing tools. |
| `config/west.yml` | West manifest for ZMK/modules used by this config. |
| `boards/shields/GeaconPolaris/Polaris.dtsi` | Shared physical layout, matrix transform, modules, aliases, LEDs, sensors, and battery node. |
| `boards/shields/GeaconPolaris/Polaris_L_Base.overlay` | Left-hand base shield overlay. |
| `boards/shields/GeaconPolaris/Polaris_L_TB.overlay` | Left-hand trackball module overlay. |
| `boards/shields/GeaconPolaris/Polaris_L_JOY.overlay` | Left-hand joystick module overlay. |
| `boards/shields/GeaconPolaris/Polaris_L_ENC.overlay` | Left-hand rotary encoder module overlay. |
| `boards/shields/GeaconPolaris/Polaris_L_TPD.overlay` | Left-hand touchpad module overlay. |
| `boards/shields/GeaconPolaris/Polaris_R_Base.overlay` | Right-hand base shield overlay. |
| `boards/shields/GeaconPolaris/Polaris_R_TB.overlay` | Right-hand trackball module overlay. |
| `boards/shields/GeaconPolaris/Polaris_R_TPD.overlay` | Right-hand touchpad module overlay. |
| `boards/shields/GeaconPolaris/Polaris_R_IQS.overlay` | Right-hand IQS module overlay. |
| `build.yaml` | ZMK build matrix. |
| `.github/workflows/build.yml` | Firmware build and artifact commit workflow. |
| `.github/workflows/draw-keymap.yml` | Physical-layout keymap SVG workflow. |
| `keymap-svg/Polaris.svg` | Generated physical-layout SVG preview. |
| `keymap-drawer/` | keymap-drawer config and output. |

## Notes / Philosophy

Polaris is named like a direction, not a product SKU.

A normal keyboard config answers: which key sends which code? GeaconPolaris asks a broader question: how many forms of input can a split ergonomic device hold before it stops being a keyboard and becomes a navigation surface?

The answer here is intentionally practical. The idea is not hidden in prose; it is expressed as shields, overlays, build artifacts, keymap layers, combos, and generated SVGs. The myth is allowed only because the implementation is there to carry it.

This is the Geacon lineage pointing north: toward a keyboard that can type, point, scroll, switch modes, and still be built from inspectable ZMK configuration.

## TODO

- Document hardware photos, assembly notes, and module installation procedure.
- Clarify whether the left-hand modules are electrically hot-swappable or simply build-time/module variants.
- Expand `layout_shift.dtsi` explanation with examples.
- Document each pointing module's runtime behavior from the user's perspective.
- Add flashing instructions for each generated artifact.
- Confirm power configuration and battery expectations in hardware terms.
- Add screenshots or rendered images for each module variant if available.

## Fact-Check Notes

The following points should be verified against hardware documentation or the author's notes before being presented as final claims:

- Physical attachment and swapping mechanism for the left-hand modules.
- Exact sensor hardware used by `Polaris_R_IQS`.
- Battery type, runtime, and charging/power path assumptions.
- Whether `nice_oled` is present for all left-hand builds or only specific hardware assemblies.
- Exact intended user semantics of `SNIPE`, `SCROLL`, and `SSNIPE` beyond their current keymap bindings.
- Meaning and expected workflow for `layout_shift.dtsi`.

## License

See [`LICENCE`](LICENCE) for license details.

---

# GeaconPolaris 日本語版

**GeaconPolaris は、単なる ZMK の設定リポジトリではありません。**

これは ZMK ベースの分割エルゴノミクス入力装置です。キー入力、ポインティング、スクロール、レイヤー切替、センサー入力を、ひとつの分割デバイスの中で扱うための実験場でもあります。

Polaris は北極星です。Geacon 系譜の中で、どの方向へ進むのかを示す目印です。トラックボール、ジョイスティック、ロータリーエンコーダー、タッチパッド、IQS 系の入力モジュールを、ただの付属品ではなく、入力装置の中心的な要素として扱います。

## コンセプト

GeaconPolaris は `seeeduino_xiao_ble` を対象にした ZMK firmware configuration です。

ただし目的は「キーを並べること」だけではありません。分割キーボードの上に、複数のポインティングデバイスやセンサー入力をどう載せるか。そのとき keymap、layer、shield、overlay、build artifact をどう整理すれば、実験と実用を両立できるか。Polaris はそのための構成です。

## 特徴

- `seeeduino_xiao_ble` 向けの ZMK 分割キーボード設定。
- `boards/shields/GeaconPolaris/` にまとまった shield 群。
- 左手側モジュール: `Polaris_L_TB`, `Polaris_L_JOY`, `Polaris_L_ENC`, `Polaris_L_TPD`。
- 右手側モジュール: `Polaris_R_TB`, `Polaris_R_TPD`, `Polaris_R_IQS`。
- `boards/shields/GeaconPolaris/Polaris.dtsi` による shared physical layout。
- `config/Polaris.keymap` に定義された 7 layers: `DEF`, `FUNC`, `NUM`, `SNIPE`, `BT`, `SCROLL`, `SSNIPE`。
- `COMBO_LANG1` / `COMBO_LANG2` による language switching combo。
- `runtime-sensor-rotate` 系 behavior によるスクロール/センサー入力。
- `layout_shift.dtsi` と `zmk,behavior-layout-shift-key-press` による layout shift 対応。
- `keymap-svg/Polaris.svg` による physical-layout ベースの SVG 表示。combo も線で表示されます。

## モジュール入力システム

Polaris の特徴は、左手側をモジュール化された入力装置として扱っている点です。

| モジュール | Shield | 役割 |
| --- | --- | --- |
| Trackball | `Polaris_L_TB` | 左手側のポインティング入力。 |
| Joystick | `Polaris_L_JOY` | 方向入力やアナログ的な入力実験。 |
| Rotary Encoder | `Polaris_L_ENC` | 回転入力、スクロール、音量、モード操作など。 |
| Touchpad | `Polaris_L_TPD` | 面入力、ジェスチャ、ポインティング実験。 |

右手側にも module variant があります。

| モジュール | Shield | 役割 |
| --- | --- | --- |
| Trackball | `Polaris_R_TB` | 右手側 trackball variant。 |
| Touchpad | `Polaris_R_TPD` | 右手側 touchpad variant。 |
| IQS | `Polaris_R_IQS` | IQS 系の右手側入力モジュール。 |

各モジュールは `build.yaml` 上で別 artifact として扱われます。つまり、どの入力装置を使うかが firmware の build route として明示されています。

TODO: モジュールの物理的な交換方法、通電中の交換可否、固定方法は別途確認が必要です。

## キーマップ

キーマップは [`config/Polaris.keymap`](config/Polaris.keymap) です。

| 定数 | 値 | layer node | label |
| --- | ---: | --- | --- |
| `DEF` | `0` | `default_layer` | `Def` |
| `FUNC` | `1` | `function_layer` | `Fnc` |
| `NUM` | `2` | `num_layer` | `Num` |
| `SNIPE` | `3` | `snipe_layer` | `Snipe` |
| `BT` | `4` | `bt_layer` | `BT` |
| `SCROLL` | `5` | `scroll_layer` | `SCROLL` |
| `SSNIPE` | `6` | `SSNIPE_layer` | `SSNIPE` |

`DEF` は通常入力の中心です。`FUNC` / `NUM` / `BT` / `SNIPE` への layer-tap や、`SCROLL` / `SSNIPE` への momentary access が置かれています。

`FUNC` は function key と navigation、`NUM` は数字と記号、`BT` は Bluetooth profile 操作です。`SNIPE`、`SCROLL`、`SSNIPE` はポインティングやスクロールのための layer として扱われています。

combo は以下の 2 つです。

| Combo | Binding | key-positions |
| --- | --- | --- |
| `COMBO_LANG1` | `&kp LANGUAGE_1` | `<1 2>` |
| `COMBO_LANG2` | `&kp LANG2` | `<2 3>` |

## ビルド

ビルド設定は [`build.yaml`](build.yaml)、GitHub Actions は [`.github/workflows/build.yml`](.github/workflows/build.yml) です。

workflow は ZMK の reusable workflow を呼びます。

```yaml
uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3
```

`workflow_dispatch` または `config/**`, `boards/**`, `build.yaml` の変更で実行されます。

主な artifact は以下です。

| Artifact | Shields |
| --- | --- |
| `Polaris_L_MODULE_TB` | `Polaris_L_Base Polaris_L_TB rgbled_adapter nice_oled` |
| `Polaris_L_MODULE_JOY` | `Polaris_L_Base Polaris_L_JOY rgbled_adapter nice_oled` |
| `Polaris_L_MODULE_ENC` | `Polaris_L_Base Polaris_L_ENC rgbled_adapter nice_oled` |
| `Polaris_L_MODULE_TPD` | `Polaris_L_Base Polaris_L_TPD rgbled_adapter nice_oled` |
| `Polaris_R_MODULE_TB` | `Polaris_R_Base Polaris_R_TB rgbled_adapter` |
| `Polaris_R_MODULE_TPD` | `Polaris_R_Base Polaris_R_TPD rgbled_adapter` |
| `Polaris_R_MODULE_IQS` | `Polaris_R_Base Polaris_R_IQS rgbled_adapter` |

ローカルビルド例:

```sh
west build -b seeeduino_xiao_ble -- \
  -DSHIELD="Polaris_L_Base Polaris_L_TB rgbled_adapter nice_oled"
```

## SVG 生成

[`.github/workflows/draw-keymap.yml`](.github/workflows/draw-keymap.yml) は `te9no/zmk-workspace` 側の reusable workflow を呼び、`keymap-svg/Polaris.svg` を生成します。

```yaml
uses: te9no/zmk-workspace/.github/workflows/draw-keymap-svg.yml@main
```

この SVG は `zmk,physical-layout` と `config/Polaris.keymap` から生成され、combo の `key-positions` も overlay として表示します。

## リポジトリ構成

| Path | 内容 |
| --- | --- |
| `config/Polaris.keymap` | keymap、layer、combo、behavior 定義。 |
| `config/Polaris.json` | 描画ツール用 layout JSON。 |
| `config/west.yml` | ZMK/modules の west manifest。 |
| `boards/shields/GeaconPolaris/Polaris.dtsi` | physical layout、matrix transform、module、LED、sensor、battery node。 |
| `boards/shields/GeaconPolaris/*.overlay` | base/module ごとの shield overlay。 |
| `build.yaml` | ZMK build matrix。 |
| `.github/workflows/build.yml` | firmware build workflow。 |
| `.github/workflows/draw-keymap.yml` | keymap SVG workflow。 |
| `keymap-svg/Polaris.svg` | physical-layout ベースの SVG preview。 |
| `keymap-drawer/` | keymap-drawer 設定と出力。 |

## 設計メモ

Polaris は、キーボードを「キーだけの装置」として閉じません。

分割入力装置は、文字入力のためのものでもあり、ポインタを動かすものでもあり、スクロールするものでもあり、状態を切り替えるものでもあります。GeaconPolaris は、その全部を ZMK の設定として見える形に落とし込もうとしています。

思想はあります。ただし、思想だけではなく、shield、overlay、keymap、build matrix、SVG として実装で確認できるようにしています。

## TODO / 確認事項

- モジュールの物理的な交換方式を記述する。
- `Polaris_R_IQS` の具体的なセンサー/入力デバイスを確認する。
- 電源、電池、充電、稼働時間に関する記述を確認する。
- `layout_shift.dtsi` の具体的な使い方を例付きで説明する。
- `SNIPE`, `SCROLL`, `SSNIPE` の意図をユーザー視点で補足する。
- 各 artifact の flash 手順を追加する。

## License

[`LICENCE`](LICENCE) を参照してください。
