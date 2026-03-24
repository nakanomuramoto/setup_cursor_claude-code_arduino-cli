# プロジェクト概要

Cursor Editor + Claude Code + WSL2 + Arduino CLI を使って ESP32 ボードに Arduino スケッチを書き込む環境。

---

## このPCでやりたいこと

このリポジトリの `install-guide.md` に記載された手順に従って、以下の環境を構築してほしい：

1. WSL2（Ubuntu）のセットアップ確認
2. Node.js と Claude Code のインストール
3. Cursor に Claude Code 拡張機能と WSL 拡張機能をインストール
4. Arduino CLI のインストールと PATH 設定
5. ESP32 ボードパッケージの追加
6. TFT_eSPI ライブラリのインストールと設定
7. USB デバイスを WSL に接続（usbipd）
8. スケッチのコンパイルと書き込み確認

---

## 環境情報

| 項目 | 内容 |
|---|---|
| OS | Windows 11 + WSL2（Ubuntu） |
| エディター | Cursor（最新版） |
| ボード | TTGO T-Display v1.1（ESP32 + ST7789V 1.14インチTFT） |
| FQBN | `esp32:esp32:esp32` |
| シリアルポート | `/dev/ttyACM0`（環境によって異なる場合あり） |
| USB-シリアル変換 | CH9102（usbipd で WSL に接続） |

---

## インストールするもの

```bash
# Node.js
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Claude Code
sudo npm install -g @anthropic-ai/claude-code

# Arduino CLI（install.sh を使う方法）
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh

# ESP32 ボードパッケージ
arduino-cli config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
arduino-cli core update-index
arduino-cli core install esp32:esp32

# TFT_eSPI ライブラリ
arduino-cli lib install "TFT_eSPI"
```

---

## TFT_eSPI のピン設定（TTGO T-Display 用）

以下のファイルを編集する：

```
~/Arduino/libraries/TFT_eSPI/User_Setup_Select.h
```

変更内容：
- `#include <User_Setup.h>` をコメントアウト（先頭に `//` を追加）
- `#include <User_Setups/Setup25_TTGO_T_Display.h>` のコメントを解除

---

## USB 接続手順（毎回必要）

**PowerShell（管理者）で実行：**

```powershell
usbipd list                        # BUSID を確認
usbipd bind --busid <BUSID>        # 初回のみ
usbipd attach --wsl --busid <BUSID>
```

**WSL で確認：**

```bash
ls /dev/ttyACM*   # → /dev/ttyACM0 が表示されればOK
```

---

## よく使うコマンド

```bash
# コンパイル
arduino-cli compile --fqbn esp32:esp32:esp32 スケッチ名.ino

# 書き込み
arduino-cli upload --fqbn esp32:esp32:esp32 --port /dev/ttyACM0 スケッチ名.ino

# dialout グループへの追加（書き込み権限エラーが出たとき）
sudo usermod -a -G dialout $USER
# → WSL を再起動してから再試行
```

---

## 注意事項

- `npm install -g` は必ず `sudo` を付ける（権限エラー回避）
- 作業フォルダは `/mnt/` 以下ではなく `~/projects/` 以下を使う（WSL パフォーマンス向上）
- Cursor は必ず「WSL: Open Folder in WSL」でフォルダを開く
- `usbipd attach` は PC 再起動やUSB抜き差しのたびに再実行が必要

---

## 参考ドキュメント

- 詳細なインストール手順: `install-guide.md`
- Claude Code への引き継ぎ方法: `claude-code-context-handoff.md`
- 参考動画: https://youtu.be/WxiSVc7muVM?si=mRzUa5MiKB8gq7SF
