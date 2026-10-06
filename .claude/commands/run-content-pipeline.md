---
description: 発信パイプライン(5体)を順番に実行する。引数があればそのテーマ、なければダッシュボードの未処理の依頼を1件処理する
argument-hint: "[テーマ(任意)]"
---
発信パイプラインを実行します。`CLAUDE.md` のダッシュボードURLと `.claude/status-protocol.md` を先に読んでください。

## 0. 依頼を決める
- 引数 `$ARGUMENTS` があれば、それをテーマにする(読者像・プラットフォームは `CLAUDE.md` の既定値)。
- なければ ArtifactData で `requests` を `status == "pending"` で query し、`createdAt` が最も古い1件を取る。無ければ「処理待ちの依頼はありません」と伝えて終了。
- 依頼ドキュメントがある場合は `status` を `running` に update する(`if_version` 付き)。

## 1. 準備
- `runId` = `YYYYMMDD-HHMM` (JST)。`runs/{runId}/` を作り、`input.md` に依頼内容(テーマ / 読者像 / プラットフォーム / 補足)を書く。
- `status/status.json` を初期化(全 stage を idle、`state: running`、`topic`、`runId`、`updatedAt`)。ダッシュボードの `pipeline/current` も同じ内容で `set` する。

## 2. 5体を順番に呼ぶ
Agent ツールで次の順に1体ずつ呼ぶ。各体のプロンプトには `runId` と、読むべきファイルのパスだけを渡す(他の体の出力内容や自分の意見を要約して渡さない)。
1. `content-strategist` → `angle.md`, `handoff-1-2.md`
2. `content-researcher` → `research.md`, `handoff-2-3.md`
3. `content-writer` → `draft.md`, `handoff-3-4.md`
4. `content-editor` → `edited.md`, `handoff-4-5.md`
5. `content-publisher` → `final.md`

各体は自分の stage の状態を自分で報告する。ある体が `blocked` を返したら、そこで止めて理由を `pipeline/current.note` に書き、ユーザーに何が必要か伝える。続行しない。

## 3. 完了処理
- `pipeline/current` を `state: done` に update。
- `runs` コレクションに1件追加(doc_id = runId): `{topic, platform, finishedAt, angle(angle.mdの角度1行), hook(フック1行), final(final.mdの全文)}`。
- 依頼ドキュメントがあれば `status` を `done` に。
- ユーザーへの報告は、角度・削減率・最終ファイルの場所(`runs/{runId}/final.md`)を3-4行で。

## 4. ツールが使えない場合
ArtifactData が無い環境では DB の更新を飛ばし、`status/status.json` と `runs/` だけで完走する。その旨を最後に1行伝える。
