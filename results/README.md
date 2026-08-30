# Results

主比較に使用した3モデルの候補出力、自動採点、歴史論述のJudge結果を収録しています。

| Directory | Model | Source result directory |
|---|---|---|
| `llmjp` | LLM-jp-4 33B Dense Thinking | `results/deployment_v06/llmjp` |
| `llmjp_a3b` | LLM-jp-4 32B-A3B Thinking | `results/deployment/llmjp_a3b` |
| `qwen` | Qwen3.8-27B | `results/deployment_v06/qwen` |

`pilot_history_judgments.jsonl`は最終的に成功した36件を収録し、OpenAI APIのprovider response IDなどは除いてあります。候補モデル名と候補側のLow/MediumはJudge promptには渡していません。

候補出力は内容を変更せず`pilot_<track>.jsonl.gz`として圧縮しています。展開例は次の通りです。

```bash
gzip -dk pilot_challenge.jsonl.gz
```
