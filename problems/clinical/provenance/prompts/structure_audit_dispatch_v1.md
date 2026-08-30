# Clinical structure/audit subagent dispatch record v1.0.0

## 実行日

2026-08-28

## 共通dispatch指示

各担当サブエージェントには次を指示する。

1. `pilot/clinical/prompts/structure_audit_expansion_v1.md`を最後まで読む。
2. `pilot/clinical/structure_audit_generation_assignments.json`の自分のassignmentだけを担当する。
3. 割当episodeのraw PN全件とraw DSを全文読む。
4. episodeごとにstructured候補1問と、指定labelのaudit候補1問を作る。
5. 既存`structured.jsonl`と`audit.jsonl`を読み、対象PNと監査主張の重複を避ける。
6. 原文行全体のSHA-256、offset、pathを機械的に再計算する。
7. 自分のcandidate JSONLだけを`apply_patch`で作成し、データ本体・コード・manifestを変更しない。

## 分担

- `structure_audit_01_03`: 高Ca血症、HHS
- `structure_audit_05_07`: 術後PE、尿路感染症
- `structure_audit_10_qc`: 胆管結石胆管炎、および共通schema観点の自己点検

割当episode、追加audit label、candidate出力先は
`pilot/clinical/structure_audit_generation_assignments.json`を唯一の割当記録とする。
