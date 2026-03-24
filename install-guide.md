# Cursor Editor で Claude Code を使って ESP32 スケッチをアップロードするインストール手順

**環境:** Windows 11  
**目的:** Cursor Editor + Claude Code + WSL2 を使って TTGO T-Display（ESP32）に Arduino スケッチを書き込む  
**作業日:** 2026年3月22日

---

## 目次

0. [このドキュメントの動機](#0-このドキュメントの動機)
1. [Cursor Editor のインストール](#1-cursor-editor-のインストール)
2. [Cursor の日本語化（任意）](#2-cursor-の日本語化任意)
3. [WSL2 のインストール](#3-wsl2-のインストール)
4. [Cursor のターミナル設定](#4-cursor-のターミナル設定)
5. [Node.js のインストール](#5-nodejs-のインストール)
6. [Claude Code のインストール](#6-claude-code-のインストール)
7. [Claude Pro へのアップグレード](#7-claude-pro-へのアップグレード)
8. [Claude Code の初回起動・ログイン](#8-claude-code-の初回起動ログイン)
9. [Cursor と Claude Code の連携](#9-cursor-と-claude-code-の連携)
10. [Arduino CLI のインストール](#10-arduino-cli-のインストール)
11. [ESP32 ボードパッケージの追加](#11-esp32-ボードパッケージの追加)
12. [TFT_eSPI ライブラリのインストール](#12-tft_espi-ライブラリのインストール)
13. [スケッチの作成と書き込み](#13-スケッチの作成と書き込み)
14. [USB デバイスを WSL に接続する（usbipd）](#14-usb-デバイスを-wsl-に接続するusbipd)
15. [コンパイルと書き込み](#15-コンパイルと書き込み)

---

## 0. このドキュメントの動機

**操作場所:** ブラウザ

以下の YouTube 動画を見て、Cursor Editor 上で Claude Code を動かしたいと思ったことがきっかけ：

- [Claude Code を Cursor で動かす（YouTube）](https://youtu.be/WxiSVc7muVM?si=mRzUa5MiKB8gq7SF)

動画では Cursor のエディター右側に Claude Code パネルが表示され、コードを選択すると Claude が内容を認識して編集・生成を行っていた。この環境を Windows 上で再現することが目標。

---

## 1. Cursor Editor のインストール

**操作場所:** ブラウザ / Windows

1. [https://www.cursor.com/](https://www.cursor.com/) にアクセス
2. 「Download for Windows」をクリックしてインストーラーをダウンロード
3. `CursorSetup.exe` を実行してインストール
4. 起動後、[cursor.com](https://cursor.com) でサインイン（Google アカウントなどで可）

---

## 2. Cursor の日本語化（任意）

**操作場所:** Cursor

1. `Ctrl+Shift+X` で拡張機能パネルを開く
2. 検索欄に `Japanese` と入力
3. **「Japanese Language Pack for VS Code」** をインストール
4. 再起動を促されたら **「Restart」** をクリック

---

## 3. WSL2 のインストール

**操作場所:** Cursor ターミナル（PowerShell）

> ※ Cursor 起動直後に「Linux 用 Windows サブシステムを更新する必要があります」というメッセージが表示された場合、**そのまま進めて問題ありませんでした**。このメッセージは WSL インストールの促進であり、Claude Code を使うために WSL は必須です。

**Cursor のターミナルを開く:** `Ctrl + @`

ターミナル右上の `∨` → PowerShell を選択し、以下を実行：

```powershell
wsl --install
```

- WSL バージョン 2.6.3 がインストールされる
- Ubuntu が自動でインストールされる
- ユーザー名とパスワードの設定を求められるので入力する

> **ポイント:** 今回は再起動なしでそのまま Ubuntu が起動した。通常は再起動が必要な場合もある。

---

## 4. Cursor のターミナル設定

**操作場所:** Cursor

> ※ **この手順は後から不要とわかりましたが、実行しています。**  
> 最初に WSL メッセージを Ctrl+C でキャンセルし、ターミナルを CMD に切り替える作業を行いましたが、**最終的には WSL ターミナルを使うため不要でした。**

ターミナルを開くと `powershell` と `cmd` に ⚠️ マークが出ることがある。これは WSL が正しく設定されていないためで、以下の手順で WSL ターミナルを選択する：

最終的に正しい設定：

1. ターミナル右上の `∨` → **「Select Default Profile」** を選択
2. 一覧から **「Ubuntu (WSL)」** または **「wsl」** を選択
3. ターミナル右上の `+` で新しいターミナルを開く
4. WSL（Ubuntu）ターミナルが開く

---

## 5. Node.js のインストール

**操作場所:** WSL（Ubuntu）ターミナル

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

バージョン確認：

```bash
node --version   # v24.14.0
npm --version    # 11.9.0
```

---

## 6. Claude Code のインストール

**操作場所:** WSL（Ubuntu）ターミナル

> ※ **後でやり直しています。**  
> 最初に `sudo` なしで実行したところ権限エラーが発生しました。

❌ 失敗したコマンド（権限エラー）：
```bash
npm install -g @anthropic-ai/claude-code
# → EACCES: permission denied エラー
```

✅ 正しいコマンド：
```bash
sudo npm install -g @anthropic-ai/claude-code
```

バージョン確認：

```bash
claude --version
# → 2.1.81 (Claude Code)
```

---

## 7. Claude Pro へのアップグレード

**操作場所:** ブラウザ

Claude Code の利用には有料プランまたは API キーが必要。

| 選択肢 | 内容 |
|---|---|
| Claude Pro | $20/月（月払い、いつでも解約可） |
| Anthropic Console | API 従量課金 |

1. [https://claude.ai](https://claude.ai) にアクセス
2. 「Upgrade to Pro」をクリック
3. 月額 $20 プランを選択してクレジットカードで支払い

> **補足:** 解約は `claude.ai → Settings → Billing → Cancel Plan` からいつでも可能。年間プランへの変更も後から可能。

---

## 8. Claude Code の初回起動・ログイン

**操作場所:** WSL（Ubuntu）ターミナル

> ※ **後でやり直しています。**  
> 最初に `/mnt/c/WINDOWS/system32`（Windows のシステムフォルダ）で起動してしまったため、作業フォルダを作成してから再起動しました。

✅ 正しい手順：

```bash
# 作業フォルダを作成
cd /mnt/e
mkdir work
cd work

# Claude Code を起動
claude
```

起動後の設定：

1. **テーマ選択:** `1. Dark mode` を選択して Enter
2. **ログイン方法:** `1. Claude account with subscription` を選択して Enter
3. **ブラウザ認証:**
   - WSL 環境ではブラウザが自動で開かないことがある
   - その場合は **`c` キーを押して URL をコピー** → ブラウザに貼り付けて認証
   - 認証後に表示されるコードをターミナルの `Paste code here if prompted >` に貼り付ける
4. **フォルダ信頼確認:** `1. Yes, I trust this folder` を選択して Enter

「Welcome back saki!」と表示されれば起動成功。

---

## 9. Cursor と Claude Code の連携

**操作場所:** Cursor / WSL ターミナル

### 9-1. Claude Code 拡張機能のインストール

> ※ **後でやり直しています。**  
> 最初に VSIX ファイルを手動でインストールしようとしました。以下のコマンドでファイルを探しましたが見つかりませんでした。

**操作場所:** WSL（Ubuntu）ターミナル

```bash
# VSIXファイルを探す（見つからなかった）
ls ~/.claude/local/node_modules/@anthropic-ai/claude-code/vendor/
# → No such file or directory

ls /usr/lib/node_modules/@anthropic-ai/claude-code/vendor/
# → audio-capture  ripgrep  tree-sitter-bash（claude-code.vsix はなし）
```

✅ **正しい方法:** Cursor の拡張機能マーケットから直接インストールする

1. `Ctrl+Shift+X` で拡張機能を開く
2. 検索欄に `Claude Code` と入力
3. **「Claude Code for VS Code」**（Anthropic 製・5.2M DL）をインストール

### 9-2. WSL 拡張機能のインストール

右下の通知「The 'Windows Subsystem for Linux (WSL)' extension is recommended」→ **Install** をクリック

### 9-3. WSL モードでフォルダを開く

`Ctrl+Shift+P` → `WSL: Open Folder in WSL` → `E:\work` を入力

タイトルバーが **「work [WSL: Ubuntu]」** になれば成功。

### 9-4. IDE 接続確認

> ※ **後でやり直しています。**  
> 最初はフォルダを開かずに Claude Code を起動していたため、`/ide` で「No available IDEs detected」が表示されました。WSL モードでフォルダを開いてから再試行することで解決しました。

WSL ターミナルで：

```bash
cd /mnt/e/work
claude
```

Claude Code 起動後に：

```
/ide
```

と入力 → `1. Cursor ✓` が表示されたら Enter

```
Connected to Cursor.
```

と表示されれば連携完了。

### 9-5. Claude Code パネルの表示

`Ctrl+Shift+P` → `Claude Code: Open` と入力して実行

右サイドバーに「CLAUDE CODE」パネルが表示される。

---

## 10. Arduino CLI のインストール

**操作場所:** Claude Code パネル（Cursor）または WSL ターミナル

> ※ 当初は以下の curl コマンドでインストールする予定でしたが、Claude Code パネルに依頼する方法で実施しました。どちらでも問題ありません。

参考：curl を使った手動インストール方法（WSL ターミナル）：
```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
```

✅ **今回実施した方法:** Claude Code パネルの入力欄に以下を入力：

```
Arduino CLIをWSLにインストールして、バージョンを確認してください
```

Claude Code が自動でインストールを実行（インストール先: `/mnt/e/work/bin/arduino-cli`）。

完了後、WSL ターミナルで PATH を設定：

```bash
source ~/.bashrc
arduino-cli version
# → arduino-cli Version: 1.4.1
```

---

## 11. ESP32 ボードパッケージの追加

**操作場所:** WSL（Ubuntu）ターミナル

```bash
arduino-cli config init
```

> ※ すでに設定ファイルが存在する場合は `Config file already exists` と表示されるが問題なし。

```bash
arduino-cli config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

```bash
arduino-cli core update-index
```

```bash
arduino-cli core install esp32:esp32
# → Platform esp32:esp32@3.3.7 installed
```

---

## 12. TFT_eSPI ライブラリのインストール

> **この作業の背景:**  
> 当初の目標（XIAO ESP32C6 + DHT11 + Zigbee）の前に、手持ちの [TTGO T-Display v1.1](https://tamanegi.digick.jp/computer-embedded/mcuboa/tdisplayv11/)（ESP32 + 1.14インチ ST7789V TFT 内蔵）でテストすることにした。

**操作場所:** WSL（Ubuntu）ターミナル

```bash
arduino-cli lib install "TFT_eSPI"
```

### User_Setup_Select.h の場所を確認

TFT_eSPI のピン設定ファイルを探す：

```bash
find ~/Arduino -name "User_Setup_Select.h" 2>/dev/null
find ~/.arduino15 -name "*.h" 2>/dev/null | head -5
ls ~/Arduino/libraries/
```

結果：
```
/home/muramoto/Arduino/libraries/TFT_eSPI/User_Setup_Select.h
```

### TFT_eSPI のピン設定（TTGO T-Display 用）

**操作場所:** Claude Code パネル（Cursor）

Claude Code パネルの入力欄に以下を入力：

```
/home/muramoto/Arduino/libraries/TFT_eSPI/User_Setup_Select.h を編集して、
TTGO T-Displayの設定を有効にしてください。Setup25_TTGO_T_Display.hの行のコメントを外してください。
```

Claude Code が自動で以下の 2 箇所を編集：
- 27行目: `#include <User_Setup.h>` → コメントアウト
- 58行目: `Setup25_TTGO_T_Display.h` → コメント解除（有効化）

---

## 13. スケッチの作成と書き込み

**操作場所:** WSL（Ubuntu）ターミナル → Claude Code パネル

### プロジェクトフォルダの作成

```bash
mkdir -p ~/projects/t-display-hello
cd ~/projects/t-display-hello
arduino-cli sketch new t-display-hello
```

### Cursor でフォルダを開く

`Ctrl+Shift+P` → `WSL: Open Folder in WSL` → `/home/muramoto/projects/t-display-hello`

### Claude Code でスケッチを生成

Claude Code パネルの入力欄に以下を入力：

```
TTGO T-Display v1.1（ESP32、ST7789V、135x240）のオンボードTFTに「Hello World」を表示する
Arduinoスケッチを作成してください。TFT_eSPIライブラリを使用し、
t-display-hello/t-display-hello.inoに保存してください。
```

> ※ **後でやり直しています。**  
> 最初に生成されたコードには TFT の初期化コードが不足していたため、Claude Code パネルに追記を依頼しました。

最終的なスケッチの内容：
- `#include <TFT_eSPI.h>` でライブラリをインクルード
- `#define TFT_BL 4` でバックライトピンを定義
- `tft.init()` で ST7789V を初期化
- `tft.setRotation(1)` で横向きに設定
- `tft.fillScreen(TFT_BLACK)` で背景を黒に
- `tft.drawString("Hello World", ...)` で中央に文字を表示
- `digitalWrite(TFT_BL, HIGH)` でバックライトを ON

---

## 14. USB デバイスを WSL に接続する（usbipd）

**操作場所:** PowerShell（管理者）

WSL から直接 USB デバイスにアクセスするために `usbipd` をインストールする。

### インストール

```powershell
winget install usbipd
```

> ※ インストール後は PowerShell を再起動する必要があります。

### デバイス確認

```powershell
usbipd list
```

TTGO T-Display が `USB-Enhanced-SERIAL CH9102 (COM3)` として表示される（BUSID: 2-8）。

### WSL に接続

```powershell
usbipd bind --busid 2-8
usbipd attach --wsl --busid 2-8
```

### WSL 側で確認

**操作場所:** WSL（Ubuntu）ターミナル

```bash
ls /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
# → /dev/ttyACM0
```

---

## 15. コンパイルと書き込み

**操作場所:** WSL（Ubuntu）ターミナル

### 権限エラーへの対処

> ※ **後でやり直しています。**  
> 最初の書き込み試行時に `/dev/ttyACM0 is not readable` エラーが発生。ユーザーを `dialout` グループに追加して WSL を再起動することで解決しました。

```bash
sudo usermod -a -G dialout muramoto
```

PowerShell で WSL を再起動：

```powershell
wsl --shutdown
```

Cursor を再起動し、再度 `usbipd attach --wsl --busid 2-8` を実行。

### コンパイル

```bash
cd ~/projects/t-display-hello/t-display-hello
arduino-cli compile --fqbn esp32:esp32:esp32 t-display-hello.ino
```

成功時の出力例：
```
Sketch uses 265472 bytes (20%) of program storage space.
Global variables use 22060 bytes (6%) of dynamic memory.
```

### 書き込み

```bash
arduino-cli upload --fqbn esp32:esp32:esp32 --port /dev/ttyACM0 t-display-hello.ino
```

成功時の出力例：
```
Chip type: ESP32-D0WDQ6 (revision v1.1)
...
Hard resetting via RTS pin...
```

✅ **T-Display の画面に「Hello World」が表示されれば完了！**

---

## 最終的な環境構成

```
Windows 11
└── Cursor Editor 2.6.20（Windows）[WSL: Ubuntu]
    ├── 左パネル: Claude Code サイドバー
    ├── 中央: エディター（スケッチ編集）
    └── ターミナル（WSL Ubuntu）
        ├── Claude Code 2.1.81（~/projects/ で動作）
        │   └── Connected to Cursor ✓
        └── Arduino CLI 1.4.1
            ├── esp32:esp32@3.3.7
            └── TFT_eSPI（Setup25_TTGO_T_Display.h 有効）
```

## インストールしたもの一覧

| ソフトウェア | バージョン | インストール場所 |
|---|---|---|
| Cursor | 2.6.20 | Windows |
| WSL2 | 2.6.3（Ubuntu） | Windows |
| Node.js | v24.14.0 | WSL 内 |
| npm | 11.9.0 | WSL 内 |
| Claude Code | 2.1.81 | WSL 内（sudo でインストール） |
| Claude Code for VS Code | 最新版 | Cursor 拡張機能 |
| WSL 拡張機能 | 最新版 | Cursor 拡張機能 |
| Arduino CLI | 1.4.1 | WSL 内（~/bin） |
| esp32:esp32 | 3.3.7 | WSL 内 |
| TFT_eSPI | 最新版 | WSL 内（~/Arduino/libraries） |
| usbipd-win | 5.3.0 | Windows |

## 料金

- **Claude Pro: $20/月**（月払い）

## 次回やるなら省けること（反省点）

1. WSL インストール促進メッセージはキャンセルせずそのまま進める
2. ターミナルプロファイルの CMD 切り替えは不要
3. `sudo npm install` で最初からインストール
4. VSIX ファイルの手動インストールは不要。Cursor マーケットから直接インストールできる
5. Cursor は最初から WSL モードでフォルダを開いておく
6. 作業フォルダはシステムフォルダを避ける（`~/projects/` 推奨）
7. `dialout` グループへの追加は書き込み前に済ませておく
8. スケッチ生成時はバックライト制御コードも含めるよう最初から指示する

---

*作成: Claude Code + Claude (Anthropic)*
