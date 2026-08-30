# Historical knowledge essays

資料を提示せず、モデルの学習済み歴史知識を用いて背景・転換・帰結を論述させるclosed-prompt / open-knowledge trackです。推論パズルではありません。

6問を収録し、各問は問題固有の内容基準10項目で採点します。各項目は0/1のbinary判定で、候補回答に対象の史実または因果関係が正確かつ明示的に書かれている場合だけ1点です。文章の巧拙や基準外の知識は加点せず、各問10点満点です。採点は`history_knowledge_essay_v2` rubricによるLLM-as-a-judgeで行います。

- `medieval_dispute_and_kenka_ryoseibai`: 中世の訴訟・自力救済と喧嘩両成敗法
- `yamato_river_diversion`: 17世紀以前の大和川と1704年の付替え、その地域別帰結
- `arab_uprisings_2011`: 2010–11年のアラブ蜂起とリビア・シリアの2024年末までの帰結
- `shinjitai_character_mergers`: 欠・缺、芸・藝の別字合流と戦後字体整理
- `shinbutsu_shugo_and_meiji_separation`: 奈良・熊野の神仏習合、明治の神仏分離と現在の景観
- `imperial_japan_opium_policy_and_hoshi`: 後藤新平、台湾阿片専売、星製薬と大陸政策の史料批判

`reference.source_basis`は問題・採点基準作成時の根拠であり、候補モデルには提示しません。第3問の知識基準日は2024-12-31です。第6問は問題文中の主張を前提として追認させず、確認できる制度・人脈と、個人蓄財・直接的資金継承という別途立証を要する主張を区別させます。
