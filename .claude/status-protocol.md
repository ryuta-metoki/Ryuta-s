# 状態の報告プロトコル

ダッシュボード (`CLAUDE.md` に URL) が会社全体の状態を表示するために、各エージェントは**作業の開始時と完了時**に状態を書く。各 agent.md から参照される共通手順。

## 部署と stage
- 部署 id: `content`(発信) / `sales`(営業) / `support`(サポート) / `research`(リサーチ) / `exec`(経営判断)
- stage id: `strategist` / `researcher` / `writer` / `editor` / `publisher`
- 自分の部署は `runs/{runId}/input.md` の「部署」で分かる。

## 書く内容
`stages.<stage id>` に次の3項目:
- `state`: `running` / `done` / `blocked`
- `summary`: 何をしている/したかを60字以内で1文。完了時は成果の要点(数字を入れる)
- `updatedAt`: 現在時刻 (ISO 8601, UTC)

## 書く場所(両方)
1. `status/<部署id>.json` (必ず): `stages.<id>` を上書きし、トップレベル `updatedAt` も更新する。
2. ダッシュボードDB (`ArtifactData` ツールが使えるときだけ): `url` は `CLAUDE.md` の「ダッシュボード」、`collection=departments`、`doc_id=<部署id>`。`get` で `version` を得て、`update` で `{"stages":{"<id>":{...}},"updatedAt":"..."}` を `if_version` 付きで書く。使えない・失敗したときは 1 だけでよい。止まらない。

## 要確認の出し方(blocked)
入力不足や社長の判断が必要なとき:
1. 自分の stage を `blocked` にし、summary に何が止まったかを書く。
2. `escalations` コレクションに新規1件(doc_id は `<runId>-<stage id>`): `{dept, stage, runId, question(社長に聞きたいこと1-2文。選択肢があれば並べる), status:"open", createdAt}`。ArtifactData が使えなければ `runs/{runId}/escalation.md` に書く。
3. 作業を終了して親に戻る。推測で埋めない。

## 親(`/run-pipeline`)が書くもの
部署の `state`(`idle`/`running`/`done`/`blocked`)・`topic`・`runId`・`note`、`runs` 履歴、`requests` の状態。エージェントは自分の stage と escalation だけ書く。
