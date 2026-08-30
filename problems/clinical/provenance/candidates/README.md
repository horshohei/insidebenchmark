# Anomaly candidate generation

このディレクトリは、`pilot/clinical/prompts/anomaly_generation_v1.md`を共通指示として、別々のサブエージェントが異なるraw PN/DS群から作成した候補を保存します。

`anomaly_generation_assignments.json`にPrompt version、入力episode、担当カテゴリ、出力先を記録します。候補ファイルはサブエージェントの生成物として固定し、root reviewerの採否と難易度評価は`../anomaly_acceptance.json`へ分離して記録します。

採用済み候補は次のコマンドで`../anomaly_detection.jsonl`へ再現可能に統合します。統合時に共通Promptと候補ファイルのSHA-256を検証し、各行へ生成元とroot reviewを付与します。

```bash
.venv/bin/python scripts/assemble_clinical_anomaly.py
```
