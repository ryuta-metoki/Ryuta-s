# 1人社長の会社: 5部署 × 5体

業務を5工程(戦略 → 事実 → 実行 → 削減 → 配信)に分け、各工程を独立した context window のサブエージェントに任せる。1つのセッションに全役割を兼任させると起きる context 汚染を、構造で防ぐ。

## ダッシュボード
- URL: https://claude.ai/artifact/YWSfZCbDpMBzmyLdxWXJfh
- 社長を頂点に5部署・25体の状態、社長への要確認、今日/7日間の実績、依頼フォーム、成果物を1画面で見られる。スマホからも依頼・回答できる。
- DB: `departments/<部署id>`(いまの状態) / `requests`(依頼) / `runs`(成果物と実績) / `escalations`(社長への要確認)。

## いつでも把握するしくみ
- 画面の「いま社長がやること」欄が、状態から次にやることを優先順に並べる(判断待ち → 止まっている部署 → 処理待ちの依頼 → 未確認の成果物 → 稼働していない部署)。
- 判断待ちや完了は `PushNotification` でスマホに通知する。
- 毎朝(JST 8:00頃)と毎週月曜に、自動で `reports` コレクションへレポートを書き、通知する。
- 依頼は毎時の定期実行で自動処理する。

## 既定値(依頼に指定がなければ使う)
- 読者像: 1人会社経営者・フリーランス
- プラットフォーム: note記事 + Xスレッド
- トーン: ですます調

## 5部署
| 部署 | id | エージェント名 |
|---|---|---|
| 発信 | content | content-{strategist,researcher,writer,editor,publisher} |
| 営業 | sales | sales-… |
| サポート | support | support-… |
| リサーチ | research | research-… |
| 経営判断 | exec | exec-… |

外部への送信・投稿・公開は一切しない。下書きまでで止め、実行は社長が行う。返金・契約撤回・法務・経営判断は必ず社長への要確認に上げる。

## 5体と受け渡し(どの部署も同じ)
| 順 | agent | 入力 | 出力 |
|---|---|---|---|
| 1 | content-strategist | input.md | angle.md |
| 2 | content-researcher | angle.md | research.md |
| 3 | content-writer | angle.md + research.md | draft.md |
| 4 | content-editor | draft.md のみ | edited.md |
| 5 | content-publisher | edited.md + 媒体 | final.md |

- 成果物は `runs/{runId}/` に置く。体と体の間は `templates/handoff.md` 形式の `handoff-N-M.md` で引き継ぐ。
- 各体は前の体の出力ファイルだけを読む。後ろの体の都合は見ない。Editor は角度とリサーチを見ない。
- 状態の報告は `.claude/status-protocol.md` に従う。

## 実行
- 一括: `/run-pipeline [部署id] [テーマ]`。部署を省略すると、ダッシュボードの未処理の依頼を古い順に1件処理する。
- 1体ずつ: 「content-strategist を使って runs/{runId}/input.md を処理して」のように呼ぶ。最初の数回はこちらで出力を確認しながら進める。

## 改善ループ
毎週1回、`docs/weekly-review.md` の手順で agent.md を更新する。
