# Clinical longitudinal pilot

J-ClinicalBenchのraw Progress Noteと対応するraw Discharge Summaryを用いた内部評価用pilotです。原文はこのディレクトリへ複製せず、`raw_path`、行ベースの`sentence_id`、文字範囲、SHA-256で参照します。

収録物は、5入院エピソードの参照一覧、統一schemaによるstructured extraction 10問、PN–DS consistency audit 10問、医学的異常検知12問、Disease NER 50文です。NERには48 entityとentityなしのnegative control 17文を含みます。問題・goldは文書内情報だけから作成しています。NERはCodex作成のreferenceであり、臨床専門家による検証前です。

`sentence_id`の`L<n>`は、Python `splitlines()`で得たrawファイルの1始まり行番号です。`char_start`と`char_end`はその行内の0始まり半開区間です。NER entityの`span`も参照行内の半開区間です。

このpilotは研究評価専用であり、診療判断には使用しません。rawデータおよびモデル出力は、原資料と同じアクセス制御下で扱ってください。

Structured extractionは、指定した単一PNから次の共通フィールドを抽出します。

- `diagnoses`: 病名と確実性（confirmed / suspected）
- `medications`: 薬剤・輸液名、投与量、単位、頻度、経路、開始・中止等のaction
- `vital_signs`: 体温、脈拍、血圧、呼吸数、SpO2、酸素投与条件
- `procedures`: 処置・手術名と実施済み／予定

明示されない値は`null`とし、一般医学知識で補いません。全10問で`clinical_structure_v1.0.0`を使い、内容と根拠行を自動採点します。検査値は文書間で項目数が大きく異なるため、この基本schemaには混在させず、独立したlab extraction taskの候補とします。アレルギー歴も対象PNに明記されない例が多いため基本schemaから除外します。PN–DS監査は10問で、5種類のlabelを各2問収録し、labelと根拠行を自動採点します。臨床trackにLLM-as-a-judgeは不要です。

追加5問ずつの生成Prompt、症例別サブエージェント分担、候補、root reviewの採否記録は、`structure_audit_generation_assignments.json`、`structure_audit_candidates/`、`structure_audit_acceptance.json`に保存しています。
