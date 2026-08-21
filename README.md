# カタグルマ CS お助けアシスタント

CSメンバーが顧客対応で躓いたときに、**FAQ・ナイスアシスト事例・テンプレ運用ノウハウ**をもとに最適な打ち手をAIが提案する社内ツールです。

- 想定する相談：①システムのドメイン知識・仕様の質問 ②「こういう使い方できる？」の応用相談 ③忙しくて活用が停滞しているユーザーへのアプローチ相談
- 構成：**ウェブアプリ（`index.html`）＋Claude.aiの「プロジェクト」機能**の組み合わせ。ウェブアプリはAIを直接呼び出さず、入力内容から質問文を組み立ててコピーするだけ。実際の回答生成は各自のClaude.ai（Free/Pro等、既存のアカウント）で行うため、**追加のAPI費用は発生しません**。
- 配布：GitHub Pages（公開URL）＋簡易パスコード。顧客法人名・個人名は**匿名化済み**。

---

## ファイル構成
| ファイル | 役割 |
|---|---|
| `index.html` | アプリ本体（質問文の組み立て・コピー・Claude.aiへの導線） |
| `kb.js` | 知識ベースの原本（匿名化済みのFAQ・事例・タイプ別アプローチ・テンプレ） |
| `claude-project-instructions.md` | Claude.aiプロジェクトの「カスタム指示」に貼り付ける文章 |
| `claude-project-knowledge.md` | Claude.aiプロジェクトの「知識」欄にアップロードするナレッジベース（`kb.js`から自動生成） |
| `CLAUDE_PROJECT_SETUP.md` | Claude.aiプロジェクトの作成手順（メンバー向け） |
| `README.md` | このガイド |
| `_serve.ps1` | ローカル動作確認用の簡易サーバ（配布不要・GitHubに上げなくてOK） |

---

## 全体の流れ

```
[CSメンバー] index.html で質問を入力
      ↓ 「質問文をコピーする」
[クリップボード] 整形済みの質問文
      ↓ 貼り付け
[Claude.ai プロジェクト]（knowledge に claude-project-knowledge.md 登録済み）
      ↓
回答（社内ナレッジを踏まえた提案）
```

## セットアップ（初回のみ）

### 1. Claude.aiプロジェクトを作る（メンバー各自）
`CLAUDE_PROJECT_SETUP.md` の手順に沿って、各CSメンバーが自分のClaude.aiアカウントに「カタグルマ CS お助けアシスタント」プロジェクトを作成します。
- カスタム指示：`claude-project-instructions.md` の中身を貼り付け
- 知識：`claude-project-knowledge.md` をアップロード

### 2. ウェブアプリを配布（管理者が1回）
1. GitHubでリポジトリを新規作成（例：`katagrma-cs-assistant`）
2. `index.html` と `kb.js` をアップロード
3. リポジトリの **Settings → Pages** → Source を `Deploy from a branch` → `main` / `/(root)` に設定して保存
4. 数分後に `https://<アカウント>.github.io/katagrma-cs-assistant/` が発行されるので、CSメンバーに共有

> ⚠️ GitHub Pagesを無料プランで使う場合、リポジトリは**公開（Public）**にする必要があります。URLと中身（`kb.js`含む）は誰でも見られる状態になるため、**社内限定でURLを共有**し、下記のパスコードも設定してください。法人名・個人名や機微な内容は`kb.js`に書かないでください（匿名化・中立表現を徹底）。

### 3. 各メンバーの初回設定
1. 共有されたURLを開く → **パスコード**を入力（既定：`katagrma-cs`）
2. 右上「⚙ 設定」→ 1で作った**自分のClaude.aiプロジェクトのURL**を登録（任意。登録すると「Claude.aiを開く」ボタンでそのプロジェクトに直接ジャンプできます）
3. 困りごと／質問を入力して「質問文をコピーする」→ Claude.aiを開いて貼り付け

---

## 使い方のコツ
- プラン・オンボ状況を指定すると、より的確な文脈で質問文が作られます。
- 停滞ユーザーの相談は、状況（園長多忙・運用バラバラ・以前断られた等）を書くほど、該当タイプの打ち手＋声かけ例が出やすくなります。
- Freeプランのメンバーは、利用回数制限に達すると一時的に使えなくなることがあります（時間経過でリセット）。

---

## 知識ベースの更新
新しいFAQや事例が溜まったら、次の手順で反映します。

1. `kb.js` を編集（**顧客法人名・個人名は必ず役割表現**＝園長／主任／理事長／本部担当 等に置換）

```js
// 事例を追加する例（cases 配列に追記）
{ theme:"見出し", service:"人財育成", situation:"状況", approach:"打ち手", result:"結果" },

// FAQを追加する例（faq 配列に追記）
{ fn:"個人面談", q:"質問", state:"OB①", a:"回答" },
```

2. `claude-project-knowledge.md` を再生成（開発者向け・Node.jsが必要）

```bash
node -e "
global.window = {};
require('./kb.js');
const k = window.KB;
let s = '# カタグルマ CS ナレッジベース（匿名化済み・'+k.updated+'）\n\n';
s += '## FAQ（フォロー手順Q&A）\n';
k.faq.forEach((x)=>{ s+='- [FAQ:'+x.fn+'] Q:'+x.q+'（対象:'+x.state+'）\n  A:'+x.a+'\n'; });
s += '\n## 停滞ユーザー タイプ別アプローチ表\n';
k.approachTypes.forEach(t=>{ s+='- タイプ「'+t.type+'」\n  特徴:'+t.feature+'\n  対策:'+t.policy+'\n  具体例:'+t.example+'\n'; });
s += '\n## ナイスアシスト事例（状況→打ち手→結果）\n';
k.cases.forEach(c=>{ s+='- 事例「'+c.theme+'」('+c.service+')\n  状況:'+c.situation+'\n  打ち手:'+c.approach+'\n  結果:'+c.result+'\n'; });
s += '\n## テンプレ・運用ノウハウ\n';
k.templates.forEach(t=>{ s+='- ['+t.fn+'] '+t.idea+'（汎用性:'+t.scope+'）\n'; });
require('fs').writeFileSync('claude-project-knowledge.md', s);
"
```

3. 更新された `claude-project-knowledge.md` を、各自のClaude.aiプロジェクトの「知識」に再アップロード（各メンバーが個別に行う必要があります）

`kb.js` 内の `updated` の日付も更新すると、アプリ画面フッターに反映されます。

## パスコードの変更
`index.html` 内の `const PASSCODE_PLAIN = "katagrma-cs";` を書き換えて再アップロード。
※簡易的な入口ガードで本格認証ではありません。本当に秘匿したい情報は`kb.js`に載せないでください。

## ローカルで試す（任意）
PowerShellで `./_serve.ps1` を実行 → ブラウザで `http://localhost:8791/` を開く。

---

## 注意事項
- 本ツールはAI生成の提案です。仕様・運用の最終確認と顧客対応の判断はCS担当者が行ってください。
- `kb.js` は匿名化済みですが、率直な社内評価コメント等の追記は避け、載せる場合も中立表現にしてください（公開URLのため）。
- ウェブアプリ自体はAI呼び出しを行わないため、質問文の内容はAnthropic以外のどこにも送信されません（クリップボードにコピーされるのみ）。実際にClaude.aiへ送られた後の扱いは、各自のClaude.aiアカウントの利用規約に準じます。
