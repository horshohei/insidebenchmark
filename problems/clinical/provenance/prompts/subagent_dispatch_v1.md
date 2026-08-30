# Clinical anomaly subagent dispatch record v1.0.0

## 実行日

2026-08-28

## 共通dispatch指示

各担当サブエージェントには、次の指示を共通して与えた。

1. `pilot/clinical/prompts/anomaly_generation_v1.md`を省略せず最後まで読む。
2. `pilot/clinical/anomaly_generation_assignments.json`の自分の`assignment_id`だけを担当する。
3. 割当エピソードごとに、指定されたraw PN全件と対応するraw DSを全文読む。
4. synthetic anomaly 2件とclean control 2件を、共通PromptのJSONL schemaで作る。
5. 医学的主張は公的機関、学会ガイドライン、添付文書、査読済み論文を優先して調査し、`source_basis`へ記録する。
6. offset、原文行全体のSHA-256、raw path、required cluesを提出前に再計算して検証する。
7. 問題を明確に成立させられない症例は無理に採用せず、割当済みの`.rejected.jsonl`へ理由を残す。
8. 自分の出力JSONL以外のデータセット、manifest、validator、他担当ファイルを変更しない。
9. ファイル編集には`apply_patch`を使う。

## 分担

- `infection_treatment`: 感染症・薬効分類、病原体と治療の不一致
- `anatomy_diagnosis`: 受傷機転・解剖・所見と臓器または診断の不一致
- `dose_ade`: 投与量・頻度・経路、薬剤有害事象後の対応

各担当の入力エピソード、出力先、差替履歴は
`pilot/clinical/anomaly_generation_assignments.json`を唯一の割当記録とする。

## Prompt改訂の通知方法

生成中にPromptを改訂した場合、全稼働担当へ新しいversion、SHA-256、変更点を通知し、
各出力の`generation.prompt_version`には実際に用いたversionを記録する。旧versionと改訂理由は
割当記録の`revision_history`へ残す。既に完了した候補のversionを遡及的に書き換えない。
