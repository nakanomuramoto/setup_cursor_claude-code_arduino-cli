# Claude Code にコンテキストを引き継ぐ方法

作業の途中でセッションが切れたり、別のプロジェクトに移ったりしたときに、  
これまでの経緯や設定を Claude Code に伝える方法をまとめます。

---

## 方法1: CLAUDE.md に書く（最も推奨）

**操作場所:** WSL ターミナル または Claude Code パネル

Claude Code はプロジェクトフォルダに `CLAUDE.md` というファイルがあると**起動時に自動で読み込みます**。毎回説明不要になるため、最も確実な方法です。

### 既存ファイルをそのままコピーする場合

```bash
cp ~/cursor-claude-code-setup/install-guide.md ~/projects/t-display-hello/CLAUDE.md
```

### Claude Code に要約・作成を依頼する場合

Claude Code パネルの入力欄に以下を入力：

```
install-guide.md の内容を要約して CLAUDE.md として保存してください
```

### CLAUDE.md に書くと良い内容の例

```markdown
## プロジェクト概要
TTGO T-Display v1.1（ESP32 + ST7789V TFT）に Arduino スケッチを書き込む環境

## 環境
- OS: Windows 11 + WSL2（Ubuntu）
- エディター: Cursor 2.6.20
- ボード: ESP32 Dev Module（FQBN: esp32:esp32:esp32）
- シリアルポート: /dev/ttyACM0
- ライブラリ: TFT_eSPI（Setup25_TTGO_T_Display.h 有効）

## よく使うコマンド
arduino-cli compile --fqbn esp32:esp32:esp32 スケッチ名.ino
arduino-cli upload --fqbn esp32:esp32:esp32 --port /dev/ttyACM0 スケッチ名.ino

## 注意事項
- USB書き込み前に usbipd attach --wsl --busid 2-8 を実行すること（PowerShell）
- PATH に /mnt/e/work/bin を追加済み（~/.bashrc）
```

---

## 方法2: `@ファイル名` でメンションする

**操作場所:** Claude Code パネル

Claude Code パネルの入力欄でファイルを直接参照できます：

```
@install-guide.md を参照して、次のステップを教えてください
```

```
@t-display-hello.ino のコードを改善してください
```

ファイルを開いた状態でコードを選択すると、Claude Code パネルに選択内容が自動で反映されます。

---

## 方法3: `/memory` コマンドで記憶させる

**操作場所:** Claude Code ターミナル（WSL）

```
/memory
```

と入力すると記憶管理画面が開き、プロジェクト固有の情報を永続的に保存できます。  
セッションをまたいで記憶が引き継がれます。

---

## 方法4: セッションを再開する

**操作場所:** WSL ターミナル

Claude Code は前回のセッションを再開できます：

```bash
claude --resume
```

または起動時に表示される以下のコマンドでも再開できます：

```bash
claude --resume セッションID
```

---

## まとめ

| 方法 | 持続性 | 手間 | 向いている用途 |
|---|---|---|---|
| CLAUDE.md | ✅ 永続 | 一度だけ | プロジェクト全体の設定・経緯 |
| @メンション | ❌ その都度 | 少ない | 特定ファイルの参照 |
| /memory | ✅ 永続 | 少ない | よく使う指示・好み |
| --resume | ❌ セッション限定 | なし | 中断した作業の続き |

**おすすめの運用:** `CLAUDE.md` にプロジェクト概要・環境・注意事項を書いておき、  
セッションごとの細かい指示は `@メンション` で補う。

---

*作成: Claude (Anthropic)*
