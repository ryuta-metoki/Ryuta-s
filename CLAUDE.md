# 1人社長 5体エージェント構成

業務を5工程(戦略 → 事実 → 実行 → 削減 → 配信)に分け、各工程を独立した context window のサブエージェントに任せる。1つのセッションに全役割を兼任させると起きる context 汚染を、構造で防ぐ。

## ダッシュボード
- URL: https://claude.ai/artifact/YWSfZCbDpMBzmyLdxWXJfh
- 5体の状態・依頼フォーム・最近の成果物を1画面で見られる。スマホからも依頼できる。
- DB: `pipeline/current`(いまの状態) / `requests`(依頼) / `runs`(成果物)。

## 既定値(依頼に指定がなければ使う)
- 読者像: 1人会社経営者・フリーランス
- プラットフォーム: note記事 + Xスレッド
- トーン: ですます調

## 5体と受け渡し
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
- 一括: `/run-content-pipeline [テーマ]`
- 1体ずつ: 「content-strategist を使って runs/{runId}/input.md を処理して」のように呼ぶ。最初の数回はこちらで出力を確認しながら進める。

## 改善ループ
毎週1回、`docs/weekly-review.md` の手順で agent.md を更新する。
