# Claude.ai「プロジェクト」でのセットアップ手順

APIキー登録・従量課金なしで、Claude.aiの既存プラン（Free/Pro等）の範囲内でこのCSアシスタントを使う方法です。

## 必要なファイル
- `claude-project-instructions.md` — カスタム指示（システムプロンプト）
- `claude-project-knowledge.md` — カタグルマCSナレッジベース（FAQ・事例・タイプ別アプローチ・テンプレ）

## 手順（メンバーごとに1回）

1. https://claude.ai を開き、ログインする
2. 左サイドバーの「プロジェクト」→「新しいプロジェクト」を作成
   - 名前例：`カタグルマ CS お助けアシスタント`
3. プロジェクトの「指示」（カスタム指示）欄に `claude-project-instructions.md` の中身を貼り付ける
4. プロジェクトの「知識」欄に `claude-project-knowledge.md` をアップロードする
5. 保存後、そのプロジェクト内で新しい会話を開始し、困りごと・質問を入力するだけで使える

## 使い方のコツ
- 質問する際、プラン（ライト/スタンダード等）やオンボ状況が分かればそのまま書き添えると、より的確な回答になります
  - 例：「プラン:スタンダード、オンボ状況:OB①。園長が多忙で活用が停滞している。どう再開を促す？」

## 注意
- Freeプランの場合、利用回数制限に達すると一時的に使えなくなることがあります（時間経過でリセット）。
- 従来の`index.html`（社内Webアプリ版）は、会社としてAPIキーの費用負担を引き受ける場合の選択肢として残していますが、今回はこちらのプロジェクト方式を優先運用とします。

## ナレッジベースの更新
`kb.js` を編集後、以下のコマンドで `claude-project-knowledge.md` を再生成できます（開発者向け）。

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

新しいFAQや事例を追加した後は、更新された `claude-project-knowledge.md` を各自のプロジェクトの「知識」に再アップロードしてください。
