# 1人社長の会社: 5部署 × 5体

発信・営業・サポート・リサーチ・経営判断の5部署に、それぞれ5体のサブエージェント(計25体)を置く設計一式。

## 6点セット
1. 5体の agent.md → `.claude/agents/content-*.md`
2. ディレクトリ構造 → 下記
3. 起動コマンド → 下記
4. ハンドオフテンプレート → `templates/handoff.md`
5. 5領域への応用 → 下記の表
6. 週次の改善ループ → `docs/weekly-review.md`

## 構造
```
.claude/agents/        25体 ({content,sales,support,research,exec}-{strategist,researcher,writer,editor,publisher})
.claude/commands/      run-pipeline.md (部署の5体を順に呼ぶ)
.claude/status-protocol.md   状態報告の共通手順
CLAUDE.md              プロジェクト文脈・ダッシュボードURL・既定値
templates/handoff.md   体と体の引き継ぎ用
runs/{runId}/          1回の実行の成果物 (input → angle → research → draft → edited → final)
status/<部署id>.json   いまの状態 (各エージェントが更新)
dashboard/index.html   ダッシュボードのソース (claude.ai に公開済み)
```

## 起動
```
claude                              # このリポジトリで起動
/run-pipeline                       # ダッシュボードの未処理の依頼を古い順に1件処理
/run-pipeline content AIエージェントを1人社長が使う理由   # 部署とテーマを指定
/run-pipeline sales ○○社への提案
/agents                             # 各エージェントの確認
```
最初の数回は、1体ずつ「content-strategist で runs/{runId}/input.md を処理して」と呼び、出力を確認しながら進める。

## 5領域への応用
各部署の役割は次のとおり。

| 領域 | Strategist | Researcher | Writer | Editor | Publisher |
|---|---|---|---|---|---|
| 発信 | 今週の角度 | 一次情報+反証 | 記事・スレ草稿 | 30%削減 | note / X / メルマガ |
| 営業 | なぜこの顧客に | 顧客・競合調査 | 提案書・商談メール | 1ページ要約 | PDF・メール・Slack |
| サポート | 返信戦略の分類 | KB・過去チケット | 返信下書き | 100字以内・トーン | チケット形式・KB差分 |
| リサーチ | 3つの問いに絞る | 動向データ | 週次レポート | 要点絞り | Slack・Notion |
| 経営判断 | 今月のリスク3点 | 指標・市場データ | 意思決定メモ | 3点に削減 | Notion・Slack |

5部署とも作成済み。まず発信で数回回し、agent.md を育ててから他部署に広げる。
