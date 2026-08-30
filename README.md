# Japanese LLM benchmark pilot

LLM-jp-4 33B Dense、LLM-jp-4 32B-A3B、Qwen3.8-27Bを比較した際の問題と結果です。

## 収録内容

| Track | 問題数 | 採点 |
|---|---:|---|
| 臨床構造化抽出 | 10 | 自動採点 |
| PN–DS根拠監査 | 10 | 自動採点 |
| 医学的異常検知 | 12 | 自動採点 |
| 歴史知識論述 | 6 | 10個のbinary基準をLLM-as-a-judgeで採点 |
| h-nabata Challenge v1.1.0 | 80 | 自動採点 |

各問題をThinking LowとMediumで実行したため、1モデル当たり236件、3モデル合計708件の候補推論を収録しています。

```text
problems/
├── clinical/   # 構造化抽出、PN–DS監査、異常検知、作成記録
├── history/    # 論述問題と問題別binary採点基準
└── challenge/  # h-nabata Challenge v1.1.0
results/
├── llmjp/      # LLM-jp-4 33B Dense
├── llmjp_a3b/  # LLM-jp-4 32B-A3B
├── qwen/       # Qwen3.8-27B
└── comparison/ # 3モデルの集計結果
```

## 臨床問題の原文

臨床問題は[J-ClinicalBench](https://github.com/seiji-shimizu/J-ClinicalBench-release)のraw Progress NoteとDischarge Summaryを参照します。raw文書はこのrepositoryには複製していません。

使用したsource commitは次のとおりです。

```text
b9333fb8f98a170499f1e0aa8555d02831c5eb36
```

J-ClinicalBenchをrepository rootの次の位置へ配置すると、問題JSONLの`raw_path`と一致します。

```text
data/J-ClinicalBench-release/
```

臨床問題のgoldはCodexで作成したpilot annotationであり、臨床専門家による検証前です。研究評価専用で、診療判断には利用しないでください。

## 結果ファイル

各モデルのディレクトリには、次のファイルがあります。

- `pilot_<track>.jsonl`: モデルのreasoning、最終回答、token数、wall time
- `pilot_<track>_scores.json`: 自動採点結果
- `pilot_history_judgments.jsonl`: provider固有のresponse metadataを除いた歴史論述のJudge結果

主な条件はBF16、vLLM、common context 65,536 token、seed 42、各条件1回です。全708件が`finish_reason=stop`で終了しています。

## 主結果

LowとMediumを合わせ、各Track内で0–100へ正規化した値です。

| モデル | 臨床 | 医学的異常検知 | 歴史論述 | Challenge |
|---|---:|---:|---:|---:|
| LLM-jp-4 33B Dense | 58.2 | 59.7 | 37.5 | 80.3 |
| LLM-jp-4 32B-A3B | 55.7 | 51.4 | 38.3 | 81.1 |
| Qwen3.8-27B | 73.1 | 75.0 | 43.3 | 98.1 |

より細かい問題種別、Thinking設定、ラップタイムは[`results/comparison/comparison.md`](results/comparison/comparison.md)を参照してください。

## 注意

- 各条件は1回のみのpilot実験です。
- 4 Trackは問題数と採点方法が異なるため、恣意的な総合点は作っていません。
- ChallengeはQwenでほぼ飽和しています。
- 権利関係と出典は[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)を参照してください。

