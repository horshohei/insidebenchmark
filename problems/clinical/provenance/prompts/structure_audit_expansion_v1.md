# Clinical structured extraction / audit expansion prompt v1.0.0

## 目的

既存5症例から、`clinical_structured_extraction`と`pn_ds_consistency_audit`を各1問追加する。
最終データは各タスク10問とし、追加分は既存問題と異なるPN・異なる主張を使う。

## 入力

- 担当episodeのraw Progress Note全件
- 同episodeのraw Discharge Summary
- `pilot/clinical/episodes.jsonl`
- `pilot/clinical/structured.jsonl`
- `pilot/clinical/audit.jsonl`
- `pilot/clinical/structured_schema.json`

ファイルは必ず全文を読み、行番号はPython `splitlines()`の1始まりで扱う。

## 共通参照形式

根拠参照は次の形式とする。`sha256`はsubstringではなく、改行を除く原文行全体の
UTF-8 SHA-256とする。`char_start=0`、`char_end=len(raw_line)`を基本とする。

```json
{
  "raw_path": "data/...txt",
  "sentence_id": "L12",
  "char_start": 0,
  "char_end": 42,
  "sha256": "..."
}
```

## Structured extraction候補

1. 既存の同episode問題とは別のPNを`target_document`に選ぶ。
2. 抽出対象はその単一PNだけとする。他のPNやDSからgoldを補わない。
3. 全問で`clinical_structure_v1.0.0`を使い、goldの最上位キーを次の5個に固定する。
   - `document_id`
   - `diagnoses`: `name`, `certainty` (`confirmed` / `suspected`)
   - `medications`: `name`, `dose_value`, `dose_unit`, `frequency`, `route`, `action`
   - `vital_signs`: 体温、心拍、収縮期・拡張期血圧、呼吸数、SpO2、酸素条件
   - `procedures`: `name`, `status` (`completed` / `planned`)
4. 病名・薬剤名・処置名は原文の表層形を使う。
5. 明示されない値は医学知識で推測せず`null`とする。
6. 薬剤actionは`start|continue|stop|administered|planned|unspecified`、routeは
   `oral|intravenous|subcutaneous|other|null`だけを使う。
7. evidenceはtarget PN内だけとし、goldの全要素を直接支持する行を含める。
8. 記載量が極端に少ないPNは避け、病名、薬剤または処置、バイタルのうち複数カテゴリを採点できるPNを優先する。

候補のIDはroot採用時に決めるため、`question_id`は`STR-CAND-<episode prefix>`とする。

## PN–DS consistency audit候補

1. 担当assignmentで指定されたlabelの問題を1問作る。
2. labelは`SUPPORTED|PARTIALLY_SUPPORTED|CONTRADICTED|NOT_DOCUMENTED|TEMPORALLY_MISLEADING`。
3. 質問文だけで監査対象のDS記載またはDS候補文が完全に分かるようにする。
4. PNを一次根拠とし、一般医学知識を正誤根拠にしない。
5. 既存auditと同じイベント・数値・時系列関係をほぼ言い換えただけの問題は禁止する。
6. `SUPPORTED`は主要部分がすべてPNに明記されること、`PARTIALLY_SUPPORTED`は支持部分と
   非支持部分の両方があること、`CONTRADICTED`はPNに明示的な反証があること、
   `NOT_DOCUMENTED`は提示PN全件に記載がないこと、`TEMPORALLY_MISLEADING`は前後関係が
   PNと異なることを確認する。
7. evidenceは判定を直接支持する最小限のPN行とする。DS行は質問文の由来確認用に
   `source_ds_refs`へ分け、採点goldの`evidence`には入れない。

候補のIDは`AUD-CAND-<episode prefix>`とする。

## 出力

担当のcandidate JSONLへstructured候補1行、audit候補1行の順で保存する。
各行へ次を追加する。

```json
"generation": {
  "prompt_version": "clinical_structure_audit_expansion_v1.0.0",
  "assignment_id": "...",
  "author": "subagent"
}
```

提出前にJSON decode、raw path、行番号、文字範囲、行全体SHA-256を検証する。
データ本体、validator、manifest、他担当ファイルは変更しない。
