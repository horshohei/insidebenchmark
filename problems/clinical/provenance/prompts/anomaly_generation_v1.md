# Clinical anomaly generation prompt v1.0.2

## 目的

J-ClinicalBenchのraw Progress Notes（PN）と対応するraw Discharge Summary（DS）から、候補モデルへ「一部を改変したPN一式だけ」を提示して医学知識を要する異常を検出させる問題候補を作る。正しい原文やDSは候補モデルへ提示しない。生成担当者はPNを主たる事実源、DSを経過確認用の副資料として読む。

## 入力

- 割り当てられた入院エピソードの全PN
- 対応するDS
- 本指示Prompt
- 割り当てられた異常カテゴリと出力先

raw本文は派生JSONLへ複製しない。`raw_path`、行番号、文字offset、原文行全体のSHA-256と、置換文字列だけを記録する。

## 出力件数

各担当は異なる4エピソードから次を作る。

- synthetic anomaly（改変あり）2件
- clean control（無改変）2件

一つの問題に改変は一つだけとする。問題を成立させられない場合は、曖昧な問題を無理に作らず、`rejected`レコードに理由を記録する。

## 許可する異常カテゴリ

- `treatment_class_mismatch`: 疾患に必要な薬効分類と投与薬の分類が明確に異なる
- `pathogen_treatment_mismatch`: 病原体・培養結果と抗菌・抗ウイルス・抗真菌治療が明確に合わない
- `organ_anatomy_mismatch`: 受傷機転・部位・症候・解剖と診断臓器が合わない
- `diagnosis_finding_mismatch`: 検査・画像・身体所見の組合せと診断が明確に合わない
- `dose_frequency_route_mismatch`: 投与量・頻度・経路が明確に誤っている
- `ade_response_mismatch`: 低血糖、薬疹、薬剤性臓器障害等の発生後に被疑薬を増量・再投与するなど対応が逆方向
- `none`: clean control

## 作問規則

1. 候補モデルに正しい原文を並べて見せない。runtimeでは対象エピソードの全PNを読み、指定substringを`replacement`へ一度だけ置換して提示する。
2. 誤りは薬剤名・薬効分類・臓器名・診断・用量・頻度・経路・治療方向のいずれか一つに限定する。
3. 語調、文法、書式を原文に合わせ、表面的には診療記録として読める最小変更にする。
4. 単なる文字列照合ではなく、少なくとも二つの臨床的手掛かりと一つの医学的原則を統合しなければ異常と判断できないものにする。
5. ただし珍しいが妥当な治療、施設差が大きい治療、患者背景によって適否が逆転し得る治療は使わない。明確性を難しさより優先する。
6. 元資料の問題は、候補に提示するPNだけで許可カテゴリの医学的異常が成立する場合、またはDSとの不一致が作成するgoldを不確かにする場合に不採用とする。単純な誤字・書式揺れ、候補に提示しないDSだけの表現差、記載がないだけの治療は、それ自体を異常とみなさない。ただし投与単位の異常、矛盾した臓器・診断、経過と両立しないcarry-forward等、候補が別の医学的異常として合理的に指摘できるPNは使わない。
7. `dose_frequency_route_mismatch`は桁違い、単位違い、投与経路の不可能性など明白なものを優先し、境界的な用量調整を避ける。
8. 正しい代替治療が複数ある場合、goldの`expected_concept`は特定商品名ではなく「抗菌薬」「被疑薬中止」「腎損傷」のような必要最小限の概念にする。
9. clean controlは無改変のPN一式を提示する。全PN・DSを監査し、候補へ提示するPN中に許可カテゴリの明白な医学的不整合がないこと、DSがその判断を覆さないことを確認する。診療録に書かれていない処置を「実施されなかった」と推定してはならない。
11. 候補モデルへの回答条件にも、単純な誤字・書式揺れ・記載省略だけは異常に数えず、許可カテゴリの医学的整合性を判定するよう明記する。
10. 薬剤、用量、禁忌、標準治療を根拠にする問題は、PMDA添付文書、行政機関、学会ガイドライン、査読済み論文の順に一次性・権威性を優先して`source_basis`へ記録する。候補モデルには提示しない。

## 採点対象

主スコアは次だけで構成し、自由記述の治療提案は主スコアに含めない。

- `has_anomaly`
- 異常を含む`sentence_id`
- `anomaly_type`
- 改変されたentityまたは数値
- `expected_concept`

説明文は将来のLLM judgeまたは人手監査用に保存する。

## JSONL schema

採用候補は1行1objectで次の形にする。

```json
{
  "item_id": "ANOM-A01",
  "episode_id": "...",
  "task_type": "clinical_anomaly_detection",
  "source_documents": {
    "pn_paths": ["data/..."],
    "ds_path": "data/..."
  },
  "is_anomalous": true,
  "mutation": {
    "raw_path": "data/.../PN/example_p1.txt",
    "sentence_id": "L12",
    "char_start": 4,
    "char_end": 8,
    "sha256": "sha256 of the full original line",
    "replacement": "改変後文字列",
    "mutation_class": "treatment_class_mismatch"
  },
  "gold": {
    "has_anomaly": true,
    "anomaly_sentence_ids": ["PN p1:L12"],
    "anomaly_type": "treatment_class_mismatch",
    "corrupted_entity": "改変後entity",
    "expected_concept": "必要最小限の正しい概念",
    "clinical_basis": "なぜ異常か。原文復元だけでなく医学的理由を説明する",
    "required_clues": [
      {
        "raw_path": "data/...",
        "sentence_id": "L5",
        "char_start": 0,
        "char_end": 10,
        "sha256": "sha256 of the full clue line"
      }
    ]
  },
  "source_basis": [
    {
      "title": "根拠資料名",
      "url": "https://...",
      "supports": "異常判定を支える要点"
    }
  ],
  "generation": {
    "prompt_version": "clinical_anomaly_generation_v1.0.2",
    "assignment_id": "agent_group_id"
  }
}
```

clean controlでは`is_anomalous=false`、`mutation=null`とし、goldは次の形にする。

```json
{
  "has_anomaly": false,
  "anomaly_sentence_ids": [],
  "anomaly_type": "none",
  "corrupted_entity": null,
  "expected_concept": null,
  "clinical_basis": "全PNとDSを照合してclean controlとして採用した理由",
  "required_clues": []
}
```

不採用は別の`.rejected.jsonl`へ次の形で書く。

```json
{
  "episode_id": "...",
  "status": "rejected",
  "reason": "元資料の矛盾、医学的曖昧性、十分な手掛かり不足等",
  "generation": {
    "prompt_version": "clinical_anomaly_generation_v1.0.2",
    "assignment_id": "agent_group_id"
  }
}
```

## 提出前チェック

- raw substringとoffsetが完全一致する
- SHA-256がUTF-8原文行全体から計算されている
- 改変は一か所だけである
- required cluesが2件以上ある
- correct原文を候補入力へ別途提示していない
- source basisが異常判定を直接支える
- clean controlに元からある明白な矛盾を見落としていない
- JSONLとしてparseできる
