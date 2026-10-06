# 状態の報告プロトコル

ダッシュボード (`CLAUDE.md` に URL) が「いま何をしているか」を表示するために、各エージェントは**作業の開始時と完了時**に状態を書く。これはエージェント定義ではなく共有手順で、各 agent.md から参照される。

## 書く内容
`stages.<stage id>` に次の3項目:
- `state`: `running`(作業中) / `done`(完了) / `blocked`(入力不足や判断待ちで止まった)
- `summary`: 何をしている/したかを60字以内で1文。完了時は成果の要点(数字を入れる)
- `updatedAt`: 現在時刻 (ISO 8601, UTC)

## 書く場所(両方)
1. `status/status.json` (必ず): `stages.<id>` を上の3項目で上書きし、トップレベルの `updatedAt` も更新する。
2. ダッシュボードDB (`ArtifactData` ツールが使えるときだけ): `url` は `CLAUDE.md` の「ダッシュボード」、`collection=pipeline`、`doc_id=current`。まず `get` して `version` を得て、`update` で `{"stages":{"<id>":{...}},"updatedAt":"..."}` を `if_version` 付きで書く。ツールが無い・失敗したときは 1 だけでよい。止まらない。

`blocked` にしたときは `summary` に「何が足りないか」を書き、作業を終了して親に戻る。推測で埋めない。

実行全体の状態(`state`・`topic`・runs 履歴・依頼ステータス)は `/run-content-pipeline` が書く。エージェントは自分の stage だけ書く。
